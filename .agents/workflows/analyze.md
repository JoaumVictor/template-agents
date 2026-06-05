---
description: Mapeia a arquitetura do projeto e gera diagnóstico de saúde.
---

1. Varra o diretório atual e identifique as tecnologias (React, Node, SQL, etc.).
2. Analise a estrutura de pastas e padrões de nomenclatura.
3. Crie (ou atualize) o arquivo `.agents/PROJECT_MAP.md` com:
   - **Stack Tecnológica.**
   - **Estrutura de Pastas.**
   - **Onde ficam os componentes/rotas/lógica.**
4. **Diagnóstico de saúde** — identifique e reporte:
   - 🔥 **Hot files:** arquivos com múltiplas responsabilidades ou muita lógica concentrada
   - 🔗 **Acoplamento alto:** módulos que importam muitos outros ou são importados por todo o projeto
   - 🧪 **Gaps de teste:** áreas críticas sem cobertura aparente
   - 💀 **Código morto:** exports, funções ou arquivos aparentemente não utilizados
   - ⚠️ **Riscos:** padrões que podem virar problema
5. Resuma o que entendeu do projeto e apresente o diagnóstico em ordem de prioridade.
