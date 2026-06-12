---
schema: 1
type: note
name: "Notion meeting dashboard "
tagline: "Meeting Notes Minimal dashboard"
description: |
  A minimal, two-panel meeting notes dashboard for Obsidian sidebar panes. Combines:
  1. Upcoming Events  a collapsible list of tasks/events due within the next N days, color-coded by priority.
  2. Audio Manager � a collapsible, time-bucketed list of every audio recording in your vault, with one-click playback, rename, delete, and "create linked note" actions.
  
  Built with Datacore JSX. No extra plugins beyond Datacore are required (though the Audio Recorder core plugin is needed to actually make recordings).
  
  ---
  
  What it does
  
  Upcoming Events
  
  - Scans configured source folders for Markdown notes that have a due, scheduled, or start frontmatter date
  - Filters out notes whose status is done, completed, or cancelled
  - Shows only events within the next 14 days (configurable)
  - Color-codes each event dot by priority frontmatter (high ? pink, medium ? orange, low ? blue) or a custom color field
  - Clicking an event opens the note
  - Reactively refreshes on any vault change
  
  Audio Manager
  
  - Finds all audio files in the vault (.mp3, .wav, .m4a, .webm, .ogg)
  - Groups them into Today, Yesterday, This week, Last week, and monthly buckets
  - Each row shows whether a linked Markdown note exists (note icon) or not (waveform icon)
  - Hover actions:
    - Note icon open existing linked note, or create a new meeting note with the recording embedded
    - Rename triggers Obsidian's built-in rename prompt
    - Delete moves to system trash with confirmation
  - New notes are created in a configurable folder with frontmatter pre-filled
  
  Installation
  
  1. Install Datacore from Community Plugins.
  2. Copy snippets/upcoming-events.css ? .obsidian/snippets/.
  3. Also copy audio-manager.css (from the Notion AI Dashboard release, or from the audio-manager release if published separately) ? .obsidian/snippets/.
  4. In Settings ? Appearance ? CSS Snippets, enable both snippets.
  5. Copy Meeting Notes Minimal.md anywhere in your vault (works best pinned as a sidebar pane).
  6. Open the note.
  
  ---
  
  Configuration
  
  All settings live at the top of the first datacorejsx block:
  
  ```js
  // How many days ahead the "Upcoming" list looks.
  const DAYS_AHEAD = 14;
  
  // Folders to scan for task/event notes.
  // Any note whose path starts with one of these prefixes is included.
  const SOURCE_PREFIXES = ["Tasks/"];
  
  // Colors per priority level (frontmatter: priority: high/medium/low).
  // A color frontmatter field on the note overrides this.
  const PRIORITY_COLORS = { high: "pink", medium: "orange", low: "blue" };
  
  // Frontmatter status values that hide a note from the list.
  const DONE_STATUSES = new Set(["done", "completed", "cancelled"]);
  ```
  
  Meeting note folder
  
  In the Audio Manager block, new linked notes are created in:
  
  ```js
  const folder = "Meeting Notes";
  ```
  
  Change this to wherever you keep meeting notes.
  
  ---
  
  Frontmatter expected on task/event notes
  
  ```yaml
  due: 2026-06-20          # or scheduled: or start:
  status: active           # omit or set to done/completed/cancelled to hide
  priority: high           # high | medium | low  (defaults to medium)
  color: blue              # optional override for the dot color
  title: "Sprint planning" # optional display title (falls back to filename)
  ```
  
  ---
  
  CSS snippets
  
  upcoming-events.css
  
  Styles the collapsible Upcoming section, event rows, color dots, and the task creation modal (if you extend it). All selectors are scoped to .upcoming-events / .ue-*.
  
  Supported dot colors via data-color attribute:
  
  | Value | Color |
  |-------|-------|
  | pink | Coral red |
  | red | Red |
  | orange | Amber |
  | blue | Steel blue |
  | green | Forest green |
  | purple | Soft lavender (default) |
  
  audio-manager.css
  
  Styles the Audio Manager list, time-bucket headers, hover action buttons, and the shared .am-* classes also used by the Notion AI Dashboard.
  
  
  
  License
  
  MIT use freely, modify, share.
author: Maws7140
categories:
  - dashboard
  - tracker
  - query
tags:
  - Notion
  - dashboard
  - meeting
screenshots:
  - https://i.imgur.com/dZst9IK.png
attached_snippets:
  - path: "snippets/audio-manager.css"
    name: "audio-manager"
  - path: "snippets/upcoming-events.css"
    name: "upcoming-events"
