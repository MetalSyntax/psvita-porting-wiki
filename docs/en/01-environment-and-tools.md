# 01. Environment & Toolchains

Developing and debugging an Android-to-PS Vita port requires a cross-compilation toolchain on your host development workstation (Linux, macOS, or WSL on Windows), reverse engineering suites, and specific kernel bridge plugins installed on physical hardware.

---

## 🛠️ Workstation Setup (Host)

### 1. VitaSDK (Official Community Toolchain)
[VitaSDK](https://github.com/vitasdk) is the GCC-based cross-compilation suite (`arm-vita-eabi-gcc`) for PlayStation Vita.

#### Quick Installation (Linux / macOS):
```bash
export VITASDK="$HOME/vitasdk"
export PATH="$VITASDK/bin:$PATH"

git clone https://github.com/vitasdk/vdpm.git
cd vdpm
./bootstrap-vitasdk.sh
./install-all.sh
```

Ensure variables are added to your shell profile (`~/.bashrc` or `~/.zshrc`):
```bash
export VITASDK="$HOME/vitasdk"
export PATH="$VITASDK/bin:$PATH"
```

Verify your setup:
```bash
arm-vita-eabi-gcc --version
# Should output: arm-vita-eabi-gcc (VitaSDK ...)
```

### 2. Mandatory Community Libraries (vdpm)
Install foundational porting packages:
```bash
vdpm vitaGL
vdpm mathneon
vdpm libvorbis
vdpm libogg
vdpm openal
vdpm zlib
```

### 3. Reverse Engineering Tools
To inspect APK contents and analyze native shared libraries:

- **jadx** (Decompiles DEX bytecode into clean Java source):
  - macOS: `brew install jadx`
  - Linux: Download from [jadx GitHub Releases](https://github.com/skylot/jadx/releases).
- **Docker + `devrvk/so-decompiler` (Headless Ghidra)**:
  Generates readable pseudo-C from any ARMv7 `.so` via CLI:
  ```bash
  docker pull devrvk/so-decompiler
  ```
- **VitaSDK Binary Utilities:**
  - `arm-vita-eabi-objdump`: Disassembler with full Thumb-2 and ARM ELF support.
  - `arm-vita-eabi-nm`: Symbol extractor.
  - `arm-vita-eabi-readelf`: ELF header, dynamic segment, and relocation inspector.

### 4. psvita-toolkit CLI
A production CLI designed for end-to-end port management:
```bash
# Verify host development dependencies
psvita-toolkit doctor

# Build with CMake presets
psvita-toolkit build --preset debug

# Instant eboot.bin deployment via FTP
psvita-toolkit deploy --eboot --yes
```

---

## 🎮 Required Kernel Plugins on PS Vita

For VitaOS to execute dynamically mapped userspace code, the following kernel plugins **must** be loaded in `ur0:tai/config.txt`:

### 1. kubridge (by TheFloW)
Repository: [github.com/TheOfficialFloW/kubridge](https://github.com/TheOfficialFloW/kubridge)  
*Why it's essential:*  
By default, VitaOS userland applications are prevented by W^X protections from creating memory blocks with simultaneous Read, Write, and Execute (**RWX**) permissions. `kubridge` exposes kernel services like `kuKernelAllocMemBlock`, allowing the loader to allocate executable memory for the `.so` image.

### 2. fd_fix (by TheFloW)
Repository: [github.com/TheOfficialFloW/fd_fix](https://github.com/TheOfficialFloW/fd_fix)  
*Why it's essential:*  
Prevents kernel crashes when an application closes duplicate or invalid POSIX file descriptors, ensuring standard I/O semantics match Android's Bionic behavior.

```ini
# Add to ur0:tai/config.txt
*KERNEL
ur0:tai/kubridge.skprx
ur0:tai/fd_fix.skprx
```

> [!CAUTION]
> Attempting to boot an SoLoader port without `kubridge.skprx` active will cause an immediate crash or black screen on launch before the game's first instruction executes.
