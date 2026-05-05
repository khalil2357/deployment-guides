# PhpStorm Remote Server Connection Guide
**Complete step-by-step guide to connecting, browsing, comparing, and syncing files with a remote server via SFTP — replacing MobaXterm entirely.**

---

## Prerequisites

- PhpStorm installed and a project open
- Remote server credentials: **IP address**, **username**, **password**, **SSH port**
- Your project already open locally in PhpStorm

---

## Step 1 — Add a Deployment Server

1. Go to `File → Settings → Build, Execution, Deployment → Deployment`
2. Click the **`+`** button (top left of the panel)
3. Select **`SFTP`**
4. Give it a name (e.g. `My Production Server`)
5. Click **OK**

---

## Step 2 — Configure the Connection Tab

Fill in the following fields:

| Field | Value |
|---|---|
| **Type** | `SFTP` |
| **SSH configuration** | Click the `...` button → fill in host, port, username, password (see below) |
| **Root path** | `/` *(leave as slash — allows full server browsing)* |
| **Web server URL** | `http://YOUR_SERVER_IP` *(optional, used for browser preview)* |
| **Use sudo to run SFTP server** | ✅ Check this if your server requires elevated permissions |

### SSH Configuration Detail

When you click `...` next to SSH configuration, a new dialog opens. Fill in:

| Field | Value |
|---|---|
| **Host** | Your server IP (e.g. `192.168.113.216`) |
| **Port** | Your SSH port (e.g. `22` or custom like `3649`) |
| **Username** | Your server username (e.g. `paystation`) |
| **Authentication type** | `Password` |
| **Password** | Your server password |
| **Save password** | ✅ Check to avoid re-entering every time |

Click **Test Connection** — you should see `"Successfully connected"`. If not, double-check credentials and port.

---

## Step 3 — Configure the Mappings Tab

Click the **Mappings** tab. This tells PhpStorm which local folder maps to which server folder.

| Field | Value |
|---|---|
| **Local path** | Your local project root (e.g. `D:\xampp\htdocs\my-project`) |
| **Deployment path** | Server path to your project (e.g. `/var/www/html/my-project`) |
| **Web path** | `/` |

> **Tip:** The Deployment path is relative to Root path. If Root path is `/`, then Deployment path must be the full absolute server path.

Click **Apply**.

---

## Step 4 — Set as Default Server

In the Deployment list on the left, make sure your server has the **green checkmark** active. If not, select it and click the green checkmark button at the top of the list.

This makes it the default target for all upload/download/compare actions.

---

## Step 5 — Exclude Unnecessary Folders

To avoid syncing `vendor`, `node_modules`, compiled assets, and other noise:

1. Click the **Excluded Paths** tab
2. Click **`+`** → choose **`Deployment Path`**
3. Add each folder as a **separate entry** (one per line):

```
/vendor
/node_modules
/storage
/.git
/public/assets
/public/build
/public/hot
```

> ⚠️ Do NOT add multiple paths in one line — each must be its own entry.

Click **Apply → OK**.

---

## Step 6 — Open the Remote Host Panel

`View → Tool Windows → Remote Host`

Or via: `Tools → Deployment → Browse Remote Host`

This opens a **file tree of your server** on the right side of PhpStorm — just like MobaXterm's file browser.

> If it shows **"Nothing to show"**, click the **folder icon** at the top of the panel to navigate to your deployment path, or press the refresh button (↻) inside the panel.

---

## Daily Workflow

### Compare a Single File with Server

1. Click a file in the **Project panel** (left sidebar) to focus it
2. Right-click → `Deployment → Compare with Deployed Version`
3. PhpStorm opens a **side-by-side diff** of local vs server instantly
4. Review changes, then right-click → `Deployment → Upload to [Server Name]` to push

---

### Compare the Entire Project with Server

1. Right-click the **root project folder** in the Project panel
2. `Deployment → Sync with Deployed To... → [Your Server]`
3. Wait for the scan to complete (1–3 minutes for large projects)

The sync window shows all differences:

| Indicator | Meaning |
|---|---|
| Blue / highlighted row | File differs between local and server |
| File on left only | Exists locally, not on server |
| File on right only | Exists on server, not locally |

---

### Filter the Sync View (Show Only Differences)

In the sync toolbar, click the **blue funnel icon (Filter)**. Turn OFF:
- `Equal files` — hides files that are identical
- `Only on server` — hides server-only files (optional)

Keep ON:
- `Different files`
- `Only on local`

---

### Push Files to Server

| Action | How |
|---|---|
| Push **selected file(s)** only | Select rows → click **`▶`** (single play button) in toolbar |
| Push **all shown files** | Click **`▶▶`** (double play button) in toolbar |
| Push a **single file** from Project panel | Right-click file → `Deployment → Upload to [Server]` |

> ✅ Upload confirmation appears as a toast notification at the bottom: `"Upload to [Server] completed: X file(s) transferred"`

> ⚠️ **Upload is immediate** — there is no undo. Always review the diff before pushing.

---

### Download a File from Server

In the **Remote Host panel**:
- Navigate to the file
- Right-click → `Download`

Or in the sync panel:
- Select the file row → click **`←`** (left arrow) to copy server → local

---

### Compare by Setting

In the sync toolbar, use the **`Compare by:`** dropdown:

| Option | Use when |
|---|---|
| **Binary Content** ✅ | Default — most accurate, catches all changes |
| **Text** | Use if you see too many false positives due to Windows (CRLF) vs Linux (LF) line endings |
| **Size** | Not recommended — misses same-size edits |
| **Size and Timestamp** | Not recommended — unreliable |

> **Laravel on Windows:** If hundreds of files show as "different" when you haven't changed them, switch to **Text** mode. This is caused by Windows CRLF vs Linux LF line ending differences.

---

## Troubleshooting

### Upload Completed but File Didn't Change on Server
- Check `View → Tool Windows → Event Log` for errors
- Verify the **Deployment path** in Mappings is exactly correct
- Confirm the server user has **write permission** on that folder
- Try: right-click a file in Project panel → `Deployment → Upload to [Server]` directly

### Remote Host Shows "Nothing to Show"
- Click the **folder icon** inside the Remote Host panel to navigate to your path
- Press **↻** refresh inside the panel
- Ensure your deployment config is saved (Apply → OK in Settings)

### Deployment Options are Greyed Out in Tools Menu
- You must have a **file focused/open in the editor** first
- Click any open file tab, place cursor inside it, then retry

### Test Connection Fails
- Double-check IP, port, username, password
- Confirm SSH is enabled on the server
- Check firewall is not blocking the port
- Try connecting with MobaXterm first to verify credentials work

---

## Quick Reference Shortcuts

| Action | Shortcut / Location |
|---|---|
| Upload current file | Right-click → `Deployment → Upload to [Server]` |
| Compare current file | Right-click → `Deployment → Compare with Deployed Version` |
| Sync entire project | Right-click root folder → `Deployment → Sync with Deployed To...` |
| Browse remote host | `Tools → Deployment → Browse Remote Host` |
| Open event log | `View → Tool Windows → Event Log` |
| Start SSH terminal | `Tools → Start SSH Session` |
| Auto-upload on save | `Settings → Deployment → Options → Upload on explicit save` |

---

## Optional — Auto Upload on Save

To automatically push every file save to the server:

`Settings → Build, Execution, Deployment → Deployment → Options`
→ Set **"Upload changed files automatically to the default server"** to `On explicit save action`

> ⚠️ Only enable this on development/staging servers — never on production.

---

*Guide based on PhpStorm 2026.x — steps may vary slightly on older versions.*
