---
hide:
  - navigation
  - toc
---

<div class="hero-container">
  <div class="hero-title">PS Vita Porting Wiki & Knowledge Base</div>
  <div class="hero-subtitle">
    Enciclopedia técnica definitiva sobre ports de videojuegos móviles (Android APK mediante <code>.so loader</code>, <code>FalsoJNI</code> y <code>vitaGL</code>) para la consola PlayStation Vita.
  </div>

  <div class="badge-row">
    <span class="feature-pill">⚡ 100% Velocidad Nativa ARMv7</span>
    <span class="feature-pill">🎮 Aceleración Hardware SCE GXM</span>
    <span class="feature-pill">🧩 FalsoJNI Runtime Emulation</span>
    <span class="feature-pill">🔬 Triangulación de Crashes con Hardware Real</span>
    <span class="feature-pill">🤖 Optimizado para Humanos & IAs</span>
  </div>

  <div style="margin-top: 1.5rem;">
    <img src="assets/develop-for-vita-assets/psvita-layout.png" alt="PS Vita Hardware" width="580" style="border-radius: 12px; box-shadow: 0 8px 24px rgba(0, 112, 209, 0.25); max-width: 95%;" />
  </div>
</div>

<div class="stats-banner">
  <div class="stat-box">
    <div class="stat-number">30+</div>
    <div class="stat-label">Ports Reales Analizados</div>
  </div>
  <div class="stat-box">
    <div class="stat-number">16</div>
    <div class="stat-label">Capítulos Técnicos</div>
  </div>
  <div class="stat-box">
    <div class="stat-number">3</div>
    <div class="stat-label">Idiomas (ES / EN / PT)</div>
  </div>
  <div class="stat-box">
    <div class="stat-number">0%</div>
    <div class="stat-label">Overhead de Emulación</div>
  </div>
</div>

---

<h2 style="text-align: center; margin-top: 2rem;">🌐 Selecciona tu Idioma / Select Language / Selecione o Idioma</h2>

<div class="lang-grid">

  <!-- Español Card -->
  <div class="lang-card">
    <div>
      <div class="lang-header">
        <span class="lang-flag">🇪🇸</span>
        <h3 class="lang-name">Español</h3>
      </div>
      <p class="lang-desc">
        Manual completo de ingeniería paso a paso: arquitectura ARM Cortex-A9, relocalizaciones ELF, simulación de JVM con FalsoJNI, shaders en vitaGL, triage de volcados <code>.psp2dmp</code> y casos de estudio reales.
      </p>
    </div>
    <a href="es/" class="lang-btn">
      Comenzar en Español <span>&rarr;</span>
    </a>
  </div>

  <!-- English Card -->
  <div class="lang-card">
    <div>
      <div class="lang-header">
        <span class="lang-flag">🇬🇧</span>
        <h3 class="lang-name">English</h3>
      </div>
      <p class="lang-desc">
        Comprehensive low-level porting manual: ARMv7 bare-metal execution, ELF relocation parsing, FalsoJNI runtime dispatch, vitaGL hardware shaders, physical <code>.psp2dmp</code> crash triage, and 30+ engine deep-dives.
      </p>
    </div>
    <a href="en/" class="lang-btn">
      Start in English <span>&rarr;</span>
    </a>
  </div>

  <!-- Português Card -->
  <div class="lang-card">
    <div>
      <div class="lang-header">
        <span class="lang-flag">🇧🇷 🇵🇹</span>
        <h3 class="lang-name">Português</h3>
      </div>
      <p class="lang-desc">
        Manual completo de engenharia passo a passo: arquitetura da CPU Cortex-A9, carregamento de ELF, emulação JNI, pipeline gráfico no vitaGL, triagem de travamentos com <code>vita-parse-core</code> e dissecção de motores.
      </p>
    </div>
    <a href="pt/" class="lang-btn">
      Iniciar em Português <span>&rarr;</span>
    </a>
  </div>

</div>

---

## ⚡ Ecosistema y Repositorios Oficiales

| Herramienta / Proyecto | Repositorio Oficial | Rol en el Porting / Função no Porting |
| :--- | :--- | :--- |
| **VitaSDK** | [github.com/vitasdk](https://github.com/vitasdk) | Toolchain oficial de compilación cruzada GCC (`arm-vita-eabi-gcc`). |
| **vitaGL** | [github.com/Rinnegatamante/vitaGL](https://github.com/Rinnegatamante/vitaGL) | Wrapper acelerado por hardware OpenGL ES 1.1 / 2.0 sobre SCE GXM. |
| **kubridge** | [github.com/TheOfficialFloW/kubridge](https://github.com/TheOfficialFloW/kubridge) | Plugin de kernel para asignar memoria ejecutable (`USER_RWX`) en espacio de usuario. |
| **fd_fix** | [github.com/TheOfficialFloW/fd_fix](https://github.com/TheOfficialFloW/fd_fix) | Corrige el cierre indebido de descriptores de archivos I/O en VitaOS. |
| **FalsoJNI** | [github.com/Rinnegatamante/FalsoJNI](https://github.com/Rinnegatamante/FalsoJNI) | Entorno simulado ligero para satisfacer el JNI de Android sin una JVM pesada. |
| **soloader-boilerplate** | [github.com/Rinnegatamante/soloader-boilerplate](https://github.com/Rinnegatamante/soloader-boilerplate) | Plantilla base de arquitectura para carga de librerías `.so` dinámicas. |
| **psvita-toolkit** | [github.com/metalsyntax/psvita-toolkit](https://github.com/metalsyntax/psvita-toolkit) | CLI automatizado para compilación, despliegue FTP, triage de crashes y shaders. |
