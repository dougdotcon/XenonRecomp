# XenonRecomp

**XenonRecomp** é uma ferramenta especializada projetada para modernizar softwares legados do Xbox 360. Ela converte executáveis PowerPC (PPC) do Xbox 360 em código-fonte C++, facilitando a recompilação Ahead-Of-Time (AOT) da lógica do jogo para execução nativa em plataformas x86.

Inspirado na metodologia utilizada no [N64Recomp](https://github.com/N64Recomp/N64Recomp), este projeto foca na análise estática e tradução de código de máquina em uma representação de alto nível.

> **⚠️ Aviso Legal:** O XenonRecomp fornece *apenas tradução de código*. Ele não inclui um ambiente de tempo de execução (runtime), camada de abstração de hardware (HAL) ou wrappers de API. Para executar o código gerado com sucesso, é necessário construir manualmente um runtime que implemente as interfaces de hardware do Xbox 360 (ex: GPU, I/O, Áudio).

## Arquitetura Principal

### Tradução de Instruções
O recompiler analisa as instruções PowerPC e as traduz diretamente em operações C++. Para manter o contexto de execução, as funções geradas aceitam uma estrutura `CpuState` que contém o valor de todos os registradores PPC, junto com um ponteiro para o endereço base do espaço de memória emulado.

*   **Endianness (Ordem dos Bytes):** O Xbox 360 é um sistema *big-endian*, enquanto o x86 moderno é *little-endian*. O XenonRecomp resolve isso:
    *   Trocando a ordem dos bytes para operações padrão de carga e armazenamento de memória.
    *   Marcando essas operações como `volatile` para evitar reordenação pelo compilador.
    *   Revertendo o layout de 16 bytes de registradores vetoriais (VMX/AltiVec) para corresponder aos requisitos de processamento SIMD.

### Ponto Flutuante e Processamento Vetorial
*   **FPU vs. VMX:** A ferramenta diferencia entre operações padrão de FPU (que preservam números denormalizados) e operações VMX (que convertem denormalizados para zero). O código gerado gerencia as flags de controle de ponto flutuante da CPU dinamicamente para garantir precisão na emulação.
*   **Implementação SIMD:** A maioria das instruções VMX são mapeadas diretamente para intrínsecos x86 (SSE/AVX). Embora atualmente otimizado para x86, a lógica pode ser teoricamente adaptada para outras arquiteturas usando bibliotecas como [SIMD Everywhere](https://github.com/simd-everywhere/simde).

### Memória e Chamadas Indiretas
*   **MMIO (Entrada/Saída Mapeada em Memória):** Operações específicas de hardware (como decodificação de áudio XMA) não estão implementadas atualmente e exigem suporte manual de runtime.
*   **Chamadas de Funções Virtuais:** A ferramenta lida com vtables de C++ gerando "hash tables perfeitas" em tempo de execução. Isso permite a resolução dinâmica de chamadas virtuais sem exigir um linkador dinâmico completo.

## Visão Geral do Fluxo de Trabalho

1.  **Análise:** O XenonRecomp analisa o executável de entrada para identificar segmentos de código e fluxo de controle.
2.  **Conversão:** Gera arquivos de código-fonte C++ representando a lógica do jogo, envolvendo instruções em chamadas de função para o `CpuState`.
3.  **Compilação:** O código C++ gerado é compilado usando um compilador C++ padrão (ex: MSVC, Clang, GCC) direcionado para a arquitetura do host.
4.  **Integração de Runtime:** Os arquivos de objeto resultantes são vinculados a um runtime personalizado que implementa gerenciamento de memória e callbacks de hardware.

## Construindo

*(Nota: Instruções de construção devem ser adicionadas aqui com base no sistema de construção específico usado, tipicamente CMake ou Make.)*

## Contribuindo

Contribuições são bem-vindas, particularmente nas áreas de:
*   Implementação de variantes de instruções PowerPC faltantes.
*   Expansão do suporte para operações MMIO.
*   Otimização da saída C++ gerada para melhores otimizações pelo compilador.