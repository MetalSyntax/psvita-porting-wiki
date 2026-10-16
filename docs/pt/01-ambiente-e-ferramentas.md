<div class="chapter-hero">
  <span class="chapter-badge">Capítulo 01 • Ambiente & Toolchains</span>
  <h1 style="margin: 0.4rem 0 0.6rem 0; font-size: 2rem;">Ambiente de Desenvolvimento e Ferramentas</h1>
  <p style="margin: 0; opacity: 0.9; font-size: 1.05rem;">
    Configuração da toolchain cruzada VitaSDK, bibliotecas vdpm (vitaGL, mathneon), suites de engenharia reversa (jadx, Ghidra Docker) e plugins de kernel (kubridge, fd_fix).
  </p>
  <div class="chapter-meta">
    <span class="chapter-meta-pill">⚡ Nível: Fundamental</span>
    <span class="chapter-meta-pill">🛠️ Ferramentas: VitaSDK, CMake, vdpm, jadx, Ghidra</span>
    <span class="chapter-meta-pill">🔌 Kernel: kubridge.skprx, fd_fix.skprx</span>
  </div>
</div>

---

Para desenvolver e depurar um port de Android para o PS Vita, é necessário um ambiente de compilação cruzada na máquina de desenvolvimento (Linux, macOS ou WSL no Windows), ferramentas de engenharia reversa e plugins de kernel instalados no console físico.

---

## 🛠️ Ferramentas na Máquina Host

### 1. VitaSDK (O Toolchain Oficial da Comunidade)
O [VitaSDK](https://github.com/vitasdk) é a cadeia de ferramentas de compilação cruzada baseada em GCC (`arm-vita-eabi-gcc`) para o PlayStation Vita.

#### Instalação Rápida (Linux / macOS):
```bash
export VITASDK="$HOME/vitasdk"
export PATH="$VITASDK/bin:$PATH"

git clone https://github.com/vitasdk/vdpm.git
cd vdpm
./bootstrap-vitasdk.sh
./install-all.sh
```

Adicione as variáveis de ambiente ao seu perfil do terminal (`~/.bashrc` ou `~/.zshrc`):
```bash
export VITASDK="$HOME/vitasdk"
export PATH="$VITASDK/bin:$PATH"
```

Verifique a instalação:
```bash
arm-vita-eabi-gcc --version
# Deve exibir: arm-vita-eabi-gcc (VitaSDK ...)
```

### 2. Bibliotecas Obrigatórias da Comunidade (vdpm)
Instale as bibliotecas mais utilizadas em ports:
```bash
vdpm vitaGL
vdpm mathneon
vdpm libvorbis
vdpm libogg
vdpm openal
vdpm zlib
```

### 3. Ferramentas de Engenharia Reversa
Para inspecionar o APK e os binários nativos:

- **jadx** (Descompila bytecode DEX para código Java limpo):
  - macOS: `brew install jadx`
  - Linux: Baixar dos [Releases do jadx no GitHub](https://github.com/skylot/jadx/releases).
- **Docker + `devrvk/so-decompiler` (Ghidra Headless)**:
  Gera pseudo-C legível a partir de qualquer `.so` ARMv7 via linha de comando:
  ```bash
  docker pull devrvk/so-decompiler
  ```
- **Utilitários Binários do VitaSDK:**
  - `arm-vita-eabi-objdump`: Desmontador com suporte a Thumb-2 e ELF ARM.
  - `arm-vita-eabi-nm`: Extrator de símbolos.
  - `arm-vita-eabi-readelf`: Inspetor de cabeçalhos ELF e seções dinâmicas.

### 4. psvita-toolkit CLI
CLI para automação de tarefas em ports de Vita:
```bash
# Diagnóstico do ambiente
psvita-toolkit doctor

# Compilação com presets do CMake
psvita-toolkit build --preset debug

# Envio instantâneo do eboot.bin via FTP
psvita-toolkit deploy --eboot --yes
```

---

## 🎮 Plugins de Kernel Obrigatórios no PS Vita

Para que o VitaOS permita a execução de código carregado dinamicamente no espaço de usuário, é **obrigatório** ter os seguintes plugins instalados em `ur0:tai/config.txt`:

### 1. kubridge (por TheFloW)
Repositório: [github.com/TheOfficialFloW/kubridge](https://github.com/TheOfficialFloW/kubridge)  
*Por que é essencial?*  
O VitaOS possui proteção W^X que impede processos em espaço de usuário de alocar memória com permissões simultâneas de Leitura, Escrita e Execução (**RWX**). O `kubridge` expõe chamadas como `kuKernelAllocMemBlock`, permitindo que o loader aloque a memória executável necessária para o arquivo `.so`.

### 2. fd_fix (por TheFloW)
Repositório: [github.com/TheOfficialFloW/fd_fix](https://github.com/TheOfficialFloW/fd_fix)  
*Por que é essencial?*  
Previne falhas de kernel e corrupção de I/O quando o jogo fecha descritores de arquivos duplicados ou inválidos, mantendo a semântica esperada pela Bionic libc do Android.

```ini
# Em ur0:tai/config.txt
*KERNEL
ur0:tai/kubridge.skprx
ur0:tai/fd_fix.skprx
```

> [!CAUTION]
> Se você tentar rodar um port baseado em SoLoader em um console sem o `kubridge.skprx` instalado, o jogo fechará imediatamente na inicialização antes de executar qualquer linha de código do motor.
