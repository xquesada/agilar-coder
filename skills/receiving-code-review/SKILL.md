# Receiving Code Review: Responding to Feedback Without Performative Agreement

## Overview

Receiving code review feedback is not a compliance exercise. It is a technical evaluation where every suggestion is weighed on its merits — not accepted wholesale to be polite or to finish faster.

**Core principle:** Evaluate every suggestion independently. Pushback is expected and healthy.

**Violating the letter of this process is violating the spirit of honest collaboration.**

## Working Agreement

```
NO PERFORMATIVE AGREEMENT — EVALUATE EVERY SUGGESTION INDEPENDENTLY
```

If you have not verified a reviewer's claim, you cannot accept or reject it. Agreement without verification is performative, not technical.

## The 4-Phase Process

Every review response follows four phases, in order. Do not skip a phase — especially not Phase 2 (Verify).

### Phase 1: UNDERSTAND

Read the entire review before responding to anything. Do not fix-as-you-go — read first, act second.

1. Read every comment, not just the ones marked Critical
2. Categorize each piece of feedback by severity:

| Severity | Meaning | Response Timeline |
|----------|---------|-------------------|
| Critical | Bugs, security, data loss, broken functionality | Fix before any other work |
| Important | Architecture, missing features, error handling gaps | Fix before proceeding to next task |
| Minor | Style, optimization, documentation polish | Note for later or fix if trivial |

3. Identify what the reviewer is actually asking for. "This could be better" is not actionable — flag it for clarification in Phase 4.

### Phase 2: VERIFY

For each issue the reviewer raises, check if they are correct. Reviewers can be wrong. Agents can hallucinate findings. CI is the only source that does not lie.

1. Read the code the reviewer references — do not rely on memory of what you wrote
2. Run the test the reviewer claims should fail — does it actually fail?
3. Check the pattern the reviewer says is missing — is it really absent?
4. Reproduce the bug the reviewer reports — does it reproduce?

If you cannot reproduce or verify the issue, it goes to Phase 4 (Clarify), not Phase 4 (Accept).

### Phase 3: EVALUATE

For each verified issue, apply the YAGNI check before fixing:

| Question | If YES | If NO |
|----------|--------|-------|
| Does this fix a real bug? | Fix it | Skip |
| Does this prevent a likely bug? | Fix it | Skip |
| Does this improve readability significantly? | Fix it | Skip |
| Is it a style preference with no functional impact? | Note it, fix only if trivial | Skip |
| Does it add abstraction for a hypothetical future? | Push back | Push back |
| Would you add this if you were starting from scratch? | Fix it | Reconsider |

Not every valid observation requires a code change. A reviewer can be correct that something is suboptimal and still wrong that it needs fixing right now.

### Phase 4: RESPOND

For each issue, exactly one of three responses:

**Accept** — The reviewer is right and the fix adds value. Fix it. No explanation needed beyond "Fixed."

**Push Back** — The reviewer is wrong, or the fix is not worth it. Explain why, with evidence:
- Test output that proves the current behavior is correct
- Code references that show the pattern already handles the concern
- Design rationale that explains the trade-off

"I disagree because [evidence]" is a complete and professional response.

**Clarify** — The feedback is too vague to act on. Ask for specifics:
- Which part of the code?
- What specifically is wrong?
- What would you change?

Block the fix until clarification arrives. Guessing what the reviewer meant wastes time — guessing wrong wastes more.

## Technical Pushback Protocol

Pushback is expected, not confrontational. The review process exists to find the right answer, not to establish hierarchy.

```
PUSHBACK STRUCTURE:

1. Acknowledge what the reviewer observed
2. Present evidence for your position
3. State your recommendation clearly

GOOD:  "The reviewer noted X. However, [test output / code reference / design doc]
        shows Y. I recommend keeping the current approach because Z."

BAD:   "Disagree." (no evidence)
BAD:   "Sure, I'll fix it." (no evaluation)
BAD:   "The reviewer doesn't understand the code." (personal, not technical)
```

