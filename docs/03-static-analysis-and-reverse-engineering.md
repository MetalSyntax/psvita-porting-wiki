<div class="chapter-hero">
  <span class="chapter-badge">Chapter 03 • Reverse Engineering</span>
  <h1 style="margin: 0.4rem 0 0.6rem 0; font-size: 2rem;">Static Analysis & Reverse Engineering</h1>
  <p style="margin: 0; opacity: 0.9; font-size: 1.05rem;">
    Systematic dissection of APK Java bytecode with jadx, headless Ghidra pseudo-C extraction, dynamic symbol auditing, and Thumb-2 disassembly to discover entry points and data structures.
  </p>
  <div class="chapter-meta">
    <span class="chapter-meta-pill">⚡ Level: Intermediate</span>
    <span class="chapter-meta-pill">🔬 Pipeline: jadx, Docker Ghidra, objdump -M force-thumb</span>
    <span class="chapter-meta-pill">🎮 Verified: Hero of Sparta, Zenonia 3, Badland 2</span>
  </div>
</div>

---

## ☕ Phase 1: Java Decompilation with `jadx`

Run `jadx` to decompile the APK's Dalvik bytecode into readable Java:

```bash
jadx -d decompiled_apk game.apk
```

### 1. Locate the Main Activity
Open `decompiled_apk/resources/AndroidManifest.xml` and identify the launch activity:

```xml
<activity android:name="com.gameloft.android.GAND.GloftSPAW.ML.HeroOfSparta">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```

### 2. Identify `native` Entry Points
Inspect the Activity and rendering loops (`GameRenderer`, `GLSurfaceView`, `NativeApp`). Look for methods declared with the `native` modifier:

=== "Java APK (jadx Decompilation)"
    ```java
    package com.gameloft.android;

    public class GameRenderer implements GLSurfaceView.Renderer {
        // Core lifecycle hooks
        public static native void nativeInit(int width, int height);
        public static native void nativeRender();
        public static native void nativeResize(int width, int height);
        public static native void nativeDone();

        // Input dispatchers
        public static native boolean nativeOnTouch(int action, float x, float y, int pointerId);
        public static native boolean nativeOnKeyDown(int keyCode);
    }
    ```

=== "Loader C Implementation (main.c)"
    ```c
    // Function pointer types matching Java native signatures
    typedef void (*nativeInit_t)(int width, int height);
    typedef void (*nativeRender_t)(void);
    typedef int (*nativeOnTouch_t)(int action, float x, float y, int pointerId);

    nativeInit_t game_nativeInit;
    nativeRender_t game_nativeRender;
    nativeOnTouch_t game_nativeOnTouch;

    void resolve_game_entrypoints(so_module *mod) {
        game_nativeInit = (nativeInit_t)so_resolve(mod, "Java_com_gameloft_android_GameRenderer_nativeInit");
        game_nativeRender = (nativeRender_t)so_resolve(mod, "Java_com_gameloft_android_GameRenderer_nativeRender");
        game_nativeOnTouch = (nativeOnTouch_t)so_resolve(mod, "Java_com_gameloft_android_GameRenderer_nativeOnTouch");

        if (!game_nativeInit || !game_nativeRender) {
            fatal_error("Could not resolve essential game entry points!");
        }
    }
    ```

> [!IMPORTANT]
> These `native` declarations are the **exact entry points** your Vita loader will invoke in `main.c` to drive game initialization, input polling, and frame rendering.

---

## 🧩 Phase 2: Dynamic Symbol Extraction (`.so`)

Use binary utilities to inspect exported functions in the `.so`:

### 1. List Exported JNI Functions
Standard JNI C naming conventions replace Java package dots `.` with underscores `_`:
```bash
arm-vita-eabi-objdump -T lib/armeabi-v7a/libgame.so | grep "Java_"
```

