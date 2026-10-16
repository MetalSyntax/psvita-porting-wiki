<div class="lang-switch-bar">
  <span>🌐 Language / Idioma:</span>
  <a href="../../03-static-analysis-and-reverse-engineering/" class="lang-switch-link">🇬🇧 English</a>
  <a href="./" class="lang-switch-link active">🇪🇸 Español</a>
  <a href="../../pt/03-analise-estatica-e-engenharia-reversa/" class="lang-switch-link">🇧🇷 Português</a>
</div>

<div class="chapter-hero">
  <span class="chapter-badge">Capítulo 03 • Ingeniería Inversa</span>
  <h1 style="margin: 0.4rem 0 0.6rem 0; font-size: 2rem;">Análisis Estático e Ingeniería Inversa</h1>
  <p style="margin: 0; opacity: 0.9; font-size: 1.05rem;">
    Disección de bytecode Dalvik con jadx, extracción de pseudo-C con Ghidra headless, auditoría de símbolos y desensamblado Thumb-2 para descubrir puntos de entrada.
  </p>
  <div class="chapter-meta">
    <span class="chapter-meta-pill">⚡ Nivel: Intermedio</span>
    <span class="chapter-meta-pill">🔬 Pipeline: jadx, Docker Ghidra, objdump -M force-thumb</span>
    <span class="chapter-meta-pill">🎮 Verificado: Hero of Sparta, Zenonia 3, Badland 2</span>
  </div>
</div>

---

El análisis estático es el proceso de examinar la estructura interna del APK, sus clases Java y sus binarios `.so` sin ejecutarlos, con el objetivo de descubrir el ciclo de vida del juego, sus llamadas nativas y sus puntos de entrada.

---

## ☕ Fase 1: Decompilación de Clases Java con `jadx`

El motor nativo en C++ (`.so`) no vive aislado; en Android interactúa constantemente con la máquina virtual Java. Ejecuta `jadx` sobre el APK:

```bash
jadx -d decompiled_apk juego.apk
```

### 1. Encontrar la Actividad Principal (`Activity`)
Abre `decompiled_apk/resources/AndroidManifest.xml` y busca la actividad que contiene el filtro `android.intent.action.MAIN`:

```xml
<activity android:name="com.gameloft.android.GAND.GloftSPAW.ML.HeroOfSparta">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```

### 2. Localizar las Declaraciones `native`
Abre esa clase y las clases asociadas a gráficos (comúnmente llamadas `GameRenderer`, `GLSurfaceView`, o `NativeApp`) en `decompiled_apk/sources/`.
Busca la palabra clave `native`:

```java
// Ejemplo real en Java decompilado:
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
> Estas funciones marcadas con `native` son los **puntos de entrada directos** que el loader de Vita deberá invocar en su bucle principal (`main.c`) para hacer funcionar el juego.

---

## 🧩 Fase 2: Extracción de Símbolos del Binario `.so`

Inspecciona la tabla de símbolos dinámicos del archivo `.so` usando las herramientas de línea de comandos.

### 1. Listar Métodos JNI Exportados
La convención estándar de nombres de JNI en C transforma los paquetes Java sustituyendo los puntos `.` por guiones bajos `_`:
```bash
objdump -T lib/armeabi-v7a/libgame.so | grep "Java_"
```

Salida típica:
```
00045a10 g    DF .text  00000034 Java_com_gameloft_android_GameRenderer_nativeInit
00045a50 g    DF .text  00000120 Java_com_gameloft_android_GameRenderer_nativeRender
00045b80 g    DF .text  00000088 Java_com_gameloft_android_HeroOfSparta_nativeOnTouch
```

### 2. ¿Existe `JNI_OnLoad`?
Busca si la librería exporta `JNI_OnLoad`:
```bash
objdump -T lib/armeabi-v7a/libgame.so | grep JNI_OnLoad
```
- **Si existe `JNI_OnLoad`:** El juego utiliza casi con certeza registro dinámico de símbolos (`RegisterNatives`) al cargarse. Tu loader **debe** llamar a `JNI_OnLoad(&jvm, NULL)` durante el arranque.
- **Si NO existe `JNI_OnLoad`:** El juego resuelve los métodos de forma estática según la convención `Java_paquete_clase_metodo`. Puedes invocar directamente las funciones exportadas resolviendo sus direcciones con `so_resolve(&so_mod, "Java_...")`.

---

## 🧠 Fase 3: Decompilación Headless con Ghidra en Docker

Para entender qué hace internamente una función nativa sin necesidad de abrir la interfaz gráfica de Ghidra en cada sesión, utilizamos el contenedor Docker `devrvk/so-decompiler`:

```bash
mkdir -p decompiled_so/libgame_armv7
docker run --rm --platform linux/amd64 \
  -v "$(pwd)/lib/armeabi-v7a:/input" \
  -v "$(pwd)/decompiled_so/libgame_armv7:/output" \
  devrvk/so-decompiler decompile /input/libgame.so /output
```

Este proceso generará tres archivos fundamentales:
1. `out_ghidra.c`: Todo el binario reconstruido en lenguaje **pseudo-C**.
2. `out_ghidra.h`: Tipos de datos, structs y firmas de función.
3. `out_angr.c`: Análisis simbólico y grafo de control de flujo.

### Búsqueda de Lógica Interna
Puedes buscar funciones específicas en `out_ghidra.c` con herramientas estándar de texto:
```bash
grep -n "nativeRender" decompiled_so/libgame_armv7/out_ghidra.c
```
Esto te revelará exactamente qué campos de struct consulta el motor, si accede a punteros globales o si comprueba banderas de depuración o licencias.

---

## ⚡ Fase 4: Desensamblado de Opcodes con `-M force-thumb`

El 95% de los binarios de Android compilados para ARMv7 usan el conjunto de instrucciones **Thumb-2** (instrucciones mixtas de 16 y 32 bits) para ahorrar tamaño de código.

> [!CAUTION]
> Si intentas desensamblar un binario Thumb-2 con el `objdump` genérico del sistema operativo, el desensamblador asumirá por defecto el modo ARM estándar de 32 bits, produciendo **instrucciones corruptas y sin sentido**.

Para desensamblar con precisión quirúrgica, utiliza siempre el toolchain de VitaSDK con la bandera `-M force-thumb`:

```bash
arm-vita-eabi-objdump -d -M force-thumb \
  --start-address=0x00045a10 \
  --stop-address=0x00045b80 \
  lib/armeabi-v7a/libgame.so
```

Salida limpia y exacta:
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

## 📝 Documento Maestro: El `PORTING_PLAN.md`

Una vez concluido el análisis estático, documenta todos los hallazgos en la raíz de tu proyecto en `PORTING_PLAN.md`:
- Paquete Java exacto.
- Nombre de la librería `.so` y tamaño en disco.
- Lista completa de métodos `Java_*` exportados.
- Lista de dependencias no resueltas (`UND`).
- Puntos de entrada para el ciclo de vida (`init`, `render`, `pause`, `resume`, `touch`).
