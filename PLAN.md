# OmniCore AI - Piano di implementazione

Base: fork/clone di CopilotKit/OpenBot (commit 7b1c20a, MIT, alpha). Il motore (server Hono, gateway
policy/audit, AG-UI, UI React) non si riscrive: si estende tramite configurazione e un livello nuovo
chiamato **Team**.

Convenzioni: ogni fase ha una "Definition of Done" (DoD) verificabile. Non si passa alla fase
successiva senza averla soddisfatta. Le personalizzazioni vanno tenute separate dal codice upstream
(`examples/omnicore/`, moduli nuovi in `server/src/teams/`) per poter fare merge di `upstream/main`.

## Stato verificato

- Clone completato in questa cartella.
- Docker 29.8.2 e Compose v5.1.3 presenti. Node 20.20.2 presente.
- Bun 1.3.14 installato (`~/.bun/bin/bun`), `bun install` eseguito.
- Fase 0 completata: `origin`→`upstream` (CopilotKit/OpenBot), nuovo `origin` privato
  (github.com/LucaPiantadosi27/OmniCore-AI), branch `omnicore/main` pushato,
  `examples/omnicore/` creato da fintech (brand OmniCore AI), `AGENTS.md` aggiunto.
- `.env` creato: `TENANT_PACKAGE_DIR=../examples/omnicore`, `KEY_ENCRYPTION_KEY` generata,
  `OPENBOT_SINGLE_USER=true`.
- Baseline: typecheck OK, lint OK, `bun test` 4948 pass / 139 fail / 57 skip —
  failure ambientali (no DB, no chiavi modello) + mismatch upstream
  `@copilotkit/react-core@1.77.0` (`applyHeadersToAgent` mancante). Da re-verificare dopo l'avvio.
- Non ancora eseguiti: login CopilotKit (CLI in attesa di sign-in browser),
  setup Intelligence, `OPENAI_API_KEY`, avvio.

## Fase 0 - Fondamenta del repository

1. Rinominare `origin` in `upstream`, creare un nuovo `origin` privato per OmniCore AI.
2. Creare il branch `omnicore/main`; `main` resta specchio di upstream.
3. Creare `examples/omnicore/` copiando `examples/fintech/` come tenant package (agents, channels, model, skills, brand).
4. Aggiungere `AGENTS.md` con comandi di build/test/lint (vedi Fase 1).
5. Verificare che `.env` sia in `.gitignore`.

DoD: repo con remote upstream/origin, tenant package `omnicore` selezionabile.

## Fase 1 - Bootstrap della piattaforma "as is"

1. Installare Bun 1.3.14 (`curl -fsSL https://bun.sh/install | bash -s "bun-v1.3.14"`).
2. `cp .env.example .env`, `bun install`.
3. `npx copilotkit@latest login` e `project select`.
4. `bun scripts/setup-learning.ts` (oppure Intelligence self-hosted, vedi README "Intelligence on your own machine").
5. Generare `KEY_ENCRYPTION_KEY` (`openssl rand -base64 32`) e impostare la chiave del modello (OpenAI/Anthropic/Google).
6. `bash scripts/start.sh`, aprire http://localhost:3010.
7. Eseguire `bun run typecheck`, `bun run lint`, `bun test` per baseline.

DoD: app raggiungibile, conversazione con "General Assistant" funzionante, baseline test registrata.
Nota: `OPENBOT_SINGLE_USER=true` solo in locale.

## Fase 2 - Agenti individuali e primo connettore (Sheets)

1. Configurare `COMPOSIO_API_KEY` in `.env`.
2. Creare l'agente **Sheets Operator** (da `/bots` oppure in `examples/omnicore/agents/`).
3. Collegare Google Sheets via Composio, selezionare solo le azioni necessarie.
4. Assegnare il connettore solo a Sheets Operator.
5. Policy: lettura consentita, scrittura con approvazione.
6. Test: leggere un intervallo di un foglio di prova, poi appendere una riga (con approvazione).
7. Verificare le righe di audit, anche per un'azione rifiutata.

