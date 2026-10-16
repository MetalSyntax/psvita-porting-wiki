# Guia de Contribuição / Contributing Guide 🤝

[Español](#español) | [English](#english) | [Português](#português)

---

## Español

¡Gracias por tu interés en aportar a la **PS Vita Porting Wiki**! Esta enciclopedia técnica es un documento vivo que crece con cada nuevo motor analizado, cada crash diagnosticado y cada biblioteca implementada.

### Estructura de Contribuciones
1. **Trilingüe (ES, EN & PT)**: Toda página nueva o sección sustancial debe ser redactada o traducida en las tres versiones (`docs/es/`, `docs/en/` y `docs/pt/`).
2. **Código Real y Riguroso**: Evita pseudocódigo inventado. Incluye extractos de código C/C++ reales, nombres de símbolos comprobados con `objdump`/`nm`, y firmas JNI exactas.
3. **Casos de Estudio de Nuevos Ports**:
   Para añadir la documentación técnica de un nuevo juego o motor portado, añade una sección en `docs/es/12-arqueologia-de-motores.md`, `docs/en/12-engine-archaeology.md` y `docs/pt/12-arqueologia-de-motores.md`.

---

## English

Thank you for contributing to the **PS Vita Porting Wiki**! This technical compendium is an evolving reference powered by developers and AI agents documenting real ports.

### Contribution Guidelines
1. **Trilingual Documentation**: All new chapters and substantial edits should be provided in Spanish (`docs/es/`), English (`docs/en/`), and Portuguese (`docs/pt/`).
2. **Real-World Verifiable Code**: Favor production-tested C snippets, actual linker flags, and verified JNI method descriptors over hypothetical code.
3. **Adding New Game/Engine Analyses**:
   Use the standardized template in `12-engine-archaeology.md` detailing Java packages, native libraries, graphics API, audio stack, critical hooks, and crash pitfalls encountered during the port.

---

## Português

Obrigado pelo seu interesse em contribuir com a **PS Vita Porting Wiki**! Esta enciclopédia técnica é uma referência viva mantida pela comunidade de desenvolvedores e agentes de IA.

### Diretrizes de Contribuição
1. **Documentação Trilíngue**: Novas seções e capítulos devem ser adicionados nas três versões (`docs/es/`, `docs/en/` e `docs/pt/`).
2. **Código Real e Testado**: Utilize trechos em C/C++ verificados no console físico, com assinaturas JNI exatas e instruções de desmonte testadas.
3. **Novos Motores e Jogos**: Adicione estudos de caso em `docs/pt/12-arqueologia-de-motores.md` detalhando particularidades de inicialização, stubs essenciais e patches em memória.
