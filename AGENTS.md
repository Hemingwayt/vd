# AGENTS.md — Venice Drive

Voice-first AI companion for a parked car. Single-file client on GitHub Pages,
private brain server on an Android phone. Read all of this before editing.

## What this is

Two halves that must not leak into each other:

- `index.html` — public, ~381 lines, no build step, served by Pages.
- `brain/` — private, Python stdlib + SQLite, runs on a phone on a hotspot.
  Holds the Venice key and the memory. Never published.

The moat is accumulated memory, not the code. Code is copyable.

## Hard constraints (verified, do not re-litigate)

- Tesla browser mic is PARKED-ONLY. No driving voice. Do not promise it.
- Tesla browser has no Bluetooth, no install, no sideload. Chromium 148.
- Pixel-density changes break the car layout. Do not restyle the shell.
- CarPlay needs an Apple entitlement. No wake word, no tool use, no car control.
- Venice API does not retain prompts. Do not add logging that contradicts this.

## Absolute rules

1. No dependencies. No npm, no bundler, no framework, no CDN script, no webfont.
2. `index.html` stays one file. If a change needs a second file, say so first.
3. All model/user text goes through `textContent`. `innerHTML` only for the
   fixed literal icon glyphs already there.
4. The Venice key never enters the browser in proxy mode. Brain URL + access
   token only. BYO-key is the fallback tier and stays optional.
5. `brain.db`, `venice.key`, `*.log` are never committed.
6. No analytics, no telemetry, no third-party requests.
7. ES5-compatible syntax in `index.html`. No arrow functions, no `let`/`const`,
   no template literals. Match the existing file.
8. Comments only where the reason isn't obvious. No comment blocks narrating
   what the code plainly says.

## Do not touch without asking

- `brain/schema.sql` (memory schema — migration cost, and it's the product)
- The VAD thresholds in `poll()` (0.022 / 900 / 450 / 15000). Tuned by ear.
- The `#hd` / `#ft` / `#bar` geometry.
- `venice_parameters` in the chat payload.
- Anything under `.github/`.

## Brain contract

Endpoints, all bearer-auth except `/health`:

    GET  /health
    GET  /v1/models
    GET  /v1/memory
    POST /v1/memory                 {text, tag?}
    DELETE /v1/memory/:id
    POST /v1/chat/completions       {model, messages[]}
    POST /v1/audio/speech
    POST /v1/audio/transcriptions

`/v1/chat/completions` injects memory server-side and streams SSE. The client
never sends memory. 401 on bad token, 429 on daily cap.

CORS is an exact-origin allowlist (`BRAIN_ORIGIN`), never `*`.

CSP `connect-src` must list the brain origin and `https://api.venice.ai`.
If you add a host, add it there or the request dies silently.

## Branch and PR convention

- One branch per task: `w1-proxy-auth`, `w2-memory-extract`, `w3-provider`.
- Never commit to `main`. Never merge your own PR without a review.
- Commit subject: imperative, ≤60 chars, no prefix theatre.
- PR body: what changed, what you tested, what you couldn't test, risk.

## Definition of done

- Loads from `file://` with no console errors.
- Renders at 1280x800 and 1920x1200 without overflow or clipping.
- Streaming still renders incrementally; mic flow still starts and stops clean.
- No new external request appears in the network panel.
- You stated which of the above you actually verified versus assumed.

## Escalate to the human (don't guess)

Apple entitlement, Stripe, pricing, copy on the landing page, anything that
changes the pricing tiers, anything that needs a physical car or phone.

## Log

Append one line per decision. Date, decision, why. Never rewrite history.

- 2026-11-15 — brain = old Android, not a Pi. Qi, mic, speaker, hotspot in one.
- 2026-11-15 — key lives on the brain; browser gets a scoped token.
- 2026-11-15 — memory injected server-side so the client stays stateless.
- 2026-11-15 — parked-only voice accepted; it's the constraint, not a bug.
