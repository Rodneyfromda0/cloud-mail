# XPass HQ Development Handoff

**Status date:** 2026-02-19
**Repository:** `https://github.com/Rodneyfromda0/cloud-mail`
**Production URL:** `https://mail.getxpass.net`

This document is the durable continuation point for XPass HQ development. It records the repository state verified during the handoff and identifies decisions or checks that still require human confirmation.

## 1. Verified repository state

The local repository is `C:\Users\Rodney\.copilot\repos\maillab_cloud-mail`.

- `main` is at `d115f4ccc5cb5fb9c602b3ba364acbca6d19f274` (`fix: resolve deployment error caused by missing default env variable`) and tracks `origin/main`.
- The XPass feature baseline branch `rodneyfromda0-mail-ui-migration` is at `83613352d332e90018a3df262a17d05a9b1ae05a` and tracks `pr-Rodneyfromda0-cloud-mail/rodneyfromda0-mail-ui-migration`.
- PR 001 branch `xpass/deployment-safety-pr` is at `16b32f095f872db1ccb9fabe8b11fd642b77ff7f` and is pushed to `pr-Rodneyfromda0-cloud-mail/xpass/deployment-safety-pr`.
- This handoff branch is `xpass/hq-handoff`, based on PR 001 commit `16b32f0`.
- The GitHub CLI was not authenticated during verification. PR 001's draft PR was therefore not confirmed as created; the branch creation URL remains `https://github.com/Rodneyfromda0/cloud-mail/pull/new/xpass/deployment-safety-pr`.

Do not infer that either PR 001 or this handoff branch has been merged or deployed.

## 2. Worktrees and preservation rules

### Original XPass feature worktree

Path: `C:\Users\Rodney\.copilot\repos\copilot-worktrees\maillab_cloud-mail\rodneyfromda0-mail-ui-migration`

Branch: `xpass/deployment-safety`, currently at `8361335`.

This worktree was dirty at verification and must remain preserved. It contained 19 modified tracked files, including workflow, README, Vue, Worker, configuration, and service files, plus 8 untracked files for push notifications and account delegation. Do not reset, clean, force-checkout, overwrite, or stage these changes without an explicit task covering them.

### Isolated development worktrees

- `deployment-safety-pr`: branch `xpass/deployment-safety-pr`, PR 001 commit `16b32f0`.
- `xpass-hq-handoff`: branch `xpass/hq-handoff`, this documentation-only continuation branch.

Use a new isolated worktree for subsequent feature work when the source branch is dirty or when a change needs a separate reviewable PR.

## 3. Completed development work

### PR 001: Deployment Safety Foundation

Commit: `16b32f095f872db1ccb9fabe8b11fd642b77ff7f`

Changed files:

- `.github/workflows/deploy-cloudflare.yml`
- `doc/deployment-safety.md`

The implementation:

- changed deployment to manual `workflow_dispatch`;
- requires the boolean `confirm_production` input;
- enforces `if: inputs.confirm_production == true` at the deployment job;
- requires the protected GitHub `production` environment;
- fails when `KV_NAMESPACE_ID` or `D1_DATABASE_ID` is missing;
- removed automatic KV/D1 discovery and creation;
- removed deployment-time D1 SQL execution;
- removed the `/api/init/<secret>` deployment call; and
- documents release validation, migration separation, backup expectations, rollback, and human approvals.

No production deployment, migration, or Cloudflare resource operation was executed during this work.

## 4. Existing functionality to preserve

The current product is XPass Mail, being expanded into XPass HQ. Existing Mail behavior and XPass customizations are product dependencies, not disposable prototype code. Preserve and regression-test at minimum:

- Vue 3 Mail UI, navigation, localization, and account flows;
- Hono-based Worker API and existing authentication/session behavior until it is safely hardened;
- mail listing, message detail, compose, send, receive/refresh, drafts, starred mail, folders, and templates where enabled;
- attachments and their intended access controls;
- account switching and administrator-only areas;
- custom signatures and account-signature application in compose;
- XPass branding and modernized layout;
- push notification work and account delegation work already present in the dirty feature worktree; and
- existing D1, KV, R2, Email, AI, Assets, custom-domain, and cron bindings.

Any refactor must preserve data and behavior or include a reviewed migration and rollback plan.

## 5. Current architecture

Verified repository structure:

- `mail-vue/`: Vue 3 frontend using Vite, Vue Router, Pinia, Element Plus, Axios, i18n, and related UI libraries.
- `mail-worker/`: Cloudflare Worker application using Hono, Wrangler, Drizzle ORM, D1, KV, R2, Email, Resend, and related Worker dependencies.
- `.github/workflows/deploy-cloudflare.yml`: controlled Cloudflare deployment workflow.
- `doc/deployment-safety.md`: deployment and migration safety procedure.
- `mail-worker/wrangler-action.toml`: generated deployment configuration populated by the workflow from GitHub configuration.

