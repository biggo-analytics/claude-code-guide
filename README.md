# Claude Code Guide (Thai)

> **คู่มือการใช้งาน Claude Code สำหรับทีมพัฒนาซอฟต์แวร์ — ภาษาไทย**
>
> A comprehensive Thai-language guide to [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (Anthropic's CLI agent) for full-stack development teams.

## Overview

This repository contains a 9-chapter internal guide covering Claude Code from fundamentals to advanced team workflows. Written in Thai for teams working with **Go / React / PostgreSQL** stacks, though the concepts apply broadly.

**Total:** ~7,100 lines across 9 chapters (~316 KB)

## Table of Contents

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
