---
name: study-note
description: Convert raw lecture/video notes into a polished study-note Markdown file. Use when the user provides draft notes from a session/episode (e.g. "convert this to a study note", "write this up as a note", "make a markdown note for episode N") and wants the output to follow the established layout used in `part_1/ais1e1.md` and `part_1/ais1e2.md`.
---

# Study Note Skill

Use this skill when the user asks to turn raw notes (bullet drafts, lecture transcripts, video summaries) into a finished study-note Markdown file.


## Required document structure

Every study note MUST follow this exact ordered layout:

1. **Title (H1)** — `# ML Foundation — AI Session <S>, Episode <E>`
2. **`## Summary`** — 2–4 sentences. State (a) the topic, (b) the key concepts introduced, (c) how they connect, and (d) — only if the user provided a reference URL — a trailing `For more check [reference](<url>).` link. Do not invent reference URLs.
3. **Horizontal rule** (`---`)
4. **`## Table of Contents`** — bulleted list of in-page anchor links to every H2 section that follows. Sub-bullets for H3 subsections when present.
5. **Horizontal rule**
6. **One H2 section per major concept**, in the order they appear in the source notes. Separate sections with horizontal rules.
7. Use H3 subsections inside an H2 when a concept has clearly distinct parts (e.g. `### ID3` and `### CART` under "How to Train a Decision Tree").

## Anchor link conventions

GitHub-style: lowercase, spaces → hyphens, drop punctuation, drop parentheses **and their contents**.

- `## Decision Tree (Model)` → `#decision-tree-model`
- `## Entropy and Information Gain` → `#entropy-and-information-gain`
- `### ID3` → `#id3`

## Formatting conventions

- **Math:** use LaTeX. Inline with `$...$`, display with `$$...$$` on its own lines with a blank line above and below.
- **Variables / symbols:** always wrap in math delimiters (e.g. `$\vec{x}$`, `$H(X)$`, `$\alpha$`), never plain text.
- **Tables:** use Markdown pipe tables to compare options, list properties, or summarize formulas. Prefer a table over a long bullet list when there are 2+ parallel items with the same shape.
- **Code blocks** (triple backticks, no language) for ASCII tree/diagram sketches.
- **Bold** key terms on first introduction (`**decision tree**`).
- **Blockquotes** (`>`) for short intuition callouts, side notes, or recommended reading.
- **Bullet lists** for enumerations; numbered lists only for genuinely ordered steps or stopping conditions.

## Content conventions

- **Faithful to the draft:** preserve the user's structure, examples, and ordering. Do not introduce concepts the draft did not mention.
- **Fill gaps explicitly:** if the draft has placeholders (`H(x) = …`, `Varricumee = sum of (x - ..)^2`, "We defined the alpha"), complete them with the standard textbook form.
- **Translate shorthand:** expand abbreviations and rough phrasings into clean prose, but keep the technical content the user wrote.
- **Examples stay concrete:** if the draft includes a worked example (Huffman coding, loan recovery), reproduce it with its numbers intact.
- **No invented citations or URLs.** If the user did not supply a reference link, omit the reference line from the summary.

## Reference template

The canonical example to mirror is `part_1/ais1e1.md`. Read it before writing if you need a reminder of spacing, table syntax, or section flow. `part_1/ais1e2.md` is a second example covering a different topic shape (algorithm comparison + worked example).
