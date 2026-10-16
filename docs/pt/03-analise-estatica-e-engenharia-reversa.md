<div class="lang-switch-bar">
  <span>🌐 Language / Idioma:</span>
  <a href="../../03-static-analysis-and-reverse-engineering/" class="lang-switch-link">🇬🇧 English</a>
  <a href="../../es/03-analisis-estatico-e-ingenieria-inversa/" class="lang-switch-link">🇪🇸 Español</a>
  <a href="./" class="lang-switch-link active">🇧🇷 Português</a>
</div>

<div class="chapter-hero">
  <span class="chapter-badge">Capítulo 03 • Engenharia Reversa</span>
  <h1 style="margin: 0.4rem 0 0.6rem 0; font-size: 2rem;">Análise Estática e Engenharia Reversa</h1>
  <p style="margin: 0; opacity: 0.9; font-size: 1.05rem;">
    Dissecação de bytecode Dalvik com jadx, extração de pseudo-C com Ghidra headless, auditoria de símbolos e desmonte Thumb-2 para descobrir pontos de entrada.
  </p>
  <div class="chapter-meta">
    <span class="chapter-meta-pill">⚡ Nível: Intermediário</span>
    <span class="chapter-meta-pill">🔬 Pipeline: jadx, Docker Ghidra, objdump -M force-thumb</span>
    <span class="chapter-meta-pill">🎮 Verificado: Hero of Sparta, Zenonia 3, Badland 2</span>
  </div>
</div>

---

A análise estática consiste em examinar o código Java e os binários nativos `.so` do jogo sem executá-los, mapeando o ciclo de vida, chamadas JNI e estruturas internas de dados.

---

## ☕ Fase 1: Descompilação Java com `jadx`

Execute o `jadx` para extrair o código Java das classes do APK:

```bash
jadx -d decompiled_apk jogo.apk
```

### 1. Localizar a Atividade Principal (`Activity`)
Abra `decompiled_apk/resources/AndroidManifest.xml` e encontre a atividade com `android.intent.action.MAIN`:

```xml
<activity android:name="com.gameloft.android.GAND.GloftSPAW.ML.HeroOfSparta">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```

### 2. Localizar Declarações `native`
Abra as classes de renderização e controle (`GameRenderer`, `GLSurfaceView`, `NativeApp`) em `decompiled_apk/sources/` e procure por métodos `native`:

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
> Esses métodos nativos são os **pontos de entrada exatos** que o carregador da Vita invocará em `main.c` para inicializar o jogo, receber toques na tela e renderizar os quadros.

---

## 🧩 Fase 2: Extração de Símbolos do Binário `.so`

Inspecione as funções exportadas na biblioteca nativa:

### 1. Listar Métodos JNI Exportados
A convenção padrão do JNI converte pontos `.` do pacote Java em sublinhados `_`:
```bash
objdump -T lib/armeabi-v7a/libgame.so | grep "Java_"
```

Saída de exemplo:
```
00045a10 g    DF .text  00000034 Java_com_gameloft_android_GameRenderer_nativeInit
00045a50 g    DF .text  00000120 Java_com_gameloft_android_GameRenderer_nativeRender
00045b80 g    DF .text  00000088 Java_com_gameloft_android_HeroOfSparta_nativeOnTouch
```

### 2. Verificar a Presença de `JNI_OnLoad`
```bash
objdump -T lib/armeabi-v7a/libgame.so | grep JNI_OnLoad
```
- **Se `JNI_OnLoad` existir:** O jogo registra métodos nativos dinamicamente via `RegisterNatives()`. O carregador deve invocar `JNI_OnLoad(&jvm, NULL)` na inicialização.
- **Se `JNI_OnLoad` não existir:** Os métodos seguem a convenção estática `Java_pacote_classe_metodo`. Você pode resolver os endereços diretamente usando `so_resolve(&so_mod, "Java_...")`.

---

## 🧠 Fase 3: Descompilação Headless com Ghidra no Docker

Para analisar structs e fluxo de controle sem a interface pesada do Ghidra, use o contêiner `devrvk/so-decompiler`:

```bash
mkdir -p decompiled_so/libgame_armv7
docker run --rm --platform linux/amd64 \
  -v "$(pwd)/lib/armeabi-v7a:/input" \
  -v "$(pwd)/decompiled_so/libgame_armv7:/output" \
  devrvk/so-decompiler decompile /input/libgame.so /output
```

Arquivos gerados:
1. `out_ghidra.c`: O binário completo reconstruído em **pseudo-C**.
2. `out_ghidra.h`: Declarações de structs e tipos.
3. `out_angr.c`: Grafo de controle de fluxo simbólico.

Pesquise por funções no pseudo-C:
```bash
grep -n "nativeRender" decompiled_so/libgame_armv7/out_ghidra.c
```

---

## ⚡ Fase 4: Desmontagem com `-M force-thumb`

Mais de 95% dos binários Android ARMv7 usam instruções compactas **Thumb-2** (16 e 32 bits).

> [!CAUTION]
> Se você tentar desmontar código Thumb-2 com o `objdump` genérico do sistema, ele assumirá o modo ARM padrão de 32 bits, gerando **código desconstruído e inválido**.

Use sempre o toolchain do VitaSDK com a flag `-M force-thumb`:

```bash
arm-vita-eabi-objdump -d -M force-thumb \
  --start-address=0x00045a10 \
  --stop-address=0x00045b80 \
  lib/armeabi-v7a/libgame.so
```

Saída precisa:
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

## 📝 Documentando no `PORTING_PLAN.md`

Registre todas as descobertas em `PORTING_PLAN.md` na raiz do repositório:
- Nome do pacote Java e Title ID escolhido.
- Nome e tamanho da biblioteca `.so`.
- Lista de métodos `Java_*` exportados.
- Métodos de ciclo de vida nativo (`init`, `render`, `touch`, `pause`).