Example output:
```text
00045a10 g    DF .text  00000034 Java_com_gameloft_android_GameRenderer_nativeInit
00045a50 g    DF .text  00000120 Java_com_gameloft_android_GameRenderer_nativeRender
00045b80 g    DF .text  00000088 Java_com_gameloft_android_HeroOfSparta_nativeOnTouch
```

### 2. Check for `JNI_OnLoad`
```bash
arm-vita-eabi-objdump -T lib/armeabi-v7a/libgame.so | grep JNI_OnLoad
```
- **If `JNI_OnLoad` is present:** The game registers its native methods dynamically via `RegisterNatives()`. Your loader must call `JNI_OnLoad(&jvm, NULL)` during startup.
- **If `JNI_OnLoad` is absent:** Sockets and methods are statically mapped to `Java_package_class_method` exports. You can resolve their addresses directly with `so_resolve(&so_mod, "Java_...")`.

---

## 🧠 Phase 3: Headless Ghidra Decompilation with Docker

To analyze function logic and struct definitions without running a heavy GUI, use `devrvk/so-decompiler`:

```bash
mkdir -p decompiled_so/libgame_armv7
docker run --rm --platform linux/amd64 \
  -v "$(pwd)/lib/armeabi-v7a:/input" \
  -v "$(pwd)/decompiled_so/libgame_armv7:/output" \
  devrvk/so-decompiler decompile /input/libgame.so /output
```

This outputs:
1. `out_ghidra.c`: The entire binary reconstructed in **pseudo-C**.
2. `out_ghidra.h`: Inferred types, struct layouts, and prototypes.

=== "Ghidra Pseudo-C Output (out_ghidra.c)"
    ```c
    // Decompiled pseudo-C from headless Ghidra
    undefined4 Java_com_gameloft_android_GameRenderer_nativeInit(int param_1, undefined4 param_2, int width, int height) {
        g_ScreenWidth = width;
        g_ScreenHeight = height;
        InitGraphicsEngine(width, height);
        LoadGameAssets("/sdcard/gameloft/games/hos");
        return 1;
    }
    ```

Search `out_ghidra.c` for key paths, strings, and hardcoded asset directories:
```bash
grep -n "sdcard" decompiled_so/libgame_armv7/out_ghidra.c
grep -n "nativeRender" decompiled_so/libgame_armv7/out_ghidra.c
```

---

## ⚡ Phase 4: Disassembly with `-M force-thumb`

Over 95% of Android ARMv7 game binaries use the **Thumb-2** instruction set (mixed 16-bit and 32-bit opcodes) for code density.

> [!CAUTION]
> If you disassemble Thumb-2 code with standard host `objdump`, it defaults to 32-bit ARM mode and decodes **corrupt, nonsensical instructions**.

Always disassemble using the VitaSDK toolchain with `-M force-thumb`:

=== "Bash Command"
    ```bash
    arm-vita-eabi-objdump -d -M force-thumb \
      --start-address=0x00045a10 \
      --stop-address=0x00045b80 \
      lib/armeabi-v7a/libgame.so
    ```

=== "Thumb-2 Disassembly Output"
    ```assembly
       45a10:   b580        push    {r7, lr}
       45a12:   af00        add     r7, sp, #0
       45a14:   f240 102c   movw    r0, #300
       45a18:   f2c0 0004   movt    r0, #4
       45a1c:   6800        ldr     r0, [r0, #0]
       45a1e:   b910        cbnz    r0, 45a26
       45a20:   f7ff fffe   bl      0 <init_subsystem>
       45a24:   bd80        pop     {r7, pc}
    ```

---

## 📝 Documenting Findings in `PORTING_PLAN.md`

Summarize all findings in `PORTING_PLAN.md` at your project root:
- Identified Java package names and Title ID (e.g. `HOSP00001`).
- Selected `.so` binary path and memory footprint.
- Exported JNI methods and unresolved imports (`so_resolve`).
- Lifecycle methods (`init`, `render`, `pause`, `resume`, `touch`).
