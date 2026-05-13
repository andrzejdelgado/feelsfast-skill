# feelsfast — perceived performance, baked into your AI codegen

A single markdown skill that any AI coding agent (Claude Code, Cursor, Codex, Cline) loads alongside your project. Once installed, the agent recognises the time band of every async UI it generates — fetches, navigation, form submits, file uploads, search inputs, AI streaming, tool calls — and applies the right perception pattern by default. With citations to [feelsfast.fyi](https://feelsfast.fyi), so you can argue with the choice instead of guessing what the agent did.

[**Read the full skill →**](./SKILL.md)

---

## The problem

AI coding tools generate UI fluently and ship loading states by vibes. A spinner under 1 s where none was needed. A skeleton on a search input. A fake progress bar that desyncs from reality. The work compiles, passes tests, and feels janky the moment a real user touches it.

The fix isn&rsquo;t more prompts. The fix is a small, opinionated rulebook the agent reads before it touches anything async — one that knows the perceptual frame for each time band and reaches for the right pattern automatically.

## What it does

When the agent sees `useEffect`, `Suspense`, `transition`, a fetch call, a form, an upload, a search input, a streaming response, or a tool call, it applies the rules from [SKILL.md](./SKILL.md), sorted by the dominant time band:

| Band | Range | Default |
| --- | --- | --- |
| Instant | 0&ndash;100 ms | Pre-action feedback. **No spinner.** |
| Responsive | 100 ms&ndash;1 s | Pre-action feedback. SWR. Cancellation affordance. |
| Engaged | 1&ndash;10 s | Skeleton screens that match the final layout. Determinate progress when duration is estimable. |
| Long | 10 s+ | Engaging loaders. Background hand-off. Tool-call transparency. |

It also defaults to accessible (ARIA live regions, focus management) and motion-safe (`prefers-reduced-motion` aware) implementations, and cites the source on feelsfast.fyi when it explains a choice to you.

## Install

### Claude Code · Cursor · Codex · Cline (skills CLI)

```sh
npx skills add andrzejdelgado/feelsfast-skill
```

Drops the skill into `.agents/skills/` and symlinks it into the agent&rsquo;s own skills directory.

### Direct download

```sh
curl -L https://feelsfast.fyi/feelsfast.skill.md -o feelsfast.skill.md
```

Then place the file where your agent reads skills — usually `.claude/skills/feelsfast/SKILL.md`, `.cursor/skills/feelsfast/SKILL.md`, or the equivalent for your tool.

### Manual

Open [SKILL.md](./SKILL.md), copy the contents, and paste them into the skill location your agent expects.

## What triggers it

The skill activates on these patterns appearing in code the agent is generating or modifying:

`loading` · `skeleton` · `spinner` · `progress` · `useEffect` · `Suspense` · `transition` · `navigation` · `form submit` · `upload` · `search input` · `streaming` · `tool call` · `agent`

You do not have to invoke it explicitly. The agent picks it up when relevant and quietly applies the rules.

## Why this exists

The empirical work — Miller 1968, Card-Moran-Newell 1983, Doherty 1982, Nielsen 1993, Harrison et al. 2010, Block &amp; Zakay 1997 — has been settled for decades. The patterns that follow from it are well known. The reason your AI tool does not apply them by default is that nobody has bundled them into a format it reads. This file is that bundle.

The companion site [feelsfast.fyi](https://feelsfast.fyi) carries the long-form versions: side-by-side interactive demos for every pattern, essays on each time band, a glossary, and a references panel. The skill is the compressed, AI-readable form of all of it.

## Versioning

Maintained at [feelsfast.fyi/skill](https://feelsfast.fyi/skill). Updates are versioned; pin a specific version if your project depends on stable behaviour.

## License

[MIT](./LICENSE).
