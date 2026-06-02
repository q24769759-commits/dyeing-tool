# Process Library Feature Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a dropdown to load saved dyeing processes from a Google Sheets library, and a password-protected button to save the current process to that library.

**Architecture:** A `染程庫` sheet is added to the existing Google Spreadsheet. The Apps Script gains `doGet()` for reading and a new `saveToLibrary` POST handler for writing (password verified server-side). `index.html` gains a library bar UI, mode switching between "設計模式" (editable) and "檢視模式" (read-only), and the JS functions to fetch/load/save.

**Tech Stack:** Vanilla JS (async/await), Google Apps Script, Google Sheets

---

## File Map

| File | Change |
|---|---|
| Google Apps Script (manual edit at script.google.com) | Add `doGet()`, `jsonResp()`, `LIBRARY_PASSWORD`, and `saveToLibrary` block in `doPost()` |
| `index.html` — `<style>` block | Add CSS for library bar, view badge, disabled inputs |
| `index.html` — `<body>` | Add library bar HTML above `.process-meta`; add 存入染程庫 button to `.btn-row` |
| `index.html` — `<script>` block | Add `fetchLibraryList`, `populateTableFromStages`, `populateActionsFromData`, `loadProcess`, `enterViewMode`, `enterDesignMode`, `saveToLibrary`; update `DOMContentLoaded` |

---

### Task 1: Update Apps Script

**Files:**
- Modify: Google Apps Script (manual update via script.google.com)

This task requires manual editing in the Apps Script web editor. No files to push.

- [ ] **Step 1: Open Apps Script editor**

  1. Open `https://docs.google.com/spreadsheets/d/1q7e2yjphF0gnX4Wg_c5tYLmVNK8_0YnpAYMmXahMfTs/edit`
  2. Click **Extensions → Apps Script**
  3. The editor opens showing the existing `doPost(e)` function

- [ ] **Step 2: Add password constant and helper at the very top of the file**

  Add these two items before any existing function (line 1 of the file):

  ```javascript
  const LIBRARY_PASSWORD = 'change-me-123'; // Change this to your actual password

  function jsonResp(obj) {
    return ContentService.createTextOutput(JSON.stringify(obj))
      .setMimeType(ContentService.MimeType.JSON);
  }
  ```

- [ ] **Step 3: Add `doGet(e)` function**

  Add this complete function after the two items above:

  ```javascript
  function doGet(e) {
    const action = (e.parameter || {}).action;
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = ss.getSheetByName('染程庫');

    if (action === 'list') {
      if (!sheet || sheet.getLastRow() < 2) return jsonResp({ names: [] });
      const names = sheet.getRange(2, 1, sheet.getLastRow() - 1, 1)
        .getValues().map(r => r[0]).filter(n => n);
      return jsonResp({ names });
    }

    if (action === 'get') {
      const name = (e.parameter || {}).name;
      if (!sheet || sheet.getLastRow() < 2) return jsonResp({ error: 'not found' });
      const rows = sheet.getRange(2, 1, sheet.getLastRow() - 1, 2).getValues();
      for (const row of rows) {
        if (row[0] === name) {
          return ContentService.createTextOutput(row[1])
            .setMimeType(ContentService.MimeType.JSON);
        }
      }
      return jsonResp({ error: 'not found' });
    }

    return jsonResp({ error: 'unknown action' });
  }
  ```

- [ ] **Step 4: Add `saveToLibrary` block inside `doPost(e)`**

  Find the existing `doPost(e)` function. It starts with `function doPost(e) {` and likely has `const data = JSON.parse(e.postData.contents)` near the top. Add the following block immediately after the `JSON.parse` line, before any existing logic:

  ```javascript
  if (data.action === 'saveToLibrary') {
    if (data.password !== LIBRARY_PASSWORD) {
      return jsonResp({ ok: false, error: 'wrong password' });
    }
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    let lib = ss.getSheetByName('染程庫');
    if (!lib) {
      lib = ss.insertSheet('染程庫');
      lib.appendRow(['染程名稱', '資料']);
    }
    const payload = JSON.stringify({ stages: data.stages, actions: data.actions });
    const lastRow = lib.getLastRow();
    if (lastRow >= 2) {
      const existing = lib.getRange(2, 1, lastRow - 1, 1).getValues();
      for (let i = 0; i < existing.length; i++) {
        if (existing[i][0] === data.name) {
          lib.getRange(i + 2, 2).setValue(payload);
          return jsonResp({ ok: true });
        }
      }
    }
    lib.appendRow([data.name, payload]);
    return jsonResp({ ok: true });
  }
  ```

  The resulting `doPost(e)` should look like:
  ```javascript
  function doPost(e) {
    const data = JSON.parse(e.postData.contents);

    if (data.action === 'saveToLibrary') {
      // ... (block above)
    }

    // ... rest of existing doPost logic unchanged below
  }
  ```

