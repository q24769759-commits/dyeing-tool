# GitHub Pages Deploy Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Publish the dyeing tool (`index.html`) to a public URL via GitHub Pages so anyone can open it in a browser.

**Architecture:** The tool is a single static HTML file — no build step needed. GitHub Pages serves `index.html` directly from the `master` branch root. The existing local git repo is connected to a new public GitHub repo, then pushed.

**Tech Stack:** Git, GitHub Pages (free static hosting)

---

### Task 1: Create GitHub Repo

**Files:**
- No files to edit — this is a GitHub web UI action

- [ ] **Step 1: Open GitHub and create a new repo**

  Go to https://github.com/new and fill in:
  - Repository name: `dyeing-tool`
  - Description: `染程工具 — 輸入溫度/時間產出曲線表格`
  - Visibility: **Public**
  - **Do NOT** check "Add a README file"
  - **Do NOT** check "Add .gitignore"
  - Click **Create repository**

- [ ] **Step 2: Copy the repo URL**

  After creation, GitHub shows a "Quick setup" page. Copy the HTTPS URL — it looks like:
  ```
  https://github.com/<your-username>/dyeing-tool.git
  ```

---

### Task 2: Connect Local Repo and Push

**Files:**
- No file edits — git commands only

- [ ] **Step 1: Add remote origin**

  In the terminal, from `c:/Users/JiayuChen/資料管理/染程工具`:
  ```bash
  git remote add origin https://github.com/<your-username>/dyeing-tool.git
  ```
  Replace `<your-username>` with your actual GitHub username.

- [ ] **Step 2: Verify remote was added**

  ```bash
  git remote -v
  ```
  Expected output:
  ```
  origin  https://github.com/<your-username>/dyeing-tool.git (fetch)
  origin  https://github.com/<your-username>/dyeing-tool.git (push)
  ```

- [ ] **Step 3: Push to GitHub**

  ```bash
  git push -u origin master
  ```
  GitHub will prompt for your username and password (use a Personal Access Token as password if prompted — GitHub no longer accepts plain passwords).

  Expected output:
  ```
  Branch 'master' set up to track remote branch 'master' from 'origin'.
  ```

- [ ] **Step 4: Verify files are on GitHub**

  Open `https://github.com/<your-username>/dyeing-tool` in a browser. You should see `index.html` listed in the repo.

---

### Task 3: Enable GitHub Pages

**Files:**
- No file edits — GitHub repo settings UI

- [ ] **Step 1: Open repo Settings**

  On the GitHub repo page, click the **Settings** tab (top menu).

- [ ] **Step 2: Navigate to Pages**

  In the left sidebar, click **Pages**.

- [ ] **Step 3: Set source**

  Under "Build and deployment" → "Source", select **Deploy from a branch**.
  Then set:
  - Branch: `master`
  - Folder: `/ (root)`
  
  Click **Save**.

- [ ] **Step 4: Wait for deployment**

  GitHub will show a banner: *"GitHub Pages source saved."*
  Wait about 1 minute, then refresh the page. A green banner will appear with the URL:
  ```
  Your site is live at https://<your-username>.github.io/dyeing-tool/
  ```

---

### Task 4: Verify and Share

- [ ] **Step 1: Open the live URL**

  Visit `https://<your-username>.github.io/dyeing-tool/` in a browser.
  
  Expected: The dyeing tool loads, input table is visible, chart renders.

- [ ] **Step 2: Test the upload function**

  Fill in a few stages, click **上傳至 Google Sheets**, and confirm the data appears in the Google Sheet.

- [ ] **Step 3: Share the URL**

  Copy `https://<your-username>.github.io/dyeing-tool/` and send to teammates.

---

## Future Updates

Any time `index.html` is changed locally, just run:
```bash
git add index.html
git commit -m "update: <description>"
git push
```
GitHub Pages will auto-redeploy within ~1 minute. The URL stays the same.
