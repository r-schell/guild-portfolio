# Bookmark Manager — Design Document

## Purpose

A personal, single-user web app for saving and organizing bookmarks. Runs entirely in the browser with no backend or login required. Data is stored locally using the browser's `localStorage` API.

---

## Features

### Core

1. Add a bookmark: URL, title, optional notes, optional tags
2. View all bookmarks in a list with the newest ones first
3. Update title, URL, notes, or tags of any saved bookmark
4. Delete a bookmark
5. Click a tag to filter the list to bookmarks with that tag
6. Search bookmarks by title or note content

### Stretch goals

1. Sort bookmarks by date or alphabetically by title
2. Copy URL to clipboard with a single click
3. Mark bookmarks as favorites, indicated by an icon on each entry

---

## Technology

| Layer | Choice |
|---|---|
| Markup | HTML5 |
| Styles | Plain CSS |
| Logic | Vanilla JavaScript (ES6+) |
| Storage | `localStorage` |
| ID generation | `crypto.randomUUID()` |

Single file: `index.html`, with CSS in a `<style>` tag and JavaScript in a `<script>` tag. No build tools, no `npm install`, no bundler.

---

## Interface

- Max-width of 960 px, centered, with side borders to contain the interface on wide screens
- Header with app title and an "Add Bookmark" button
- Search bar below the header
- Two-column layout: a tag list on the left, bookmark cards on the right
- Each card shows the title, URL, notes, and tags, with Edit and Delete buttons
- Adding a bookmark expands an inline form at the top of the card list; editing a bookmark expands an inline form in place of that card
- On narrow screens the tag list stacks above the cards
- Retro visual style: monospace font, muted warm or cool palette (e.g. off-white background, soft greens or ambers), flat UI with hard borders instead of shadows or rounded corners, minimal decoration

---

## Out of Scope

- User accounts or login
- Sync across devices or browsers
- Import or export of bookmarks
- Undo for deletes
