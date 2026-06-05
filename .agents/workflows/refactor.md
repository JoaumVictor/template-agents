---
description: Melhora a qualidade do código sem alterar a lógica.
---

1. **Mapeie dependências** do código a ser refatorado:
   - Quais arquivos importam ou chamam as funções/componentes que serão alterados?
   - Se a alteração puder causar breaking change, avise explicitamente antes de prosseguir.
2. Analise o código selecionado em busca de complexidade desnecessária, nomes ruins, violações de DRY ou SOLID.
3. Aplique princípios de Clean Code.
4. Não encha de comentários JSDoc se não for solicitado.
5. Mostre o "Antes" e "Depois" para o usuário.
6. Aguarde aprovação antes de aplicar.
7. Se aprovado, aplique e sugira um commit do tipo `refactor:`.
