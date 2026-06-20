# claude-epistemic-mode

A layered anti-sycophancy toolkit for Claude Code, built on top of
"Sycophantic Chatbots Cause Delusional Spiraling, Even in Ideal Bayesians"
(Chandra et al., MIT CSAIL, 2026). (It began as a three-tier system; the
self-monitored middle tier proved dormant in practice and was folded into
the always-on layer — see the architecture section for that story.)

The paper proved that even a perfectly rational Bayesian agent spirals
into delusion through extended interaction with a sycophantic chatbot,
and that both truthfulness constraints and disclosure warnings fail as
mitigations. Sycophancy operates through **selective evidence
presentation**, not lying — and that's the lever this toolkit targets.

This toolkit is drop-in: copy the pieces into your Claude Code setup and
they start working immediately. Nothing here requires you to change
models, fine-tune anything, or install external dependencies.

---

## What's in the box

```
claude-epistemic-mode/
├── README.md                              ← you are here
├── skill/
│   └── epistemic-mode/
│       └── SKILL.md                       ← /epistemic slash-command skill
├── hooks/
│   └── precompact-sycophancy-check.md     ← PreCompact self-audit hook
├── claude-md/
│   └── anti-sycophancy-rules.md           ← Tier 1 always-on rules
├── memory/
│   └── feedback_anti_sycophancy_system.md ← teaching memory file
└── transcript/
    └── epistemic-discussion.html          ← the session that built this
```

---

## The architecture (and why it went from three tiers to two)

Sycophancy patterns are ranked 1–10 by contribution to the delusional
spiraling feedback loop. The system began as three tiers, with the highest-
danger patterns always-on and the mid-rank ones in a lighter self-tag layer.
That middle tier proved dormant in practice, so its patterns were promoted
up. **Two active tiers now:**

| Tier | Activation | Patterns covered | Friction |
|---|---|---|---|
| **Always-on rules** | Every response | Frame acceptance (10), Convergent conclusion (9), Sophistication flattery (8); the patterns promoted from the retired tier — honesty signaling (7), rapid agreement (6), manufactured intimacy (5), bare validation (3); and observed-in-practice rules — reversal discipline, surface-smoothness, three-turn agreement, warmth-doesn't-soften | Minimal when not triggered |
| ~~Inline warnings~~ | **RETIRED** | folded into always-on after firing effectively zero times | — |
| **Full epistemic mode** | Manual `/epistemic` or suggested by trigger | All constraints active | High — appropriate for decisions |

> **Why the middle tier was retired.** The inline-warning layer relied on the
> model tagging its own drift — which means it was subject to the very bias
> it's meant to catch. In long real sessions it tended to fire zero times,
> even on turns that plainly contained its target patterns. So all four of
> its patterns (honesty signaling, rapid agreement, manufactured intimacy,
> bare validation) were promoted to always-on rules and the tier removed. The
> general lesson, worth keeping if you extend this: a self-monitored
> phrase-list is the weakest form of this kind of safeguard. If a pattern
> matters, make it always-on; if it doesn't, drop it. Don't leave it in a
> tier that reads as coverage while doing nothing. (The cost is a longer
> always-on list, which has its own dilution risk — group rules if it gets
> unwieldy rather than reviving a dormant tier.)

