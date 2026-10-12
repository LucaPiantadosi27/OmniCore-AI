# OmniCore AI

Fork di CopilotKit/OpenBot (import vendored da `upstream/main` @ 7b1c20a, MIT).
Le personalizzazioni OmniCore vivono in `examples/omnicore/` (tenant package) e in
moduli nuovi separati dal codice upstream. Vedi `PLAN.md` per il piano completo.

## Branch e remote

- `main`: branch di lavoro OmniCore, pushato su `origin` (repo privato
  LucaPiantadosi27/OmniCore-AI). Storia orfana: **non** contiene i commit upstream,
  quindi il repo mostra solo i contributor di OmniCore.
- `upstream`: remote read-only verso CopilotKit/OpenBot, usato solo in locale per i sync.

### Sync da upstream

```sh
git fetch upstream
# Prima volta soltanto (storie non correlate): risolve i conflitti
# prendendo la versione upstream, sicuro finche' le modifiche OmniCore
# restano in file/directory separati.
git merge --allow-unrelated-histories -X theirs upstream/main
# Dal sync successivo upstream/main e' antenato di main: merge normale.
git merge upstream/main
```

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
