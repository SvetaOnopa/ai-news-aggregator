# AI News Aggregator — Study Plan

> **Total estimated time:** 30–38 hours across 14 sessions
> **Pace:** 1–2 sessions per day (each session is self-contained with a clear stopping point)
> **Prerequisite:** Basic Python knowledge (variables, functions, lists, dictionaries, loops)

---

## Session 1: Understand the Big Picture

**Estimated duration:** 1.5–2 hours (reading only, no coding)

**Chapters to read:**
- Part 0: Before You Begin (lines 30–85)
- Part 1: Project Overview (lines 87–196)

**Concepts to learn:**
- What a data pipeline is and why each step is separate
- Why a database is used to pass data between pipeline steps (not Python variables)
- The 5-step pipeline: scrape → enrich → summarize → rank → email
- What RSS feeds, API keys, and SMTP are (just awareness — details come later)

**Coding tasks:** None. This is a reading-only session.

**Commands to run:**
```bash
# Open the project folder and browse the files — just look, don't change anything
cd /Users/svetlanaonopa/Desktop/Projects/AI_Project/Git_repo_clone/ai-news-aggregator
ls
ls app/
ls app/scrapers/
ls app/database/
ls app/services/
ls app/agent/
```

**Success criteria:**
- [ ] You can explain what this project does in 3 sentences to a friend, without mentioning any code
- [ ] You can name all 5 pipeline steps from memory
- [ ] You can explain why the project uses a database instead of passing data between Python functions
- [ ] You drew the pipeline flow on paper (scrape → enrich → summarize → rank → email)

---

## Session 2: Architecture and Mental Map

**Estimated duration:** 1.5–2 hours (reading + light exploration)

**Chapters to read:**
- Part 2: Architecture Explained (lines 197–315)

**Concepts to learn:**
- How the project folders map to pipeline stages (`scrapers/` → step 1, `services/` → steps 2–5, etc.)
- How Python files import each other (the dependency chain)
- What `main.py` → `daily_runner.py` → `runner.py` → scrapers/services means
- The difference between a "scraper" (collects data), a "service" (orchestrates work), and an "agent" (uses AI)

**Coding tasks:**
- Open each file mentioned in the architecture diagram in VS Code
- Read just the first 5–10 lines of each file (the imports) — notice how they import from each other

**Commands to run:**
```bash
# See the full file tree
find . -name "*.py" | head -30

# Look at how main.py starts the chain
head -10 main.py
head -15 app/daily_runner.py
head -10 app/runner.py
```

**Success criteria:**
- [ ] You can redraw the architecture diagram from memory (boxes and arrows)
- [ ] You know which folder handles each pipeline step
- [ ] You understand that `main.py` calls `daily_runner.py`, which calls everything else
- [ ] You can explain what each folder is responsible for in one sentence

---

## Session 3: Core Concepts — Part A (Python Tooling)

**Estimated duration:** 2–2.5 hours (reading + hands-on)

**Chapters to read:**
- Part 3, Concepts 1–2: Virtual Environments, uv (lines 322–363)
- Part 3, Concept 9: What Is a .env File? (lines 528–563)

**Concepts to learn:**
- What a virtual environment is and why each project needs its own
- What `uv` does and how it differs from `pip`
- What `pyproject.toml` and `uv.lock` are (grocery list vs. receipt analogy)
- What `.env` files are and why secrets must never be in code

**Coding tasks:**
- Open `pyproject.toml` and read every dependency — look up any you don't recognize
- Open `app/example.env` and identify each variable's purpose

**Commands to run:**
```bash
# Check if uv is installed
uv --version

# Look at what the project depends on
cat pyproject.toml

# Look at the example environment file
cat app/example.env

# See the .gitignore to confirm .env is excluded
cat .gitignore
```

**Success criteria:**
- [ ] You can explain the virtual environment "apartment" analogy
- [ ] You know that `uv sync` reads `pyproject.toml` and installs packages into `.venv`
- [ ] You understand why `.env` is in `.gitignore`
- [ ] You can explain the difference between `pyproject.toml` (what you want) and `uv.lock` (what you got)

---

