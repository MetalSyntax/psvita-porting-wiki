<div class="chapter-hero">
  <span class="chapter-badge">Capítulo 00 • Fundamentos del Hardware</span>
  <h1 style="margin: 0.4rem 0 0.6rem 0; font-size: 2rem;">Fundamentos y Arquitectura de Hardware</h1>
  <p style="margin: 0; opacity: 0.9; font-size: 1.05rem;">
    Análisis exhaustivo del hardware de PlayStation Vita (Cortex-A9, SGX543MP4+, particiones CDRAM y RAM) y los fundamentos de la ejecución nativa ARMv7.
  </p>
  <div class="chapter-meta">
    <span class="chapter-meta-pill">⚡ Nivel: Fundamental</span>
    <span class="chapter-meta-pill">🧠 Enfoque: ARMv7-A, SGX543 TBDR, CDRAM vs RAM</span>
    <span class="chapter-meta-pill">🎮 Casos Reales: Hero of Sparta, Zenonia 3, Badland 2</span>
  </div>
</div>

---

Para dominar el porting de juegos móviles hacia la PlayStation Vita, es imperativo comprender la arquitectura de hardware de la consola y la base teórica del método **SoLoader**.

---

## 🔬 Arquitectura de Hardware de PS Vita

```mermaid
graph TD
    subgraph "SoC Sony CXD5315GG"
        subgraph "CPU: ARM Cortex-A9 MPCore"
            Core0["Core 0 (App Main)"]
            Core1["Core 1 (App Worker/Audio)"]
            Core2["Core 2 (App Worker/Render)"]
            Core3["Core 3 (Reservado OS / CapUnlocker)"]
            L1["L1 Cache: 32KB I + 32KB D / Core"]
            L2["L2 Cache: 2MB Compartida"]
        end
        subgraph "GPU: PowerVR SGX543MP4+"
            USSE["4 Cores USSE2 (222 MHz)"]
            TBDR["Tile-Based Deferred Rendering (TBDR)"]
        end
    end

    subgraph "Subsistema de Memoria"
        RAM["RAM Principal: 512 MiB LPDDR2"]
        CDRAM["VRAM (CDRAM): 112 MiB Dedicada"]
        SYS_VRAM["VRAM Reservada OS: 16 MiB"]
    end

    Core0 & Core1 & Core2 & Core3 --> L1 --> L2 --> RAM
    USSE --> TBDR --> CDRAM
```

