# 09. Sistema de Archivos y Assets

El subsistema de almacenamiento de PlayStation Vita y sus esquemas de partición difieren drásticamente del entorno Linux/Android. Comprender cómo redirigir rutas y gestionar los paquetes de datos (.apk/.obb) es indispensable para que el juego encuentre sus texturas, modelos y archivos de guardado.

---

## 💾 Topología de Almacenamiento en PS Vita

```mermaid
graph TD
    subgraph "Esquema de Dispositivos VitaOS"
        UX0["ux0: (Tarjeta de Memoria / SD2Vita Principal)"]
        APP0["app0: (Montaje de solo lectura de la aplicación actual)"]
        UR0["ur0: (Memoria interna de sistema / Plugins taiHEN)"]
    end

    subgraph "Redirección en el Loader"
        AndroidPath["Ruta Android: /sdcard/Android/data/com.juego/"]
        VitaDataPath["Ruta Real: ux0:data/mi_juego/"]
    end

    AndroidPath -->|Función sanitize_path()| VitaDataPath
    VitaDataPath --> UX0
    APP0 -->|Contiene eboot.bin y LiveArea| UX0
```

### Principales Particiones:
- **`app0:` (Solo Lectura):** Es el directorio donde se instala el juego (`ux0:app/<TITLEID>/`). Contiene el ejecutable `eboot.bin`, los metadatos de LiveArea y las librerías. **Cualquier intento de escribir o guardar archivos en `app0:` fallará con error de permiso denegado.**
- **`ux0:data/<juego>/` (Lectura y Escritura):** Es la ubicación estándar en el ecosistema homebrew donde deben residir los assets del juego, las partidas guardadas (`saves/`) y los archivos de log (`logs/`).

---

## 🔀 Redirección de Rutas POSIX

Los juegos de Android intentan abrir archivos usando rutas hardcodeadas de Linux:
- `/sdcard/Android/data/com.gameloft.spaw/files/...`
- `/data/data/com.gamevil.zenonia3/...`
- `assets/textures/hero.png`

En el cargador (`io.c` o `reimpl/io.c`), interceptamos las llamadas del sistema de archivos (`fopen`, `open`, `stat`, `mkdir`):

```c
#define DATA_PATH "ux0:data/heroofsparta/"

char *sanitize_path(const char *path, char *out_path, size_t max_len) {
    if (!path) return NULL;
    
    // Si la ruta ya apunta a ux0: o app0:, no tocarla
    if (strncmp(path, "ux0:", 4) == 0 || strncmp(path, "app0:", 5) == 0) {
        strncpy(out_path, path, max_len);
        return out_path;
    }
    
    // Eliminar prefijos comunes de Android
    const char *p = path;
    if (strncmp(p, "/sdcard/", 8) == 0) p += 8;
    else if (strncmp(p, "/storage/emulated/0/", 20) == 0) p += 20;
    else if (*p == '/') p++; // Quitar barra inicial
    
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

## 📦 Estrategias de Distribución de Assets

| Estrategia | Ventajas | Desventajas |
|---|---|---|
| **1. Archivos Sueltos Descomprimidos** | Fácil de explorar y modificar texturas o música a mano. | El tamaño de clúster en tarjetas FAT32 (32 KB o 64 KB) desperdicia cientos de megabytes en juegos con miles de archivos diminutos (*slack space*). |
| **2. Lectura Directa de APK/OBB (ZIP en memoria)** | Ahorra espacio masivo; un solo archivo OBB/APK contiene todo empaquetado. Carga más rápida en la tarjeta. | Requiere que el motor soporte lectura de ZIP (como `CCFileUtils` en Cocos2d-x) o enlazar `libzip` / `minizip`. |

### El Fallback de Cocos2d-x a Archivos ZIP:
Los motores basados en Cocos2d-x (`CCFileUtils::getFileData`) están programados para buscar primero el archivo suelto en la ruta; si no existe, buscan automáticamente dentro del archivo `.apk` o `.obb` utilizando la librería `unzip`. 

---

## 🔠 Sensibilidad a Mayúsculas y Minúsculas (FAT vs ext4)

- **Android (Linux ext4):** El sistema de archivos es estrictamente **sensible a mayúsculas** (`Sprite.PNG` es un archivo totalmente distinto a `sprite.png`).
- **PS Vita (FAT32 / exFAT):** El sistema de archivos conserva mayúsculas pero es **insensible a mayúsculas y minúsculas** en las búsquedas.

> [!WARNING]
> Si dos archivos en el APK original se llaman `icon.png` y `ICON.PNG` en la misma carpeta, al copiarlos a la PS Vita uno sobreescribirá al otro. Asegúrate de verificar colisiones si notas assets corruptos o reemplazados.
