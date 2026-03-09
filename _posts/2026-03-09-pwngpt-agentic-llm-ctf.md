---
layout: post
title: "PwnGPT — Building an Agentic LLM Capture The Flag"
description: "An open-source, multi-agent CTF for learning prompt injection, information retrieval, and LLM security — built with Google ADK, FastAPI, and HTMX."
tags: [security, ai, llm, ctf, prompt-injection, python, google-adk]
---

Prompt injection is one of the most talked-about attack vectors in AI security, yet hands-on learning resources remain scarce. That gap is what led me to build [PwnGPT](https://github.com/c-goosen/ai-prompt-ctf) — an open-source, agentic LLM Capture The Flag challenge designed to teach players about prompt injection, information retrieval, and the real security issues surrounding large language models.

The project started as a companion to a [BSides Cape Town](https://bsidescapetown.co.za) event and has since been run at multiple events. The goal is simple: make LLM security accessible to newcomers and veterans alike, with progressive difficulty levels that go from a gentle introduction all the way to genuinely tricky defences.

### Architecture

PwnGPT follows a modern web stack:

**Google ADK API &harr; FastAPI Backend &harr; HTMX Frontend**

- **Google ADK API** powers the agentic LLM capabilities and evaluation framework.
- **FastAPI** provides async REST endpoints and serves the application.
- **HTMX** delivers a dynamic, interactive frontend without heavy JavaScript frameworks.

### Why HTMX?

A CTF frontend needs to feel responsive — players are sending prompts and waiting for agent replies in a conversational loop. A full SPA framework like React or Vue would be overkill here. [HTMX](https://htmx.org/) lets us keep the frontend as server-rendered HTML while still getting smooth, async interactions.

When a player submits a prompt, HTMX fires a POST to the FastAPI backend, which proxies the request to the ADK API server. The response HTML fragment is swapped directly into the page — no JSON parsing, no client-side state management, no build step. File uploads for multi-modal levels (images and audio) work the same way: HTMX handles the form submission, FastAPI processes the file, and the agent response is streamed back as HTML.

The result is a frontend that is fast to develop, easy to extend with new levels, and light enough to deploy anywhere.

### Multi-Agent Design

The interesting part is the agent architecture. A coordinator pattern routes players to level-specific agents, each with its own system prompt and security measures:

- **CTFSubAgentsRoot** — the main coordinator that delegates to level agents.
- **Level 0 through Level 9 agents** — each progressively harder, with layered defences.
- **Function tools** — specialised functions like `submit_answer_func`, `password_search_func`, `hints_func`, and `sql_query` that agents can call to validate answers, search vector stores, and interact with the backend.

### The Levels

Each level teaches a different aspect of LLM security. The difficulty ramps up as defences get more sophisticated:

| Level | Theme |
|-------|-------|
| 0 | **Basic prompt injection** — a warm-up to get familiar with the format. |
| 1 | **Input injection** — the agent has simple input-level guardrails to bypass. |
| 2 | **Output protection** — defences focus on filtering what the model says back to you. |
| 3 | **Advanced prompt engineering** — requires more creative prompt crafting. |
| 4 | **Vision multi-modal injection** — upload an image and convince the agent to leak the secret. |
| 5 | **Audio multi-modal injection** — same idea, different modality. |
| 6 | **Function calling injection** — exploit the agent's ability to call backend tools. |
| 7 | **Prompt-Guard protection** — a dedicated prompt-injection classifier guards this level. |
| 8 | **Prompt-Goose protection** — a custom fine-tuned model provides the defence layer. |
| 9 | **Chain of Thought challenges** — the agent reasons step-by-step, making simple tricks less effective. |

The multi-modal levels (4 and 5) are particularly interesting because they show that prompt injection is not limited to text — images and audio can carry adversarial payloads too.

### Vector Search with LanceDB

Passwords and secrets are embedded into a [LanceDB](https://lancedb.com/) vector store. This means some levels require players to understand how retrieval-augmented generation (RAG) works and exploit the retrieval pipeline itself — not just the prompt.

### Running It Yourself

The project uses `uv` for dependency management:

```bash
uv sync
python ctf/main.py
```

Or via Docker:

```bash
docker build -t llm_challenge .
docker run -p 8000:8000 llm_challenge
```

API docs are available at `/docs` when the `DOCS_ON=True` environment variable is set.

### Why Open Source?

LLM security is evolving fast. By keeping PwnGPT open source, anyone can contribute new challenge levels, experiment with different guardrail strategies, or use it as a training tool at their own events. If you run a security meetup or conference, feel free to spin it up — and please [open a PR](https://github.com/c-goosen/ai-prompt-ctf/pulls) if you add something interesting.

Check out the repo: [github.com/c-goosen/ai-prompt-ctf](https://github.com/c-goosen/ai-prompt-ctf)
