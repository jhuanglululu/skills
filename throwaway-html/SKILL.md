---
name: throwaway-html
description: Writing throwaway html for presenting longer or more complex content to user. Use this with planning skill or interviewing skill. Don't use this when designing a repeatedly-used html page.
allowed-tools: Bash(command -v showme), Bash(showme *), Read
---

# Throwaway HTML

General guidance, not law: when the user's prompt says otherwise, the prompt wins.

HTML allows for a more expressive way of delivering content to the user than markdown does.

## When to use

- Content with a structure markdown handles poorly: side-by-side comparisons, decision trees, timelines, dense tables with grouping
- Presenting a plan/proposal where visual hierarchy aids skimming
- NOT for: short answers, code review, anything the user will edit or reuse

## General guidelines

- Write to `context/presentations/` (see the **context-folder** skill for location, naming, and the throwaway rule).
- A single self-contained file: inline the JS and CSS.
- If the page includes code, syntax-highlight it — a CDN library is fine — assume an internet connection.
- One file per presentation (one AskUserQuestion tool call or one summary); if the content has multiple categories, use tabs to structure them.
- Keep it simple; reference plans or designs by filename if needed.

## Style defaults

- Simple, light-mode, top-down page; GitHub's markdown rendering is a good starting point
- No boilerplate polish: skip headers/footers/branding, dark-mode toggles, responsive breakpoints
- Give each section a clear title so that the user can reference it easily.

## Presenting the content

!`command -v showme >/dev/null 2>&1 && showme --how-to-use || echo "showme (a CLI tool that serves files to a private website) is not installed. Ask the user whether the showme invocation at ${CLAUDE_SKILL_DIR}/SKILL.md should be removed; only remove it if they confirm."`

- Point the user to the file. Do not open the file in a browser for the user through any bash command; doing so can interrupt what the user is working on.
