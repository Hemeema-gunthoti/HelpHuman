# Development Setup

Written 2026-10-14.

## Nothing in this repository is runnable

There is no application code, no dependency manifest, and no test suite. There is nothing to install and nothing to run. Every command in this document is marked unverified because none has been executed against this project — they describe the intended Phase 1 environment, not a working one.

To work on the repository as it stands, you need a text editor and git. That is all.

```bash
git clone https://github.com/Hemeema-gunthoti/HelpHuman.git
cd HelpHuman
git checkout claude/laughing-johnson-j3ut1n
```

## Intended Phase 1 prerequisites (unverified)

When the walking skeleton is built, it is expected to need:

| Requirement | Intended version | Purpose |
| :--- | :--- | :--- |
| Python | 3.11 or later | Backend core; asyncio streaming support |
| `uv` or `pip` | current | Dependency management |
| `ruff` | current | Lint and format |
| `pytest` | current | Test harness |
| `mypy` | current | Type checking |
| Redis | 7.0 or later | In-call session state |
| PostgreSQL | 15 or later, with `pgvector` | Longitudinal event store |
| openSMILE | current | Acoustic feature extraction; C++ with Python bindings |
| ngrok or equivalent tunnel | current | Exposing a local WebSocket endpoint to Twilio during development |

The tunnel requirement is easy to overlook and blocks the first useful test: Twilio Media Streams must reach your development machine over a public URL before a single call can be exercised locally.

A local Redis and PostgreSQL are most simply run via containers. No `docker-compose.yml` exists yet; writing one is part of Phase 1.

## Accounts and credentials needed

Five external services, each requiring an account before Phase 1 can be exercised end to end: Twilio (with a provisioned phone number), a speech-to-text provider, a text-to-speech provider, a model provider, and a cloud account if anything is deployed. Under the research framing, check whether each provider's terms and your IRB protocol permit participant audio to transit that service before signing up — the question is easier to answer before accounts exist than after.

## Configuration variables

Variable names only. Values never enter this repository, and no `.env` file is tracked — `.gitignore` excludes `.env` and `.env.*` while permitting a committed `.env.example` that contains names and empty values only.

| Variable | Purpose |
| :--- | :--- |
| `TWILIO_ACCOUNT_SID` | Twilio account identifier |
| `TWILIO_AUTH_TOKEN` | Twilio authentication |
| `TWILIO_PHONE_NUMBER` | Provisioned inbound number |
| `TWILIO_TEST_TRANSFER_NUMBER` | Test destination for escalation. Never a live crisis line. |
| `STT_API_KEY` | Speech-to-text provider |
| `TTS_API_KEY` | Text-to-speech provider |
| `ANTHROPIC_API_KEY` | Model provider, if Claude is used |
| `REDIS_URL` | In-call state store |
| `DATABASE_URL` | PostgreSQL connection string |
| `CALLER_HASH_SALT` | Salt for caller identifier hashing. Rotating this breaks longitudinal continuity — treat as long-lived and back it up separately from the database. |
| `APP_ENV` | Environment discriminator gating any real-telephony behaviour |
| `SAFETY_CLASSIFIER_REQUIRED` | Fail-safe switch; see degraded mode in `docs/architecture.md` |

`TWILIO_TEST_TRANSFER_NUMBER` and `APP_ENV` exist to make the project's central safety rule enforceable in code rather than by convention: no development or test configuration should be capable of dialling a live crisis line. Build that gate before building the escalation path, not after.

## Troubleshooting

There is nothing yet to troubleshoot. This section will be populated once Phase 1 produces commands that can fail. Two problems are predictable enough to anticipate:

Twilio Media Streams requires a publicly reachable `wss://` endpoint. A local server behind NAT will connect, appear healthy, and receive no audio. Verify the tunnel before debugging the pipeline.

Round-trip latency is cumulative across five hops and the 700 ms budget is tight. Measure each stage separately from the first day rather than measuring only the total, or the stage responsible for a regression will be unidentifiable.

## Verifying this document

When Phase 1 lands, each command here must be executed and the unverified marking removed only for those that actually ran. Do not remove the marking on a command that merely looks correct.
