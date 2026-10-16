# PS Vita Porting Wiki & Knowledge Base

<div align="center">
  <img src="assets/develop-for-vita-assets/psvita-layout.png" alt="PS Vita Architecture" width="550" style="border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.15);" />
  <p><em>Enciclopedia técnica definitiva sobre ports de Android (.so loader, FalsoJNI, vitaGL) para PlayStation Vita.</em></p>
</div>

---

## 🌐 Selecciona tu idioma / Select your language

<div class="grid cards" markdown>

-   :es: **[Español (ES) — Entrar a la Wiki](es/index.md)**

    ---

    Documentación completa paso a paso, arquitectura de memoria, emulación de JNI, shaders, triage de crashes con hardware real y análisis de más de 30 motores.

    [:octicons-arrow-right-24: Comenzar en Español](es/index.md)

-   :gb: **[English (EN) — Enter Wiki](en/index.md)**

    ---

    Complete step-by-step engineering manual, memory architecture, JNI emulation, shaders, physical hardware crash triage, and 30+ engine deep-dives.

    [:octicons-arrow-right-24: Start in English](en/index.md)

</div>

---

## ⚡ Enlaces rápidos al Ecosistema / Quick Ecosystem Links

| Herramienta / Proyecto | Repositorio Oficial | Rol en el Porting |
| :--- | :--- | :--- |
| **VitaSDK** | [github.com/vitasdk](https://github.com/vitasdk) | Toolchain de compilación cruzada GCC para PS Vita (`arm-vita-eabi-gcc`). |
| **vitaGL** | [github.com/Rinnegatamante/vitaGL](https://github.com/Rinnegatamante/vitaGL) | Wrapper acelerado por hardware OpenGL ES 1.1 / 2.0 sobre el motor nativo SCE GXM. |
| **kubridge** | [github.com/TheOfficialFloW/kubridge](https://github.com/TheOfficialFloW/kubridge) | Plugin de kernel que permite a procesos de usuario resolver símbolos del kernel e interactuar con memoria. |
| **fd_fix** | [github.com/TheOfficialFloW/fd_fix](https://github.com/TheOfficialFloW/fd_fix) | Corrige el cierre indebido de descriptores de archivos del kernel en VitaOS. |
| **FalsoJNI** | [github.com/Rinnegatamante/FalsoJNI](https://github.com/Rinnegatamante/FalsoJNI) | Entorno simulado ligero para satisfacer el JNI de Android sin ejecutar una máquina virtual Java pesada. |
| **soloader-boilerplate** | [github.com/Rinnegatamante/soloader-boilerplate](https://github.com/Rinnegatamante/soloader-boilerplate) | Plantilla base de arquitectura para carga de librerías `.so` dinámicas en PS Vita. |
| **psvita-toolkit** | [github.com/metalsyntax/psvita-toolkit](https://github.com/metalsyntax/psvita-toolkit) | CLI automatizado para compilación, despliegue FTP, triage de volcados `.psp2dmp`, LiveArea y shaders. |
