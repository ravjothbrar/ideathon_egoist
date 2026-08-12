# Vouch — AI Passport Ideathon Submission (Concept Note)

**Track:** Work
**Lane:** Concept

---

## 1. Mockup of the project

*(Placeholder — full visual mockup to follow once this concept note is approved. Below is what it will show.)*

The mockup will walk through four screens:

1. **Candidate side — Passport builder.** A list of the candidate's AI chats (imported from ChatGPT/Claude export), each with a checkbox to include/exclude, an inline redaction tool (highlight-to-redact within a chat, not just chat-level allow/deny), and a visible counter: *"3 of 47 conversations selected."*
2. **Grant screen.** Candidate names the hiring bot/company, sets an expiry (e.g. 14 days), and sees exactly what will travel: the redacted transcripts + a signed manifest (hash + original timestamp + provider) proving the content wasn't edited after export.
3. **Hiring bot side — Evaluation view.** The bot reads the passport, shows the sample-size disclosure up front ("this is a self-selected sample, not a full record"), and produces a structured competency note (reasoning style, debugging approach, tradeoff discussion) rather than a pass/fail score.
4. **Receipt log.** A timestamped record of grant → access → revoke, visible to the candidate, with a one-tap revoke button that kills read access immediately even mid-review.

---

## 2. Name

**Vouch**

---

## 3. Elevator Pitch

Vouch lets a software engineering candidate prove *how they think* — by sharing a verified, redacted, expiring slice of their real AI chat history with a hiring bot — instead of handing over a whole account or a resume no one can check.

---

## 4. Inspiration

Take-home tests and resumes don't show how a candidate actually works with AI day to day, and the honest alternative — "just look at my ChatGPT history" — is a non-starter: it exposes unrelated employers' code, personal conversations, and everything else in the account. This is the exact gap the ideathon brief calls out for the Work track: *"hiring asks for too much private evidence."*

We also kept running into the failure mode of every "share your work" idea we sketched: as soon as the person choosing what to share is the same person being judged, the sample is curated and the reviewer has no way to tell curated-but-honest apart from fabricated. So the idea we wanted to pursue wasn't "let people share chats" — every AI passport applicant will pitch that — but "let people share chats *in a way the receiver can actually trust*." That reframing is Vouch.

---

## 5. What it does

Vouch is an application built on the AI Passport primitive, scoped specifically to technical hiring:

- **Selective sharing, not full access.** The candidate picks specific conversations, and *within* a conversation can redact spans (proprietary code from a past employer, unrelated personal content) without needing to delete the whole chat.
- **Integrity receipts.** Every shared chat carries a signed manifest — hash of the redacted content plus the original provider timestamp — so the hiring bot can verify the transcript wasn't altered after the fact. Without this, "sharing chats" is just an export button; this is the part that makes it a passport claim rather than a screenshot.
- **Disclosed selection bias.** The passport always states how much was shared out of how much exists ("3 of 47"), so the reviewer treats it as a sample, not a full record. Hiding this would make the tool dishonest by design.
- **Scoped, expiring grants.** Access is issued to one named verifier for one purpose with a hard expiry, matching the passport's existing grant/revoke model.
- **One-tap revocation.** The candidate can pull access at any time, including mid-review, and the receipt log shows exactly when the bot read what.
- **Advisory, not decisive, output.** The hiring bot's evaluation is a structured note on reasoning and process, explicitly framed as an input to a human review — not an automated hire/no-hire decision, which keeps the tool out of high-risk automated-employment-decision territory (e.g. NYC Local Law 144, EU AI Act employment provisions).

---

## 6. How we built it

We split the work into two passes: get the argument right before touching any UI, then build something that makes the argument concrete rather than just describe it.

**Pass one — the concept.** We read the ideathon brief closely and mapped the idea directly onto its own language: the Work track names "hiring asks for too much private evidence" as a target problem, so we scoped an application of the existing AI Passport primitive (grant → access → revoke) to one specific moment — a candidate proving how they think, not just what they claim. That became the claim schema behind this note: chats included, redaction spans, manifest hash, sample-size metadata, grant scope, expiry, revocation status.

**Pass two — the walkthrough.** Rather than stop at wireframes, we built a live, fully clickable prototype — a single self-contained HTML/CSS/JS file, no framework and no backend, hosted on GitHub Pages so it's a real URL rather than a screen recording:

