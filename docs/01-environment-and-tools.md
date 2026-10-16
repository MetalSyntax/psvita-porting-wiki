<div class="chapter-hero">
  <span class="chapter-badge">Chapter 01 • Toolchain & Hardware</span>
  <h1 style="margin: 0.4rem 0 0.6rem 0; font-size: 2rem;">Environment Setup & Toolchains</h1>
  <p style="margin: 0; opacity: 0.9; font-size: 1.05rem;">
    Comprehensive setup guide for host cross-compilation environments (VitaSDK, vdpm, CMake), reverse engineering pipelines (jadx, Ghidra), and mandatory PS Vita kernel bridge plugins.
  </p>
  <div class="chapter-meta">
    <span class="chapter-meta-pill">⚡ Level: Fundamental</span>
    <span class="chapter-meta-pill">🛠️ Tools: VitaSDK, CMake, vdpm, jadx, Ghidra</span>
    <span class="chapter-meta-pill">🔌 Kernel: kubridge.skprx, fd_fix.skprx</span>
  </div>
</div>

---

## 🛠️ Workstation Setup (Host)

### 1. VitaSDK Cross-Compilation Toolchain
VitaSDK provides `arm-vita-eabi-gcc` and associated binary utilities for building ARMv7 homebrew on PlayStation Vita.

=== "macOS (Homebrew / Script)"
    ```bash
    # Set environment variables
    export VITASDK="/usr/local/vitasdk"
    export PATH="$VITASDK/bin:$PATH"

    # Install vdpm package manager
    git clone https://github.com/vitasdk/vdpm.git
    cd vdpm
    ./bootstrap-vitasdk.sh
    ./install-all.sh
    ```

=== "Ubuntu / Debian / WSL2"
    ```bash
    # Install prerequisites
    sudo apt-get update
    sudo apt-get install -y build-essential cmake git curl python3 xz-utils

    # Set environment variables in ~/.bashrc
    export VITASDK="$HOME/vitasdk"
    export PATH="$VITASDK/bin:$PATH"

    git clone https://github.com/vitasdk/vdpm.git
    cd vdpm
    ./bootstrap-vitasdk.sh
    ./install-all.sh
    ```

=== "Docker (Fast Prebuilt)"
    ```bash
    # Pull official VitaSDK docker container
    docker pull vitasdk/vitasdk:latest

    # Build any port project directly inside container
    docker run --rm -v $(pwd):/workspace -w /workspace vitasdk/vitasdk:latest cmake -B build && make -C build
    ```

Verify your setup:
```bash
arm-vita-eabi-gcc --version
# Expected output: arm-vita-eabi-gcc (VitaSDK ...) 10.x / 11.x
```

---

### 2. Mandatory Porting Libraries via `vdpm`
Install foundational graphics, audio, and compression libraries:

```bash
vdpm vitaGL
vdpm mathneon
vdpm libvorbis
vdpm libogg
vdpm openal
vdpm zlib
```

---

### 3. Production `CMakeLists.txt` Boilerplate
Every modern SoLoader port uses a standard CMake structure linking VitaSDK and vitaGL:

=== "CMakeLists.txt"
    ```cmake
    cmake_minimum_required(VERSION 3.20)
    project(game_vita C CXX)

    if(NOT DEFINED ENV{VITASDK})
      message(FATAL_ERROR "Please define VITASDK to point to your SDK path!")
    endif()

    include("$ENV{VITASDK}/share/vita.cmake" REQUIRED)

    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3 -mfloat-abi=hard -ffast-math -fsigned-char")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -mfloat-abi=hard -ffast-math -fno-rtti -fno-exceptions")

    add_executable(game_vita
      src/main.c
      src/so_util.c
      src/falso_jni.c
      src/patch.c
      src/gl_util.c
    )

    target_include_directories(game_vita PRIVATE
      include
      $ENV{VITASDK}/arm-vita-eabi/include/vitaGL
    )

    target_link_libraries(game_vita
      vitaGL
      vitashark
      SceShaccCgExt
      taihen_stub
      kubridge_stub
      mathneon
      openal
      vorbisfile
      vorbis
      ogg
      SceAppMgr_stub
      SceCommonDialog_stub
      SceCtrl_stub
      SceMotion_stub
      SceTouch_stub
      SceAudio_stub
      ScePower_stub
    )

    vita_create_self(eboot.bin game_vita UNSAFE)
    vita_create_vpk(game.vpk "GAME00001" eboot.bin
      VERSION "01.00"
      NAME "Sample Port"
      FILE livearea/icon0.png sce_sys/icon0.png
      FILE livearea/bg.png sce_sys/livearea/contents/bg.png
      FILE livearea/template.xml sce_sys/livearea/contents/template.xml
    )
    ```

---

### 4. Reverse Engineering Suites

- **jadx** (Decompile APK DEX bytecode into readable Java):
  ```bash
  # macOS
  brew install jadx
  # Linux / WSL
  sudo apt-get install -y jadx || curl -L "https://github.com/skylot/jadx/releases/latest/download/jadx-1.5.0.zip" -o jadx.zip
  ```
- **Docker + `devrvk/so-decompiler` (Headless Ghidra Decompiler)**:
  ```bash
  # Decompile any ARMv7 .so file into high-fidelity C pseudo-code in seconds
  docker run --rm -v $(pwd):/data devrvk/so-decompiler /data/libgame.so /data/decompiled_so.c
  ```
- **VitaSDK Binary Analysis Tools:**
  ```bash
  # Disassemble ARM & Thumb-2 instructions
  arm-vita-eabi-objdump -d libgame.so > disassembly.asm

  # Extract exported and imported symbols
  arm-vita-eabi-nm -D libgame.so > symbols.txt

  # Inspect dynamic sections and relocations
  arm-vita-eabi-readelf -r libgame.so > relocations.txt
  ```

---

## 🎮 Required Kernel Plugins on PS Vita

For VitaOS to execute dynamically mapped userspace code and provide POSIX compatibility, these kernel plugins **must** be loaded in `ur0:tai/config.txt`:

### 1. kubridge (by TheFloW)
- **Repo:** [TheOfficialFloW/kubridge](https://github.com/TheOfficialFloW/kubridge)
- **Role:** By default, VitaOS userland applications are prevented by W^X protections from creating memory blocks with simultaneous Read, Write, and Execute (**RWX**) permissions. `kubridge` exposes kernel services like `kuKernelAllocMemBlock`, allowing the loader to allocate executable memory for the `.so` image.

### 2. fd_fix (by TheFloW)
- **Repo:** [TheOfficialFloW/fd_fix](https://github.com/TheOfficialFloW/fd_fix)
- **Role:** Prevents kernel crashes when an application closes duplicate or invalid POSIX file descriptors, ensuring standard I/O semantics match Android's Bionic behavior.

```ini
# Edit ur0:tai/config.txt
*KERNEL
ur0:tai/kubridge.skprx
ur0:tai/fd_fix.skprx
```

> [!CAUTION]
> Attempting to boot an SoLoader port without `kubridge.skprx` active will cause an immediate crash (`C2-12828-1`) or black screen on launch before the game's first instruction executes.
