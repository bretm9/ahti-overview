# Ahti - a writing app for Mac, iPad, and iPhone

> Ahti is an app for writing and organizing long documents. It has a binder, corkboard, outliner, rich-text editor, snapshots, collections, and writing targets, and can export manuscripts as ePub, Word, or PDF files. Projects are stored as folders of Markdown files. Ahti is available on macOS, iPad, and iPhone, with a browser version built from the same codebase.

[**Mac App Store**](https://apps.apple.com/us/app/ahti/id6781643515?platform=mac) · [**iPhone & iPad**](https://apps.apple.com/us/app/ahti/id6781643515?platform=iphone) · [Support](https://www.bretmerritt.com/ahti) · [Documentation](https://www.bretmerritt.com/ahti/documentation) · [Privacy policy](https://www.bretmerritt.com/ahti/privacy-policy)

This repository describes Ahti's features, implementation, and releases. The source code is private.

<p align="center">
  <img src="screenshots/mac-01-promo.png" width="900" alt="Ahti on macOS">
</p>

---

## Contents
1. [Overview](#overview)
2. [Screenshots](#screenshots)
3. [Features](#features)
4. [Technologies](#technologies)
5. [Architecture](#architecture)
6. [Implementation notes](#implementation-notes)
7. [Releases](#releases)
8. [Development](#development)

---

## Overview

Ahti is designed for novels, screenplays, research, and other long documents. Its Markdown storage, `[[wiki-links]]`, and document previews are inspired by Obsidian.

Each project is a folder on disk. Documents are `NN Title.md` files with YAML frontmatter and a Markdown body. Folder metadata lives in `_index.md`, and labels, statuses, collections, and targets are stored in `.ahti/project.json`. The desktop app watches for changes, so edits made in another Markdown editor or through Finder appear in Ahti. The save engine only rewrites files it created and does not delete files the user added separately.

```
The Salt Light/                       ← your vault folder
├── Manuscript/
│   ├── 01 Chapter One/
│   │   ├── _index.md                 ← the folder's synopsis/label/status
│   │   ├── 01 The Long Road.md       ← frontmatter + your prose
│   │   └── 02 The Keeper.md
│   └── 02 Chapter Two/ …
├── Research/   ·   Trash/
└── .ahti/
    ├── project.json                  ← labels, statuses, collections, targets
    └── snippets/*.css                ← optional per-vault CSS
```

**Privacy:** Ahti collects no user data, requires no account, and works offline. Writing stays in files on the user's device, with optional sync through their iCloud account.

---

## Screenshots

### macOS
| Editor | Corkboard |
|---|---|
| ![Editor](screenshots/mac-02-editor.png) | ![Corkboard](screenshots/mac-03-corkboard.png) |

| Composition mode |
|---|
| ![Composition mode](screenshots/mac-04-composition.png) |

### iPad & iPhone
<p>
  <img src="screenshots/ipad-02-editor.png" width="640" alt="Ahti on iPad">
</p>
<p>
  <img src="screenshots/iphone-01-promo.png" width="200" alt="iPhone">
  <img src="screenshots/iphone-02-editor.png" width="200" alt="iPhone editor">
  <img src="screenshots/iphone-03-light-dark.png" width="200" alt="Light and dark">
  <img src="screenshots/iphone-04-custom-themes.png" width="200" alt="Custom themes">
</p>

### Browser build (same codebase, IndexedDB-backed)
| Outliner | Corkboard |
|---|---|
| ![Outliner](screenshots/web-outliner.png) | ![Corkboard](screenshots/web-corkboard.png) |

---

## Features

### Organize
- **Binder:** A drag-and-drop document tree with Manuscript, Research, and Trash roots. It supports custom icons, labels, statuses, inline renaming, multi-select, copy/paste/duplicate, grouping, sorting, and imports from files or OPML, including "import & split".
- **Collections:** Manual lists and saved searches shown as colored tabs. A collection can be read continuously or viewed as a corkboard or outliner.
- **Document links:** `[[wiki-links]]` with autocomplete, optional title autolinking that leaves the source text unchanged, and editable previews of linked documents on hover.
- **Search and navigation:** Project-wide search with case, whole-word, and regex options; Quick Open (⌘P); keyboard tree navigation; and Back/Forward history for view changes.

### Write
- **Rich-text editor** (TipTap / ProseMirror): Inline formatting, highlights, text color, headings, lists, quotes, code, page breaks, tables, images, alignment, indentation, and spacing.
- **Comments & footnotes** anchored to ranges, editable in the inspector, convertible to one another, collected at compile.
- **Named styles**, smart typography and text transforms, find-by-formatting, linguistic focus (tint a part of speech), and **focus mode** (dim everything but the current sentence/paragraph).
- **Continuous editing** of a whole folder as one manuscript, a **composition mode** for distraction-free writing, a **split editor**, and find/replace (document or project-wide).

### Plan
- **Corkboard:** Index cards with synopses, label colors, status stamps, numbering, and images, in a grid or freeform layout.
- **Outliner:** A table of manuscript sections with configurable columns that can be reordered, resized, and mostly edited inline.
- **Inspector:** Synopsis, notes, formatting, characters, bookmarks, metadata, keywords, snapshots, comments, and footnotes.
- **Character sheets:** Add characters to scenes from templates, including TTRPG sheets, view their stats, and roll dice with an optional 3D display.

### Track & finish
- **Snapshots:** Save versions of documents, then browse, compare, restore, or rename them across the project.
- **Writing targets:** Manuscript and daily goals, progress tracking, pace suggestions based on a deadline, a writing-history heatmap, and project statistics.
- **Compile:** Export a folder or collection as ePub, DOCX, RTF, HTML, Markdown, plain text, or print/PDF. A live preview shows the title page, front and back matter, section layouts, chapter numbering, and table of contents. Text replacements, ebook metadata, and reusable presets are also supported.

### Customize
- Tabbed settings (theme, accent, fonts, widths, formatting defaults, auto-backups), a metadata manager (labels, statuses, typed custom fields), per-vault **CSS snippets**, and per-area undo/redo routed by focus (editor, binder, inspector).

### On iOS
The iOS version uses touch-controlled drawers for the binder and inspector, a fixed viewport, a custom keyboard accessory, and a formatting bar above the keyboard. It also supports press-and-hold reordering and vault sync through iCloud.

---

## Technologies

React 18 · TypeScript · Vite · **Tauri 2 (Rust)** for macOS + iOS · Tailwind CSS · Zustand (immer) · TipTap / ProseMirror · dnd-kit · `notify` (file watching) · marked + js-yaml (Markdown/frontmatter) · localforage / IndexedDB (browser build) · `@3d-dice/dice-box` (WASM) · Objective-C++ (iOS shell) · objc2 Rust FFI (iCloud, security-scoped bookmarks) · Vitest · App Store Connect / TestFlight / Transporter

---

## Architecture

A Zustand store holds the current `Project`, and components use it to read and update project data. The `lib/` directory contains tested helpers for selectors, word counts, compilation, persistence, and sample data. The macOS and iOS apps read and write Markdown files through Rust commands and watch for external file changes. The browser version stores projects in IndexedDB and exports them as `.ahti.json` files.

Comments and footnotes use editor marks. On save, those marks are removed from the Markdown body and stored as character-offset ranges in frontmatter. They are restored when the document loads, keeping the Markdown readable in other editors.

---

## Implementation notes

- **Mac App Store sandbox:** Vault access uses security-scoped bookmarks created in Rust and resolved when the app relaunches. I fixed Transporter validation errors involving the app category, a missing `application-identifier` during manual signing, and a quarantine attribute included in the bundle. I also traced a blank window in the sandboxed build to WKWebView helper processes requiring the `network.client` entitlement, even though Ahti makes no network calls.
- **Window closing during App Review:** Reviewers encountered a broken close button on a fresh installation's welcome screen. The close handler called Tauri's `destroy` command without the required capability. I fixed the capability and verified the close button through accessibility automation.
- **iOS keyboard handling:** WKWebView's `interactive-widget=resizes-content` did not behave consistently. The app instead fixes its root to the visual viewport and uses Objective-C++ to disable the outer scroll view and keep its offset at zero. A native bridge sends keyboard height and animation duration to JavaScript so the caret moves with the keyboard. A background-task guard allows pending saves to finish when the app enters the background.
- **Focus-mode animation:** ProseMirror creates dimmed blocks in the same update as their decorations, leaving no initial frame for a CSS transition. I replaced the transition with a keyframe animation and verified it in WebKit using Playwright.
- **Long manuscripts:** Continuous editing uses virtualization above 30 scenes, rendering off-screen scenes as matching static HTML. On a 220-scene manuscript, this reduced live editors by roughly 98% and cut heap use in half.
- **Reliability fixes:** Two review rounds identified 283 bugs that were verified and fixed. These included browser dialogs (`alert`, `confirm`, and `prompt`) and `<a download>` not working in the WebView. I replaced them with in-app dialogs and native save/open handling. Vault writes now use a temporary file and rename, covered by Rust tests. I also added a Content Security Policy, a single-instance lock, and recovery when the WebView process exits.

---

## Releases

- First commit on **June 15, 2026**, with **v1.1.1 released on July 6, 2026** (157 commits in three weeks).
- ~43k lines of TypeScript across ~200 files, plus a Rust layer; **530 unit tests (Vitest) + Rust tests**.
- First signed TestFlight build on June 18 and version 1.0.0 submitted on June 25. iOS was approved; the macOS rejection was fixed in 1.0.1 on July 1. Version 1.1.0 released on both stores on July 5. Both use one App Store Connect record with Universal Purchase (`com.ahti.app`).
- Support, privacy, and documentation site built and deployed for App Review.

---

## Development

I'm the sole developer and use Claude Code for coding assistance. I chose the file format and architecture, reviewed and tested the code on real devices, and handled signing, entitlements, App Review, and releases.

---

*Bret Merritt · [GitHub](https://github.com/bretm9) · [LinkedIn](https://www.linkedin.com/in/bret-merritt) · merrittbret9@gmail.com*
