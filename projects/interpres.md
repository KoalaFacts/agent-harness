# Interpres — project rules

Owner-controlled seed for the "Interpres 推进计划" scheduled task (repo KoalaFacts/Interpres). The repo skill is `.claude/skills/advance-interpres/SKILL.md`; if it is missing on main, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

## Docs files in this repo
The ledger is a "Roadmap" section of README.md (create it there; no separate roadmap file). Engine test results already in README.md are an evidence record: keep them, and only add rows when the owner runs a new live test. Design decisions that need writing down go in docs/decisions/, one short record per decision.

## Verification boundary
The real boundary is the lecture pipeline driven end to end against local stand-ins: the CLI streaming a fixture WAV into a mock Soniox STT + translation WebSocket and a mock Soniox TTS WebSocket (extend `tools/dev/mock-server.ts`), the Worker and Durable Object under Miniflare with local D1, the caption page and the student listening page in headless Chromium. Never call the real Soniox, OpenAI, Azure or Cloudflare APIs: every live call costs the owner money and is the owner's to run (the `tools/` scripts exist for that).

## Project

### P1 WHAT AND WHY
- AI live interpretation for an in-person medical lecture in Australia. The speaker talks Mandarin, often switching into English terms or whole English passages; listeners are Mandarin, Cantonese and English speaking students.
- Output: projector captions (Traditional Chinese line over an English line, via an OBS browser source with a transparent background) and English speech that students hear on their own phones after scanning a QR code and entering a personal code.
- Priorities, in order: accuracy and no dropouts; zero data retention and low latency; everything else is nice to have.
- Decided engine (owner, 2026-11-13): pure Soniox — real-time STT + one-way translation `stt-rt-v6` with medical context and `translation_terms`, default endpointing, and translated text sent to Soniox TTS `tts-rt-v2` as one short stream per clause at speed 1.25 with `reduce_silence`. Azure Speech is the captions-only fallback. OpenAI gpt-realtime-translate was rejected (wrong drug names, silence on English). Faster endpointing was rejected (number and unit errors).
- Hosting: Cloudflare only (Workers, Pages, D1, Durable Objects, Realtime SFU); no AWS.

### P2 AUTHORITY SPECIFICS
- Branch prefixes `feat/`, `fix/`, `test/`, `docs/`, `refactor/`, `ci/`, `tools/`; squash merge.
- Never run `wrangler deploy`, `wrangler d1 ... --remote` or anything else against a remote Cloudflare account; never create Cloudflare, Soniox, Azure or OpenAI resources or keys.
- Every open PR in the repo (the owner's, Codex, Dependabot, other sessions) is in scope for open-PR triage (autonomous-dev §3 step 0). Codex review comments are evidence to verify, not orders.

### P3 SETUP AND GATE
- Node 24+, TypeScript only: no .mjs/.cjs, no bash, Python or PowerShell scripts; scripts are `.ts` run directly by Node. Code, comments, test names and commit messages in English.
- LOCAL GATE: `npm ci`, `npm run typecheck`, the unit tests, the Worker tests under Miniflare, the browser tests in headless Chromium, and the pipeline run against the mock servers. The gate is wired into one `npm run verify` script and a GitHub Actions workflow running the same steps (no secrets, no deploy).

### P4 LEDGER AND ORDER
- Ledger: README.md "Roadmap" section.
- Initial order:
  P1 Monorepo skeleton: npm workspaces `packages/cli`, `packages/worker`, `packages/web`, shared types package if needed; `npm run verify`; CI workflow; move `tools/` into the workspace without changing what the scripts measure.
  P2 Worker: D1 schema (lectures, students with a hash of the personal code only), `/api/login` (code in, short-lived token out, the code alone identifies the lecture), rate limit on login, `LectureRoom` Durable Object per lecture holding student → active session, kicking the older device with a `revoked` message, broadcasting caption messages to listeners; tests under Miniflare.
  P3 CLI engine: Soniox adapter (STT + translation + per-clause TTS as decided in P1), glossary file, OpenCC `cn` → `hk` conversion (character conversion only) with a custom dictionary hook, caption messages `{ type: "caption", id, zh, en, final }`, partial captions replaced by finals, speech only from finals.
  P4 No-dropout layer: language state machine (whole English ≥ 1.5 s → original-audio passthrough, back to translation after ≥ 1 s of Mandarin, hysteresis), passthrough queued behind any translated speech still playing with a 150–300 ms crossfade, silence watchdog (speaker talking and no English speech for 4 s → passthrough), automatic reconnect plus a warm standby STT connection swapped at a pause, captions-only fallback to Azure when Soniox fails. Each behaviour proven against the mock with a scripted failure.
  P5 Caption page (two lines, Traditional Chinese over English, large type, transparent background) and student listening page (code entry, "start listening" button for iOS Safari, Screen Wake Lock, channel choice English speech / Mandarin original, Traditional Chinese caption toggle); browser tests.
  P6 Audio fan-out design for Cloudflare Realtime SFU: written as a decision record first, then the client and server code tested against a local stand-in; any real SFU session is the owner's to run.
  P7 Number, dose and negation guard: compare numbers, dose units (mg, mcg, µg, g, mL, IU, mmol/L) and negations between source and translation, and flag mismatches on the captions; tests from the README's known cases.
  P8 `interpres` CLI commands `init`, `deploy` (prints the wrangler steps for the owner, runs nothing remote), `students import`, `doctor`, `start`, with secrets in the OS keychain or environment, never in files in the repo.

### P5 PROJECT RULES
- Extra merge condition — accuracy: a change that touches the engine, the state machine or the guard must keep every fixture expectation for numbers, doses, negations and English passages passing; a change that would make captions or speech skip content is not merged.
- Privacy: no audio, transcript or caption is written to disk or D1 by the product; the Durable Object may keep the last few captions in memory only. Never commit recordings, reports, keys or `.env` files.
- Interfaces carry no redundant parameters (owner rule): a value derivable from another argument is not passed separately.
- Never put the owner's personal name in code, docs, commit metadata or pages; sign as Koala Facts.

### P6 HOUSEKEEPING SCOPE
- README.md and docs/**: keep the setup, commands and roadmap true to the code; fold superseded notes into their owning page and delete them.

### P7 REPORT EXTRAS
- Which mock scenarios ran (code switch, dropout, reconnect, fallback) and their results; anything the owner must run live next, with the exact command.
