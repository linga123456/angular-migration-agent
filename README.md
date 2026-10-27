# Angular Migration Agent

A repository-scoped migration control plane for the specific migration workflow: Angular 11 + AG Grid 24 Short Coverage → existing Angular 21 target remote, using the manually migrated Order Management module as the reference and QA manual regression cases as verification requirements.

This repository contains the agent application only. It deliberately does not include demo/sample Angular projects or fabricated source code for Short Coverage or Order Management.

## Technology
- Java 21, Spring Boot 4.1.x, Spring AI 2.0.x
- LangGraph4j + Spring AI integration for agent/tool execution
- Angular 21 management UI
- Angular CLI MCP server and AG Grid `ag-mcp` MCP server
- Official Angular Agent Skills (`angular-developer`)
- Playwright Test for generated UI regression tests
- Local file-backed state for single-instance development (no database required)

## Security posture
This is a production-oriented foundation, not a certified production deployment. Before connecting confidential repositories, complete the deployment/security checklist in `docs/PRODUCTION_READINESS.md`. The agent must run against explicitly configured workspace roots, use an approved model endpoint, avoid secrets, require approval before applying proposed file changes, and run only allow-listed verification commands. Never expose the service directly to an untrusted network.

## Prerequisites
- Java 21 and Maven 3.9+
- Node.js 22 LTS and npm
- BNP-approved internal Qwen gateway details and an individually issued developer credential
- Local checkout paths for Short Coverage source, Angular 21 target remote, and migrated Order Management reference
- QA's manual regression test cases (CSV/XLSX/Markdown/TXT)

## Setup
1. Copy `.env.example` to `.env` for local-only configuration. Set `QWEN_BASE_URL`, `QWEN_MODEL`, and `QWEN_API_KEY` from the approved BNP developer onboarding instructions. The values shown are placeholders, not real endpoints or model IDs. Do not commit `.env`; prefer the corporate secret manager or injected environment variables for real credentials. Spring AI uses its OpenAI-compatible client only if the internal Qwen gateway supports that protocol. If BNP requires custom headers, a different API protocol, or a token exchange, implement the adapter at the marked TODO in `docs/IMPLEMENTATION_PSEUDOCODE.md`.
2. Set `MIGRATION_WORKSPACE_ROOT` to a dedicated parent directory containing the allowed source/target/reference checkouts. The backend rejects paths outside this root.
3. Start the API: `cd backend && mvn spring-boot:run`. State is stored under `MIGRATION_STATE_ROOT` (default `${user.home}/.angular-migration-agent/state`) and the directory should be outside the Git checkout and writable only by the service account.
4. Start the UI: `cd frontend && npm install && npm start`.
5. Install official Angular AI skills in the repository using `npx skills add https://github.com/angular/skills`. Review the installed skill and pin the commit/version according to your supply-chain policy.
6. Ensure the Angular CLI and AG Grid MCP servers are installed/approved in the agent runtime. The app includes MCP configurations; availability must be checked at runtime.

## Workflow
1. Register the source, target, reference and QA cases using the UI/API.
2. Run **Analyse**. The agent inventories source and target versions, routes, components, services, styling, shared libraries, AG Grid usage and existing tests. It compares with the reference migration and extracts QA cases.
3. Review the inventory and proposed plan. No source writes are allowed during analysis.
4. Approve the plan. The implementation agent may propose file changes, which are stored as change proposals and require separate explicit approval before application.
5. Verify the target build using configured allow-listed commands.
6. Generate Playwright tests from QA manual regression cases and observed UI selectors/routes; review them before execution.
7. Run the Playwright suite against the target remote integrated with the shell. Results are linked to the run and QA case IDs.
8. Review the diff, failed checks, coverage gaps and manual checks before merging.

## API overview
- `POST /api/migrations` — create a run from configured relative workspace paths
- `GET /api/migrations/{id}` — get run status and artifacts summary
- `POST /api/migrations/{id}/analyse` — run read-only discovery/planning
- `POST /api/migrations/{id}/approve-plan` — approve the migration plan
- `GET /api/migrations/{id}/changes` — list proposed changes
- `POST /api/migrations/{id}/changes/{changeId}/approve` — approve one change for application
- `POST /api/migrations/{id}/apply-approved` — apply approved changes only
- `POST /api/migrations/{id}/verify` — run configured build/tests
- `POST /api/migrations/{id}/qa-cases` — upload QA regression cases
- `POST /api/migrations/{id}/playwright/generate` — generate Playwright tests from accepted QA cases
- `POST /api/migrations/{id}/playwright/run` — run Playwright against configured target URL

Some endpoints are deliberately guarded/configuration-dependent; review controller implementation and readiness docs before exposing them beyond local development.

## MCP
- Angular CLI MCP: `npx -y @angular/cli mcp --read-only` for analysis; enable write-capable tools only in a sandbox after review.
- AG Grid MCP: `npx -y ag-mcp` as a placeholder command; verify the current official package name, transport and any authentication options before enabling it in the BNP environment.
- Official Angular skills: `npx skills add https://github.com/angular/skills`. Review the official `angular-developer` skill, pin the approved revision, and make its instructions available to the agent runtime. The Spring AI backend does not automatically execute IDE/CLI skills merely because they are present in the repository; connect the skill content through a reviewed prompt/context loader or the coding-agent runtime.

MCP is an external tool boundary. Allow-list server commands, review package provenance, pin versions and isolate execution. Do not assume the servers are available simply because they are configured.

## Important limitations
The actual migration cannot be run until the real Short Coverage source repository (Angular 11 / AG Grid 24), Angular 21 target remote, manually migrated Order Management reference, and QA manual regression cases are available and workspace paths are configured. This is a production-oriented foundation, not a production-certified application. State is persisted to a single JSON file using atomic replacement for a single application instance; it is not a distributed database, multi-instance lock service or tamper-proof audit store. Back up and protect the state directory. Before multi-instance production use, integrate an organisation-approved shared durable store or durable LangGraph4j checkpoint backend. The LangGraph4j agent-executor integration is present, but the full durable multi-stage graph, enterprise identity/roles, runtime sandbox, real Qwen gateway compatibility, official MCP runtime verification, Playwright generation review and end-to-end acceptance still require implementation and validation. See `docs/IMPLEMENTATION_PSEUDOCODE.md` for exact TODOs. Never treat an unexecuted build/test as passing.
# angular-migration-agent
