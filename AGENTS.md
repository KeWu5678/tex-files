# Repository Guidelines

## Project Structure & Module Organization

This repository contains LaTeX notes and thesis material. Top-level folders group subject areas:

- `thesis/`: main master's thesis source, bibliography, generated PDF, plots, and reference notes.
- `thesis/Mthesis.tex`: primary thesis document.
- `thesis/ref.bib`: BibTeX database used by the thesis.
- `thesis/plot/`: tracked PNG figures referenced from `Mthesis.tex`.
- `Analysis/`, `Optimization/`, `Numerica/`, `Stat_Stoch/`: standalone course or research note `.tex` files.
- Root Markdown files such as `CLAUDE.md`, `LOG.md`, and `greedy_algorithm_q_penalty.md` contain project notes.

The `.gitignore` is restrictive: only `.tex`, `.md`, `thesis/ref.bib`, `thesis/Mthesis.pdf`, and thesis plot PNGs are intended to be tracked.

## Build, Test, and Development Commands

Build the thesis from inside `thesis/` so relative figure and bibliography paths resolve:

```sh
cd thesis
latexmk -pdf Mthesis.tex
```

If `latexmk` is unavailable, use the manual sequence:

```sh
cd thesis
pdflatex Mthesis.tex
bibtex Mthesis
pdflatex Mthesis.tex
pdflatex Mthesis.tex
```

Clean auxiliary LaTeX outputs:

```sh
cd thesis
latexmk -c Mthesis.tex
```

There is no automated test suite. Treat a clean LaTeX build without unresolved references or missing figures as the main validation step.

## Coding Style & Naming Conventions

Use standard LaTeX formatting with readable line breaks around paragraphs, displayed equations, theorem environments, and long lists. Keep labels descriptive and stable, for example `\label{assumption on actfun}` or `\label{alg:q}`. Prefer ASCII text unless the file already uses non-ASCII names or citations. Store thesis figures in `thesis/plot/` and reference them with paths relative to `thesis/`.

## Testing Guidelines

Before committing substantial edits, compile the affected main document. For thesis changes, inspect `Mthesis.pdf` when possible and check for warnings about undefined citations, missing references, or missing image files. Bibliography edits should be verified by running BibTeX as part of the full build.

## Commit & Pull Request Guidelines

Recent commits use short, informal summaries such as `updates`, `updated the experiment`, and `sync all the .md and plots`. Keep commits brief but more specific when possible, for example `revise activation discussion` or `add thesis reference notes`.

Pull requests should describe the changed documents, mention whether `Mthesis.tex` was rebuilt successfully, and include screenshots or PDF page references for visual/layout changes.

## Agent-Specific Instructions

Do not automatically edit repository files. First explain the intended change and wait for explicit approval before applying patches or modifying files.

When answering mathematical questions, do not use LaTeX markup. Use plain text or readable math symbols instead.
