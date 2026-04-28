---
name: study-notes-system
description: Organize scattered personal study materials into a durable note system with plan, source, notes, review, practice, and snippets layers. Use when a workspace has many learning documents, code demos, or course materials that need to be sorted, renamed, indexed, deduplicated, or extended without losing the main topic entrypoint.
---

# Study Notes System

Use this skill to keep a study repository usable as the number of notes grows.

## Core Rules

1. Keep daily planning and topic knowledge separate.
2. Treat each topic as a long-lived thread, not as a date-bound file.
3. Prefer updating an existing topic note over creating a same-meaning duplicate.
4. Keep explanations, review notes, practice code, and reusable templates in different places.

## Repo Layers

Use or maintain this structure when the repo matches a personal study workflow:

- `00-计划/`: Daily plan, progress, task checklists
- `01-原始资料/`: Course handouts, screenshots, PDFs, copied tutorial material
- `02-学习笔记/`: Main topic notes written in the learner's own words
- `03-复习速记/`: Short review sheets, easy-to-forget tricks, interview-style answers
- `04-练习代码/`: SQL files, Java demos, runnable practice artifacts
- `05-常用片段/`: Reusable templates, starter snippets, command patterns

## Sorting Rules

When deciding where something belongs:

- Put teacher-provided or copied material in `01-原始资料/`.
- Put explained understanding in `02-学习笔记/`.
- Put short recall aids in `03-复习速记/`.
- Put runnable verification in `04-练习代码/`.
- Put reusable starter patterns in `05-常用片段/`.

When content spans multiple days:

- Continue the same topic file in `02-学习笔记/`.
- Continue the same review file in `03-复习速记/`.
- Append new exercises under the same practice topic directory.
- Record dates inside the document only when timeline matters.

## File Growth Guardrails

When the repo starts to feel messy:

1. Identify the topic's main note before creating a new file.
2. Merge duplicate "summary", "final", or "review again" files into one durable topic note.
3. Add or refresh `README.md` index files for note-heavy directories.
4. Keep file names direct and topic-based; avoid vague names such as `整理版`, `最终版`, or `笔记1`.

## Operating Rhythm

Use this rhythm for ongoing study:

1. Start from `00-计划/` to decide today's topics.
2. Learn with raw materials in `01-原始资料/` and write the durable note in `02-学习笔记/`.
3. End the session by extracting a short review file into `03-复习速记/`.
4. When code or SQL is involved, add runnable examples under `04-练习代码/`.
5. When a pattern becomes reusable, move it into `05-常用片段/`.

## Output Style

When updating the repo with this skill:

- Prefer Chinese names for note files if the repo already uses Chinese names.
- Prefer English names for code directories and runnable project folders.
- Keep changes minimal and structural; do not rewrite well-formed notes unless needed.
- When reorganizing, preserve existing content and move it into clearer layers instead of deleting it.
