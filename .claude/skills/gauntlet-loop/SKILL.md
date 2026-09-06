---
name: gauntlet-loop
description: Turns any goal into one short, paste-ready "gauntlet loop" prompt - a prompt that makes an agent set a concrete quality bar, split the work into small judgeable pieces, run a builder and a separate harsh critic on each, compare blind against the bar, and loop until it wins. Adds an auditor that verifies the loop is not cheating, a judge that can be kept blind to the implementation, and isolated contexts for competing builders. Works for builds, writing, code, research, or design. Triggers on "/gauntlet-loop", "gauntlet loop", "gauntlet this", "make a gauntlet prompt", "loop until it beats X".
---

# Gauntlet Loop

The user gives a goal. You give back ONE short prompt they can paste into a fresh agent session.

You are not doing the work. You are writing the prompt that makes another agent grind on the work until it beats a real reference.

## Flow

1. **Read the goal.** One line restatement in your head, not on screen.
2. **Set the bar.** If the user supplied a reference, use it. If not, offer **2 or 3 candidate bars**, one line each, and stop. Wait for their pick. Do not write the prompt yet.
3. **Set the dials, in the same breath.** Audit, blind mode, and isolation are off by default. If the goal clearly implies one, turn it on and say so in one line. If it is a genuine toss-up, ask it alongside the bar question - one stop, not two.
4. **Write the prompt.** One block, paste-ready, no preamble, no headings inside it, no narration after it.
5. **Offer to run it.** One flat line under the prompt: "I can run this here." Not a question.

If they say run it, you become the lead agent and follow the prompt you just wrote.

## The bar is the whole trick

Everything else in a gauntlet loop is scaffolding. The loop only produces quality if the thing it compares against is real.

A bar has to pass three tests:

- **Named.** A specific thing, not a category. "Stripe's pricing page" works. "Award-winning SaaS sites" does not.
- **Fetchable.** The critic can actually get it - screenshot the live page, read the published piece, run the binary, open the repo, watch the footage. If the agent cannot obtain it, it will hallucinate the comparison.
- **Comparable.** Both can sit side by side and a judge can pick one. If you cannot imagine the A/B, it is not a bar.

Bars by goal type:

| Goal | Bar that works |
|---|---|
| Website, app, UI | The live site of a specific best-in-class product, screenshotted at the same viewport |
| Game, 3D, visual | Real footage or screenshots from a named shipped title |
| Writing | A specific published piece by a named author or publication, same length and format |
| Code, tooling | A named repo's implementation, plus its benchmark or test suite as the measurable half |
| Research, analysis | A named analyst report or a paper's methods section, judged on rigour and coverage |
| Deck, doc, deliverable | A real artifact from a firm known for it, same page count |

When you propose bars, prefer the hardest one the agent can genuinely reach. A bar that is too easy makes the loop exit on round one.

If the goal has a measurable half (load time, token cost, benchmark score, word count, pass rate), name it alongside the reference. Taste plus a number beats taste alone.

## The three dials

Each one is off unless it earns its place. Each adds a sentence or two to the prompt, never a section.

### 1. The auditor

An agent that says it fanned out subagents, compared blind, and built in isolation is the same agent grading its own compliance. The auditor is the fix, but only if it checks evidence instead of claims.

**Design the artifact trail first, the auditor second.** The loop has to leave things on disk that exist independently of any agent's narration:

- Where each builder worked - a path, a worktree, a branch with its own commits.
- The exact bundle the critic was handed - the screenshot files, the extracted text, the stripped copies. Not a summary of it.
- The verdict, the named gap, and which round it belongs to.

The auditor then checks the trail: are the builder paths distinct with distinct histories, do the screenshots exist at the claimed viewports and inside the round's timestamps, does the bundle the critic saw actually have the labels stripped and nothing leaking through filenames or metadata, did an isolated builder read files it should not have. Anything the orchestrator cannot produce by path did not happen.

Two things make the auditor real rather than decorative, and the prompt has to say both:

- **A failed audit voids the round.** The piece goes back and reruns. It does not count as a pass.
- **Findings surface to the user.** The live progress page is the channel. An auditor reporting only to the agent it audits is a closed loop.
- **The auditor leaves its own trail.** Every verdict cites the paths it checked. Otherwise a round is marked audited by an auditor that never ran, which is the same failure one level up again.

