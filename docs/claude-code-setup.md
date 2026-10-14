# Claude Code and Tooling Setup

Written 2026-10-14. Prioritized by what the repository actually needs today, which is little, because there is no code to tool.

The guiding principle: each added integration costs permissions, supply-chain surface and maintenance. A repository with no source files earns almost none of them. Add tools when the thing they operate on exists.

## Essential now

| Item | Scope | Why |
| :--- | :--- | :--- |
| `.gitignore` | Project | Added 2026-10-14. The repository had none through four commits, directly upstream of a phase that introduces five service credentials and streams caller audio. |
| `CLAUDE.md` | Project | Added 2026-10-14. Without it, every session re-derives that this repository is pre-code by reading 823 lines of planning prose. |
| GitHub MCP | Session | Already connected and in use. Covers repository, issue, pull request and CI operations. No action needed. |

## Add when Phase 1 code exists

| Item | Scope | Why |
| :--- | :--- | :--- |
| Python toolchain: `uv` or `pip`, `ruff`, `pytest`, `mypy` | Project | Nothing to lint, test or type-check today. |
| Pre-commit or `PreToolUse` hook running `ruff` and `pytest` | Project | A hook earns its keep once there is a check it can run. Deterministic validation is more reliable than instructions asking a model to remember to test. Not before. |
| `docker-compose.yml` for Redis and PostgreSQL | Project | Part of Phase 1 itself. |

## Useful, with caveats

A documentation-lookup MCP server is the one optional integration with a clear case. Deepgram, Cartesia, LiveKit, Pipecat and Twilio all move faster than model training data, and the stale model names in this repository's own planning documents are exactly the failure mode such a server prevents.

Two caveats before installing one. First, `WebSearch` and `WebFetch` already cover this need at lower fidelity and zero setup, so the gain is incremental rather than enabling. Second, provenance matters: the official MCP Registry at `registry.modelcontextprotocol.io` remains a preview service, and a listing there implies nothing about a server's security, maintenance, licensing or tool quality (see <https://roxyapi.com/blogs/mcp-registries-where-to-list-your-server>, checked 2026-10-14). Verify the publishing organization and the package's provenance directly before installing, rather than trusting a directory entry.

## Deferred, with reasons

These are listed so the reasoning is on record and the question does not get re-litigated.

PostgreSQL MCP. No schema exists. When one does, grant read-only access against a local development database only. It must never reach a store containing participant data — a query tool with access to caller risk events is a data-exposure path with no corresponding benefit.

Playwright MCP or browser automation. No user interface exists, and the product is deliberately phone-only: zero-friction access with no app and no login is the first of the five core pillars. Should a clinician review console ever be built, Chromium and Playwright are already available in this environment with `PLAYWRIGHT_BROWSERS_PATH` preconfigured.

Twilio MCP. Telephony operations belong in application code and the Twilio CLI, under test, not in an agent session. A server able to place calls on this project's account is a high blast radius for near-zero auditability — the project's own safety rule is that no development session should be capable of dialling a live crisis line, and granting an agent outbound-call tooling contradicts it directly.

Cloud provider, monitoring, issue tracking, design and API testing MCPs. Nothing is deployed, nothing is instrumented, and there is no design surface. Each would grant permissions against something that does not exist.

## Claude Code features

A root `CLAUDE.md` is justified and now present. Nothing else is.

Skills encode repeated workflows; this project has no repeated workflow yet. Subagents are useful for independent audits across a large codebase; there are five Markdown files. Hooks need a check to run, which arrives with `ruff` and `pytest`.

The one thing worth revisiting early is a `SessionStart` hook once a dependency manifest exists, so cloud sessions can run tests without re-deriving the environment each time. That becomes relevant at the same moment Phase 1 lands, not before.

## Permissions posture

For as long as this repository holds documentation only, read-only access is sufficient everywhere except the designated development branch. GitHub access stays scoped to `Hemeema-gunthoti/HelpHuman`. No credential belongs in a project settings file.

The boundary worth setting now, before any code exists to tempt otherwise: nothing in this repository should ever be able to place an outbound call, dial a crisis line, or write to a store holding participant data from a developer or agent session. Encode it in configuration — see `APP_ENV` and `TWILIO_TEST_TRANSFER_NUMBER` in `docs/development-setup.md` — rather than relying on anyone remembering it.
