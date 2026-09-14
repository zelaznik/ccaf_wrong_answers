---
name: ccaf-review
description: >-
  Re-quiz a Claude Certified Architect (CCAF) exam question the user got wrong.
  Use when the user pastes a wrong-answer review that has a SCENARIO, a question,
  a list of answer options, and markers like "Correct answer" / "Your answer is
  incorrect" / "Explanation" / "Overall explanation" / "Domain". Also use when the
  user invokes /ccaf-review. Archive the raw paste to the next questions/NNN.txt file,
  then show only the de-biased, reshuffled question and options; grade and explain only
  after the user answers.
---

# CCAF Wrong-Answer Review

The user is studying for the Claude Certified Architect Foundations exam. They paste
a question they got wrong (exported from a test review) and want to re-attempt it
without seeing which option they originally picked or which one is correct. Run it as
a two-phase quiz.

## Input format

The pasted text is a raw copy from a test-review page and is messy. Expect:

- A header like `Question 60Incorrect` (question number jammed against "Incorrect") — ignore it.
- `SCENARIO:` or `Scenario:` — sometimes prefixed with a short title (e.g. `Scenario: Developer Productivity with Claude`). Keep the situational text; drop the title label if it's just a heading.
- `QUESTION:` — the question stem. **This label is sometimes absent** (e.g. the stem is folded into the end of the scenario paragraph and ends in `?`). If there's no `QUESTION:` label, infer the stem as the interrogative sentence(s), usually the last sentence(s) ending in `?`.
- Several answer options. Each option is a line (or few lines) of answer text, each followed by an `Explanation` label and an explanation paragraph.
- `Correct answer` — a **leading** marker: the option text *immediately after* it is the correct one.
- `Your answer is incorrect` — marks the option the user *originally* chose. **This is irrelevant to the re-quiz — ignore it entirely.**
- `Overall explanation` — a longer teaching explanation (may contain HTML tags like `<p>`, `<code>`, `<b>`, `<a href>`). Strip the tags; keep the prose and any doc links.
- `Domain` — the exam domain, e.g. `Domain 3: Claude Code Configuration & Workflows`. Note it for the reveal.

When parsing, separate the four pieces per option: (1) the answer text, (2) its per-option explanation, (3) whether it's the correct answer, (4) whether it was the user's original pick. You need #1 for phase 1 and #1–#3 (plus overall explanation) for phase 2.

## Phase 1 — present the de-biased question

If the user invoked the skill with no question yet, reply exactly with a short ready line like: `Ready — paste the question whenever. I'll return just the stem and the options in a fresh random order.` Then stop.

Once you have the question text, output **only**:

1. The scenario (situational context).
2. The question stem.
3. The answer options, **re-shuffled into a new random order** and relabeled `A.`, `B.`, `C.`, `D.` in that new order.

Reshuffling matters: the source may have the correct answer in a fixed slot or contain "A is correct"-style leakage, so never preserve the original ordering or labels. Randomize.

Output **nothing** about which option is correct, which the user picked, no explanations, no domain, no hints, no counts of how many are correct. Do not comment on difficulty. End by inviting them to answer with a letter.

Keep track (silently, do not print) of which shuffled letter maps to which original option and which one is correct, so you can grade next turn.

## Phase 2 — grade and explain

When the user replies with their choice (a letter matching your shuffled labels, or the answer text):

1. State plainly whether their pick was **correct** or **incorrect**, and name the correct option by its shuffled letter and text.
2. If they were wrong, briefly say what their chosen option gets wrong (use that option's per-option explanation).
3. Explain **why the correct answer is right** — synthesize the correct option's explanation and the overall explanation into a clear, tutoring answer. Surface the key insight, not just a restatement.
4. Briefly note why the other distractors fail (one line each), drawing on their per-option explanations.
5. Mention the exam **Domain** and include any documentation links from the overall explanation.

Match the depth and tone of a good tutor: explain the underlying principle so the concept transfers to other questions, not just this one. Do not be sycophantic; if they got it wrong, be direct and focus on the reasoning.
