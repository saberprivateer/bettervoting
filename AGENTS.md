# BetterVoting Assistant Rules

This repository guidelines file provides project context, constraints, and instructions for AI agents working in this repository, imported and adapted from `CLAUDE.md`, `CONTEXT.md`, and `CONTRIBUTING.md`.

---

## 1. Project Context & Domain Vocabulary

BetterVoting is an online election and polling platform built by the Equal Vote Coalition, supporting multiple voting methods (STAR, IRV, Approval, Ranked Robin, and others).

### Domain Language & Glossary
- **Support Actions**: The canonical trio of ways a visitor can support the project — **Volunteer**, **Donate**, **Merch** — surfaced together in both the nav's "Support Us" dropdown and the landing page's support stripe.
- `/volunteer` redirects to the codebase contribution guide; therefore, **Volunteer** already covers code and documentation contributions — it is not a separate action.
- **Avoid**: "Contribute" as a Support Action (it points to the same destination as Volunteer and is not a distinct action).
- Consult [CONTEXT.md](file:///c:/Users/Chill/Documents/GitHub/bettervoting/CONTEXT.md) and any relevant ADRs in [dev-docs/adr/](file:///c:/Users/Chill/Documents/GitHub/bettervoting/dev-docs/adr) when working on architectural and domain areas.
- If changes or proposals contradict an existing ADR, explicitly surface the conflict rather than silently overriding it.

### Contributing Front Door
- New contributors, volunteer questions, and general inquiries should be directed to https://bettervoting.com/volunteer.

---

## 2. Git & PR Protocols

### Remote & Push Safety
- Always use the **full remote URL** (not a remote name) when running `git push` to avoid ambiguity across multiple remotes.
- **Never push** to any URL matching `github.com/Equal-Vote/*` without explicit confirmation.

### PR Requirements
1. **Draft PRs**:
   - PRs opened by an agent must always be created as drafts (`gh pr create --draft`) so the user can review them before they are visible to the dev lead.
   - Only mark a PR ready for review (`gh pr ready`) when the user explicitly instructs it in that moment — never automatically, and never as a default follow-up.
   - After opening a draft PR, notify the user that it is a draft ready for their review before going to their dev lead.
2. **Human Summary on PRs**:
   - Every PR opened by an agent must start with a short, human-written note above the AI-generated summary, under a `## Human Summary` heading.
   - Before running `gh pr create`, stop and ask the user:
     > *"Before I open this PR — in a sentence or two: what are you trying to accomplish, and why? I'll put this at the top of the PR description, above my own summary, so reviewers get your context alongside the AI-generated details."*
   - Require both the *what* and the *why*. If only one is provided, ask a follow-up for the other.
   - Put the answer verbatim under `## Human Summary` at the top of the PR body, followed by `## Summary` and `## Test plan`.
   - If no human is available to answer, **do not open the PR** — stop and surface that you are blocked.

---

## 3. Architecture & Monorepo Structure

TypeScript monorepo with three packages: `packages/shared/`, `packages/backend/`, and `packages/frontend/`, plus E2E testing in `testing/`.

### Shared (`packages/shared/`)
- TypeScript domain types (`Ballot`, `Candidate`, `Election`, `Race`, `ElectionRoll`, etc.), shared utilities, and JSON schemas.
- Imported by backend and frontend via `@equal-vote/star-vote-shared`.

### Backend (`packages/backend/`)
- Express app on port 5000 (or `BACKEND_PORT`). Entry point: `src/index.ts` (`makeApp()`, `setupSockets()`).
- Database: Kysely query builder over PostgreSQL (`src/Models/`, `src/Migrations/`).
- Routes & Controllers: `/API/Elections`, `/API/Ballots`, `/API/Roll`, `/API/Token`, `/API/Docs`, `/API/SendGridWebhook`.
- Services: Azure Blob Storage, SendGrid email, EventQueue (`pg-boss`), logging, account.
- Tabulators (`src/Tabulators/`): STAR, IRV, Approval, Ranked Robin, Plurality, STV.
- Auth: Keycloak JWT integration (`src/auth/`).

### Frontend (`packages/frontend/`)
- React 17 app with Material-UI, built with RSBuild. Entry point: `src/index.tsx` -> `App.tsx`.
- Routing: React Router v6 (`/`, `/new_election`, `/election/:id` or `/:id`, `/manage`, `/browse`, `/sandbox`).
- State: React Context API (`FeatureFlagContext`, `ThemeContext`, `AuthSessionContext`, `ConfirmDialogContext`, `SnackbarContext`, `ReturnToClassicContext`).
- Real-time: `socket.io-client`.
- i18n: `i18next` (`src/i18n/`).

### E2E Testing (`testing/`)
- Playwright tests. Setup project `auth.setup.ts` handles authentication.
- URL conventions:
  - Admin subpages use `/${id}/admin/<page>` (e.g. `/${id}/admin/voters`, `/${id}/admin/build_ballot`, `/${id}/admin/settings`, `/${id}/admin/publish`).
  - Do not omit `/admin/` in admin routes.
  - Admin sidebar links change based on election state: draft ballot link is "Voting Page"; once finalized/open it becomes "Live Ballot". Results link is always "Live Results".
  - Avoid `waitForURL(**/${id}/)` with trailing slashes since React Router navigates without trailing slash.
- UI quirks:
  - In MUI 9, switches use `role="switch"` and `FormControlLabel` with `labelPlacement="start"`. Match by label text or substring for i18n `!tip()` syntax.
  - First-time clicking "Add Voters" prompts a confirmation dialog; click "Submit" to dismiss before interacting with the voter form.

---

## 4. Key Commands

```bash
# Frontend dev
npm run dev -w @equal-vote/star-vote-frontend

# Backend dev
npm run dev -w @equal-vote/star-vote-backend

# Full docker stack
docker compose up

# Build
npm run build -ws
npm run build -w @equal-vote/star-vote-backend
npm run build -w @equal-vote/star-vote-frontend
npm run build -w @equal-vote/star-vote-shared

# Backend tests
npm test -w @equal-vote/star-vote-backend
npx jest --testPathPattern=<filename> -w @equal-vote/star-vote-backend

# E2E Playwright tests
cd testing && npx playwright test --reporter=list
cd testing && npx playwright test tests/<filename>.spec.ts --reporter=list

# Database migrations
npm run migrate:latest -w @equal-vote/star-vote-backend
npm run migrate:up -w @equal-vote/star-vote-backend
npm run migrate:down -w @equal-vote/star-vote-backend

# Linting
npm run lint -w @equal-vote/star-vote-frontend
```

---

## 5. Dependency & Issue Tracker Notes

- Root `package.json` overrides for `qs` exist because Netlify's mirror lagged behind `qs@6.15.2`. Safe to remove once Netlify deploy without it is verified.
- Issues live in GitHub Issues on `Equal-Vote/bettervoting`.
- Triage labels: only `ready-for-agent` is tracked, using the label `sandcastle`.
