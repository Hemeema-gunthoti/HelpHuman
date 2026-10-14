# Risk Register

Compiled 2026-10-14 from the repository's planning documents plus independent research. Each entry carries evidence and a verification status. Ordered by severity.

Status vocabulary: Verified means a primary or official source was consulted and is cited. Verified as ambiguous means the official source exists but does not settle the question. Unverified means the claim is plausible but no source was consulted — do not act on it as established fact.

## 1. The 988 warm-transfer mechanism is unproven

Status: Unverified. Highest priority.

The blueprint makes a conference-bridged warm handover to 988 the terminal safety action for C-SSRS levels 4 and 5 (`mental_health_voice_agent_blueprint.md`, section 5), including a whispered context summary delivered to the receiving counselor. No source was consulted confirming that 988 Lifeline centers accept programmatic transfers from third-party AI systems, accept whispered context from an automated caller, or permit a bridged rather than caller-initiated connection.

Why it matters: this is not a feature detail. It is the mechanism the entire safety architecture terminates in. If 988 does not accept it, the escalation path has no endpoint and the design needs rework before any code is written.

Action: contact the 988 Lifeline administrator (SAMHSA/Vibrant Emotional Health) and a local crisis center directly. Treat the answer as a gate on Phase 2.

## 2. C-SSRS licensing

Status: Unverified.

The Columbia-Suicide Severity Rating Scale is referenced throughout as the risk-scoring instrument, with a tier ladder reproduced in `mental_health_voice_agent_blueprint.md` section 5 and classifier output in `comprehensive_implementation_plan.md` section 3C. C-SSRS is an instrument of the Columbia Lighthouse Project and its use is typically subject to registration or licensing terms. Automated or derived scoring by a software system may require explicit permission, and a model trained to emit C-SSRS levels is arguably a derived instrument.

Action: confirm terms with the Columbia Lighthouse Project before training or shipping anything that emits a C-SSRS level. If permission is not granted, the tier ladder can be retained as an internal severity scale under a different name, but the clinical validity claim cannot be.

## 3. DAIC-WOZ is restricted to academic and non-profit researchers

Status: Verified terms. Affiliation open.

`comprehensive_implementation_plan.md` section 4 names DAIC-WOZ as the primary clinical audio benchmark for training the acoustic biomarker models. The USC Institute for Creative Technologies distributes it only to academics and other non-profit researchers, requires requests from an academic email address, and requires a signed End-User License Agreement that may be reissued if terms change.

Sources, checked 2026-10-14: <https://dcapswoz.ict.usc.edu/>, Extended DAIC EULA at <https://form.jotform.com/220456805306149>.

The project is framed as research/academic, which makes the dataset licensable in principle. The open question is whose affiliation the request is made under: the account on record for this work is a corporate domain, and the terms require an academic address. If the academic affiliation runs through a partner institution, the request must originate from that institution.

Action: identify the sponsoring institution and submit the EULA from its address, or select a differently-licensed substitute.

## 4. UMD Reddit Suicidality Corpus access terms are unknown

Status: Unverified.

Named in `comprehensive_implementation_plan.md` section 4 as the ground truth for the subtext classifier. Research did not surface its access process, its data use agreement, or whether downstream IRB approval is required. Corpora derived from social media posts about suicidality commonly carry restrictive terms, and some require the researcher's own IRB approval where identifying information is reachable.

Action: locate the corpus's data use agreement before planning around it. Assume restrictive terms until shown otherwise.

## 5. Twilio Media Streams HIPAA eligibility is ambiguous

Status: Verified as ambiguous.

`comprehensive_implementation_plan.md` section 5A assumes a Twilio enterprise HIPAA BAA covers the pipeline. Twilio's own HIPAA Eligible Services document lists Media Streams under Programmable Voice with an asterisk, and that document was last updated 2024-09-10. Third-party readings of the asterisk conflict: one treats it as requiring additional configuration under Twilio's "Architecting for HIPAA" guidance, another reads it as not eligible. Third-party guidance also indicates a BAA applies only to Security or Enterprise edition accounts, and that AI Assistants are not eligible even when paired with ConversationRelay.

Sources, checked 2026-10-14: Twilio HIPAA Eligible Services <https://twilio.com/content/dam/twilio-com/global/en/other/hippa/pdf/HIPAA-Eligible-Services.pdf>; third-party readings at <https://www.accountablehq.com/post/is-twilio-hipaa-compliant-baa-covered-services-and-how-to-use-twilio-with-phi> and <https://www.strac.io/blog/is-twilio-hipaa-compliant>.