## Session 4: Core Concepts — Part B (Infrastructure)

**Estimated duration:** 2–2.5 hours (reading + hands-on)

**Chapters to read:**
- Part 3, Concepts 3–5: Database, PostgreSQL, Docker (lines 365–474)
- Part 3, Concept 8: What Is an API Key? (lines 506–527)

**Concepts to learn:**
- What a relational database is (tables, columns, rows — the Excel analogy)
- What PostgreSQL is and why this project chose it
- What Docker is and why PostgreSQL runs inside a container
- The "portable kitchen" analogy for Docker
- What an API key is and why OpenAI requires one

**Coding tasks:**
- Open `docker/docker-compose.yml` and read it line-by-line with the tutorial's explanation
- Identify which values in `docker-compose.yml` match which `.env` variables

**Commands to run:**
```bash
# Read the Docker Compose file
cat docker/docker-compose.yml

# Check if Docker is installed
docker --version

# Check if Docker Desktop is running (this will fail if Docker is not running — that's OK)
docker ps
```

**Success criteria:**
- [ ] You can explain what a database table, column, and row are
- [ ] You understand that Docker runs PostgreSQL in an isolated container
- [ ] You know that `docker compose up -d` starts the database and `-d` means "in the background"
- [ ] You can explain what an API key is and where to get the OpenAI one

---

## Session 5: Core Concepts — Part C (ORM and Data)

**Estimated duration:** 1.5–2 hours (reading + code exploration)

**Chapters to read:**
- Part 3, Concepts 6–7: SQLAlchemy, RSS Feeds (lines 441–527)
- Part 3, Concepts 10–11: AI Agent, Pydantic (lines 564–620)

**Concepts to learn:**
- What an ORM (Object-Relational Mapper) is — writing Python classes instead of SQL
- What `engine`, `session`, and `Base` mean in SQLAlchemy
- What an RSS feed is and why it makes scraping easier than parsing HTML
- What Pydantic does (validates data structure and types)
- What "AI Agent" means in this project (not a chatbot — a function that calls GPT with a specific prompt)

**Coding tasks:**
- Open `app/database/models.py` and match each Python class to the table it creates
- Open one agent file (e.g., `app/agent/digest_agent.py`) and find where GPT is called

**Commands to run:**
```bash
# Look at the database models
head -60 app/database/models.py

# Look at how the digest agent calls GPT
cat app/agent/digest_agent.py
```

**Success criteria:**
- [ ] You can explain why the project uses SQLAlchemy instead of raw SQL
- [ ] You know the difference between an SQLAlchemy `engine`, `session`, and model class
- [ ] You can explain what RSS is in one sentence
- [ ] You understand that "agent" in this project = Python function + GPT prompt, not a chatbot

---

## Session 6: Full Setup — Get It Running

**Estimated duration:** 2.5–3 hours (hands-on setup)

**Chapters to read:**
- Part 4: Beginner Setup Guide — all 11 steps (lines 622–1034)
- Part 8: Environment Variables Deep Dive (lines 2411–2506) — as reference while creating `.env`

**Concepts to learn:**
- The complete local development setup workflow
- How `uv sync` installs dependencies
- How Docker Compose starts PostgreSQL
- How `create_tables.py` creates the database schema
- How `main.py` runs the full pipeline

**Coding tasks:**
- Create your `.env` file from `example.env`
- Fill in your actual API keys and email credentials

**Commands to run:**
```bash
# Step 1: Verify Python
python3 --version

# Step 2: Verify uv
uv --version

# Step 5: Navigate to project
cd /Users/svetlanaonopa/Desktop/Projects/AI_Project/Git_repo_clone/ai-news-aggregator

# Step 7: Create .env from example
cp app/example.env .env
# Then open .env in VS Code and fill in your values

# Step 8: Install dependencies
uv sync

# Step 9: Start PostgreSQL
cd docker && docker compose up -d && cd ..

# Step 10: Create tables
uv run python -m app.database.create_tables

# Step 11: Run the pipeline
uv run python main.py
```

