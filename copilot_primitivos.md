# Primitivos Copilot (agent, skill, prompt e instructions)

## Sumário

- [O que é cada primitivo](#o-que-é-cada-primitivo)
  - [Agent (.agent.md)](#agent-agentmd)
    - [Crie seus próprios agentes personalizados (.agent.md)](#crie-seus-próprios-agentes-personalizados-agentmd)
  - [Skill (SKILL.md + pasta)](#skill-skillmd--pasta)
    - [Estrutura básica — skills](#estrutura-básica--skills)
    - [Como o Copilot descobre e aplica automaticamente?](#como-o-copilot-descobre-e-aplica-automaticamente)
    - [Onde podem ser armazenadas?](#onde-podem-ser-armazenadas)
    - [Diferença entre ~/.copilot/ e ~/.agents/](#diferença-entre-copilot-e-agents)
    - [Como criar a estrutura no terminal](#como-criar-a-estrutura-no-terminal)
  - [Prompt (.prompt.md)](#prompt-promptmd)
  - [Instructions (.instructions.md)](#instructions-instructionsmd)
    - [Onde podem ser armazenadas?](#onde-podem-ser-armazenadas-1)
    - [Escopos por applyTo](#escopos-por-applyto)
- [Diferença: Skill vs Agente vs Prompt vs Instructions](#diferença-skill-vs-agente-vs-prompt-vs-instructions)
  - [Assets bundled (recursos empacotados)](#assets-bundled-recursos-empacotados)
  - [Isolamento de contexto](#isolamento-de-contexto)
  - [Restrição de ferramentas](#restrição-de-ferramentas)
  - [Uso ideal — resumo mental](#uso-ideal--resumo-mental)
  - [Exemplo concreto dos três juntos](#exemplo-concreto-dos-três-juntos)
- [Como criar primitivos](#como-criar-primitivos)
  - [Para criar do zero](#para-criar-do-zero)
  - [Para atualizar um existente](#para-atualizar-um-existente)
  - [Para depurar quando não funciona](#para-depurar-quando-não-funciona)
  - [Dica principal](#dica-principal)
- [Caso de uso: (Skill) de Code Review](#caso-de-uso-skill-de-code-review)
- [Caso de uso: Projeto Mei Contabilidade](#caso-de-uso-projeto-mei-contabilidade)

---

### O que é cada primitivo

#### Agent (`.agent.md`)
Persona especializada com **ferramentas restritas** e **isolamento de contexto**. Quando invocado como subagente, recebe apenas o que o agente pai delega e retorna um único resultado. Ideal para enforçar que o Copilot nunca faça algo fora do escopo definido.


##### Crie seus próprios agentes personalizados (.agent.md)
Você pode definir agentes Copilot especializados como arquivos .agent.md dentro do repositório. 

Esses agentes têm:

- Acesso completo ao contexto do workspace
- Compreensão de código
- Ferramentas configuráveis
- Modelo de linguagem preferido
- Conexões MCP a fontes externas de conhecimento
- Eles aparecem no seletor de agentes e ficam disponíveis para toda a equipe.

---
#### Skill (`SKILL.md` + pasta)
**Pasta de instruções + scripts + recursos** carregados sob demanda. O carregamento é progressivo:
1. **Descoberta (~100 tokens)**: lê `name` e `description`
2. **Instruções (<5000 tokens)**: carrega o corpo do `SKILL.md` quando relevante
3. **Recursos**: arquivos adicionais só carregam quando referenciados

Ideal para workflows repetíveis que precisam de scripts ou documentação de apoio empacotados junto.


##### Estrutura básica — skills
 ``` 
    .github/skills/<nome-da-skill>/
    ├── SKILL.md           # Arquivo principal (obrigatório)
    ├── scripts/           # Scripts executáveis
    ├── references/        # Documentação de apoio
    └── assets/            # Templates, boilerplates
 ```

---
##### Como o Copilot descobre e aplica automaticamente?
O carregamento é progressivo em 3 etapas:

1. Descoberta (~100 tokens): O agente lê apenas name e description de cada skill
2. Instruções (<5000 tokens): Se a conversa é relevante, carrega o corpo do SKILL.md
3. Recursos: Arquivos adicionais só são carregados quando referenciados
Por isso o campo description é crítico — ele é o gatilho de descoberta.

Por isso o campo description é crítico — ele é o gatilho de descoberta.

---
##### Onde podem ser armazenadas?

| Caminho | Escopo |
|---|---|
| `.github/skills/<nome>/` | Projeto (compartilhado com o time) |
| `.agents/skills/<nome>/` | Projeto |
| `~/.copilot/skills/<nome>/` | Perfil pessoal (legado) |
| `~/.agents/skills/<nome>/` | Perfil pessoal (recomendado) |

##### Diferença entre `~/.copilot/` e `~/.agents/`

| | `~/.copilot/skills/` | `~/.agents/skills/` |
|---|---|---|
| **Origem** | Path legado do Copilot extension | Path do sistema moderno de agentes |
| **Alinhamento** | Configurações gerais do Copilot | Primitivos agent/skill/prompt |
| **Uso recomendado** | Skills antigas / compatibilidade | Skills novas (abordagem atual) |

> Para skills novas, prefira `~/.agents/skills/` — é o path alinhado com o sistema de primitivos.

##### Como criar a estrutura no terminal

Perfil pessoal (`~/.agents/skills/`):

```bash
mkdir -p ~/.agents/skills/<nome-da-skill>/{scripts,references,assets} && touch ~/.agents/skills/<nome-da-skill>/SKILL.md
```

Dentro do projeto (`.github/skills/`):

```bash
mkdir -p .github/skills/<nome-da-skill>/{scripts,references,assets} && touch .github/skills/<nome-da-skill>/SKILL.md
```

---
#### Prompt (`.prompt.md`)
Tarefa única com **inputs parametrizados**. Aparece como comando `/nome` no chat. Não empacota assets, não restringe ferramentas. Ideal para operações pontuais com variáveis.

---
#### Instructions (`.instructions.md`)
**Regras de comportamento e convenções** que o Copilot aplica continuamente, sem precisar ser invocado. Não é um comando, não cria agente, não empacota recursos. Age como uma política base que entra automaticamente no contexto de toda interação relevante.

Características principais:
- **Passivo**: entra no contexto automaticamente, sem o usuário chamar explicitamente
- **Sem isolamento**: não cria subagente, roda no mesmo contexto do chat
- **Sem restrição de ferramentas**: não controla o que o Copilot pode ou não executar
- **Escopo por arquivo/pasta**: via `applyTo`, você pode definir regras diferentes para frontend, backend, testes etc.

##### Onde podem ser armazenadas?

| Caminho | Escopo |
|---|---|
| `.github/copilot-instructions.md` | Projeto (repositório inteiro, legado) |
| `.github/instructions/<nome>.instructions.md` | Projeto (com `applyTo` por glob) |
| `~/.config/github-copilot/instructions/<nome>.instructions.md` | Perfil pessoal |

##### Escopos por `applyTo`

O campo `applyTo` no frontmatter define para quais arquivos as instruções se aplicam:

```markdown
---
applyTo: "src/frontend/**"
---
# Regras do Frontend
- Use sempre componentes funcionais com TypeScript
- Nunca use `any`
```

Sem `applyTo`, as instruções se aplicam a toda conversa no repositório.

---
### Diferença: Skill vs Agente vs Prompt vs Instructions

#### Assets bundled (recursos empacotados)

| Primitivo | Suporte | Detalhe |
|---|---|---|
| Skill | ✅ | A pasta da skill contém scripts, templates e docs de referência junto com as instruções. O agente carrega progressivamente conforme precisa. |
| Agente | ❌ | Um `.agent.md` é apenas um único arquivo de texto com frontmatter + instruções. Não tem pasta, não empacota scripts. |
| Prompt | ❌ | Igual ao agente — é um único `.prompt.md`. Pode referenciar arquivos do repo, mas não empacota nada. |
| Instructions | ❌ | Um único `.instructions.md`. Não empacota scripts nem assets. Só texto de diretrizes. |

> **Exemplo prático:**:

> Precisa rodar um script de cobertura de testes como parte do processo? → **Skill**.

>Só precisa de instruções textuais? → **Agente** ou **Prompt**.

---

#### Isolamento de contexto

| Primitivo | Isolado | Detalhe |
|---|---|---|
| Skill | ❌ | Roda no mesmo contexto da conversa atual. O histórico e ferramentas do chat pai estão visíveis. |
| Agente | ✅ | Quando invocado como subagente, recebe apenas o que o agente pai delega. Retorna um único resultado, sem "vazar" contexto interno para o pai. |
| Prompt | ❌ | Também roda no contexto atual, sem isolamento. |
| Instructions | ❌ | Entra como contexto base em toda interação — sem isolamento, sem subagente. É o primitivo mais "transparente". |

> **Por que isso importa?** Quando você tem um pipeline em etapas (ex: pesquisar → analisar → gerar relatório), cada agente especializado pode focar somente no seu pedaço, sem se distrair com o contexto das outras etapas.

---

#### Restrição de ferramentas

| Primitivo | Restringe | Detalhe |
|---|---|---|
| Skill | ❌ | Herda todas as ferramentas disponíveis no agente que a carregou. Não tem controle próprio. |
| Agente | ✅ | O frontmatter define exatamente quais ferramentas estão disponíveis via `tools: [read, search]`. Você pode criar um agente que só lê, nunca edita, ou só usa um servidor MCP específico. |
| Prompt | ❌ | Sem controle de ferramentas próprio. |
| Instructions | ❌ | Não controla ferramentas. Só orienta o comportamento textual do Copilot. |

> **Exemplo prático:** Um agente de auditoria de segurança com `tools: [read, search]` — fisicamente incapaz de modificar arquivos, por design.

---

#### Uso ideal — resumo mental

| Situação | Primitivo |
|---|---|
| Tenho scripts/templates para empacotar junto | **SKILL** |
| Preciso restringir ferramentas ou isolar contexto | **AGENTE** (`.agent.md`) |
| É uma tarefa pontual com inputs variáveis | **PROMPT** (`.prompt.md`) |
| Quero definir padrões e convenções contínuas | **INSTRUCTIONS** (`.instructions.md`) |

---

#### Exemplo concreto dos três juntos

Imagine um workflow de deploy:

- **Instructions** (`.instructions.md`): `applyTo: "**"` com regras como "nunca use `console.log` em produção" e "sempre escreva testes para cada função" — ativas em toda conversa, sem precisar chamar
- **Prompt** (`.prompt.md`): `"Gere o changelog para a versão {{version}}"` — tarefa única, input parametrizado
- **Skill**: `"Como fazer deploy"` — contém scripts de build, checklist em `references/` e templates de PR
- **Agente** (`.agent.md`): Agente "Deploy Reviewer" com `tools: [read, search]` (sem acesso a `execute`) que valida o código antes de qualquer deploy, garantindo que nunca vai rodar comandos acidentalmente

---
### Como criar primitivos

Basta descrever o que você quer fazer e o Copilot identifica automaticamente qual primitivo usar. Alguns exemplos diretos:

#### Para criar do zero

- `"Cria um agent para o frontend que só lê e pesquisa arquivos TypeScript"`
- `"Cria uma skill para rodar os testes Playwright e interpretar os erros"`
- `"Cria um prompt para gerar um endpoint FastAPI com os padrões do projeto"`
- `"Cria um instructions com as convenções de código do time para arquivos TypeScript"`

#### Para atualizar um existente

- `"Atualiza o agent backend-specialist para também ter acesso à ferramenta execute"`
- `"Adiciona à skill mei-report um script que exporta PDF além do CSV"`
- `"Atualiza o prompt add-transaction para incluir um campo de categoria"`

#### Para depurar quando não funciona

- `"O Copilot não está descobrindo minha skill automaticamente, o que está errado?"`
- `"Meu agent está aparecendo no seletor mas ignorando as restrições de ferramentas"`

---

#### Dica principal

A frase mais útil é descrever o **problema** em vez do primitivo:

| Em vez de... | Diga... |
|---|---|
| `"Cria um agent"` | `"Quero que o Copilot só edite arquivos Python quando trabalhar no backend"` |
| `"Cria uma skill"` | `"Quero que o Copilot saiba gerar relatórios seguindo as regras da Receita"` |
| `"Cria um prompt"` | `"Quero um comando rápido para registrar uma transação com tipo, valor e data"` |
| `"Cria um instructions"` | `"Quero que o Copilot sempre siga os padrões de nomenclatura do time nos arquivos TypeScript"` |

> O Copilot escolhe o primitivo certo baseado no problema que você descreve.

---
### Caso de uso: (Skill) de Code Review
Imagine um time que tem convenções específicas de code review. Com uma skill:

**Local onde está armazenado a skill**

```
  .github/skills/code-review/SKILL.md:
```
**Exemplo:**

```
---
name: code-review
description: 'Revise código seguindo as convenções do time. Use when: reviewing PRs, checking code quality, validating naming conventions, checking test coverage.'
argument-hint: 'Arquivo ou PR para revisar'
---

# Code Review

## Quando usar
- Review de PRs
- Validação de padrões do time

## Procedimento
1. Leia o arquivo [checklist de convenções](./references/conventions.md)
2. Verifique nomenclatura, testes e tratamento de erros
3. Aponte problemas com severidade (crítico / sugestão)
4. Execute [script de métricas](./scripts/coverage-check.sh)

## Formato de saída
- Liste cada problema com arquivo + linha
- Classifique: CRÍTICO / MELHORIA / SUGESTÃO

```

**Como usar: Basta digitar no chat:**

/code-review src/services/payment.ts

ou simplesmente descrever o que quer — o Copilot descobre a skill automaticamente pela descrição.

---
### Caso de uso: Projeto Mei Contabilidade
Projeto criado também como **laboratório prático** para estudar os três primitivos de customização do GitHub Copilot: **Agent**, **Skill** e **Prompt**. Cada um resolve um problema diferente.

Informações implementação primitivos e repositório: Atualizar link projeto e doc 