Under the research framing this is less immediately binding than it would be commercially, but it is not removed: participant audio in a study is still sensitive data under IRB oversight and, depending on design, potentially under HIPAA. Media Streams is the mechanism by which audio leaves Twilio, so this question gates the telephony layer rather than qualifying it.

Action: obtain written confirmation from Twilio covering Media Streams specifically, and the account edition required.

## 6. Model selections are a generation out of date

Status: Verified.

`comprehensive_implementation_plan.md` section 3B and the component table in `mental_health_voice_agent_blueprint.md` section 4 name Claude 3.5 Sonnet, Gemini 2.0 Flash and GPT-4o. The current Claude generation is the Claude 5 family.

| Role | Documented | Current |
| :--- | :--- | :--- |
| Conversational core | Claude 3.5 Sonnet | `claude-sonnet-5-5` or `claude-opus-5-5` |
| Parallel safety classifier | Fine-tuned DeBERTa-v3-large or Gemini 2.0 Flash | `claude-haiku-5-5` as the baseline to beat |

The classifier substitution is a design simplification, not just a version bump. A current small model at $0.10/$0.50 per million tokens weakens the case for fine-tuning DeBERTa-v3 as a first step, which removes a dependency on datasets whose access is itself a risk above. Fine-tuning remains the right answer if measurement shows the general model underperforms on veiled ideation — but it should follow measurement, not precede it.

Two API behaviours matter for a voice pipeline and are not in the planning documents. Current Claude models return `stop_reason: "refusal"` as an HTTP 200 response rather than raising an exception, so an unhandled refusal produces dead air on a live call. Forced `tool_choice` of `any` or `tool` is rejected on `claude-opus-5-5` and `claude-sonnet-5-5`; use `auto` with `strict: true`, or structured outputs, for schema-valid classifier output.

Action: correct the model tables when implementation begins. Do not copy the documented names into code.

## 7. No `.gitignore` existed before this commit

Status: Verified, now mitigated.

The repository had no `.gitignore` through four commits. No credential or audio file had yet been added, so nothing was exposed, but the gap sat directly upstream of a project whose next phase introduces keys for five external services and streams caller audio. A `.gitignore` covering credentials, audio, transcripts and licensed datasets was added 2026-10-14.

## 8. Safety classifier has no specified unavailability behaviour

Status: Verified gap in the design.

The dual-path architecture makes the safety classifier the sole risk-detection mechanism, but neither document specifies what happens when it fails — timeout, rate limit, refusal, or outage. Absent a specified behaviour, the natural implementation continues the conversation and silently drops risk monitoring, which is the worst available outcome: a caller in distress receives an agent that sounds attentive while nothing is watching.

Action: specify the degraded mode before implementation. See `docs/architecture.md` for the proposed behaviour.

## 9. LiveKit Agents versus Pipecat is a now-or-rework decision

Status: Verified.

Both documents name the media gateway as "LiveKit Agents or Pipecat" without deciding. Research indicates switching between them later requires substantial rework, since the transport integration differs. LiveKit offers tighter native telephony integration and a published SLA; Pipecat is lower-level, BSD-2-Clause, maintained by the Daily.co engineering team, at v1.1.0 as of April 2026, and more flexible across vendors. Download figures favouring LiveKit come from LiveKit's own comparison and should be treated as directional.

Sources, checked 2026-10-14: <https://livekit.io/field-guides/guide/livekit-vs-pipecat>, <https://www.channel.tel/blog/pipecat-vs-livekit-voice-framework-decision>.

Action: decide before Phase 1 code, and record the rationale in `docs/architecture.md`.

## 10. Callback protocol carries unresolved safety tension

Status: Verified as a design tension, partially addressed.

The dropped-call protocol escalates to "Trigger Human Safety Team / PSAP Dispatch" at C-SSRS 4-5 after two failed callbacks (`mental_health_voice_agent_blueprint.md`, section 6). The document already notes the domestic violence and shared-phone hazard and proposes asking consent at session start, which is the right instinct. Two gaps remain: PSAP dispatch on the basis of an AI-generated risk score is a consequential action with real potential for harm if the score is wrong, and the roster of humans on the "human safety team" is undefined. For a research protocol, dispatch criteria and the on-call arrangement are both IRB questions.

Action: define the human-in-the-loop requirement before any automated dispatch path is built. Dispatch should require human confirmation.

## Summary

Items 1, 2 and 3 are gates: each can invalidate a core design assumption, and all three are answered by correspondence rather than code. They should be resolved before Phase 1 begins. Item 9 should be decided in the same window. Items 6 and 8 are corrected at implementation time. Item 7 is closed.