**Success criteria:**
- [ ] `uv sync` completed without errors
- [ ] `docker compose up -d` started the PostgreSQL container
- [ ] `create_tables` ran without errors
- [ ] `python main.py` ran the pipeline (even if some scrapers found 0 articles)
- [ ] You received an email in your inbox (or saw "email sent" in the logs)

**Important note:** If any step fails, use Part 13 (Debugging Guide) to troubleshoot before moving on. Do not skip this session — everything after this depends on having a working setup.

---

## Session 7: Read the Code — Data Layer

**Estimated duration:** 2.5–3 hours (reading + exploring)

**Chapters to read:**
- Part 5: File-by-File Explanation — root files and `app/database/` (lines 1035–1427)
- Part 10: Database Deep Dive (lines 2637–2740)

**Concepts to learn:**
- What `main.py`, `pyproject.toml`, `uv.lock`, `.python-version`, and `.gitignore` each do
- How `models.py` defines database tables as Python classes
- How `connection.py` creates the SQLAlchemy engine and session
- How `repository.py` wraps database operations (save, query, check duplicates)
- How `create_tables.py` turns models into actual SQL tables
- What primary keys are and how each table prevents duplicates
- The column-by-column breakdown of `youtube_videos` and `digests` tables

**Coding tasks:**
- Read `app/database/models.py` fully and annotate (on paper or in a notebook) what each column means
- Read `app/database/repository.py` and trace one save operation
- Query the database to see real data from your pipeline run

**Commands to run:**
```bash
# Connect to PostgreSQL and run queries
docker exec -it ai-news-aggregator-db psql -U postgres -d ai_news_aggregator

# Inside psql, run:
# \dt                                          (list all tables)
# SELECT COUNT(*) FROM youtube_videos;
# SELECT COUNT(*) FROM digests;
# SELECT title, article_type FROM digests LIMIT 5;
# \q                                           (exit psql)
```

**Success criteria:**
- [ ] You can explain what every column in `youtube_videos` means
- [ ] You understand why `transcript` is nullable (it starts as NULL, gets filled later)
- [ ] You successfully ran SQL queries against your local database
- [ ] You can trace how `repository.py` saves a scraped article to the database
- [ ] You can explain the difference between `models.py` (defines shape) and `repository.py` (does operations)

---

## Session 8: Read the Code — Scrapers and Services

**Estimated duration:** 2.5–3 hours (reading + exploring)

**Chapters to read:**
- Part 5: File-by-File — `app/scrapers/` folder (lines 1217–1298)
- Part 5: File-by-File — `app/services/` folder (lines 1424–1493)
- Part 6: Function-by-Function — scraper and service functions (lines 1614–1742)

**Concepts to learn:**
- How `youtube.py` uses the RSS feed to find new videos, then fetches transcripts
- How `openai.py` and `anthropic.py` scrape blog posts via RSS
- What `feedparser` does (parses RSS XML into Python dictionaries)
- How `process_anthropic.py` downloads full article text using `docling`
- How `process_youtube.py` fetches transcripts using `youtube-transcript-api`
- How `process_digest.py` sends content to GPT for summarization
- How `process_email.py` orchestrates ranking, email writing, and sending
- How `email.py` sends email via Gmail SMTP

**Coding tasks:**
- Read `app/scrapers/youtube.py` fully and identify: where RSS is fetched, where data is filtered by date, where results are returned
- Read `app/services/process_digest.py` and trace how articles become summaries
- Read `app/services/email.py` and identify the SMTP connection setup

**Commands to run:**
```bash
# Explore a raw RSS feed to see what the scraper receives
uv run python -c "
import feedparser
feed = feedparser.parse('https://openai.com/news/rss.xml')
print(f'Feed title: {feed.feed.get(\"title\", \"N/A\")}')
print(f'Number of entries: {len(feed.entries)}')
if feed.entries:
    entry = feed.entries[0]
    print(f'First entry title: {entry.get(\"title\", \"N/A\")}')
    print(f'First entry link: {entry.get(\"link\", \"N/A\")}')
"
```

