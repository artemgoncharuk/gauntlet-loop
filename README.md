<p align="center">
  <img src="assets/banner.png" alt="gauntlet loop" width="100%">
</p>

# Gauntlet Loop

A skill that turns any goal into one short, paste-ready prompt. That prompt makes your agent pick a real quality bar, split the work into small pieces, run a builder and a separate harsh critic on each one, compare blind against the bar, and keep looping until it wins.

Most agent output stops at "good enough" because nothing is holding it to a standard. This gives it a standard it cannot argue with - and, when you ask for them, an auditor that checks the loop is not faking its own compliance, a judge kept blind to the implementation, and isolated builders that cannot copy each other.

> The gauntlet loop is [Matt Shumer's](https://github.com/mshumer) idea. He wrote the original prompt and named the technique while building [Claude of Duty](https://github.com/mshumer/Claude-of-Duty). This repo packages that pattern as a reusable skill.

## Quick start

```
git clone https://github.com/robonuggets/gauntlet-loop
```

Copy the skill folder into your project:

```
cp -r gauntlet-loop/.claude/skills/gauntlet-loop your-project/.claude/skills/
```

Then in your agent:

```
/gauntlet-loop build me a pricing page for my SaaS
```

It offers you 2 or 3 quality bars to aim at, you pick one, and it hands back a single prompt you paste into a fresh session.

## What's included

```
.claude/skills/gauntlet-loop/
└── SKILL.md      # the whole skill, one file
README.md
LICENSE           # CC BY 4.0
```

## How it works

1. **You give a goal.** Anything. A site, an essay, a CLI tool, a research brief.
2. **It offers 2 or 3 bars.** Each one is a specific, real thing your agent can actually fetch and compare against. Not "award-winning design", but a named page, a named post, a named repo.
3. **You pick one.** It also turns on any of the three dials the goal calls for - audit, implementation-blind judging, isolated builders - then writes one short prompt, around 150 words, and stops.
4. **You paste it into a fresh session.** That agent splits the work, runs builder and critic pairs, and loops.

The critic is the part that matters. It is a separate agent with fresh context, it opens the actual output, it puts your work next to the bar with the labels stripped, and it says which one is better. Not a score out of 10, which drifts upward every round. A pick.

The loop exits when your work wins the blind comparison, or when you stop the run. Never after a fixed number of rounds.

## Why a bar and not a rubric

A rubric asks the agent to grade itself against words it wrote. A bar makes it compare against something that already exists and is undeniably good.

The skill will not accept a vague bar. It checks three things before it writes anything:

- **Named.** A specific thing, not a category.
- **Fetchable.** The critic can screenshot it, read it, run it, or open it. If the agent cannot get the reference, it hallucinates the comparison and approves everything.
- **Comparable.** Both can sit side by side and a judge can pick one.

## The three dials

All three are off by default. Every extra instruction is one fewer decision the agent makes with its own judgment, so each has to earn its place. The skill turns one on when the goal calls for it, or asks you.

**The auditor.** An agent that says it fanned out subagents and compared blind is grading its own compliance. So the loop is required to leave a trail on disk - the worktree each builder used, the exact files the critic was handed, the verdict - and a separate auditor checks the trail, not the claims. Anything that cannot be proved by path did not happen: the round is void, it reruns, and the finding goes on the progress page where you can see it.

**Two kinds of blind.** *Authorship-blind* means the judge does not know which candidate is yours; that is the default. *Implementation-blind* means the judge only ever sees the output surface - the rendered page, the finished text, the tool's behaviour - and never the code, file tree or commit history behind it. Otherwise a judge gets impressed by clever code behind a mediocre surface. Say which one you mean, or the agent does one and reports both. The one place it does not fit is a bar that *is* someone's source code - then you judge behaviour and the benchmark, or leave the dial off.

**Isolated builders.** If you want competing solutions rather than one refined answer, the builders have to actually be unable to see each other's work. Telling them not to look is not isolation - they will grep the repo. Each gets its own worktree, starting from the brief and nothing else, and whether they may read the existing implementation is stated up front. The critic then ranks them blind, the winner merges back, the rest are discarded.

## Examples

```
/gauntlet-loop a landing page for my running brand, dark and green, has to feel alive
```
Bar becomes a specific brand's live campaign page, screenshotted at desktop and mobile.

```
/gauntlet-loop a 2000 word explainer on vector databases for non-engineers
```
Bar becomes a named writer's actual published posts, judged on which one a non-engineer understands faster.

```
/gauntlet-loop a CLI that formats JSON logs
```
Bar becomes a named tool's implementation plus its benchmark, so taste and a number both have to win.

## Works with any agent

`/loop`, `ultracode` and worktree isolation are Claude Code features. `/loop` reruns a prompt until you stop it, `ultracode` opts a turn into multi-agent orchestration, and worktree isolation gives each subagent its own checkout.

For any other agent, the skill swaps those lines for plain instructions: keep looping until the critic picks ours, run the builders and critics as parallel subagents, and isolate them with whatever that runtime has - separate directories, separate clones, or an explicit allowlist of the only files each builder may open. The auditor and both kinds of blind carry over unchanged, because they are about what gets written down and what gets handed to the judge, not about any one runtime.

## What breaks it

- A vague bar. The critic invents a comparison and approves everything. By far the most common failure.
- The builder judging its own work. The critic needs fresh context and no knowledge of how hard the builder tried.
- A soft critic. Give it a binary job, not a score.
- A fixed round count. The exit is winning, or you calling it.
- An auditor that reads claims instead of paths. Asking the orchestrator whether it fanned out is the builder judging itself, one level up.
- An audit with no teeth. If a failed round does not rerun and does not reach you, it is a log line.
- Isolation by instruction. "Do not look at the other builder's code" is not isolation. Separate worktrees are - and isolation with no stated merge point leaves you with forks and no answer.

## Credit

The gauntlet loop technique is **[Matt Shumer's](https://github.com/mshumer)**. He built [Claude of Duty](https://github.com/mshumer/Claude-of-Duty), wrote the [original prompt](https://github.com/mshumer/Claude-of-Duty/blob/main/prompt.md), and named the loop. Every idea underneath this skill - the harsh critic, the blind comparison, the refusal to stop until the work wins - comes from that prompt.

This repo is not the technique. It is a skill that writes a gauntlet loop prompt for you, for any goal, so you do not have to hand-write one each time.

Related reading: [Anthropic on building effective agents](https://www.anthropic.com/engineering/building-effective-agents), which covers the evaluator pattern the loop is built on.

## License

CC BY 4.0. Free to use with attribution.

Skill by Jay E at [RoboNuggets](https://robonuggets.com). Technique by Matt Shumer.
