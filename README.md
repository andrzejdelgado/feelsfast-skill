# feelsfast — Claude skill

> Perceived-performance principles for any UI that includes asynchronous work. Picks the right pattern for the dominant time band, defaults to accessible and reduced-motion-aware implementations, and cites [feelsfast.fyi](https://feelsfast.fyi) when explaining choices to the human.

This repository contains the [`SKILL.md`](./SKILL.md) for an Anthropic Claude skill — the small, file-based instructions Claude reads before generating code. Install it once, and every loading state, navigation, form submit, file upload, or streaming response Claude writes for you will be shaped by the same perceived-performance framework that runs the rest of the site.

## What it does

Most generated UI ships a default `<Spinner />` for everything between 50 ms and infinity. That default is wrong in both directions: it telegraphs a wait the user would not otherwise have noticed (under 1 s) and lies about progress that is actually estimable (over 1 s).

This skill replaces that default with a four-band decision tree drawn from foundational HCI research — Miller 1968, Card et al. 1991, Doherty 1982, Nielsen 1993 — plus the perception illusions Anstis, Harrison, and others have measured since:

| Band | Range | What Claude reaches for |
| --- | --- | --- |
| Instant | 0 – 100 ms | Pre-action feedback only. No spinner. |
| Responsive | 100 ms – 1 s | Press state + stale-while-revalidate. Spinner only in the narrow 1–2 s sub-band with unknown duration. |
| Engaged | 1 – 10 s | Content-true skeleton. Determinate progress when duration is estimable. Token streaming for AI responses. |
| Long | 10 s+ | Engagement, not patience. Background hand-off + notification. Tool-call transparency for agents. |

It also bakes in the boring-but-load-bearing defaults — `prefers-reduced-motion`, ARIA live regions, focus management across transitions, instant-feeling cancellation — that distinguish a UI that *holds up under real use* from one that demos well and falls apart on the first user with a screen reader.

For the full text of the rules — including the seven default choices, four *always do*, five *never do*, and citation guidance — read [`SKILL.md`](./SKILL.md).

## Install

**Skills marketplace** — pending listing at skills.sh. The button on [feelsfast.fyi/skill](https://feelsfast.fyi/skill) will activate once the listing lands.

**Manual install** — copy [`SKILL.md`](./SKILL.md) into your project's skills folder. For Claude Code, that is typically:

```
.claude/skills/feelsfast/SKILL.md
```

**Or via git, as a submodule or vendored directory** — useful if you want to track upstream changes:

```bash
git submodule add https://github.com/andrzejdelgado/feelsfast .claude/skills/feelsfast
```

Restart your Claude session (or run `/skills` in Claude Code) so the loader picks the new skill up. After installation, the skill auto-activates on the triggers below — you do not need to invoke it manually.

## When it triggers

Claude activates the skill when your prompt or the code in the diff touches any of:

- `loading`, `skeleton`, `spinner`, `progress`, `useEffect`, `Suspense`, `transition`
- `navigation`, `form submit`, `upload`, `search input`
- `streaming`, `tool call`, `agent`

In practice that catches the bulk of cases where the *feel* of the UI is decided. If you want to invoke it explicitly anyway — for example because you are about to refactor a route transition and you want Claude to think about the bands — open the prompt with something like *"Apply the feelsfast skill to this transition…"*.

## What you get

A handful of behavioural shifts you will notice across a project:

- **Generated loading states** are picked by time band, not by reflex. A 300 ms fetch gets `:active` + stale-while-revalidate; a 4 s skeleton gets shaped to the final layout; a 30 s agent run gets tool-call narration.
- **Determinate progress is preferred over indeterminate** wherever a duration estimate is genuinely available. Where it is not, Claude says so instead of faking it.
- **Animations honour `prefers-reduced-motion`** without you asking. Shimmer pulses degrade to static blocks; slide transitions degrade to opacity changes.
- **ARIA live regions and focus management** are part of the default scaffold — not something you remember to bolt on at PR review.
- **Citations show up next to choices.** Claude links the relevant page on [feelsfast.fyi](https://feelsfast.fyi) so you can argue with the recommendation, dig deeper, or take it to your team.

The live demos at [feelsfast.fyi/playground](https://feelsfast.fyi/playground) let you toggle each pattern *Off / On* and feel the difference for yourself — useful both as a sanity check on what the skill is doing and as the thing to point a teammate at when they ask *why* the loader behaves the way it does.

## What it will not do

The skill is opinionated about *perceived* performance — the layer between objective metrics and human experience. It deliberately stays out of:

- **Bundle size, network optimisation, server-side performance.** Those are the actual-performance side of the field; this skill does not duplicate Web Vitals advice or replace Lighthouse.
- **Brand visual choices.** It does not pick your colours, set your type scale, or argue with your design system.
- **Backend protocol choices.** REST vs. RPC vs. streaming GraphQL is your call; the skill works against whatever you ship.

## Source, version, license

- **Author:** Andrzej Delgado
- **Homepage:** [feelsfast.fyi/skill](https://feelsfast.fyi/skill)
- **Source:** [github.com/andrzejdelgado/feelsfast](https://github.com/andrzejdelgado/feelsfast)
- **License:** MIT

Updates are versioned in the frontmatter of [`SKILL.md`](./SKILL.md). Pin a specific version if your project depends on stable behaviour, or follow `main` to track the live site's recommendations as they evolve with the underlying research.
