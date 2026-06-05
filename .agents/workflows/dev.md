---
description: Executa o plano e registra a memória da alteração.
---

1. **Pré-voo:** Antes de escrever qualquer código, verifique:
   - O `git status` está limpo (sem arquivos esquecidos no stage)?
   - Existe algum teste quebrando? (rode o comando de test do projeto se disponível)
   - As dependências estão instaladas? (`node_modules`, `venv` ou equivalente)
   - Se qualquer item falhar, **pare e informe** ao dev antes de continuar.
2. Leia o `CURRENT_TASK.md` e execute as tarefas uma por uma, marcando `[x]` ao concluir cada item.
3. Siga as regras de segurança (checar .envs) e o estilo de código definido em `rules.md`.
4. **Critério de aceite:** Antes de concluir, confirme que o comportamento descrito no campo "Critério de aceite" do `CURRENT_TASK.md` está funcionando.
5. Após finalizar, rode `git config user.name` para identificar o dev. Crie ou atualize o log em `.agents/history/{{current_date}}_resumo.md` usando o formato:
   ```
   ## [agente - {nome_do_dev}] DD/MM/YYYY — HH:MM
   **Comando:** /dev
   **Complexidade:** 🟢 Pequena / 🟡 Média / 🔴 Grande
   **Arquivos tocados:**
   - caminho/arquivo.ext (criado | modificado | deletado)
   **O que foi feito:** Resumo em 2-3 linhas.
   **Critério de aceite:** ✅ [descrição do que foi validado]
   **Status:** ✅ Concluído | ⚠️ Parcial | ❌ Bloqueado
   ```
   Se já existe arquivo do dia de hoje, **acrescente** ao final — nunca crie arquivo novo.
6. Delete o arquivo `CURRENT_TASK.md` apenas se todos os itens foram concluídos.
7. Sugira o commit final conforme o padrão definido.
