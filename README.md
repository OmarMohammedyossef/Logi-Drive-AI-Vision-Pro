# Logi Drive AI Vision Pro — Documentation

### 🌐 **[View the live site → logi-drive-ai-vision-pro.vercel.app](https://logi-drive-ai-vision-pro.vercel.app/)**

---

Documentation site for the graduation project: a **Raspberry Pi + STM32 Advanced
Driver Assistance System (ADAS)**. A central Linux gateway (Raspberry Pi, custom
Yocto image) coordinates the safety subsystems — ACC, LKA, FCW — which run on
STM32 + FreeRTOS boards and talk to the gateway over **CAN**. Gateway-local
services communicate over **D-Bus**, the driver sees a **Qt/QML** dashboard, and a
**SIM900 GSM/GPRS** module provides the cellular uplink.

Built with [MkDocs](https://www.mkdocs.org/) and the
[Material](https://squidfunk.github.io/mkdocs-material/) theme.

---

## Quick start

```bash
# 1. Install the toolchain (once)
pip install -r requirements.txt

# 2. Run the site locally
mkdocs serve -a 127.0.0.1:8000
# → open http://127.0.0.1:8000

# 3. Build the static site into site/
mkdocs build --strict
```

> [!IMPORTANT]
> This project lives on an NTFS (`fuseblk`) mount, where `mkdocs serve`'s
> file-watcher (inotify) does **not** reliably fire. Live-reload will silently
> stop picking up your edits. **If a change doesn't appear, stop the server
> (`Ctrl+C`) and start it again.** `mkdocs build` is a one-shot process and is
> unaffected.

---

## Deployment (Vercel)

Live at **<https://logi-drive-ai-vision-pro.vercel.app/>**, rebuilt automatically
on every push to `main`.

Vercel has no built-in MkDocs preset, so the build is declared explicitly in
[`vercel.json`](vercel.json):

| Setting | Value | Why |
| --- | --- | --- |
| `framework` | `null` | Stops Vercel from guessing a preset and running `npm install` |
| `installCommand` | `python3 -m pip install -r requirements.txt` (with `--user` / `--break-system-packages` fallbacks) | Pinned versions, so the deploy matches local builds. `python3 -m pip` needs no `pip3` on `PATH`, and the fallbacks survive a build image that marks its Python [externally managed](https://peps.python.org/pep-0668/) |
| `buildCommand` | `python3 -m mkdocs build && find site -name '*.map' -delete` | `python3 -m` avoids `PATH` issues; source maps are dev-only |
| `outputDirectory` | `site` | Where MkDocs writes the static site |
| `ignoreCommand` | `git diff --quiet "$VERCEL_GIT_PREVIOUS_SHA" HEAD -- docs mkdocs.yml …` | Skips the build entirely when a commit doesn't touch the site |

### The built site is committed

`site/` is **checked into the repository**, and both `installCommand` and
`buildCommand` start by testing for `site/index.html`. Finding it, they skip
straight past the Python install and the MkDocs build, and Vercel publishes the
committed directory as-is.

So a deploy installs nothing, builds nothing, and cannot fail on a dependency,
a Python version, or a MkDocs release. Vercel is doing one job: serving files.

The build path is still there as a fallback — delete `site/` and the very next
deploy installs `requirements.txt` and runs `mkdocs build` instead. Nothing
selects between the two; the presence of the directory is the whole signal, so
there is no dashboard setting or env var that can drift out of sync with it.
The build log always says which path it took:

| | Log line |
| --- | --- |
| `site/` present | `==> Publishing the uploaded site/ as-is.` |
| `site/` absent | `==> No prebuilt site/ - building the docs with MkDocs.` |

> [!CAUTION]
> **Editing `docs/` is not enough — you must rebuild and commit `site/` too**,
> or the deploy will quietly publish the old pages. Vercel cannot tell a fresh
> build from a month-old one. Every docs change is therefore two steps:
>
> ```bash
> python3 -m mkdocs build --strict && find site -name '*.map' -delete
> git add docs site && git commit && git push
> ```
>
> If you would rather not think about it, delete `site/` and re-add the ignore
> rule — Vercel will go back to building the docs itself on every push.

### Importing the repo into Vercel

The repo is self-configuring: everything Vercel needs is committed, so a fresh
import needs **no dashboard settings at all**.

1. Push the repo to GitHub.
2. Vercel → **Add New… → Project** → import the repository.
3. Leave every field on its default — **Framework Preset `Other`**, **Root
   Directory** the repository root, and *do not* override Build/Output/Install
   Command. `vercel.json` takes precedence over the dashboard anyway.
4. **Deploy.** The first build installs MkDocs from `requirements.txt` and
   publishes `site/`.

> [!IMPORTANT]
> **Root Directory must be the repository root** (left empty) — *not* `site/`.
> Vercel reads `vercel.json` from the root directory, so pointing it at `site/`
> makes it skip the build entirely.
>
> `site/` is git-ignored — on a Git deploy Vercel regenerates it every time.

One more thing worth knowing: `ignoreCommand` in `vercel.json` **overrides** the
dashboard's *Ignored Build Step* setting, so changing that in the UI does nothing.
Vercel's convention is also inverted from a shell's — **exit code 0 skips the
build**, non-zero builds it. The command deliberately fails open: no previous
deployment, or a previous SHA missing from Vercel's shallow clone, means the
build runs.

**A 404 on the deployed URL almost always means the output directory was empty** —
either the build didn't run, or Vercel was looking in the wrong folder.

### Resource usage

This site is **100% static** — plain HTML, CSS, and JS. There are no serverless
functions, no SSR, and no API routes, so it uses **no runtime compute at all**:
every request is served straight from Vercel's CDN. The only consumption is build
minutes, and those are minimised too:

- **`ignoreCommand`** skips the build when a push doesn't touch `docs/`,
  `mkdocs.yml`, `requirements.txt`, or `vercel.json`.
- **Source maps are stripped** after the build (~1.3 MB less per deploy).
- **Fingerprinted assets** (`main.<hash>.min.css`, `bundle.<hash>.min.js`, …) are
  served `immutable` with a one-year cache; images, fonts, and the PDF get a week
  with `stale-while-revalidate`. Non-fingerprinted files such as `extra.css` are
  deliberately left on a short cache so edits show up immediately.

---

## Repository layout

```text
Logi-Drive-AI-Vision-Pro/
├── mkdocs.yml                 # site config: theme, plugins, navigation
├── vercel.json                # Vercel build command, output dir, cache headers
├── .vercelignore              # what a `vercel` CLI upload leaves out
├── requirements.txt           # pinned MkDocs / Material / glightbox versions
├── README.md                  # this file
├── site/                      # BUILT OUTPUT, committed — this is what Vercel serves
│                              # regenerate with `mkdocs build` after editing docs/
├── docs/
│   ├── index.md               # homepage: hero, architecture diagram, section cards
│   ├── assets/
│   │   ├── favicon.svg        # hand-written SVG favicon
│   │   └── stylesheets/
│   │       └── extra.css      # full-width layout + hero/chip styling
│   ├── architecture/          # main system, logger/diagnostics, ADAS feature survey
│   ├── server/                # backend + Qt/Docker build issues
│   ├── systemd/               # packaging components as services
│   ├── can-bus/               # virtual CAN, SocketCAN on the Pi, QCanBusDevice
│   ├── yocto/                 # custom embedded image, SSH, Qt6
│   ├── freertos/              # porting FreeRTOS to STM32F103
│   ├── unit-testing/          # GoogleTest
│   ├── dbus/                  # IPC: signals, properties, custom/enum types, policy
│   ├── network/               # wpa_supplicant, nmcli, static IP, SIM900
│   ├── sim-module/            # SIM900 hardware, socat, HTTP GET/POST  (+ img/)
│   ├── car-hardware/          # encoder, IR over CAN
│   ├── gui/                   # Qt/QML dashboard notes
│   └── roadmap.md
└── site/                      # build output (generated, git-ignored)
```

---

## How this site was built

The source material was a flat **Obsidian vault**: ~50 loose Markdown files in
numbered folders (`01-Server/`, `03-Handle-Can/`, …) plus a shared `Images/`
folder, written as personal notes rather than as a published site. Turning it
into this site took four passes.

### 1. Restructure the content

Every note was moved into a topic folder under `docs/` and given a descriptive,
URL-friendly filename (`03-Handle-Can/01-Virtual-Can.md` → `can-bus/virtual-can.md`).
Files that had no `#` heading got one derived from their nav title, so each page
renders with a proper `<h1>` and a sensible browser tab title.

Images moved next to the pages that use them (`docs/sim-module/img/`,
`docs/server/img/`) so each section stays self-contained.

### 2. Fix what Obsidian tolerated but the web doesn't

Obsidian silently accepts things a real Markdown build does not. Fixed:

- **Obsidian wiki-embeds** — `![[photo.jpg]]` → standard `![alt](img/photo.jpg)`.
- **Broken image paths and casing** — `../images/...` pointed at a folder that
  didn't exist at that path, and `09-SIM-Antenna.PNG` didn't match the actual
  `.png` on disk. Harmless on case-insensitive setups, a 404 on a real web server.
- **~44 dead in-page anchors** — hand-written tables of contents linked to
  `#Step-1-Scanning-Available-Networks`, but the generated slug is lowercase and
  strips punctuation. A script re-slugified every `](#…)` link to match how the
  `toc` extension actually generates IDs.
- **Phantom `---` headings** — two `---` lines in a row make the second one
  *underline* the first, producing a setext heading literally titled `---` that
  showed up in the sidebar table of contents.

### 3. Fix Markdown that rendered wrong

These were the real bugs, and they are **invisible when reading the raw file** —
they only appear once rendered:

- **`1)` style lists.** `1)` is not Markdown list syntax. Whole step-by-step
  guides (Yocto custom image, Qt6 integration, STM32 build steps) were collapsing
  into a single run-on paragraph. Rewritten as real `1.` lists with the code
  blocks indented *inside* each step.
- **Lists glued to the paragraph above them.** Without a blank line separating
  them, the list is absorbed into the preceding paragraph — same collapse.
- **Stray leading tabs.** A tab at the start of a line makes Markdown treat it as
  an indented code block. This silently turned an intro paragraph into a code
  block on the CAN page, and swallowed a heading entirely on the Raspberry Pi page.

Because none of these are visible in a diff, **every page that was edited was
verified by rendering it in a real browser and checking the screenshot** — not by
re-reading the Markdown.

### 4. Theme and layout

`mkdocs.yml` configures Material with dark mode as the default (light-mode toggle
in the header), instant search with suggestions and highlighting, copy buttons on
code blocks, and image lightboxes via `glightbox`. The navigation is written out
explicitly so section and page ordering is deliberate rather than alphabetical,
and section labels are kept short so all 14 tabs fit in the header bar without
being clipped.

`docs/index.md` is a purpose-built homepage — a hero banner, a tech-stack chip
row, a **Mermaid** diagram of the end-to-end architecture (subsystems → CAN →
gateway → D-Bus/GSM → cloud), and a card grid linking into each section.

`docs/assets/stylesheets/extra.css` overrides Material's default `61rem` cap so
the header, tabs, and content span the **full viewport width**, with gutters and
side-rail widths that grow at wider breakpoints. Long-form prose keeps a readable
measure on very wide screens while tables, code blocks, and diagrams stay
full-bleed.

---

## Verification

```bash
mkdocs build --strict     # fails on broken internal links, bad nav, missing files
```

`--strict` passes with **zero warnings**: every nav entry resolves, every internal
link and anchor points at something real, and every referenced image exists.
Layout was verified by rendering pages headlessly at 390px (mobile), 1280px,
1366px, 1920px, and 2560px.