environment:
  obsidian_version: "unknown"
  theme: "Minimal"
  os: "Windows"
files:
  - path: "Releases/obsidianvaulthub/meeting-notes-minimal/Meeting Notes Minimal.md"
    type: md
    size: 14985
---

# Notion meeting dashboard 

Meeting Notes Minimal dashboard

## Description

A minimal, two-panel meeting notes dashboard for Obsidian sidebar panes. Combines:
1. Upcoming Events  a collapsible list of tasks/events due within the next N days, color-coded by priority.
2. Audio Manager � a collapsible, time-bucketed list of every audio recording in your vault, with one-click playback, rename, delete, and "create linked note" actions.

Built with Datacore JSX. No extra plugins beyond Datacore are required (though the Audio Recorder core plugin is needed to actually make recordings).

---

What it does

Upcoming Events

- Scans configured source folders for Markdown notes that have a due, scheduled, or start frontmatter date
- Filters out notes whose status is done, completed, or cancelled
- Shows only events within the next 14 days (configurable)
- Color-codes each event dot by priority frontmatter (high ? pink, medium ? orange, low ? blue) or a custom color field
- Clicking an event opens the note
- Reactively refreshes on any vault change

Audio Manager

- Finds all audio files in the vault (.mp3, .wav, .m4a, .webm, .ogg)
- Groups them into Today, Yesterday, This week, Last week, and monthly buckets
- Each row shows whether a linked Markdown note exists (note icon) or not (waveform icon)
- Hover actions:
  - Note icon open existing linked note, or create a new meeting note with the recording embedded
  - Rename triggers Obsidian's built-in rename prompt
  - Delete moves to system trash with confirmation
- New notes are created in a configurable folder with frontmatter pre-filled

Installation

1. Install Datacore from Community Plugins.
2. Copy snippets/upcoming-events.css ? .obsidian/snippets/.
3. Also copy audio-manager.css (from the Notion AI Dashboard release, or from the audio-manager release if published separately) ? .obsidian/snippets/.
4. In Settings ? Appearance ? CSS Snippets, enable both snippets.
5. Copy Meeting Notes Minimal.md anywhere in your vault (works best pinned as a sidebar pane).
6. Open the note.

---

Configuration

All settings live at the top of the first datacorejsx block:

```js
// How many days ahead the "Upcoming" list looks.
const DAYS_AHEAD = 14;

// Folders to scan for task/event notes.
// Any note whose path starts with one of these prefixes is included.
const SOURCE_PREFIXES = ["Tasks/"];

// Colors per priority level (frontmatter: priority: high/medium/low).
// A color frontmatter field on the note overrides this.
const PRIORITY_COLORS = { high: "pink", medium: "orange", low: "blue" };

// Frontmatter status values that hide a note from the list.
const DONE_STATUSES = new Set(["done", "completed", "cancelled"]);
```

Meeting note folder

In the Audio Manager block, new linked notes are created in:

```js
const folder = "Meeting Notes";
```

Change this to wherever you keep meeting notes.

---

Frontmatter expected on task/event notes

```yaml
due: 2026-06-20          # or scheduled: or start:
status: active           # omit or set to done/completed/cancelled to hide
priority: high           # high | medium | low  (defaults to medium)
color: blue              # optional override for the dot color
title: "Sprint planning" # optional display title (falls back to filename)
```

---

CSS snippets

upcoming-events.css

Styles the collapsible Upcoming section, event rows, color dots, and the task creation modal (if you extend it). All selectors are scoped to .upcoming-events / .ue-*.

Supported dot colors via data-color attribute:

| Value | Color |
|-------|-------|
| pink | Coral red |
| red | Red |
| orange | Amber |
| blue | Steel blue |
| green | Forest green |
| purple | Soft lavender (default) |

audio-manager.css

Styles the Audio Manager list, time-bucket headers, hover action buttons, and the shared .am-* classes also used by the Notion AI Dashboard.



License

MIT use freely, modify, share.

## Attached Snippets

- `snippets/audio-manager.css` - audio-manager
- `snippets/upcoming-events.css` - upcoming-events

## Screenshots

![Screenshot 1](https://i.imgur.com/dZst9IK.png)
## Installation

1. Download the `.md` file(s) from this repo
2. Place them in your vault
3. Copy the attached CSS snippet files into your vault's CSS snippets folder
4. Enable them in Settings > Appearance > CSS Snippets

---
*Published via [Vault Hub](https://obsidianvaulthub.com)*
