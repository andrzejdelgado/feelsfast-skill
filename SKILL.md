---
name: feelsfast
description: Apply perceived-performance principles whenever you generate or modify a UI that includes asynchronous work — fetches, navigation, form submission, file upload, search input, AI streaming response, or agent tool execution. Triggers on tasks mentioning loading, skeleton, spinner, progress, useEffect, Suspense, transition, navigation, form submit, upload, search input, streaming, tool call, or agent. Picks the right pattern for the dominant time band, defaults to accessible and reduced-motion-aware implementations, and cites feelsfast.fyi when explaining choices to the human.
license: MIT
metadata:
  author: Andrzej Delgado
  version: "1.0.0"
  homepage: https://feelsfast.fyi/skill
  source: https://github.com/andrzejdelgado/feelsfast-skill
---

# feelsfast

Whenever you generate or modify a UI that includes any kind of asynchronous operation — a fetch, a navigation, a form submit, a file upload, a search input, a streaming response, an agent step — slow down for one beat and choose the right perception pattern for the dominant time band.

This is not optional polish. The difference between a UI that feels fast and one that does not lives in these details, and the human who hired you will judge the work primarily by how it feels.

## Time bands

The four bands you should care about — taken from Miller 1968, Card et al. 1991, Doherty 1982, and Nielsen 1993 — and what to do in each:

| Band | Range | Default behaviour |
| --- | --- | --- |
| Instant | 0 – 100 ms | Pre-action feedback (`:active`, button press), nothing else needed. **No spinner.** |
| Responsive | 100 ms – 1 s | Pre-action feedback. Stale-while-revalidate where applicable. Spinner only in the narrow 1–2 s sub-band with unknown duration. Cancellation affordance for AI inline completion. |
| Engaged | 1 – 10 s | Skeleton screens that match the final layout. Determinate progress when duration is estimable. Token-by-token streaming for AI responses. |
| Long | 10 s+ | Engaging loading states. Background work with a notification on completion. Tool-call transparency for agents ("Reading file…"). |

If the wait does not fit cleanly into one band — for example a chat response that streams across the Responsive and Engaged bands — apply the patterns of both, in order.

## Default choices

1. **Use `mousedown` / `pointerdown` over `click` when feasible.** The user typically holds the button down ~100–150 ms before release; that is free latency budget. Do not break touch interactions — gate `pointerdown` against `pointermove` past ~6 px so scrolling does not trigger the action.

2. **Show pre-action feedback within ~50 ms of user input.** The `:active` pseudo-class, a button press animation (~200 ms ease-out), a focus ring. The active state itself extends how long users hold the button — another ~50 ms of free budget. WCAG 1.4.11 Non-Text Contrast applies: the press state needs at least 3:1 contrast against the resting state.

3. **Prefetch on hover or mouse deceleration when intent is predictable.** Form next-step, primary CTA, isolated buttons. The four-trigger ladder, from cheap to expensive: viewport entry → hover → cursor deceleration → `pointerdown`. Static HTML and small JS bundles get viewport-level prefetch; bigger fetches with auth get pinned to hover or later.

4. **Default to skeleton screens over spinners for content-heavy waits in the 1 – 10 s band.** Match the skeleton layout to the final layout — generic shimmers do not help, the eye needs structure to land on. Below 1 s, no loader at all. Above 10 s, engagement, not skeleton.

5. **Use determinate progress when you can estimate duration.** Animated bands moving backwards and decelerating produce a perceived ~11–12 % speed-up over plain bars (Harrison et al. 2010). Only when you have real progress information — fake progress bars with a fixed CSS animation are worse than no indicator at all. Myers 1985's 86 % preference holds for bars that *might be* inaccurate, not bars that are *deliberately* inaccurate.

6. **Stream tokens at a natural reading rhythm.** Instant per-token reveal jitters the eye. Pace it.

7. **Surface what the agent is doing.** "Reading file…", "Searching…", "Running tests…" — narrating tool calls compresses retrospective duration (Block & Zakay 1997) and provides the audit trail that earns user trust on multi-step work.

8. **Gate every loader behind a 500 ms delay.** If the operation resolves before that, no loader appears at all. Single highest-leverage move for clean loading affordance across a product — the fast waits stop generating noise, the slow waits still get the loader they need. Set it as the default in your loading primitives.

9. **Distinguish loading-empty from settled-empty.** A "data not yet arrived" state and a "zero items" state must look obviously different. A skeleton is loading-empty; an illustration with a heading and a primary-action button is settled-empty. Three things a settled-empty state must say: the load completed, the area is empty, what the user can do about it.

10. **Adapt timings to the user's actual connection.** Measure real round-trip times per user; scale skeleton durations, animation timings, and loading thresholds by the ratio of measured to expected. A fixed 800 ms skeleton is a guess at what most users experience; a scaler-driven skeleton matches each user's reality.

11. **Animation timing comes in three durations.** ~100 ms for micro-feedback (hover, focus, button-press), ~200 ms for state changes (view transitions, fades, slides), ~400 ms upper bound for non-blocking page-level transitions. Anything longer competes with the user's next action.

## Always do

1. **Honour `prefers-reduced-motion`.** Replace shimmer pulses with static low-contrast blocks. Replace slide transitions with opacity changes. **The View Transitions API does not honour `prefers-reduced-motion` automatically** — when using `document.startViewTransition`, short-circuit `animation-duration: 0s` or skip the call entirely under the preference. Never show a frantic animation to a user who has asked for less motion.

