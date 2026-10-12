# OmniCore AI

Fork di CopilotKit/OpenBot. Le personalizzazioni OmniCore vivono in `examples/omnicore/`
(tenant package) e in moduli nuovi separati dal codice upstream, per poter fare merge di
`upstream/main`. Vedi `PLAN.md` per il piano completo.

## Requisiti

- Bun 1.3.14 (packageManager fissato in `package.json`)
- Node 20+, Docker + Compose (per i servizi: Postgres, Intelligence self-hosted, ecc.)
- `.env` generato da `.env.example` (`cp .env.example .env`)

## Comandi

| Azione | Comando |
| --- | --- |
| Installare dipendenze | `bun install` |
| Avvio completo (script + docker) | `bash scripts/start.sh` → http://localhost:3010 |
| Stop | `bash scripts/stop.sh` |
| Dev (app + server) | `bun run dev` |
| Build | `bun run build` |
| Typecheck | `bun run typecheck` |
| Lint | `bun run lint` |
| Format | `bun run format` / check: `bun run format:check` |
| Test | `bun test` |
| Test CI | `bun run test:ci` |
| Smoke test | `bun run test:smoke` |

## Note

- Tenant package selezionato via `TENANT_PACKAGE_DIR` in `.env` (`../examples/omnicore`).
- `OPENBOT_SINGLE_USER=true` solo in locale.
- Branch di lavoro: `omnicore/main`; `main` resta specchio di upstream.
