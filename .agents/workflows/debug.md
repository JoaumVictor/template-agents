---
description: Investiga e resolve erros de forma estruturada.
---

1. **Capture o contexto completo:**
   - Mensagem de erro exata (stack trace completo, não resumido).
   - Ambiente onde acontece (local, staging, produção).
   - O erro é determinístico ou intermitente?
   - Rode `git log --oneline -10` para ver o que mudou recentemente.

2. **Leia o histórico:** Consulte `.agents/history/` para verificar se o módulo com erro foi tocado recentemente.

3. **Formule hipóteses** — liste de 2 a 4 causas prováveis em ordem de probabilidade. Não edite código antes disso.

4. **Teste uma hipótese por vez:**
   - Valide a mais provável primeiro.
   - Use logs temporários para confirmar ou descartar.
   - Confirme a causa raiz antes de avançar.

5. **Aplique o fix** com o mínimo de mudança necessário. Não refatore agora — crie um `CURRENT_TASK.md` separado se precisar de refactor depois.

6. **Registre no histórico** em `.agents/history/{{current_date}}_resumo.md`:

   ```
   ## [agente - {nome_do_dev}] DD/MM/YYYY — HH:MM
   **Comando:** /debug
   **Erro:** Descrição curta do erro
   **Causa raiz:** O que estava causando o problema
   **Hipóteses testadas:** (1) hipótese → ✅ confirmada / ❌ descartada
   **Arquivos tocados:**
   - caminho/arquivo.ext (modificado)
   **Fix aplicado:** Descrição do que foi feito
   **Status:** ✅ Resolvido | ⚠️ Mitigado | ❌ Bloqueado
   ```

7. Sugira um commit do tipo `fix:` conforme o padrão definido.