**Success criteria:**
- [ ] You can explain what `feedparser.parse()` returns
- [ ] You can trace a YouTube video from RSS entry → Python object → database row
- [ ] You understand why transcripts and markdown are fetched separately from the initial scrape
- [ ] You can explain how `process_email.py` connects the curator agent, email agent, and SMTP sender
- [ ] You can describe the SMTP email sending process in plain English

---

## Session 9: Read the Code — AI Agents

**Estimated duration:** 2–2.5 hours (reading + experimenting)

**Chapters to read:**
- Part 5: File-by-File — `app/agent/` folder (lines 1493–1572)
- Part 5: File-by-File — `app/profiles/user_profile.py` (lines 1544–1572)
- Part 6: Function-by-Function — agent functions (lines 1680–1742)
- Part 5: File-by-File — `app/config.py` (lines 1148–1168)

**Concepts to learn:**
- How `digest_agent.py` sends article content to GPT-4o-mini and gets back a structured summary
- How `curator_agent.py` sends all summaries to GPT-4.1 and gets back a ranked list
- How `email_agent.py` writes the personalized email greeting
- What a "system prompt" is and how it controls GPT's behavior
- How `user_profile.py` describes your interests so the AI can personalize rankings
- How `config.py` stores YouTube channel IDs and other settings

**Coding tasks:**
- Read `app/agent/digest_agent.py` and find: the system prompt, the user prompt, the model name, and how the response is parsed
- Read `app/agent/curator_agent.py` and understand how it returns a ranked list
- Read `app/profiles/user_profile.py` and think about how you would customize it for yourself

**Commands to run:**
```bash
# Read the user profile that controls AI personalization
cat app/profiles/user_profile.py

# Read the config file with YouTube channel IDs
cat app/config.py

# Read the digest agent to see the GPT prompt
cat app/agent/digest_agent.py
```

**Success criteria:**
- [ ] You can explain what a "system prompt" is and find it in each agent file
- [ ] You understand why `curator_agent.py` uses a more powerful model (GPT-4.1) than `digest_agent.py` (GPT-4o-mini)
- [ ] You can explain how `user_profile.py` influences the email content
- [ ] You can describe the full chain: article → digest agent → curator agent → email agent → email

---

## Session 10: Docker and Database Deep Dive

**Estimated duration:** 1.5–2 hours (reading + hands-on)

**Chapters to read:**
- Part 9: Docker Explained for This Project (lines 2507–2636)
- Part 10: Database Deep Dive — remaining sections (lines 2697–2740)

**Concepts to learn:**
- The "portable kitchen" analogy for Docker
- What a Docker image vs. container is
- What Docker Compose does and why it's simpler than raw `docker run`
- How port mapping works (host 5432 → container 5432)
- How volumes persist data even when the container is deleted
- How data flows through all 4 database tables across the pipeline

**Coding tasks:**
- Trace a single article through all pipeline stages on paper: RSS entry → scraper → database row → enrichment → digest row → email

**Commands to run:**
```bash
# See running containers
docker ps

# See the volume that stores database data
docker volume ls

# Check the database logs
cd docker && docker compose logs && cd ..

# Connect to the database and trace one article
docker exec -it ai-news-aggregator-db psql -U postgres -d ai_news_aggregator -c "
SELECT d.title, d.article_type, d.summary
FROM digests d
LIMIT 3;
"
```

**Success criteria:**
- [ ] You can explain the difference between a Docker image and a container
- [ ] You know what happens to your data if you run `docker compose down` vs. `docker compose down -v`
- [ ] You can trace an Anthropic article from RSS → `anthropic_articles` (markdown=NULL) → enrichment (markdown filled) → `digests` (summary created)
- [ ] You understand how port mapping lets Python connect to PostgreSQL inside Docker

---

## Session 11: Build It Yourself — Lessons 1–6

**Estimated duration:** 3–4 hours (coding)

**Chapters to read:**
- Part 7: Build Roadmap, Lessons 1–6 (lines 1780–2214)

**Concepts to learn:**
- How to create a Python project from scratch with `uv`
- How to define SQLAlchemy models from scratch
- How to create a database connection and repository pattern
- How to write a scraper that parses RSS feeds

