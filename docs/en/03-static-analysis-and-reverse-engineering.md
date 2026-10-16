# 03. Static Analysis & Reverse Engineering

Static analysis is the systematic process of dissecting the APK's Java bytecode and native `.so` binaries without executing them, identifying lifecycle hooks, exported methods, and underlying data structures.

---

## ☕ Phase 1: Java Decompilation with `jadx`

Run `jadx` to decompile the APK's DEX bytecode into Java source:

```bash
jadx -d decompiled_apk game.apk
```

### 1. Locate the Main Activity
Open `decompiled_apk/resources/AndroidManifest.xml` and find the activity handling `android.intent.action.MAIN`:

```xml
<activity android:name="com.gameloft.android.GAND.GloftSPAW.ML.HeroOfSparta">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```

### 2. Search for `native` Method Declarations
Inspect the Activity and related rendering classes (`GameRenderer`, `GLSurfaceView`, `NativeApp`) inside `decompiled_apk/sources/`. Look for the `native` keyword:

```java
public class GameRenderer implements GLSurfaceView.Renderer {
    public static native void nativeInit(int width, int height);
    public static native void nativeRender();
    public static native void nativeResize(int width, int height);
    public static native void nativeDone();
    public static native boolean nativeOnTouch(int action, float x, float y, int pointerId);
    public static native boolean nativeOnKeyDown(int keyCode);
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
objdump -T lib/armeabi-v7a/libgame.so | grep "Java_"
```

Example output:
```
00045a10 g    DF .text  00000034 Java_com_gameloft_android_GameRenderer_nativeInit
00045a50 g    DF .text  00000120 Java_com_gameloft_android_GameRenderer_nativeRender
00045b80 g    DF .text  00000088 Java_com_gameloft_android_HeroOfSparta_nativeOnTouch
```

### 2. Check for `JNI_OnLoad`
```bash
objdump -T lib/armeabi-v7a/libgame.so | grep JNI_OnLoad
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
3. `out_angr.c`: Symbolic execution graphs.

Search `out_ghidra.c` for key functions:
```bash
grep -n "nativeRender" decompiled_so/libgame_armv7/out_ghidra.c
```

---

## ⚡ Phase 4: Disassembly with `-M force-thumb`

Over 95% of Android ARMv7 game binaries use the **Thumb-2** instruction set (mixed 16-bit and 32-bit opcodes) for code density.

> [!CAUTION]
> If you disassemble Thumb-2 code with standard system `objdump`, it defaults to 32-bit ARM mode and decodes **corrupt, nonsensical instructions**.

Always disassemble using the VitaSDK toolchain with `-M force-thumb`:

```bash
arm-vita-eabi-objdump -d -M force-thumb \
  --start-address=0x00045a10 \
  --stop-address=0x00045b80 \
  lib/armeabi-v7a/libgame.so
```

Output:
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

## 📝 Documenting in `PORTING_PLAN.md`

Summarize all findings in `PORTING_PLAN.md` at your project root:
- Identified Java package names and Title ID.
- Selected `.so` binary path and size.
- Exported JNI methods and unresolved imports.
- Lifecycle methods (`init`, `render`, `pause`, `resume`, `touch`).
