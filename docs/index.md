---
hide:
  - navigation
  - toc
---

<div class="hero-container">
  <h1 class="hero-title">PS Vita Porting Wiki & Knowledge Base</h1>
  <div class="hero-subtitle">
    The definitive technical manual and engineering reference for porting Android mobile games (via <code>.so loader</code>, <code>FalsoJNI</code>, and <code>vitaGL</code>) to the PlayStation Vita handheld console.
  </div>

  <div class="badge-row">
    <span class="feature-pill">⚡ 100% Native ARMv7 Speed</span>
    <span class="feature-pill">🎮 Hardware-Accelerated SCE GXM</span>
    <span class="feature-pill">🧩 FalsoJNI Runtime Dispatch</span>
    <span class="feature-pill">🔬 Physical Hardware Crash Triage</span>
    <span class="feature-pill">🤖 Engineered for Humans & AI Agents</span>
  </div>

  <div style="margin-top: 1.5rem;">
    <img src="assets/develop-for-vita-assets/psvita-layout.png" alt="PS Vita Hardware" width="580" style="border-radius: 12px; box-shadow: 0 8px 24px rgba(0, 112, 209, 0.25); max-width: 95%;" />
  </div>
</div>

<div class="stats-banner">
  <div class="stat-box">
    <div class="stat-number">30+</div>
    <div class="stat-label">Production Ports Analyzed</div>
  </div>
  <div class="stat-box">
    <div class="stat-number">16</div>
    <div class="stat-label">Core Engineering Chapters</div>
  </div>
  <div class="stat-box">
    <div class="stat-number">3</div>
    <div class="stat-label">Languages Available</div>
  </div>
  <div class="stat-box">
    <div class="stat-number">0%</div>
    <div class="stat-label">CPU Emulation Overhead</div>
  </div>
</div>

---

<h2 style="text-align: center; margin-top: 2rem;">🚀 Porting Engineering Roadmap</h2>

<div class="lang-grid">

  <!-- Core Runtime Card -->
  <div class="lang-card">
    <div>
      <div class="lang-header">
        <span class="lang-flag">🏗️</span>
        <h3 class="lang-name">01. Core Architecture</h3>
      </div>
      <p class="lang-desc">
        Understand why 32-bit ARMv7 Android binaries run natively on the Vita's Cortex-A9 CPU without emulation. Learn ELF userspace mapping, RWX memory allocation via <code>kubridge</code>, and ARM dynamic relocation processing.
      </p>
    </div>
    <a href="00-fundamentals-architecture/" class="lang-btn">
      Explore Architecture <span>&rarr;</span>
    </a>
  </div>

  <!-- JNI & Subsystems Card -->
  <div class="lang-card">
    <div>
      <div class="lang-header">
        <span class="lang-flag">🧩</span>
        <h3 class="lang-name">02. FalsoJNI & Subsystems</h3>
      </div>
      <p class="lang-desc">
        Simulate the JVM without overhead using FalsoJNI. Wire up Android lifecycle hooks, decode method signatures, and implement active stubs for video/billing callbacks that prevent permanent black screens.
      </p>
    </div>
    <a href="05-falsojni-and-java-bridge/" class="lang-btn">
      Explore FalsoJNI <span>&rarr;</span>
    </a>
  </div>

  <!-- Graphics & Shaders Card -->
  <div class="lang-card">
    <div>
      <div class="lang-header">
        <span class="lang-flag">🎨</span>
        <h3 class="lang-name">03. vitaGL & Shaders</h3>
      </div>
      <p class="lang-desc">
        Translate OpenGL ES 1.1 and 2.0 to hardware SCE GXM command buffers with vitaGL. Transpile GLSL to Cg/GXP binaries, resolve FBO Y-flipping, and fix premultiplied alpha texture borders.
      </p>
    </div>
    <a href="06-vitagl-graphics-pipeline/" class="lang-btn">
      Explore Graphics <span>&rarr;</span>
    </a>
  </div>

  <!-- Crash Triage Card -->
  <div class="lang-card">
    <div>
      <div class="lang-header">
        <span class="lang-flag">🔬</span>
        <h3 class="lang-name">04. Hardware Crash Triage</h3>
      </div>
      <p class="lang-desc">
        Master the Triangulation Methodology: parse <code>.psp2dmp</code> crash dumps with <code>vita-parse-core</code>, distinguish PC vs LR call frames, match offsets in Ghidra pseudo-C, and fix ARMv7 unaligned memory traps.
      </p>
    </div>
    <a href="11-crash-triage-and-debugging/" class="lang-btn">
      Explore Triage <span>&rarr;</span>
    </a>
  </div>

  <!-- Real Engine Case Studies Card -->
  <div class="lang-card">
    <div>
      <div class="lang-header">
        <span class="lang-flag">⚔️</span>
        <h3 class="lang-name">05. Engine Archaeology</h3>
      </div>
      <p class="lang-desc">
        Battle-tested case studies from 30+ production ports: Gameloft proprietary engines (Hero of Sparta, DH2), Gamevil Nexus2/Clet (Zenonia 2/3), Com2uS (Inotia 2/3), and Cocos2d-x (Badland 2).
      </p>
    </div>
    <a href="12-engine-archaeology/" class="lang-btn">
      Explore Case Studies <span>&rarr;</span>
    </a>
  </div>

  <!-- AI-Assisted Porting Card -->
  <div class="lang-card">
    <div>
      <div class="lang-header">
        <span class="lang-flag">🤖</span>
        <h3 class="lang-name">06. AI Pair Programming</h3>
      </div>
      <p class="lang-desc">
        Protocols for collaborating with AI coding agents: token economy strategies, filtering 50MB decompilation files, deterministic prompt templates for JNI stubs, and hallucination prevention.
      </p>
    </div>
    <a href="13-ai-assisted-porting/" class="lang-btn">
      Explore AI Guide <span>&rarr;</span>
    </a>
  </div>

</div>

---

## ⚡ Official Ecosystem Repositories

| Project / Tool | Official Repository | Role in Mobile Porting |
| :--- | :--- | :--- |
| **VitaSDK** | [github.com/vitasdk](https://github.com/vitasdk) | Official GCC-based cross-compilation toolchain (`arm-vita-eabi-gcc`). |
| **vitaGL** | [github.com/Rinnegatamante/vitaGL](https://github.com/Rinnegatamante/vitaGL) | Hardware-accelerated OpenGL ES 1.1 / 2.0 translation layer over SCE GXM. |
| **kubridge** | [github.com/TheOfficialFloW/kubridge](https://github.com/TheOfficialFloW/kubridge) | Kernel bridge plugin enabling userspace `USER_RWX` executable memory allocation. |
| **fd_fix** | [github.com/TheOfficialFloW/fd_fix](https://github.com/TheOfficialFloW/fd_fix) | Fixes kernel I/O file descriptor closing crashes in VitaOS. |
| **FalsoJNI** | [github.com/Rinnegatamante/FalsoJNI](https://github.com/Rinnegatamante/FalsoJNI) | Lightweight simulated JNIEnv runtime satisfying Android JNI queries in C. |
| **soloader-boilerplate** | [github.com/Rinnegatamante/soloader-boilerplate](https://github.com/Rinnegatamante/soloader-boilerplate) | Architectural reference template for dynamic `.so` userspace loading. |
| **psvita-toolkit** | [github.com/metalsyntax/psvita-toolkit](https://github.com/metalsyntax/psvita-toolkit) | All-in-one automation CLI for building, FTP deployment, `.psp2dmp` triage, and shaders. |
