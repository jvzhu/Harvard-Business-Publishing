# LaTeX Document Setup Reference

A practical reference for preamble structure, Overleaf code environments, bibliography styles, and multi-byte (CJK) encoding — geared toward humanities manuscripts that mix Western and East Asian text.

---

## 1. Custom LaTeX Preambles

A preamble is easiest to maintain when it's organized in a consistent order: engine/encoding → fonts → page geometry → language → math/graphics → cross-referencing → bibliography → custom macros. Loading things out of order is the most common source of mysterious compile errors.

```latex
% ========== 1. Document class ==========
\documentclass[12pt, a4paper]{article}

% ========== 2. Encoding & engine ==========
% If compiling with pdfLaTeX:
\usepackage[utf8]{inputenc}
\usepackage[T1]{fontenc}
% If compiling with XeLaTeX/LuaLaTeX, omit inputenc/fontenc —
% UTF-8 and font access are native. See Section 4.

% ========== 3. Fonts ==========
\usepackage{fontspec}       % XeLaTeX/LuaLaTeX only
\setmainfont{Times New Roman}
% \usepackage{mathptmx}      % pdfLaTeX alternative (Times-like text+math)

% ========== 4. Page geometry ==========
\usepackage[margin=1in]{geometry}
\usepackage{setspace}
\doublespacing

% ========== 5. Language & typography ==========
\usepackage[english]{babel}
\usepackage{csquotes}       % smart quotes, required by biblatex

% ========== 6. Graphics, tables, math ==========
\usepackage{graphicx}
\usepackage{booktabs}
\usepackage{amsmath, amssymb}

% ========== 7. Cross-referencing (load late, in this order) ==========
\usepackage{hyperref}
\usepackage[nameinlink]{cleveref}   % load AFTER hyperref

% ========== 8. Bibliography ==========
\usepackage[style=chicago-authordate, backend=biber]{biblatex}
\addbibresource{references.bib}

% ========== 9. Custom macros ==========
\newcommand{\jpn}[1]{\textit{#1}}   % e.g. italicize romanized Japanese terms
```

**Preamble conventions worth adopting:**
- Keep one `\usepackage` per conceptual purpose, not per line-saving instinct — makes diffing in Overleaf's history view much easier.
- `hyperref` should load near the *end* of the preamble (most packages need to know about it, not the reverse); `cleveref` must load *after* `hyperref`.
- For a multi-file project (e.g., splitting chapters), keep the preamble in its own `preamble.tex` and pull it in with `\input{preamble.tex}` — this is the cleanest way to version-control a shared style across a dissertation with several chapter files.
- Comment liberally with `% ==========` section breaks; six months later you will not remember why a package is there.

---

## 2. Overleaf Code Environments

Overleaf-specific behavior to know:

- **Project structure**: `main.tex` is the compiled root by default (set under Menu → Settings → "Main document" if you have multiple `.tex` files). Chapters/sections should be separate `.tex` files pulled in via `\input{}` (keeps everything in one compile) or `\include{}` (allows selective compilation of individual chapters, useful for long dissertations — but each `\include`d file starts a new page).
- **Compiler choice**: Menu → Settings → Compiler. For CJK content, use **XeLaTeX** or **LuaLaTeX**, not pdfLaTeX (see Section 4).
- **Biber vs. BibTeX backend**: Menu → Settings → "Bibliography processor." If you're using `biblatex` with `backend=biber` in your preamble, this dropdown must also say Biber, or Overleaf will silently fail to update your bibliography.
- **Verbatim/code blocks**: for reproducing code, transliteration tables, or monospaced excerpts:
  ```latex
  \usepackage{listings}
  \lstset{basicstyle=\ttfamily\small, breaklines=true}
  \begin{lstlisting}[language=Python]
  print("example")
  \end{lstlisting}
  ```
  or for simpler cases, the built-in `verbatim` environment (no package needed):
  ```latex
  \begin{verbatim}
  literal text, preserved exactly
  \end{verbatim}
  ```
- **Git integration**: Overleaf Premium projects can sync to a GitHub repo (Menu → GitHub). Given your `conference-paper` repo already holds the LaTeX source for "The Corporate Stage," linking it lets you edit in Overleaf and push straight to GitHub instead of manual export/upload.
- **Rich text mode**: fine for quick edits, but switch back to source/code mode before touching anything with custom macros, `\cite` commands, or CJK input — rich text mode can mangle raw commands.

---

## 3. Bibliography Style Files (BibTeX/biblatex)

Two ecosystems exist — don't mix them:

| | Classic BibTeX (`.bst`) | biblatex (`.bbx`/`.cbx`, backend biber) |
|---|---|---|
| Engine | `\bibliographystyle{}` + `\bibliography{}` | `\usepackage[style=...]{biblatex}` |
| Flexibility | Fixed styles, hard to customize | Highly customizable via options |
| CJK/Unicode | Poor — BibTeX itself is 8-bit | Native UTF-8 support |
| Recommended for new projects | No | **Yes** |

