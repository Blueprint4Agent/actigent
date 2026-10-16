# Actigent — Phase 1

Actigent collects activities across services. Phase 1 establishes the B4A foundation only; collectors are not implemented.

## Repository and baseline

- Origin: https://github.com/Blueprint4Agent/actigent (public GitHub fork).
- Upstream: https://github.com/Blueprint4Agent/B4FastAPI.git.
- Initial upstream main: `b354de05ccdd5567ff249be2a13142dc029bc395`.
- Pinned B4React: `3caa104d49f5e09138a53c4b9a0960839e2927e9` in `src/frontend`.
- Local clone: `/Users/choi-hyk/blueprint4agent/actigent`.
- Preserve upstream license and history. The original local B4FastAPI checkout is not modified.

## Reproduce development setup

```sh
git clone --recurse-submodules https://github.com/Blueprint4Agent/actigent.git
cd actigent
git remote add upstream https://github.com/Blueprint4Agent/B4FastAPI.git
git config remote.upstream.pushurl DISABLED
git config remote.pushDefault origin
git fetch upstream
make init
make install
```

Phase 1 preserves the original B4A UI identity and test fixtures. Service rebranding is deferred: the inherited browser suite contains literal B4A assertions. No public project manifest is committed yet. Use the existing project initializer in a later branding task together with an independently maintained frontend fork if needed. Env files and secrets are ignored. Generate a unique local `SECRET_KEY` (for example with Python `secrets.token_urlsafe(48)`); never copy B4A credentials or databases.

For this clone, backend `.env` uses `APP_NAME=Actigent API`, `APP_PORT=8100`, `DB_NAME=actigent.db`, `APP_BASE_URL=http://localhost:5180`, and CORS origins for localhost/127.0.0.1 on 5180 (plus the existing Tauri origin). Frontend `.env` uses `VITE_API_BASE_URL=http://127.0.0.1:8100`. Set these ignored values after initialization on another machine. SQLite and in-memory Redis isolate development from B4A. Email, OAuth, Stripe and external telemetry remain disabled; this profile does not validate external Redis/Celery, payment, SMTP or cloud storage integrations.

Run in separate terminals:

```sh
make backend-dev BACKEND_HOST=127.0.0.1 BACKEND_PORT=8100
make frontend-dev FRONTEND_HOST=127.0.0.1 FRONTEND_PORT=5180
```

Open http://127.0.0.1:5180 and http://127.0.0.1:8100/docs. Restart frontend after changing env. For the complete initial baseline:

```sh
make verify-plan
VERIFY_FULL=1 make verify
```

The frontend submodule remains a pinned upstream dependency. Do not push Actigent-specific changes into B4React. If future product UI requires source changes, establish a separately owned child fork and its upstream policy before changing the parent gitlink.

## Upstream synchronization policy

Review upstream weekly and promptly for security fixes; this is a manual policy, not a scheduled workflow. Merge reviewed snapshots into a dedicated branch, preserving shared ancestry and Actigent commits. Never reset Actigent main to upstream or force-push it. Always target Actigent explicitly in `gh` commands because fork tooling can otherwise select the parent.

```sh
git switch main
git pull --ff-only origin main
git fetch upstream --tags
git switch -c chore/upstream-sync-YYYYMMDD
git log --oneline HEAD..upstream/main
git diff HEAD...upstream/main --stat
# Review migrations, dependencies, env examples, contracts, hooks and child pin.
git merge --no-ff upstream/main
git submodule update --init --recursive
make install
# Add a matching numbered worklog for any authored resolution/documentation commit.
VERIFY_BASE=$(git merge-base origin/main HEAD) make verify-plan
VERIFY_FULL=1 VERIFY_BASE=$(git merge-base origin/main HEAD) make verify
# Stage, validate governance and commit any authored changes before pushing.
git push -u origin HEAD
```

Use a ready PR against `Blueprint4Agent/actigent:main`, with Summary, Scope, Reason, Verification, Documentation and Risk / Impact. Run `PR_NUMBER=<number> make git-governance-pr-check`, then merge using `gh pr merge <number> --repo Blueprint4Agent/actigent --merge`. Main requires a PR, resolved conversations and prohibits deletion/force pushes. Manual Actions and local hooks retain the inherited model; no automatic CI is claimed. Existing upstream commit/worklog conventions remain subject to the inherited branch-range harness. If a future upstream commit fails that harness, resolve policy compatibility transparently in the sync task; do not bypass hooks or rewrite upstream history.

Resolve conflicts individually, keeping Actigent local configuration and domain behavior while incorporating applicable framework fixes. Validate API contracts and the exact updated child pin. Back up non-disposable databases before migrations; test upgrades on a copy. Abort an unresolved merge with `git merge --abort`. Roll back a merged synchronization using a reviewed revert PR (`git revert -m 1 <sync-merge-sha>`), assessing database compatibility separately.

## Future collection architecture

Adapters will translate source-specific APIs/payloads into a common activity contract. Strategies will encapsulate polling/webhook, cursor and retry behavior. A factory/registry will select adapters explicitly by configured source. Future design must specify identifiers, cursors, idempotency, credentials, rate limits and cancellation before implementation. Keep orchestration in services, HTTP transport in routers, persistence in models/repositories, and durable jobs in the existing task loop; publish domain events only after successful state changes. No new interface, scheduler, collector, API or migration is introduced in Phase 1.