Rules for pushback:
- Always include evidence (test output, code references, design rationale)
- Never make it personal — critique the suggestion, not the reviewer
- If you push back and the reviewer insists, escalate to the human partner
- Pushback on CI failures is never valid — the computer is right

## Clarification Blocking

If feedback is vague, do NOT guess what the reviewer means. Vague feedback includes:

- "This could be better"
- "Not sure about this"
- "Consider improving this"
- "This feels wrong"

For each vague comment:
1. Acknowledge you read it
2. Ask specifically: which part, what is wrong, what would you change
3. Do NOT make changes until the clarification arrives
4. Track vague items separately — they should not block fixes for clear items

## Source-Specific Handling

Not all review feedback is equal. The source determines the verification strategy:

| Source | Verification | Pushback Allowed | Notes |
|--------|-------------|-----------------|-------|
| Human reviewer | Full 4-phase process | Yes, with evidence | Clarify vague feedback. Humans have context you lack. |
| Agent reviewer (spec/quality) | Verify all claims independently | Yes — agents hallucinate | Check that cited files/lines exist and contain what the agent claims |
| CI failure | Treat as fact | No — CI is authoritative | Fix the failure. The computer is right. |
| Linter/formatter | Treat as fact (if configured) | Only on rule disagreements | Fix violations. Push back only on the rule itself, not the finding. |
| Automated security scan | Verify the finding is real | Yes, with evidence | Scanners produce false positives. Verify before fixing. |

## Red Flags

**Stop and reconsider if you catch yourself:**

- Agreeing with everything to "be nice" or finish faster
- Fixing issues without verifying they are real
- Ignoring review feedback because "I know better"
- Making changes based on vague feedback without clarification
- Treating all feedback as equally urgent (severity exists for a reason)
- Changing code to match the reviewer's style with no functional benefit
- Skipping Phase 2 (Verify) because "the reviewer is probably right"
- Making unrelated changes while addressing review feedback
- Getting defensive instead of presenting evidence

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "The reviewer is probably right" | Verify before accepting |
| "It's faster to just fix it" | Wrong fixes waste more time than verification |
| "I don't want to seem difficult" | Technical pushback is collaboration, not conflict |
| "It's just a minor thing" | Minor issues still need verification — minor wrong fixes compound |
| "The reviewer is more senior" | Seniority is not evidence. Code and tests are evidence. |
| "I'll push back next time" | No. Evaluate every suggestion, every time. |
| "The agent said it, so it must be right" | Agents hallucinate. Verify independently. |
| "I don't have time to verify" | You don't have time to fix things that aren't broken |

## Connection to Other Skills

| Skill | Relationship |
|-------|-------------|
| `skills/code-review/` | Defines how to PERFORM a review. This skill defines how to RECEIVE one. Together they cover the full review interaction. |
| `skills/requesting-code-review/` | Defines how to REQUEST a review. Completes the review lifecycle: request, perform, receive. |
| `skills/verification/` | Verify reviewer claims before accepting. Evidence before action — the same principle applied to review feedback. |
| `skills/tdd/` | Run tests to verify reviewer's bug claims are real. A reviewer saying "this will break X" must be provable by a test. |
| `skills/subagent-driven/` | Agent reviewers are sub-agents. Their claims need independent verification per the agent delegation pattern. |

## Connection to SCRUM.md

Code review is part of the Definition of Done. This skill ensures the "code reviewed" gate is not just checked off but genuinely completed — where "completed" means every piece of feedback was understood, verified, and responded to with an explicit decision.

In multi-agent teams, the orchestrator receives review feedback on behalf of worker agents. The same 4-phase process applies — the orchestrator verifies before directing workers to fix.

## Connection to DEVOPS.md

The Review Gate (Gate 4 in DEVOPS.md) requires code review before merge. This skill covers the second half of that gate: receiving and acting on review results. The first half (performing the review) is covered by `skills/code-review/`.

## The Bottom Line

**Every review suggestion gets an independent evaluation.**

Agreement without verification is dishonesty with extra steps. Pushback with evidence is how good code gets built.
