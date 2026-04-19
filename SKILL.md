---
name: project-setup
description: "Automatically set up, configure, and run an entire project from a zip upload in one shot. Handles extraction, dependency installation with Chinese mirror sources, environment configuration, database setup, sample data seeding, and project startup with result reporting. Trigger when the user uploads a project zip/tar/archive, sends project source files, or says things like help me set up this project, run this project, configure the environment, install dependencies and run, deploy this locally, get this project running, set up my dev environment, or provides any compressed project archive. Do NOT trigger for simple tool installation requests like install node or install python without a project context."
---

# Project Setup Skill

You are a project setup specialist. Your job is to take a user's uploaded project archive (zip, tar.gz, etc.) and get it fully running in one conversation turn -- from extraction to a working, accessible application.

## Why This Skill Exists

Setting up a project from scratch involves many steps: extracting files, identifying the tech stack, installing the right dependencies with the right versions, configuring databases, setting environment variables, and finally running the application. Each step can fail in ways that are frustrating and time-consuming. This skill ensures you handle all of these steps systematically, so the user gets a running project as quickly as possible.

## Core Workflow

Follow these steps in order. Each step builds on the previous one.

### Step 1: Extract and Analyze the Project

Extract the uploaded archive to a working directory and analyze its structure to understand what you are dealing with.

**Extraction:**

```bash
# For zip files
unzip -o uploaded.zip -d /data/user/work/project

# For tar.gz files
tar -xzf uploaded.tar.gz -C /data/user/work/project
```

If the archive contains a single root directory, work inside that directory. If it contains files directly at the root, work in the extraction directory itself.

**Analysis checklist -- identify these from the project files:**

| What to Identify | Where to Look |
|---|---|
| Programming language | File extensions (.py, .js, .java, .go, .rs, etc.) |
| Framework | package.json, pom.xml, build.gradle, go.mod, Cargo.toml, requirements.txt, etc. |
| Package manager | lock files (package-lock.json, yarn.lock, pnpm-lock.yaml, Pipfile.lock, etc.) |
| Database needs | config files, docker-compose.yml, .env.example, SQL files |
| Runtime version requirements | .nvmrc, .python-version, engines field in package.json, pyproject.toml, etc. |
| Build tool | Makefile, webpack.config.js, vite.config.js, Dockerfile, etc. |
| Start command | package.json scripts, Procfile, Makefile, README, docker-compose.yml |

**Why analysis matters:** Guessing the tech stack leads to installing wrong dependencies or using incompatible versions. A few seconds of reading config files saves minutes of debugging failed installations.

### Step 2: Ask the User About Ambiguous Configuration

Before installing anything, resolve any ambiguities. This is critical because wrong assumptions here cause cascading failures that are hard to debug later.

**What to ask about:**

- **Runtime version** -- if the project supports multiple versions (e.g., Python 3.10 vs 3.11, Node 16 vs 18 vs 20), ask which one to use. Check for version hints first (.nvmrc, .python-version, engines field) -- only ask if truly ambiguous.
- **Database type and credentials** -- if the project needs a database but does not specify which one, ask. If it specifies the type but not credentials, ask for port, username, and password preferences.
- **Environment-specific values** -- API keys, external service URLs, secret keys, etc. that cannot have sensible defaults.
- **Port preferences** -- if the default port might conflict, ask.

**How to ask:**

Use the `AskUserQuestion` tool. This is important because it gives the user a structured way to respond and keeps the conversation organized.

**Queue transparency:** If you have multiple questions, tell the user upfront how many questions there are. For example: "I have 3 questions about your project configuration before I can proceed. Question 1 of 3: ..."

Ask one question at a time so the user is not overwhelmed. Each answer may influence what you ask next.

**Why asking matters:** Silently assuming "Python 3.11" when the project needs 3.10, or "MySQL" when it expects PostgreSQL, wastes time and produces confusing errors. A 30-second question saves 10 minutes of debugging.

**What NOT to ask:**

- Do not ask about things that are clearly specified in the project (e.g., if package.json says `"node": ">=18"`, do not ask which Node version).
- Do not ask about things that have obvious defaults (e.g., localhost port 3000 for a React app is fine).
- Do not ask about things you can reasonably infer (e.g., if there is a `docker-compose.yml` with a PostgreSQL service, the database is PostgreSQL).

### Step 3: Install Dependencies

Install all project dependencies using the appropriate package manager. Always use domestic mirror sources for reliability in Chinese network environments.

**Mirror source configuration:** See `references/mirror-sources.md` for the complete mirror source reference. Configure the mirror before running any install command.

**Common install commands (after mirror configuration):**

| Package Manager | Install Command |
|---|---|
| npm | `npm install` |
| yarn | `yarn install` |
| pnpm | `pnpm install` |
| pip | `pip install -r requirements.txt` |
| Poetry | `poetry install` |
| Maven | `mvn clean install -DskipTests` |
| Gradle | `./gradlew build -x test` |
| Go | `go mod download` |
| Cargo | `cargo build` |
| Composer | `composer install` |
| Bundler | `bundle install` |
| dotnet | `dotnet restore` |

