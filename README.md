# notes-app-rs

An Apple Notes-style desktop app written in Rust with [egui](https://github.com/emilk/egui) / eframe.

It is not a pixel clone. The structure, menu wording and behaviour were taken from the real
Notes.app by reverse engineering it with [REA](https://github.com/morluto/rea) (Reverse Engineer
Anything) and [Ghidra](https://ghidra-sre.org/), instead of guessing.

## Features

**Organising**
- Sidebar with **All Notes**, **Notes**, your own folders and **Recently Deleted**
- New Folder from the sidebar; right-click a custom folder to **Delete Folder** (its notes move to Recently Deleted)
- `#tags` are picked up from note text and listed in the sidebar
- **Pinned** section at the top of the list
- Search across titles and bodies

**Notes**
- First line is the title, the rest is the preview; list rows show title, preview and date
- Heading markers: `# ` (Heading), `## ` (Subheading), larger text in the editor
- **Checklist** button, Duplicate Note, **Move to** folder
- Pin Note / Unpin Note, "Edited: ..." status line

**Views and sorting**
- List view and **Gallery** view (card grid), switched with the ▦ / ☰ buttons
- **Sort By** Date Edited / Date Created / Title, remembered **per folder**

**Deleting**
- Delete moves a note to Recently Deleted; Recover brings it back (to Notes if its folder is gone)
- Notes older than **30 days** in Recently Deleted are purged on launch
- Deleted notes can't be edited; Delete Forever removes them
- The "moved to Recently Deleted" alert is shown only once

**Storage**
- Plain JSON at `~/Library/Application Support/notes-app/notes.json` (macOS; the platform data dir elsewhere)
- Atomic writes (temp file + rename), so a crash can't truncate your notes
- Reads the old bare-array format as well as the current one

## How it was reverse engineered

| Step | Tool | What it gave |
|------|------|--------------|
| UI structure | `rea decode-interface-builder` on `Notes.app` | Real nibs: main window, note list cell (title / summary / date), folder list, smart folder and tags |
| Wording | `Localizable.loctable` in `Notes.app` | English strings: "Recently Deleted", "Pin Note", "Sort By", "View as Gallery", ... |
| Behaviour | `rea analyze` + `search` with Ghidra on the arm64e slice | Symbols showing per-folder sort type, one-time delete alert, trash purge and the Pin/Unpin rule |

Findings from the binary that shaped the app:

- Notes.app is a thin shell; most logic lives in private frameworks (NotesShared, NotesUI, NotesSupport).
- `ICMNoteBrowserController` has `currentCustomNoteSortType`, so sort is stored per folder.
- `ICDeleteDecision.didShowMoveToRecentlyDeletedFolderAlert` means the delete alert is shown once.
- `decisionObjectsForFolder:` means deleting a folder involves its notes.
- `markOldTrashedNotesForDeletionInContext:` and `ICPutDeletedNotesInTrashLaunchTask` handle the trash purge.

## Build and run

Requires Rust (stable, edition 2024).

```
cargo run --release
cargo test
```

## Project layout

```
src/main.rs   model (Note, Store), load / persist / purge, sidebar, list, gallery, editor
Cargo.toml    eframe, serde, serde_json, dirs
```

## Not implemented

Checklist toggling on click, locked notes, sharing, smart folders, attachments and tables,
date grouping (Today / Previous 7 Days; no evidence for it in the binary yet), iCloud sync.

## Disclaimer

Independent project, not affiliated with Apple. Notes and Apple Notes are Apple trademarks.
No Apple code or assets are included; only observed behaviour and UI wording informed the design.