**Coding tasks:**
- Create a new empty folder (e.g., `ai-news-from-scratch/`)
- Follow Lessons 1–6 exactly, typing every line yourself (do not copy from the original project)
- After each lesson, run the test command to verify it works

**Commands to run:**
```bash
# Create a fresh project folder outside the cloned repo
mkdir ~/Desktop/Projects/AI_Project/ai-news-from-scratch
cd ~/Desktop/Projects/AI_Project/ai-news-from-scratch

# Then follow the commands in each lesson:
# Lesson 1: uv init, create folder structure, create main.py
# Lesson 2: Create app/database/models.py
# Lesson 3: Create connection.py and repository.py
# Lesson 4: Create app/scrapers/youtube.py
# Lesson 5: Create openai.py and anthropic.py scrapers
# Lesson 6: Create app/runner.py and wire scrapers to database
```

**Success criteria:**
- [ ] Your from-scratch project has the correct folder structure
- [ ] `uv sync` installs all dependencies
- [ ] Database models can be imported without errors
- [ ] The YouTube scraper can fetch RSS data (even without saving to DB yet)
- [ ] `runner.py` can scrape all three sources and save to the database

---

## Session 12: Build It Yourself — Lessons 7–12

**Estimated duration:** 3–4 hours (coding)

**Chapters to read:**
- Part 7: Build Roadmap, Lessons 7–12 (lines 2214–2410)

**Concepts to learn:**
- How to build service layers that orchestrate work
- How to integrate OpenAI's API into your code
- How to structure an AI agent with a system prompt
- How to send email with Python's `smtplib`
- How to wire everything into a single pipeline runner
- How to set up Docker Compose for local PostgreSQL

**Coding tasks:**
- Continue building in your `ai-news-from-scratch/` folder
- Implement Lessons 7–12 by typing each file yourself
- Run the full pipeline end-to-end when done

**Commands to run:**
```bash
# Continue in your from-scratch folder
cd ~/Desktop/Projects/AI_Project/ai-news-from-scratch

# Lesson 7: Create process_anthropic.py, process_youtube.py
# Lesson 8: Create app/agent/digest_agent.py
# Lesson 9: Create app/agent/curator_agent.py
# Lesson 10: Create email_agent.py and email.py
# Lesson 11: Create daily_runner.py
# Lesson 12: Create docker/docker-compose.yml

# Final test: run the full pipeline
cd docker && docker compose up -d && cd ..
uv run python -m app.database.create_tables
uv run python main.py
```

**Success criteria:**
- [ ] Your from-scratch project runs the full pipeline without errors
- [ ] AI summaries are generated and saved to the database
- [ ] You receive a personalized email digest
- [ ] You can explain every file you created and why it exists
- [ ] You built the entire project by understanding, not copying

---

## Session 13: Branch Comparison and Deployment

**Estimated duration:** 2.5–3 hours (reading + hands-on)

**Chapters to read:**
- Part 11: Branch Comparison (lines 2741–3017)
- Part 12: Deployment to Render.com (lines 3018–3332)

**Concepts to learn:**
- Why the project has 3 branches and what each adds
- What `DATABASE_URL` is and why deployment needs it
- Why `postgres://` must become `postgresql://` for SQLAlchemy
- What a Dockerfile does and how it differs from Docker Compose
- What Render.com is and how it runs your code on a schedule
- What `render.yaml` (Blueprint) does
- How cron schedules work (`"0 5 * * *"`)

**Coding tasks:**
- Switch to the `deployment` and `deployment-final` branches and compare key files
- Read the Dockerfile and `render.yaml`
- (Optional) Deploy to Render.com following the step-by-step guide

**Commands to run:**
```bash
# Go back to the cloned project
cd /Users/svetlanaonopa/Desktop/Projects/AI_Project/Git_repo_clone/ai-news-aggregator

# Compare branches
git diff master..deployment -- app/database/connection.py
git diff deployment..deployment-final -- app/daily_runner.py

# Look at deployment files
git show deployment-final:Dockerfile
git show deployment-final:render.yaml
```

