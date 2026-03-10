# Claude Code Guide (Thai)

> **คู่มือการใช้งาน Claude Code สำหรับทีมพัฒนาซอฟต์แวร์ — ภาษาไทย**
>
> A comprehensive Thai-language guide to [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (Anthropic's CLI agent) for full-stack development teams.

## Overview

This repository contains a 9-chapter internal guide covering Claude Code from fundamentals to advanced team workflows. Written in Thai for teams working with **Go / React / PostgreSQL** stacks, though the concepts apply broadly.

**Total:** ~10,900 lines across 9 chapters + 8 example files (~480 KB)

## Table of Contents

### Chapters (Reference)

| # | File | Title | Description |
|---|------|-------|-------------|
| 01 | [01_intro.md](01_intro.md) | Introduction & Fundamentals | What Claude Code is, installation, configuration, CLAUDE.md, and first workflows |
| 02 | [02_agentic_mastery.md](02_agentic_mastery.md) | Agentic Mastery & Best Practices | Mastering agentic coding — prompt patterns, iterative development, effective collaboration |
| 03 | [03_advanced_usage.md](03_advanced_usage.md) | Advanced Usage | MCP servers, hooks, custom slash commands, multi-repo workflows, CI/CD integration |
| 04 | [04_fullstack_collaboration.md](04_fullstack_collaboration.md) | Full-Stack Collaboration | Working across Backend (Go/Fiber), Frontend (React/TS), Database (PostgreSQL/GORM), Docker, and DEVLOG patterns |
| 05 | [05_spec_driven_dev.md](05_spec_driven_dev.md) | Specification-Driven Development | Using specs and structured documents to drive development with Claude Code |
| 06 | [06_architecture_design.md](06_architecture_design.md) | Architecture & Design | System architecture, design patterns, and architectural decision-making with Claude Code |
| 07 | [07_qa_security_review.md](07_qa_security_review.md) | QA, Security & Review | Quality assurance, security auditing, code review workflows |
| 08 | [08_roles_responsibilities.md](08_roles_responsibilities.md) | Roles & Responsibilities | How different roles (Dev, SA, BA, PM, QA, DevOps) can leverage Claude Code |
| 09 | [09_cheat_sheet.md](09_cheat_sheet.md) | Cheat Sheet | Quick reference — commands, workflows, and common patterns |

### Examples — ShopFast Project Walkthrough

End-to-end narrative walkthrough of a mid-size E-commerce project, showing how each role uses Claude Code across all project phases.

| File | Phase | Description |
|------|-------|-------------|
| [00_overview.md](examples/00_overview.md) | — | Project intro, team roster, tech stack, reading guide |
| [01_phase1_inception.md](examples/01_phase1_inception.md) | Inception | Architecture, CLAUDE.md, specs, dev env setup |
| [02_phase2_database_backend.md](examples/02_phase2_database_backend.md) | DB & Backend | ER design, GORM models, migrations, base CRUD |
| [03_phase3_core_features.md](examples/03_phase3_core_features.md) | Core Features | Auth, business logic, search, tests, debugging |
| [04_phase4_frontend.md](examples/04_phase4_frontend.md) | Frontend | React admin panel, TypeScript types, API hooks |
| [05_phase5_integration_cicd.md](examples/05_phase5_integration_cicd.md) | Integration & CI/CD | Integration tests, CI pipeline, production Docker |
| [06_phase6_maintenance.md](examples/06_phase6_maintenance.md) | Maintenance | 5 scenarios: bug fix, new feature, refactor, onboarding, perf |
| [07_prompt_catalog.md](examples/07_prompt_catalog.md) | — | All prompts (~80) organized by role AND by phase |

## Who Is This For?

| Role | Key Chapters |
|------|-------------|
| **Developers** | 01–04 (fundamentals → full-stack workflows) |
| **Solution Architects** | 05–06 (spec-driven dev, architecture) |
| **Business Analysts** | 05, 08 (specs, roles) |
| **Project Managers** | 08 (roles & responsibilities) |
| **QA Engineers** | 07 (QA, security, review) |
| **DevOps Engineers** | 03–04 (advanced usage, CI/CD, Docker) |

## Tech Stack Context

Examples and workflows in this guide are based on:

- **Backend:** Go (Fiber framework, GORM)
- **Frontend:** React, TypeScript
- **Database:** PostgreSQL
- **Infrastructure:** Docker, CI/CD pipelines

The principles and patterns are applicable to other stacks as well.

## How to Use

1. **New to Claude Code?** Start with [Chapter 01](01_intro.md) for installation and basics.
2. **Want to level up?** Read [Chapter 02](02_agentic_mastery.md) for effective prompting and agentic workflows.
3. **Looking for specific topics?** Use the table of contents above to jump to the relevant chapter.
4. **Quick reference?** Go straight to the [Cheat Sheet](09_cheat_sheet.md).
5. **Want a real-world walkthrough?** Read the [ShopFast Examples](examples/00_overview.md) — an end-to-end project walkthrough showing all roles and phases.
6. **Just need prompts?** Copy from the [Prompt Catalog](examples/07_prompt_catalog.md) — 80+ prompts organized by role and phase.

## Updating This Guide

Claude Code evolves rapidly. To keep this guide current:

- Check [Anthropic's changelog](https://docs.anthropic.com/en/docs/claude-code) for new features
- Update relevant chapters when workflows change
- Add examples from real team usage
- Note deprecated features or changed behaviors

## Contributing

1. Fork this repository
2. Create a feature branch
3. Make your changes (keep Thai language consistent)
4. Submit a pull request with a description of what changed and why

## License

This work is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — you are free to share and adapt with attribution.
