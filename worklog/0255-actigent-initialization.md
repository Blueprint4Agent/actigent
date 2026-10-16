# Commit Title

chore: initialize Actigent development environment

# Changed File Scope

README.md, notes/ko/README.md, notes/actigent.md, notes/ko/actigent.md, worklog/0255-actigent-initialization.md.

# Reason

Create an independently configured Actigent fork while preserving B4A history and pinned frontend.

# Design

Preserve baseline B4A UI and tests; defer public identity manifest after discovering hardcoded brand expectations. Configure independent ignored local env. Keep local secrets, SQLite and build output ignored. Origin is Actigent; upstream is read-only B4FastAPI. Document reviewed merge-based upstream updates. No collection implementation.

# Verification Plan

Run make init/install, make verify-plan and VERIFY_FULL=1 make verify; smoke-test isolated local backend and frontend. Verify fork ancestry, remotes and unchanged original checkout.

# Impact

New fork and local development environment only. No production deployment or original repository changes.

# Loop Alignment

Backend request/event/task and frontend API/realtime/connectivity/UI loops are unchanged: no runtime implementation is introduced. Future adapters must follow these existing loops.

# Verification

- GitHub fork created in Blueprint4Agent; upstream baseline b354de05ccdd5567ff249be2a13142dc029bc395 (394 commits), frontend pin 3caa104d49f5e09138a53c4b9a0960839e2927e9.
- make init/install/project-init/project-check passed. Main ruleset active: PR, resolved conversations, no deletion/force push. Origin default and upstream push disabled locally.
- Initial VERIFY_FULL=1 make verify passed backend 502 tests and contracts, then exposed an inherited frontend test assumption: name and short_name equal caused getByText to match two nodes. Set short_name=ACT, retain full name Actigent; no child source edits.
- Branded browser suite: 244 passed, 6 failed because public-discovery/runtime-mode fixtures expect literal B4A identity. Reverted the optional identity manifest and generated child branding, keeping the baseline UI unchanged. Rebranding is deferred rather than changing upstream source or weakening tests.
- Final documentation changes select docs-only; explicitly run full baseline verification for Phase 1 acceptance. VERIFY_FULL=1 make verify passed all selected stages: hooks, initializer, architecture/env/lint, 502 backend tests, 209 frontend tests, 250 UI cases, production routes/discovery builds, style studio, contract/packaging and isolated custom-brand build. Logs: .git/verification-logs/0.log through 9.log. Final documentation make verify is also required before commit.
- Live backend on loopback 8100: /health/ready reports database/redis ok, /config reports development and disabled external integrations, OpenAPI title Actigent API. Final fresh Chromium on 5180 reaches /home with baseline title Blueprint4FastAPI, B4A wordmark, no page errors; screenshot /tmp/actigent-home.png.
- Original checkout remains clean at 32cb17676771e190bc50314dee2b2d4b3bf4468d. B4React source/gitlink unchanged.
- External Redis/Celery, SMTP/OAuth/Stripe/cloud storage, native desktop installers and production deployment omitted: Phase 1 uses isolated SQLite/in-memory Redis and no external credentials. No runtime loop implementation changed.
