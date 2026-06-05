---
description: Planeja a próxima tarefa e cria um checklist.
---

Não use a interface de 'Implementation Plan' da IDE. Escreva todas as tarefas obrigatoriamente no arquivo CURRENT_TASK.md dentro da pasta do projeto.

1. Verifique se existe um `CURRENT_TASK.md` ativo. Se houver algo em andamento, pergunte se devo pausar ou concluir antes de seguir.
2. Consulte `.agents/history/` para entender o contexto das últimas tarefas.
3. **Classifique a complexidade** do pedido:
   - 🟢 **Pequena** — mudança isolada em 1-2 arquivos, sem impacto em outros módulos
   - 🟡 **Média** — afeta 3-5 arquivos ou requer integração entre módulos
   - 🔴 **Grande** — afeta fluxos críticos, múltiplos módulos, banco ou segurança. Divida em subtarefas com checkpoints separados.
4. Crie o arquivo `CURRENT_TASK.md` com:
   - **Complexidade:** 🟢 Pequena / 🟡 Média / 🔴 Grande
   - Checklist de implementação (itens marcáveis com `[ ]`).
   - Arquivos que serão afetados.
   - **Critério de aceite:** descrição objetiva de quando a tarefa estará 100% pronta (comportamento esperado, não apenas "código escrito").
   - Variáveis de ambiente necessárias (se houver).
5. Peça autorização para começar a execução (/dev).