### 1. La CPU (ARM Cortex-A9 MPCore)
La PlayStation Vita monta un procesador **ARM Cortex-A9** de cuatro núcleos de 32 bits (arquitectura ARMv7-A).
- **Frecuencia nominal de Sony:** 333 MHz (reducida originalmente por preservación de batería).
- **Frecuencia máxima nativa:** 444 MHz / 500 MHz (desbloqueable sin riesgo mediante llamadas en código o plugins como `PSVshell`).
- **Cachés:** 32 KiB L1 de instrucciones + 32 KiB L1 de datos por núcleo; 2 MiB de caché L2 unificada compartida.
- **Unidad de punto flotante:** VFPv3-D32 y SIMD NEON de 128 bits integrados por hardware.
- **El 4º Núcleo:** Reservado por defecto para servicios del sistema operativo. Mediante el plugin de kernel [CapUnlocker](https://github.com/GrapheneCt/CapUnlocker) de GrapheneCt, un homebrew puede reclamar el 4º núcleo asignando la máscara de afinidad `0x80000` con `sceKernelChangeThreadCpuAffinityMask()`.

### 2. La GPU (PowerVR SGX543MP4+)
Un chip gráfico de cuatro núcleos de la serie SGX543 (PowerVR Series5XT) con arquitectura **TBDR** (Tile-Based Deferred Rendering).
- **Frecuencia:** Hasta 222 MHz (por defecto 111 MHz o 166 MHz).
- **Pipeline:** Totalmente programable mediante shaders (OpenGL ES 2.0 / DirectX 9 nivel de capacidades).
- **TBDR:** La GPU divide la pantalla en tiles de 16x16 píxeles, descarta fragmentos ocultos antes de ejecutar los pixel shaders (*Early-Z / HSR - Hidden Surface Removal*), ahorrando un ancho de banda masivo de memoria.

### 3. Memoria Principal vs Memoria de Vídeo (CDRAM)
Físicamente, la Vita posee dos buses y particiones de memoria claramente diferenciados:

| Partición | Tamaño Físico | Disponible para Homebrew | Propósito y Características |
|---|---|---|---|
| **RAM Principal** | 512 MiB | ~300 MiB a 420 MiB | Memoria de sistema accesible por CPU. Aloja el ejecutable, el heap de C, el archivo `.so` cargado, los assets descomprimidos y la máquina FalsoJNI. |
| **CDRAM (VRAM)** | 128 MiB | **112 MiB** | Memoria gráfica ultrarrápida ligada directamente al bus de la GPU. Reservada para framebuffers, texturas activas, vertex buffers e índices de renderizado. |
| **VRAM Sistema** | 16 MiB | 0 MiB | Reservada por Sony para la UI del LiveArea y el framebuffer de overlay del OS. |

> [!WARNING]
> Intentar colocar texturas gigantescas (ej. 4096x4096 o múltiples 2048x2048 sin comprimir) agotará rápidamente los 112 MiB de CDRAM, causando un crash `0x80010018` (`SCE_KERNEL_ERROR_NO_MEMORY`). En la Vita, la compresión de texturas y el empaquetado eficiente en atlas es mandatorio.

---

## 💡 ¿Qué es exactamente el método SoLoader?

La mayoría de los emuladores interpretan instrucciones de una CPU foránea o usan un recompilador dinámico (JIT) para traducir instrucciones en tiempo real. **El método SoLoader NO es un emulador ni un recompilador.**

### ¿Por qué Android ARMv7 ejecuta NATIVO en PS Vita?
Tanto la PlayStation Vita como los smartphones Android de la era 2010–2016 comparten la **misma arquitectura de conjunto de instrucciones: ARMv7-A (32 bits)** con instrucciones Thumb-2 y punto flotante VFP/NEON.

Cuando compilas un juego para Android en C++ usando el Android NDK, el compilador genera un archivo de librería compartida ELF: `libgame.so`.
Ese binario contiene **instrucciones de máquina ARM puras**.

| Plataforma | CPU ISA | Formato de Binario | ABI de Llamada | Capa de Sistema Operativo |
|---|---|---|---|---|
| **Android 32-bit** | ARMv7-A (Cortex-A8/A9/A15) | ELF32 (`.so`) | AAPCS estándar | Bionic Libc + Java VM (JNI) |
| **Apple iOS 32-bit** | ARMv7-A (Cortex-A8/A9) | Mach-O 32-bit (`MH_EXECUTE`) | Darwin ARM ABI (r9 scratch, stack 4B) | Darwin Libc + Objective-C Runtime |
| **PlayStation Vita** | ARMv7-A (Cortex-A9 MPCore) | ELF32 / Sony SELF (`eboot.bin`) | AAPCS (SoftFP / HardFP) | VitaOS Microkernel + Newlib Libc |

Debido a que el procesador de la PlayStation Vita comprende directamente los opcodes ARM generados tanto para Android como para iOS, **la CPU de la Vita ejecuta las instrucciones del binario móvil directamente en silicio a velocidad de hardware nativa (cero overhead de emulación)**.

```mermaid
sequenceDiagram
    autonumber
    participant App as Binario Móvil (ELF o Mach-O)
    participant Loader as Vita Loader (eboot.bin)
    participant Bridge as Bridge (FalsoJNI / Mini-ObjC)
    participant VitaGL as vitaGL (OpenGL ES)
    participant VitaOS as Vita OS / Hardware

    Loader->>Loader: Reserva memoria RWX con kubridge (sceKernelAllocMemBlock)
    Loader->>Loader: Aplica relocalizaciones (R_ARM_RELATIVE o rebase.bin)
    Loader->>Loader: Enlaza tabla de imports a funciones nativas de Vita
    Loader->>Bridge: Inicializa entorno (JNIEnv o Despachador objc_msgSend)
    Loader->>App: Invoca punto de entrada principal (JNI_OnLoad o main)
    App->>Bridge: Llama a servicios de plataforma (Java API o selectores Cocoa)
    Bridge-->>App: Devuelve respuesta simulada válida (no-op seguro)
    App->>VitaGL: Dibuja geometría (glDrawElements / glDrawArrays)
    VitaGL->>VitaOS: Convierte a command buffers SCE GXM por hardware
```

### Los Dos Ecosistemas Huéspedes: Android e iOS
Si las instrucciones son compatibles, ¿por qué no podemos simplemente ejecutar el binario directamente?
Porque el código fue concebido esperando uno de dos entornos de sistema operativo:
1. **El Entorno Android (ELF):** Espera la biblioteca estándar de C de Android (**Bionic libc**), gráficos en superficie **EGL**, audio vía **OpenSL ES** o **AudioTrack**, y la máquina virtual Java (**Dalvik/ART**) para llamadas **JNI**. Se atiende mediante el cargador ELF y **FalsoJNI**.
2. **El Entorno iOS (Mach-O):** Espera **Darwin libc**, gráficos vía **EAGLContext / CAEAGLLayer**, audio vía **CoreAudio / AudioQueue**, y llamadas al sistema mediante el despachador de mensajería **`objc_msgSend`**. Se atiende mediante el cargador Mach-O y el **Mini-Runtime de Objective-C** en C puro.

El **Loader** (empaquetado en tu `eboot.bin` de Vita) cumple exactamente ese papel:
- **Carga el binario:** Mapea los segmentos de código y datos en la memoria de la Vita.
- **Relocaliza los punteros:** Ajusta todas las direcciones de memoria absoluta (`.rel.dyn` en ELF o `rebase.bin` en Mach-O).
- **Intercepta las importaciones:** Redirige llamadas del sistema hacia la implementación nativa de VitaSDK y vitaGL.
- **Suministra la plataforma simulada:** Responde a las consultas del juego sobre rutas de almacenamiento, controladores, dimensiones de pantalla y audio sin necesidad de emular sistemas operativos completos.

---

## ⚖️ Comparativa: SoLoader vs Otras Técnicas

| Característica | SoLoader (Este método) | Recompilación Estática | Emulación (ej. Vita3K / PPSSPP) |
|---|---|---|---|
| **Rendimiento** | **100% Nativo** (Sin pérdida de FPS por CPU). | **100% Nativo**. | Pérdida de 30% a 70% por traducción dinámica de instrucciones. |
| **Consumo de Memoria** | **Mínimo** (~5 MB de overhead para el loader). | Mínimo. | Alto (requiere buffers de emulador, JIT cache). |
| **Código Fuente Original** | **NO requerido** (Parte del binario compilado `.so`). | **NO requerido** (parte del binario desensamblado). | **NO requerido**. |
| **Tiempo de Desarrollo** | De días a semanas (según complejidad de JNI y motor). | Meses de ingeniería inversa y recreación. | N/A (el emulador ya existe). |
| **Precisión de Gráficos** | Acelerado por GPU real vía vitaGL (GXM). | Acelerado por GPU. | Emulado por software o hardware wrapper. |
