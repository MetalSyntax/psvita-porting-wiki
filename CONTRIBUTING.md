# Guía de Contribución / Contributing Guide 🤝

[Español](#español) | [English](#english)

---

## Español

¡Gracias por tu interés en aportar a la **PS Vita Porting Wiki**! Esta enciclopedia técnica es un documento vivo que crece con cada nuevo motor analizado, cada crash diagnosticado y cada biblioteca implementada.

### Estructura de Contribuciones

Para mantener la máxima calidad técnica y legibilidad tanto para ingenieros humanos como para agentes de IA:

1. **Bilingüe (ES & EN)**: Toda página nueva o sección sustancial debe ser redactada o traducida en ambas versiones (`docs/es/` y `docs/en/`).
2. **Código Real y Riguroso**: Evita pseudocódigo inventado. Incluye extractos de código C/C++ reales, nombres de símbolos comprobados con `objdump`/`nm`, y firmas JNI exactas.
3. **Casos de Estudio de Nuevos Ports**:
   Para añadir la documentación técnica de un nuevo juego o motor portado, añade una sección en `docs/es/12-arqueologia-de-motores.md` y `docs/en/12-engine-archaeology.md` con la siguiente plantilla estándar:

```markdown
### [Nombre del Juego / Motor]
- **Paquete Java:** `com.desarrollador.juego`
- **Librerías NDK (.so):** `libjuego.so`, `libengine.so`
- **Gráficos:** GLES1 (Fixed Function) o GLES2 (Shaders)
- **Audio:** OpenSL ES / SoundPool / OpenAL
- **Particularidades del Loader:**
  - Desafío 1 y resolución (ej. manejo de texturas PVRTC o ETC1).
  - Stubs críticos (callbacks que si no se responden congelan el juego).
  - Parches binarios aplicados a offsets específicos del `.so`.
- **Estructura de Assets:** Ubicación de archivos, empaquetado OBB / APK.
```

### Protocolo para Agentes de IA

Si eres un modelo de lenguaje o agente autónomo (Antigravity, Claude, GPT, etc.) agregando documentación:
- **No inventes símbolos JNI ni offsets de memoria**. Verifica siempre con el archivo `.so` y la decompilación `jadx`/Ghidra existente.
- Usa alertas estándar de GitHub (`> [!NOTE]`, `> [!TIP]`, `> [!WARNING]`, `> [!IMPORTANT]`).
- Los diagramas de flujo y arquitectura deben expresarse en formato **Mermaid**.

---

## English

Thank you for contributing to the **PS Vita Porting Wiki**! This technical compendium is an evolving reference powered by developers and AI agents documenting real ports.

### Contribution Guidelines

1. **Bilingual Documentation**: All new chapters and substantial edits should be provided in both Spanish (`docs/es/`) and English (`docs/en/`).
2. **Real-World Verifiable Code**: Favor production-tested C snippets, actual linker flags, and verified JNI method descriptors over hypothetical code.
3. **Adding New Game/Engine Analyses**:
   Use the standardized template in `docs/en/12-engine-archaeology.md` detailing Java packages, native libraries, graphics API, audio stack, critical hooks, and crash pitfalls encountered during the port.
