<div class="lang-switch-bar">
  <span>🌐 Language / Idioma:</span>
  <a href="../../15-quick-reference-and-cheatsheets/" class="lang-switch-link">🇬🇧 English</a>
  <a href="../../es/15-referencia-rapida-y-cheatsheets/" class="lang-switch-link">🇪🇸 Español</a>
  <a href="./" class="lang-switch-link active">🇧🇷 Português</a>
</div>

<div class="chapter-hero">
  <span class="chapter-badge">Capítulo 15 • Tabelas de Referência</span>
  <h1 style="margin: 0.4rem 0 0.6rem 0; font-size: 2rem;">Referência Rápida e Cheat Sheets</h1>
  <p style="margin: 0; opacity: 0.9; font-size: 1.05rem;">
    Tabelas de consulta ultrarrápida para assinaturas de tipos JNI, códigos hexadecimais de erro do kernel do VitaOS, equivalências Bionic para Newlib e opcodes de patches ARMv7.
  </p>
  <div class="chapter-meta">
    <span class="chapter-meta-pill">⚡ Nível: Referência</span>
    <span class="chapter-meta-pill">📋 Tabelas: Assinaturas JNI, Erros VitaOS, Opcodes ARMv7</span>
    <span class="chapter-meta-pill">🚀 Cheat Sheet Pronto para Copiar</span>
  </div>
</div>

---

Tabelas de consulta rápida para assinaturas JNI, códigos de erro do VitaOS e conversão de chamadas da Bionic libc para o VitaSDK.

---

## 🔡 Tabela Mestra de Assinaturas JNI

| Tipo Java | Símbolo de Assinatura | Tipo C / C++ | Largura em Bits |
|---|---|---|---|
| `boolean` | `Z` | `jboolean` | 8 bits (unsigned) |
| `byte` | `B` | `jbyte` | 8 bits (signed) |
| `char` | `C` | `jchar` | 16 bits (UTF-16 unsigned) |
| `short` | `S` | `jshort` | 16 bits (signed) |
| `int` | `I` | `jint` | 32 bits (signed) |
| `long` | `J` | `jlong` | 64 bits (signed) |
| `float` | `F` | `jfloat` | 32 bits (IEEE 754) |
| `double` | `D` | `jdouble` | 64 bits (IEEE 754) |
| `void` | `V` | `void` | 0 |
| `java.lang.String` | `Ljava/lang/String;` | `jstring` | Ponteiro |
| `java.lang.Class` | `Ljava/lang/Class;` | `jclass` | Ponteiro |
| `int[]` | `[I` | `jintArray` | Ponteiro |
| `byte[]` | `[B` | `jbyteArray` | Ponteiro |
| `String[]` | `[Ljava/lang/String;` | `jobjectArray` | Ponteiro |

---

## ⚠️ Códigos de Erro Comuns no VitaOS

| Código Hexadecimal | Constante no SDK | Causa Comum em Ports | Solução Recomendada |
|---|---|---|---|
| **`0x80010018`** | `SCE_KERNEL_ERROR_NO_MEMORY` | Esgotamento de memória na partição solicitada (RAM ou CDRAM). | Reduza texturas pela metade ou mova texturas secundárias da CDRAM para a RAM principal. |
| **`0x8010113D`** | `SCE_SBL_ERROR_COMMON_INVALID_FORMAT` | Imagens do LiveArea fora do padrão ao instalar o VPK. | Certifique-se de que `icon0.png`, `bg0.png`, `startup.png` sejam PNGs indexados a 8 bits e abaixo do limite de KiB. |
| **`0x800200CB`** | `SCE_KERNEL_ERROR_ILLEGAL_MEMBLOCK_TYPE` | Tentativa de alocar memória executável sem o `kubridge`. | Instale o plugin de kernel `kubridge.skprx` em `ur0:tai/config.txt`. |
| **`0x80020002`** | `SCE_ERROR_ERRNO_ENOENT` | Arquivo ou pasta não encontrado. | Verifique se o caminho começa com `ux0:data/<jogo>/` e se os assets estão no cartão físico. |
| **`0x800200D9`** | `SCE_KERNEL_ERROR_CAN_NOT_ALLOC` | Limite máximo de blocos de memória atingido pelo kernel. | Utilize pools de memória reutilizáveis em vez de centenas de chamadas individuais `sceKernelAllocMemBlock`. |

---

## 🔄 Tabela de Correspondência Bionic libc -> VitaSDK / Newlib

| Função Android (Bionic) | Papel no Android | Equivalente no PS Vita (VitaSDK) |
|---|---|---|
| `__android_log_print` | Envia mensagens para o logcat. | Substitua por logger customizado gravando em arquivo com `fflush` e chamando `sceClibPrintf`. |
| `dlopen(path, flags)` | Carrega biblioteca dinâmica. | Roteie para o carregador interno (`so_file_load`). |
| `dlsym(handle, name)` | Resolve ponteiro de função. | Roteie para `so_resolve(&so_mod, name)`. |
| `pthread_create` | Cria thread nativa POSIX. | Suportado por threads POSIX do VitaSDK ou mapeado para `sceKernelCreateThread`. |
| `mprotect` | Altera permissões de páginas. | Desnecessário se a memória foi alocada inicialmente como `USER_RWX` com o `kubridge`. |
| `gettimeofday` | Lê carimbo de data/hora UNIX. | Totalmente suportado pela Newlib do VitaSDK. |
| `epoll_create` / `inotify` | Eventos assíncronos do Linux. | Não suportado diretamente; simule com thread de sondagem ou retorne descritor dummy. |