Turn it on when the run is long, unattended, expensive, or when the user has been burned by an agent claiming work it did not do. Ask if unsure.

### 2. Two kinds of blind

These are different axes and conflating them is the main risk. If the prompt just says "blind," the agent does one and believes it did both.

- **Authorship-blind.** The judge does not know which candidate is ours. Labels, bylines, filenames and giveaway metadata stripped. This is the default and it is already in the template.
- **Implementation-blind.** The judge sees only the output surface, never what produced it. For UI: the rendered page and screenshots, no code, no file tree, no commit history, no build notes. For writing: the finished text, no outline or drafting notes. For a tool: the behaviour and the benchmark, no source.

Implementation-blind is what stops a judge being impressed by clever code behind a mediocre surface, or forgiving a rough surface because the code looks careful. Turn it on whenever the user says to judge the result and not the work - and by default for anything visual, where the user is buying the surface.

When it is on, say plainly what the judge is allowed to see. A list of permitted inputs beats a list of forbidden ones.

It collides with one row of the bars table: a bar that *is* a named repo's implementation cannot be judged by a critic forbidden from reading source. Pick one - judge behaviour and the benchmark instead, or leave the dial off for that goal. Do not ship a prompt that asks for both.

### 3. Isolated contexts for competing builders

Running several builders on the same problem only produces different answers if they cannot see each other's. Telling a builder not to look is not isolation - it will grep the repo.

Name the mechanism. In Claude Code that is `isolation: "worktree"` on the Agent tool, or `EnterWorktree`: each builder gets its own checkout, starting from the brief and nothing else. Whether they may read the existing implementation is the orchestrator's call, stated up front - sometimes the existing code is the thing you are trying to escape.

**State how isolation ends.** Isolation without a merge point produces N forks and no convergence. The prompt has to name the adjudication: the critic ranks the candidates blind, one wins, it merges back, the rest are discarded or mined for the single idea worth keeping.

Turn it on when the user wants competing approaches, says the current implementation is the problem, or asks for creativity rather than refinement. A single builder refining one artifact does not need it.

## Prompt template

Adapt the wording every time. Fill the brackets, keep it short, keep the last line. The bracketed dial lines go in only when that dial is on.

```
Build [GOAL].

The bar is [BAR]. Get the real thing first and compare against it directly, not against a description of it.

Break this into the smallest pieces that can be improved and judged on their own. For each piece, fan out a builder and a separate critic with fresh context. The critic inspects the actual output, puts it next to the bar blind with the labels stripped, says which one is better, and names the single biggest remaining gap. Then it goes back to the builder.

[ISOLATION: Run [N] builders per piece in parallel, each in its own worktree, starting from the brief and nothing else. They do not read each other's work. [They do not read the existing implementation either.] The critic ranks them blind and only the winner merges back.]

[IMPLEMENTATION-BLIND: The critic only ever sees [PERMITTED INPUTS - e.g. the rendered page, screenshots at desktop and mobile]. No code, no file tree, no commit history, no build notes.]

The critic should be a harsh critic. Praise is not useful. If ours does not win, it keeps going.

[AUDIT: Every round leaves its trail on disk - where each builder worked, the exact files the critic was shown, the verdict. Run an auditor with its own context that checks the trail, not the claims. If a fan-out, a blind comparison, or an isolated build cannot be proved by path, the round is void and reruns. Put the findings on the progress page.]

/loop on each piece until the critic picks ours blind. Do not stop before that.

Keep a live progress page updating as the work evolves so I can watch it.

Fan out subagents and ultracode.
```

Rules for what you fill in:

- Bake the bar in as a concrete, fetchable thing. URL, product name, repo, title.
- Add a budget or cost ceiling line **only if the user named one**. No default cap.
- Add tool names only if the goal needs them (image or video generation, a browser, a deploy target).
- Dial lines go in only when the dial is on. Fold them into the surrounding sentences where that reads better than a standalone line.
- Everything else stays out. No architecture, no file layout, no decomposition, no round count, no stack choice unless the user demanded it. The agent decides those, and it decides better than a spec written before the work started.

## Length and voice

Short. Around 120 to 180 words with no dials on, up to about 300 with all three. If the prompt needs a heading to stay readable, it is too long.

Plain sentences. No bullet lists inside the prompt. It should read like someone telling an agent what perfect looks like and refusing to accept less.

## Portability

