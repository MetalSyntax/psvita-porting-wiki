<div class="chapter-hero">
  <span class="chapter-badge">Chapter 09 • File System & Storage</span>
  <h1 style="margin: 0.4rem 0 0.6rem 0; font-size: 2rem;">File System & Asset Management</h1>
  <p style="margin: 0; opacity: 0.9; font-size: 1.05rem;">
    Bridging Linux ext4 paths (/sdcard/, /data/data/) to PS Vita storage partitions (ux0:, app0:), implementing transparent path sanitization, and handling FAT32 cluster slack.
  </p>
  <div class="chapter-meta">
    <span class="chapter-meta-pill">⚡ Level: Intermediate</span>
    <span class="chapter-meta-pill">📁 Partitions: ux0: (Read-Write), app0: (Read-Only)</span>
    <span class="chapter-meta-pill">🎮 Verified: Hero of Sparta, Zenonia 3, Badland 2</span>
  </div>
</div>

---

## 💾 Storage Topology & Partitions

```mermaid
graph TD
    subgraph "VitaOS Partitions"
        UX0["ux0: (Primary Memory Card / SD2Vita)"]
        APP0["app0: (Read-only mount of current app)"]
        UR0["ur0: (System memory / taiHEN plugins)"]
    end

    subgraph "Loader Path Translation"
        AndroidPath["Android Path: /sdcard/Android/data/com.game/"]
        VitaDataPath["Target Path: ux0:data/my_game/"]
    end

    AndroidPath -->|sanitize_path()| VitaDataPath
    VitaDataPath --> UX0
    APP0 -->|Contains eboot.bin & LiveArea| UX0
```

### Partition Roles:
- **`app0:` (Read-Only):** The application sandbox mount (`ux0:app/<TITLEID>/`). Houses `eboot.bin` and LiveArea assets. **Writing to `app0:` returns an immediate permission error from the kernel.**
- **`ux0:data/<game>/` (Read-Write):** The standard homebrew data directory where the game stores loose assets, cache, user configuration, and save files.

---

## 🔀 Transparent POSIX Path Redirection

Mobile games hardcode Linux filesystem paths such as:
- `/sdcard/Android/data/com.gameloft.spaw/files/...`
- `/data/data/com.gamevil.zenonia3/...`
- `/mnt/sdcard/...`

Intercept standard C runtime I/O functions (`fopen`, `open`, `stat`, `mkdir`):

=== "Path Sanitization Engine (src/so_util.c)"
    ```c
    #include <stdio.h>
    #include <string.h>
    #include <sys/stat.h>

    #define DATA_PATH "ux0:data/heroofsparta/"

    char *sanitize_path(const char *path, char *out_path, size_t max_len) {
        if (!path) return NULL;

        // Passthrough if already pointing to a Vita native mount
        if (strncmp(path, "ux0:", 4) == 0 || strncmp(path, "app0:", 5) == 0) {
            strncpy(out_path, path, max_len - 1);
            out_path[max_len - 1] = '\0';
            return out_path;
        }

        const char *p = path;
        // Strip common Android root prefixes
        if (strncmp(p, "/sdcard/", 8) == 0) p += 8;
        else if (strncmp(p, "/storage/emulated/0/", 20) == 0) p += 20;
        else if (strncmp(p, "/mnt/sdcard/", 12) == 0) p += 12;
        else if (strncmp(p, "/data/data/", 11) == 0) {
            // Skip package name (/data/data/<package>/)
            p += 11;
            while (*p && *p != '/') p++;
            if (*p == '/') p++;
        }
        else if (*p == '/') p++;

        snprintf(out_path, max_len, "%s%s", DATA_PATH, p);
        return out_path;
    }

    FILE *vita_fopen(const char *filename, const char *mode) {
        char clean_path[1024];
        sanitize_path(filename, clean_path, sizeof(clean_path));
        return fopen(clean_path, mode);
    }

    int vita_stat(const char *pathname, struct stat *statbuf) {
        char clean_path[1024];
        sanitize_path(pathname, clean_path, sizeof(clean_path));
        return stat(clean_path, statbuf);
    }
    ```

=== "Hook Registration Table"
    ```c
    // In your so_util relocation table:
    {"fopen", (uintptr_t)&vita_fopen},
    {"open", (uintptr_t)&vita_open},
    {"stat", (uintptr_t)&vita_stat},
    {"mkdir", (uintptr_t)&vita_mkdir},
    {"remove", (uintptr_t)&vita_remove},
    {"unlink", (uintptr_t)&vita_unlink},
    ```

---

## 📦 Asset Extraction & Cluster Optimization

PlayStation Vita memory cards and SD2Vita cards are formatted with **FAT32 or exFAT** using 32 KB or 64 KB cluster sizes. 

If a mobile game contains 10,000 tiny sound and XML files (e.g. 500 bytes each), loose extraction can waste over **300 MB of space purely in cluster slack**.

=== "Automated Asset Preparation (Bash)"
    ```bash
    #!/usr/bin/env bash
    set -euo pipefail

    APK="$1"
    OUT_DIR="ux0_data_pack"
    mkdir -p "$OUT_DIR"

    echo "Extracting assets from $APK..."
    # Extract only the game asset tree
    unzip -q "$APK" "assets/*" -d "$OUT_DIR"

    # Normalize to lowercase if needed
    echo "Done. Transfer $OUT_DIR/assets contents to ux0:data/<game>/"
    ```

---

## 🔠 Case Sensitivity Gotchas

- **Android (ext4):** Strict case sensitivity (`Hero.png` != `hero.png`).
- **PS Vita (FAT32 / exFAT):** Case-preserving, but case-insensitive lookups.

> [!WARNING]
> If an APK contains two distinct files differing only in capitalization within the same directory (e.g. `icon.png` and `ICON.PNG`), copying them onto the Vita filesystem will silently overwrite one with the other. Always check for duplicate asset names during extraction.
