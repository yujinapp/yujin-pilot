# Yujin Pilot

**The embedded chat + voice cockpit for NAC-3 apps.** A single-file
IIFE bundle that drops into any NAC-3-decorated page and gives it a
floating sumi-e button. Click the button, paste an API key, talk to
your app -- Pilot reads the registered manifest, sends your intent to
the configured LLM, dispatches the action chain via
`NAC.click_by_verb`, and reads back the result.

> Status: **embed bundle shipped**, 2026-05-20.
> Multi-provider (Anthropic, OpenAI, Google, DeepSeek, Groq), STT via
> Web Speech, TTS via Web Speech / ElevenLabs / Google Cloud TTS.
> Validated end-to-end on the calc migration paper.

## What it is

Pilot is the **driver** half of the NAC-3 ecosystem:

```
NAC (Apache-2.0)  -- the protocol
Yujin Forge       -- builds NAC-3 apps  (yujinapp/yujin-forge)
Yujin Pilot       -- drives NAC-3 apps  (you are here)
```

Where Forge bakes the agent-driveable surface into a page, Pilot is
the agent. The two products ship separately and can be mixed: a
Forge-decorated app can be driven by something other than Pilot
(e.g. an MCP server, a custom Playwright harness); Pilot can drive
any NAC-3-compliant app, however it was decorated.

## Quick start

### Install into a Forge-decorated app

```bash
yf pilot install ./my-decorated-app
# Copies pilot.bundle.js + pilot.css; injects 2 tags into index.html.
```

### Install manually

```html
<!-- after nac.browser.js + NAC.register(manifest) -->
<link rel="stylesheet" href="pilot.css">
<script src="pilot.bundle.js"></script>
```

That's it. A floating sumi-e branch button appears bottom-right. The
first time the user clicks it, the chat panel asks them to open
`Configuracion` and paste at least one API key.

## Features

| | |
|---|---|
| **Multi-provider** | Anthropic, OpenAI, Google, DeepSeek, Groq. All models from the [NAC3 v2.3 benchmark](https://yujin.app/nac-spec/benchmark/) are in the selector. |
| **Voice** | STT via Web Speech API (Chrome/Edge). TTS via Web Speech (default), ElevenLabs, or Google Cloud TTS (with keys). |
| **3 modes** | Globito (just a balloon), chat (text panel + history), pizarra (chat + voice + action trace). Mode is per-user in localStorage. |
| **Status probe** | Optional CSS selector to read state after every dispatch (e.g. `#display` for the calc). Result is included in Pilot's reply. |
| **On / off** | Right-click the floating button (or shift-click) to open the menu and toggle Pilot off without uninstalling. |
| **Model selector** | Per-conversation in the chat header. Default Sonnet 4.6; switchable on the fly. |
| **No CDN deps** | The bundle is self-contained vanilla JS. No build step on the consumer side. |

## How it works

1. On load, Pilot reads `window.NAC.list_registered_plugins()` and
   `window.NAC.manifest(slug)` to learn what verbs are available.
2. User types or speaks an intent ("calcula 3 mas 4").
3. Pilot builds a system prompt with the manifest embedded and
   sends it + the user utterance to the configured LLM via direct
   HTTP (no proxy in client-side mode).
4. The LLM returns JSON: `{say, plugin, actions: [{verb}, ...]}`.
5. Pilot walks `actions[]` and dispatches each via
   `await NAC.click_by_verb(plugin, verb)`.
6. If a status probe selector is configured, Pilot reads
   `document.querySelector(...).textContent` after dispatch and
   includes it in the reply.
7. Pilot renders the reply in the chat panel and speaks it via TTS
   if enabled. The pizarra panel (large mode) shows the verb trace.

Configuration persists in `localStorage` under `yujin_pilot.config`.

## Security

API keys live in `localStorage`. **Client-side mode is for local +
demo use only**. For production deployments, put the LLM call behind
a server proxy that holds the key, and have Pilot talk to your proxy
instead of the model API directly.

## API surface

Pilot exposes `window.YujinPilot`:

```js
window.YujinPilot.open();                    // open the chat panel
window.YujinPilot.close();                   // close it
window.YujinPilot.configure();               // open the settings modal
window.YujinPilot.enable() / disable();      // toggle the bundle on/off
window.YujinPilot.sendMessage(text);         // programmatically submit an intent
window.YujinPilot.setModel('gpt-5.5');       // switch model
window.YujinPilot.reset();                   // wipe localStorage config
window.YujinPilot.version;                   // '0.1.0'
```

These are useful for embedders who want to integrate Pilot with
their own onboarding, deep-link a specific intent, or wire it to a
keyboard shortcut.

## Files

```
yujin-pilot/
  dist/
    pilot.bundle.js          # IIFE, ~25 KB. Vanilla JS, no deps.
    pilot.css                # Companion stylesheet. Yujin tokens, sumi-e accent.
  README.md
  CLAUDE.md                  # repo policy for AI-assisted contributions
  LICENSE                    # Apache-2.0 (commercial license pending)
  docs/
    SPEC.md                  # full product spec
```

## Versioning + release

Pilot tracks NAC3 versions. Pilot `0.1.x` requires NAC `2.3.x`. The
bundle imports `window.NAC` (provided by `nac.browser.js`); the
two ship and version independently.

## See also

- [NAC3 spec](https://yujin.app/nac-spec/SPEC.html)
- [Yujin Forge](https://github.com/yujinapp/yujin-forge) -- the sibling build tool
- [Migration paper](https://yujin.app/nac-spec/migration/) -- live demos showing Pilot embedded in calc-forge-silent / -assisted / sumi-manual
- [NAC3 v2.3 benchmark](https://yujin.app/nac-spec/benchmark/) -- the model catalog Pilot's selector exposes
