# 09. File System & Asset Management

The PlayStation Vita storage topology and partitioning schemes differ significantly from standard Android Linux environments. Properly mapping paths and configuring asset packages (.apk/.obb) ensures your game locates its models, audio, textures, and save files.

---

## 💾 Storage Topology

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
- **`app0:` (Read-Only):** Where the app package is installed (`ux0:app/<TITLEID>/`). Contains `eboot.bin` and LiveArea data. **Writing to `app0:` triggers an immediate permission error.**
- **`ux0:data/<game>/` (Read-Write):** The standard homebrew location for game data files, saved states (`saves/`), and runtime log files (`logs/`).

---

## 🔀 POSIX Path Redirection

Mobile games hardcode Linux filesystem paths:
- `/sdcard/Android/data/com.gameloft.spaw/files/...`
- `/data/data/com.gamevil.zenonia3/...`

Intercept standard I/O calls (`fopen`, `open`, `stat`, `mkdir`):

```c
#define DATA_PATH "ux0:data/heroofsparta/"

char *sanitize_path(const char *path, char *out_path, size_t max_len) {
    if (!path) return NULL;
    
    if (strncmp(path, "ux0:", 4) == 0 || strncmp(path, "app0:", 5) == 0) {
        strncpy(out_path, path, max_len);
        return out_path;
    }
    
    const char *p = path;
    if (strncmp(p, "/sdcard/", 8) == 0) p += 8;
    else if (strncmp(p, "/storage/emulated/0/", 20) == 0) p += 20;
    else if (*p == '/') p++;
    
    snprintf(out_path, max_len, "%s%s", DATA_PATH, p);
    return out_path;
}

FILE *vita_fopen(const char *filename, const char *mode) {
    char clean_path[1024];
    sanitize_path(filename, clean_path, sizeof(clean_path));
    return fopen(clean_path, mode);
}
```

---

## 📦 Asset Packaging Strategies

| Strategy | Advantages | Drawbacks |
|---|---|---|
| **1. Unpacked Loose Files** | Easy to inspect and mod textures/music directly. | FAT32 cluster slack (32KB/64KB per cluster) wastes hundreds of megabytes on games with thousands of small files. |
| **2. In-Memory APK/OBB (ZIP)** | Saves massive space; single compressed archive. Faster sequential card reads. | Requires engine ZIP support (e.g. Cocos2d-x `CCFileUtils`) or linking `libzip` / `minizip`. |

---

## 🔠 Case Sensitivity (FAT vs ext4)

- **Android (ext4):** Strict case-sensitive paths (`Texture.PNG` != `texture.png`).
- **PS Vita (FAT32 / exFAT):** Case-preserving, but case-insensitive lookups.

> [!WARNING]
> If the APK contains two files differentiated only by casing in the same directory (e.g. `icon.png` and `ICON.PNG`), copying them to the Vita will overwrite one with the other. Check for collisions if assets appear corrupted.
