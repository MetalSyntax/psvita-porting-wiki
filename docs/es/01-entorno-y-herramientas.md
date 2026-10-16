<div class="chapter-hero">
  <span class="chapter-badge">Capítulo 01 • Entorno & Toolchains</span>
  <h1 style="margin: 0.4rem 0 0.6rem 0; font-size: 2rem;">Entorno de Desarrollo y Herramientas</h1>
  <p style="margin: 0; opacity: 0.9; font-size: 1.05rem;">
    Configuración de la toolchain cruzada VitaSDK, librerías vdpm (vitaGL, mathneon), suites de ingeniería inversa (jadx, Ghidra Docker) y plugins de kernel (kubridge, fd_fix).
  </p>
  <div class="chapter-meta">
    <span class="chapter-meta-pill">⚡ Nivel: Fundamental</span>
    <span class="chapter-meta-pill">🛠️ Herramientas: VitaSDK, CMake, vdpm, jadx, Ghidra</span>
    <span class="chapter-meta-pill">🔌 Kernel: kubridge.skprx, fd_fix.skprx</span>
  </div>
</div>

---

Para desarrollar y depurar un port de Android para PS Vita se necesita una cadena de compilación cruzada en el sistema anfitrión (Linux, macOS o WSL en Windows) y un conjunto de herramientas de ingeniería inversa y plugins en la consola física.

---

## 🛠️ Herramientas en la Máquina de Desarrollo (Host)