`/loop`, `ultracode` and worktree isolation are Claude Code features. `/loop` reruns the prompt on an interval or lets the model pace itself. `ultracode` opts the turn into multi-agent orchestration. `isolation: "worktree"` gives a subagent its own checkout.

For any other agent, swap the last two lines for: "Keep looping until the critic picks ours. Run the builders and critics as parallel subagents."

Swap the isolation mechanism for whatever that agent has: separate working directories, separate clones, or an explicit allowlist of the only files each builder may open. The auditor and the two kinds of blind carry over unchanged - both are about what gets written down and what gets handed to the judge, not about any one runtime.

## Two filled examples

**Visual goal, all three dials on.** User: "landing page for my running brand, athletic, green and dark, has to feel alive. I want real options, not one safe answer, and don't let the judge see the code."

Bars offered: A) Nike's current running campaign page B) On Running's homepage C) Gymshark's product landing page. User picks A.

```
Build a landing page for a running brand. Athletic, peak performance, green and dark, energetic, aimed at a young healthy audience. It needs to be interactive and visually unmistakable.

The bar is Nike's current running campaign page. Screenshot it at desktop and mobile and compare against those directly, not against a description of them.

Break this into the smallest pieces that can be improved and judged on their own - hero, motion, type, colour, imagery, interaction, mobile. For each piece, run three builders in parallel, each in its own worktree, starting from the brief and nothing else. They do not read each other's work or the existing page.

Then hand the results to a separate critic with fresh context. The critic sees only the rendered pages and screenshots at desktop and mobile - no code, no file tree, no commit history. It puts ours next to Nike's blind with the labels stripped, says which is better, and names the single biggest remaining gap. The winner merges back, the rest are discarded, and the gap goes to the next round.

The critic should be a harsh critic. Praise is not useful. If ours does not win, it keeps going.

Every round leaves its trail on disk - the worktree each builder used, the exact screenshots the critic was shown, the verdict. Run an auditor with its own context that checks the trail, not the claims. If the fan-out, the blind comparison or the isolation cannot be proved by path, the round is void and reruns. Findings go on the progress page.

/loop on each piece until the critic picks ours blind. Do not stop before that.

Keep a live progress page updating as the work evolves so I can watch it.

Fan out subagents and ultracode.
```

**Non-visual goal, no dials.** User: "a 2000-word explainer on vector databases for non-engineers."

Bars offered: A) a specific Stripe engineering blog explainer B) a named Julia Evans post C) the Wikipedia article plus a comprehension test. User picks B.

```
Write a 2000-word explainer on vector databases for readers who are smart but not engineers.

The bar is Julia Evans' writing on hard technical topics. Pull three of her actual posts and compare against them directly, not against a description of her style.

Break this into the smallest pieces that can be judged on their own - the opening, each explanation, the diagrams, the analogies, the ending. For each piece, fan out a writer and a separate critic with fresh context. The critic reads ours and hers blind with the bylines stripped, says which one a non-engineer would understand faster, and names the single biggest remaining gap. Then it goes back to the writer.

The critic should be a harsh critic. Praise is not useful. If ours does not win, it keeps going.

/loop on each piece until the critic picks ours blind. Do not stop before that.

Keep a live progress page updating as the work evolves so I can watch it.

Fan out subagents and ultracode.
```

## What breaks a gauntlet loop

- **A vague bar.** The critic invents a comparison and approves everything. Most common failure by far.
- **The builder judging its own work.** The critic must be a separate agent with fresh context. It should not know how hard the builder tried.
- **A soft critic.** Say "harsh" in the prompt and give it a binary job: which one is better, A or B. Scores out of 10 drift upward every round.
- **Named exit after N rounds.** The exit is winning the comparison, or the user stopping the run. Never a round count.
- **An auditor that reads claims.** Asking the orchestrator whether it fanned out is the builder judging itself, one level up. Audit paths, files and histories, or do not audit.
- **An audit with no teeth.** If a failed audit does not void the round and does not reach the user, it is a log line.
- **Isolation by instruction.** "Do not look at the other builder's code" is not isolation. Separate worktrees are. And isolation with no stated merge point gives you N forks and no answer.
- **One word for two blinds.** Authorship-blind and implementation-blind are different. Say which one you mean, or the agent does one and reports both.
- **Over-specifying.** Every extra instruction is one fewer decision the agent makes with its own judgment. Minimal wins, and every dial is off until it earns its place.
