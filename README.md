# AI News Aggregator

**An AI-powered platform that automatically collects, processes, summarizes, ranks, and delivers AI industry news through personalized daily email digests.**

![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI_GPT--4-412991?logo=openai&logoColor=white)
![Render](https://img.shields.io/badge/Deployed_on-Render-46E3B7?logo=render&logoColor=white)

---

## Overview

AI research moves faster than any individual can track. News, model releases, API updates, and research papers are scattered across blogs, RSS feeds, and YouTube channels — making it easy to miss critical developments.

This platform solves that problem with a fully automated, end-to-end pipeline:

- **Collects** articles from OpenAI, Anthropic, and AI-focused YouTube channels
- **Extracts** YouTube video transcripts automatically
- **Summarizes** each article using OpenAI GPT-4 with relevance scoring
- **Ranks** the top 10 most important articles daily
- **Delivers** a formatted HTML email digest via Gmail SMTP
- **Runs autonomously** on Render with scheduled cron execution — zero manual intervention

The system is **deployed in production** and delivers daily digests automatically.

---

## Sample Daily Digest

The platform generates a professional HTML email digest delivered to the user's inbox every morning:

![Daily AI News Digest](example_of_use.png)

**How it works:**
- The pipeline runs daily at 5:00 AM UTC via a Render cron job
- Articles are scored and ranked by relevance to the user's AI interests
- Each article includes an LLM-generated summary highlighting key business implications
- The top 10 articles are formatted into a clean HTML email and delivered via Gmail SMTP

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     RENDER CLOUD (Cron)                      │
│                   Daily at 5:00 AM UTC                       │
└──────────────────────────┬──────────────────────────────────┘
                           │
            ┌──────────────▼──────────────┐
            │      1. DATA COLLECTION      │
            └──────────────┬──────────────┘
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
   ┌─────────────┐ ┌─────────────┐ ┌──────────────┐
   │  Anthropic   │ │   OpenAI    │ │   YouTube    │
   │  Blog (RSS)  │ │ Blog (Web)  │ │  Channels    │
   └──────┬──────┘ └──────┬──────┘ └──────┬───────┘
          │               │               │
          └───────────────┼───────────────┘
                          ▼
            ┌─────────────────────────┐
            │   2. PostgreSQL Database │
            │  Articles · Transcripts  │
            │        · Digests         │
            └────────────┬────────────┘
                         │
            ┌────────────▼────────────┐
            │   3. AI PROCESSING       │
            │                          │
            │   OpenAI GPT-4           │
            │   • Summarization        │
            │   • Relevance Scoring    │
            │                          │
            │   Digest Generator       │
            │   • Top-N Ranking        │
            │   • HTML Formatting      │
            └────────────┬────────────┘
                         │
            ┌────────────▼────────────┐
            │   4. EMAIL DELIVERY      │
            │   Gmail SMTP             │
            └────────────┬────────────┘
                         │
            ┌────────────▼────────────┐
            │   5. END USER INBOX      │
            └─────────────────────────┘
```

### Pipeline Stages

| Stage | Component | Description |
|-------|-----------|-------------|
| 1 | **Scrapers** | Collect articles from Anthropic (RSS), OpenAI (web scraping), and YouTube (API) |
| 2 | **Database** | Store articles, transcripts, and generated digests in PostgreSQL |
| 3 | **AI Processing** | Summarize articles and score relevance using OpenAI GPT-4, then rank and format the top results |
| 4 | **Email Delivery** | Generate an HTML email digest and send via Gmail SMTP |
| 5 | **Delivery** | Curated daily digest arrives in the user's inbox |

---

## Features

- **Multi-source collection** — Anthropic blog (RSS), OpenAI blog (web scraper), YouTube AI channels (transcript extraction)
- **LLM-powered summarization** — OpenAI GPT-4 generates concise, business-relevant summaries for each article
- **Intelligent ranking** — Articles scored and ranked by relevance to the user's AI interests
- **Automated email delivery** — Professional HTML digest delivered daily via Gmail SMTP
- **PostgreSQL persistence** — All articles, transcripts, and digests stored for history and deduplication
- **Docker containerization** — Consistent environment across development and production
- **Cloud deployment** — Fully deployed on Render with managed PostgreSQL and scheduled cron execution
- **Modular architecture** — Scrapers, processors, and delivery components are independently extensible
- **Structured logging** — Full pipeline visibility with stage-by-stage progress tracking

---

## Tech Stack

| Category | Technologies |
|----------|-------------|
| **Language** | Python 3.12 |
| **Database** | PostgreSQL (Render managed) |
| **AI / LLM** | OpenAI API (GPT-4) |
| **NLP** | YouTube Transcript API |
| **Email** | Gmail SMTP |
| **Infrastructure** | Docker, Render (cron jobs) |
| **Data Sources** | RSS feeds, Web scraping, YouTube API |
| **Version Control** | Git / GitHub |

---

## Project Structure

```
ai-news-aggregator/
├── main.py                     # Entry point
├── app/
│   ├── config.py               # Environment configuration
│   ├── daily_runner.py         # 5-stage pipeline orchestrator
│   ├── runner.py               # Scraper execution coordinator
│   ├── agent/                  # AI agent layer
│   │   ├── base.py             # Base agent class
│   │   ├── curator_agent.py    # Content curation agent
│   │   ├── digest_agent.py     # Digest generation agent
│   │   └── email_agent.py      # Email composition agent
│   ├── database/               # Data persistence
│   │   ├── connection.py       # PostgreSQL connection management
│   │   ├── models.py           # SQLAlchemy models
│   │   ├── repository.py       # Data access layer
│   │   └── create_tables.py    # Schema initialization
│   ├── scrapers/               # Data collection
│   │   ├── base.py             # Base scraper class
│   │   ├── anthropic.py        # Anthropic blog scraper (RSS)
│   │   ├── openai.py           # OpenAI blog scraper (web)
│   │   └── youtube.py          # YouTube channel scraper
│   ├── services/               # Processing & delivery
│   │   ├── process_anthropic.py
│   │   ├── process_youtube.py
│   │   ├── process_digest.py   # LLM summarization & ranking
│   │   ├── process_email.py    # Email digest generation
│   │   └── email.py            # SMTP delivery
│   └── profiles/               # User interest profiles
│       └── user_profile.py
├── Dockerfile                  # Container configuration
├── render.yaml                 # Render deployment blueprint
├── pyproject.toml              # Dependencies (uv)
├── uv.lock                     # Locked dependencies
└── docker/
    └── docker-compose.yml      # Local development setup
```

---

## Deployment

The platform runs in production on **Render** with the following infrastructure:

| Component | Service | Details |
|-----------|---------|---------|
| **Application** | Render Cron Job | Runs daily at 5:00 AM UTC |
| **Database** | Render PostgreSQL | Managed instance, free tier |
| **Container** | Docker | Built from `Dockerfile` |
| **Configuration** | `render.yaml` | Infrastructure-as-code blueprint |

### Production Workflow

1. Render triggers the cron job daily at 5:00 AM UTC
2. Docker container starts and runs `python main.py`
3. The 5-stage pipeline executes: Scrape → Process → Summarize → Rank → Deliver
4. Pipeline logs are available in the Render dashboard
5. The container shuts down after completion

### Environment Variables

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | PostgreSQL connection string (auto-provisioned by Render) |
| `OPENAI_API_KEY` | OpenAI API key for GPT-4 summarization |
| `MY_EMAIL` | Recipient email address for the daily digest |
| `APP_PASSWORD` | Gmail app password for SMTP delivery |

---

## Local Development

### Prerequisites

- Python 3.12+
- [uv](https://docs.astral.sh/uv/) package manager
- PostgreSQL (or use Docker Compose)
- OpenAI API key
- Gmail account with [app password](https://support.google.com/accounts/answer/185833)

### Setup

```bash
# Clone the repository
git clone https://github.com/SvetaOnopa/ai-news-aggregator.git
cd ai-news-aggregator

# Install dependencies
uv sync

# Configure environment
cp app/example.env .env
# Edit .env with your credentials

# Run with Docker Compose (includes PostgreSQL)
docker compose -f docker/docker-compose.yml up

# Or run directly
uv run python main.py
```

### Configuration

Copy `app/example.env` to `.env` and configure:

```env
DATABASE_URL=postgresql://user:password@localhost:5432/ai_news_aggregator
OPENAI_API_KEY=sk-...
MY_EMAIL=your-email@gmail.com
APP_PASSWORD=your-gmail-app-password
```

---

## My Contributions

This project was built and deployed end-to-end with the following work:

- **Environment setup** — Python environment configuration, dependency management with uv, and `.env` configuration
- **Docker containerization** — Dockerfile creation and optimization for production deployment
- **PostgreSQL configuration** — Database setup, connection management, and schema design
- **OpenAI API integration** — GPT-4 integration for article summarization and relevance scoring
- **Email delivery system** — Gmail SMTP configuration, HTML template design, and delivery logic
- **Cloud deployment** — Full Render deployment with managed PostgreSQL, cron job scheduling, and infrastructure-as-code via `render.yaml`
- **Debugging & troubleshooting** — End-to-end debugging across Docker builds, database connections, API integrations, and email delivery in production
- **Git workflow** — Branch management, merge operations, and production release strategy
- **Pipeline validation** — Full end-to-end testing from data collection through email delivery, verifying each pipeline stage independently

---

## License

This project is for portfolio and demonstration purposes.
