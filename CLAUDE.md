# PROJECT_NAME

A Japanese mathematics book written in LuaLaTeX (`ltjsbook`). Build with `mise run build` (llmk, see `src/llmk.toml`); the PDF is `src/main.pdf`.

TODO: Describe the book (topic, audience, target event or release).

# RTK (Rust Token Killer)

Prefix every shell command with `rtk`, including each command in an `&&` chain — it is always safe (a dedicated filter cuts noisy output for tests, builds, git, and more; anything without one passes through unchanged). The full command reference is in the global `~/.claude/RTK.md` (already loaded, if set up). Meta commands: `rtk gain` (savings so far), `rtk discover` (missed opportunities in past sessions), `rtk proxy <cmd>` (run unfiltered, for debugging).

## Working conventions

TODO: Describe the development conventions for this project (branching strategy, commit granularity, whether reviews are required, etc.).

- `git commit` runs the lefthook hooks. If they fail, fix the reported issues. Do not use `--no-verify`.

- Run `mise run check` after making changes.
- Format TeX sources with `mise run fmt` (latexindent, settings in `.latexindent.yaml`). Indent with tabs.
- Write inline math as `\(...\)`. Use `\cref`/`\Cref` for references, with labels of the form `Env:id` (e.g. `Def:group`, `Thm:lagrange`).
- Mark new terms with `\term[reading]{term}` so they enter the word index; add symbols to the symbol index with `\index[sidx]{reading@symbol}`.
- Japanese prose uses `，` and `．` as punctuation and the である style.
- The prose of `.tex` files is checked by textlint (`pnpm lint`, via `@enunun/textlint-plugin-latex`). When you define a new command or environment whose argument or body is prose, register how to read it under `'@enunun/latex'` in `.textlintrc.yml` (for example `textCommands`). Silence a false positive with `% textlint-disable` / `% textlint-enable` around the smallest possible range.

## Code map

- `src/main.tex`: entry point. Book metadata (`\booktitle`, `\bookauthor`, ...) and the `\include` order of chapters.
- `src/preamble/`: content-independent preamble (packages, fonts, layout, theorem environments, generic math macros, indexes, hyperref/cleveref/biblatex). A new theorem environment goes in both `theorems.tex` and `references.tex`.
- `src/contents/`: one file per chapter. `intro.tex` is front matter, `answer.tex` holds exercise solutions.
- `src/colophon.tex`: colophon, built from the metadata in `main.tex`.
- `src/reference/book.bib`: bibliography. `src/fig/`: figures.
- `.textlintrc.yml`, `.textlintignore`: textlint settings for `.tex` and `.md`. `src/preamble/` and `src/colophon.tex` are not checked.
- `pnpm-workspace.yaml`: allows the build script of the textlint plugin installed from GitHub; remove it once the plugin is installed from npm.
- `.devcontainer/`: dev container based on the official `texlive/texlive` image, with mise copied in from the official mise image.
- `.github/workflows/build.yml`: manually triggered build (workflow_dispatch); given a tag input, it attaches the PDF to a GitHub Release.

# Artifact Cleanup

## Golden Rule

**Whenever you produce an artifact, always run the `system-development-skills:finalize-artifacts` skill to clean it up before reporting the work as done.**

An artifact is any deliverable you create or substantially rewrite: documents, READMEs, code and code comments, config files, scripts, commit messages, PR descriptions, and so on.

- Invoke the skill via the Skill tool (`system-development-skills:finalize-artifacts`) after the artifact is written and before the final reply.
- The skill edits the artifact files in place. Do not append a changelog of the cleanup to the artifact; in the final reply, mention what changed in a sentence or two at most unless the user asks for a full report.
- Skip it only for replies that produce no artifact (answering questions, explaining code, running read-only commands).
- Provided by the `enunun/system-development-skills` plugin (see `extraKnownMarketplaces`/`enabledPlugins` in `.claude/settings.json`).
