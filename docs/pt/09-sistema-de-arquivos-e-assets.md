<div class="lang-switch-bar">
  <span>🌐 Language / Idioma:</span>
  <a href="../../09-file-system-and-assets/" class="lang-switch-link">🇬🇧 English</a>
  <a href="../../es/09-sistema-de-archivos-y-assets/" class="lang-switch-link">🇪🇸 Español</a>
  <a href="./" class="lang-switch-link active">🇧🇷 Português</a>
</div>

<div class="chapter-hero">
  <span class="chapter-badge">Capítulo 09 • Armazenamento & Arquivos</span>
  <h1 style="margin: 0.4rem 0 0.6rem 0; font-size: 2rem;">Sistema de Arquivos e Gestão de Assets</h1>
  <p style="margin: 0; opacity: 0.9; font-size: 1.05rem;">
    Redirecionamento de caminhos Linux ext4 (/sdcard/, /data/data/) para partições do PS Vita (ux0:, app0:), sanitização transparente de I/O e mitigação de cluster slack FAT32.
  </p>
  <div class="chapter-meta">
    <span class="chapter-meta-pill">⚡ Nível: Intermediário</span>
    <span class="chapter-meta-pill">📁 Partições: ux0: (Leitura/Escrita), app0: (Apenas Leitura)</span>
    <span class="chapter-meta-pill">🎮 Verificado: Hero of Sparta, Zenonia 3, Badland 2</span>
  </div>
</div>

---

A topologia de armazenamento e as partições do PlayStation Vita diferem substancialmente do ambiente Android Linux. O redirecionamento de caminhos e o correto empacotamento dos arquivos de dados garantem que o jogo encontre seus modelos, texturas e arquivos de progresso (*saves*).

---

## 💾 Topologia de Armazenamento no VitaOS

```mermaid
graph TD
    subgraph "Partições do VitaOS"
        UX0["ux0: (Cartão de Memória Principal / SD2Vita)"]
        APP0["app0: (Montagem somente leitura da aplicação atual)"]
        UR0["ur0: (Memória do sistema / Plugins taiHEN)"]
    end

    subgraph "Tradução de Caminhos"
        AndroidPath["Caminho Android: /sdcard/Android/data/com.jogo/"]
        VitaDataPath["Destino Real: ux0:data/meu_jogo/"]
    end

    AndroidPath -->|sanitize_path()| VitaDataPath
    VitaDataPath --> UX0
    APP0 -->|Contém eboot.bin e LiveArea| UX0
```

### Papel das Partições:
- **`app0:` (Somente Leitura):** Diretório de instalação do app (`ux0:app/<TITLEID>/`). Contém o `eboot.bin` e metadados. **Qualquer tentativa de escrita em `app0:` resulta em erro imediato de permissão.**
- **`ux0:data/<jogo>/` (Leitura e Escrita):** Localização padrão da comunidade homebrew para dados do jogo, arquivos de progresso (`saves/`) e registros de log (`logs/`).

---

## 🔀 Redirecionamento de Caminhos POSIX

Jogos Android utilizam caminhos fixos do Linux:
- `/sdcard/Android/data/com.gameloft.spaw/files/...`
- `/data/data/com.gamevil.zenonia3/...`

No carregador (`io.c`), intercepte as funções de I/O (`fopen`, `open`, `stat`, `mkdir`):

```c
#define DATA_PATH "ux0:data/heroofsparta/"

char *sanitize_path(const char *path, char *out_path, size_t max_len) {
    if (!path) return NULL;
    
    if (strncmp(path, "ux0:", 4) == 0 || strncmp(path, "app0:", 5) == 0) {
        strncpy(out_path, path, max_len);
        return out_path;
    }
    
    const char *p = path;
    if (strncmp(p, "/sdcard/", 8) == 0) p += 8;
    else if (strncmp(p, "/storage/emulated/0/", 20) == 0) p += 20;
    else if (*p == '/') p++;
    
    snprintf(out_path, max_len, "%s%s", DATA_PATH, p);
    return out_path;
}

FILE *vita_fopen(const char *filename, const char *mode) {
    char clean_path[1024];
    sanitize_path(filename, clean_path, sizeof(clean_path));
    return fopen(clean_path, mode);
}
```

---

## 📦 Estratégias de Empacotamento de Assets

| Estratégia | Vantagens | Desvantagens |
|---|---|---|
| **1. Arquivos Soltos Descompactados** | Fácil de inspecionar e modificar texturas e áudio. | O tamanho de cluster do FAT32 (32KB/64KB) desperdiça centenas de megabytes em jogos com milhares de arquivos pequenos (*slack space*). |
| **2. Leitura Direta de APK/OBB (ZIP na RAM)** | Economiza espaço considerável; pacote único comprimido. Leituras mais rápidas no cartão. | Requer suporte a ZIP nativo no motor (ex: `CCFileUtils` no Cocos2d-x) ou vinculação com `libzip` / `minizip`. |

---

## 🔠 Sensibilidade a Maiúsculas e Minúsculas (FAT vs ext4)

- **Android (ext4):** Estritamente sensível a maiúsculas e minúsculas (`Textura.PNG` != `textura.png`).
- **PS Vita (FAT32 / exFAT):** Preserva maiúsculas, mas a busca é insensível.

> [!WARNING]
> Se o APK contiver arquivos com o mesmo nome diferenciados apenas por maiúsculas na mesma pasta, ao copiá-los para o Vita um sobrescreverá o outro.