- [ ] **Step 5: Save and redeploy**

  1. Press **Ctrl+S** to save
  2. Click **Deploy → Manage deployments**
  3. Click the pencil (edit) icon on the existing deployment
  4. Change "Version" to **New version**
  5. Click **Deploy**

- [ ] **Step 6: Test `doGet` in browser**

  Open this URL in a browser tab (use the actual deployment URL from your Apps Script):
  ```
  https://script.google.com/macros/s/AKfycbzlzZus1wHcoy_p0rGnqhLnJ8jPOUQkLySm-Q6RjnldCWqQYqpcfhF5D3ztUbCllWQ5/exec?action=list
  ```
  Expected: browser shows `{"names":[]}` (empty list — correct, no processes saved yet).

  If you see an error, re-check Steps 2–5 and verify the deployment has "Who has access: Anyone".

---

### Task 2: Add Library Bar HTML and CSS

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add CSS**

  In `index.html`, find the closing `</style>` tag. Add the following CSS immediately before it:

  ```css
  /* ── 染程庫列 ── */
  .library-bar {
    display: flex;
    align-items: center;
    gap: 12px;
    margin-bottom: 16px;
    padding: 12px 16px;
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 1px 4px rgba(0,0,0,.12);
    font-size: 15px;
    flex-wrap: wrap;
  }
  .library-bar label { color: #555; white-space: nowrap; }
  .library-bar select {
    flex: 1;
    max-width: 320px;
    min-width: 160px;
    border: 1px solid #bbb;
    border-radius: 4px;
    padding: 6px 10px;
    font-size: 15px;
    font-family: inherit;
    color: #222;
  }
  .btn-load         { background: #4a7c9e; color: #fff; }
  .btn-new-process  { background: #7d6608; color: #fff; display: none; }
  .btn-save-lib     { background: #1e8449; color: #fff; }
  .view-badge {
    display: none;
    background: #d6eaf8;
    color: #1a5276;
    border-radius: 4px;
    padding: 4px 10px;
    font-size: 13px;
    font-weight: bold;
  }
  /* 唯讀模式：inputs 看起來像文字 */
  input[type="number"]:disabled,
  input[type="text"]:disabled {
    background: transparent;
    color: #333;
    border-color: transparent;
    cursor: default;
  }
  td input[type="number"]:disabled:focus,
  input[type="text"]:disabled:focus {
    background: transparent;
    outline: none;
  }
  ```

- [ ] **Step 2: Add library bar HTML**

  Find this comment in the `<body>`:
  ```html
  <!-- 染程基本資訊 -->
  ```

  Add the following block immediately before that comment:
  ```html
  <!-- 染程庫 -->
  <div class="library-bar">
    <label>染程庫：</label>
    <select id="librarySelect">
      <option value="">— 選擇染程 —</option>
    </select>
    <button id="loadBtn" class="btn-add btn-load">載入</button>
    <button id="newProcessBtn" class="btn-new-process">新建染程</button>
    <span class="view-badge" id="viewBadge">檢視模式</span>
    <span id="libraryStatus" style="font-size:13px;color:#888;"></span>
  </div>
  ```

- [ ] **Step 3: Add 存入染程庫 button**

  Find the existing `.btn-row`:
  ```html
  <div class="btn-row">
    <button id="addStageBtn" class="btn-add">＋ 新增階段</button>
    <button id="delStageBtn" class="btn-del">－ 刪除最後階段</button>
  </div>
  ```

  Replace it with:
  ```html
  <div class="btn-row">
    <button id="addStageBtn" class="btn-add">＋ 新增階段</button>
    <button id="delStageBtn" class="btn-del">－ 刪除最後階段</button>
    <button id="saveLibBtn" class="btn-save-lib">存入染程庫</button>
  </div>
  ```