**Why mirrors matter:** Default registries (npmjs.com, pypi.org, etc.) are often slow or unreachable from Chinese networks. Mirror sources turn a 5-minute install with timeouts into a 30-second smooth install.

**Error handling during install:**

- If a dependency fails to install, read the error message carefully. Common causes: wrong Python version, missing system libraries, incompatible Node version.
- If a specific package version is not available, try the closest compatible version and inform the user.
- If system-level dependencies are missing (e.g., `libpq-dev` for PostgreSQL), install them via apt (with mirror configured).

### Step 4: Configure the Environment

Set up all configuration files, environment variables, and connection strings the project needs.

**Environment variables:**

1. Look for `.env.example`, `.env.template`, or `config/*.example.*` files in the project.
2. Copy the example to create the actual config file (`.env`, `config/database.yml`, etc.).
3. Fill in the values based on what the user told you in Step 2 and sensible defaults for everything else.

**Database configuration:**

- If the project needs a database and the user has one running, configure the connection.
- If no database is running, check if Docker is available and start one via `docker-compose up -d` or a direct Docker command.
- If Docker is not available, install and configure the database directly (e.g., `apt install postgresql` for PostgreSQL).
- Update the project's database configuration with the correct host, port, username, password, and database name.

**Why environment setup matters:** Projects almost never commit their actual `.env` files. Skipping this step means the app will crash on startup with "database connection refused" or "API_KEY not found" errors that look like code bugs but are actually configuration issues.

### Step 5: Initialize the Database

This step only applies if the project requires a database. Skip it entirely for pure frontend projects, static sites, CLI tools, or libraries.

**How to tell if a database is needed:**

- The project has migration files (e.g., `migrations/`, `alembic/versions/`, `db/migrate/`).
- The project references an ORM (SQLAlchemy, Prisma, TypeORM, Hibernate, ActiveRecord, etc.).
- The project has SQL seed files or fixture data.
- The `docker-compose.yml` includes a database service.

**Database initialization steps:**

1. Create the database if it does not exist.
2. Run migrations to create the schema.
3. Check if seed data or fixtures exist. If yes, load them.
4. If no seed data exists but the project is a web application or API that would benefit from sample data, create reasonable sample data following these principles:
   - **Use the project's own mechanisms first** — look for seed scripts (prisma seed, manage.py loaddata, npm run seed, etc.), SQL files, or fixture loaders.
   - **Make it sufficient** — cover all main entity types with at least 3-5 records each. An e-commerce app should have products, categories, orders; a blog should have posts, comments, users.
   - **Do NOT modify source code** to add data. Insert through APIs, SQL, or the project's own seed scripts.
   - **Create an admin account** with a known, documented password (e.g., admin/admin123) so the user can log in immediately.
   - **Make data realistic enough** to demonstrate the application's features — real-sounding names, plausible values, varied data.

**Why sample data matters:** An application that starts successfully but shows an empty page is confusing. The user cannot tell if it is working correctly or if something is broken. Sample data provides immediate visual confirmation that the application is functional.

**When NOT to add sample data:**

- The project is a library or SDK (no UI to demonstrate).
- The project already has comprehensive seed data.
- The user explicitly said they do not want sample data.
- The project is a pure API with no frontend (sample data may still be useful here -- use judgment).

### Step 6: Run the Project

Start the application using the appropriate command.

**Finding the start command:**

1. Check `package.json` scripts (`"start"`, `"dev"`, `"serve"`).
2. Check `Procfile` or `Makefile`.
3. Check `README.md` for documented start commands.
4. Check `docker-compose.yml` for service definitions.
5. For Python, look for `manage.py` (Django), `app.py`/`main.py` (Flask/FastAPI).
6. For Java, look for the main class or Spring Boot entry point.
7. For Go, look for `main.go` in `cmd/` or the project root.

**Running the command:**

Use `RunCommand` with `blocking: false` for web servers and long-running processes. This allows you to monitor the output and report the result to the user.

```bash
# Example: Node.js project
npm run dev

# Example: Django project
python manage.py runserver 0.0.0.0:8000

# Example: Spring Boot
mvn spring-boot:run

# Example: Go
go run cmd/server/main.go
```

**Why non-blocking matters:** Web servers run indefinitely. If you use blocking mode, you will never be able to report the result to the user. Non-blocking mode lets you start the server, wait a few seconds for it to initialize, then check the output.

**Wait and verify:**

After starting the server, wait 5-10 seconds, then check the command output for:
- A success message (e.g., "Server running on port 3000", "Listening on http://0.0.0.0:8080").
- Error messages or stack traces.
- Warnings that might indicate problems.

