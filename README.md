# PS Vita Porting Wiki & Knowledge Base 🎮📖

[![Deploy to GitHub Pages](https://github.com/metalsyntax/psvita-porting-wiki/actions/workflows/deploy.yml/badge.svg)](https://github.com/metalsyntax/psvita-porting-wiki/actions/workflows/deploy.yml)
[![License: MIT & CC-BY-4.0](https://img.shields.io/badge/License-MIT%20%26%20CC--BY--4.0-blue.svg)](LICENSE)
[![Bilingual: ES & EN](https://img.shields.io/badge/Languages-Espa%C3%B1ol%20%7C%20English-green.svg)](#)
[![Built with: Material for MkDocs](https://img.shields.io/badge/Material_for_MkDocs-7289DA?logo=materialformkdocs&logoColor=white)](https://squidfunk.github.io/mkdocs-material/)

---

### [Español](#español) | [English](#english)

---

## Español

Bienvenido a la **PS Vita Porting Wiki**, la base de conocimiento y enciclopedia técnica más completa, detallada y profunda creada para el desarrollo de ports de videojuegos móviles (Android APK mediante la técnica de `.so loader`, wrappers JNI, y recreación de subsistemas nativos) para la consola **PlayStation Vita**.

Esta wiki está redactada con un **lenguaje natural, riguroso y técnico**, diseñada tanto para **desarrolladores humanos** (de nivel principiante a avanzado) como para **agentes de Inteligencia Artificial (IAs)** que requieran instrucciones exactas, matrices de decisión, firmas de tipo de bajo nivel y arquitectura de código ejecutable sin ambigüedades.

### 🌟 Fuentes de Inspiración y Respaldo Real

1. **Guía Base de gl33ntwine**: Inspirada y construida a partir del legendario trabajo *"Getting Started with PSVita Homebrew Development"* de [Vitaliy Atamanenko (gl33ntwine)](https://web.archive.org/web/20241206184722/https://gl33ntwine.com/posts/develop-for-vita/), preservado y actualizado con anotaciones técnicas contemporáneas.
2. **Experiencia Práctica en más de 30 Ports Reales**: Todo el conocimiento aquí vertido proviene del desarrollo, depuración y finalización de ports reales en consola física:
   - *Gameloft Engine Era*: **Hero of Sparta**, **Dungeon Hunter 2 & 3**, **N.O.V.A. 2**, **Shadow Guardian**, **Gangstar Rio / Miami**, **Eternal Legacy**, **Six-Guns**, **Brothers in Arms 2**, **Asphalt 5 & 6**.
   - *Gamevil Nexus2/Clet*: **Zenonia 2, 3 y 4**, **Destinia**, **ILLUSIA 1 y 2**, **Advena**.
   - *Com2uS Engine*: **Inotia 2**, **Inotia 3**, **Inmortal Dusk**.
   - *Motores Propietarios y 2D/3D*: **Badland 2**, **Resident Evil Degeneration**, **Rayman Jungle Run**, **Real Racing 2**, **Riptide GP**, **Mike V Skateboard Party**, **Snail Mail**, **The Impossible Game**, **PES 2011 & 2012**.
3. **Ecosistema Oficial**: Conexión directa y documentación de los proyectos nucleares de la escena:
   - [VitaSDK Toolchain](https://github.com/vitasdk)
   - [vitaGL](https://github.com/Rinnegatamante/vitaGL) (Rinnegatamante)
   - [kubridge](https://github.com/TheOfficialFloW/kubridge) (TheFloW)
   - [fd_fix](https://github.com/TheOfficialFloW/fd_fix) (TheFloW)
   - [FalsoJNI](https://github.com/Rinnegatamante/FalsoJNI) (Rinnegatamante)
   - [soloader-boilerplate](https://github.com/Rinnegatamante/soloader-boilerplate) (Rinnegatamante)
   - [psvita-toolkit / psvita-port-toolkit](https://github.com/metalsyntax/psvita-toolkit)

---

### 🗺️ Estructura de la Wiki (Español)

1. [**00. Fundamentos y Arquitectura**](docs/es/00-fundamentos-arquitectura.md): CPU ARM Cortex-A9, GPU PowerVR SGX543MP4+, Particiones de RAM (512MB) y VRAM CDRAM (112MB). ¿Qué es un `.so loader` y por qué ejecuta nativo en Vita?
2. [**01. Entorno y Herramientas**](docs/es/01-entorno-y-herramientas.md): VitaSDK, CMake, Ninja, Docker + Ghidra headless (`devrvk/so-decompiler`), jadx, psvita-toolkit, transferencias FTP y plugins de kernel requeridos.
3. [**02. Triaje y Selección de APK**](docs/es/02-triaje-y-seleccion-apk.md): Anatomía del APK, análisis de ABIs (`armeabi`, `armeabi-v7a`), GLES1 vs GLES2, dependencias de librerías (`libc++`, OpenAL, FMOD) y matriz de viabilidad.
4. [**03. Análisis Estático e Ingeniería Inversa**](docs/es/03-analisis-estatico-e-ingenieria-inversa.md): Decompilación automatizada de Java con `jadx`, generación de pseudo-C con Ghidra, volcado de tablas de símbolos con `objdump -T` y `nm`.
5. [**04. Arquitectura SoLoader y Relocalizaciones**](docs/es/04-arquitectura-soloader-y-relocalizaciones.md): Carga de binarios ELF en espacio de usuario, asignación de bloques RWX (`sceKernelAllocMemBlock`), resolución de relocalizaciones ARM (`R_ARM_RELATIVE`, `R_ARM_GLOB_DAT`, `R_ARM_JUMP_SLOT`), tablas de parches e intercepción de funciones.
6. [**05. FalsoJNI y Puente Java**](docs/es/05-falsojni-y-puente-java.md): Simulación del entorno de máquina virtual Java (`JavaVM`, `JNIEnv`), firmas de métodos JNI (`(ILjava/lang/String;)V`), registro de funciones nativas, gestión de UTF-8 / UTF-16 y trampas de stubs (stubs activos vs no-ops peligrosos que cuelgan motores).
7. [**06. Pipeline Gráfico con vitaGL**](docs/es/06-pipeline-grafico-vitagl.md): Inicialización de vitaGL sobre SCE GXM, configuración del framebuffer (960x544), conversión de shaders GLSL a Cg/GXP (`psp2cgc`, `spirv-cross`), alfa premultiplicado y gestión de memoria CDRAM.
8. [**07. Subsistema de Audio**](docs/es/07-subsistema-de-audio.md): `sceAudioOut`, OpenAL-Soft para PS Vita, decodificación de audio comprimido (Vorbis/MP3), puentes para emular Android `AudioTrack`, `SoundPool` y `MediaPlayer`.
9. [**08. Controles y Sistema de Entrada**](docs/es/08-controles-e-input.md): Pantalla táctil delantera y trasera, botones físicos y joysticks (`SceCtrlData`). La trampa mortal de corrupción de memoria por paso del ID de reporte táctil crudo y cómo emular pads táctiles virtuales.
10. [**09. Sistema de Archivos y Assets**](docs/es/09-sistema-de-archivos-y-assets.md): Mapeo a `ux0:data/<titleid>/`, redirección de llamadas POSIX (`fopen`, `stat`), insensibilidad a mayúsculas/minúsculas de FAT/exFAT, y lectura directa de assets desde APK/OBB vía ZIP en memoria.
11. [**10. LiveArea y Empaquetado VPK**](docs/es/10-livearea-y-empaquetado-vpk.md): Reglas estrictas de formato PNG indexado a 8-bit (`bg0.png`, `startup.png`, `icon0.png`, `pic0.png`) para evitar el error `0x8010113D`, estructura de `param.sfo`, `template.xml` y limpieza de basura `._*` de macOS.
12. [**11. Triage de Crashes y Hardware Debugging**](docs/es/11-triage-de-crashes-y-debugging.md): Metodología de triangulación: volcado `.psp2dmp` + `vita-parse-core` + distinción `PC` vs `LR` + resolución de offsets contra `.so` y pseudo-C de Ghidra. Trampas de desalineación de memoria ARMv7 (`ldrd`/`strd`).
13. [**12. Arqueología de Motores Reales**](docs/es/12-arqueologia-de-motores.md): Casos de estudio pormenorizados de motores Gameloft, Gamevil Nexus2/Clet, Com2uS e implementaciones Cocos2d-x personalizadas.
14. [**13. Guía de Porting Asistido con IA**](docs/es/13-guia-porting-con-ia.md): Patrones de colaboración Humano + IA, economía de tokens, prompts blindados para generación de stubs y prevención de alucinaciones en ingeniería inversa.
15. [**14. Guía Base (gl33ntwine)**](docs/es/14-guia-base-gl33ntwine.md): Texto íntegro traducido y adaptado de la guía histórica de Vitaliy Atamanenko.
16. [**15. Referencia Rápida y Cheat Sheets**](docs/es/15-referencia-rapida-y-cheatsheets.md): Tablas de tipos JNI, códigos de error habituales de VitaOS, y correspondencias de Bionic libc a Newlib de VitaSDK.

---

## English

Welcome to the **PS Vita Porting Wiki**, the definitive technical reference and knowledge base dedicated to porting mobile games (Android APKs via `.so loader`, JNI simulation, and native subsystem recreation) to the **PlayStation Vita** handheld console.

This wiki is written in a **natural, precise, and deeply technical tone**, catering equally to **human software engineers** and **AI developer agents** seeking unambiguous specifications, exact function signatures, memory safety guidelines, and reproducible workflows.

### 🗺️ Wiki Structure (English)

1. [**00. Fundamentals & Architecture**](docs/en/00-fundamentals-architecture.md): ARM Cortex-A9 CPU, PowerVR SGX543MP4+ GPU, RAM (512MB) and VRAM CDRAM (112MB) partitions. What is a `.so loader` and why does it run natively on Vita?
2. [**01. Environment & Toolchains**](docs/en/01-environment-and-tools.md): VitaSDK, CMake, Ninja, Docker + Headless Ghidra (`devrvk/so-decompiler`), jadx, psvita-toolkit, FTP deployment, and mandatory kernel plugins.
3. [**02. APK Triage & Viability Analysis**](docs/en/02-apk-triage-and-selection.md): APK anatomy, ABI differences (`armeabi`, `armeabi-v7a`), GLES1 vs GLES2, library dependencies (`libc++`, OpenAL, FMOD), and the viability matrix.
4. [**03. Static Analysis & Reverse Engineering**](docs/en/03-static-analysis-and-reverse-engineering.md): Automated Java decompilation with `jadx`, pseudo-C generation via Ghidra, dynamic symbol table dumps using `objdump -T` and `nm`.
5. [**04. SoLoader Core & Relocations**](docs/en/04-soloader-architecture-and-relocations.md): Userspace ELF binary loading, RWX block allocation (`sceKernelAllocMemBlock`), ARM relocation mechanics (`R_ARM_RELATIVE`, `R_ARM_GLOB_DAT`, `R_ARM_JUMP_SLOT`), patch tables, and symbol hooking.
6. [**05. FalsoJNI & Java Bridge**](docs/en/05-falsojni-and-java-bridge.md): Simulating the JVM/JNIEnv environment, JNI type signatures (`(ILjava/lang/String;)V`), registering native callbacks, UTF string conversions, and stub safety (active stubs vs game-freezing no-ops).
7. [**06. Graphics Pipeline with vitaGL**](docs/en/06-vitagl-graphics-pipeline.md): Initializing vitaGL on top of SCE GXM, framebuffer setup (960x544), shader translation from GLSL to Cg/GXP (`psp2cgc`, `spirv-cross`), premultiplied alpha, and CDRAM management.
8. [**07. Audio Subsystem**](docs/en/07-audio-subsystem.md): `sceAudioOut`, OpenAL-Soft for PS Vita, compressed audio decoding (Vorbis/MP3), and bridging Android `AudioTrack`, `SoundPool`, and `MediaPlayer`.
9. [**08. Controls & Input Mapping**](docs/en/08-controls-and-input.md): Front and rear touchscreens, physical buttons and analog sticks (`SceCtrlData`). The infamous touch report ID heap corruption trap and virtual touch pad emulation.
10. [**09. File System & Asset Management**](docs/en/09-file-system-and-assets.md): Mapping to `ux0:data/<titleid>/`, POSIX path redirection (`fopen`, `stat`), FAT/exFAT case insensitivity quirks, and in-memory APK/OBB asset decompression.
11. [**10. LiveArea & VPK Packaging**](docs/en/10-livearea-and-vpk-packaging.md): Strict 8-bit indexed PNG requirements (`bg0.png`, `startup.png`, `icon0.png`, `pic0.png`) to eliminate error `0x8010113D`, `param.sfo` structure, `template.xml`, and macOS `._*` cleanup.
12. [**11. Crash Triage & Hardware Debugging**](docs/en/11-crash-triage-and-debugging.md): The triangulation methodology: `.psp2dmp` crash dump + `vita-parse-core` + `PC` vs `LR` resolution + Ghidra disassembly matching. Resolving ARMv7 unaligned memory access crashes.
13. [**12. Real Engine Archaeology**](docs/en/12-engine-archaeology.md): Detailed case studies covering 30+ production ports: Gameloft proprietary engines, Gamevil Nexus2/Clet, Com2uS, and custom Cocos2d-x setups.
14. [**13. AI-Assisted Porting Guide**](docs/en/13-ai-assisted-porting.md): Human + AI pair programming protocols, token economy strategies, prompt templates for JNI stub generation, and reverse-engineering hallucination mitigation.
15. [**14. Base Guide (gl33ntwine)**](docs/en/14-base-guide-gl33ntwine.md): Full original text by Vitaliy Atamanenko annotated with modern homebrew practices.
16. [**15. Quick Reference & Cheat Sheets**](docs/en/15-quick-reference-and-cheatsheets.md): JNI type signature tables, common PS Vita kernel error codes, and Android Bionic libc to VitaSDK Newlib translation tables.

---

## 🚀 Publicación en GitHub Pages / Publishing to GitHub Pages

Este repositorio está preconfigurado para compilarse y publicarse automáticamente con **GitHub Actions** y **MkDocs Material**:

1. Sube este repositorio a tu cuenta de GitHub (por ejemplo `https://github.com/tu-usuario/psvita-porting-wiki`).
2. En GitHub, dirígete a **Settings** > **Pages**.
3. En **Build and deployment** > **Source**, selecciona **GitHub Actions**.
4. Cada vez que hagas `git push` a la rama `main`, la GitHub Action `.github/workflows/deploy.yml` compilará la wiki con soporte completo de búsqueda, modo oscuro/claro, resaltado de sintaxis y diagramas Mermaid.

Para previsualizar localmente (requiere Python 3):
```bash
pip install mkdocs-material
mkdocs serve
```

---

## 🤝 Cómo Contribuir / How to Contribute

Consulta [CONTRIBUTING.md](CONTRIBUTING.md) para agregar nuevos análisis de motores, notas sobre bugs resueltos, plantillas de stubs o correcciones de documentación.