- [ ] **Step 4: Verify layout**

  Open `index.html` locally in a browser (double-click the file).
  Expected:
  - White bar at top of the page with dropdown "— 選擇染程 —" and blue "載入" button
  - Green "存入染程庫" button next to 新增/刪除 buttons
  - No console errors

---

### Task 3: Implement fetchLibraryList and loadProcess

**Files:**
- Modify: `index.html` (JS `<script>` block)

- [ ] **Step 1: Add fetchLibraryList**

  Find this comment in the `<script>` block:
  ```javascript
  // ── 初始化 ────────────────────────────────────────────────────
  ```

  Add the following functions immediately before that comment:

  ```javascript
  // ── 染程庫：讀取清單 ──────────────────────────────────────────
  async function fetchLibraryList() {
    const status = document.getElementById('libraryStatus');
    try {
      const res = await fetch(GAS_URL + '?action=list');
      const data = await res.json();
      const sel = document.getElementById('librarySelect');
      sel.innerHTML = '<option value="">— 選擇染程 —</option>';
      (data.names || []).forEach(name => {
        const opt = document.createElement('option');
        opt.value = name;
        opt.textContent = name;
        sel.appendChild(opt);
      });
    } catch (e) {
      status.textContent = '無法載入染程庫';
    }
  }
  ```

