---
title: Documentation contribution guide
description: Write, translate, review, and add screenshots to the Freya NLE manual.
---

# Documentation contribution guide

Documentation is maintained in the separate
[Aviqtl-Plus-docs repository](https://github.com/GT-610/Aviqtl-Plus-docs).
Do not submit documentation changes through the application repository's
submodule directory unless you are intentionally updating the documentation
repository itself.

## Information architecture

- `guide/`: task-oriented user documentation; this is the primary manual.
- `reference/`: terminology and detailed lookup material.
- `developer/`: building, architecture, APIs, effects, and plugins.
- `contributing/`: documentation process and project participation.
- `zh-CN/`: Simplified Chinese pages mirroring the English structure.
- `public/images/`: screenshots and other static media.

## Writing style

- Start with the result the reader wants.
- Use numbered steps for a workflow and bullets for choices or checks.
- Use the exact interface label, then explain what it does.
- Keep developer implementation details out of user tasks.
- State when a feature or page is incomplete.
- Do not copy text or images from another editor's manual.

## English and Simplified Chinese

English is the source language. Add or update the English page first, then
update the matching path under `zh-CN/`. A translation may be published later,
but it should not invent behavior absent from the English source.

## Add screenshots

Add the image file under `public/` first, then use VitePress's standard
Markdown image syntax:

```md
![Meaningful description of the interface state](/images/en/guide/example.webp)
```

Add the image under `public/` at the matching path; the example is
`public/images/en/guide/example.webp`. Use English captures under `images/en/`
and Simplified Chinese captures under `images/zh-CN/`.

## Preview changes

```bash
bun install
bun run docs:dev
```

Before opening a pull request, run:

```bash
bun run docs:build
```

Local search is generated during the VitePress build, so no external search
service or index file needs to be committed.
