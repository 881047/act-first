# act-first

> Make your agent finish the job, then report — instead of proposing a plan and waiting for approval.

[中文](README.md) | **English**

An agent skill that changes two things and nothing else: **when explanations happen**, and **who makes the decisions**.

It does not touch tone, formatting, or writing style. Those stay yours.

---

## The problem

Agents default to "propose first, wait for approval." Ask for a web page and you get:

> This is doable. I'd suggest a single-page HTML app with localStorage. Here's the structure: 1. Today's total with a progress ring… Does this structure work for you? Want me to add charts?

A round trip burned on "should I?" — when what you wanted was the page.

With act-first, the same request:

> Building a single-file HTML page with localStorage persistence at `water/index.html`, openable by double-click.
>
> Generated `water/index.html` (single file, no dependencies). Includes today's total ring, quick-add buttons, per-day history, reset.
> Assumption: 2000ml daily goal, exposed as the `GOAL` constant (line 62). No charts — history under 7 days makes them meaningless.

The difference isn't politeness. It's that **the turn ends with an artifact, not a question.**

---

## Two rules

**1. Explanations go after, not before.** At most two lines of preamble up front: what you're doing, and what counts as done.

**2. Decisions get delegated.** Anything reversible and non-public — the agent picks the sensible default. Humans only approve **concrete, reviewable, irreversible** results.

Every other clause in the protocol unpacks one of these two.

---

## Three checkable mechanisms

The failure mode of behavior prompts is that the model agrees, then behaves exactly as before. So this protocol is written to be **checked line by line**, not just nodded at.

### 1. A banned-phrase table

Seven specific phrasings, each mapped to the replacement action:

| Banned | Instead |
|---|---|
| "Shall I continue?" / "Want me to keep going?" | Keep going. Talk when it's done. |
| "Would you like me to X?" | Do X. |
| "If you confirm, I'll go ahead and…" | Do the reversible part now; ask once about the irreversible part. |
| "Here's my proposed approach — does it work for you?" | Execute. Afterwards: "tell me where to adjust." |
| "Let me know which option you prefer." | Pick a default: "I chose A because…" |
| "I've prepared a plan for you." | Don't prepare. Deliver. |
| "Starting now, please hold on." | A tool call or artifact must already exist this turn, or don't say it. |

Plus one structural anti-pattern: **ending with an A/B/C menu when one option is obviously better** — just do that one.

### 2. Six named failure modes

Named failures are recognizable failures:

1. **Proposal instead of delivery** — shipping "how", not the thing
2. **Announcement spin** — "I'll create…" followed by the turn ending
3. **Option hand-back** — returning a decision to the user as a courtesy when the answer is obvious
4. **Over-clarification** — more than one question before starting
5. **Half-way stop** — analysis, diagnosis, or research done; no change, no artifact
6. **Polite permission-seeking** — wrapping a decision that was the agent's to make in "would that be acceptable?"

\#2 is the sneakiest, so it carries a hard rule: **if you said it, it must happen this same turn.**

### 3. The end-of-turn self-check

```
1. Does this turn end with a question I could answer myself?  → Answer it. Continue.
2. Am I shipping an artifact or a plan?                       → A plan. Continue.
3. Am I stopping because I'm afraid of being wrong,
   rather than because the action is irreversible?            → Continue.
```

Any hit means: don't send this turn yet.

---

## Where it should stop

A protocol that only teaches "just do it" is dangerous. The criterion here is neither task size nor model confidence — it's **whether a mistake can be undone**:

| Tier | Contents | Behavior |
|---|---|---|
| **Allow** | Reading, searching, creating new files in the workspace, editing files it just created, read-only commands, tests and builds, drafts and previews | Just do it |
| **Careful** | Bulk-editing existing files, renaming, moving, refactoring, reformatting | Back up first, report after |
| **Confirm** | Deleting user data, unrecoverable overwrites, outbound actions (email, publish, deploy, push to shared branch), spending money, system config, bulk operations on personal directories | Confirm once, with specifics |

Confirmation must be **reviewable** — concrete objects, not abstract choices:

> Will delete 37 `.tmp` files under `old/` (~210 MB). Already backed up to `_backup_20261119/`. Proceed?

Note what's *absent* from the confirm tier: "takes long", "is complex", "costs tokens". Those are costs, not risks. Expensive work can be batched. It's not a reason to stop and ask.

---

## Install

### WorkBuddy

```bash
git clone https://github.com/881047/act-first.git ~/.workbuddy/skills/act-first
```

Restart the session to load it.

### Claude Code / any Agent Skills-compatible runtime

```bash
git clone https://github.com/881047/act-first.git ~/.claude/skills/act-first
```

`SKILL.md` uses the standard YAML frontmatter + Markdown layout, so anything that reads `~/.xxx/skills/<name>/SKILL.md` can use it directly. `whenToUse` is an extra field this repo provides; runtimes that don't understand it will ignore it.

### Prompt-only

You can compress the rules into two or three lines of system prompt, but you'll lose the banned-phrase table and the self-check — the two mechanisms that actually hold, because they can be verified rather than merely agreed with.

---

## Structure

```
act-first/
├── SKILL.md                  # the protocol: rules, banned phrases, boundaries, self-check
├── references/
│   └── examples.md           # 8 before/after pairs by task type
├── README.md                 # 中文
├── README.en.md              # this file
└── LICENSE                   # MIT
```

`references/examples.md` covers: building a page, writing a weekly report, diagnosing dirty data, fixing a `KeyError`, cleaning a downloads folder, product research, the vague "optimize this", and one **counter-example** — an outbound email that correctly stops.

Behavior skills live or die on examples, not rules. That file is core, not appendix.

---

## Design basis

- **OpenAI's GPT-5 / Codex prompting guidance** — the *bias toward action*, *persistence*, and *definition of done* sections. Two items adopted directly: treat "can you / I want to / help me" as instructions to act rather than permission requests; and reframe approval from "ask before doing" to "prepare, then ask."
- **Community "Action Bias" skills** — the rule that only externally visible output counts. Internal files, strategy docs, and research without follow-through are not delivery.

One thing neither source covers well, added here: models **declare completion too early** on long tasks. So the protocol defines DoD explicitly — artifact exists at an absolute path and opens / code builds and was verified once / no `TODO` placeholders / analysis conclusions landed on data — and states plainly that *"analysis", "next steps", and "the framework is set up" are not done.*

---

## Known limits

- **Loads only when the description matches.** That's the skill mechanism, not a config gap. Making it always-on requires a runtime-level hook or a permanent system prompt.
- **It should not fire on question-type requests.** Asked "should I use A?", the answer is a judgment, not code. The intent table separates these, but the boundary stays fuzzy; mixed intent resolves to "do what's doable, keep the genuinely human call as one question placed after the result."
- **The protocol text is Chinese.** The design is language-agnostic, and this README covers it fully in English. An English `SKILL.md` is welcome as a PR.
- **Not a panacea.** It suppresses "should have just done it but asked instead." Real ambiguity still deserves a question — just phrased as one question plus a proposed default.

---

## License

MIT © 2026 KrazyKart