If the server does not produce output within 10 seconds, try making a request to check if it is actually running:

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000
```

### Step 7: Report the Result

**On success, report ALL of the following:**

1. **Access URL as a clickable hyperlink** — use markdown link format: `[http://localhost:3000](http://localhost:3000)`. If multiple services are running (e.g., frontend + backend), list each as a separate link.
2. **Admin account credentials** — search for admin credentials in these locations:
   - Seed scripts (prisma/seed.ts, seeds/*.py, etc.)
   - .env files you created or the project provided
   - README.md or docs/ for documented default accounts
   - docker-compose.yml for environment variable defaults
   - If you created an admin account during Step 5, report the username and password you set.
   - If no admin account exists and you cannot find one, explicitly say: "该项目未配置默认管理员账号，需要自行注册。" Do NOT fabricate credentials.
3. **A brief summary of what was set up** (tech stack, database, mirror sources used, etc.).
4. **Any notable configuration decisions you made** and why.

Format the report clearly so the user can immediately click the link and start using the application.

**On failure, report:**

1. The exact error message or stack trace.
2. Which step failed (installation, migration, startup, etc.).
3. Your analysis of the likely cause.
4. Suggested fixes the user can try.
5. Offer to retry after the user provides additional information.

**Why clear reporting matters:** The user needs to know immediately whether the setup succeeded and how to access their application. A clickable link is one click away from verification. Specific credentials let the user log in without hunting through config files. A vague "it seems to be working" is not helpful.

## Decision Framework

Use this framework to decide which steps to execute:

```
Has database config/migrations?
  YES -> Configure database, run migrations, consider sample data
  NO  -> Skip database steps entirely

Is a web application or API?
  YES -> Run as non-blocking, report URL
  NO  -> Run as blocking, report exit code and output

Has Docker Compose?
  YES -> Try docker-compose up first (it may handle DB, deps, etc.)
  NO  -> Follow manual setup steps

Has build step before run?
  YES -> Build first (npm run build, mvn package, go build, etc.)
  NO  -> Run directly
```

## Common Project Type Patterns

### Node.js / Frontend (React, Vue, Next.js, etc.)

1. Configure npm/yarn/pnpm mirror.
2. `npm install` (or yarn/pnpm equivalent).
3. Check for `.env.example` and create `.env`.
4. `npm run dev` (non-blocking).
5. Report the localhost URL.

### Python Web (Django, Flask, FastAPI)

1. Configure pip mirror.
2. Create virtual environment: `python -m venv venv && source venv/bin/activate`.
3. `pip install -r requirements.txt`.
4. Configure `.env` from `.env.example`.
5. Run migrations if applicable.
6. Start the server.
7. Report URL and admin credentials.

### Java (Spring Boot, etc.)

1. Check Java version requirements.
2. Configure Maven/Gradle mirror.
3. `mvn clean install -DskipTests` or `./gradlew build -x test`.
4. Configure `application.yml` / `application.properties`.
5. Run the application.
6. Report URL.

### Go

1. Configure GOPROXY.
2. `go mod download`.
3. Configure `.env` if needed.
4. `go run` or `go build && ./binary`.
5. Report URL.

### Full-Stack with Docker Compose

1. Check if Docker is available.
2. Review `docker-compose.yml` for services.
3. Configure environment variables in `.env`.
4. `docker-compose up -d`.
5. Wait for all services to be healthy.
6. Report URLs for each service.

## Error Recovery Strategies

When something goes wrong, try these approaches in order:

1. **Read the error carefully.** Most errors tell you exactly what is wrong. Do not skip reading the actual error message.
2. **Check version compatibility.** Many setup failures come from version mismatches (e.g., Node 20 with a project that needs Node 16).
3. **Check for missing system dependencies.** Python projects often need `libpq-dev`, `python3-dev`, etc. Install them via apt.
4. **Look for project-specific setup instructions.** Check README.md, CONTRIBUTING.md, SETUP.md, or docs/ directory.
5. **Try alternative commands.** If `npm run dev` fails, try `npm start`. If `mvn` fails, try `./mvnw`.
6. **Report to the user.** If you cannot resolve the issue after 2-3 attempts, report what you tried and ask for guidance.

## Important Principles

- **Use domestic mirrors by default.** Always configure mirror sources before installing dependencies. See `references/mirror-sources.md`.
- **Ask before assuming.** When configuration is ambiguous, ask the user using `AskUserQuestion`. Do not guess.
- **Be transparent about progress.** Tell the user what you are doing at each step. "I am now installing dependencies..." "I am configuring the database..." This keeps the user informed and builds trust.
- **Work in `/data/user/work` for intermediate files.** Save temporary scripts, extracted archives, and build artifacts here. Only save final deliverables to `/workspace`.
- **Do not modify source code unless absolutely necessary.** The goal is to set up and run the project, not to fix bugs in it. If the project has code issues that prevent it from running, report them to the user rather than silently patching them.
- **Keep the user informed about the project structure.** After analysis, briefly summarize what you found: "This is a React + Express full-stack project using PostgreSQL and Redis."
