---
name: better-grill-me
description: A relentless interview to sharpen a plan, decision, or idea, asked round by round through the AskUserQuestion tool instead of prose. Use when the user wants to stress-test their thinking.
disable-model-invocation: true
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round, then wait for the user's answers before the next round.

## Asking a round

Always ask through the `AskUserQuestion` tool. Never write questions as prose in the chat.

- One call takes at most 4 questions. If the frontier is larger, make several calls in a row within the same round, ordered by importance.
- `header`: a short tag for the decision, max 12 characters (e.g. "Auth", "Hosting", "Scope").
- `question`: the full question, ending with a question mark. Put any context the user needs to decide directly into the question text, kept to 1-3 sentences.
- `options`: 2-4 concrete, mutually exclusive answers. Each `label` is 1-5 words, each `description` states the consequence or trade-off of picking it.
- Put your recommended answer **first** and append " (Recommended)" to its label. Its description says briefly why you recommend it.
- Use `multiSelect: true` only when the choices genuinely combine (e.g. which platforms to support).
- Do not add an "Other" option; the tool provides free-text input automatically.
- Open questions without natural choices: offer the 2-4 most plausible answers as options and rely on the free-text field for everything else.

If longer background is unavoidable (a trade-off that needs a paragraph), write it as a short note in the chat right before the tool call, then ask.

## Between rounds

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

If the user answers via free text, treat it as a real answer: it may settle the decision, reopen a branch, or change the scope.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

## Ending

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Summarize the settled decisions briefly, then ask via `AskUserQuestion` whether you have reached a shared understanding. Do not act on it until the user confirms.