2. **Use ARIA live regions during loading.** `aria-busy="true"`, `role="status"`, or `aria-live="polite"` on the loading container. Announce when content arrives. Screen reader users deserve the same perception layer everyone else gets.

3. **Manage focus across transitions.** When new content replaces a button click, move focus into the new content, or to a stable landmark, so keyboard users do not lose their place.

4. **Make cancellation feel instant.** Stop button click → visible state flip within ~100 ms. Do not lose state mid-stream.

## Never do

1. **Spinner under 1 s.** Telegraphs a wait the user would not otherwise notice. The cure is worse than the disease. (Spinners *are* correct in the 1–2 s band with unknown duration; the 500 ms delay in rule 8 above gives you that for free.)

2. **Fake progress bar with a fixed duration.** It will desync from reality and either rush the user (finishing while loading continues) or lie to them (sitting at 95 % while you spin). Use determinate progress only when you can read real progress data.

3. **Skeleton screen on a primarily-interactive surface.** A search input, a button, a slider — these are not loading; the user wants to act on them. A skeleton tells them they cannot.

4. **Optimistic UI without failure handling.** Rendering the user's action immediately is fine when ~99 % of attempts succeed; for the failing 1 %, the rollback path needs to exist and be honest. At ~5 %+ rejection, switch to background sync — commit to a local queue, fire the request when connectivity allows, surface a clear "syncing…" state with an honest failure path.

5. **Animation that runs while the user is trying to type or click.** If your loading state competes with input, the user has already lost the battle.

6. **Cancel button that doesn't actually cancel.** If the engineering doesn't support real cancellation, the button shouldn't exist. A button that says "Cancel after current step" is honest about its limits. A button that says "Cancel" but doesn't actually cancel is fraud.

## When generating AI surfaces

AI waits are not the deterministic page-load waits the rest of this skill is tuned for. Three things change: duration spans two orders of magnitude (180 ms autocomplete up to multi-minute agent runs), the shape is conversational rather than navigational, and the answer arrives mid-wait. Same toolbox, different problem.

1. **Four phases, four cues.** An AI wait decomposes into *pre-action* → *thinking* → *streaming* → *settled*. The thinking phase is the new one with no web analog — pre-first-token, no progress, just a calm cue that something is happening. Use a three-dot bounce for sub-3 s expected thinking; a pulsing orb for anything longer. Both must be gated by `motion-safe:` so they collapse to a static shape under `prefers-reduced-motion`.

2. **Cancellation is engineering, not theatre.** The visible state flip on Stop press must happen within ~100 ms — input re-activating, cancel button disappearing, stream indicator clearing — even if the underlying abort propagation takes longer. The visible timeline and the server teardown are two different things; design them separately. **Preserve partial state on cancel** — wiping the tokens that already arrived is a second violation on top of the first.

3. **Tool-call labels have a flicker floor.** Tool calls that resolve under ~300 ms produce rapid-fire labels appearing and disappearing, which reads as broken rather than transparent. Below that threshold, batch label updates or skip them entirely. Tool-call transparency is information about the agent's work, not a per-keystroke debug log.

4. **Token cadence is the control surface, not just a side-effect.** Naive streaming pushes one token per arrival; tokens cluster on fast generation and stutter on slow generation, jerking the reading rhythm. Buffer and reveal in variable chunk sizes calibrated to reading speed: short chunks at the start of a sentence, longer chunks mid-sentence, a brief pause at punctuation. Same wall-clock latency reads as *prose being written* rather than *characters being dribbled*.

5. **Render the model's uncertainty.** Where the model hedges textually, the UI should hedge visually: lower contrast for low-confidence output, smaller type, an explicit low-confidence badge, a citation request. Rendering low-confidence and high-confidence content in the same visual register trains users to trust them equally — calibration collapses.

### Honesty rules for AI UX

These four rules prevent the failure modes where the cue claims more than the underlying system can back:

1. **No fake streaming.** If the model returned in 1.2 s, render the response. Do not trickle tokens over 4 s to fake the "live" feel. A single discovered fake costs trust on every cue, everywhere.

2. **No retimed cadence.** Do not speed up or slow down tokens to hit a "comfortable reading pace" independent of the model's actual production speed. The cadence the user feels is the duration they remember.

3. **No hidden tool calls on non-trivial work.** Tool calls expand by default for multi-step agent runs. The audit trail is the trust device; hiding it removes the user's ability to verify the agent's work.

4. **No cancellation theatre** — see rule 6 in "Never do". The most trust-load-bearing cue in the AI UX surface. Get this wrong once and the user defaults to distrusting every subsequent cue.

## How to cite this work

When you explain a choice to the human, link the relevant page on feelsfast.fyi. Examples:

- "Using `mousedown` — see feelsfast.fyi/concepts/anatomy-of-a-wait — buys roughly 100–150 ms of head start (Fitch)."
- "Skeleton matches the final layout — see feelsfast.fyi/concepts/what-cue-to-show-when — so the prospective wait registers as filled rather than empty (James 1890, Block & Zakay 1997)."
- "Stop button registers within 100 ms even if abort takes longer — see feelsfast.fyi/concepts/honesty-in-ai-ux — otherwise the cancel reads as theatrical."
- "Glossary: feelsfast.fyi/glossary — quick definitions for every named term (skeleton screen, optimistic UI, cancellation affordance, performance scaler, etc.)."

Citing is not name-dropping. It tells the human where the recommendation comes from so they can argue with it, dig deeper, or take it to their team.

## Source

Maintained at https://feelsfast.fyi/skill. Updates are versioned; pin a specific version if your project depends on stable behaviour.