The intended HQ direction is a shared identity, authorization, organization, audit, and data model across Mail, CRM, Finance, Projects, Documents, People/HR, Support, Analytics, Approvals, Notifications, Search, Integrations, and AI. Do not introduce module-specific permission shortcuts that bypass backend authorization.

## 6. Security backlog and priorities

These are audit findings requiring verification and remediation; they are not claims of exploitation:

1. Remove or redesign initialization through a JWT secret in a URL.
2. Narrow broad authentication exclusions and verify every public route.
3. Protect R2 objects through authorization rather than direct predictable paths.
4. Replace or reduce sensitive token storage in browser `localStorage` with a safer session design.
5. Replace Mail path-prefix permissions with backend-enforced organization, role, and resource authorization.
6. Remove email-based administrator bypasses and implement explicit roles/claims.
7. Add meaningful automated tests for authentication, authorization, account isolation, mail flows, and security-sensitive routes.
8. Replace the unsafe `mail-worker` `test` script, which currently invokes `wrangler deploy`, with a genuinely non-deploying test command before using it in automation.
9. Finish and review account delegation permissions, lifecycle, and auditability.
10. Implement and rehearse documented backup and restoration procedures.

Recommended order: establish identity/organization/role primitives and authorization tests first; then harden sessions, initialization, R2 access, delegation, and backups; then build shared HQ modules on those primitives.

## 7. Deployment after PR 001

Normal application deployment is no longer push-triggered. The workflow requires:

1. A reviewed commit and approved pull request.
2. Human verification of the Cloudflare account, Worker, D1, KV, R2, Email, Assets, AI, domain, and cron targets.
3. Existing, human-verified `D1_DATABASE_ID` and `KV_NAMESPACE_ID` configuration.
4. Manual dispatch with `confirm_production=true`.
5. Approval of the protected GitHub `production` environment.
6. Post-release validation using controlled test accounts and mailboxes.

The normal deploy workflow does not run D1 SQL, create resources, or call `/api/init/<secret>`. Migrations remain separate reviewed operations with backup, restore, impact, and rollback controls. See `doc/deployment-safety.md` for the operational checklist.

The workflow still contains the intentional `pnpm wrangler deploy -c wrangler-action.toml` deployment command. Never run it, any `wrangler deploy`, or a Cloudflare resource command without explicit authorization.

## 8. Testing and validation limitations

- The repository does not currently provide a root package manifest or a single verified cross-module test command.
- `mail-vue` exposes build/dev scripts but no test script.
- `mail-worker` has Vitest dependencies, but its `test` script is unsafe because it invokes deployment; do not use that script as a test.
- YAML parser/actionlint tooling was unavailable during PR 001 validation. Workflow validation used static invariant checks and `git diff --check`, not a full YAML parser.
- GitHub PR merge state and GitHub Actions results were not locally confirmed because GitHub CLI authentication was unavailable.
- Production behavior, resource identity, secrets, environment protection, and deployed version require human verification in GitHub/Cloudflare.

Use non-deploying tests only. Add focused tests before changing authentication or shared data contracts.

## 9. Immediate development sequence

1. Finish human verification and review of PR 001; create the draft PR against `rodneyfromda0-mail-ui-migration` if it is not already open. Do not merge or deploy from this handoff.
2. Create a clean security-hardening branch from the accepted XPass baseline, not from the dirty worktree.
3. Inventory all Worker routes and authentication exclusions, then add backend authorization tests before changing access behavior.
4. Define the shared HQ organization, user, role, permission, audit-event, and tenant/resource ownership model. Keep the first schema change additive and separately reviewed.
5. Harden session/token handling, initialization, R2 access, administrator authorization, and account delegation in small reviewable changes.
6. Add a safe non-deploying Worker test command and a frontend build/test baseline.
7. Build the HQ shell and module navigation only after identity and permission contracts are stable.
8. Implement CRM before Finance, reusing shared identities, organizations, contacts, activities, audit events, and permissions.
9. Design Finance with immutable transaction records, explicit reconciliation states, idempotency, audit trails, and financial-data owner review before implementation.

## 10. Decisions and unresolved dependencies

Decisions already made:

- Preserve the dirty XPass worktree; never use destructive Git cleanup to isolate work.
- Use isolated worktrees and focused branches for reviewable changes.
- Keep migrations separate from normal deployment.
- Enforce permissions on the backend and preserve existing Mail behavior.
- Never deploy or modify production without explicit authorization.

Still unresolved:

- Whether PR 001 has been manually created, reviewed, approved, or merged.
- Whether the protected GitHub `production` environment and required reviewers are configured correctly.
- Which Cloudflare account, Worker, D1, KV, R2, Email, Assets, AI, domain, and cron resources are the approved production targets.
- The accepted session architecture and organization/tenant model for HQ.
- The migration and backup tooling that will be approved for shared HQ data.
- The test framework and CI policy for frontend, Worker, authorization, and integration tests.
- Product ownership and approval requirements for Finance and other sensitive modules.

Resolve these dependencies through review and documented decisions; do not infer them from local configuration or deploy to discover them.
