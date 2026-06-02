# Tab UI Design — 檢視／寫入 Mode Switch

**Date:** 2026-06-02

## Goal

Replace the current library bar with a two-tab interface (檢視／寫入) at the top of the page, making the mode the user is in visually explicit and the switch action direct.

## Architecture

This is a pure UI refactor of the existing library bar in `index.html`. No changes to Apps Script or data flow. The existing `enterViewMode()`, `enterDesignMode()`, `loadProcess()`, and `fetchLibraryList()` functions are retained and slightly adjusted to sync the active tab highlight. A new `switchTab(tab)` function handles tab switching.

## HTML Structure

The existing `.library-bar` div is replaced by a `.tab-bar` div:

```
┌────────────────────────────────────────────────────────┐
│  [檢視]  [寫入]  │  下拉選單 ▼  [載入]  [檢視模式 badge] │
└────────────────────────────────────────────────────────┘
```

- Left side: two tab buttons `#tabView` (檢視) and `#tabWrite` (寫入)
- Right side (`#tabViewPanel`): library controls — `#librarySelect`, `#loadBtn`, `#viewBadge`, `#libraryStatus`
- `#tabViewPanel` is hidden in 寫入 mode, visible in 檢視 mode
- The `#newProcessBtn` button is removed (clicking 寫入 replaces it)

## CSS Changes

- `.tab-bar` — same white card style as current `.library-bar`
- `.tab-btn` — default tab style (grey background)
- `.tab-btn.tab-active` — active tab style (blue `#4a7c9e` background, white text)
- `.tab-divider` — a thin vertical separator between tabs and panel content
- Remove `.library-bar` CSS (replaced by `.tab-bar`)
- Remove `.btn-new-process` CSS (button removed)

## Behaviour

### Default state on page load
- 寫入 tab is active
- Library panel is hidden
- Table is editable (same as current behaviour)
- `fetchLibraryList()` is still called on init (populates dropdown in background)

### Clicking 檢視 tab
- `#tabView` gets `.tab-active` class; `#tabWrite` loses it
- `#tabViewPanel` becomes visible
- Table state is unchanged (does not clear or disable)

### Clicking 載入 (inside 檢視 panel)
- `loadProcess(name)` is called
- Table fills with selected process data
- `enterViewMode()` is called → inputs disabled, add/delete/save buttons hidden

### Clicking 寫入 tab
- `#tabWrite` gets `.tab-active` class; `#tabView` loses it
- `#tabViewPanel` is hidden
- `enterDesignMode()` is called → table cleared, inputs enabled, buttons shown

## JS Changes

### New function: `switchTab(tab)`
```javascript
function switchTab(tab) {
  const isView = tab === 'view';
  const wasView = document.getElementById('tabView').classList.contains('tab-active');
  document.getElementById('tabView').classList.toggle('tab-active', isView);
  document.getElementById('tabWrite').classList.toggle('tab-active', !isView);
  document.getElementById('tabViewPanel').style.display = isView ? '' : 'none';
  // Only call enterDesignMode when switching FROM 檢視 TO 寫入,
  // not on initial page load (when neither tab has tab-active yet).
  if (!isView && wasView) enterDesignMode();
}
```

On init: `switchTab('write')` is called before either tab has `.tab-active`, so `wasView` is false and `enterDesignMode()` is not triggered. The table retains its default content.

### Updated `enterViewMode()`
- Add: `document.getElementById('tabView').classList.add('tab-active'); document.getElementById('tabWrite').classList.remove('tab-active');`
- Remove: `document.getElementById('newProcessBtn').style.display = ''` (button is removed)
- Remove: `document.getElementById('viewBadge').style.display = ''` — badge stays managed inside `#tabViewPanel`

### Updated `enterDesignMode()`
- Remove: `document.getElementById('newProcessBtn').style.display = 'none'`
- Remove: `document.getElementById('viewBadge').style.display = 'none'`
- The badge hides naturally when `#tabViewPanel` is hidden

### DOMContentLoaded
- Replace `#loadBtn` listener (already wired to `loadProcess`)
- Remove `#newProcessBtn` listener (button removed)
- Add `#tabView` and `#tabWrite` click listeners → call `switchTab('view')` / `switchTab('write')`
- Call `switchTab('write')` on init (default state)

## What Is Removed

| Element | Reason |
|---|---|
| `#newProcessBtn` button | Replaced by clicking 寫入 tab |
| `.btn-new-process` CSS | Button removed |
| `document.getElementById('viewBadge').style.display` in enterViewMode/enterDesignMode | Badge is inside `#tabViewPanel`, hidden/shown automatically |

## Out of Scope

- Remembering the last active tab across page reloads
- Showing the loaded process name in the 寫入 tab button
