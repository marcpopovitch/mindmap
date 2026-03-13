# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-page interactive mindmap application for visualizing banking "extras" organized by consumer profiles. The entire app lives in one file: `index.html` — no build step, no bundler, no package manager.

## Architecture

- **Single-file app**: All HTML, CSS, and JavaScript are in `index.html`. The language is French.
- **Layout**: CSS Grid with 3 columns — left profiles, center node, right profiles. Each profile is a collapsible `<details>` element ("bubble") containing categorized item lists.
- **Pan/Zoom**: Custom viewport with mouse drag panning, wheel/trackpad scrolling, and zoom buttons. Transform applied to a `.stage` wrapper.
- **SVG Connectors**: Dynamically drawn cubic Bézier curves connecting the center node to each bubble, recalculated on resize/toggle/zoom via `drawConnectors()`.
- **Firebase Sync**: Uses Firebase JS SDK (ESM imports from CDN) for real-time cloud sync. Auth is anonymous. State is stored in Firestore at `artifacts/{appId}/public/data/mindmap/state2` as two arrays: `deletedIds` and `addedItems`.
- **Dual environment**: Detects Canvas environment (`__firebase_config`, `__initial_auth_token`) vs GitHub Pages deployment (hardcoded config fallback).
- **Undo/Redo**: In-memory history stack tracking `deletedIds` and `addedItems` snapshots, synced back to Firestore on each undo/redo action.

## Development

Open `index.html` directly in a browser — no server required. For Firebase sync to work, the page must be served over HTTP(S) (e.g., `python3 -m http.server`).

## Key Data Flow

1. Items are identified by base64-encoded IDs derived from their text content
2. Deletions add IDs to `deletedIds`; additions push objects to `addedItems`
3. `applySyncUI()` hides deleted items and appends custom items to the DOM
4. Firestore `onSnapshot` listener keeps all connected clients in sync
