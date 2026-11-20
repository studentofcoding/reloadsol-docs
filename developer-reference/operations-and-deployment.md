---
description: Production deployment workflow, standalone build shipping, migration preflight, CI quality gates, Cloudflare hardening, and troubleshooting runbooks.
---

# Operations, Deployment & Troubleshooting

reloadSol enforces strict operational guardrails (`docs/ops/CHANGE-RULES.md`) because production runs on a constrained `< 4 GiB RAM` VPS (`flowey-vps`) serving live wallets and automated trading loops.

---

## 1. Local Verification Gate (`npm run ci:local`)

Every change must pass the full local verification gate before merging or shipping:

```bash
rm -rf .next/ \
  && npm run lint \
  && npm run verify:no-raw-useeffect \
  && npm run verify:no-hardcoded-sol-price \
  && npm run verify:no-server-env-in-client \
  && npm run build \
  && npm run start
```

Additional schema & unit checks:
* `npm test` (Vitest) + `go test ./...` (Go cron scheduler tests)
* `npm run db:check-migrations` (double-applies `db/init/*.sql` on a throwaway Postgres + runs `verify:sql` `PREPARE` sweep across 550+ queries)
* `npm run verify:migrations` (confirms every `db/init` file is applied to production)

{% hint style="warning" %}
**Serialized Builds:** Always run `npm run build`—never raw `npx next build`. `npm run build` acquires `/tmp/reloadsol-build.lock` (`scripts/with-build-lock.js`) so concurrent agent or developer sessions queue cleanly instead of starving CPU/RAM.
{% endhint %}

---

## 2. Production Deploy Chain

### Step 1: Apply Migrations *Before* Code Lands
Migrations are **not** run automatically by `docker-deploy.sh`. Because every migration is additive and idempotent (`ADD COLUMN IF NOT EXISTS`, guarded constraints), apply it out-of-band first (after explicit owner approval):

```bash
ssh flowey-vps 'docker exec -i reloadsol-db psql -U reloadsol -d reloadsol_db -v ON_ERROR_STOP=1' \
  < db/init/<NN>-<name>.sql
```

### Step 2: Ship Pre-Built Standalone Bundle (Never Build Next.js on the VPS)
The VPS has `< 4 GiB` RAM and refuses host `next build`. Build on a development machine with RAM and ship via `scripts/ship-standalone-to-vps.sh`:

* Verifies CI green on the head SHA (`scripts/ship-ci-gate.sh`).
* Refuses to ship inside the active trading window (`16:00–04:00 WIB`) unless explicitly overridden (`SHIP_IN_TRADING_WINDOW=1`).
* Acquires `/tmp/reloadsol-deploy.lock` to serialize concurrent deploys (exits `75` if held).
* Tags the running container `reloadsol-web:rollback-pre<sha8>` before swapping.
* Stamps `.next/{standalone,static}/.deploy-git-sha` with the VPS `HEAD`, verifies `onnxruntime-node` native binaries (`verify-standalone-build.sh`), recreates only the scoped container (`up -d --no-deps web`), warms caches (`scripts/warm-cache.sh`), and automatically rolls back if `/api/health` fails.

### Step 3: Smoke-Test with Real Host Header
Nginx `default_server` drops bare-IP requests with `HTTP 444`. Always smoke-test with `Host: reloadsol.app`:

```bash
curl -fsS -H "Host: reloadsol.app" http://127.0.0.1/api/health
```

---

## 3. Environment & Security Guardrails

* **`.env` Preflight:** Keep exactly one `WEB_PORT=` and one `CRON_PORT=` line; `DATABASE_URL` must point at `reloadsol-bouncer:5432/reloadsol_db`; `CRON_SERVICE_URL` must stay `http://cron:8080`.
* **Cloudflare Edge Hardening (`SPEC-cloudflare-edge-hardening-v1`):**
  * Full (Strict) TLS using a 15-year Cloudflare Origin CA certificate on Nginx `:443`.
  * `01-cloudflare-realip.conf` + `getClientIp()` (`CF-Connecting-IP`) for accurate rate limiting.
  * Direct-to-IP origin bypass blocked (`HTTP 444`).
* **Never run `docker compose down -v`:** Destroys `postgres_data` and `redis_data` volumes.

---

## 4. Common Failure Runbooks

| Symptom | Root Cause to Check First | Fix |
| :--- | :--- | :--- |
| **DB `timeout exceeded when trying to connect` every ~90s** | Check `[db-pool]` logs: `waiting=0` with `total` climbing usually indicates **`reloadsol-web` V8 heap OOM** (`FATAL ERROR: Reached heap limit`), not pool exhaustion. | Inspect `docker logs reloadsol-web` for heap limit errors; verify `WEB_NODE_OPTIONS` / container memory (`768M–1G`) and check `[db-slow-query]` (`> 5000 ms`). |
| **502 Bad Gateway after deploy while `web` is healthy** | Stale Nginx upstream or recreated `web` container IP. | Run `docker exec reloadsol-nginx nginx -t && docker exec reloadsol-nginx nginx -s reload`. |
| **Sim/Social history gaps** | `reloadsol-cron` or `reloadsol-social-ingest` stopped (remember `npm run dev` runs no cron). | Check `GET /api/workers/status` and `docker ps`; check `social-ingest` logs for `Ingest OK (200)`. |
| **Real trading halted unexpectedly** | Global circuit breaker tripped (`bot_trading_state.real_trading_halted = true`). | Query `SELECT * FROM bot_trading_state;` — auto-resets after `BOT_TRADING_HALT_MINUTES` or on next success. Sim workers are unaffected. |
| **VPS disk > 75% before deploy** | Old `rollback-pre*` images (~3.1 GiB each) or host caches. | Keep 2 rollback images max; run `scripts/cron-disk-prune.sh`. Never prune Docker volumes or `~/reloadsol-backups/`. |