**Success criteria:**
- [ ] You can explain why `deployment` branch changes `connection.py`
- [ ] You know what the `postgres://` → `postgresql://` fix does and why
- [ ] You can read a Dockerfile and explain each line
- [ ] You understand cron syntax well enough to change the schedule
- [ ] (Optional) You deployed to Render.com and received an email from the cloud

---

## Session 14: Debugging, Checkpoints, and Final Review

**Estimated duration:** 2–2.5 hours (reading + self-testing)

**Chapters to read:**
- Part 13: Debugging Guide (lines 3334–3605)
- Part 14: Learning Checkpoints — all sections (lines 3606–3735)
- Part 15: Final Reference (lines 3737–4094)

**Concepts to learn:**
- How to read Python error messages (traceback → bottom line → fix)
- The 12 most common errors in this project and how to fix each one
- How to verify your understanding with the checkpoint checklists

**Coding tasks:**
- Go through every checkbox in Part 14 and honestly check what you know
- For any checkbox you cannot check off, go back to that part and re-read it
- Bookmark the Command Cheat Sheet (Part 15) for daily use

**Commands to run:**
```bash
# Intentionally trigger some common errors to practice debugging:

# Error 5: ModuleNotFoundError (run without uv)
python3 main.py  # Will fail because packages are in .venv, not global Python

# Error 9: .env not loaded (rename .env temporarily)
mv .env .env.backup
uv run python -c "import os; from dotenv import load_dotenv; load_dotenv(); print(os.getenv('OPENAI_API_KEY'))"
# Should print None
mv .env.backup .env

# Run the full pipeline one more time to confirm everything works
uv run python main.py
```

**Success criteria:**
- [ ] You can read a Python traceback and find the actual error
- [ ] You checked off at least 80% of the checkpoints in Part 14
- [ ] You know where to find the Command Cheat Sheet when you need it
- [ ] You ran the full pipeline successfully one final time
- [ ] You feel confident explaining this entire project to someone else

---

## Summary Table

| Session | Topic | Duration | Type |
|---------|-------|----------|------|
| 1 | Big Picture — what and why | 1.5–2h | Reading |
| 2 | Architecture — how it fits together | 1.5–2h | Reading + exploring |
| 3 | Concepts A — Python tooling (venv, uv, .env) | 2–2.5h | Reading + hands-on |
| 4 | Concepts B — Infrastructure (DB, Docker, APIs) | 2–2.5h | Reading + hands-on |
| 5 | Concepts C — ORM, RSS, Pydantic, AI agents | 1.5–2h | Reading + exploring |
| 6 | Full Setup — get the project running | 2.5–3h | Hands-on setup |
| 7 | Code Reading — data layer | 2.5–3h | Reading + SQL |
| 8 | Code Reading — scrapers and services | 2.5–3h | Reading + experimenting |
| 9 | Code Reading — AI agents | 2–2.5h | Reading + experimenting |
| 10 | Docker and Database deep dive | 1.5–2h | Reading + hands-on |
| 11 | Build from scratch — Lessons 1–6 | 3–4h | Coding |
| 12 | Build from scratch — Lessons 7–12 | 3–4h | Coding |
| 13 | Branch comparison and deployment | 2.5–3h | Reading + hands-on |
| 14 | Debugging, checkpoints, final review | 2–2.5h | Self-testing |

---

## Tips for Getting the Most Out of This Plan

1. **Do not rush.** If a session takes longer than estimated, that is fine. Understanding beats speed.
2. **Take notes on paper.** Writing by hand activates different memory pathways than typing.
3. **Explain out loud.** After each session, explain what you learned to an imaginary friend (or a real one). If you get stuck, that is what you need to review.
4. **Do not skip the "build from scratch" sessions (11–12).** Reading code and writing code use different parts of your brain. You will be surprised how much you missed until you type it yourself.
5. **Use the checkpoints.** Part 14 of the tutorial has honest self-tests. Unchecked boxes are not failures — they are your study guide for what to review.
6. **Break sessions further if needed.** If a 3-hour session feels too long, split it across two sittings. The section headers make natural stopping points.
7. **Ask Claude Code for help.** If you get stuck on any concept or error, describe what you see and what you expected. That is exactly what I am here for.