### Chicago (Humanities / Notes-Bibliography or Author-Date)
```latex
\usepackage[style=chicago-notes, backend=biber]{biblatex}   % footnote style
% or
\usepackage[style=chicago-authordate, backend=biber]{biblatex}  % (Author Year)
```
Requires the `biblatex-chicago` package. Notes style auto-generates `\footcite{}`; author-date uses `\cite{}` / `\parencite{}`.

### MLA 9th edition
```latex
\usepackage[style=mla-new, backend=biber]{biblatex}
```
Provided by `biblatex-mla`. Handles the MLA 9th "container" model (nested sources — e.g., an article within a journal within a database) more accurately than older MLA packages.

### APA 7th edition
```latex
\usepackage[style=apa, backend=biber]{biblatex}
\DeclareLanguageMapping{english}{english-apa}
```
Provided by `biblatex-apa`. The `\DeclareLanguageMapping` line is required for correct "and"/"&" and date formatting per APA rules.

### Common setup regardless of style
```latex
\addbibresource{references.bib}
...
\printbibliography
```
In Overleaf: set the bibliography processor to **Biber** (not BibTeX) whenever using any `biblatex`-based style above — this is the single most common cause of "my bibliography won't update" tickets.

**`.bib` entry tip for East Asian sources**: use the `langid` field to trigger correct sorting/punctuation per entry, and store romanization + original script together:
```bibtex
@book{zhang1695,
  author    = {Zhang, Zhupo},
  title     = {Piping Jin Ping Mei (批評金瓶梅)},
  year      = {1695},
  langid    = {english},
  note      = {Commentary edition},
}
```

---

## 4. Multi-byte Character Encoding (CJK & Unicode)

This is the section most likely to cause silent corruption if skipped.

**The core decision: compiler.**
- **pdfLaTeX** was designed for 8-bit encodings. It *can* be forced to handle UTF-8 via `inputenc`, but it cannot natively render CJK glyphs — you'd need `CJKutf8` and a separate font-loading mechanism, which is fragile.
- **XeLaTeX** or **LuaLaTeX**: both are Unicode-native, read UTF-8 source files directly, and use `fontspec` to access any installed system font (including CJK fonts) without special encoding packages. **This is the right choice for any document mixing Chinese/Japanese with Western text.**

### Recommended CJK setup (XeLaTeX/LuaLaTeX)
```latex
\documentclass{article}
\usepackage{fontspec}
\usepackage{xeCJK}                  % XeLaTeX; use luatexja for LuaLaTeX instead

\setmainfont{Times New Roman}
\setCJKmainfont{Noto Serif CJK SC}  % or Noto Serif CJK JP / KR as needed
\XeCJKsetup{CJKspace=true}          % correct spacing around CJK punctuation
```

For **mixed Chinese + Japanese in the same document** (relevant for comparative work spanning both traditions), you can define separate font commands rather than relying on a single CJK font that may not have full glyph coverage for both:
```latex
\newCJKfontfamily\zhfont{Noto Serif CJK SC}
\newCJKfontfamily\jpfont{Noto Serif CJK JP}

% usage:
\zhfont{金瓶梅}
\jpfont{間}
```

### Practical pitfalls
- **File encoding**: save/verify all `.tex` and `.bib` files as UTF-8 (Overleaf does this by default; watch for issues only if you're uploading files edited elsewhere, e.g., older Word exports with Windows-1252 remnants).
- **BibTeX + CJK don't mix**: classic BibTeX cannot handle multi-byte characters in `.bib` entries. If any bibliography entry has CJK author names, titles, or annotations, you need `biblatex` + `biber`, not `bibtex`, full stop.
- **Punctuation width**: CJK punctuation (、。「」) is full-width by convention; `xeCJK`'s `CJKspace` and `CJKecglue` options control spacing between CJK and Latin characters — worth checking visually rather than trusting defaults, especially at line breaks.
- **Search/copy-paste from PDF**: XeLaTeX-compiled PDFs with CJK are searchable/copyable by default as long as the font used embeds proper Unicode mappings (Noto and most modern OpenType CJK fonts do; some older bitmap CJK fonts don't).
- **Line-breaking**: CJK text doesn't use spaces between words, so LaTeX's default space-based line breaker won't wrap it correctly — `xeCJK`/`luatexja` both patch this automatically, which is another reason to avoid hand-rolling a pdfLaTeX CJK solution.

---

## Quick-start template (Chicago author-date + CJK, XeLaTeX)

```latex
\documentclass[12pt]{article}
\usepackage{fontspec}
\usepackage{xeCJK}
\setmainfont{Times New Roman}
\setCJKmainfont{Noto Serif CJK SC}

\usepackage[margin=1in]{geometry}
\usepackage{setspace}\doublespacing
\usepackage{csquotes}
\usepackage{graphicx, booktabs, amsmath}
\usepackage{hyperref}
\usepackage[nameinlink]{cleveref}

\usepackage[style=chicago-authordate, backend=biber]{biblatex}
\addbibresource{references.bib}

\title{Your Title}
\author{Your Name}

\begin{document}
\maketitle
% content
\printbibliography
\end{document}
```

Remember to set **Compiler: XeLaTeX** and **Bibliography processor: Biber** in Overleaf's project settings — the preamble alone won't do it if those dropdowns are set to defaults.
