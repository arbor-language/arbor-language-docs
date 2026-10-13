# Arbor documentation

The first language-documentation edition is based on [DESIGN.md](DESIGN.md).
The site contains a language tour, topic-based reference, tooling documentation,
and a catalog of provisional syntax and unresolved decisions. It describes the
design rather than claiming implementation availability.

## Preview locally

Use Python 3.10 or newer. From this repository:

```sh
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/mkdocs serve
```

Open the local URL printed by MkDocs. To build the static site with warnings
treated as errors:

```sh
.venv/bin/mkdocs build --strict
```

The generated site is written to `site/`, which is ignored by Git. No Arbor
compiler is needed to build the documentation. The dependencies in
`requirements.txt` provide MkDocs and its Material theme.

## Edit the documentation

- Start at [docs/index.md](docs/index.md) for the reader-facing entry point.
- Edit topic pages under `docs/language/`, `docs/tooling/`, and `docs/library/`.
- Keep the language tour consistent with the reference chapters.
- Add new pages to `nav` in [mkdocs.yml](mkdocs.yml).
- Keep local links relative and include meaningful labels for examples with provisional syntax.
- Record unresolved semantics in [Design status](docs/about/status.md) and in the affected chapter.
- Update the design baseline alongside any intentional semantic change.

Arbor examples currently render as plain code with a copy button because an
Arbor syntax highlighter is not configured. They have not been compiler-checked.
Do not turn illustrative names, APIs, or manifest fields into finalized rules
without a design decision.

MkDocs remains the site generator until Arbor's own documentation tool is ready.
Markdown content and ordinary relative links keep the documentation independent
of that future implementation. See the upstream [MkDocs configuration guide](https://www.mkdocs.org/user-guide/configuration/)
and [Material asset configuration](https://squidfunk.github.io/mkdocs-material/setup/changing-the-logo-and-icons/)
for site-maintenance details.
