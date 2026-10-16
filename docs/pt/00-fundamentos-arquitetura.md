# 00. Fundamentos e Arquitetura

Para dominar o desenvolvimento de ports de jogos móveis para o PlayStation Vita, é indispensável compreender a arquitetura de hardware do console e os princípios teóricos do método **SoLoader**.

---

## 🔬 Arquitetura de Hardware do PS Vita

```mermaid
graph TD
    subgraph "SoC Sony CXD5315GG"
        subgraph "CPU: ARM Cortex-A9 MPCore"
            Core0["Core 0 (Thread Principal da App)"]
            Core1["Core 1 (Worker/Áudio)"]
            Core2["Core 2 (Worker/Render)"]
            Core3["Core 3 (Reservado OS / CapUnlocker)"]
            L1["L1 Cache: 32KB I + 32KB D por Núcleo"]
            L2["L2 Cache: 2MB Unificada Compartilhada"]
        end
        subgraph "GPU: PowerVR SGX543MP4+"
            USSE["4 Núcleos USSE2 (222 MHz)"]
            TBDR["Tile-Based Deferred Rendering (TBDR)"]
        end
    end

    subgraph "Subsistema de Memória"
        RAM["RAM Principal: 512 MiB LPDDR2"]
        CDRAM["VRAM (CDRAM): 112 MiB Dedicada"]
        SYS_VRAM["VRAM Reservada OS: 16 MiB"]
    end

    Core0 & Core1 & Core2 & Core3 --> L1 --> L2 --> RAM
    USSE --> TBDR --> CDRAM
```

### 1. A CPU (ARM Cortex-A9 MPCore)
O PlayStation Vita conta com um processador **ARM Cortex-A9** quad-core de 32 bits (arquitetura ARMv7-A).
- **Frequência padrão da Sony:** 333 MHz (reduzida de fábrica para economizar bateria).
- **Frequência máxima nativa:** 444 MHz / 500 MHz (desbloqueável com segurança por código ou plugins como o `PSVshell`).
- **Caches:** 32 KiB L1 de instruções + 32 KiB L1 de dados por núcleo; 2 MiB de cache L2 unificada compartilhada.
- **Ponto Flutuante:** VFPv3-D32 por hardware e motor vetorial SIMD NEON de 128 bits.
- **O 4º Núcleo:** Reservado pelo VitaOS para processos de fundo do sistema. Usando o plugin de kernel [CapUnlocker](https://github.com/GrapheneCt/CapUnlocker) de GrapheneCt, o homebrew pode reivindicar este núcleo aplicando a máscara de afinidade `0x80000` com `sceKernelChangeThreadCpuAffinityMask()`.

### 2. A GPU (PowerVR SGX543MP4+)
Um chip gráfico quad-core da série SGX543 operando até 222 MHz, utilizando uma arquitetura **TBDR** (Tile-Based Deferred Rendering).
- O processador gráfico divide o quadro em blocos de 16x16 pixels e executa descarte de superfícies ocultas (*Hidden Surface Removal / Early-Z*) antes dos fragment shaders, economizando largura de banda massiva de memória.

### 3. Memória Principal vs Memória de Vídeo (CDRAM)

| Partição | Tamanho Físico | Disponível para Homebrew | Propósito e Características |
|---|---|---|---|
| **RAM Principal** | 512 MiB | ~300 MiB a 420 MiB | Memória acessível pela CPU. Abriga o código, heap em C, o arquivo `.so` carregado, assets descompactados e a máquina FalsoJNI. |
| **CDRAM (VRAM)** | 128 MiB | **112 MiB** | Memória ultrarrápida ligada diretamente ao barramento da GPU. Utilizada para framebuffers, texturas ativas, vertex e index buffers. |
| **VRAM Sistema** | 16 MiB | 0 MiB | Reservada pela Sony para a interface do LiveArea e overlays do sistema operacional. |

---

## 💡 O Que é Exatamente o Método SoLoader?

A maioria dos emuladores interpreta instruções de uma CPU estrangeira ou as traduz em tempo real usando JIT. **O método SoLoader NÃO é um emulador nem um recompilador.**

### Por que o Android ARMv7 Executa NATIVAMENTE no PS Vita?
Tanto o PlayStation Vita quanto os smartphones Android da era 2010–2016 utilizam a **mesma arquitetura de conjunto de instruções: ARMv7-A (32 bits)** com instruções Thumb-2 e ponto flutuante VFP/NEON.

Quando um jogo é compilado no Android NDK em C++, o compilador gera uma biblioteca compartilhada no formato ELF: `libgame.so`. Esse arquivo contém **código de máquina ARM puro**.

| Plataforma | CPU ISA | Formato de Binário | Convenção de Chamada |
|---|---|---|---|
| **Android 32-bit** | ARMv7-A (Cortex-A8/A9/A15) | ELF32 (`.so`) | AAPCS (ARM Architecture Procedure Call Standard) |
| **PlayStation Vita** | ARMv7-A (Cortex-A9 MPCore) | ELF32 / Sony SELF (`eboot.bin`) | AAPCS (com softfp / hardfp compatível) |

Como a CPU do PlayStation Vita compreende diretamente essas instruções ARM, **a Vita executa o código do jogo diretamente no silício com velocidade nativa (zero perda por emulação de CPU).**

```mermaid
sequenceDiagram
    participant APK as Android APK (libgame.so)
    participant Loader as Vita Loader (eboot.bin)
    participant FalsoJNI as FalsoJNI Bridge
    participant VitaGL as vitaGL (OpenGL ES)
    participant VitaOS as Vita OS / Hardware

    Loader->>Loader: Lê ELF, aloca bloco RWX com sceKernelAllocMemBlock
    Loader->>Loader: Processa realocações (R_ARM_RELATIVE, R_ARM_GLOB_DAT)
    Loader->>Loader: Enlaça tabela de imports com funções do Loader
    Loader->>FalsoJNI: Inicializa JVM fictícia e JNIEnv
    Loader->>APK: Invoca JNI_OnLoad / nativeInit
    APK->>FalsoJNI: Chama Android Java API (CallStaticVoidMethod)
    FalsoJNI-->>APK: Retorna resposta simulada válida (stub ativo seguro)
    APK->>VitaGL: Envia geometria (glDrawElements)
    VitaGL->>VitaOS: Emite comandos SCE GXM por hardware
```

### O Carregador como "Sistema Operacional Convidado"
Se as instruções são nativamente compatíveis, por que não podemos rodar o `.so` diretamente?
Porque o `.so` foi construído esperando um ambiente **Linux / Android**:
1. Espera a biblioteca C padrão do Android (**Bionic libc**), com syscalls como `open()`, `mmap()`, `pthread_create()`.
2. Espera a superfície gráfica do Android (**EGL / SurfaceView / OpenGL ES**).
3. Espera o subsistema de áudio (**OpenSL ES**, **AudioTrack** ou **SoundPool**).
4. Espera uma máquina virtual Java (**Dalvik / ART**) atendendo chamadas **JNI** (`JNIEnv*`, `JavaVM*`).

O **SoLoader** (embutido no seu `eboot.bin` de Vita) preenche exatamente essa lacuna:
- Mapeia os segmentos de código e dados do ELF na memória da Vita.
- Corrige os ponteiros absolutos usando a tabela de realocações ARM.
- Intercepta importações de funções e as roteia para as implementações do VitaSDK e vitaGL.
- Fornece o ambiente simulado **FalsoJNI**, respondendo a todas as solicitações Java em C puro sem precisar de uma JVM real.
