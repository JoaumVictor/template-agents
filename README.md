# 🤖 Template Agents

Repositório central de configurações de **Agentes de IA** para seus projetos. Mantenha-o sempre atualizado e copie a pasta correspondente ao seu setup sempre que iniciar ou atualizar um projeto.

## 🎯 Objetivo

Garantir padrão de qualidade, segurança e produtividade em todos os projetos, com **Regras de Ouro** (zero secrets, clean code, commits padronizados) e workflows customizados (`/analyze`, `/propose`, `/dev`, `/refactor`, `/commit`) sempre na versão mais recente.

---

## 🚀 Como utilizar

Clone este repositório (ou `git pull` para atualizar) e copie a pasta correspondente à sua ferramenta:

### 🟣 Usando Claude (Anthropic via Claude.ai / API direta)

Copie a pasta **`.claude`** para a raiz do seu projeto:

```bash
cp -r .claude /caminho/do/seu/projeto/
```

Os comandos ficam em `.claude/commands/` e são ativados digitando `/nome-do-comando` no chat.

### 🔵 Usando Copilot / Cursor / Windsurf / outro agente com suporte a `.agents`

Copie a pasta **`.agents`** para a raiz do seu projeto:

```bash
cp -r .agents /caminho/do/seu/projeto/
```

Os workflows ficam em `.agents/workflows/` e as regras em `.agents/rules/`.

---

## 🛠️ Comandos disponíveis (funcionam nos dois setups)

| Comando         | O que faz                                              | Quando usar                                     |
| :-------------- | :----------------------------------------------------- | :---------------------------------------------- |
| **`/analyze`**  | Mapeia a arquitetura e tecnologias do projeto          | Na primeira abertura ou após grandes mudanças   |
| **`/propose`**  | Cria um plano de ação (checklist) em `CURRENT_TASK.md` | Antes de começar qualquer feature ou correção   |
| **`/dev`**      | Executa o plano e registra o histórico em `history/`   | Após o `/propose` ser aprovado                  |
| **`/refactor`** | Melhora legibilidade e performance sem mudar a lógica  | Quando o código funciona mas precisa de um tapa |
| **`/commit`**   | Revisa segurança e sugere mensagem de commit           | Sempre antes de commitar                        |

---

## 📂 Estrutura das pastas

```
.claude/          → Para Claude (Anthropic)
  commands/       → Definição dos comandos /slash
  history/        → Memória de longo prazo (um arquivo por dia)
  rules.md        → Regras sempre ativas (idioma, segurança, padrões)

.agents/          → Para outros agentes (Copilot, Cursor, etc.)
  workflows/      → Fluxos passo a passo por tarefa
  rules/          → Regras de sistema
  history/        → Memória de longo prazo
```

---

_Mantenha este repositório atualizado para que seus agentes fiquem cada vez mais inteligentes!_ 🧠✨
