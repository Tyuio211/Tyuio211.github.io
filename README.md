# 📻 EchoReader — SQLite-WASM Powered Single Page RSS Engine

EchoReader is a modern, high-performance, local-first **Single Page RSS & Atom Reader** that runs entirely within your web browser.

Unlike traditional browser readers that quickly exhaust their storage quotas or break due to strict CORS rules, EchoReader features a professional-grade **SQLite WebAssembly (WASM)** transactional engine using the official **Key-Value Virtual File System (`kvvfs`)** mapped over `localStorage`. It stores metadata permanently while using a volatile on-demand streaming system for full article content, providing a fast, scalable, and resilient reader with zero backend dependencies.

## 🚀 Key Features

* **Local-First SQLite Engine**: Powered by official compiled SQLite WASM. Run transactional statements, triggers, cascades, and data compaction operations natively in your browser's main thread.

* **Smart Split-Cache Storage**:

  * **Persistent (SQLite)**: Core subscription channels, metadata fields, titles, publish dates, and read/bookmark indices are safely stored.

  * **Volatile (In-Memory)**: Heavy full-length HTML article content structures are cached in volatile memory and fetched live on-demand, preventing browser storage limit exhaustion.

* **Auto-Pruner Engine**: Proactively monitors storage usage. When reaching a safe threshold limit (\~4.2MB), it automatically deletes older unstarred articles and issues `VACUUM` routines to keep the database small and clean.

* **Blank Slate Philosophy**: Built without annoying pre-populated subscription noise. Launch with a clean dashboard ready for your custom sources.

* **CORS Sandbox-Safe Proxies**: Uses a rotation chain of redundant sandbox-friendly proxies (`AllOrigins JSON`, `CorsProxy.io`, `CodeTabs`) that automatically route feeds through iframe environments without breaking.

* **OPML Backup Integration**: Easily import existing subscription lists from Feedly, Inoreader, or NetNewsWire, or export your current database to a standard `.opml` backup file.

* **Detailed Link Inspector**: Click on any active subscription in the sidebar to view, audit, or copy its original source XML/Atom endpoint URL with a one-click clipboard tool.

* **Aesthetic Premium UI**: Featuring a responsive, elegant layout, custom fluid scrollbars, dynamic sidebar categorization, global search, reading lists, typography size zoom levels, and persistent dark/light mode configurations.

## 🛠️ SQLite Database Schema

EchoReader configures standard relational constraints directly inside the browser storage container. When initialized, the following relational schemas are generated:

```sql
-- Active Subscriptions Table
CREATE TABLE IF NOT EXISTS feeds (
    id TEXT PRIMARY KEY,
    url TEXT UNIQUE,
    title TEXT,
    category TEXT,
    lastFetched TEXT
);

-- Meta Articles Pointer Table (With cascading deletes)
CREATE TABLE IF NOT EXISTS articles (
    id TEXT PRIMARY KEY,
    feedId TEXT,
    feedTitle TEXT,
    title TEXT,
    link TEXT,
    date TEXT,
    author TEXT,
    read INTEGER DEFAULT 0,
    bookmarked INTEGER DEFAULT 0,
    FOREIGN KEY(feedId) REFERENCES feeds(id) ON DELETE CASCADE
);

-- Enable standard cascading rules
PRAGMA foreign_keys = ON;
```

## 📦 How to Deploy on GitHub Pages

Since EchoReader is a fully self-contained single-page application, deploying it to GitHub Pages takes less than a minute.

### Option 1: Quick Web Upload

1. Create a new public repository on GitHub (e.g., `echoreader`).

2. Upload the `index.html` file directly to the root of your repository.

3. Head to **Settings** > **Pages** inside your GitHub repository sidebar.

4. Set the **Source** to `Deploy from a branch`, choose `main` (or `master`) and the `/root` folder, then hit **Save**.

5. Your custom feed engine will be live at `https://<your-username>.github.io/echoreader/` within seconds!

### Option 2: Command Line Git Deploy

Clone, add, commit, and push your file to GitHub:

```bash
# Initialize repository
git init
git add index.html README.md
git commit -m "feat: initial release of SQLite RSS Engine"
git branch -M main

# Add your repository url and deploy
git remote add origin [https://github.com/your-username/echoreader.git](https://github.com/your-username/echoreader.git)
git push -u origin main
```

## 🧪 Browser Storage Safe Limits & Auto-Pruning

Browsers restrict `localStorage` domains to a hard quota of approximately **5MB**. EchoReader works smart to prevent crashes or write failures:

1. **Size Auditing**: Every database modification evaluates the global size of the storage footprint.

2. **Phase 1 Purge**: If consumption surpasses `4.2MB`, the pruner queries SQLite to delete all read, non-starred articles (`read = 1 AND bookmarked = 0`) and triggers `VACUUM`.

3. **Phase 2 Purge**: If storage is still tight, it performs an aggressive wipe of unread, non-starred articles older than the latest 100 entries.

## 📝 License

This project is open-source and available under the [MIT License](LICENSE). Feel free to modify, self-host, and customize it to match your personal offline workflow!
