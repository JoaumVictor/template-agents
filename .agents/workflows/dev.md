---
description: Executa o plano e registra a memória da alteração.
---

1. Leia o `CURRENT_TASK.md` e execute as tarefas uma por uma.
2. Siga as regras de segurança (checar .envs) e o estilo de código definido em `rules.md`.
3. Após finalizar, crie um log em `.agent/history/{{current_date}}_resumo.md` contendo:
   - O que foi feito.
   - Caminho dos arquivos alterados.
4. Delete o arquivo `CURRENT_TASK.md`.
5. Sugira o commit final conforme o padrão definido.