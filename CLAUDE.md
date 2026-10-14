# CLAUDE.md — HelpHuman

HelpHuman is a planned inbound telephony voice AI that offers non-judgmental listening to callers experiencing loneliness or early depression, with clinical risk monitoring and warm escalation to a crisis line.

## Status: planning stage — no application code exists

This repository contains design documents only. There is no source code, no dependency manifest, no test suite, and no CI. Do not report build, run, or test results for this project: there are no such commands yet, and none have ever been executed here.

Framing: research/academic project. IRB review, not HIPAA alone, is the governing frame for any work involving human participants.

## Repository map

| Path | Contents |
| :--- | :--- |
| `README.md` | Product pitch and five core pillars. |
| `mental_health_voice_agent_blueprint.md` | Product blueprint: intent engine, event schema, call flow, C-SSRS escalation ladder, dropped-call protocol, roadmap. |
| `comprehensive_implementation_plan.md` | Engineering specification: architecture, stack, models, datasets, regulatory analysis, alignment protocols. |
| `foundations_and_non_judgmental_interaction.md` | Psychological and clinical foundations; six interaction principles. |
| `deep_dive_psychology_and_differentiation.md` | Loneliness neurobiology, polyvagal theory, competitive differentiators. |
| `docs/` | Onboarding layer — see below. Written 2026-10-14. |

Read `docs/` before the root Markdown files. The root files are longer, and two of them contain claims that `docs/risks.md` corrects.

## Intended stack — none of this is implemented

Python 3.11+ with FastAPI and asyncio; LiveKit Agents or Pipecat as media gateway (decision open, see `docs/risks.md`); Twilio Programmable Voice with Media Streams; Deepgram for STT; Cartesia Sonic or ElevenLabs for TTS; openSMILE for acoustic features; Redis for in-call state; PostgreSQL with pgvector for the longitudinal event store.

Current Claude model IDs, if Claude is used: `claude-opus-5-5` or `claude-sonnet-5-5` for the conversational path, `claude-haiku-5-5` for the parallel safety classifier. The planning documents name Claude 3.5 Sonnet, Gemini 2.0 Flash and GPT-4o; those are a generation out of date and must not be copied into new code.

## Project safety rules

- Never commit audio, transcripts, caller phone numbers, or any caller identifier. `.gitignore` covers the known patterns; check before adding a new data path.
- Never place a real outbound call, and never dial 988 or any live crisis line, from a development, test, or agent session. Use a dedicated test number.
- The parallel safety classifier must remain independent of conversational context. Its independence is what makes it resistant to prompt injection; routing it through the conversational prompt destroys the property the architecture exists to provide.
- The safety classifier is a required dependency, not an enhancement. If it is unavailable, the call degrades to a defined safe behaviour — it does not continue unmonitored. See `docs/architecture.md`.
- Treat the regulatory sections of the planning documents as unverified analysis, not as legal clearance.

## Documentation

- `docs/architecture.md` — intended design, with each component labelled implemented, planned, or unverified.
- `docs/development-setup.md` — prerequisites and configuration variable names for the intended Phase 1.
- `docs/claude-code-setup.md` — prioritized tooling and MCP guidance.
- `docs/risks.md` — risk register with evidence and verification status. Start here.
- `docs/verification-checklist.md` — project-specific checks for when code exists.

## Conventions

Develop on branch `claude/laughing-johnson-j3ut1n`. Do not push to `main`. Do not open a pull request unless asked.
