# Yujin Pilot -- Commercial plan + execution backlog

**Status:** decided 2026-05-11. Not executed yet.
**Sibling repo:** `yujin-forge` ships under the same plan; see
that repo's `docs/COMMERCIAL_PLAN.md` for the supply side.

This document is the source of truth for the business model,
pricing, packaging, and execution backlog of Yujin Pilot. Any
change must update this file in the same commit.

---

## The model in one paragraph

Yujin Pilot is a low-friction controller ($5/mo standalone, FREE
when bundled with Forge) for NAC3-compliant apps. Voice + chat +
keyboard dispatch across multiple apps. BYOK on the AI side.
Pilot's reason to exist is the NAC3 ecosystem -- without
NAC3-compliant apps to control, Pilot is decorative.

That is why Pilot ships alongside Forge, not before. Forge
generates the supply of NAC3-compliant apps; Pilot consumes that
supply. Forge carries the revenue load. Pilot is the moat
enforcer (every Forge-paying dev installs Pilot for free,
generating organic viral exposure when their colleagues see it
running) and the public-facing demo of "look what NAC3 apps do".

---

## Pricing tiers

| Tier | Price | Trial | Support | What you get |
|------|-------|-------|---------|--------------|
| Pilot Free | $0 | -- | GitHub issues only | Basic NAC3 spec control via reference `nac-chat-client.js`. No registry browser, no cross-app routing, no voice premium voices, no mobile companion |
| Pilot Pro standalone | $5/mo | 30 days | GitHub issues only | Registry browser, cross-app routing, premium TTS voices, mobile companion, multi-device sync |
| Pilot Pro (bundled with Forge) | $0 | bundled | inherits Forge support | Same as Pilot Pro standalone, included automatically for every Forge Pro subscriber |

**Why 30-day trial on Pilot vs 14 on Forge:** in the first 3
months of public release, NAC3 supply is thin. Users need extra
time to find apps to control before deciding Pilot is worth
paying for. As supply grows, this trial shortens to 14 days.

## Trial -> degradation

After 30 days unpaid, Pilot does NOT block launch. It:

- Rate-limits the agent to 3 dispatch actions per day.
- Hides the registry browser (manual .well-known paste still
  works).
- Disables premium TTS voices (falls back to Web Speech API).
- Disables mobile companion sync.
- Shows a dismissible banner: "Trial expired -- support at
  {polar_link}".

We push to convert via friction accumulation, not UI lockout.
Audience includes both technical users (who would patch a modal)
and non-technical accessibility users (who would just churn
silently if blocked). Both groups respond better to graceful
degradation.

## BYOK policy (canonical text)

> Yujin Pilot is BYOK (bring your own key) for AI features. You
> pay Anthropic, OpenAI, or Google directly for AI usage. We
> never see your tokens. The subscription covers the controller
> tooling -- registry browser, cross-app routing, voice
> recognition, premium TTS, multi-device sync. It does not cover
> AI consumption.
>
> Why: AI API pricing changes monthly. We refuse to charge a
> markup that fluctuates with someone else's pricing.

Verbatim in: README, pricing page, setup-wizard step 1, EULA.

## Open core line

| Lives in MIT nac-spec | Lives in private yujin-pilot repo |
|----------------------|-----------------------------------|
| NAC3 spec | The Pilot desktop/mobile app shells |
| `nac-chat-client.js` reference | Registry browser UI |
| `nac-mcp-interop.js` reference | Multi-app sidebar + cross-app routing |
| .well-known/nac3-manifest.json schema | Premium TTS voice catalog |
| | OS keychain + secure pairing token store |
| | Mobile companion (iOS / Android) |
| | Multi-device sync |
| | Polished sumi-e voice activation UI |
| | Auto-update + telemetry |

If a feature is not on either column, it does not exist yet.

---

## Execution backlog (decided, not started)

Tracks A through F below mirror the tasks in the Yujin internal
task tracker. Tracks C + D are the Pilot-facing slices; tracks
A + F are shared with Forge; tracks B + E are owned by Forge.

### A. Strategy + commercial framing (shared)

- [A1] `docs/COMMERCIAL_PLAN.md` in both repos. [DONE here]
- [A2] Trial structure + graceful degradation policy doc.
- [A3] Open-core line drawn in each product README.
- [A4] BYOK token policy block in product README + landing.

### C. Pilot product MVP

- [C1] Registry browser ("Descubrir apps NAC3" tab).
- [C2] Pairing flow (open / oauth / api_key).
- [C3] Voice + chat dispatch to paired apps.
- [C4] .well-known/nac3-manifest.json manual paste flow.

### D. Registry infrastructure (lives in yujin.app)

- [D1] DB schema + backend.
- [D2] Submission endpoint + DNS verify.
- [D3] List endpoint + i18n + CDN cache.
- [D4] GitHub topic `nac3-compliant` seeder cron.
- [D5] Moderation queue UI + verified badge.
- [D6] Public landing page yujin.app/registry.

### E. Supply bootstrap (owned by Forge side but Pilot consumes)

- [E1] yujin-CRM as NAC3 day-1 reference app for Pilot to
  control on first launch.
- [E2] Seed bounty -- the 10 OSS apps become Pilot's day-1
  catalog beyond yujin-CRM.

### F. Payments + legal (shared)

- [F1] Polar.sh setup for Forge + Pilot products.
- [F2] License key + offline activation flow.
- [F3] EULA + Privacy + ToS texts.
- [F4] Anthropic affiliate check.

---

## Sequencing

1. Track A locks first (decisions only).
2. Track D (registry) ships before Track C (Pilot UI) because
   the Pilot registry browser depends on the registry endpoint
   being live.
3. Track C in order: C1 (browser) -> C4 (manual paste) -> C2
   (pairing) -> C3 (dispatch). Manual paste before pairing
   because it is simpler and gives a working test path even
   while pairing is half-built.
4. Track E (supply) runs concurrent with C; without supply
   Pilot has nothing to demo on first launch.
5. Track F (payments) blocks public launch but not dogfood.

---

## Why Pilot exists at $5 not $19

Pilot's audience is broader than devs: accessibility users,
voice-first users, end-users of NAC3-compliant apps that they
did not build themselves. That audience is price-sensitive in
ways the dev audience is not. $5 is "I do not deliberately
decide to pay each month" -- friction-free for end users -- but
still filters freeloaders and gives Polar's micro-recurring
something to bill.

For the dev audience (Forge subscribers), Pilot is bundled free
because the marginal cost to us is zero and the marketing value
(every Forge dev runs Pilot on their laptop in front of their
colleagues) is high.

## What we deliberately did NOT do

- No standalone $19 Pilot tier. Pilot is commodity priced; the
  $19 lives in Forge.
- No blocking modal at trial expiry. Audience includes
  accessibility users who would lose access entirely instead of
  paying.
- No "free with ads" tier. Voice ads are user-hostile and would
  burn the brand permanently.

## See also

- `yujin-forge/docs/COMMERCIAL_PLAN.md` -- the supply side.
- `nac-spec/SPEC.md` -- the protocol Pilot speaks.
- `nac-spec/docs/NAC_INTEROP_MCP.md` -- the v2.3 interop layer
  that cross-app routing builds on.

## License

This document is published under CC-BY-4.0. The code it
describes is split: nac-spec MIT/Apache; yujin-pilot
proprietary.
