# Critique — bookmark-manager/index.html

---

## Bugs

**1. `.page` div is never closed — line 330 vs 357**
`<div class="page">` opens at line 330. The comment `</div><!-- .page -->` at line 357 is wrong — that `</div>` closes `#main` (which opened at line 347). `.page` is left open and the browser silently auto-closes it before `</body>`. The `<script>` block ends up outside `.page` in the intended structure.

**2. `activeTag` goes stale after deleting the last bookmark with a tag**
If you delete the last bookmark that carries a tag while that tag is active, `activeTag` still holds the tag name. The sidebar button for it disappears (so there's nothing to click to clear it), but `filtered()` keeps filtering by it — permanently showing an empty list. You're stuck until you reload.

**3. `type="search"` clear button may not trigger search update in Firefox**
The native ×-clear button on `<input type="search">` fires an `input` event in Chrome but a `search` event in Firefox. Only `input` is wired up (line 610), so clicking the native clear button may leave a stale filtered list in Firefox.

**4. `localStorage` corruption silently destroys all data**
`load()` catches any `JSON.parse` error and returns `[]`. If the stored data is ever corrupted (e.g., partial write), every subsequent load silently overwrites it with an empty array on the next save. No warning is shown.

---

## Security

**5. Stored XSS via `javascript:` URLs**
`esc()` (lines 595–601) only escapes `&`, `<`, `>`, `"`. It does nothing to block `javascript:` scheme URLs. A bookmark saved with URL `javascript:alert(document.cookie)` passes through unchanged and renders as a live `<a href="javascript:...">`. Clicking it executes arbitrary JS. This is a stored XSS vulnerability.

**6. No URL validation — browser validation is bypassed**
The form uses `<input type="url">` which *would* invoke browser-native URL validation — but only on `<form>` submit. The save button is a plain `<button>` inside a `<div>`, not a `<form>`, so the browser never validates. The only check is `!url` (line 503), which accepts any non-empty string including `javascript:...` or `not a url at all`.

**7. `navigator.clipboard` fails silently in non-secure contexts**
`navigator.clipboard.writeText()` (line 564) only exists on HTTPS or `localhost`. On plain HTTP, `navigator.clipboard` is `undefined` and the call throws a `TypeError` with no catch and no fallback. The copy button just silently does nothing.

---

## Poor Practice

**8. Duplicate DOM IDs**
`buildForm` always stamps `f-title`, `f-url`, `f-notes`, `f-tags`, `f-save`, `f-cancel` into the document. The logic prevents two forms from being open simultaneously, so it happens to work — but IDs must be unique per document. The `<label for="f-title">` association would bind to the wrong element if two forms ever coexisted. Since `wrap.querySelector` is already used to read values, the IDs serve no purpose that a `name` attribute or a class couldn't fill.

**9. `filtered()` reads state from the DOM instead of variables**
Lines 390–391: `document.getElementById('search').value` and `document.getElementById('sort-select').value`. Sort order and search query are app state — they should be stored in JS variables alongside `activeTag`, `editingId`, etc. Reading them from the DOM couples logic to element IDs and makes the function untestable in isolation.

**10. Textarea value set via `innerHTML`**
Line 485: the notes textarea's content is injected via template literal into `innerHTML`. It works because `esc()` handles the dangerous characters, but setting a textarea's initial value via `.innerHTML` is fragile — the correct approach is to set `.value` after creation.

**11. `alert()` and `confirm()` for validation and delete**
Lines 504, 585. These are blocking browser dialogs that look foreign in any styled app, can be suppressed by popup-blocking settings, and are disabled entirely in some embedded contexts (iframes, certain mobile browsers).

**12. `.btn-add` duplicates `.btn-primary` exactly**
Both classes declare identical rules: `background: var(--accent); color: #fff; border-color: var(--accent)` with the same hover. One can be removed.

**13. `--accent2` is declared but never used**
Line 15. The amber `#b87333` was presumably intended for something but goes unreferenced throughout the CSS.

**14. Magic number `calc(100vh - 97px)`**
Line 95. `97` is the sum of the header and toolbar heights, hardcoded. If either wraps onto a second line (e.g., on a mid-width viewport), the sidebar background stops short of the page bottom. This will silently break under resize.

---

## Inefficiencies

**15. Full DOM teardown on every interaction**
Every keystroke in the search box calls `render()` → `renderTags()` + `renderCards()`, which sets `innerHTML = ''` and rebuilds every element and event listener from scratch. A search through 100 bookmarks fires this on every character typed.

**16. `allTags()` re-scans all bookmarks on every render**
`allTags()` iterates the entire bookmarks array and all their tag arrays every time the sidebar is painted — which is every render cycle. For a personal app the cost is trivial, but it runs even when only a card's favorite state changed.

**17. Card HTML built with template literals then queried back**
`buildCard` assembles a string of HTML (line 539) containing user data, sets it with `innerHTML`, and immediately queries back into it with `querySelector` to attach listeners. Mixing string-built HTML with post-hoc DOM queries is the riskiest pattern here — if `esc()` ever has a gap, there's no second line of defence. Building the data-carrying elements with `createElement`/`textContent`/`setAttribute` would be safer and remove the dependence on `esc()` for correctness.