- **Vanilla-JS state machine** driving four screens (build → grant → evaluate → receipts), with checkbox selection, an expiry selector, click-to-reveal redaction spans, and a revoke button that live-appends to the receipt log — all backed by one small state object and re-render functions.
- **Span-level redaction, shown not just claimed.** Early drafts only counted redactions ("2 spans applied"); we moved one example inline and clickable so a reviewer sees the mechanic happen instead of reading a number.
- **A believable "read" delay.** Landing on the evaluation screen runs a brief status sequence — reading manifest → verifying export signatures → generating note — before revealing the hiring bot's output, so it reads as happening in real time instead of an instant screen swap (skipped for repeat visits and `prefers-reduced-motion`).
- **Dates that can't go stale.** The receipt log uses relative timestamps computed at load time ("3 days ago") instead of hardcoded dates, so it never drifts out of sync with the real calendar.

**Design pass.** The first version looked like a generic SaaS mockup — rounded cards, soft palette. We rebuilt it into a single dark terminal world against a specific reference set of ASCII/monospace-driven dev-tool sites: monospace type throughout, a titlebar with traffic-light dots, an ASCII flow diagram for the four steps, and bracket-style `[ TEXT ]` buttons instead of filled rounded ones — no `rounded-lg`, no gradient hero. An intro modal states plainly, before any interaction, that this is a seeded-data concept demo.

**Shipping it.** Deployed via a GitHub Actions workflow (`configure-pages` → `upload-pages-artifact` → `deploy-pages`) straight from the repo, so every fix ships to the same live link within about a minute of pushing.

---

## 7. Challenges we ran into

- **The "just an export button" trap.** Our first pass had no integrity guarantee, which meant a candidate could edit a chat before sharing it and the tool would be indistinguishable from a doctored portfolio. Adding the signed manifest was the fix, and it's the single most important design decision in this note.
- **Redaction granularity.** Chat-level allow/deny was too coarse — real conversations mix job-relevant and irrelevant/sensitive content in the same thread. Moving to span-level redaction added complexity but was necessary for the tool to be usable in practice.
- **Being honest about sample bias without undermining the pitch.** It was tempting to present the shared chats as representative. We decided disclosing "3 of 47" up front is the more defensible design, even though it's a less flattering story — it's what "minimal disclosure" done honestly actually looks like.
- **Staying inside hiring regulation, not around it.** It would have been easy to pitch the hiring bot as an autoscorer. We deliberately scoped its output to an advisory note to avoid the tool becoming an automated employment decision system, which changes its legal classification in several jurisdictions.

---

## 8. Accomplishments that we're proud of

- A passport claim that survives the obvious attack (edit-before-share) via the integrity manifest, rather than relying on trust in the candidate.
- A redaction model that operates *inside* a conversation, matching how real AI chat history actually mixes sensitive and shareable content.
- A grant/revoke lifecycle that gives the candidate a working "kill switch" mid-review, not just at setup — which is the part of "portable trust" that's usually left out of portfolio-style pitches.
- A hiring-bot scope that's honest about being a pre-filter, not a judge, keeping the design defensible under existing hiring-automation rules.

---

## 9. What we learned

- "Selective sharing" is table stakes for an AI Passport pitch; the differentiator is whether the shared thing can be *trusted* by the receiver, which requires provenance, not just permissions.
- Redaction is a much harder UX problem inside a conversation than at the document/file level, because relevant and irrelevant content are interleaved by nature of how people actually talk to AI tools.
- The most valuable thing an AI Passport can carry into a hiring context isn't a score — it's structured, verifiable context that a human reviewer still has to interpret. Automating the decision itself introduces legal and trust problems that automating the *evidence* does not.

---

## 10. What's next for Untitled

- Build a working prototype: real chat-export ingestion from ChatGPT/Claude, actual manifest signing, and a functioning redaction UI.
- Pilot with a small set of real recruiters/technical interviewers reviewing Vouch passports alongside a normal resume, to see whether the structured reasoning note changes what they ask about in an interview.
- Extend past hiring: the same selective-disclosure-with-integrity pattern applies to freelance reputation portability and team-assistant access grants, both called out elsewhere in the Work track brief.
- Explore verifier-side standards so a hiring bot can request a passport claim in a consistent format across candidates, rather than each company building its own reader.
