# Process Library Feature Design

**Date:** 2026-06-02

## Goal

Allow users to save dyeing processes to a shared library (Google Sheets) and load any saved process by selecting from a dropdown. Access to save is password-protected; loading is open to all.

## Architecture

The tool remains a static single-page HTML application. Google Sheets acts as the database via the existing Apps Script web app. A new sheet `染程庫` stores all saved processes. Apps Script gains a `doGet()` endpoint for reading and a new `saveToLibrary` handler in `doPost()` for writing. The password is validated server-side in Apps Script only — it is never stored in the HTML.

## Google Sheets Data Structure

Sheet name: `染程庫` (created in the existing Google Spreadsheet)

| Column A: 染程名稱 | Column B: 資料 (JSON string) |
|---|---|
| 聚酯-標準 | `{"stages":[{"s":30,"e":30,"t":10},...], "actions":[{"temp":30,"label":"加入助劑"},...]}` |
| 棉-深色 | `{"stages":[...],"actions":[...]}` |

- Each row = one process
- Written by the tool via Apps Script (users never edit this sheet directly)
- Saving a process with a name that already exists overwrites that row

## Apps Script Changes

### New: `doGet(e)`

Two endpoints:

**List all process names:**
```
GET ?action=list
→ { "names": ["聚酯-標準", "棉-深色", ...] }
```

**Get one process by name:**
```
GET ?action=get&name=聚酯-標準
→ { "stages": [...], "actions": [...] }
```

Both return JSON with `Content-Type: application/json`.

### Modified: `doPost(e)` — new action

Existing upload behavior is unchanged. New action:

```
POST { action: "saveToLibrary", password: "xxx", name: "染程名稱", stages: [...], actions: [...] }
→ success: { ok: true }
→ failure: { ok: false, error: "wrong password" }
```

Password is a constant defined at the top of the Apps Script file. If the name already exists in `染程庫`, that row is overwritten.

## Tool UI Changes

### New: Library Bar (top of page)

```
┌─────────────────────────────────────────────┐
│  染程庫  [下拉選單 ▼]  [載入]                  │
└─────────────────────────────────────────────┘
```

- On page load: fetch `?action=list` and populate the dropdown
- [載入] click: fetch `?action=get&name=...`, populate table/actions, enter View Mode

### View Mode (loaded from library)

- All stage table inputs: `disabled`
- All action inputs: `disabled`
- 新增/刪除階段 buttons: hidden
- Process name field: shows library name, disabled
- A **[新建染程]** button appears at top → clears all fields, re-enables inputs, returns to Design Mode

### Design Mode (default)

- All existing behaviour unchanged
- New **[存入染程庫]** button in the export/action area
- On click: prompt for process name (pre-filled with current 染程名稱 value) → prompt for password → POST `saveToLibrary` to Apps Script
- Success: show "✓ 已存入染程庫" message; reload dropdown
- Wrong password: show "✗ 密碼錯誤" message
- Network error: show "✗ 儲存失敗" message

## State Machine

```
Design Mode (default)
  → user clicks [載入] with a selection → View Mode
  → user clicks [存入染程庫] → password prompt → saves → stays in Design Mode

View Mode
  → user clicks [新建染程] → Design Mode (cleared)
```

## Error Handling

| Situation | Behaviour |
|---|---|
| Dropdown fetch fails on load | Show "無法載入染程庫" next to dropdown |
| Process not found | Show "找不到染程" |
| Wrong password | Show "✗ 密碼錯誤" inline |
| Save network error | Show "✗ 儲存失敗，請重試" inline |

## Out of Scope

- Deleting processes from the library (done directly in Google Sheets)
- Renaming processes (done directly in Google Sheets)
- Per-user authentication (password is shared among authorized users)
- Editing a loaded process and saving changes back
