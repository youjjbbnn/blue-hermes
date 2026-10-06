# Blue Hermes 💙

<div dir="rtl">

**بلو هرمس** — یک تمپلیت آماده‌ی دیپلوی برای [Hermes Agent](https://github.com/NousResearch/hermes-agent) با پنل ادمین تحت وب.

**بهترین ویژگی:** به‌جای محدودیت به یک Endpoint سفارشی، می‌تونید **بی‌نهایت پروایدر OpenAI-compatible** با Base URLهای کاملاً متفاوت اضافه کنید — vLLM، Ollama، LM Studio، پراکسی خصوصی، یا هر سروری که API سازگار با OpenAI داره.

</div>

---

**Blue Hermes** is a ready-to-deploy template for [Hermes Agent](https://github.com/NousResearch/hermes-agent) by [Nous Research](https://nousresearch.com/), with a web-based admin dashboard for configuration, gateway management, and user pairing.

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/hermes-agent-ai?referralCode=QXdhdr&utm_medium=integration&utm_source=template&utm_campaign=generic)

Hermes Agent is an autonomous AI agent that lives on your server, connects to your messaging channels (Telegram, Discord, Slack, etc.), and gets more capable the longer it runs.

## ✨ What makes Blue Hermes different

The headline feature: **unlimited custom OpenAI-compatible endpoints.**

Most templates let you configure a *single* custom endpoint. Blue Hermes stores a **dynamic array** of providers, each fully independent:

```json
[
  {
    "id": "vllm-local",
    "name": "Local vLLM",
    "baseUrl": "https://api.domain-a.com/v1",
    "apiKey": "sk-vllm-secret",
    "models": ["qwen-32b", "llama-70b"]
  },
  {
    "id": "home-ollama",
    "name": "Home Ollama",
    "baseUrl": "http://192.168.1.50:11434/v1",
    "apiKey": "sk-ollama-secret",
    "models": ["gpt-oss:120b", "qwen3:32b"]
  }
]
```

Point it at vLLM, Ollama, LM Studio, a private proxy, or any host with an OpenAI-compatible `/v1` API. Each provider keeps its own URL, its own key, and its own model list — and Hermes routes every request to the correct URL based on the model you select.

## 📑 Table of Contents

- [Features](#-features)
- [Quick Start (Railway)](#-quick-start-railway)
- [Quick Start (Docker)](#-quick-start-docker)
- [Adding a Custom Provider](#-adding-a-custom-provider)
- [Using the Admin Dashboard](#-using-the-admin-dashboard)
- [Environment Variables](#-environment-variables)
- [Supported Providers](#-supported-providers)
- [Channels & Tools](#-channels--tools)
- [Architecture](#-architecture)
- [Updating Hermes](#-updating-hermes)
- [Backup & Restore](#-backup--restore)
- [FAQ (فارسی)](#-faq-فارسی)

## 🎯 Features

- **Unlimited custom endpoints** — vLLM, Ollama, LM Studio, or any OpenAI-compatible server, each with its own URL, key, and models. Add and remove from the UI; nothing needs a redeploy.
- **Per-endpoint Test Connection** — each provider row has its own test button that hits `<baseUrl>/models` and lists the models it actually serves. Verified before you save, never after.
- **Masked API keys** — keys are masked in the UI and never echoed back in full; a masked value on save means "keep the stored one."
- **Admin Dashboard** — dark-themed setup wizard at `/setup` to configure providers, channels, tools, and manage the gateway.
- **Full Hermes Dashboard** — the native Hermes web UI (Chat, Keys, Skills, Kanban, Analytics, Console) is proxied at `/`, behind the same login.
- **One-Page Setup** — provider dropdown, checkbox-based channel/tool toggles — no config files to edit.
- **Gateway Management** — start, stop, restart the Hermes gateway from the browser, with automatic restart if it crashes.
- **Live Status** — stat cards for gateway state, uptime, model, and pending pairing requests.
- **Live Logs** — streaming gateway log viewer.
- **User Pairing** — approve or deny users who message your bot, revoke access anytime.
- **Password-Protected** — one cookie-based login guards both the setup wizard and the Hermes dashboard.
- **Backup & Restore** — download a full snapshot (config, credentials, chat history, memories, skills) as a zip, and restore it — including into a fresh project. A safety snapshot is taken automatically before every restore.

## 🚀 Quick Start (Railway)

### 1. Get an LLM provider key (free)

1. Register for free at [OpenRouter](https://openrouter.ai/)
2. Create an API key from your [OpenRouter dashboard](https://openrouter.ai/keys)
3. Pick a free model from the [model list sorted by price](https://openrouter.ai/models?order=pricing-low-to-high) (e.g. `google/gemma-3-1b-it:free`, `meta-llama/llama-3.1-8b-instruct:free`)

### 2. Set up a Telegram bot (fastest channel)

Hermes Agent interacts entirely through messaging channels — there is no chat UI like ChatGPT. Telegram is the quickest to set up:

1. Open Telegram and message [@BotFather](https://t.me/BotFather)
2. Send `/newbot`, follow the prompts, and copy the **Bot Token**
3. Send a message to your new bot — it will appear as a pairing request in the admin dashboard
4. To find your Telegram user ID, message [@userinfobot](https://t.me/userinfobot)

### 3. Deploy to Railway

1. Click the **Deploy on Railway** button at the top of this README
2. Set the `ADMIN_PASSWORD` environment variable (or a random one will be generated and printed to deploy logs)
3. Attach a **volume** mounted at `/data` (persists config across redeploys)
4. Open your app URL — log in with username `admin` and your password

### 4. Configure in the admin dashboard

1. **LLM Provider** — select OpenRouter from the dropdown, paste your API key, enter the model name
2. **Messaging Channel** — check Telegram, paste the Bot Token from BotFather
3. Click **Save & Start** — the gateway will start and your bot goes live

### 5. Start chatting

Message your Telegram bot. If you're a new user, a pairing request will appear in the admin dashboard under **Users** — click **Approve**, and you're in.

## 🐳 Quick Start (Docker)

```bash
docker build -t blue-hermes .
docker run --rm -it -p 8080:8080 \
  -e PORT=8080 \
  -e ADMIN_PASSWORD=changeme \
  -v blue-hermes-data:/data \
  blue-hermes
```

Open `http://localhost:8080` and log in with `admin` / `changeme`.

Prefer plain Docker Compose:

```yaml
services:
  blue-hermes:
    build: .
    ports:
      - "8080:8080"
    environment:
      PORT: 8080
      ADMIN_PASSWORD: changeme
    volumes:
      - ./data:/data
    restart: unless-stopped
```

```bash
docker compose up -d
```

> Both forms mount `/data` — that volume is where every config, key, and chat session lives. Losing it means starting over.

## 🔌 Adding a Custom Provider

This is the feature Blue Hermes is built around. In the admin dashboard:

1. Open **Setup** → scroll to the **Custom Endpoints** section
2. Click **+ Add Provider**
3. Fill in the fields for that provider:

| Field | Example | Notes |
|-------|---------|-------|
| **Name** | `Local vLLM` | A label for the dropdown, purely for you |
| **Base URL** | `https://api.domain-a.com/v1` | Must end in `/v1` or equivalent |
| **API Key** | `sk-...` | Masked after save; blank means no key |
| **Models** | `qwen-32b, llama-70b` | Comma-separated model IDs this URL serves |

4. Click **Test Connection** on that row — it sends `GET <baseUrl>/models` and lists what the endpoint actually serves, so you know the URL and key work *before* you save
5. Click **Save & Start**

Repeat for every provider you want. Each one is stored independently with its own URL, key, and model list, and Hermes routes each request to the correct URL based on the model you select.

Common setups this is designed for:

- **vLLM** — `http://<host>:8000/v1`
- **Ollama** — `http://<host>:11434/v1`
- **LM Studio** — `http://<host>:1234/v1`
- **Private proxy** — `https://your-proxy.example.com/v1`
- **Any OpenAI-compatible gateway** — 9Router, OmniRoute, LiteLLM, etc.

> **Migrating from the old format?** A template that only had the single `CUSTOM_PROVIDER_*` pair auto-migrates to the array on first boot — nothing to do manually.

## 🛠 Using the Admin Dashboard

Once deployed, everything is managed from the browser — no config files, no SSH.

| Page | What it's for |
|------|---------------|
| **Setup** | Providers, channels, tools, custom endpoints, model selection |
| **Dashboard** | The full native Hermes UI — Chat, Keys, Skills, Kanban, Analytics, Console |
| **Logs** | Streaming gateway log output |
| **Users** | Approve/deny pairing requests, revoke access |
| **Backup** | Download or restore a full snapshot zip |

The first visit redirects to `/setup` if nothing is configured yet. After that, `/` proxies to the native Hermes dashboard.

## 🔐 Environment Variables

Only three live outside the dashboard:

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `8080` | Web server port (set automatically by Railway) |
| `ADMIN_USERNAME` | `admin` | Login username |
| `ADMIN_PASSWORD` | *(auto-generated)* | Login password — if unset, a random password is printed to the deploy logs. Changing it redeploys the service, which signs everyone out. |
| `HERMES_REF` | *(pinned in Dockerfile)* | Hermes Agent version to install (any upstream git tag/branch). Override the Dockerfile default without editing code — see [Updating Hermes](#-updating-hermes). |

All other configuration — providers, models, channels, tools, custom endpoints — is managed through the admin dashboard and stored in `/data/.hermes/`.

## 🧠 Supported Providers

Selectable from the setup wizard's dropdown:

OpenRouter, Anthropic (Claude), Google AI Studio, xAI (API key **or** SuperGrok OAuth), DeepSeek, Qwen Cloud (DashScope), GLM / Z.AI, Kimi, MiniMax (global **and** China), NVIDIA NIM, Fireworks AI, NovitaAI, Arcee AI, Step Plan, GMI Cloud, Hugging Face, GitHub Copilot, OpenCode Zen, OpenCode Go, Kilo Code, Ollama Cloud, Actual Computer, AWS Bedrock, Azure Foundry, **9Router**, **OmniRoute**, and **any number of OpenAI-compatible custom endpoints**.

9Router and OmniRoute are separately deployed routing gateways, not services bundled into this image. Select either one in the setup wizard and enter the gateway's reachable OpenAI-compatible `/v1` URL, its endpoint API key, and a model or combo name. Both can remain configured at the same time; OmniRoute defaults the model field to its `auto` smart-routing alias.

Every other provider Hermes supports can still be configured from the Hermes Dashboard → **Keys** tab — the wizard covers the common ones, not the limit.

> **OpenCode Free:** Hermes v2026.9.21 removed the anonymous `opencode-free` provider because its upstream relay now rejects external anonymous clients. Use OpenCode Zen or OpenCode Go with the corresponding API key instead.

## 📡 Channels & Tools

**Channels:** Telegram, Discord, Slack, WhatsApp, Email, Mattermost, Matrix

**Tools:** Parallel (search), Firecrawl (scraping), Keenable (search), Tavily (search), Perplexity (search), FAL (image gen), Browserbase, GitHub, OpenAI Voice (Whisper/TTS), Honcho (memory)

## 🏗 Architecture

One container runs a single public process that fronts two managed Hermes subprocesses:

```
Railway Container
└── server.py — Starlette + Uvicorn on 0.0.0.0:$PORT   (the only public surface)
    ├── /login, /logout    — cookie login (7-day, httponly)
    ├── /health            — health check (no auth)
    ├── /setup             — this template's setup wizard
    ├── /setup/api/*       — config, status, logs, gateway, pairing, backup, OAuth
    ├── /  and  /*         — reverse-proxied to the native Hermes dashboard
    │
    ├── hermes dashboard   — native Hermes web UI, bound to 127.0.0.1:9119
    └── hermes gateway     — the agent itself (Telegram, Discord, …), auto-restarted
```

The Hermes dashboard is **never exposed directly** — it binds loopback and is reachable only through the proxy, so one login covers both UIs. The gateway is supervised: if it crashes or is OOM-killed, `server.py` restarts it with backoff, giving up only if it fails repeatedly (Railway would not restart it on its own, because `server.py` is still alive and healthy). If a config save or restore respawns the dashboard while Chat is open, the terminal connection retries automatically.

The image builds and verifies SQLite 3.53.4 rather than using Debian Bookworm's affected 3.40.1 library. This matches Hermes v2026.9.21's official container requirement and protects session/FTS databases from SQLite's WAL-reset defect.

Hermes v2026.9.21 can serve all live named profiles from this one gateway. The template's `/setup` page remains the default-profile bootstrap/admin surface; use the native Hermes Dashboard's profile selector for named-profile configuration and pairing. Start, Stop, and Restart act on the shared gateway and therefore affect every profile it serves.

Config lives on the `/data` volume at `/data/.hermes/` (`.env`, `config.yaml`, `auth.json`, `custom_endpoints.json`, sessions, pairing state) and survives redeploys. Gateway output is captured into a ring buffer and streamed to the Logs panel.

## 🔄 Updating Hermes

This template pins a specific Hermes Agent release in the `Dockerfile` (`ARG HERMES_REF`, currently `v2026.9.21`). To upgrade:

- **Recommended:** set a `HERMES_REF` service variable in Railway to any upstream [release tag](https://github.com/NousResearch/hermes-agent/releases) (e.g. `v2026.9.21`), then redeploy. It's passed in as a Docker build arg and overrides the Dockerfile default — no code change needed.
- **Or** bump `ARG HERMES_REF` in the `Dockerfile` and redeploy.

The "Update" button inside the Hermes dashboard is a **no-op on Railway** (it detects a container install and refuses) — the image is immutable, so a runtime self-update wouldn't survive a redeploy. Bump `HERMES_REF` and redeploy instead. When jumping releases, re-check that the Dockerfile's install extras still match upstream's `pyproject.toml`.

## 💾 Backup & Restore

From the dashboard's **Backup** page:

- **Download** a zip containing config, credentials, chat history, memories, and skills
- **Restore** it into the same project, or into a fresh one to clone a deployment

A safety snapshot is taken automatically before every restore. The zip is not encrypted — store it somewhere safe. Your custom endpoints and their keys are included, so a restore brings the whole provider list back.

## ❓ FAQ (فارسی)

<div dir="rtl">

**۱. آیا واقعاً می‌تونم چندتا vLLM/Ollama با آدرس‌های مختلف اضافه کنم؟**
بله. هر تعداد که بخوای. هر پروایدر Base URL، API Key و لیست مدل‌های خودش رو داره و کاملاً مستقل از بقیه‌ست.

**۲. وقتی مدل انتخاب می‌کنم، چطور می‌فهمه به کدوم URL بفرسته؟**
هر مدل به پروایدرش متصل می‌شه. وقتی مدلی از `home-ollama` رو انتخاب می‌کنی، درخواست به همون `http://192.168.1.50:11434/v1` با کلید همون پروایدر می‌ره.

**۳. کلیدهای API امنن؟**
تو UI ماسک می‌شن (مثل `sk-vllm-***`). وقتی ذخیره می‌کنی و کلید رو ماسک می‌ذاری، یعنی «همون کلید قبلی رو نگه دار». تو فایل ZIP بکاپ کلید واقعی ذخیره می‌شه، پس فایل بکاپ رو جای امن نگه دار.

**۴. از تمپلیت قدیمی (تک اندپوینت) دارم، چی می‌شه؟**
تنظیمات `CUSTOM_PROVIDER_*` قدیمی تو اولین بوت خودکار به آرایه‌ی جدید مهاجرت می‌کنه. هیچ کاری لازم نیست انجام بدی.

**۵. وقتی یه پروایدر رو حذف می‌کنم، کلیدش چی می‌شه؟**
به‌طور خودکار از فایل `.env` حذف می‌شه، تا یه پروایدر مرده قابل دسترس بمونه و کلید قدیمی لو نره.

**۶. تو Railway داده‌ها حفظ می‌شن؟**
بله، به شرطی که یه volume به `/data` وصل کرده باشی. همه چیز — کانفیگ، کلیدها، تاریخچه چت — اونجا ذخیره می‌شه.

**۷. پنل روی پورت دیگه‌ای اجرا می‌کنم. چطور؟**
متغیر `PORT` رو تنظیم کن. تو Railway خودکار هست، تو Docker با `-e PORT=8080`.

</div>

## 📜 Credits

- [Hermes Agent](https://github.com/NousResearch/hermes-agent) by [Nous Research](https://nousresearch.com/)
- UI originally inspired by the OpenClaw admin template

Blue Hermes is an independent template and is not affiliated with or endorsed by Nous Research. The name refers to this deployment template only; the agent it runs is upstream Hermes Agent, unmodified.