- [ ] **Step 2: Add populateTableFromStages**

  Add immediately after `fetchLibraryList`:

  ```javascript
  function populateTableFromStages(stages) {
    const tbody = document.querySelector('#stageTable tbody');
    tbody.innerHTML = '';
    stages.forEach((st, i) => {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td class="stage-num">${i + 1}</td>
        <td><input type="number" value="${st.s}"></td>
        <td><input type="number" value="${st.e}"></td>
        <td><input type="number" value="${st.t}"></td>
        <td class="type-hold">—</td>
        <td class="rate">—</td>`;
      tbody.appendChild(tr);
      updateRow(tr);
      attachRowListeners(tr);
    });
  }
  ```

- [ ] **Step 3: Add populateActionsFromData**

  Add immediately after `populateTableFromStages`:

  ```javascript
  function populateActionsFromData(actions) {
    const tbody = document.querySelector('#actionTable tbody');
    tbody.innerHTML = '';
    actions.forEach(act => {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td><input type="number" value="${act.temp}"></td>
        <td><input type="text" class="action-input-text" value="${String(act.label).replace(/"/g, '&quot;')}"></td>
        <td><button class="btn-remove">✕</button></td>`;
      tbody.appendChild(tr);
      attachActionRowListeners(tr);
    });
  }
  ```

- [ ] **Step 4: Add loadProcess**

  Add immediately after `populateActionsFromData`:

  ```javascript
  async function loadProcess(name) {
    const status = document.getElementById('libraryStatus');
    status.textContent = '載入中…';
    try {
      const res = await fetch(GAS_URL + '?action=get&name=' + encodeURIComponent(name));
      const data = await res.json();
      if (data.error) { status.textContent = '找不到染程'; return; }
      populateTableFromStages(data.stages || []);
      populateActionsFromData(data.actions || []);
      document.getElementById('processName').value = name;
      renderChart();
      enterViewMode(name);
      status.textContent = '';
    } catch (e) {
      status.textContent = '載入失敗，請重試';
    }
  }
  ```

- [ ] **Step 5: Wire up [載入] button and call fetchLibraryList on init**

  Find the entire `DOMContentLoaded` handler (starts with `document.addEventListener('DOMContentLoaded', () => {` and ends with the closing `});`). Replace it completely with:

  ```javascript
  document.addEventListener('DOMContentLoaded', () => {
    document.querySelectorAll('#stageTable tbody tr').forEach(attachRowListeners);
    document.querySelectorAll('#actionTable tbody tr').forEach(attachActionRowListeners);

    document.getElementById('addStageBtn').addEventListener('click', addStageRow);
    document.getElementById('delStageBtn').addEventListener('click', deleteLastRow);
    document.getElementById('addActionBtn').addEventListener('click', addActionRow);
    document.getElementById('processName').addEventListener('input', updateMeta);

    document.getElementById('uploadBtn').addEventListener('click', uploadToSheets);
    document.getElementById('copyTableBtn').addEventListener('click', copyTableTSV);
    document.getElementById('exportPNGBtn').addEventListener('click', exportPNG);
    document.getElementById('exportCSVBtn').addEventListener('click', exportCSV);

    document.getElementById('loadBtn').addEventListener('click', () => {
      const name = document.getElementById('librarySelect').value;
      if (name) loadProcess(name);
    });
    document.getElementById('newProcessBtn').addEventListener('click', enterDesignMode);
    document.getElementById('saveLibBtn').addEventListener('click', saveToLibrary);

    fetchLibraryList();
    renderChart();
  });
  ```

- [ ] **Step 6: Test fetch**

  Open `index.html` locally. Open DevTools (F12) → Console.
  Expected:
  - No CORS or network errors in Console
  - Dropdown shows "— 選擇染程 —" (empty — correct, no processes saved yet)
  - `libraryStatus` span is empty (not showing error)

---

### Task 4: Implement enterViewMode and enterDesignMode

**Files:**
- Modify: `index.html` (JS `<script>` block)

- [ ] **Step 1: Add enterViewMode**

  Find the `loadProcess` function you just added. Add this function immediately after `loadProcess`:

  ```javascript
  function enterViewMode(name) {
    document.querySelectorAll('#stageTable input, #actionTable input, #processName')
      .forEach(el => { el.disabled = true; });
    document.querySelectorAll('#actionTable .btn-remove')
      .forEach(btn => { btn.disabled = true; });
    document.getElementById('addStageBtn').style.display   = 'none';
    document.getElementById('delStageBtn').style.display   = 'none';
    document.getElementById('addActionBtn').style.display  = 'none';
    document.getElementById('saveLibBtn').style.display    = 'none';
    document.getElementById('newProcessBtn').style.display = '';
    document.getElementById('viewBadge').style.display     = '';
  }
  ```

- [ ] **Step 2: Add enterDesignMode**

  Add immediately after `enterViewMode`:

  ```javascript
  function enterDesignMode() {
    document.querySelectorAll('#stageTable input, #actionTable input, #processName')
      .forEach(el => { el.disabled = false; });
    document.querySelectorAll('#actionTable .btn-remove')
      .forEach(btn => { btn.disabled = false; });
    document.getElementById('addStageBtn').style.display   = '';
    document.getElementById('delStageBtn').style.display   = '';
    document.getElementById('addActionBtn').style.display  = '';
    document.getElementById('saveLibBtn').style.display    = '';
    document.getElementById('newProcessBtn').style.display = 'none';
    document.getElementById('viewBadge').style.display     = 'none';
    document.getElementById('librarySelect').value         = '';
    document.getElementById('processName').value           = '';
    document.getElementById('libraryStatus').textContent   = '';
    // Reset table to one empty row
    const tbody = document.querySelector('#stageTable tbody');
    tbody.innerHTML = `<tr>
      <td class="stage-num">1</td>
      <td><input type="number" value=""></td>
      <td><input type="number" value=""></td>
      <td><input type="number" value=""></td>
      <td class="type-hold">—</td>
      <td class="rate">—</td></tr>`;
    document.querySelectorAll('#stageTable tbody tr').forEach(attachRowListeners);
    document.querySelector('#actionTable tbody').innerHTML = '';
    renderChart();
  }
  ```

- [ ] **Step 3: Test mode switching in DevTools console**

  Open `index.html` locally. In DevTools Console, run:
  ```javascript
  enterViewMode('測試')
  ```
  Expected:
  - All table inputs become greyed-out / non-clickable
  - 新增/刪除/存入染程庫 buttons disappear
  - "新建染程" button (brown) appears
  - "檢視模式" badge appears

  Then run:
  ```javascript
  enterDesignMode()
  ```
  Expected:
  - Inputs re-enabled
  - All buttons restored
  - Table has one empty row
  - "新建染程" and badge gone

---

### Task 5: Implement saveToLibrary

**Files:**
- Modify: `index.html` (JS `<script>` block)

- [ ] **Step 1: Add saveToLibrary function**

  Add immediately after `enterDesignMode`:

  ```javascript
  async function saveToLibrary() {
    const name = document.getElementById('processName').value.trim();
    if (!name) { alert('請先輸入染程名稱'); return; }
    const stages = readStages();
    if (!stages.length) { alert('請先輸入染程階段'); return; }
    const password = prompt('請輸入儲存密碼：');
    if (password === null) return; // user cancelled

    const actions = readActions();
    const status = document.getElementById('libraryStatus');
    status.textContent = '儲存中…';
    status.style.color = '#888';

    try {
      const res = await fetch(GAS_URL, {
        method: 'POST',
        mode: 'cors',
        headers: { 'Content-Type': 'text/plain;charset=utf-8' },
        body: JSON.stringify({ action: 'saveToLibrary', password, name, stages, actions })
      });
      const data = await res.json();
      if (data.ok) {
        status.textContent = '✓ 已存入染程庫';
        status.style.color = '#1e8449';
        fetchLibraryList();
        setTimeout(() => { status.textContent = ''; status.style.color = '#888'; }, 3000);
      } else {
        status.textContent = '✗ ' + (data.error === 'wrong password' ? '密碼錯誤' : '儲存失敗');
        status.style.color = '#c0392b';
      }
    } catch (e) {
      status.textContent = '✗ 儲存失敗，請重試';
      status.style.color = '#c0392b';
    }
  }
  ```

  **Note on CORS:** `saveToLibrary` uses `mode: 'cors'` (unlike the existing upload which uses `mode: 'no-cors'`) so we can read the password-check response. This requires the Apps Script to include CORS headers — Apps Script web apps deployed with "Anyone" access do this automatically. If a CORS error appears in DevTools console after clicking "存入染程庫", verify the Apps Script deployment has "Who has access: Anyone".

- [ ] **Step 2: Test save to library**

  1. Enter process name `測試染程` in the tool
  2. Add stages: row 1 — 30, 30, 10; row 2 — 30, 70, 20
  3. Click **存入染程庫**
  4. Enter the password set in Step 2 of Task 1 (default: `change-me-123`)
  5. Expected: green "✓ 已存入染程庫" message, dropdown now shows "測試染程"
  6. Open Google Sheets → `染程庫` tab → verify a row was written with the process name and JSON data

- [ ] **Step 3: Test wrong password**

  Click **存入染程庫** again, enter an incorrect password.
  Expected: red "✗ 密碼錯誤" message. No new row added to Google Sheets.

- [ ] **Step 4: Test load after save**

  1. Select `測試染程` from the dropdown
  2. Click **載入**
  3. Expected: table fills with saved stages (30→30 10min, 30→70 20min), chart renders, "檢視模式" badge shown
  4. Click **新建染程** → table clears, back to design mode

---

### Task 6: Full Integration Test, Commit, and Push

**Files:**
- `index.html` (no changes — test and deploy only)

- [ ] **Step 1: Full end-to-end test**

  1. Save process `聚酯-標準` with 3 stages and 2 actions; verify in Sheets
  2. Save process `棉-深色` with different stages; verify in Sheets
  3. Reload the page → both names appear in dropdown
  4. Select `聚酯-標準` → click [載入] → correct stages and chart shown
  5. Click [新建染程] → clears, returns to design mode
  6. Select `棉-深色` → click [載入] → correct data loads
  7. Existing upload to Google Sheets still works (regression check: fill a process and click 上傳至 Google Sheets)

- [ ] **Step 2: Commit**

  ```bash
  cd "c:/Users/JiayuChen/資料管理/染程工具"
  git add index.html
  git commit -m "feat: add process library with dropdown load and password-protected save"
  ```

- [ ] **Step 3: Push**

  ```bash
  git push
  ```

  Wait ~1 minute. Open `https://q24769759-commits.github.io/dyeing-tool/` in a browser and verify the library bar appears and the feature works on the live site.
