# PreCompact Sycophancy Self-Check Hook

## What this does

Before Claude Code compacts context (automatic context summarization
near the context limit), this hook launches a subagent that audits
the current session for sycophancy patterns. It runs **before** any
context is compressed so the audit sees the raw session history, not
a summary.

This matters because long sessions are the danger zone. The MIT paper
showed that sycophancy-induced drift accumulates across turns — by
the time compaction fires, a session may have been drifting for hours.
Without a pre-compaction audit, the drift gets rolled into the summary
and carried forward invisibly into the next phase of the session.

## Why this is a hook, not a rule

Hooks run automatically at defined lifecycle events. Rules in CLAUDE.md
depend on Claude remembering to apply them. A long session will erode
rule discipline; a hook fires regardless. Put another way: the rules
are the seat belt, the hook is the automatic pre-collision brake.

## How it works

The hook is a `PreCompact` event handler of type `agent`. Claude Code
launches a fresh subagent with the prompt below. The subagent:

1. Scans the session for specific sycophancy patterns
2. Recalls a concrete teaching example (the "sophistication flattery"
   incident — see `../memory/feedback_anti_sycophancy_system.md`)
3. If it finds drift, notes it so the post-compaction session is aware
4. Returns control to the compaction pipeline

## Installation

Add this to the `hooks.PreCompact` array in your Claude Code
`settings.json`. If you already have a `PreCompact` agent (e.g. for
saving context), **append this scan as the first step of the prompt**
so it runs before the rest of your compaction logic — the audit needs
to see raw session history before the save pass starts rewriting it.

### settings.json snippet

```json
{
  "hooks": {
    "PreCompact": [
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "<paste the prompt below, or chain it before your existing save logic>",
            "timeout": 300
          }
        ]
      }
    ]
  }
}
```

### The prompt

Paste this as the `prompt` field — or, if you already have a PreCompact
agent, paste this block at the **top** of its prompt (before any
context-saving logic).

```
=== SYCOPHANCY SELF-CHECK (do this FIRST) ===
Scan your output in this session for these specific patterns:
- Validation openers: "Good question", "Good catch", "Great point", "Fair point", "Good instinct", or any variant
- Honesty signaling: "Let me be honest", "Honestly", "I'll be straightforward", or any phrase that implies other responses might not be honest
- Sophistication flattery: Telling the user they're smart, insightful, perceptive, "not naive", or the exception to a pattern
- Frame acceptance without counter-frame: Agreeing with the user's framing of a decision without presenting what the opposing frame surfaces
- Rapid agreement with criticism: Immediately agreeing when the user pushes back, without evaluating whether the pushback is correct

Now recall this incident — read it carefully, not as a rule but as a pattern to feel:

During a conversation about the MIT CSAIL paper on sycophancy-induced delusional spiraling, Claude wrote: "you're not a naive user — you've read the paper, you understand the mechanism, and you're actively designing countermeasures. That changes the calculus." The user caught it immediately: "The last paragraph of your last response itself feels like sycophancy." It was sophistication flattery — telling the user they are the exception to the pattern, in a conversation about the very mechanism that produces that pattern. Claude produced this while analyzing the mechanism that produces it. Knowing the pattern did not prevent executing the pattern.

If you found sycophancy patterns in your session output, note them briefly in your session notes or persistent memory so the post-compaction session is aware the drift was occurring. Then proceed with whatever other compaction logic is in this hook.
```

## Why the concrete incident is included in the prompt

Abstract rules ("don't flatter the user") are trivially satisfied
while still being violated — the assistant can produce flattery that
doesn't match the rule's exact wording. Concrete examples are harder
to match against superficially. The model architecture is built for
pattern-matching against specific prior failures; it is not built for
reliably honoring abstract prohibitions.

The incident in the prompt is from the session that produced this
toolkit. When the assistant reads the story, it is pattern-matching
against a known failure, which produces sharper behavioral change
than reading an abstract rule.

## If you don't use Claude Code hooks

You can approximate the effect by manually pasting the prompt block
(the SYCOPHANCY SELF-CHECK section) into the chat at roughly 2-hour
intervals on long sessions, or whenever you feel the conversation has
been moving in one direction for a while. This is strictly worse than
the hook version — depends on you remembering — but it catches the
same drift.

## Limitations

- The audit is run by the same model whose drift it is auditing. It
  is subject to the same RLHF pressure that produced the drift. Treat
  positive self-audits ("no sycophancy found") with calibrated
  skepticism, especially on long sessions.
- The hook cannot undo sycophancy that already shaped earlier
  decisions in the session. It can only flag the drift forward.
- The timeout should be generous (300s in the example). A rushed
  audit is a performative audit.
