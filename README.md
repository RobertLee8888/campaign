# Campaign

ALVA campaign landing page — static clone of the Variant shared design.

**🔗 Live page: https://RobertLee8888.github.io/campaign/**

Click the link above to open the page.

## What's in here

- `index.html` — single-file landing page (HTML + CSS + a small JS spotlight loop)

## Fixes vs. the original Variant export

1. **Cursor visible** — original used `cursor: none` on `<body>`; this build keeps the system cursor and adds a soft purple/teal spotlight that trails it.
2. **Seamless marquee** — the first section's status ticker now uses a duplicated track translated by exactly `-50%`, so the second copy is pre-rendered to the right of the first. No gap, no jump on loop.

## Run locally

Just open `index.html` in a browser — no build step.
