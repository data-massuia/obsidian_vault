# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a personal Obsidian vault for daily journaling, task tracking, and knowledge management. It is not a software project — there are no build steps, tests, or deployable code.

## Vault Structure

- `00 Daily 2026/` — Daily journal entries named `YYYY-MM-DD.md`
- `Templates/` — Templater-powered templates for daily notes
- `Learning/` — Learning and reference notes
- `Weekly Notes/`, `Monthly Notes/` — Planned periodic review folders
- `.obsidian/` — Obsidian configuration (do not edit manually unless necessary)

## Key Plugins & Their Syntax

- **Templater** — Templates use `FOR2PMT` syntax for dynamic dates
- **Dataview** — Notes use `dataview` code blocks for dynamic queries (e.g., listing notes created/modified on a given date)
- **Tasks** — Checkbox tasks (`- [ ]`) are tracked across the vault with due dates and recurrence
- **Calendar** — Provides calendar navigation for daily notes

## Daily Note Template

The primary template (`Templates/Daily Note Template.md`) generates:
- YAML frontmatter with `created`, `tags: daily-notes`, and `week` properties
- Navigation links to previous/next day
- Sections: Morning Intentions, Tasks, Daily Log, Evening Reflection, Today's Activity

## Conventions

- Daily notes use `YYYY-MM-DD` format for filenames and linking
- Weekly notes link format: `YYYY-[W]ww` (e.g., `2026-W07`)
- Monthly notes link format: `YYYY-MM` (e.g., `2026-02`)
- Dataview queries in daily notes use `{{date:YYYY-MM-DD}}` placeholder (resolved by Obsidian core daily notes)
- Templater expressions use `undefined` delimiters (resolved when template is applied)
