# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A single-file, zero-dependency web app (`index.html`) that generates random questions and answers via a locally running [Ollama](https://ollama.com) instance. Everything — markup, styles, and logic — lives in one file. There is no build step, no package manager, and no test suite.

## Running the app

Open `index.html` directly in a browser, or serve it with any static file server:

```bash
python3 -m http.server 8080
# then visit http://localhost:8080
```

Ollama must be running locally (default: `http://localhost:11434`) and have at least one model pulled (e.g. `ollama pull llama3`).

## Architecture

All code is in `index.html` in three sections:

- **CSS (`<style>`)** — design-token-driven (CSS custom properties in `:root`), no framework.
- **HTML (`<body>`)** — five `.card` sections: Ollama settings, topic picker, question display, answer display, history list.
- **JavaScript (`<script>`)** — vanilla JS, no modules. Key pieces:
  - `streamGenerate(prompt, onChunk, signal)` — POSTs to `/api/generate` with `stream: true`, reads the NDJSON response body with a `ReadableStream` reader, and calls `onChunk` for each token.
  - `generateQuestion()` / `answerQuestion()` — orchestrate streaming into the DOM with a blinking cursor element; guard re-entrancy with the `streaming` boolean.
  - `history` array (capped at 20) and `renderHistory()` — in-memory only, not persisted.
  - `pingOllama()` — called on page load; hits `/api/tags` to discover available models.

## Key constraints

- **No CORS proxy.** The browser calls Ollama directly, so Ollama must allow cross-origin requests or the page must be served from the same origin. Ollama's default config allows `localhost` origins.
- **Streaming via NDJSON.** Each line from `/api/generate` is a JSON object with a `response` field; `obj.done === true` signals completion.
- **In-memory state only.** Refreshing the page clears history and the current question.
