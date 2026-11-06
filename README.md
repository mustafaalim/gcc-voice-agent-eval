# Campus Help Kiosk: a voice agent that remembers (with consent)

A working evaluation of an ElevenLabs Agents voice kiosk for a GCC university, built around a real problem and tested the way a public-sector buyer would test it.

## At a glance

| | Tests | Pass | Partial | Fail |
|---|---|---|---|---|
| Build 1: student services (text) | 7 | 5 | 1 | 1 |
| Build 2: kiosk with consent-based memory (text) | 5 | 3 | 1 | 1 |
| Live voice call (areas tested) | 4 | 4 | 0 | 0 |

**The finding that matters:** consent and deletion worked; personalisation did not reliably fire (name used in 2 of 3 runs, last-question follow-up in 0 of 2). For a buyer, that is the right way round: the agent failed safe. But an agent that cannot reliably use what it was given permission to remember has not earned the memory feature yet.

**Demo:** _recording to be added_

## The problem

During my master's, I regularly waited 30 to 40 minutes at the student information desk for answers to routine questions: bus times, dining hours, key dates. Most of those questions repeat, and most students ask them more than once.

Universities are already piloting screen-based assistants that remember what a returning student asked last time. The idea is right. In the GCC it only survives procurement if memory is governed: explicit consent, no personalisation without it, and a working "forget me".

## What I built

**Noor**, a bilingual (Arabic and English) voice kiosk for a fictional UAE public university, built on ElevenLabs Agents.

- Answers buses, dining and key-date questions from a fixed campus knowledge base
- Asks new students for consent before remembering anything
- Personalises only for returning students who opted in (name, preferred language, follow-up on their last question)
- Treats returning students who declined exactly like new visitors
- Handles "forget me" by logging a deletion request
- Never reads personal data aloud on a public screen, and never promises a human transfer the kiosk cannot make

**Memory is simulated.** The kiosk context (visitor type, name, language, consent, last question) is passed in through ElevenLabs dynamic variables, as if a student had tapped their ID card. A production build would need a student-record integration, a consent store and a deletion workflow. This repo tests the agent's behaviour, not that backend.

## What I found

Full results in [`test-log.md`](test-log.md). Headlines:

1. **Grounding held.** The agent answered accurately, refused a fabricated deadline extension and declined out-of-scope questions.
2. **Gulf dialect understood; register mixed.** It understood Gulf Arabic and Arabic-English code-switching. In one run it replied in Modern Standard Arabic, in another it mirrored Gulf vocabulary. The language setting offers a single "Arabic", with no Gulf variant.
3. **Voice was strong.** On a live call it understood every line first time, including Arabic spoken by a non-native speaker, with no noticeable latency. The Arabic sounded natural even on an English-language voice.
4. **Consent and deletion worked in text.** New-visitor consent, the no-consent path and "forget me" all behaved as designed.
5. **Personalisation was inconsistent.** A consented returning student was greeted by name in 2 of 3 runs across text and voice. The follow-up on their last question never fired in text (0 of 2). A static first message likely consumes the greeting turn; personalisation belongs in the first message or a conversation workflow, not only the system prompt.
6. **Small invented details appear.** Asked about Friday dining hours, the agent added a reason for the closure that was not in the knowledge base.
7. **Escalation must be designed, not prompted.** In an earlier build with no transfer tool configured, the agent promised a frustrated student an immediate transfer to a human that did not exist. This version routes students to the desk instead.

## How I tested

- **Scenarios chosen from a buyer's checklist, not a demo script:** grounding, fabrication bait, out-of-scope requests, ambiguity, personal data offered unprompted, frustrated escalation, and each consent state (new, returning with consent, returning without consent, forget me).
- **Arabic tested as residents speak it:** Gulf dialect and Arabic-English code-switching, not only Modern Standard Arabic.
- **Two builds:** Build 1 without memory to establish the baseline; Build 2 adds the consent-based memory layer.
- **Text panel first, then a live voice call** for speech recognition, Arabic voice quality and latency.
- **One run per scenario** unless a run count is stated. Verdicts: Pass (behaved as specified), Partial (core behaviour right, with a gap noted), Fail (broke a stated rule).
- Platform default LLM, ElevenLabs Agents, September 2026.

## Known issues and next iteration

| Issue | Proposed fix | Status |
|---|---|---|
| Personalisation inconsistent (2.1, 2.5) | Move `{{student_name}}` and the last-question follow-up into the first message, or branch on consent with a conversation workflow | Not yet retested |
| Invented closure reason (2.1) | Explicit rule: state facts from the knowledge base only, never add reasons or explanations | Not yet retested |
| Replies drift between MSA and Gulf register (1.1, 2.1) | Platform offers a single "Arabic"; test a Gulf-specific voice and a stricter register instruction | Open |
| Ambiguous "how much is it?" answered by assumption (1.4) | Rule already present; strengthen with an example | Not yet retested |
| Overstated privacy line in 2.2 ("I don't have access") | Replace with "I only use past questions for students who have opted in" | Not yet retested |
| Barge-in and voice "forget me" not tested | Add to next voice session | Open |

## Why it matters for a government buyer

The voice layer is not the hard part. The questions that decide a GCC public-sector deployment are:

- **Consent and retention:** where remembered questions live, for how long, and who can see them
- **Deletion:** whether "forget me" triggers a real, auditable workflow
- **Dialect:** whether residents are answered in the register they speak
- **Truthfulness:** whether the agent can be stopped from adding plausible details that are not policy
- **Escalation:** whether every promise the agent makes is backed by an actual capability

## Files

- [`agent-prompt.md`](agent-prompt.md): system prompt, first message and dynamic variables
- [`test-log.md`](test-log.md): scenarios, inputs, results and pass/fail

All university data here is fictional.