### 1. VitaSDK (El Toolchain Oficial de la Comunidad)
[VitaSDK](https://github.com/vitasdk) es la suite de compilación cruzada basada en GCC (`arm-vita-eabi-gcc`) para la PlayStation Vita.

#### Instalación rápida (Linux / macOS):
```bash
# Definir directorio de instalación (ejemplo en ~/.vitasdk)
export VITASDK="$HOME/vitasdk"
export PATH="$VITASDK/bin:$PATH"

# Descargar e instalar mediante el script oficial
git clone https://github.com/vitasdk/vdpm.git
cd vdpm
./bootstrap-vitasdk.sh
./install-all.sh
```

Añade las variables a tu archivo de configuración de shell (`~/.bashrc` o `~/.zshrc`):
```bash
export VITASDK="$HOME/vitasdk"
export PATH="$VITASDK/bin:$PATH"
```

Verifica la instalación ejecutando:
```bash
arm-vita-eabi-gcc --version
# Debe responder: arm-vita-eabi-gcc (VitaSDK ...) 10.x o superior
```

### 2. Librerías de Enlace Indispensables (vdpm)
Instala las librerías base de la comunidad requeridas por la mayoría de los ports:
```bash
vdpm vitaGL
vdpm mathneon
vdpm libvorbis
vdpm libogg
vdpm openal
vdpm zlib
```

### 3. Herramientas de Ingeniería Inversa (Reverse Engineering)
Para abrir el APK y examinar el binario nativo:

- **jadx** (Decompilador de bytecode DEX de Android a código Java limpio):
  - macOS: `brew install jadx`
  - Linux: Descargar desde [Releases de jadx en GitHub](https://github.com/skylot/jadx/releases).
- **Docker + `devrvk/so-decompiler` (Ghidra Headless)**:
  Permite decompilar cualquier `.so` de ARMv7 a pseudo-C legible desde la línea de comandos sin tener que interactuar con la interfaz gráfica de Ghidra:
  ```bash
  docker pull devrvk/so-decompiler
  ```
- **Herramientas binarias de VitaSDK:**
  - `arm-vita-eabi-objdump`: Desensamblador con soporte específico de Thumb-2 y ELF ARM.
  - `arm-vita-eabi-nm`: Extractor de símbolos.
  - `arm-vita-eabi-readelf`: Lector de cabeceras ELF, tablas dinámicas y relocalizaciones.

### 4. psvita-toolkit CLI
El CLI de automatización desarrollado para gestionar ports de forma estandarizada:
```bash
# Diagnóstico del entorno
psvita-toolkit doctor

# Compilación con CMake / Ninja
psvita-toolkit build --preset debug

# Despliegue ultra-rápido de eboot.bin por FTP
psvita-toolkit deploy --eboot --yes
```

---

## 🎮 Plugins de Kernel Requeridos en la PS Vita

Para que la PlayStation Vita permita ejecutar código cargado dinámicamente en espacio de usuario, es **obligatorio** que la consola física tenga instalados los siguientes plugins de kernel en `ur0:tai/config.txt`:

### 1. kubridge (de TheFloW)
Repositorio: [github.com/TheOfficialFloW/kubridge](https://github.com/TheOfficialFloW/kubridge)  
*¿Por qué es vital?*  
La PlayStation Vita por seguridad no permite que una aplicación en modo usuario asigne páginas de memoria con permisos simultáneos de lectura, escritura y ejecución (**RWX**), ni permite resolver símbolos directamente del kernel. `kubridge` expone las llamadas de kernel `kuKernelAllocMemBlock` y servicios de puente que hacen posible que el cargador asigne la memoria ejecutable donde se mapea el archivo `.so`.

### 2. fd_fix (de TheFloW)
Repositorio: [github.com/TheOfficialFloW/fd_fix](https://github.com/TheOfficialFloW/fd_fix)  
*¿Por qué es vital?*  
En el kernel de PlayStation Vita, cuando una aplicación intenta cerrar un descriptor de archivo duplicado o inválido, el sistema operativo puede generar un kernel panic o corromper el árbol de descriptores I/O de la aplicación. `fd_fix` previene este comportamiento garantizando compatibilidad total con la semántica de descriptores POSIX esperada por los motores de Android.

```ini
# Fragmento obligatorio en ur0:tai/config.txt
*KERNEL
ur0:tai/kubridge.skprx
ur0:tai/fd_fix.skprx
```

> [!CAUTION]
> Si intentas arrancar un port basado en SoLoader en una consola que **no** tiene `kubridge.skprx` instalado o activo, el juego se cerrará inmediatamente al iniciar con un error de acceso a memoria o pantalla negra antes de ejecutar la primera línea de código del motor.

---

## 🗂️ Estructura del Repositorio de un Port

Un repositorio típico de porting mantiene la siguiente organización limpia:

```
mi-juego-vita/
├── CMakeLists.txt              # Configuración de compilación con VitaSDK y vitaGL
├── PORTING_PLAN.md             # Plan maestro, matriz de símbolos y checklist
├── CLAUDE.md                   # Instrucciones para asistentes de IA y desarrollo
├── .psvita-toolkit.json        # Configuración de IP de la consola y rutas locales
├── source/                     # Código fuente del cargador (C / C++)
│   ├── main.c                  # Punto de entrada, inicialización de vitaGL, bucle principal
│   ├── dynlib.c / so_util.c    # Cargador de ELF, procesador de relocalizaciones
│   ├── patch.c                 # Parches de memoria en caliente e intercepción de funciones
│   ├── falso_jni/              # Entorno FalsoJNI (JVM, JNIEnv y callbacks Java)
│   │   ├── falso_jni.c
│   │   └── falso_jni.h
│   └── reimpl/                 # Reimplementaciones de subsistemas (audio, input, io)
├── extras/                     # Materiales gráficos para LiveArea
│   └── livearea/
│       ├── icon0.png           # 128x128 (8-bit indexado)
│       ├── startup.png         # 280x158 (8-bit indexado)
│       ├── bg0.png             # 840x500 (8-bit indexado)
│       ├── pic0.png            # 960x544 (opcional)
│       └── template.xml        # Plantilla XML del LiveArea
└── ux0_data/                   # Assets necesarios para copiar a ux0:data/mi-juego/
    └── mi-juego/
        └── libgame.so          # Archivo .so original extraído del APK
```
