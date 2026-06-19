# AI News Aggregator — Complete Beginner's Learning Guide

> **Who this is for:** Svetlana, and anyone learning Python and backend development by following this project. You do not need prior experience with databases, Docker, deployment, or APIs. Every concept is explained from scratch before it is used.

> **How to read this guide:** Read it in order the first time. Later, use the table of contents to jump to any topic you need.

---

## Table of Contents

- [Part 0: Before You Begin](#part-0-before-you-begin)
- [Part 1: Project Overview (Plain English)](#part-1-project-overview)
- [Part 2: Architecture Explained](#part-2-architecture-explained)
- [Part 3: Key Concepts for Beginners](#part-3-key-concepts-for-beginners)
- [Part 4: Beginner Setup Guide](#part-4-beginner-setup-guide)
- [Part 5: File-by-File Explanation](#part-5-file-by-file-explanation)
- [Part 6: Function-by-Function Explanation](#part-6-function-by-function-explanation)
- [Part 7: Step-by-Step Build Roadmap (12 Lessons)](#part-7-step-by-step-build-roadmap)
- [Part 8: Environment Variables Deep Dive](#part-8-environment-variables-deep-dive)
- [Part 9: Docker Explained for This Project](#part-9-docker-explained-for-this-project)
- [Part 10: Database Deep Dive](#part-10-database-deep-dive)
- [Part 11: Branch Comparison](#part-11-branch-comparison)
- [Part 12: Deployment to Render.com (Hands-On)](#part-12-deployment-to-rendercom)
- [Part 13: Debugging Guide](#part-13-debugging-guide)
- [Part 14: Learning Checkpoints](#part-14-learning-checkpoints)
- [Part 15: Final Reference](#part-15-final-reference)

---

## Part 0: Before You Begin

### What You Will Build

By the end of this guide, you will have a working system that:

1. **Automatically collects** the latest AI news every day from three sources:
   - YouTube (specific channels you choose)
   - The OpenAI blog
   - The Anthropic blog

2. **Saves** all collected content to a local database

3. **Enriches** the content by downloading full article text and video transcripts

4. **Summarizes** every article using OpenAI's GPT model (so you get a 2–3 sentence digest instead of reading the full article)

5. **Ranks** the summaries based on your personal interests and background

6. **Emails** you a beautiful, personalized HTML digest each morning with the top 10 most relevant articles

7. **Runs automatically in the cloud** on a schedule, with no computer required on your part

### What You Will Learn

- How to structure a real Python backend project
- How to use a relational database (PostgreSQL) from Python
- How to run services locally using Docker
- How to call the OpenAI API
- How to scrape web data with RSS feeds
- How to send email programmatically
- How to use environment variables for secrets
- How to deploy a Python project to the cloud (Render.com)
- How to use `uv`, a modern Python package manager

### What You Need Installed (We Will Install These Together in Part 4)

| Tool | What It Is | Why You Need It |
|------|-----------|----------------|
| Python 3.12 | The programming language | Runs all the code |
| uv | Package manager | Installs dependencies |
| VS Code | Code editor | Where you write and read code |
| Docker Desktop | Container runner | Runs your local database |
| Git | Version control | Already on your machine (came with the repo) |

### How to Use This Guide

- **Callout boxes** like the one below mark places where the original video moves fast or skips an explanation:

> 💡 **Video Gap:** This concept was not explained in the video. Read this section carefully before moving on.

- **Code blocks** show exact commands or code. Type them exactly as shown (or copy/paste).
- **"What you should see"** sections tell you what success looks like.
- **Learning Checkpoints** at the end of each major part let you self-test.

---

## Part 1: Project Overview

### What This Project Does (Plain English)

Imagine you want to stay informed about AI news, but:
- There are dozens of YouTube channels posting daily
- OpenAI and Anthropic publish blog posts regularly
- You don't have time to check all of them every day
- You want to read only the most relevant things for your interests

This project solves that problem. It is an **automated daily newsletter** that:
1. Collects everything published in the last 24 hours
2. Uses AI to summarize each piece of content
3. Uses AI to rank the summaries by how relevant they are *to you specifically*
4. Emails you the top 10 most important things to read — every morning

The key word is **automated**. Once set up, it runs by itself on a schedule. You never have to start it manually.

### What Problem It Solves

**The problem:** There is too much AI news to track manually. Checking YouTube, two blogs, and Twitter every day takes 30–60 minutes and you still miss things.

**The solution:** A personalized, automated digest that reads everything so you don't have to, and delivers only what matters to you.

### What the Final Result Looks Like

Every morning you receive an email that looks like this:

```
Subject: Daily AI News Digest - June 19, 2026

Hey Svetlana, here is your daily digest of AI news for June 19, 2026.

Today's top stories cover a major new model release from Anthropic,
a practical guide to building RAG systems, and three important
research papers on AI alignment. Here are your top 10 picks:

---

## Claude 4 Sets New Benchmark in Reasoning Tasks

Anthropic's latest model achieves a 15% improvement over previous
versions on complex multi-step reasoning. The key innovation is a
new training technique called...

[Read more →](https://anthropic.com/...)

---

## How to Build a Production RAG System in 2026

This YouTube tutorial walks through building a retrieval-augmented
generation system from scratch, covering vector databases,
chunking strategies, and...

[Read more →](https://youtube.com/...)
```

### How the Pipeline Works, Step by Step

Here is exactly what happens when the project runs:

```
STEP 0: Start up
  └─ Python script begins (main.py → daily_runner.py)
  └─ Load secrets from .env file
  └─ Connect to the database

STEP 1: Scraping (runner.py)
  ├─ YouTube: fetch RSS feed for each channel
  │   └─ Get list of videos published in last 24 hours
  │   └─ Save video metadata to database
  ├─ OpenAI: fetch RSS feed from openai.com/news/rss.xml
  │   └─ Get articles published in last 24 hours
  │   └─ Save to database
  └─ Anthropic: fetch 3 RSS feeds (news, research, engineering)
      └─ Get articles published in last 24 hours
      └─ Save to database

STEP 2: Enrich Anthropic articles (process_anthropic.py)
  └─ For each Anthropic article that has no full text yet:
      └─ Download the full article from its URL
      └─ Convert it to Markdown text
      └─ Save the Markdown back to the database

STEP 3: Enrich YouTube videos (process_youtube.py)
  └─ For each YouTube video that has no transcript yet:
      └─ Download the auto-generated transcript from YouTube
      └─ Save the transcript text to the database

STEP 4: Generate AI summaries (process_digest.py)
  └─ For each article/video that does not have a summary yet:
      └─ Send title + content to GPT-4o-mini
      └─ Receive a 2-3 sentence summary
      └─ Save the summary (called a "digest") to the database

STEP 5: Rank, write, and send email (process_email.py)
  ├─ Get all summaries from the last 24 hours
  ├─ Send them all to GPT-4.1 (CuratorAgent)
  │   └─ AI ranks them 1–N based on your personal profile
  ├─ Take the top 10 ranked summaries
  ├─ Send to GPT-4o-mini (EmailAgent)
  │   └─ AI writes a personalized greeting and introduction
  └─ Send the full email via Gmail SMTP
```

> 💡 **Video Gap:** The video jumps between these steps quickly. Notice that STEPS 1–5 are separate, and each step reads from and writes to the **same database**. This is how the pipeline "passes data" between steps — not by passing Python variables, but through the database. This is a real-world pattern called a **data pipeline**.

---

## Part 2: Architecture Explained

### Full Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│                    EXTERNAL DATA SOURCES                          │
│                                                                   │
│  YouTube RSS Feed    OpenAI RSS Feed    Anthropic RSS Feeds (×3) │
│  youtube.com/feeds   openai.com/news   github.com/Olshansk/...   │
└──────────┬───────────────┬──────────────────┬────────────────────┘
           │               │                  │
           ▼               ▼                  ▼
┌──────────────────────────────────────────────────────────────────┐
│                      SCRAPERS (app/scrapers/)                     │
│                                                                   │
│   youtube.py          openai.py           anthropic.py           │
│   - Parse RSS         - Parse RSS         - Parse 3 RSS feeds    │
│   - Get video IDs     - Get articles      - Get articles         │
│   - Filter by date    - Filter by date    - Filter by date       │
└──────────────────────────┬───────────────────────────────────────┘
                           │ save raw data
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│               DATABASE (PostgreSQL via Docker)                    │
│                                                                   │
│   youtube_videos      openai_articles     anthropic_articles     │
│   ─────────────       ───────────────     ─────────────────      │
│   video_id (PK)       guid (PK)           guid (PK)             │
│   title               title               title                  │
│   url                 url                 url                    │
│   transcript ◄──┐     description         description            │
│   description   │     published_at        markdown ◄──┐          │
│   published_at  │     category            published_at│          │
│                 │     created_at          category    │          │
│                 │                         created_at  │          │
│                 │                                     │          │
│   ┌─────────────┘                         ┌───────────┘          │
│   │ process_youtube.py                    │ process_anthropic.py │
│   │ (fetch YouTube transcripts)           │ (fetch full articles)│
│   │                                       │                      │
│   └──────────────┐          ┌─────────────┘                      │
│                  ▼          ▼                                     │
│               digests                                            │
│               ───────                                            │
│               id (type:article_id)                               │
│               title (AI-generated)                               │
│               summary (AI-generated, 2-3 sentences)             │
│               url                                                │
│               article_type                                       │
│               created_at                                         │
└────────────────────────────┬─────────────────────────────────────┘
                             │ read summaries
                             ▼
┌──────────────────────────────────────────────────────────────────┐
│                   AI AGENTS (app/agent/)                          │
│                                                                   │
│  digest_agent.py          curator_agent.py      email_agent.py  │
│  ─────────────────        ───────────────────   ──────────────  │
│  GPT-4o-mini              GPT-4.1               GPT-4o-mini     │
│  Reads raw content        Ranks summaries       Writes intro    │
│  Writes 2-3 sentence      by relevance to       and greeting    │
│  summary + title          your profile          for the email   │
└──────────────────────────────┬───────────────────────────────────┘
                               │ top 10 ranked articles
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│                   EMAIL (app/services/email.py)                   │
│                                                                   │
│   Convert Markdown → HTML                                        │
│   Send via Gmail SMTP (smtplib)                                  │
│   Recipient: your Gmail address                                  │
└──────────────────────────────────────────────────────────────────┘
                               │
                               ▼
                    📧 YOUR INBOX (daily email digest)
```

### What Each Folder Is Responsible For

| Folder | Responsibility | Analogy |
|--------|---------------|---------|
| `app/scrapers/` | Collects raw data from the internet | The "librarian" who gathers books |
| `app/database/` | Stores and retrieves all data | The "filing cabinet" |
| `app/services/` | Orchestrates multi-step processes | The "manager" who coordinates tasks |
| `app/agent/` | Makes AI API calls | The "AI assistant" who reads and writes |
| `app/profiles/` | Stores your personal preferences | Your "reader profile" |

### How the Files Connect

```
main.py
  └─ imports daily_runner.py
       ├─ imports runner.py (step 1: scraping)
       │    ├─ imports app/scrapers/youtube.py
       │    ├─ imports app/scrapers/openai.py
       │    ├─ imports app/scrapers/anthropic.py
       │    └─ imports app/database/repository.py
       ├─ imports services/process_anthropic.py (step 2)
       │    ├─ imports scrapers/anthropic.py
       │    └─ imports database/repository.py
       ├─ imports services/process_youtube.py (step 3)
       │    ├─ imports scrapers/youtube.py
       │    └─ imports database/repository.py
       ├─ imports services/process_digest.py (step 4)
       │    ├─ imports agent/digest_agent.py
       │    └─ imports database/repository.py
       └─ imports services/process_email.py (step 5)
            ├─ imports agent/curator_agent.py
            ├─ imports agent/email_agent.py
            ├─ imports profiles/user_profile.py
            ├─ imports database/repository.py
            └─ imports services/email.py
```

Every file that needs the database imports `database/repository.py`. The repository is the single place where all database reads and writes happen.

---

## Part 3: Key Concepts for Beginners

Before writing or running any code, you need to understand the building blocks. Each concept below is explained first in plain English, then in the context of this project.

---

### Concept 1: Python Virtual Environment

**What is it (plain English)?**

When you install Python packages (like `requests` or `openai`), they are installed somewhere on your computer. The problem: if you have two Python projects that need *different versions* of the same package, they will conflict.

A **virtual environment** is an isolated folder that contains its own copy of Python and all packages. It is like a separate apartment for each project — what happens inside one apartment does not affect another.

**How this project handles it:**

This project uses `uv` (explained next) which manages the virtual environment automatically. When you run `uv sync`, it creates a `.venv` folder in the project directory. That folder contains everything the project needs, isolated from the rest of your system.

**What would break without it:**

If you install packages globally (without a virtual environment), two projects on your computer could end up with conflicting package versions. One project might need `openai==1.0` and another `openai==2.0`, which cannot coexist globally.

---

### Concept 2: uv — Modern Python Package Manager

**What is it (plain English)?**

`uv` is a tool that does two things:
1. Manages virtual environments (the isolated "apartment" for your project)
2. Installs and tracks Python packages (like `openai`, `feedparser`, etc.)

It is similar to the older tool called `pip`, but much faster and smarter.

**Key analogy:** `pip` is like manually shopping at a supermarket — you pick items one by one. `uv` is like ordering a meal kit — you say "I want this recipe" and it brings you exactly the right ingredients, in the right amounts, ready to use.

**Why this project uses uv instead of pip:**
- It installs packages much faster (written in Rust, not Python)
- It reads `pyproject.toml` to know what the project needs
- It creates a `uv.lock` file with exact version numbers, so everyone using the project gets the exact same packages

**Key files:**
- `pyproject.toml` — lists what packages the project needs (like a grocery list)
- `uv.lock` — records exactly which version of each package was installed (like a receipt)

> 💡 **Video Gap:** The video uses `uv` from the start without explaining it. If you have only used `pip` before, just know that `uv sync` does what `pip install -r requirements.txt` does — but better.

---

### Concept 3: What Is a Database?

**What is it (plain English)?**

A database is an organized place to store information so you can find it again later. You can think of it like a very organized Excel spreadsheet — data is stored in **tables** (like sheets), and each table has **columns** (like column headers) and **rows** (like individual entries).

**Why this project needs one:**

The pipeline runs in 5 stages, and each stage needs access to data from the previous stage. Instead of passing data directly between Python functions (which only works while the program is running), the project saves data to a database. This way:
- If one stage fails, you do not lose the data from earlier stages
- You can run stages independently for testing
- Data persists between runs (yesterday's articles are still there today)

**Without the database:**
- All collected articles would disappear when the script finishes
- You could not re-summarize articles without re-downloading them
- You could not track which articles have already been emailed

---

### Concept 4: PostgreSQL — The Database This Project Uses

**What is it (plain English)?**

PostgreSQL (often called "Postgres") is one of the most popular and powerful databases in the world. It is free, open source, and used by companies like Instagram, Reddit, and Spotify.

It stores data in tables with rows and columns. You communicate with it using a language called SQL (Structured Query Language), though in this project, Python handles the SQL for you automatically.

**How this project uses it:**

This project has 4 tables in PostgreSQL:
1. `youtube_videos` — stores YouTube video metadata and transcripts
2. `openai_articles` — stores OpenAI blog articles
3. `anthropic_articles` — stores Anthropic blog articles (plus full markdown text)
4. `digests` — stores AI-generated summaries of all the above

**Local vs. cloud:**
- When developing locally, PostgreSQL runs on your computer inside Docker (explained next)
- When deployed to Render.com, PostgreSQL runs on Render's servers

---

### Concept 5: Docker — Running Software in Containers

**What is it (plain English)?**

Normally, to run PostgreSQL on your computer, you would need to install it, configure it, set up a user, a password, etc. This takes time and can be confusing.

**Docker** lets you run software inside a **container** — a self-contained, pre-configured package. Think of it like a shipping container: no matter where it goes, it carries everything it needs inside. A PostgreSQL container already has PostgreSQL installed, configured, and ready to use.

**Key terms:**

| Term | Plain English | Analogy |
|------|--------------|---------|
| **Image** | A blueprint or recipe for creating a container | A recipe for a dish |
| **Container** | A running instance created from an image | The actual dish you made |
| **Docker Compose** | A tool for defining and running multiple containers from a single file | The meal plan that says "make this dish with these settings" |
| **Port** | A numbered "door" on your computer where services listen | Like a building's apartment number |
| **Volume** | A folder that persists even when the container stops | A USB drive that you plug in and out |

**In this project:**

The file `docker/docker-compose.yml` tells Docker:
- Use the PostgreSQL 17 image
- Create a container named `ai-news-aggregator-db`
- Expose it on port `5432` (the standard PostgreSQL port)
- Save all database data to a volume called `postgres_data` (so your data survives if the container restarts)

**Why port 5432 matters:**

When your Python code connects to the database, it sends a request to `localhost:5432`. `localhost` means "my own computer." `5432` is the port where PostgreSQL is listening. This is like calling an internal phone number — you know exactly which extension (port) to dial.

> 💡 **Video Gap:** The video shows `docker compose up -d` but does not explain what `-d` means. It stands for "detached mode" — it runs the container in the background so your terminal stays free. Without `-d`, the container would print logs directly to your terminal and block it.

---

### Concept 6: SQLAlchemy — The Bridge Between Python and PostgreSQL

**What is it (plain English)?**

Your Python code does not speak SQL directly. PostgreSQL does not speak Python. SQLAlchemy is a **translator** that sits between them.

Without SQLAlchemy, saving a video to the database would look like:
```sql
INSERT INTO youtube_videos (video_id, title, url, ...) VALUES ('abc123', 'My Video', 'https://...', ...);
```

With SQLAlchemy, you write Python instead:
```python
video = YouTubeVideo(video_id='abc123', title='My Video', url='https://...')
session.add(video)
session.commit()
```

SQLAlchemy translates your Python into SQL and sends it to the database for you.

**Key concepts in SQLAlchemy used in this project:**

| Concept | What It Is | Where You See It |
|---------|-----------|-----------------|
| **Model** | A Python class that represents a database table | `app/database/models.py` |
| **Engine** | The connection to the database | `app/database/connection.py` |
| **Session** | A "conversation" with the database (like a shopping cart — add items, then commit) | `Repository.__init__` |
| **Base** | The parent class all models inherit from | `models.py`: `Base = declarative_base()` |
| `create_all()` | Creates all tables from your models | `create_tables.py` |

**ORM:** SQLAlchemy is an ORM — an **Object-Relational Mapper**. "Object" = Python object. "Relational" = relational database (tables). "Mapper" = it maps one to the other.

---

### Concept 7: What Is an RSS Feed?

**What is it (plain English)?**

An RSS feed is a special web page that lists recent updates from a website in a machine-readable format (XML). Instead of scraping a website's HTML (which is messy and breaks often), you read the RSS feed and get clean, structured data.

Most blogs and YouTube channels publish RSS feeds. They look like:
```xml
<item>
  <title>Claude 4 is here</title>
  <link>https://anthropic.com/news/claude-4</link>
  <pubDate>Thu, 19 Jun 2026 09:00:00 +0000</pubDate>
  <description>Today we're releasing Claude 4...</description>
</item>
```

**How this project uses RSS:**

All three scrapers use `feedparser` — a Python library that reads RSS feeds and returns the data as Python objects. Instead of complex web scraping, you just call `feedparser.parse(url)` and get a list of articles.

**YouTube RSS:** YouTube provides a per-channel RSS feed at:
```
https://www.youtube.com/feeds/videos.xml?channel_id=CHANNEL_ID_HERE
```

This gives you all recent videos from that channel, with titles, URLs, and publish dates.

> 💡 **Video Gap:** The video does not explain why `feedparser` is used instead of `requests` + `BeautifulSoup` (a common web scraping combo). RSS feeds are **much more reliable** — they are designed to be machine-read, they have consistent structure, and websites don't break them when they redesign their layout.

---

### Concept 8: What Is an API Key?

**What is it (plain English)?**

An API key is a secret password that identifies you when you call an external service like OpenAI.

When this project calls `gpt-4o-mini` to summarize an article, OpenAI needs to know:
- Who is making this request?
- Are they allowed to use this service?
- How many requests have they made? (for billing)

The API key answers all three questions. It is like a membership card that you show at the door.

**Important rules:**
1. **Never share your API key** with anyone
2. **Never commit your API key to Git** (put it in `.env`, which is gitignored)
3. Each API call costs money — OpenAI charges per token (words processed)

**Where to get an OpenAI API key:** At platform.openai.com → API Keys → Create new secret key

---

### Concept 9: What Is a .env File?

**What is it (plain English)?**

A `.env` file (pronounced "dot env") is a plain text file where you store **secrets and configuration values** for your project. Each line contains one key-value pair:

```
OPENAI_API_KEY=sk-proj-abc123...
MY_EMAIL=svetlana@gmail.com
POSTGRES_PASSWORD=mypassword
```

**Why not just put these values in the Python code?**

Imagine you write `api_key = "sk-proj-abc123"` directly in your Python file, and then push that file to GitHub. Now your API key is **public** and anyone can use it (and charge to your account). People have been billed thousands of dollars because of this mistake.

The `.env` file solves this by:
1. Keeping secrets in a file that is **never committed to Git** (it's in `.gitignore`)
2. Loading them into memory at runtime with `load_dotenv()`
3. Keeping your code free of secrets so it can be shared safely

**How it works in Python:**

```python
from dotenv import load_dotenv
import os

load_dotenv()  # reads .env and loads all values into environment variables

api_key = os.getenv("OPENAI_API_KEY")  # retrieves the value
```

> 💡 **Video Gap:** The video calls `load_dotenv()` without explaining that it must be called **before** any `os.getenv()` call. If you call `os.getenv()` before `load_dotenv()`, you will get `None` instead of your actual value. Order matters!

---

### Concept 10: What Is an AI Agent (in This Project)?

**What is it (plain English)?**

In this project, "agent" is used to describe a Python class that wraps a call to the OpenAI API. Each agent has a specific job:

| Agent | File | Model | Job |
|-------|------|-------|-----|
| `DigestAgent` | `agent/digest_agent.py` | gpt-4o-mini | Reads an article, writes a 2–3 sentence summary |
| `CuratorAgent` | `agent/curator_agent.py` | gpt-4.1 | Reads all summaries, ranks them by relevance to your profile |
| `EmailAgent` | `agent/email_agent.py` | gpt-4o-mini | Reads the top 10 ranked articles, writes a personalized intro |

**Why different models?**

- `gpt-4o-mini` is cheaper and fast — good for simple summarization (many articles)
- `gpt-4.1` is more powerful and better at complex reasoning — good for ranking

> 💡 **Video Gap:** You may notice `temperature=0.3` for the curator and `temperature=0.7` for the digest and email agents. **Temperature** controls how "creative" vs "predictable" the AI is. A low temperature (0.3) makes the AI more consistent and analytical — good for ranking. A higher temperature (0.7) makes it slightly more creative — good for writing engaging email intros.

---

### Concept 11: Pydantic — Structured, Validated Data in Python

**What is it (plain English)?**

Pydantic is a Python library that lets you define the **shape** of your data and automatically validates that the data matches. It is like a form with required fields — if you try to submit without filling in a required field, it stops you immediately.

In this project, Pydantic is used extensively:

```python
from pydantic import BaseModel

class ChannelVideo(BaseModel):
    title: str        # must be a string
    url: str          # must be a string
    video_id: str     # must be a string
    published_at: datetime  # must be a datetime object
    description: str
    transcript: Optional[str] = None  # can be string or None, defaults to None
```

**Why use Pydantic instead of a plain dictionary?**

```python
# Without Pydantic (fragile):
video = {"title": "My Video", "url": "https://...", "published_at": "2026-06-19"}
# No validation — "published_at" is a string here, not a datetime
# If you try to compare it to another datetime, you'll get an error

# With Pydantic (safe):
video = ChannelVideo(title="My Video", url="https://...", published_at=datetime(2026, 6, 19))
# Validated — if you pass the wrong type, you get a clear error immediately
```

**`Optional[str]`** means "either a string or None." This is used for fields that might not have a value — like `transcript`, because some videos don't have transcripts.

---

## Part 4: Beginner Setup Guide

This section walks you through every setup step. Do these **in order**. Do not skip steps.

---

### Step 1: Check Your Python Version

**What we are doing:** Confirming Python 3.12 is installed (this project requires it).

**Where to run:** In the Terminal (on Mac: press `Cmd + Space`, type "Terminal", press Enter)

```bash
python3 --version
```

**What you should see:**
```
Python 3.12.x
```

**If you see Python 3.10 or 3.11:** You need to install Python 3.12.
Go to python.org → Downloads → Download Python 3.12.x → Run the installer.

**If you see "command not found":** Python is not installed. Go to python.org → Downloads.

**How to verify success:** The version number starts with `3.12`.

---

### Step 2: Install uv

**What we are doing:** Installing `uv`, the tool that will manage the project's packages.

**Where to run:** In your Terminal

**On Mac/Linux:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**What this command does:** Downloads and runs an installer script from astral.sh (the company that makes uv).

**After installing, restart your terminal** (close and reopen it), then verify:

```bash
uv --version
```

**What you should see:**
```
uv 0.x.x
```

**How to verify success:** The command prints a version number without errors.

> **What is `curl`?** It is a command-line tool that downloads files from the internet. Think of it as "wget" or a command-line browser.

---

### Step 3: Install VS Code and the Python Extension

**What we are doing:** Installing the code editor you will use to read and write code.

1. Go to code.visualstudio.com → Download → Install
2. Open VS Code
3. Click the **Extensions** icon on the left sidebar (it looks like four squares)
4. Search for **"Python"** (by Microsoft)
5. Click **Install**

**Why VS Code?** It has excellent Python support: syntax highlighting, code completion, built-in terminal, and easy navigation between files.

**How to open a terminal in VS Code:** Menu → Terminal → New Terminal (or press `` Ctrl+` `` on Windows/Linux, `` Cmd+` `` on Mac)

---

### Step 4: Install Docker Desktop

**What we are doing:** Installing Docker so we can run PostgreSQL locally without installing it manually.

1. Go to docker.com/products/docker-desktop
2. Download the version for your operating system (Mac M1/M2 chip or Intel/AMD)
3. Run the installer
4. Open Docker Desktop and complete any first-time setup steps
5. You should see a green "Engine running" status in the bottom left

**How to verify Docker is working:**

Open your Terminal and run:
```bash
docker --version
```

**What you should see:**
```
Docker version 27.x.x, ...
```

**How to verify Docker Desktop is running:** The Docker icon should be visible in your system tray/menu bar with a green dot.

> **Important:** Docker Desktop must be **running** (the application open) every time you start working on this project locally. If Docker is not running, the database will not start.

---

### Step 5: Open the Project in VS Code

**What we are doing:** Opening the cloned project folder in VS Code so we can see and edit all the files.

**Option A: From the Terminal**

```bash
cd /Users/svetlanaonopa/Desktop/Projects/AI_Project/Git_repo_clone/ai-news-aggregator
code .
```

**What `cd` does:** Changes your current directory (folder). Like clicking into a folder in Finder.

**What `code .` does:** Opens VS Code with the current folder (`.` means "current folder").

**Option B: From VS Code**

1. Open VS Code
2. File → Open Folder
3. Navigate to the `ai-news-aggregator` folder
4. Click Open

**What you should see:** VS Code opens with the file tree on the left showing all the project files.

---

### Step 6: Select the Correct Python Interpreter in VS Code

**What we are doing:** Telling VS Code which Python installation to use for this project.

1. Open any `.py` file in VS Code (e.g., `main.py`)
2. Look at the bottom-left corner — you should see "Python 3.x.x"
3. Click on it
4. A list appears at the top of VS Code
5. Select "Python 3.12.x" (or the version from `.venv` once we create it in Step 8)

**Why this matters:** If VS Code uses the wrong Python version, it may show false errors and the terminal inside VS Code may run the wrong Python.

---

### Step 7: Create Your .env File

**What we are doing:** Creating a file with your personal secrets so the project can connect to APIs and the database.

The project includes a template called `example.env` inside the `app/` folder. You will copy it and fill in your values.

**Step 7a: Copy the template**

In your terminal (inside VS Code: Terminal → New Terminal):
```bash
cp app/example.env .env
```

**What `cp` does:** Copies a file. `cp source destination` — copies `app/example.env` to `.env` in the project root.

**Step 7b: Open the .env file**

In VS Code, look for `.env` in the file tree and click on it. You should see:

```
OPENAI_API_KEY=
MY_EMAIL=
APP_PASSWORD=
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=ai_news_aggregator
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
```

**Step 7c: Fill in your values**

```
OPENAI_API_KEY=sk-proj-your-key-here
MY_EMAIL=svetlana@gmail.com
APP_PASSWORD=your-16-char-app-password
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=ai_news_aggregator
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
```

**How to get your OpenAI API Key:**
1. Go to platform.openai.com
2. Sign in (create an account if you don't have one)
3. Click your profile → API Keys → Create new secret key
4. Copy the key immediately — you will not be able to see it again
5. Add some credit to your account (Settings → Billing) — even $5 is enough to run this project many times

**How to get a Gmail App Password:**

Gmail requires a special 16-character password for SMTP access (regular password does not work).

1. Go to myaccount.google.com
2. Click **Security**
3. Enable **2-Step Verification** if not already on (required)
4. Search for "App passwords" in the search bar on the security page
5. Create a new app password → choose "Mail" → "Mac" (or your device)
6. Copy the 16-character password shown (it looks like: `abcd efgh ijkl mnop`)
7. Remove the spaces when pasting into `.env`: `APP_PASSWORD=abcdefghijklmnop`

> 💡 **Video Gap:** The video briefly mentions App Passwords but does not explain *why* you need them. Google disabled direct password access for SMTP in 2022 for security reasons. "App Passwords" are separate credentials specifically for applications like this one.

**Save the file:** Cmd+S (Mac) or Ctrl+S (Windows/Linux)

**What should NEVER be committed to Git:**

The `.gitignore` file already includes `.env`, which means Git will never track it. But be careful:
- Never rename it to something like `env.txt` or `secrets.py`
- Never paste your API key into any Python file
- Never paste your API key into a commit message

---

### Step 8: Install Project Dependencies with uv

**What we are doing:** Installing all the Python packages this project needs (like `openai`, `feedparser`, `sqlalchemy`, etc.)

**Where to run:** In your terminal, inside the project folder

```bash
uv sync
```

**What this command does:**
1. Reads `pyproject.toml` to see what packages are needed
2. Reads `uv.lock` to see exactly which versions to install
3. Creates a `.venv` folder in the project (the virtual environment)
4. Installs all packages into `.venv`

**What you should see:**
```
Resolved 147 packages in 0.5ms
Installed 147 packages in 2.3s
...
```

The exact numbers will vary, but it should complete without red error messages.

**How to verify success:**
```bash
uv run python -c "import openai; print('openai OK')"
uv run python -c "import feedparser; print('feedparser OK')"
uv run python -c "import sqlalchemy; print('sqlalchemy OK')"
```

Each command should print the "OK" message without errors.

> **What is `uv run`?** It runs a command inside the virtual environment. `uv run python` runs Python using the packages installed in `.venv`. Without `uv run`, your system Python might run instead — and it would not have the project's packages installed.

**After this step, tell VS Code about the new virtual environment:**
1. Press Cmd+Shift+P (Mac) or Ctrl+Shift+P (Windows)
2. Type "Python: Select Interpreter"
3. Choose the one that shows `.venv` — something like `Python 3.12.x ('.venv': uv)`

---

### Step 9: Start the PostgreSQL Database with Docker

**What we are doing:** Starting a PostgreSQL database in a Docker container so the project has somewhere to store data.

**Make sure Docker Desktop is open and running first.**

```bash
docker compose -f docker/docker-compose.yml up -d
```

**What this command does:**
- `docker compose` — runs Docker Compose
- `-f docker/docker-compose.yml` — uses the compose file at that path
- `up` — starts the services defined in the file
- `-d` — detached mode (runs in background, frees your terminal)

**What you should see:**
```
[+] Running 2/2
 ✔ Network ai-news-aggregator_default  Created
 ✔ Container ai-news-aggregator-db     Started
```

**How to verify the database is running:**

```bash
docker ps
```

**What you should see:**
```
CONTAINER ID   IMAGE         COMMAND                  CREATED        STATUS                   PORTS                    NAMES
abc123def456   postgres:17   "docker-entrypoint.s…"   5 seconds ago  Up 4 seconds (healthy)   0.0.0.0:5432->5432/tcp   ai-news-aggregator-db
```

The key thing: **STATUS** should say `Up` and eventually `(healthy)`.

**How to stop the database (when you are done for the day):**
```bash
docker compose -f docker/docker-compose.yml down
```

> **Important:** Stopping the container does NOT delete your data. The data is stored in the `postgres_data` volume. When you start the container again, all your data will still be there.

**To delete all data and start fresh:**
```bash
docker compose -f docker/docker-compose.yml down -v
```
The `-v` flag deletes volumes — use this only if you want to completely reset the database.

---

### Step 10: Create the Database Tables

**What we are doing:** Running a one-time script that tells PostgreSQL to create the four tables the project needs (`youtube_videos`, `openai_articles`, `anthropic_articles`, `digests`).

The database container is now running, but it is empty — no tables yet. SQLAlchemy can create them automatically from our Python model definitions.

```bash
uv run python app/database/create_tables.py
```

**What this command does:**
1. Python reads `app/database/models.py` (which defines the table structures)
2. SQLAlchemy generates the SQL `CREATE TABLE` commands
3. Those commands are sent to the running PostgreSQL container
4. The tables are created

**What you should see:**
```
Tables created successfully
```

**How to verify the tables were created:**

```bash
docker exec -it ai-news-aggregator-db psql -U postgres -d ai_news_aggregator -c "\dt"
```

**What this command does:** Connects to the running PostgreSQL container and lists all tables.

**What you should see:**
```
              List of relations
 Schema |         Name          | Type  |  Owner
--------+-----------------------+-------+----------
 public | anthropic_articles    | table | postgres
 public | digests               | table | postgres
 public | openai_articles       | table | postgres
 public | youtube_videos        | table | postgres
(4 rows)
```

All four tables should be listed.

> 💡 **Video Gap:** The video does not explain what `Base.metadata.create_all(engine)` does internally. Here is what happens: SQLAlchemy looks at every class that inherits from `Base` (all your model classes in `models.py`), reads their column definitions, and generates `CREATE TABLE IF NOT EXISTS` SQL for each one. If the tables already exist, it does nothing (safe to run multiple times).

---

### Step 11: Run the Full Pipeline

**What we are doing:** Running the complete pipeline for the first time.

Make sure:
- [x] `.env` file exists with your real API key and email credentials
- [x] Docker Desktop is open and the container is running (`docker ps` shows `Up`)
- [x] Tables are created (previous step)

```bash
uv run python main.py
```

**What you should see** (this takes several minutes):

```
2026-06-19 10:00:00 - INFO - ============================================================
2026-06-19 10:00:00 - INFO - Starting Daily AI News Aggregator Pipeline
2026-06-19 10:00:00 - INFO - ============================================================
2026-06-19 10:00:01 - INFO -
[1/5] Scraping articles from sources...
2026-06-19 10:00:05 - INFO - ✓ Scraped 2 YouTube videos, 3 OpenAI articles, 5 Anthropic articles
2026-06-19 10:00:05 - INFO -
[2/5] Processing Anthropic markdown...
2026-06-19 10:01:30 - INFO - ✓ Processed 5 Anthropic articles (0 failed)
2026-06-19 10:01:30 - INFO -
[3/5] Processing YouTube transcripts...
2026-06-19 10:01:45 - INFO - ✓ Processed 2 transcripts (0 unavailable)
2026-06-19 10:01:45 - INFO -
[4/5] Creating digests for articles...
2026-06-19 10:02:30 - INFO - ✓ Created 10 digests (0 failed out of 10 total)
2026-06-19 10:02:30 - INFO -
[5/5] Generating and sending email digest...
2026-06-19 10:02:45 - INFO - ✓ Email sent successfully with 10 articles
...
2026-06-19 10:02:45 - INFO - Pipeline Summary
2026-06-19 10:02:45 - INFO - Duration: 165.2 seconds
2026-06-19 10:02:45 - INFO - Email: Sent
```

**If the pipeline ran but "0 articles" were found:** This means no new content was published in the last 24 hours on the channels you monitor. This is normal. You can test with a larger window:

```bash
uv run python main.py 168 10
```

This looks at content from the last 168 hours (7 days) instead of 24 hours.

**Check your inbox:** You should receive an email within a few seconds of the pipeline finishing.

---

## Part 5: File-by-File Explanation

Every file in this repository is explained below. For each file: what it does, why it exists, when it runs, and what would break without it.

---

### Root-Level Files

#### `main.py`

**What it is:** The main entry point — the file you run to start everything.

**Why it exists:** Python projects need a single "front door." This file is that door. It accepts optional command-line arguments and calls the pipeline.

**When it runs:** Every time you run `python main.py`. In production, it is called by the Render.com cron job every day.

**What it contains:**
```python
from app.daily_runner import run_daily_pipeline

def main(hours: int = 24, top_n: int = 10):
    return run_daily_pipeline(hours=hours, top_n=top_n)

if __name__ == "__main__":
    # allows: python main.py 48 5
    # (look back 48 hours, send top 5 articles)
```

**What would break without it:** Nothing technically — you could call `daily_runner.py` directly. But having `main.py` is a Python convention for the entry point, and it is what the Dockerfile and Render.com are configured to call.

> 💡 **Video Gap:** `if __name__ == "__main__"` is Python's way of saying "only run this code if this file is run directly, not if it is imported by another file." When you run `python main.py`, Python sets `__name__` to `"__main__"`. When another file does `from main import something`, `__name__` is `"main"` instead. This pattern lets files be both runnable scripts and importable modules.

---

#### `pyproject.toml`

**What it is:** The project configuration file. Defines the project name, Python version requirement, and all dependencies.

**Why it exists:** Every modern Python project has one. It is the authoritative list of what the project needs.

**Key section:**
```toml
dependencies = [
    "beautifulsoup4>=4.14.2",  # HTML parsing
    "docling>=2.61.2",         # converts web pages to Markdown
    "feedparser>=6.0.12",      # reads RSS feeds
    "markdown>=3.7.0",         # converts Markdown to HTML (for email)
    "markdownify>=0.11.6",     # converts HTML to Markdown
    "openai>=2.7.2",           # OpenAI API client
    "psycopg2-binary>=2.9.11", # PostgreSQL driver for Python
    "pydantic>=2.0.0",         # data validation
    "python-dotenv>=1.2.1",    # reads .env files
    "requests>=2.32.5",        # HTTP requests
    "sqlalchemy>=2.0.44",      # ORM / database toolkit
    "youtube-transcript-api>=1.2.3",  # fetches YouTube transcripts
]
```

**What would break without it:** `uv sync` would fail — it reads this file to know what to install.

---

#### `uv.lock`

**What it is:** An automatically generated file that records the exact version of every package installed.

**Why it exists:** If `pyproject.toml` says `openai>=2.7.2`, it allows any version 2.7.2 or higher. But `uv.lock` pins it to exactly `2.7.4` (for example). This guarantees that if someone else installs the project, they get the same exact versions.

**When it is updated:** Automatically by `uv` when you add or update packages.

**What would break without it:** `uv sync` would still work, but might install slightly different versions on different machines, which can cause subtle bugs.

---

#### `.python-version`

**What it is:** A one-line file containing `3.12`. Tells `uv` (and tools like `pyenv`) which Python version this project requires.

**Why it exists:** So `uv` automatically uses Python 3.12 for this project.

**What would break without it:** `uv` might use whatever Python is currently active on your system, which could be a different version.

---

#### `.gitignore`

**What it is:** A list of files and folders that Git should never track or commit.

**Important entries:**
```
.env              ← your secrets — NEVER commit this
.venv/            ← virtual environment — large, recreatable
__pycache__/      ← Python bytecode cache — auto-generated
*.db              ← SQLite databases if any
.DS_Store         ← Mac OS file system metadata
```

**What would break without it:** If you accidentally committed `.env`, your API keys and passwords would be in Git history. If you committed `.venv`, your repository would be hundreds of megabytes.

---

### `app/` Folder

#### `app/__init__.py`

**What it is:** An empty file (usually) that tells Python "this folder is a package."

**Why it exists:** Without `__init__.py`, Python cannot import from this folder. If `app/__init__.py` did not exist, then `from app.scrapers.youtube import YouTubeScraper` would fail with "ModuleNotFoundError."

> 💡 **Video Gap:** Every folder in this project has an `__init__.py` file. These are not mistakes or leftovers — they are required. Python uses them to recognize importable packages. In Python 3.3+, "namespace packages" made them optional in some cases, but this project uses them explicitly for clarity.

---

#### `app/config.py`

**What it is:** A simple configuration file containing the YouTube channel IDs to monitor.

**What it contains:**
```python
YOUTUBE_CHANNELS = [
    "UCawZsQWqfGSbCI5yjkdVkTA",  # Matthew Berman's channel ID
]
```

**Why it exists:** Putting configuration in a separate file (instead of hardcoding it in the scraper) makes it easy to add or remove channels without touching the scraping logic.

**How to find a YouTube channel ID:**
1. Go to a YouTube channel
2. Right-click → View Page Source
3. Search for `"channel_id"` or `"externalId"`
4. Or use a website like commentpicker.com/youtube-channel-id.php

---

#### `app/daily_runner.py`

**What it is:** The master pipeline orchestrator. It runs all 5 steps in sequence and logs what happens.

**Why it exists:** `main.py` is just the entry point. The actual pipeline logic lives here, separate from the entry point. This makes it easier to test the pipeline in isolation.

**When it runs:** Called by `main.py` every time.

**Key pattern:**
```python
try:
    # step 1
    scraping_results = run_scrapers(hours=hours)
    # step 2
    anthropic_result = process_anthropic_markdown()
    # ... etc
    results["success"] = True
except Exception as e:
    logger.error(f"Pipeline failed: {e}")
    results["error"] = str(e)
```

The `try/except` means if any step crashes, the pipeline logs the error and exits gracefully instead of crashing with a scary traceback.

**What would break without it:** `main.py` would have nowhere to call, and the pipeline would not run.

---

#### `app/runner.py`

**What it is:** The scraping-only step. Creates all three scrapers, runs them, and saves results to the database.

**Why is it separate from `daily_runner.py`?** Because you might want to run *just the scraping* without running the full pipeline. For example, during development you might scrape once, then repeatedly test the AI summarization without re-scraping.

**How to run it alone:**
```bash
uv run python app/runner.py
```

**What you should see:**
```
YouTube videos: 2
OpenAI articles: 3
Anthropic articles: 5
```

---

### `app/scrapers/` Folder

#### `app/scrapers/youtube.py`

**What it is:** The YouTube scraper. Fetches recent video metadata from YouTube's RSS feed, and (optionally) downloads transcripts.

**Key classes:**

`ChannelVideo` (Pydantic model):
```
title: str
url: str
video_id: str
published_at: datetime
description: str
transcript: Optional[str]  # None until fetched separately
```

`YouTubeScraper`:
- `get_latest_videos(channel_id, hours)` — fetches the RSS feed, filters by date, returns a list of `ChannelVideo` objects (without transcripts)
- `get_transcript(video_id)` — fetches the transcript for one video from YouTube
- `scrape_channel(channel_id, hours)` — combines both (used for manual one-off runs)

**Why transcripts are fetched separately:**

The RSS feed gives you video metadata instantly. But transcripts require a separate API call per video, which is slow. By separating them:
1. `runner.py` saves videos quickly (no transcripts yet)
2. `process_youtube.py` (step 3) fetches transcripts in a second pass
3. If step 3 fails for one video, the video is still saved

> 💡 **Video Gap:** The scraper skips YouTube Shorts:
> ```python
> if "/shorts/" in entry.link:
>     continue
> ```
> Shorts rarely have transcripts and are usually not the kind of content you want in a news digest. This filter is easy to miss but important.

---

#### `app/scrapers/openai.py`

**What it is:** The OpenAI blog scraper. Reads the OpenAI RSS feed and returns a list of recent articles.

**RSS feed URL:** `https://openai.com/news/rss.xml`

**Key class:**

`OpenAIArticle` (Pydantic model):
```
title: str
description: str    # short excerpt from the RSS feed
url: str
guid: str           # unique ID for the article (usually same as URL)
published_at: datetime
category: Optional[str]
```

Note: The OpenAI scraper does NOT fetch the full article text (unlike Anthropic). It only uses the RSS description. This is because OpenAI's articles are available in the RSS description in enough detail for summarization.

---

#### `app/scrapers/anthropic.py`

**What it is:** The Anthropic blog scraper. Reads 3 RSS feeds and returns recent articles.

**RSS feed URLs:**
- News: a GitHub-hosted feed (Anthropic does not publish a native RSS feed)
- Research: same source
- Engineering: same source

**Why 3 feeds instead of 1?** Anthropic publishes different types of content in different categories. Three feeds ensure nothing is missed.

**Important method:** `url_to_markdown(url)` — uses the `docling` library to download the full article from its URL and convert it to clean Markdown text.

> 💡 **Video Gap:** Why `docling` instead of `requests.get(url)` + BeautifulSoup? Because:
> 1. Anthropic's pages include complex layouts, sidebars, navigation bars
> 2. `requests.get()` + BeautifulSoup would give you the entire page including menus, footers, ads
> 3. `docling` uses AI-based document understanding to extract just the article content and convert it cleanly to Markdown
> 4. This Markdown is then what GPT reads to generate summaries

---

### `app/database/` Folder

#### `app/database/models.py`

**What it is:** Python classes that define the structure of your database tables. Each class is one table.

**Why it exists:** SQLAlchemy needs to know the shape of your data before it can create tables or run queries. These model classes are the single source of truth.

**The four models:**

```python
class YouTubeVideo(Base):
    __tablename__ = "youtube_videos"
    video_id = Column(String, primary_key=True)  # unique ID from YouTube
    title = Column(String, nullable=False)
    url = Column(String, nullable=False)
    channel_id = Column(String, nullable=False)
    published_at = Column(DateTime, nullable=False)
    description = Column(Text)
    transcript = Column(Text, nullable=True)      # added in step 3
    created_at = Column(DateTime, default=datetime.utcnow)

class OpenAIArticle(Base):
    __tablename__ = "openai_articles"
    guid = Column(String, primary_key=True)       # unique ID from RSS
    title = Column(String, nullable=False)
    url = Column(String, nullable=False)
    description = Column(Text)                    # excerpt from RSS
    published_at = Column(DateTime, nullable=False)
    category = Column(String, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)

class AnthropicArticle(Base):
    __tablename__ = "anthropic_articles"
    guid = Column(String, primary_key=True)
    title = Column(String, nullable=False)
    url = Column(String, nullable=False)
    description = Column(Text)
    published_at = Column(DateTime, nullable=False)
    category = Column(String, nullable=True)
    markdown = Column(Text, nullable=True)        # added in step 2
    created_at = Column(DateTime, default=datetime.utcnow)

class Digest(Base):
    __tablename__ = "digests"
    id = Column(String, primary_key=True)         # format: "type:article_id"
    article_type = Column(String, nullable=False)  # "youtube", "openai", "anthropic"
    article_id = Column(String, nullable=False)
    url = Column(String, nullable=False)
    title = Column(String, nullable=False)         # AI-generated title
    summary = Column(Text, nullable=False)         # AI-generated summary
    created_at = Column(DateTime, default=datetime.utcnow)
```

> 💡 **Video Gap:** The `Digest.id` is set to `f"{article_type}:{article_id}"` — for example `"youtube:abc123"` or `"anthropic:https://anthropic.com/..."`. This **composite key** ensures you never create two digests for the same article. If you try to insert a digest with an `id` that already exists, PostgreSQL rejects it — and the code checks for this before inserting.

---

#### `app/database/connection.py`

**What it is:** The file that creates the connection to the PostgreSQL database.

**Why it exists:** All database operations need a connection. This file creates it once and makes it available to the rest of the app.

**Key objects:**
- `engine` — the low-level connection pool to PostgreSQL. SQLAlchemy creates and manages multiple connections through it.
- `SessionLocal` — a factory for creating database sessions
- `get_session()` — creates and returns one session. Called every time you need to talk to the database.

**What a session is:** A session is like a conversation with the database. You add objects to it, and when you call `.commit()`, the conversation is saved permanently. If something goes wrong, you can call `.rollback()` to undo everything since the last commit.

**The database URL format:**
```
postgresql://username:password@host:port/database_name
postgresql://postgres:postgres@localhost:5432/ai_news_aggregator
```

---

#### `app/database/repository.py`

**What it is:** The single file that handles all database reads and writes. All other parts of the app go through this file to access the database.

**Why it exists:** This is the **Repository Pattern** — a software design principle where all database access is centralized. Benefits:
- Easy to understand: all DB operations are in one place
- Easy to change: if you switch databases, only this file needs updating
- Easy to test: you can test database operations in isolation

**Key methods:**
- `bulk_create_youtube_videos(videos)` — insert multiple videos at once
- `get_anthropic_articles_without_markdown(limit)` — find articles needing enrichment
- `update_anthropic_article_markdown(guid, markdown)` — save fetched article text
- `get_articles_without_digest(limit)` — find articles needing AI summaries
- `create_digest(...)` — save an AI-generated summary
- `get_recent_digests(hours)` — get summaries from the last N hours for emailing

> 💡 **Video Gap:** Every `bulk_create_*` method checks if the item already exists before inserting:
> ```python
> existing = self.session.query(YouTubeVideo).filter_by(video_id=v["video_id"]).first()
> if not existing:
>     new_items.append(...)
> ```
> This makes the operation **idempotent** — safe to run multiple times. If you run the scraper twice in one hour, articles won't be duplicated. This is critical for a daily automated system.

---

#### `app/database/create_tables.py`

**What it is:** A one-time setup script. Run it once when setting up the project.

**What it does:**
```python
from app.database.models import Base
from app.database.connection import engine

Base.metadata.create_all(engine)
print("Tables created successfully")
```

`Base.metadata.create_all(engine)` — tells SQLAlchemy to look at all models that inherit from `Base`, generate `CREATE TABLE` SQL for each, and run them. It uses `CREATE TABLE IF NOT EXISTS`, so it is safe to run multiple times.

**When to run it:** Once, during initial setup. If you add a new column to a model later, you need to handle that differently (with migrations — more on this in Part 11).

---

### `app/services/` Folder

These files orchestrate multi-step processes: read from DB → call scraper or AI → write back to DB.

#### `app/services/process_anthropic.py`

**What it does:** For each Anthropic article that has no Markdown yet, downloads the full article text and saves it.

**Flow:**
1. `repo.get_anthropic_articles_without_markdown()` → get articles needing full text
2. For each: `scraper.url_to_markdown(article.url)` → download and convert
3. `repo.update_anthropic_article_markdown(guid, markdown)` → save to DB

---

#### `app/services/process_youtube.py`

**What it does:** For each YouTube video that has no transcript yet, fetches the transcript.

**The `__UNAVAILABLE__` marker:**

```python
TRANSCRIPT_UNAVAILABLE_MARKER = "__UNAVAILABLE__"

if transcript_result:
    repo.update_youtube_video_transcript(video.video_id, transcript_result.text)
else:
    repo.update_youtube_video_transcript(video.video_id, TRANSCRIPT_UNAVAILABLE_MARKER)
```

> 💡 **Video Gap:** This is a clever pattern. `transcript = None` means "we have not tried to fetch this transcript yet." `transcript = "__UNAVAILABLE__"` means "we tried and YouTube does not have a transcript for this video." This distinction matters because:
> - `get_articles_without_digest()` filters OUT videos where `transcript == "__UNAVAILABLE__"` — no point summarizing a video with no content
> - If we just stored `None` for both "not tried" and "unavailable," we would retry fetching unavailable transcripts every time the pipeline runs (wasting API calls)

---

#### `app/services/process_digest.py`

**What it does:** For each article/video without a digest yet, calls the AI to generate a summary.

**Flow:**
1. `repo.get_articles_without_digest()` — returns YouTube videos (with transcripts), OpenAI articles, and Anthropic articles (with markdown)
2. For each: `agent.generate_digest(title, content, type)` — sends to GPT-4o-mini
3. `repo.create_digest(...)` — saves the AI-generated summary

---

#### `app/services/process_email.py`

**What it does:** Retrieves recent digests, ranks them using the CuratorAgent, generates an email intro using the EmailAgent, and sends the email.

**Flow:**
1. `repo.get_recent_digests(hours=hours)` — get all summaries from last 24h
2. `curator.rank_digests(digests)` — AI ranks them 1-N
3. `email_agent.create_email_digest_response(ranked_articles, ...)` — AI writes intro
4. `send_email(subject, body_text, body_html)` — sends via Gmail SMTP

---

#### `app/services/email.py`

**What it does:** Sends email via Gmail's SMTP server.

**What SMTP is:** SMTP (Simple Mail Transfer Protocol) is the standard protocol for sending email. Gmail provides an SMTP server at `smtp.gmail.com` on port 465 (SSL) that you can use to send emails programmatically.

**Why HTML and plain text?** The email is sent in both formats. Email clients that support HTML will show the beautiful formatted version. Clients that don't (like some terminal-based ones) show the plain Markdown text. This is called a "multipart" email.

---

### `app/agent/` Folder

#### `app/agent/digest_agent.py`

**What it does:** Calls GPT-4o-mini to summarize one article into a title and 2-3 sentence summary.

**Key detail — structured output:**
```python
class DigestOutput(BaseModel):
    title: str
    summary: str

response = self.client.responses.parse(
    model=self.model,
    text_format=DigestOutput,  # tells OpenAI to return exactly this structure
    ...
)
```

`text_format=DigestOutput` uses OpenAI's **structured output** feature — the model is forced to return a JSON object matching the `DigestOutput` schema. This is more reliable than parsing free-form text.

---

#### `app/agent/curator_agent.py`

**What it does:** Sends all summaries to GPT-4.1 along with your user profile, and receives a ranked list back.

**Structured output:**
```python
class RankedArticle(BaseModel):
    digest_id: str
    relevance_score: float  # 0.0 to 10.0
    rank: int               # 1 = most relevant
    reasoning: str          # why this rank was given

class RankedDigestList(BaseModel):
    articles: List[RankedArticle]
```

The model returns a `RankedDigestList` containing one `RankedArticle` per input article.

---

#### `app/agent/email_agent.py`

**What it does:** Generates a personalized greeting and introduction for the email, then assembles the final email response.

**`to_markdown()` method on `EmailDigestResponse`:** Converts the structured response into readable Markdown that is then converted to HTML for sending.

---

### `app/profiles/user_profile.py`

**What it is:** A Python dictionary describing the reader's interests, background, and preferences.

**How to customize it for yourself:**

```python
USER_PROFILE = {
    "name": "Svetlana",
    "title": "Python Developer",
    "background": "Learning backend development and AI engineering",
    "interests": [
        "Python backend development",
        "Large Language Models (LLMs)",
        "Building AI-powered applications",
        "Docker and deployment",
    ],
    "preferences": {
        "prefer_practical": True,
        "prefer_beginner_friendly": True,
        "prefer_tutorials": True,
    },
    "expertise_level": "Beginner to Intermediate"
}
```

Change `"name"` to your name and adjust `"interests"` to your actual interests. This is what the CuratorAgent uses to rank articles.

---

### `docker/docker-compose.yml`

**What it is:** The Docker Compose configuration that defines how to run the PostgreSQL database locally.

**Full explanation:**
```yaml
services:
  postgres:
    image: postgres:17              # use the official PostgreSQL 17 image
    container_name: ai-news-aggregator-db  # name for the container
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-postgres}     # username (from .env, default: postgres)
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-postgres}  # password
      POSTGRES_DB: ${POSTGRES_DB:-ai_news_aggregator}    # database name
    ports:
      - "${POSTGRES_PORT:-5432}:5432"  # host_port:container_port
    volumes:
      - postgres_data:/var/lib/postgresql/data  # persist data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]  # check if DB is ready
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:  # named volume that persists on your machine
```

**Port mapping `5432:5432`:** The left side is the port on your computer. The right side is the port inside the container. They are the same here — so when Python connects to `localhost:5432`, Docker forwards it to port 5432 inside the container where PostgreSQL is running.

---

### `app/example.env`

**What it is:** A template showing what environment variables the project needs, with empty values.

**Why not call it `.env` directly?** Because `.env` should never be committed to Git (it has your secrets). `example.env` has no secrets — it is safe to commit and shows new users what variables they need to set.

---

## Part 6: Function-by-Function Explanation

This section explains the most important functions in the codebase.

---

### `YouTubeScraper.get_latest_videos(channel_id, hours)`

**File:** `app/scrapers/youtube.py`

**What it receives:**
- `channel_id` — a string like `"UCawZsQWqfGSbCI5yjkdVkTA"`
- `hours` — how many hours back to look (default: 24)

**What it returns:** A list of `ChannelVideo` objects (Pydantic models) — one for each video published within the time window.

**How it works step by step:**
1. Builds the RSS URL: `https://www.youtube.com/feeds/videos.xml?channel_id=...`
2. `feedparser.parse(url)` — downloads and parses the RSS feed
3. Calculates `cutoff_time = now - timedelta(hours=hours)`
4. Loops through all entries in the feed
5. Skips any entry with `/shorts/` in the URL
6. Converts the `published_parsed` tuple (like `(2026, 6, 19, 9, 0, 0, ...)`) to a `datetime` object
7. If `published_time >= cutoff_time`, adds it to the results list
8. Returns the list

**Why it needs it:** To limit results to recent content only — you don't want to summarize a 5-year-old video.

---

### `YouTubeScraper.get_transcript(video_id)`

**File:** `app/scrapers/youtube.py`

**What it receives:** A YouTube video ID (like `"jqd6_bbjhS8"`)

**What it returns:** A `Transcript` object (with a `.text` attribute containing all spoken words), or `None` if unavailable.

**How it works:**
1. Calls `youtube_transcript_api.fetch(video_id)` — an unofficial YouTube API that retrieves transcripts
2. Joins all snippets into one long string
3. Returns `Transcript(text=combined_text)`
4. If YouTube has disabled transcripts or there are none, catches the error and returns `None`

---

### `Repository.get_articles_without_digest(limit)`

**File:** `app/database/repository.py`

**What it receives:** Optional `limit` (max number of articles to return)

**What it returns:** A list of dictionaries, each representing one article/video that needs a summary.

**How it works:**
1. Gets the IDs of all existing digests (to know which articles already have summaries)
2. Queries YouTube videos WHERE transcript IS NOT NULL AND transcript != `"__UNAVAILABLE__"`
3. Queries ALL OpenAI articles (they always have a description)
4. Queries Anthropic articles WHERE markdown IS NOT NULL
5. Combines all results into one list, skipping any that already have a digest
6. Returns the combined list

**Why this complexity?** The project has 3 source tables but 1 digest table. This method bridges them — it knows which articles from any source still need summarizing.

---

### `DigestAgent.generate_digest(title, content, article_type)`

**File:** `app/agent/digest_agent.py`

**What it receives:**
- `title` — the original article/video title
- `content` — the full text (transcript, markdown, or description)
- `article_type` — `"youtube"`, `"openai"`, or `"anthropic"`

**What it returns:** A `DigestOutput` object with `.title` and `.summary`, or `None` if the API call failed.

**How it works:**
1. Truncates content to 8,000 characters (`content[:8000]`) — to fit within token limits
2. Sends a prompt to GPT-4o-mini with the PROMPT system message
3. Uses structured output (`text_format=DigestOutput`) to get a guaranteed format back
4. Returns the parsed `DigestOutput`

---

### `CuratorAgent.rank_digests(digests)`

**File:** `app/agent/curator_agent.py`

**What it receives:** A list of digest dictionaries (from `repo.get_recent_digests()`)

**What it returns:** A list of `RankedArticle` objects, ordered by rank

**How it works:**
1. Formats all digests into a readable text block (ID, title, summary, type)
2. Sends all digests + the user profile to GPT-4.1 in one call
3. Asks the model to assign a relevance score (0.0–10.0) and rank to each
4. Returns the structured list of `RankedArticle` objects

**Why send all articles at once?** The model needs to see all articles to make relative rankings — it cannot rank them if it only sees one at a time.

---

### `send_email(subject, body_text, body_html, recipients)`

**File:** `app/services/email.py`

**What it receives:**
- `subject` — email subject line
- `body_text` — plain text version (Markdown)
- `body_html` — HTML version (rendered beautifully)
- `recipients` — list of email addresses (defaults to `MY_EMAIL` from `.env`)

**What it returns:** Nothing (sends the email)

**How it works:**
1. Creates a `MIMEMultipart("alternative")` message — a container for two versions
2. Attaches the plain text part
3. Attaches the HTML part (email clients prefer this version if they support HTML)
4. Connects to Gmail's SMTP server using SSL on port 465
5. Logs in with your email and App Password
6. Sends the message

**Why port 465 and SSL?** Port 465 is Gmail's SMTPS port (SMTP over SSL). It encrypts the connection from the start, so your credentials and email content cannot be intercepted.

---

## Learning Checkpoint — Parts 1–6

### You should now understand:

- [ ] What this project does and why (automated AI news digest)
- [ ] The 5-step pipeline: scrape → enrich → summarize → rank → email
- [ ] What a virtual environment is and why this project uses `uv`
- [ ] What a database is and why the pipeline uses one
- [ ] What Docker is and what the PostgreSQL container does
- [ ] What SQLAlchemy does (translates Python to SQL)
- [ ] What an RSS feed is and why it is used instead of web scraping
- [ ] What a `.env` file is and why secrets go there
- [ ] What each folder (`scrapers/`, `database/`, `services/`, `agent/`) is responsible for
- [ ] What every file in the project does

### You should now be able to:

- [ ] Open the project in VS Code
- [ ] Create the `.env` file and fill in your credentials
- [ ] Run `uv sync` to install dependencies
- [ ] Start the database with `docker compose up -d`
- [ ] Run `python app/database/create_tables.py` to create tables
- [ ] Run `python main.py` to execute the full pipeline
- [ ] Check Docker logs: `docker ps`
- [ ] Understand what each log line in the pipeline output means

### Check Yourself:

1. What is the difference between `runner.py` and `daily_runner.py`?
2. Why does the Anthropic scraper use `docling` instead of just `requests`?
3. What does `transcript = "__UNAVAILABLE__"` mean, and why is it different from `transcript = None`?
4. Why does every model class inherit from `Base`?
5. Where does the `Digest.id` value come from? (Hint: look at `repository.py`)
6. Why is `load_dotenv()` called at the top of several files?

---

---

## Part 7: Step-by-Step Build Roadmap

This section teaches you how to **rebuild** this project from scratch. Each lesson has a goal, what to create, what the code means, how to test it, and what success looks like.

Work through these lessons in order. After each one, your project should be in a runnable state.

---

### Lesson 1: Project Skeleton and Dependencies

**Goal:** Create the project folder structure and set up dependency management with `uv`.

**What you need first:** `uv` installed (see Part 4, Step 2)

**Step 1: Create the project folder**

Open your terminal and run:
```bash
mkdir ai-news-aggregator
cd ai-news-aggregator
```

**What these do:**
- `mkdir ai-news-aggregator` — creates a new folder called `ai-news-aggregator`
- `cd ai-news-aggregator` — enters that folder

**Step 2: Initialize the project with uv**

```bash
uv init
```

**What this does:** Creates a `pyproject.toml` file with basic project information.

**Step 3: Replace the generated pyproject.toml with the correct one**

Open `pyproject.toml` in VS Code and replace its contents with:

```toml
[project]
name = "ai-news-aggregator"
version = "0.1.0"
description = "Automated AI news aggregator with daily email digest"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "beautifulsoup4>=4.14.2",
    "docling>=2.61.2",
    "feedparser>=6.0.12",
    "markdown>=3.7.0",
    "markdownify>=0.11.6",
    "openai>=2.7.2",
    "psycopg2-binary>=2.9.11",
    "pydantic>=2.0.0",
    "python-dotenv>=1.2.1",
    "requests>=2.32.5",
    "sqlalchemy>=2.0.44",
    "youtube-transcript-api>=1.2.3",
]
```

**Step 4: Install dependencies**

```bash
uv sync
```

**Step 5: Create the folder structure**

```bash
mkdir -p app/scrapers app/database app/services app/agent app/profiles docker
touch app/__init__.py
touch app/scrapers/__init__.py
touch app/database/__init__.py
touch app/services/__init__.py
touch app/agent/__init__.py
touch app/profiles/__init__.py
```

**What `mkdir -p` does:** Creates folders, including any parent folders that do not exist yet.
**What `touch` does:** Creates an empty file (or updates its timestamp if it already exists).

**Step 6: Create main.py**

Create `main.py` in the project root:

```python
from app.daily_runner import run_daily_pipeline


def main(hours: int = 24, top_n: int = 10):
    return run_daily_pipeline(hours=hours, top_n=top_n)


if __name__ == "__main__":
    import sys

    hours = 24
    top_n = 10

    if len(sys.argv) > 1:
        hours = int(sys.argv[1])
    if len(sys.argv) > 2:
        top_n = int(sys.argv[2])

    result = main(hours=hours, top_n=top_n)
    exit(0 if result["success"] else 1)
```

**What `sys.argv` is:** A list of command-line arguments. When you run `python main.py 48 5`, then `sys.argv = ["main.py", "48", "5"]`. This lets you call the pipeline with different parameters.

**What `exit(0 if result["success"] else 1)` does:** Exits the program with code 0 (success) or 1 (failure). This is important for automated systems — Render.com will know the run failed if it sees exit code 1.

**How to test this lesson:**

```bash
uv run python -c "from app import __init__; print('Imports OK')"
```

**Expected output:** `Imports OK`

---

### Lesson 2: Database Models (The Shape of Your Data)

**Goal:** Define the four database tables as Python classes.

**Create `app/database/models.py`:**

```python
from datetime import datetime
from typing import Optional
from sqlalchemy import Column, String, DateTime, Text
from sqlalchemy.orm import declarative_base

Base = declarative_base()


class YouTubeVideo(Base):
    __tablename__ = "youtube_videos"

    video_id = Column(String, primary_key=True)
    title = Column(String, nullable=False)
    url = Column(String, nullable=False)
    channel_id = Column(String, nullable=False)
    published_at = Column(DateTime, nullable=False)
    description = Column(Text)
    transcript = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)


class OpenAIArticle(Base):
    __tablename__ = "openai_articles"

    guid = Column(String, primary_key=True)
    title = Column(String, nullable=False)
    url = Column(String, nullable=False)
    description = Column(Text)
    published_at = Column(DateTime, nullable=False)
    category = Column(String, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)


class AnthropicArticle(Base):
    __tablename__ = "anthropic_articles"

    guid = Column(String, primary_key=True)
    title = Column(String, nullable=False)
    url = Column(String, nullable=False)
    description = Column(Text)
    published_at = Column(DateTime, nullable=False)
    category = Column(String, nullable=True)
    markdown = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)


class Digest(Base):
    __tablename__ = "digests"

    id = Column(String, primary_key=True)
    article_type = Column(String, nullable=False)
    article_id = Column(String, nullable=False)
    url = Column(String, nullable=False)
    title = Column(String, nullable=False)
    summary = Column(Text, nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
```

**Line-by-line explanation of key parts:**

| Code | What it means |
|------|--------------|
| `Base = declarative_base()` | Creates the base class all models inherit from |
| `__tablename__ = "youtube_videos"` | Names the actual table in the database |
| `Column(String, primary_key=True)` | String column that uniquely identifies each row |
| `Column(Text, nullable=True)` | Long text column that can be empty (NULL) |
| `Column(DateTime, default=datetime.utcnow)` | Date column that auto-fills with current UTC time |
| `nullable=False` | This column must always have a value |

**How to test this lesson:**

```bash
uv run python -c "from app.database.models import YouTubeVideo, Digest; print('Models OK')"
```

**Expected output:** `Models OK`

---

### Lesson 3: Database Connection and Repository

**Goal:** Connect to PostgreSQL and create all database read/write functions.

**Step 1: Create `app/database/connection.py`**

```python
import os
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from dotenv import load_dotenv

load_dotenv()


def get_database_url() -> str:
    user = os.getenv("POSTGRES_USER", "postgres")
    password = os.getenv("POSTGRES_PASSWORD", "postgres")
    host = os.getenv("POSTGRES_HOST", "localhost")
    port = os.getenv("POSTGRES_PORT", "5432")
    db = os.getenv("POSTGRES_DB", "ai_news_aggregator")
    return f"postgresql://{user}:{password}@{host}:{port}/{db}"


engine = create_engine(get_database_url())
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)


def get_session():
    return SessionLocal()
```

**What `os.getenv("POSTGRES_USER", "postgres")` means:** Get the environment variable `POSTGRES_USER`. If it is not set, use `"postgres"` as the default. The second argument is the fallback value.

**Step 2: Create `app/database/create_tables.py`**

```python
import sys
from pathlib import Path

sys.path.insert(0, str(Path(__file__).parent.parent.parent))

from app.database.models import Base
from app.database.connection import engine

if __name__ == "__main__":
    Base.metadata.create_all(engine)
    print("Tables created successfully")
```

**What `sys.path.insert(0, ...)` does:** This script is run directly (not through the `app` package), so Python might not find `app.database.models` unless we tell it where to look. This line adds the project root to Python's search path.

**What `Path(__file__).parent.parent.parent` is:**
- `__file__` = `/path/to/project/app/database/create_tables.py`
- `.parent` = `/path/to/project/app/database/`
- `.parent.parent` = `/path/to/project/app/`
- `.parent.parent.parent` = `/path/to/project/` ← the project root

**Step 3: Create `app/database/repository.py`** (see full file in Part 5 — copy from there)

**How to test this lesson:**

First make sure Docker is running and the database container is started:
```bash
docker compose -f docker/docker-compose.yml up -d
```

Then create tables:
```bash
uv run python app/database/create_tables.py
```

**Expected output:** `Tables created successfully`

Verify:
```bash
docker exec -it ai-news-aggregator-db psql -U postgres -d ai_news_aggregator -c "\dt"
```

**Expected output:** A list of 4 tables.

---

### Lesson 4: YouTube Scraper

**Goal:** Build the YouTube scraper that fetches recent video metadata via RSS.

**Create `app/config.py`:**

```python
YOUTUBE_CHANNELS = [
    "UCawZsQWqfGSbCI5yjkdVkTA",  # Matthew Berman
    # Add more channel IDs here
]
```

**Create `app/scrapers/youtube.py`** (copy from Part 5 / the repository)

**How to test in isolation:**

```bash
uv run python -c "
from app.scrapers.youtube import YouTubeScraper
scraper = YouTubeScraper()
videos = scraper.get_latest_videos('UCawZsQWqfGSbCI5yjkdVkTA', hours=168)
print(f'Found {len(videos)} videos')
for v in videos[:3]:
    print(f'  - {v.title}')
"
```

**Expected output:**
```
Found 5 videos
  - My Latest Video Title
  - Another Video Title
  - ...
```

**Common issue:** If you get `0 videos`, try increasing `hours=` to a larger number like `720` (30 days). The channel may not have posted in the last 7 days.

---

### Lesson 5: OpenAI and Anthropic Scrapers

**Goal:** Build scrapers for the OpenAI and Anthropic blogs.

**Create `app/scrapers/openai.py`** (copy from Part 5 / the repository)

**Create `app/scrapers/anthropic.py`** (copy from Part 5 / the repository)

**How to test OpenAI scraper:**
```bash
uv run python -c "
from app.scrapers.openai import OpenAIScraper
scraper = OpenAIScraper()
articles = scraper.get_articles(hours=168)
print(f'Found {len(articles)} OpenAI articles')
for a in articles[:3]:
    print(f'  - {a.title}')
"
```

**How to test Anthropic scraper:**
```bash
uv run python -c "
from app.scrapers.anthropic import AnthropicScraper
scraper = AnthropicScraper()
articles = scraper.get_articles(hours=168)
print(f'Found {len(articles)} Anthropic articles')
for a in articles[:3]:
    print(f'  - {a.title}')
"
```

---

### Lesson 6: Runner (Scraping + Saving to Database)

**Goal:** Wire together the three scrapers and the database repository to complete Step 1 of the pipeline.

**Create `app/runner.py`** (copy from Part 5 / the repository)

**How to test:**

Make sure the database is running, then:
```bash
uv run python app/runner.py
```

**Expected output:**
```
YouTube videos: 2
OpenAI articles: 3
Anthropic articles: 5
```

**Verify data was saved:**
```bash
docker exec -it ai-news-aggregator-db psql -U postgres -d ai_news_aggregator -c "SELECT title FROM youtube_videos LIMIT 5;"
```

You should see video titles printed from the database.

---

### Lesson 7: Content Processing Services

**Goal:** Build the services that enrich raw scraped data with transcripts and full article text.

**Create `app/services/process_youtube.py`** (copy from Part 5)

**Create `app/services/process_anthropic.py`** (copy from Part 5)

**How to test transcript fetching:**
```bash
uv run python app/services/process_youtube.py
```

**Expected output:**
```
Total videos: 2
Processed: 2
Unavailable: 0
Failed: 0
```

> **Note:** Fetching transcripts can take 10–30 seconds per video. This is normal — the YouTube API has rate limits.

**How to test Anthropic markdown fetching:**
```bash
uv run python app/services/process_anthropic.py
```

**Expected output:**
```
Total articles: 5
Processed: 5
Failed: 0
```

> **Note:** `docling` downloads and converts full web pages. This is slower than simple RSS parsing — expect 30–60 seconds per article.

---

### Lesson 8: AI Digest Agent (Summarizer)

**Goal:** Build the agent that calls GPT-4o-mini to generate summaries.

**Before you start:** Make sure `OPENAI_API_KEY` is set in your `.env` file.

**Create `app/agent/digest_agent.py`** (copy from Part 5)

**Create `app/services/process_digest.py`** (copy from Part 5)

**How to test (generate summaries for all articles in the database):**
```bash
uv run python app/services/process_digest.py
```

**Expected output:**
```
2026-06-19 10:00:00 - INFO - Starting digest processing for 7 articles
2026-06-19 10:00:00 - INFO - [1/7] Processing youtube: My Video Title (ID: abc123)
2026-06-19 10:00:02 - INFO - ✓ Successfully created digest for youtube abc123
2026-06-19 10:00:02 - INFO - [2/7] Processing openai: GPT-5 Released (ID: https://...)
...
2026-06-19 10:00:15 - INFO - Processing complete: 7 processed, 0 failed out of 7 total
```

**Verify digests were saved:**
```bash
docker exec -it ai-news-aggregator-db psql -U postgres -d ai_news_aggregator -c "SELECT title, summary FROM digests LIMIT 3;"
```

**Cost note:** Summarizing 10 articles with gpt-4o-mini typically costs less than $0.01. The `content[:8000]` truncation keeps costs predictable.

---

### Lesson 9: AI Curator Agent (Ranker)

**Goal:** Build the agent that ranks summaries by relevance to your personal profile.

**Create `app/profiles/user_profile.py`**

```python
USER_PROFILE = {
    "name": "Svetlana",
    "title": "Python Developer & Learner",
    "background": "Learning backend development and building AI-powered applications",
    "interests": [
        "Python backend development",
        "Large Language Models (LLMs) and their applications",
        "Building AI-powered applications",
        "Practical AI tutorials and guides",
        "Docker and deployment",
        "Database design and SQLAlchemy",
    ],
    "preferences": {
        "prefer_practical": True,
        "prefer_beginner_friendly": True,
        "prefer_tutorials": True,
        "prefer_technical_depth": True,
        "avoid_marketing_hype": True,
    },
    "expertise_level": "Beginner to Intermediate"
}
```

**Create `app/agent/curator_agent.py`** (copy from Part 5)

**Create `app/services/process_curator.py`** (copy from Part 5)

**How to test:**
```bash
uv run python app/services/process_curator.py
```

**Expected output:**
```
2026-06-19 10:00:00 - INFO - Curating 7 digests from the last 24 hours
2026-06-19 10:00:00 - INFO - User profile: Svetlana - Learning backend development...
2026-06-19 10:00:03 - INFO - Successfully ranked 7 articles

=== Top 10 Ranked Articles ===

Rank 1 | Score: 9.2/10.0
Title: How to Build a RAG System with Python
Type: youtube
Reasoning: Directly relevant to Svetlana's interest in practical AI tutorials...

Rank 2 | Score: 8.7/10.0
Title: Claude 4 Features for Developers
Type: anthropic
Reasoning: Strong match with LLM interest and practical applications...
```

**Cost note:** GPT-4.1 is more expensive than gpt-4o-mini. Ranking 10–20 articles costs approximately $0.02–$0.05.

---

### Lesson 10: Email Agent and SMTP Sending

**Goal:** Build the email generation and sending functionality.

**Create `app/agent/email_agent.py`** (copy from Part 5)

**Create `app/services/email.py`** (copy from Part 5)

**Create `app/services/process_email.py`** (copy from Part 5)

**Test email sending first (without the full pipeline):**

Create a temporary test file `test_email.py` in the project root:
```python
from dotenv import load_dotenv
load_dotenv()
from app.services.email import send_email

send_email(
    subject="Test email from AI News Aggregator",
    body_text="Hello Svetlana! Your email sending is working correctly.",
    body_html="<h1>Hello Svetlana!</h1><p>Your email sending is working correctly.</p>"
)
print("Email sent!")
```

Run it:
```bash
uv run python test_email.py
```

**Expected output:** `Email sent!`

Check your inbox — you should receive the test email within seconds.

**Common error:** `SMTPAuthenticationError` — your App Password is wrong or 2-Step Verification is not enabled. See Part 13 for fixes.

After testing, delete `test_email.py` (it contains no secrets but is just clutter).

---

### Lesson 11: Pipeline Runner (Wiring Everything Together)

**Goal:** Create the daily runner that orchestrates all 5 steps in sequence.

**Create `app/daily_runner.py`** (copy from Part 5)

**Run the full pipeline:**
```bash
uv run python main.py
```

Watch the logs carefully. Each step should print a `✓` success message.

**If step 5 (email) fails but steps 1–4 succeeded:** Don't panic. The data is saved in the database. Fix the email issue (see Part 13) and run again — steps 1–4 will skip articles that are already processed.

**Running with different time windows:**
```bash
# Last 48 hours, top 5 articles
uv run python main.py 48 5

# Last 7 days, top 10 articles
uv run python main.py 168 10
```

---

### Lesson 12: Docker Setup for Local Database

**Goal:** Understand the Docker setup and how to manage the database container.

**The docker-compose.yml already exists at `docker/docker-compose.yml`.** Here are the key commands to know:

```bash
# Start the database (background mode)
docker compose -f docker/docker-compose.yml up -d

# Check if it is running
docker ps

# See database logs
docker logs ai-news-aggregator-db

# Stop the database (data is preserved)
docker compose -f docker/docker-compose.yml down

# Stop and delete all data (fresh start)
docker compose -f docker/docker-compose.yml down -v

# Connect to the database with a SQL prompt
docker exec -it ai-news-aggregator-db psql -U postgres -d ai_news_aggregator

# Inside the SQL prompt, useful commands:
# \dt          — list all tables
# \d youtube_videos  — describe a table
# SELECT COUNT(*) FROM digests;  — count rows
# \q           — quit
```

---

## Part 8: Environment Variables Deep Dive

### What `.env` Is

A `.env` file is a plain text configuration file. Every line is either:
- A comment: `# this is a comment`
- A key-value pair: `VARIABLE_NAME=value`

Your Python code reads it at startup using `python-dotenv`:

```python
from dotenv import load_dotenv
import os

load_dotenv()  # reads .env file, loads all variables into the process environment

value = os.getenv("MY_VARIABLE")  # retrieves a specific variable
```

### How `load_dotenv()` Works

When Python starts, it has an "environment" — a dictionary of key-value pairs it inherits from your operating system. `load_dotenv()` reads your `.env` file and adds its key-value pairs into this environment dictionary. After calling `load_dotenv()`, `os.getenv("OPENAI_API_KEY")` can find your key.

**Critical order:** `load_dotenv()` must be called before any `os.getenv()` call. In `daily_runner.py`, you will see it called right at the top of the file.

### Every Variable Explained

| Variable | What It Is | Example Value | Where to Get It |
|----------|-----------|---------------|----------------|
| `OPENAI_API_KEY` | Your OpenAI API key | `sk-proj-abc123...` | platform.openai.com → API Keys |
| `MY_EMAIL` | Your Gmail address | `svetlana@gmail.com` | Your Gmail account |
| `APP_PASSWORD` | Gmail SMTP app password (16 chars, no spaces) | `abcdefghijklmnop` | Google Account → Security → App Passwords |
| `POSTGRES_USER` | PostgreSQL username | `postgres` | Set in docker-compose.yml (keep as `postgres`) |
| `POSTGRES_PASSWORD` | PostgreSQL password | `postgres` | Set in docker-compose.yml (keep as `postgres` for local dev) |
| `POSTGRES_DB` | Database name | `ai_news_aggregator` | Set in docker-compose.yml |
| `POSTGRES_HOST` | Where PostgreSQL is running | `localhost` | Always `localhost` for local development |
| `POSTGRES_PORT` | Port PostgreSQL is listening on | `5432` | Standard PostgreSQL port |

**Added in `deployment` branch (needed for Render.com):**

| Variable | What It Is | Example Value |
|----------|-----------|---------------|
| `DATABASE_URL` | Full connection string for cloud database | `postgresql://user:pass@host/db` |
| `ENVIRONMENT` | Which environment is running | `LOCAL` or `PRODUCTION` |
| `PROXY_USERNAME` | Webshare proxy username (optional) | `yourproxy` |
| `PROXY_PASSWORD` | Webshare proxy password (optional) | `yourpass` |

### Why `POSTGRES_HOST=localhost`

`localhost` is a special hostname that always refers to your own computer. When your Python code says "connect to `localhost:5432`", it means "connect to port 5432 on this same machine." Docker maps the container's port 5432 to your machine's port 5432, so this works seamlessly.

### What Must NEVER Be Committed to Git

```
.env                    ← ALL secrets
app/google-service-account.json  ← if you add Google APIs
any file with "secret", "key", "password" in the name
```

**How to verify `.env` is ignored:**
```bash
git status
```

The `.env` file should NOT appear in the output. If it does, add `.env` to `.gitignore` immediately.

**What to do if you accidentally commit secrets:**
1. Delete the key immediately from the service (OpenAI, Google, etc.)
2. Create a new key
3. Remove the secret from git history (ask for help — it is complex)
4. The lesson: always add `.env` to `.gitignore` before your first commit

### Your Complete `.env` File (with Svetlana's values)

```bash
# === OpenAI API ===
OPENAI_API_KEY=sk-proj-your-actual-key-here

# === Email (Gmail) ===
MY_EMAIL=svetlana@gmail.com
APP_PASSWORD=your16charpassword

# === Local Database (for Docker) ===
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=ai_news_aggregator
POSTGRES_HOST=localhost
POSTGRES_PORT=5432

# === Optional: leave empty for local development ===
DATABASE_URL=
ENVIRONMENT=LOCAL
```

---

## Part 9: Docker Explained for This Project

### The "Portable Kitchen" Analogy

Imagine you want to serve soup to guests. You could:

**Option A (no Docker):** Build a kitchen from scratch in your living room. Install pipes, buy appliances, configure everything. Time-consuming, and the kitchen might not work on someone else's house layout.

**Option B (with Docker):** Use a food truck. The food truck has its own kitchen inside — fully equipped and self-contained. Park it outside your house and start serving soup immediately. Move it to someone else's house and it works exactly the same.

**Docker containers are like food trucks:**
- Self-contained — everything they need is inside
- Portable — run identically on any computer with Docker
- Quick to start — no setup, just run
- Easy to discard — stop it, and it is gone. The food truck drives away.

### What an Image Is

An image is the **blueprint** (recipe) for creating a container. The `postgres:17` image is an official PostgreSQL image published by the PostgreSQL team. It contains:
- A Linux operating system (Alpine or Debian, very small)
- PostgreSQL 17 already installed
- Configuration files
- A startup script

Images are downloaded from **Docker Hub** (hub.docker.com) the first time you use them. After that, they are cached on your machine.

### What a Container Is

A container is a **running instance** of an image. When you run:
```bash
docker compose -f docker/docker-compose.yml up -d
```

Docker takes the `postgres:17` image and creates a running container from it. The container is like a tiny isolated computer running inside your computer, with PostgreSQL running inside it.

You can have multiple containers from the same image. Like using the same cake recipe to make multiple cakes.

### What Docker Compose Is

Docker Compose is a tool for defining and running containers using a configuration file (`docker-compose.yml`). Instead of running a long `docker run` command every time, you write the configuration once:

```yaml
services:
  postgres:
    image: postgres:17
    container_name: ai-news-aggregator-db
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: ai_news_aggregator
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
```

And then just run `docker compose up -d`. Compose reads the file and does everything automatically.

### Why PostgreSQL Runs in Docker (Not Installed Directly)

| Without Docker | With Docker |
|---------------|-------------|
| Install PostgreSQL manually (different for Mac/Windows/Linux) | One command: `docker compose up -d` |
| Configure users, passwords, databases | Pre-configured in `docker-compose.yml` |
| Conflicts with other PostgreSQL versions | Completely isolated |
| Hard to "reset" | `docker compose down -v` deletes everything, `up -d` starts fresh |
| Permanent installation on your computer | Temporary — remove Docker image and it is completely gone |

### Understanding Ports

A port is like an apartment number in a building. Your computer is the building. Different services "live" on different ports and listen for incoming connections:

- Port `80` — HTTP web traffic
- Port `443` — HTTPS (secure web traffic)
- Port `5432` — PostgreSQL (the standard)
- Port `6379` — Redis
- Port `8080` — Common for local web servers

**Port mapping in docker-compose.yml:**
```yaml
ports:
  - "5432:5432"
```

This says: "Take port 5432 on my computer (left) and forward it to port 5432 inside the container (right)." When Python connects to `localhost:5432`, Docker forwards that to the PostgreSQL running inside the container.

**Port conflict:** If something else is already using port 5432 on your machine, Docker will fail to start with an error like "port is already in use." Solution: Stop the other service, or change the left side to `5433:5432` (use port 5433 on your machine instead).

### Understanding Volumes

A volume is a persistent storage location. Without volumes, all data inside a container is lost when the container stops.

```yaml
volumes:
  postgres_data:/var/lib/postgresql/data
```

This creates a "named volume" called `postgres_data` on your host machine. Docker maps it to `/var/lib/postgresql/data` inside the container (where PostgreSQL stores its data files). When the container stops and starts again, the data is still there in `postgres_data`.

**Think of it as:** A USB drive that you plug into the food truck. You can take the drive out (stop the container), plug it into a different truck (new container), and all your data is still there.

**Where volumes are stored:** Docker manages them automatically. On Mac: `~/Library/Containers/com.docker.docker/Data/vms/...`

### How Python Connects to the Container

The connection chain:

```
Python (your code)
  │
  │  uses connection URL: "postgresql://postgres:postgres@localhost:5432/ai_news_aggregator"
  ▼
SQLAlchemy (creates engine)
  │
  │  uses psycopg2 driver
  ▼
psycopg2 (PostgreSQL driver)
  │
  │  sends TCP connection to localhost:5432
  ▼
Docker (port mapping: host:5432 → container:5432)
  │
  ▼
PostgreSQL (running inside container, listening on port 5432)
```

The "magic" is that to your Python code, it looks like PostgreSQL is running directly on `localhost:5432`. Docker handles the forwarding transparently.

---

## Part 10: Database Deep Dive

### The Four Tables

| Table | What It Stores | Primary Key | Key Special Column |
|-------|---------------|-------------|-------------------|
| `youtube_videos` | YouTube video metadata | `video_id` (YouTube's ID) | `transcript` (NULL until fetched) |
| `openai_articles` | OpenAI blog posts | `guid` (from RSS) | `description` (RSS excerpt) |
| `anthropic_articles` | Anthropic blog posts | `guid` (from RSS) | `markdown` (NULL until fetched) |
| `digests` | AI-generated summaries | `id` (type:article_id) | `summary` (AI-generated) |

### Column-by-Column: `youtube_videos`

| Column | Type | Nullable | What It Stores |
|--------|------|---------|----------------|
| `video_id` | String | No (PK) | YouTube's unique ID (e.g., `"jqd6_bbjhS8"`) |
| `title` | String | No | Video title |
| `url` | String | No | Full YouTube URL |
| `channel_id` | String | No | The channel's ID from `config.py` |
| `published_at` | DateTime | No | When the video was published |
| `description` | Text | Yes | YouTube video description |
| `transcript` | Text | Yes | Full spoken text, `NULL` = not fetched yet, `"__UNAVAILABLE__"` = no transcript exists |
| `created_at` | DateTime | No | When we added this row (auto-set) |

### Column-by-Column: `digests`

| Column | Type | Nullable | What It Stores |
|--------|------|---------|----------------|
| `id` | String | No (PK) | `"youtube:jqd6_bbjhS8"` or `"openai:https://..."` |
| `article_type` | String | No | `"youtube"`, `"openai"`, or `"anthropic"` |
| `article_id` | String | No | The original article's primary key |
| `url` | String | No | URL to the original content |
| `title` | String | No | AI-generated title (2–7 words) |
| `summary` | Text | No | AI-generated summary (2–3 sentences) |
| `created_at` | DateTime | No | When the digest was created |

### How SQLAlchemy Models Map to Tables

```
Python Class               Database Table
──────────────────         ──────────────────────────────
class YouTubeVideo:        CREATE TABLE youtube_videos (
  __tablename__ =              ...
    "youtube_videos"       );
  video_id = Column(       video_id VARCHAR PRIMARY KEY,
    String,
    primary_key=True)
  title = Column(          title VARCHAR NOT NULL,
    String,
    nullable=False)
  transcript = Column(     transcript TEXT,
    Text,
    nullable=True)
  created_at = Column(     created_at TIMESTAMP DEFAULT NOW()
    DateTime,
    default=datetime.utcnow)
```

SQLAlchemy reads your Python class and generates exactly this SQL. `Base.metadata.create_all(engine)` executes the SQL against your database.

### How Data Flows Through the Pipeline

```
SCRAPING (Step 1)
  YouTube RSS → YouTubeVideo rows (transcript=NULL)
  OpenAI RSS  → OpenAIArticle rows (description from RSS)
  Anthropic RSS → AnthropicArticle rows (markdown=NULL)

ENRICHMENT (Steps 2–3)
  AnthropicArticle (markdown=NULL)
    → docling downloads full page
    → AnthropicArticle (markdown="# Claude 4 is here\n...")

  YouTubeVideo (transcript=NULL)
    → YouTube Transcript API fetches audio transcript
    → YouTubeVideo (transcript="Welcome back everyone...")

AI SUMMARIZATION (Step 4)
  YouTubeVideo (transcript != NULL, != "__UNAVAILABLE__")
  + OpenAIArticle (all)
  + AnthropicArticle (markdown != NULL)
    → GPT-4o-mini reads each one
    → Digest row created for each

EMAIL (Step 5)
  Digest (created_at >= 24h ago)
    → CuratorAgent ranks all
    → EmailAgent writes intro
    → send_email() sends HTML email
```

### Primary Keys and Uniqueness

Each table's primary key ensures **no duplicates**:

- `youtube_videos.video_id` — YouTube's video ID is already unique worldwide
- `openai_articles.guid` — the RSS `<guid>` element is unique per article
- `anthropic_articles.guid` — same
- `digests.id` = `f"{type}:{article_id}"` — custom composite key

If the scraper runs twice on the same day and finds the same article, the `existing = session.query(...).filter_by(...).first()` check in the repository prevents a duplicate.

---

## Part 11: Branch Comparison

### The Three-Branch Strategy

The project is organized across three branches, each representing a "checkpoint" in the build:

```
master ─────────────────────────────────────────────►
         Local setup + core functionality

deployment ─────────────────────────────────────────►
         + Docker image + render.yaml + DATABASE_URL support

deployment-final ───────────────────────────────────►
         + Production hardening + refactoring + sent_at tracking
```

### `master` — Core Local Functionality

**What it contains:**
- All scrapers (YouTube, OpenAI, Anthropic)
- Database models, connection, repository
- All services (process_anthropic, process_youtube, process_digest, process_email)
- All agents (DigestAgent, CuratorAgent, EmailAgent)
- User profile
- Docker Compose for local PostgreSQL
- Full pipeline runner

**What it does NOT have:**
- Dockerfile (cannot be containerized yet)
- Deployment configuration
- Support for `DATABASE_URL` environment variable (only supports individual `POSTGRES_*` vars)

**What you learn from `master`:**
- Full Python backend project structure
- SQLAlchemy ORM patterns
- AI agent design
- RSS scraping
- Email sending

**When to use this branch:** For all local development and testing.

---

### `deployment` — Containerization + Cloud Deployment

**New files added:**

| File | What It Does |
|------|-------------|
| `Dockerfile` | Instructions for building a Docker image of the app |
| `.dockerignore` | Files to exclude from the Docker image |
| `render.yaml` | Infrastructure-as-code for Render.com deployment |
| `RENDER_SETUP.md` | Quick setup guide |
| `requirements.txt` | pip-compatible package list (needed by some CI/CD systems) |

**Modified files:**

| File | What Changed | Why |
|------|-------------|-----|
| `app/database/connection.py` | Now checks for `DATABASE_URL` first | Render.com provides `DATABASE_URL` automatically |
| `app/example.env` | Added `DATABASE_URL`, `ENVIRONMENT`, proxy vars | New variables needed for deployment |

**The `Dockerfile` explained:**

```dockerfile
FROM python:3.12-slim
# Start from an official Python 3.12 image (slim = smaller size)

WORKDIR /app
# All commands run from /app directory inside the container

RUN apt-get update && apt-get install -y \
    gcc \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*
# Install system dependencies needed by some Python packages

COPY pyproject.toml ./
# Copy just the dependency file first (so Docker can cache this layer)

RUN pip install --no-cache-dir uv && \
    uv pip install --system -e .
# Install uv, then install all project dependencies

COPY . .
# Copy the rest of the project code

CMD ["python", "main.py"]
# Default command to run when the container starts
```

**The `render.yaml` explained:**

```yaml
databases:
  - name: ai-news-aggregator-db     # Render creates a PostgreSQL database with this name
    databaseName: ai_news_aggregator
    user: ai_news_user
    plan: free                       # free tier!

services:
  - type: cron                       # this is a scheduled job, not a web server
    name: daily-digest-job
    env: docker                      # run using Docker (uses our Dockerfile)
    dockerfilePath: ./Dockerfile
    schedule: "0 0 * * *"           # cron syntax: midnight UTC every day
    dockerCommand: python main.py    # what to run
    envVars:
      - key: DATABASE_URL
        fromDatabase:
          name: ai-news-aggregator-db
          property: connectionString  # Render auto-fills this from the DB above
      - key: OPENAI_API_KEY
        sync: false                   # you set this manually in the Render dashboard
      - key: MY_EMAIL
        sync: false
      - key: APP_PASSWORD
        sync: false
```

**The cron schedule `"0 0 * * *"` explained:**

Cron expressions have 5 fields:
```
┌─── minute (0-59)
│ ┌─── hour (0-23)
│ │ ┌─── day of month (1-31)
│ │ │ ┌─── month (1-12)
│ │ │ │ ┌─── day of week (0-7, 0 and 7 are Sunday)
│ │ │ │ │
0 0 * * *
```

`0 0 * * *` = at minute 0 of hour 0 every day = midnight UTC every day.

`0 5 * * *` = 5:00 AM UTC every day (used in `deployment-final`)

**The key change in `connection.py`:**

```python
def get_database_url() -> str:
    database_url = os.getenv("DATABASE_URL")
    if database_url:
        # Render uses "postgres://" but SQLAlchemy needs "postgresql://"
        if database_url.startswith("postgres://"):
            database_url = database_url.replace("postgres://", "postgresql://", 1)
        return database_url

    # Fallback to individual variables (for local development)
    user = os.getenv("POSTGRES_USER", "postgres")
    ...
```

> 💡 **Video Gap:** This `postgres://` → `postgresql://` fix is critical. Render.com (and Heroku) provide database URLs using the older `postgres://` scheme. SQLAlchemy 1.4+ rejected this, requiring `postgresql://`. Without this fix, the deployed app crashes immediately with a connection error. Many beginners have spent hours debugging this.

---

### `deployment-final` — Production Hardening

This branch adds polish and robustness to the deployed system.

**New files:**

| File | What It Does |
|------|-------------|
| `app/agent/base.py` | `BaseAgent` abstract class — shared initialization for all agents |
| `app/scrapers/base.py` | `BaseScraper` abstract class — shared RSS parsing logic for OpenAI/Anthropic scrapers |
| `app/services/base.py` | Base class for services |
| `app/database/check_connection.py` | Diagnostic script to verify DB connection and table state |
| `app/database/README.md` | Documentation for database management |
| `docs/DEPLOYMENT.md` | Full deployment guide |
| `docs/RENDER_SETUP.md` | Quick Render setup guide |

**Key changes:**

**1. Auto-create tables on startup (`daily_runner.py`)**

```python
logger.info("\n[0/5] Ensuring database tables exist...")
Base.metadata.create_all(engine)
logger.info("✓ Database tables verified/created")
```

In production, you cannot run `create_tables.py` manually before the first run. Adding this to the pipeline runner means tables are created automatically on the first deploy. This is crucial for a working automated deployment.

**2. `sent_at` column on `Digest` model**

```python
class Digest(Base):
    ...
    sent_at = Column(DateTime, nullable=True)  # NEW
```

This tracks when a digest was included in an email. The new `mark_digests_as_sent()` repository method sets this after sending.

**Why this matters:** Without this, every time the pipeline runs, it sends ALL digests from the last 24 hours — even ones that were already sent. With `sent_at`, you can filter: "only send digests that have not been emailed yet."

**3. Skip-if-nothing-new logic (`process_email.py`)**

```python
def send_digest_email(hours: int = 24, top_n: int = 10) -> dict:
    repo = Repository()
    digests = repo.get_recent_digests(hours=hours)

    if len(digests) == 0:
        logger.info("No new digests to send. Nothing to send.")
        return {
            "success": True,
            "skipped": True,
            "message": "No new digests available",
            "articles_count": 0,
        }
    ...
```

If no new content was published today, the pipeline skips sending an email instead of failing. This prevents 500-error email failures when there is simply nothing to report.

**4. Refactoring with base classes (`BaseScraper`)**

```python
# Before (deployment branch): OpenAI and Anthropic scrapers
# had identical RSS parsing code copy-pasted into each

# After (deployment-final): common code in BaseScraper
class BaseScraper(ABC):
    @property
    @abstractmethod
    def rss_urls(self) -> List[str]:
        pass

    def get_articles(self, hours: int = 24) -> List[Article]:
        # shared RSS parsing logic (was duplicated in both scrapers)
        ...

class OpenAIScraper(BaseScraper):
    @property
    def rss_urls(self):
        return ["https://openai.com/news/rss.xml"]
    # get_articles() is inherited from BaseScraper — no duplication!
```

This is the **DRY principle** (Don't Repeat Yourself) — a core programming practice.

**5. Improved Dockerfile**

```dockerfile
# Before: pip install uv, then uv pip install
RUN pip install --no-cache-dir uv && uv pip install --system -e .

# After: copy uv binary directly from official image (faster, more reliable)
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
RUN uv pip install --system -r pyproject.toml
CMD ["uv", "run", "main.py"]
```

**6. Cron schedule change**

`render.yaml` cron changed from `"0 0 * * *"` (midnight UTC) to `"0 5 * * *"` (5 AM UTC). This ensures the digest runs after most of Europe and the US East Coast have had time to publish overnight content.

### What to Learn From Each Branch

| Branch | Key Learning |
|--------|-------------|
| `master` | Python project structure, SQLAlchemy ORM, AI agents, RSS scraping, email sending |
| `deployment` | Docker containerization, cloud deployment basics, environment-aware configuration, cron jobs |
| `deployment-final` | Production hardening, DRY principle/refactoring, database migrations, skip logic, operational tooling |

**Recommended learning path:**
1. Get `master` working locally completely
2. Study the `deployment` changes file-by-file
3. Study the `deployment-final` changes and understand *why* each change was made
4. Deploy using `deployment-final`


---

## Part 12: Deployment to Render.com

### What "Deployment" Means

When you run `python main.py` on your laptop, the pipeline only works while:
- Your laptop is on
- Your laptop is connected to the internet
- You manually trigger it

**Deployment** means running your code on someone else's computer — a server in the cloud — that is always on, always connected, and runs your code automatically on a schedule.

After deployment:
- Your laptop can be off
- The pipeline runs at 5 AM UTC every day automatically
- You receive your email digest without doing anything

### What Render.com Is

Render.com is a cloud hosting platform. It is simpler than AWS or Google Cloud — designed for developers who want to deploy without needing to manage servers. The free tier is generous enough for this project.

What Render provides for this project:
- A **PostgreSQL database** running 24/7 (free tier: 1 GB storage, 90 days active)
- A **cron job** that runs `python main.py` on a schedule (free tier: available)
- **Automatic restarts** if the job fails

### Step 1: Push Your Code to GitHub

Before deploying to Render, your code must be on GitHub. Render will pull your code from there.

**If you do not have a GitHub account:**
1. Go to github.com → Sign up → Create a free account

**Create a new GitHub repository:**
1. On GitHub, click the **+** button → **New repository**
2. Repository name: `ai-news-aggregator`
3. Set to **Private** (your code contains your profile and configuration)
4. Do NOT initialize with README (you already have code)
5. Click **Create repository**

**GitHub will show you commands to connect your local code. Use the "push an existing repository" option:**

First, make sure you are in the project folder:
```bash
cd /Users/svetlanaonopa/Desktop/Projects/AI_Project/Git_repo_clone/ai-news-aggregator
```

Initialize git (if the project is not already a git repository):
```bash
git init
git add .
git commit -m "Initial commit: AI News Aggregator"
```

**Before adding all files, verify `.env` is gitignored:**
```bash
git status
```

Make sure `.env` does NOT appear in the list of files. If it does:
```bash
echo ".env" >> .gitignore
git add .gitignore
```

**Connect to GitHub and push:**
```bash
git remote add origin https://github.com/yourusername/ai-news-aggregator.git
git branch -M main
git push -u origin main
```

Replace `yourusername` with your actual GitHub username.

**What you should see:**
```
Enumerating objects: 45, done.
Counting objects: 100% (45/45), done.
Writing objects: 100% (45/45), ...
To https://github.com/yourusername/ai-news-aggregator.git
 * [new branch]      main -> main
```

**Verify:** Go to `github.com/yourusername/ai-news-aggregator` in your browser. You should see all your project files. Confirm `.env` is NOT visible.

**Switch to the `deployment-final` branch for deployment:**

For production deployment, use the `deployment-final` branch (it has the most robust code):
```bash
git checkout -b deployment-final origin/deployment-final
git push -u origin deployment-final
```

> **Note:** If you have been working from the cloned repository (not building from scratch), both branches already exist. Just push both:
> ```bash
> git push origin master
> git push origin deployment-final
> ```

---

### Step 2: Create a Render Account

1. Go to **render.com**
2. Click **Get Started for Free**
3. Sign up with your GitHub account (recommended — makes connecting easier)
4. Verify your email address

**What you should see:** The Render dashboard — a clean interface showing "No services yet."

---

### Step 3: Deploy Using `render.yaml` (Blueprint)

The `render.yaml` file in your repository tells Render exactly what infrastructure to create. You do not need to click through settings manually — Render reads the file and sets everything up.

**Step 3a: Create a New Blueprint**

1. In the Render dashboard, click **New +** → **Blueprint**
2. Click **Connect a repository**
3. If prompted, click **Configure account** → authorize Render to access GitHub
4. Select your `ai-news-aggregator` repository
5. Select the `deployment-final` branch
6. Click **Connect**

**What you should see:** Render reads your `render.yaml` file and shows you a preview of what it will create:
```
Services to be created:
  ✓ PostgreSQL Database: ai-news-aggregator-db
  ✓ Cron Job: daily-digest-job
```

7. Click **Apply**

**What happens next:** Render creates the database and the cron job service. This takes 2–5 minutes. You will see a progress screen.

**What you should see after it finishes:**
```
✓ ai-news-aggregator-db (PostgreSQL)  Active
✓ daily-digest-job (Cron Job)          Suspended (waiting for first run)
```

---

### Step 4: Set Your Environment Variables

The `render.yaml` file sets `DATABASE_URL` automatically. But your personal secrets (`OPENAI_API_KEY`, email credentials) must be set manually in the Render dashboard.

**Step 4a: Navigate to the cron job service**

1. In the Render dashboard, click **daily-digest-job**
2. In the left sidebar, click **Environment**

**Step 4b: Add your environment variables**

Click **Add Environment Variable** for each one:

| Key | Value |
|-----|-------|
| `OPENAI_API_KEY` | `sk-proj-your-actual-key` |
| `MY_EMAIL` | `svetlana@gmail.com` |
| `APP_PASSWORD` | `your16charapppassword` |

> **Do NOT add `DATABASE_URL`** — Render sets this automatically from your `render.yaml` configuration. If you add it manually, it might conflict.

Click **Save Changes**.

**What you should see:** The variables are saved and shown with their values masked (••••••).

---

### Step 5: Trigger a Manual Test Run

Do not wait until midnight to find out if your deployment works. Trigger it manually:

1. In the Render dashboard, click **daily-digest-job**
2. Click **Manual Deploy** (or **Trigger Deploy** in newer UI)
3. Click **Deploy**

**What you should see:** A "Deploying..." status, then Render starts the Docker container and runs `python main.py`.

---

### Step 6: Read the Deployment Logs

1. Click on the running deployment to see its logs
2. You should see the same pipeline output you see locally:

```
2026-06-19 05:00:00 - INFO - ============================================================
2026-06-19 05:00:00 - INFO - Starting Daily AI News Aggregator Pipeline
...
2026-06-19 05:00:05 - INFO - ✓ Scraped 2 YouTube videos, 3 OpenAI articles, 5 Anthropic articles
...
2026-06-19 05:02:45 - INFO - ✓ Email sent successfully with 10 articles
```

**If you see errors:** Read Part 13 (Debugging) for common deployment errors and fixes.

**If you see "Exit 0":** The job completed successfully.
**If you see "Exit 1":** The job failed. Read the logs to find the error.

---

### Step 7: Verify the Database Was Set Up

On the first run, `daily_runner.py` (deployment-final) automatically creates the tables. Verify this worked:

1. In Render dashboard → **ai-news-aggregator-db** (the database service)
2. Click **Info** → scroll down to **Connection**
3. Note the **External Database URL** (you can use this to connect with a tool like TablePlus or DBeaver)

Or use the Render shell (if available on your plan): connect and run:
```sql
\dt
SELECT COUNT(*) FROM digests;
```

---

### Step 8: Check Your Email

After the pipeline completes (look for "Exit 0" in logs), check your inbox.

**Subject line format:** `Daily AI News Digest - June 19, 2026`

**What you should see:** A formatted HTML email with:
- A personalized greeting: "Hey Svetlana, here is your daily digest..."
- An introduction paragraph
- 10 ranked articles with titles, summaries, and "Read more →" links

---

### Step 9: Understand the Automatic Schedule

Once deployed, Render will run the cron job automatically according to the schedule in `render.yaml`:

**`deployment` branch:** `"0 0 * * *"` = midnight UTC
**`deployment-final` branch:** `"0 5 * * *"` = 5:00 AM UTC

**Convert UTC to your timezone:**
- UTC → Moscow (MSK): add 3 hours → 8:00 AM
- UTC → Eastern US (ET): subtract 5 hours (winter) or 4 hours (summer)
- UTC → UK (GMT/BST): same as UTC in winter, +1 in summer

**To change the schedule**, edit `render.yaml`:
```yaml
schedule: "0 8 * * *"  # 8 AM UTC = 11 AM Moscow
```

Then push to GitHub. Render will automatically pick up the change on the next deploy.

**Cron schedule reference:**
```
"0 5 * * *"    # 5 AM every day
"0 5 * * 1-5"  # 5 AM Monday-Friday only
"0 5 * * 1"    # 5 AM every Monday
"30 7 * * *"   # 7:30 AM every day
```

---

### Understanding What Each Render Setting Means

| Setting | Value | What It Means |
|---------|-------|---------------|
| `type: cron` | — | This is a scheduled job, not a long-running server |
| `env: docker` | — | Use our Dockerfile to build the container |
| `dockerfilePath: ./Dockerfile` | — | Where to find the Dockerfile |
| `schedule: "0 5 * * *"` | 5 AM UTC | When to run the job |
| `dockerCommand: python main.py` | — | What to execute inside the container |
| `plan: free` | — | Use the free tier |

---

### Common Deployment Errors and How to Fix Them

**Error: `Error: relation "youtube_videos" does not exist`**

The tables were not created. Make sure you are using the `deployment-final` branch where `daily_runner.py` auto-creates tables. Or run `create_tables.py` manually (connect to the DB from your local machine using the External DB URL).

**Error: `SMTPAuthenticationError`**

Check `MY_EMAIL` and `APP_PASSWORD` in Render's environment variables. The App Password must have no spaces. Gmail 2-Step Verification must be enabled.

**Error: `openai.AuthenticationError: Incorrect API key`**

Check `OPENAI_API_KEY` in Render's environment variables. Copy it fresh from platform.openai.com.

**Error: `postgres:// is not a valid URL scheme`**

You are NOT using the `deployment` or `deployment-final` branch. The `master` branch's `connection.py` does not handle this. Switch to `deployment-final`.

**Error: Build fails with `cannot install psycopg2`**

Make sure your Dockerfile installs `gcc` and `postgresql-client`. The `deployment` Dockerfile includes this:
```dockerfile
RUN apt-get update && apt-get install -y gcc postgresql-client
```

**Error: `No digests available` / Email not sent**

The pipeline ran but found no articles. This happens when:
- No new content was published in the last 24 hours on monitored channels
- The `hours` parameter is too small
- The scrapers returned 0 articles (check step 1 logs)

Solution for testing: trigger a manual deploy and check the logs. If scrapers return 0 results, expand the hours window by setting `hours=168` in `main.py`.

**Cron job not running on schedule:**

- Check that the service is not "Suspended" in the Render dashboard
- Verify the schedule syntax at crontab.guru
- Note: Render free tier cron jobs may have slight delays

---

## Part 13: Debugging Guide

### How to Read Python Error Messages

Python errors have a specific structure:
```
Traceback (most recent call last):
  File "main.py", line 5, in <module>
    result = main()
  File "main.py", line 3, in main
    return run_daily_pipeline()
  File "app/daily_runner.py", line 38, in run_daily_pipeline
    scraping_results = run_scrapers(hours=hours)
  File "app/runner.py", line 8, in run_scrapers
    from .config import YOUTUBE_CHANNELS
ModuleNotFoundError: No module named 'app'
```

**Read from the bottom up.** The last line is the actual error. The lines above show the "call stack" — how Python got there.

In this example: `ModuleNotFoundError: No module named 'app'` — Python cannot find the `app` package.

---

### Error 1: `command not found: uv`

**What it means:** `uv` is not installed, or not in your PATH.

**Solution:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# Restart your terminal after this
```

If already installed but not found:
```bash
# Add uv to PATH manually (Mac/Linux)
export PATH="$HOME/.local/bin:$PATH"
# Add this line to ~/.zshrc or ~/.bashrc for permanent fix
```

---

### Error 2: `command not found: code`

**What it means:** VS Code's command-line tool is not installed.

**Solution:**
1. Open VS Code
2. Press Cmd+Shift+P → type "Shell Command" → click "Install 'code' command in PATH"
3. Restart your terminal

---

### Error 3: Docker Not Running

**Symptoms:**
```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock
```
or
```
Error response from daemon: dial unix /var/run/docker.sock: connect: no such file or directory
```

**Solution:**
1. Open Docker Desktop (the application)
2. Wait for the whale icon in your menu bar to show a green dot
3. Re-run your `docker compose` command

---

### Error 4: Database Connection Failed

**Symptoms:**
```
sqlalchemy.exc.OperationalError: (psycopg2.OperationalError) could not connect to server: Connection refused
Is the server running on host "localhost" and accepting TCP/IP connections on port 5432?
```

**Causes and solutions:**

| Cause | Solution |
|-------|---------|
| Docker container not started | `docker compose -f docker/docker-compose.yml up -d` |
| Container is starting up (not ready yet) | Wait 10 seconds, try again |
| Wrong port in `.env` | Check `POSTGRES_PORT=5432` |
| Another service using port 5432 | `lsof -i :5432` to find it, or change port |
| Database container crashed | `docker logs ai-news-aggregator-db` to see why |

**Check container status:**
```bash
docker ps
# Look for ai-news-aggregator-db with STATUS "Up" and "(healthy)"
```

---

### Error 5: `ModuleNotFoundError`

**Symptoms:**
```
ModuleNotFoundError: No module named 'feedparser'
ModuleNotFoundError: No module named 'openai'
```

**Causes and solutions:**

| Cause | Solution |
|-------|---------|
| Dependencies not installed | `uv sync` |
| Running Python outside virtual environment | Use `uv run python ...` instead of `python ...` |
| Wrong Python interpreter in VS Code | Cmd+Shift+P → "Python: Select Interpreter" → choose `.venv` |

---

### Error 6: Wrong Python Interpreter in VS Code

**Symptoms:**
- VS Code shows red underlines under `import feedparser` even though it installed fine
- Running code in VS Code's terminal shows `ModuleNotFoundError`

**Solution:**
1. Press Cmd+Shift+P → "Python: Select Interpreter"
2. You should see `Python 3.12.x ('.venv': uv)` in the list
3. Select it
4. Reload VS Code (Cmd+Shift+P → "Developer: Reload Window")

---

### Error 7: Package Installed But Not Visible

**Symptoms:** `uv sync` ran without errors, but `python -c "import openai"` fails.

**Solution:**

Always use `uv run` to execute Python:
```bash
# Wrong (may use system Python):
python -c "import openai"

# Correct (uses project virtual environment):
uv run python -c "import openai"
```

Or activate the virtual environment first:
```bash
source .venv/bin/activate  # Mac/Linux
# Then:
python -c "import openai"  # works because .venv is active
```

---

### Error 8: Port Already in Use

**Symptoms:**
```
Error response from daemon: Ports are not available: listen tcp 0.0.0.0:5432: bind: address already in use
```

**What it means:** Something else on your computer is using port 5432 (another PostgreSQL installation, or another Docker container).

**Solution:**
```bash
# Find what is using port 5432:
lsof -i :5432   # Mac/Linux

# Option A: Stop whatever is using it (if it is another PostgreSQL)
brew services stop postgresql  # if installed via Homebrew

# Option B: Change the port in docker-compose.yml
# Change: "5432:5432"  to:  "5433:5432"
# And in .env: POSTGRES_PORT=5433
```

---

### Error 9: `.env` Not Loaded / `os.getenv()` Returns `None`

**Symptoms:**
```
ValueError: OPENAI_API_KEY environment variable is not set
```
or API calls fail with authentication errors.

**Causes and solutions:**

| Cause | Solution |
|-------|---------|
| `.env` file does not exist | `cp app/example.env .env` then fill in values |
| `load_dotenv()` not called before `os.getenv()` | Check that file calls `load_dotenv()` at the top |
| `.env` file in wrong location | It must be in the project root, same folder as `main.py` |
| Variable name typo | Check exact spelling: `OPENAI_API_KEY` not `OPENAI_KEY` |
| Spaces around `=` sign | Wrong: `KEY = value` → Correct: `KEY=value` |

**Debug what variables are loaded:**
```bash
uv run python -c "
from dotenv import load_dotenv
import os
load_dotenv()
print('OPENAI_API_KEY:', 'SET' if os.getenv('OPENAI_API_KEY') else 'NOT SET')
print('MY_EMAIL:', os.getenv('MY_EMAIL'))
print('POSTGRES_HOST:', os.getenv('POSTGRES_HOST'))
"
```

---

### Error 10: Email Not Sending (`SMTPAuthenticationError`)

**Symptoms:**
```
smtplib.SMTPAuthenticationError: (535, b'5.7.8 Username and Password not accepted.')
```

**Causes and solutions:**

| Cause | Solution |
|-------|---------|
| Using your regular Gmail password | You need an App Password (16 chars), not your regular password |
| 2-Step Verification not enabled | Go to Google Account → Security → enable 2-Step Verification |
| App Password copied with spaces | Remove all spaces: `abcd efgh ijkl mnop` → `abcdefghijklmnop` |
| Wrong Gmail address | Check `MY_EMAIL` in `.env` matches the account the App Password was created for |
| App Password expired or revoked | Create a new one at myaccount.google.com → Security → App Passwords |

**Step-by-step: Create a new Gmail App Password**
1. Go to myaccount.google.com
2. Click **Security** in the left sidebar
3. Under "How you sign in to Google," verify **2-Step Verification** is ON
4. In the search bar at the top of the security page, type "App passwords"
5. Click **App passwords**
6. Under "App name," type something like `AI News Aggregator`
7. Click **Create**
8. Google shows you a 16-character password (with spaces for readability)
9. Copy it and remove the spaces when pasting into `.env`

---

### Error 11: `openai.RateLimitError`

**Symptoms:**
```
openai.RateLimitError: 429 Rate limit exceeded
```

**What it means:** You have hit OpenAI's rate limits — too many API requests in a short time.

**Solutions:**
- Wait a few minutes and try again
- Add a billing payment method to your OpenAI account (higher rate limits)
- The pipeline already processes articles one-by-one, which helps with rate limits

---

### Error 12: YouTube Transcripts Return `None` for All Videos

**Symptoms:** All videos show `transcript = "__UNAVAILABLE__"` in the database.

**Possible causes:**
1. **Channels with transcripts disabled** — some creators disable auto-generated captions
2. **IP blocking** — YouTube blocks transcript requests from certain cloud or VPN IP addresses
3. **Region restrictions** — some transcripts are not available in all countries

**Solutions:**
- Try videos from different channels to see if it is channel-specific
- For local development, this usually works fine. For cloud deployment, you may need the Webshare proxy (set `PROXY_USERNAME` and `PROXY_PASSWORD` in `.env`)
- Videos without transcripts are simply excluded from digest generation — the pipeline continues normally

---

## Part 14: Learning Checkpoints

### After Part 1 (Project Overview)

**You should now understand:**
- [ ] What this project does in plain English
- [ ] Why a database is used (to pass data between pipeline steps)
- [ ] The 5 steps of the pipeline
- [ ] What the final email looks like

**Check yourself:**
> If someone asked "what does this project do?", could you explain it in 3 sentences without mentioning code?

---

### After Part 2 (Architecture)

**You should now understand:**
- [ ] How data flows from the internet → scrapers → database → AI → email
- [ ] What each folder (`scrapers/`, `database/`, `services/`, `agent/`) is responsible for
- [ ] How files import each other (the dependency chain)

**Check yourself:**
> Draw the architecture diagram from memory. Include all 5 pipeline steps and the database.

---

### After Part 3 (Key Concepts)

**You should now understand:**
- [ ] What a virtual environment is and why uv manages it
- [ ] What PostgreSQL is and why it runs in Docker
- [ ] What SQLAlchemy's ORM pattern does
- [ ] What an RSS feed is
- [ ] What `.env` is for and why secrets must not be in code
- [ ] What Pydantic models are for

**Check yourself:**
> What is the difference between an SQLAlchemy `engine`, a `session`, and a `model`?

---

### After Part 4 (Setup Guide)

**You should be able to:**
- [ ] Create a `.env` file from `example.env`
- [ ] Run `uv sync` to install dependencies
- [ ] Start PostgreSQL with Docker Compose
- [ ] Create database tables
- [ ] Run the full pipeline with `python main.py`
- [ ] Receive the email in your inbox

**Check yourself:**
> If you deleted your `.venv` folder and your Docker volume, could you restore everything from scratch using this guide?

---

### After Parts 5–6 (File and Function Explanations)

**You should be able to:**
- [ ] Open any file in the project and explain what it does
- [ ] Trace what happens when `main.py` is run (which functions are called in what order)
- [ ] Explain why each `__init__.py` exists
- [ ] Explain the `__UNAVAILABLE__` transcript marker
- [ ] Explain why `load_dotenv()` must be called before `os.getenv()`

**Check yourself:**
> What happens if `app/database/__init__.py` is deleted? (Hint: try it and read the error)

---

### After Part 7 (Build Roadmap)

**You should be able to:**
- [ ] Create the project structure from scratch
- [ ] Write the database models from memory (or near-memory)
- [ ] Create and run the scrapers independently
- [ ] Generate AI summaries for a batch of articles
- [ ] Send a test email

**Check yourself:**
> Start a new empty folder and rebuild Lesson 1 (project skeleton) from scratch without looking at this guide.

---

### After Parts 8–10 (Config, Docker, Database)

**You should be able to:**
- [ ] Explain every variable in `.env`
- [ ] Explain what `docker compose up -d` does step-by-step
- [ ] Connect to the running PostgreSQL database and run queries
- [ ] Explain the primary key strategy for each table
- [ ] Trace a piece of data from its RSS entry to its final row in `digests`

**Check yourself:**
> Run `docker exec -it ai-news-aggregator-db psql -U postgres -d ai_news_aggregator` and run these queries:
> - `SELECT COUNT(*) FROM youtube_videos;`
> - `SELECT title, article_type FROM digests LIMIT 5;`
> - `SELECT guid, title, markdown IS NOT NULL as has_markdown FROM anthropic_articles LIMIT 5;`

---

### After Part 11 (Branch Comparison)

**You should understand:**
- [ ] Why `deployment` adds `DATABASE_URL` support
- [ ] Why `postgres://` must be changed to `postgresql://`
- [ ] What `sent_at` column adds (prevents re-sending already-emailed articles)
- [ ] What the `BaseScraper` refactor achieves (DRY principle)
- [ ] Why `Base.metadata.create_all(engine)` was added to `daily_runner.py`

**Check yourself:**
> Look at the diff between `master` and `deployment-final` for `connection.py`. Can you explain every line that changed and why?

---

### After Part 12 (Deployment)

**You should be able to:**
- [ ] Create a GitHub repository and push code to it
- [ ] Deploy using Render's Blueprint feature
- [ ] Set environment variables in the Render dashboard
- [ ] Trigger a manual run and read the deployment logs
- [ ] Customize the cron schedule
- [ ] Diagnose and fix common deployment errors

**Check yourself:**
> Trigger a manual deploy and follow the logs in real-time. Before checking your email, predict from the logs: how many articles were scraped? How many digests were created? Did the email send successfully?

---

## Part 15: Final Reference

### Command Cheat Sheet

#### Setup Commands

```bash
# Install dependencies
uv sync

# Start database
docker compose -f docker/docker-compose.yml up -d

# Stop database (keeps data)
docker compose -f docker/docker-compose.yml down

# Reset database (deletes all data)
docker compose -f docker/docker-compose.yml down -v

# Create tables (run once)
uv run python app/database/create_tables.py
```

#### Running the Pipeline

```bash
# Run full pipeline (last 24 hours, top 10 articles)
uv run python main.py

# Run with custom parameters (last 7 days, top 5 articles)
uv run python main.py 168 5

# Run only the scraping step
uv run python app/runner.py

# Run only transcript fetching
uv run python app/services/process_youtube.py

# Run only Anthropic markdown fetching
uv run python app/services/process_anthropic.py

# Run only AI summarization
uv run python app/services/process_digest.py

# Run only email sending
uv run python app/services/process_email.py
```

#### Database Commands

```bash
# Open database SQL prompt
docker exec -it ai-news-aggregator-db psql -U postgres -d ai_news_aggregator

# Inside SQL prompt:
# List tables:              \dt
# Describe table:           \d youtube_videos
# Count rows:               SELECT COUNT(*) FROM digests;
# View recent digests:      SELECT title, created_at FROM digests ORDER BY created_at DESC LIMIT 10;
# View all videos:          SELECT title, transcript IS NOT NULL as has_transcript FROM youtube_videos;
# Exit:                     \q
```

#### Docker Commands

```bash
# See running containers
docker ps

# See ALL containers (including stopped)
docker ps -a

# See container logs
docker logs ai-news-aggregator-db

# Follow container logs (live)
docker logs -f ai-news-aggregator-db

# Remove all stopped containers and unused images
docker system prune
```

#### Git Commands (for Deployment)

```bash
# Check status
git status

# Stage all files
git add .

# Commit
git commit -m "Your commit message"

# Push to GitHub
git push

# Switch branches
git checkout deployment-final

# See all branches
git branch -a
```

#### Testing Individual Components

```bash
# Test YouTube scraper
uv run python -c "
from app.scrapers.youtube import YouTubeScraper
scraper = YouTubeScraper()
videos = scraper.get_latest_videos('UCawZsQWqfGSbCI5yjkdVkTA', hours=168)
print(f'{len(videos)} videos found')
"

# Test email sending
uv run python -c "
from dotenv import load_dotenv; load_dotenv()
from app.services.email import send_email
send_email('Test', 'Hello Svetlana!', '<h1>Hello!</h1>')
print('Sent!')
"

# Test database connection
uv run python -c "
from app.database.connection import engine
from sqlalchemy import text
with engine.connect() as conn:
    result = conn.execute(text('SELECT 1'))
    print('Database connected:', result.scalar())
"
```

---

### File Creation Checklist (Building from Scratch)

Use this checklist when building the project from zero:

#### Project Foundation
- [ ] `pyproject.toml` — dependencies
- [ ] `.python-version` — Python 3.12
- [ ] `.gitignore` — includes `.env`, `.venv/`, `__pycache__/`
- [ ] `main.py` — entry point
- [ ] `.env` — your secrets (copied from `example.env`)

#### Folder Structure (all need `__init__.py`)
- [ ] `app/__init__.py`
- [ ] `app/scrapers/__init__.py`
- [ ] `app/database/__init__.py`
- [ ] `app/services/__init__.py`
- [ ] `app/agent/__init__.py`
- [ ] `app/profiles/__init__.py`
- [ ] `docker/` folder

#### Configuration
- [ ] `app/config.py` — YouTube channel IDs
- [ ] `app/profiles/user_profile.py` — your profile (name: Svetlana!)
- [ ] `app/example.env` — template (safe to commit)
- [ ] `docker/docker-compose.yml` — PostgreSQL setup

#### Database Layer
- [ ] `app/database/models.py` — 4 table models
- [ ] `app/database/connection.py` — SQLAlchemy engine and session
- [ ] `app/database/repository.py` — all read/write functions
- [ ] `app/database/create_tables.py` — one-time setup script

#### Scrapers
- [ ] `app/scrapers/youtube.py` — YouTube RSS + transcripts
- [ ] `app/scrapers/openai.py` — OpenAI blog RSS
- [ ] `app/scrapers/anthropic.py` — Anthropic blog RSS + docling

#### Agents (AI)
- [ ] `app/agent/digest_agent.py` — GPT-4o-mini summarizer
- [ ] `app/agent/curator_agent.py` — GPT-4.1 ranker
- [ ] `app/agent/email_agent.py` — GPT-4o-mini email writer

#### Services (Orchestration)
- [ ] `app/services/process_anthropic.py` — enrich articles
- [ ] `app/services/process_youtube.py` — fetch transcripts
- [ ] `app/services/process_digest.py` — generate summaries
- [ ] `app/services/process_email.py` — rank + send email
- [ ] `app/services/email.py` — SMTP sending

#### Pipeline
- [ ] `app/runner.py` — scraping step
- [ ] `app/daily_runner.py` — full pipeline orchestrator

#### For Deployment (deployment-final branch)
- [ ] `Dockerfile` — build instructions
- [ ] `.dockerignore` — exclude files from image
- [ ] `render.yaml` — Render.com infrastructure
- [ ] `requirements.txt` — pip-compatible dependency list

---

### Dependency Checklist

| Package | What It Does | Used In |
|---------|-------------|---------|
| `feedparser` | Parses RSS feeds | All scrapers |
| `docling` | Converts web pages to Markdown | Anthropic scraper |
| `youtube-transcript-api` | Fetches YouTube transcripts | YouTube scraper |
| `openai` | OpenAI API client | All agents |
| `pydantic` | Data validation and models | Scrapers, agents |
| `sqlalchemy` | ORM / database toolkit | database/ |
| `psycopg2-binary` | PostgreSQL driver | database/connection.py |
| `python-dotenv` | Reads `.env` files | All files with `load_dotenv()` |
| `requests` | HTTP requests | (indirect, used by docling) |
| `beautifulsoup4` | HTML parsing | (indirect, used by docling) |
| `markdown` | Converts Markdown to HTML | services/email.py |
| `markdownify` | Converts HTML to Markdown | (available if needed) |

---

### Recommended Order to Rebuild the Project

If you are rebuilding from scratch for maximum learning, follow this order:

```
Phase 1: Foundation (no code runs yet)
├─ Create project folder and initialize uv
├─ Write pyproject.toml with dependencies
├─ Run `uv sync`
├─ Create all __init__.py files
├─ Create .gitignore and .env
└─ Create main.py (stub)

Phase 2: Database (can test with psql)
├─ Write database/models.py
├─ Write database/connection.py
├─ Write database/create_tables.py
├─ Start Docker: `docker compose up -d`
└─ Run create_tables.py — verify 4 tables exist

Phase 3: Scrapers (can test independently)
├─ Write app/config.py
├─ Write scrapers/youtube.py — test: get_latest_videos()
├─ Write scrapers/openai.py — test: get_articles()
└─ Write scrapers/anthropic.py — test: get_articles()

Phase 4: Database operations (can test with DB queries)
├─ Write database/repository.py
├─ Write runner.py
└─ Run runner.py — verify data appears in DB tables

Phase 5: Enrichment services (data gets richer)
├─ Write services/process_anthropic.py
├─ Run it — verify markdown column fills in
├─ Write services/process_youtube.py
└─ Run it — verify transcript column fills in

Phase 6: AI agents (requires OpenAI API key)
├─ Write agent/digest_agent.py
├─ Write services/process_digest.py
├─ Run process_digest.py — verify digests table fills in
├─ Write profiles/user_profile.py (use YOUR interests!)
└─ Write agent/curator_agent.py

Phase 7: Email (requires Gmail App Password)
├─ Write services/email.py
├─ Test with a simple test email first
├─ Write agent/email_agent.py
└─ Write services/process_email.py

Phase 8: Full pipeline
├─ Write daily_runner.py
├─ Run main.py — verify full pipeline
└─ Check your inbox!

Phase 9: Deployment (after everything works locally)
├─ Study deployment branch changes
├─ Update connection.py for DATABASE_URL support
├─ Write Dockerfile
├─ Write render.yaml
├─ Push to GitHub
└─ Deploy to Render.com
```

---

### Full Project Architecture Summary

```
AI NEWS AGGREGATOR — COMPLETE ARCHITECTURE

INPUT SOURCES                    OUTPUT
─────────────                    ──────
YouTube RSS Feeds ──┐            Daily Email Digest
OpenAI Blog RSS ────┤            to: svetlana@gmail.com
Anthropic RSS (×3) ─┤            subject: "Daily AI News Digest - [date]"
                    │            contents: top 10 ranked articles
                    ▼
              ┌─────────────┐
              │  SCRAPERS   │  Reads RSS, filters by date,
              │  (Step 1)   │  saves to database
              └──────┬──────┘
                     │
                     ▼
              ┌─────────────────────────────┐
              │   POSTGRESQL DATABASE        │
              │   (Docker locally,           │
              │    Render.com in production) │
              │                             │
              │  youtube_videos             │
              │  openai_articles            │
              │  anthropic_articles         │
              │  digests                    │
              └──────┬──────────────────────┘
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   Fetch         Fetch AI     Generate
   Transcripts   Markdowns    Summaries
   (Step 3)      (Step 2)     (Step 4)
        │            │            │
        └────────────┴────────────┘
                     │ all enriched data
                     ▼
              ┌─────────────┐
              │ AI AGENTS   │  Rank by profile + write email
              │  (Step 5)   │  GPT-4.1 (curator)
              │             │  GPT-4o-mini (digest + email)
              └──────┬──────┘
                     │
                     ▼
              ┌─────────────┐
              │   GMAIL     │  Send via SMTP (smtplib)
              │   SMTP      │
              └──────┬──────┘
                     │
                     ▼
              📧 Your Inbox

AUTOMATION: Render.com cron job runs this at 5 AM UTC every day
STORAGE:    PostgreSQL remembers everything between runs
SECRETS:    .env file (never committed to Git)
```

---

### What to Build Next (After This Project)

Once you have this project working, here are natural extensions to deepen your learning:

1. **Add more sources:** Add an X/Twitter scraper, Reddit scraper (r/MachineLearning), or arXiv paper scraper
2. **Web dashboard:** Build a simple Flask or FastAPI web UI to browse digests instead of just reading email
3. **Multiple recipients:** Support sending to multiple email addresses (e.g., for a team)
4. **Better deduplication:** Detect when the same story appears in multiple sources and merge them
5. **Feedback loop:** Add a "rate this article" link in the email to improve future rankings
6. **Alembic migrations:** Replace `create_tables.py` with proper migration management (industry standard)
7. **Testing:** Write unit tests for scrapers and agents using `pytest`
8. **Monitoring:** Add error notifications (Slack, Discord) when the pipeline fails

---

*This tutorial was created for Svetlana as a companion to the AI News Aggregator live-coding project. Every concept was explained from the ground up, and every gap from the original video was filled in. The goal was not just to make the project run, but to understand every part of how and why it works.*

