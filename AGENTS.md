# Repository Guidelines

> **This file is the single source of truth for all AI agents working on this project.**
> It is accessible to Claude Code via a symlink: `CLAUDE.md → AGENTS.md`.
> **All procedures defined here are mandatory. Non-compliance blocks commits and task completion.**

**Primary references**: see `docs/PRD-Sinsay-PoC.md` and `docs/ADR-Sinsay-PoC.md` before making changes.

**Scoped guidelines** (always read the relevant file before working in that area):
- Backend (Java / Spring Boot): `src/AGENTS.md`
- Frontend (React / TypeScript): `Frontend/AGENTS.md`

---

## Project Overview

This repo hosts a Proof of Concept for Sinsay returns/complaints verification using multimodal AI. The target flow is a form-to-chat experience where users submit order context and images, then receive a streamed verdict in Polish. The backend is Spring Boot + Spring AI; the frontend is React 19 + assistant-ui. Streaming must follow the Vercel AI SDK Data Stream Protocol.

## Current vs. Planned Structure

- **Current (in repo)**: single Spring Boot app under `src/` with Maven wrapper and minimal tests.
- **Planned (ADR)**: a monorepo with separate `backend/` and `frontend/` directories and a build that bundles the frontend into the backend `static/` folder. If you introduce the monorepo layout, keep the existing root `pom.xml` aligned or migrate intentionally.

## Project Structure & Module Organization

- `src/main/java/com/silkycoders1/jsystemssilkycodders1/`: Spring Boot entry point.
- `src/main/resources/`: `application.properties`, plus `static/` and `templates/` (frontend build output targets `static/`).
- `src/test/java/com/silkycoders1/jsystemssilkycodders1/`: JUnit tests.
- `docs/`: PRD, ADR, and research notes used as requirements.
- `docs/sinay/`: terms of returns and complaints from Sinsay as input for AI Agent system prompt / knowledge base.
- `Frontend/`: React + Tailwind frontend (see `Frontend/AGENTS.md` for frontend-specific rules).

### Target Modules (from ADR)

If/when split into modules, use the following layout and names:
- `backend/src/main/java/com/sinsay/`: `config/`, `controller/`, `service/`, `model/`.
- `frontend/src/`: `app/` (screens), `components/ui/` (Shadcn), form + chat components.

---

## Claude Code Tools and Resources

Claude Code has access to three categories of tools. **Agents must actively use these tools** to maximize code quality and development efficiency. Do not ignore available tools — their use is part of the standard workflow.

### Plugins (Enabled)

#### `code-review` — Automated PR Code Review

- **Command**: `/code-review`
- **What it does**: Launches 4 parallel agents to independently review a pull request — 2 agents check AGENTS.md/CLAUDE.md compliance, 1 scans for bugs in changed code, 1 analyzes git history for context. Issues are confidence-scored 0–100; only issues ≥80 are reported to minimize false positives.
- **When to use**: Before merging any non-trivial PR; after implementing a feature or fixing a bug; when touching critical paths (SSE adapter, AI model calls, persistence).
- **Integration into workflow**: Run `/code-review` as the final step before marking a PR ready for merge.

#### `playwright` — Browser Automation and Visual Verification

- **What it does**: Provides browser automation tools (navigate, click, snapshot, screenshot, evaluate, network inspection) powered by Playwright MCP integration.
- **When to use**:
  - Frontend work: mandatory visual verification of every rendered page or component change
  - API testing: verify the running application responds correctly end-to-end
  - DOM inspection: analyze component structure, accessibility tree, layout
- **Integration into workflow**: After any frontend change, use Playwright to open the page, take a screenshot, inspect the DOM, and confirm correctness before committing.

---

### MCP Servers (Model Context Protocol)

#### `jetbrains` — IntelliJ IDEA Integration

- **What it does**: Bridges Claude Code with IntelliJ IDEA 2025.3 — open files, navigate symbols, run configurations, read diagnostics.
- **When to use**: Navigating large Java codebases, running/debugging Spring Boot from the IDE, checking live compiler errors before committing.
- **Note**: Requires IntelliJ IDEA to be running with the MCP server plugin active.

#### `context7` — Live Library Documentation

- **What it does**: Fetches up-to-date documentation for project libraries on demand. Always fetch current docs before writing new integration code.
- **When to use**: Whenever implementing features using a library from the list below.
- **Available documentation handlers**:

  | Handler | Library |
  |---|---|
  | `/websites/spring_io_projects_spring-ai` | Spring AI |
  | `/spring-projects/spring-boot` | Spring Boot |
  | `/projectlombok/lombok` | Lombok |
  | `/openai/openai-java` | OpenAI Java SDK |
  | `/websites/platform_openai` | OpenAI Platform |
  | `/vercel/ai` | Vercel AI SDK |
  | `/assistant-ui/assistant-ui` | assistant-ui |
  | `/reactjs/react.dev` | React 19 |
  | `/tailwindlabs/tailwindcss.com` | Tailwind CSS |
  | `/shadcn-ui/ui` | Shadcn/ui |

---

### Skills

Skills are loaded knowledge packs that prime Claude Code with deep expertise in a specific domain. Invoke them when working in their target area.

| Skill | Invocation | When |
|---|---|---|
| `java-spring-boot` | `Skill("java-spring-boot")` | Any backend Spring Boot task |
| `java-testing` | `Skill("java-testing")` | Any Java test class |

> Full invocation details and security notes are in `src/AGENTS.md`.

---

## General Code Quality

- No unused imports, dead code, or commented-out blocks
- Environment variables for all secrets; never hardcode credentials
- No `.db` database files committed to the repository
- Commit messages: `Area: short summary` (e.g., `Feature: add chat SSE endpoint`, `Fix: null response on empty order`)

---

## Coding Style

- Java / Spring Boot conventions: see `src/AGENTS.md`
- TypeScript / React conventions: see `Frontend/AGENTS.md`

---

## Commit & Pull Request Guidelines

- Follow `Area: short summary` prefix convention (e.g., `Docs:`, `Feature:`, `Fix:`)
- PRs should include: goal, scope of changes, and any required setup notes (env vars, database files, API keys)
- Run `/code-review` before marking a PR ready for review

## Security & Configuration

- Configure all keys in environment variables (e.g., `OPENAI_API_KEY`)
- SQLite DB file is local-only; never commit `.db` files

## Agent Workflow Expectations

- Start by reading `docs/PRD-Sinsay-PoC.md` and `docs/ADR-Sinsay-PoC.md`
- Read the scoped `AGENTS.md` for the area you are working in (`src/AGENTS.md` for backend, `Frontend/AGENTS.md` for frontend)
- Keep changes aligned with the PoC scope (no auth, no production deployment)
- When adding new structure (backend/frontend), update the relevant `AGENTS.md` accordingly
- Use Context7 MCP to fetch current library documentation before implementing new integrations
- Use the JetBrains MCP for IDE-level navigation and diagnostics when available
- Run `/code-review` on all non-trivial PRs

---

## Symlink

This file is the single source of truth. Claude Code accesses it through a symlink:

```bash
# Run once from the project root to create the symlink:
ln -s AGENTS.md CLAUDE.md
```

Do not maintain a separate `CLAUDE.md` file — the symlink ensures both agents and Claude Code read identical content.