DoD: lettura e scrittura approvata riuscite su foglio di test; scrittura non approvata bloccata e loggata.
Nota: il connettore Google Drive nativo e' sola lettura, non sostituisce Sheets.

## Fase 3 - Linear

1. Collegare Linear (toolkit Composio o MCP dedicato).
2. Creare l'agente **Linear Operator** con permessi minimi.
3. Test: cercare issue, poi creare una issue di test con approvazione.
4. Verificare isolamento: Sheets Operator non vede Linear e viceversa.

DoD: matrice agente x strumento verificata nei log.

## Fase 4 - Entita' Team (sviluppo principale)

Prima di scrivere codice: leggere `server/src/agents/`, `server/src/team-bots/`, `server/src/db/schema/`
e `docs/coworkers.md`. Attenzione: esiste gia' "Team Bots" (pubblicazione di un Bot a persone/gruppi),
che NON e' il nostro Team; scegliere un nome distinto nel codice (es. `agent_teams`).

1. Schema DB: `agent_teams` (id, nome, descrizione, coordinatore opzionale), `agent_team_members` (team, agente, ruolo) + migrazione.
2. API server in `server/src/teams/` (CRUD, validazione, permessi admin).
3. UI in `app/src/routes/` seguendo le skill `openbot-data-access` (queryOptions/mutationOptions) e `openbot-screen-layout` (PageShell, PageSection).
4. Team iniziali: Development, Operations, Research (via seed nel tenant package).
5. Config per agente: team, collaborazioni consentite, regole operative.
6. Test unitari e di integrazione; audit delle modifiche ai team.

DoD: si crea un team da UI, si assegnano agenti, la config persiste e produce righe di audit.

## Fase 5 - Collaborazione e coordinatore opzionale

1. Studiare il meccanismo di handoff esistente (`docs/architecture.md`, `docs/configuration.md`): permessi espliciti, limiti di profondita' e numero.
2. Collegare le "collaborazioni consentite" del Team alle autorizzazioni di handoff.
3. Aggiungere un agente Coordinatore opzionale per team, con delega solo ai membri autorizzati.
4. Approvazione umana prima di operazioni sensibili.
5. Caso d'uso reale: team Development (analisi task, modifica codice, verifica test, raccolta risultati).

DoD: un task multi-agente completato end-to-end, con handoff tracciati e limiti rispettati.
Si parte solo se un caso d'uso lo richiede (evitare chiamate a modelli inutili).

## Fase 6 - Estendibilita' e motori diversi

1. Registrare agenti AG-UI esterni (LangGraph, Mastra, Pydantic AI: esempi in `examples/` e `agent-*`).
2. Documentare come aggiungere un connettore (Composio, MCP custom, API controllata).
3. Template di agente/team in `examples/omnicore/catalog/`.

DoD: due agenti su framework diversi nello stesso team.

## Fase 7 - Hardening e produzione

1. Disattivare `OPENBOT_SINGLE_USER`, configurare sign-in (sezione "Sign in" del README).
2. Segreti: `MANAGED_AGENT_TOKEN`, `AGENT_TOOL_TOKEN`, `WORKER_SHARED_SECRET` non di default.
3. Export audit (OTLP), retention.
4. Deploy (`docs/deployment.md`, chart Helm in `charts/`), backup PostgreSQL.
5. CI: lint, typecheck, test; processo di merge periodico da upstream.

DoD: istanza accessibile a piu' utenti con autenticazione, audit esportato, procedura di backup provata.

## Rischi

- Alpha: API e schema possono cambiare; mantenere le modifiche isolate.
- Intelligence e' un servizio CopilotKit (free plan o self-hosted): dipendenza esterna.
- Composio e' un broker terzo: valutare dati e costi.
- Il nome "Team Bots" gia' presente puo' generare confusione.
