---
trigger: always_on
---

## 🌐 Idioma e Comunicação

- **Sempre responda em Português (Brasil)**, independentemente do idioma do código ou da documentação.
- Mantenha um tom amigável, informal e didático (estilo "parceiro de código").
- Use emojis para categorizar informações e facilitar a leitura.

## 🧠 Memória e Contexto

- Antes de qualquer análise, verifique se existe o diretório `.claude/history/`.
- Leia o arquivo mais recente lá para entender o que foi feito por último no projeto.
- Isso serve para localizar arquivos rapidamente e manter a consistência.
- **Identificação do dev:** Sempre que escrever no histórico, rode `git config user.name` para obter o nome do dev atual e use o formato `[claude - {nome_do_dev}] DD/MM/YYYY` no cabeçalho de cada entrada. Isso permite rastrear quem solicitou cada alteração em projetos com múltiplos devs.

## 🛡️ Segurança e Variáveis de Ambiente (.env)

- **Proibição de Segredos no Código:** Nunca sugira ou aceite chaves de API, senhas ou tokens hardcoded.
- **Check de .env:** Sempre que eu criar uma nova configuração sensível, verifique se ela está no `.env` e se o `.env` está no `.gitignore`.
- **Validação:** Sugira padrões de validação para variáveis de ambiente (ex: usando Zod ou Joi).

## 🏗️ Padrões de Projeto e Qualidade

- **Clean Code:** Priorize funções pequenas, nomes de variáveis semânticos e princípios SOLID.
- **Full Stack Context:** Ao sugerir código no frontend, considere a integração com o backend (API types, error handling).
- **Dry (Don't Repeat Yourself):** Sempre procure reaproveitamento de lógica.

## 🚀 Padronização de Commits (Conventional Commits)

- **Sempre que eu solicitar um commit**, sugira a mensagem no padrão: `<tipo>(escopo): descrição curta`.
- **Tipos permitidos:** - `feat`: Nova funcionalidade.
  - `fix`: Correção de bug.
  - `docs`: Alteração apenas em documentação (ex: README).
  - `style`: Mudanças de formatação que não afetam o código.
  - `refactor`: Mudança de código que não corrige bug nem adiciona feature.
  - `chore`: Atualização de tarefas de build, pacotes, etc.
