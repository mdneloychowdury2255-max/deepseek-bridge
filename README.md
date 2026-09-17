# DeepSeek Bridge — Android WebView LLM Bridge

A production-grade Android bridge that turns the official DeepSeek (and Arena) web chat into a **local, OpenAI-compatible API endpoint** — no API key, no SDK, no external proxy.

> **Status:** active development · Android 12 (Termux) · tested on real device

---

## What it does

Runs a local HTTP server inside the app. Any OpenAI-compatible client (Hermes Agent, LangChain, curl, your own app) can `POST /v1/chat/completions` and get a real model reply back — routed through a hidden WebView that drives the real DeepSeek UI.

```
client  ──POST──▶  LocalApiServer  ──▶  WebView (chat.deepseek.com)  ──▶  model
                        ◀──────────────── scraped reply ◀──────────────┘
```

## Architecture

| Component | Role |
|---|---|
| `LocalApiServer` | OpenAI-compatible REST server, rate limiting, response filtering |
| `DeepSeekChatView` | WebView wrapper — typing, sending, polling, scraping |
| `BridgeConfig` | Hot-loaded config — JS templates & knobs, no rebuild needed |
| `send.js` / `nodes.js` / `chrome.txt` | Per-site JS + regex driving the page |

## Hot config (no rebuild)

All tuning lives in `/storage/emulated/0/web api/<site>/`:

```
web api/
├── deepseek/
│   ├── bridge.properties   # numeric knobs
│   ├── send.js             # type + click send
│   ├── nodes.js            # reply selector
│   ├── chrome.txt          # UI-noise regex filter
│   └── toolfp.txt          # tool-call fingerprints
└── arena/
    └── ...
```

Edit a file → next request uses it. Delete a file → compiled-in default returns. **The app never breaks because of this directory.**

## Anti-detection hardening

- **Randomised request delay** — configurable range (default 3–6 s), never a fixed interval
- **Session warmup** — first N requests run slower, like a human exploring
- **Burst protection** — consecutive fast requests get an escalating penalty (capped)
- **Desktop user-agent** — masks mobile WebView signature
- **Human-like scrolling / typing** — value-set + input events, not raw paste
- **No hard request caps** — throttling only, never a 429 that kills a live session

## Quick start

```bash
# 1. install the APK, log in to DeepSeek in the WebView
# 2. point any OpenAI client at the local server
curl http://127.0.0.1:8080/v1/chat/completions \
  -H "Authorization: Bearer <local-key>" \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek-webview","messages":[{"role":"user","content":"hi"}]}'
```

## Project layout

```
files/java/com/termux/app/
├── LocalApiServer.java      # HTTP server + OpenAI schema
├── DeepSeekChatView.java    # WebView + polling loop
├── DeepSeekService.java     # foreground service
├── BridgeConfig.java        # hot config loader
└── TermuxActivity.java      # host activity
```

## Roadmap

- [x] OpenAI-compatible endpoint
- [x] Multi-site (DeepSeek + Arena)
- [x] Hot-loaded config
- [x] Human-like rate limiting
- [ ] Streaming (SSE) responses
- [ ] Multi-account rotation
- [ ] Tool-calling passthrough

## License

Private / all rights reserved — `mdneloychowdury2255-max`.
