# Blue Hermes 💙

<div dir="rtl">

**بلو هرمس** — یک تمپلیت آماده‌ی دیپلوی برای [Hermes Agent](https://github.com/NousResearch/hermes-agent) با پنل ادمین تحت وب.

**بهترین ویژگی:** به‌جای محدودیت به یک Endpoint سفارشی، می‌تونید **بی‌نهایت پروایدر OpenAI-compatible** با Base URLهای کاملاً متفاوت اضافه کنید — vLLM، Ollama، LM Studio، پراکسی خصوصی، یا هر سروری که API سازگار با OpenAI داره.

</div>

---

**Blue Hermes** is a ready-to-deploy template for [Hermes Agent](https://github.com/NousResearch/hermes-agent) by [Nous Research](https://nousresearch.com/), with a web-based admin dashboard for configuration, gateway management, and user pairing.

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
- [Deploy to Railway (step by step)](#-deploy-to-railway-step-by-step)
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

## 🚀 Deploy to Railway (step by step)

The whole thing takes about 10 minutes. The only thing you need beforehand is a GitHub account.

<div dir="rtl">

### راهنمای فارسی — دیپلوی روی Railway

تمام کار حدود ۱۰ دقیقه طول می‌کشه. فقط یه اکانت گیت‌هاب لازم داری.

</div>

---

### English

#### Step 1 — Get an LLM provider key (free)

You need a model for your agent to think with. OpenRouter is the easiest way in:

1. Register for free at [openrouter.ai](https://openrouter.ai/)
2. Go to your [API keys page](https://openrouter.ai/workspaces/default/keys) and click **Create Key**
3. Copy the key — it starts with `sk-or-...`
4. Pick a model from the [model list](https://openrouter.ai/models) (filter by "Free" for no-cost options, e.g. `google/gemma-3-1b-it:free`)

<div dir="rtl">

#### مرحله ۱ — گرفتن کلید LLM (رایگان)

به یه مدل نیاز داری تا عاملت باهاش فکر کنه. OpenRouter ساده‌ترین راهه:

۱. تو [openrouter.ai](https://openrouter.ai/) رایگان ثبت‌نام کن
۲. به [صفحه کلیدها](https://openrouter.ai/workspaces/default/keys) برو و **Create Key** رو بزن
۳. کلید رو کپی کن — با `sk-or-...` شروع می‌شه
۴. یه مدل از [لیست مدل‌ها](https://openrouter.ai/models) انتخاب کن (با فیلتر "Free" می‌تونی مدل‌های رایگان رو ببینی، مثلاً `google/gemma-3-1b-it:free`)

</div>

#### Step 2 — Create a Telegram bot (fastest channel)

Hermes Agent has no chat UI — it talks through messaging apps. Telegram is the quickest:

1. Open Telegram, message [@BotFather](https://t.me/BotFather)
2. Send `/newbot` and follow the prompts
3. Copy the **Bot Token** it gives you
4. Send any message to your new bot (this registers it)
5. To find your own Telegram user ID, message [@userinfobot](https://t.me/userinfobot)

<div dir="rtl">

#### مرحله ۲ — ساخت ربات تلگرام (سریع‌ترین کانال)

هرمس رابط چت نداره — از طریق پیام‌رسان‌ها حرف می‌زنه. تلگرام سریع‌ترینه:

۱. تلگرام رو باز کن و به [@BotFather](https://t.me/BotFather) پیام بده
۲. `/newbot` رو بفرست و مراحل رو طی کن
۳. **Bot Token**ی که بهت می‌ده رو کپی کن
۴. یه پیام به رباتت بفرست (این کار ربات رو ثبت می‌کنه)
۵. برای پیدا کردن آیدی کاربری خودت، به [@userinfobot](https://t.me/userinfobot) پیام بده

</div>

#### Step 3 — Deploy the project

1. Log in to [railway.com](https://railway.com) with your GitHub account
2. Click **New Project** (top right)
3. Select **Deploy from GitHub repo**
4. Choose `youjjbbnn/blue-hermes` (if it's not listed, click **Configure GitHub App** and grant access), then click **Deploy Now**

Railway clones the repo, builds the Docker image, and starts it. The first build takes a few minutes.

<div dir="rtl">

#### مرحله ۳ — دیپلوی پروژه

۱. با اکانت گیت‌هاب وارد [railway.com](https://railway.com) شو
۲. گوشه‌ی بالا سمت راست **New Project** رو بزن
۳. **Deploy from GitHub repo** رو انتخاب کن
۴. `youjjbbnn/blue-hermes` رو انتخاب کن (اگه تو لیست نیست، **Configure GitHub App** رو بزن و دسترسی بده)، بعد **Deploy Now** رو کلیک کن

ریلوی ریپو رو کلون می‌کنه، ایمیج Docker رو می‌سازه و اجراش می‌کنه. بیلد اول چند دقیقه طول می‌کشه.

</div>

#### Step 4 — Add your admin password

1. In your Railway project, click the service (it's named after the repo)
2. Open the **Variables** tab
3. Click **New Variable** and add:

   | Name | Value |
   |------|-------|
   | `ADMIN_PASSWORD` | a strong password you choose |

4. The service redeploys automatically. If you skip this, Railway generates a random password and prints it in the **Deploy Logs** — copy it from there.

<div dir="rtl">

#### مرحله ۴ — افزودن رمز ادمین

۱. تو پروژه‌ی ریلوی، روی سرویس کلیک کن (اسمش همون اسم ریپوئه)
۲. تب **Variables** رو باز کن
۳. **New Variable** رو بزن و این رو اضافه کن:

   | نام | مقدار |
   |------|-------|
   | `ADMIN_PASSWORD` | یه رمز قوی انتخاب کن |

۴. سرویس خودش دوباره دیپلوی می‌شه. اگه این کار رو نکنی، ریلوی یه رمز تصادفی می‌سازه و تو **Deploy Logs** چاپش می‌کنه — از اونجا کپیش کن.

</div>

#### Step 5 — Attach a data volume

This is what keeps your config, keys, and chat history alive across redeploys.

1. Click your service, open the **Settings** tab
2. Scroll to **Volumes** and click **Add Volume**
3. Mount path: `/data`
4. Leave size at the default (or increase it if you expect heavy use)

<div dir="rtl">

#### مرحله ۵ — وصل کردن Volume داده‌ها

این کار باعث می‌شه کانفیگ، کلیدها و تاریخچه‌ی چت‌ها تو ری‌دیپلوی‌ها حفظ بشن.

۱. روی سرویس کلیک کن و تب **Settings** رو باز کن
۲. به بخش **Volumes** برو و **Add Volume** رو بزن
۳. مسیر مونت: `/data`
۴. سایز رو روی پیش‌فرض بذار (یا اگه استفاده‌ی زیادی داری بیشترش کن)

</div>

#### Step 6 — Open the admin dashboard

1. On your service's **Settings** tab, find **Networking** → **Generate Domain**
2. Railway gives you a URL like `blue-hermes-production-xxxx.up.railway.app`
3. Open it — you land on the login page
4. Log in with username `admin` and the password from Step 4

<div dir="rtl">

#### مرحله ۶ — باز کردن پنل ادمین

۱. تو تب **Settings** سرویس، بخش **Networking** رو پیدا کن و **Generate Domain** رو بزن
۲. ریلوی یه آدرس مثل `blue-hermes-production-xxxx.up.railway.app` بهت می‌ده
۳. بازش کن — به صفحه‌ی لاگین می‌رسی
۴. با نام کاربری `admin` و رمزی تو مرحله ۴ گذاشتی وارد شو

</div>

#### Step 7 — Configure providers and channels

1. On the **Setup** page, open the **LLM Provider** dropdown and pick **OpenRouter**
2. Paste your OpenRouter API key from Step 1
3. Enter the model name (e.g. `google/gemma-3-1b-it:free`)
4. In **Messaging Channels**, check **Telegram** and paste the Bot Token from Step 2
5. Click **Save & Start** — the gateway spins up and your bot goes live

To add your own vLLM / Ollama / LM Studio / private proxy, see [Adding a Custom Provider](#-adding-a-custom-provider) below.

<div dir="rtl">

#### مرحله ۷ — تنظیم پروایدرها و کانال‌ها

۱. تو صفحه‌ی **Setup**، منوی **LLM Provider** رو باز کن و **OpenRouter** رو انتخاب کن
۲. کلید OpenRouter از مرحله ۱ رو اینجا بذار
۳. اسم مدل رو وارد کن (مثلاً `google/gemma-3-1b-it:free`)
۴. تو بخش **Messaging Channels** تیک **Telegram** رو بزن و Bot Token مرحله ۲ رو پیست کن
۵. **Save & Start** رو بزن — گیتوی راه می‌افته و رباتت آنلاین می‌شه

برای اضافه کردن vLLM / Ollama / LM Studio / پراکسی خودت، [اضافه کردن پروایدر سفارشی](#-adding-a-custom-provider) رو پایین ببین.

</div>

#### Step 8 — Approve yourself and start chatting

1. Send a message to your Telegram bot
2. Back in the dashboard, go to the **Users** tab — you'll see a pending pairing request
3. Click **Approve**
4. Done — the bot now replies

<div dir="rtl">

#### مرحله ۸ — تایید خودت و شروع چت

۱. به ربات تلگرامت یه پیام بفرست
۲. تو پنل، به تب **Users** برو — یه درخواست pending می‌بینی
۳. **Approve** رو بزن
۴. تمام — ربات حالا جواب می‌ده

</div>

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