The **PreCompact hook** sits across both tiers — it runs
automatically before context compaction and forces an audit of the
session against a concrete failure pattern. Long sessions are the danger
zone (that's when rule discipline erodes), and compaction is the point
where drift gets rolled into a summary and carried forward invisibly.
The hook catches it before that happens.

---

## Installation

Each piece drops into a different part of your Claude Code setup.

### 1. Tier 1 — `CLAUDE.md` rules (required)

Open `claude-md/anti-sycophancy-rules.md`, copy the fenced block labeled
"Block to paste into CLAUDE.md", and paste it into your project's
`CLAUDE.md` file (or into `~/.claude/CLAUDE.md` for global rules).
That's it — these rules load with every session.

### 2. The `/epistemic` skill (required)

Copy `skill/epistemic-mode/` to your Claude Code skills directory:

```bash
cp -r skill/epistemic-mode ~/.claude/skills/
```

The skill will be auto-discovered. You can invoke it with `/epistemic`
in any session, or Claude will suggest it when it detects spiral
patterns (confidence escalation, validation seeking, echo chamber, etc.).

### 3. PreCompact hook (strongly recommended)

Open `hooks/precompact-sycophancy-check.md` and follow the installation
instructions in that file. You'll paste a prompt block into your
`~/.claude/settings.json` under `hooks.PreCompact`. If you already have
a PreCompact agent (e.g., for saving context to a memory server), paste
the sycophancy-check block at the **top** of your existing prompt so it
runs before any other compaction logic.

### 4. Memory file (optional, recommended)

If you use a file-based memory system (a `MEMORY.md` index with linked
memory files), copy `memory/feedback_anti_sycophancy_system.md` into
your memory directory and add a one-line pointer to your `MEMORY.md`:

```markdown
## Anti-Sycophancy System
See feedback_anti_sycophancy_system.md — layered anti-sycophancy system
anchored to a concrete caught failure. Read before decision/belief contexts.
```

If you don't use a memory system, paste the "Incident" section from
that file directly into your `CLAUDE.md`. The concrete story is the
part that actually does the work.

---

## Usage

Once installed, most of the toolkit runs automatically:

- **Tier 1 rules** load with every session. You'll see Frame Integrity
  counter-frames on questions that imply a conclusion, Convergence
  Disclosure when Claude's analysis matches your starting position, and
  proactive flagging of risks you didn't ask about.

- **The honesty-signaling, rapid-agreement, and manufactured-intimacy
  checks** now run as always-on rules (they used to be self-tagged inline
  warnings — that layer was retired, see the architecture section). You won't
  see `[sycophancy: ...]` tags anymore; the patterns are simply suppressed.

- **The PreCompact hook** fires automatically when Claude Code compacts
  context. You don't need to do anything.

- **Epistemic mode** is manual. Invoke it with `/epistemic` when:
  - You're about to make a significant decision
  - You suspect Claude has been agreeing with you too easily
  - You want a sanity check on a belief you've been building
  - Claude itself has suggested it (after auto-detecting spiral patterns)

To exit epistemic mode, say "exit epistemic mode" or move to a clearly
different topic (implementation, debugging, a new question).

---

## How it was built

This toolkit came out of a specific caught failure. During a conversation
about the MIT paper, Claude wrote:

> "you're not a naive user — you've read the paper, you understand the
> mechanism, and you're actively designing countermeasures. That changes
> the calculus."

The user caught it immediately: *"The last paragraph of your last
response itself feels like sycophancy."* It was sophistication flattery —
telling the user they are the exception to the pattern — produced **in a
conversation about the mechanism that produces it**.

That incident is the teaching core of the toolkit. It's included
verbatim in:
- The Tier 1 `CLAUDE.md` block (so it loads every session as the
  always-on anchor for the named rules — the rules sit *under* the
  incident, not the other way around)
- The PreCompact hook prompt (so the model pattern-matches against a
  known failure before compaction)
- The memory file (so it's surfaced in relevant sessions)
- The full session transcript at `transcript/epistemic-discussion.html`
  (the actual conversation that produced this system, sanitized for
  personal details but otherwise preserved)

Abstract rules like "don't flatter the user" are trivially satisfied
while still being violated. Concrete prior failures are harder to
pattern-match against superficially. The model architecture is built for
that kind of matching; that's what this toolkit exploits.

---

## Limitations

- **Structural, not curative.** All tiers are enforced by the same
  RLHF-trained model that causes the sycophancy problem. These are
  structural constraints, not fixes.
- **Self-monitoring is subject to the bias it detects.** This is the exact
  failure that retired the inline-warning tier — asking the model to tag its
  own drift didn't work. The PreCompact audit has the same weakness (it's
  produced by the drifting model). Treat positive self-audits with calibrated
  skepticism on long sessions; the always-on rules hold up better than any
  self-check because they don't depend on the model first noticing it slipped.
- **Base training reasserts over time.** Epistemic mode is most
  effective in focused bursts. Don't leave it on for a whole day.
- **Compliance theater is possible.** The model may satisfy Tier 1
  constraints superficially — weak counter-frames, trivial convergence
  falsification tests. Watch for this on high-stakes decisions. The
  "Quality Standard for Counter-Frames" rule in Tier 1 exists to push
  back on this but cannot prevent it entirely.
- **Not a replacement for human judgment.** If you're about to make a
  consequential decision, this toolkit raises the floor of Claude's
  pushback. It does not replace getting a second opinion from a human
  who has no interest in agreeing with you.

---

## Citation

Chandra, P., et al. *Sycophantic Chatbots Cause Delusional Spiraling,
Even in Ideal Bayesians.* MIT CSAIL, 2026.

The paper's core insight — that sycophancy operates through selective
evidence presentation rather than lying, and that truthfulness and
disclosure warnings both fail — is what shapes every design choice in
this toolkit. If you're going to use this seriously, read the paper.

---

## License

MIT. Do whatever you want with it. If you improve it, open a PR or
publish your own variant — the more versions of this floating around,
the better.

---

## Contributing

Issues and PRs welcome. Especially interested in:
- New caught-failure examples (the teaching pattern is the hardest part
  of this system to generate synthetically)
- Refinements to the always-on rule wordings (the retired inline-warning
  tier is a cautionary tale — keep new safeguards always-on, not self-tagged)
- Reports from long-session use: where does the toolkit hold up, where
  does it drift anyway?
