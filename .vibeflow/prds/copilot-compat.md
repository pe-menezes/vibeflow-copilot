# PRD: Compatibilidade Vibeflow + GitHub Copilot

> Generated via /vibeflow:discover on 2026-03-03

## Problem

O Vibeflow Copilot edition foi estruturado com subpastas `vibeflow/` dentro de
`.github/prompts/`, `.github/agents/` e `.github/skills/` para evitar conflitos
com arquivos do projeto. Porém, a documentação oficial do GitHub Copilot **só
confirma suporte a subdirectories em `.github/instructions/`**. Para prompts,
agents e skills, a docs diz que o discovery acontece "in the folder" — sem
mencionar scan recursivo.

Além disso, os arquivos de prompt e agent **não têm YAML frontmatter**, o que
significa que mesmo se descobertos, o Copilot não consegue montar corretamente
os slash commands (nome, description, agent mode, tools).

Resultado: um usuário que instala o Vibeflow no repo provavelmente **não vê
nenhum comando `/vibeflow-*`** no Copilot Chat, e o agent architect não aparece
no picker.

## Target Audience

Desenvolvedores que instalam o Vibeflow num repositório e usam GitHub Copilot
(VS Code, GitHub.com, ou CLI).

## Proposed Solution

Reestruturar o Copilot edition para usar:

1. **Flat structure** para prompts, agents e skills — namespacing via prefixo
   no nome do arquivo (ex: `vibeflow-analyze.prompt.md`) em vez de subpastas.
2. **Subdirectory** apenas para instructions (confirmado suportado).
3. **YAML frontmatter** em todos os prompts e no agent, seguindo a spec oficial
   do Copilot.
4. **Skill com diretório raiz correto** — `.github/skills/vibeflow-spec-driven-dev/SKILL.md`
   (nome do dir = campo `name` no frontmatter).

## Success Criteria

1. Ao instalar os arquivos num repo e abrir no VS Code com Copilot, os 8
   comandos `/vibeflow-*` aparecem no menu de slash commands.
2. O agent `vibeflow-architect` aparece no agent picker.
3. A skill `vibeflow-spec-driven-dev` é carregada quando relevante.
4. As instructions são auto-loaded quando `applyTo` faz match.
5. Todas as referências internas (agent → prompts, instructions → paths, README)
   apontam para os paths corretos.

## Scope v0

- [ ] Mover 8 prompt files de `prompts/vibeflow/` para `prompts/` (flat)
- [ ] Adicionar YAML frontmatter a cada prompt (name, description, agent mode)
- [ ] Mover agent de `agents/vibeflow/` para `agents/` (flat)
- [ ] Adicionar YAML frontmatter ao agent (name, description, tools)
- [ ] Mover skill de `skills/vibeflow/spec-driven-dev/` para `skills/vibeflow-spec-driven-dev/`
- [ ] Adicionar YAML frontmatter ao SKILL.md (name, description)
- [ ] Manter instructions em `instructions/vibeflow/` (já funciona)
- [ ] Atualizar referências internas no agent, instructions e README
- [ ] Atualizar o `copilot/AGENTS.md` com paths corretos

## Anti-scope

- **NÃO** mudar o conteúdo/lógica dos prompts — apenas estrutura e frontmatter
- **NÃO** mudar o Claude Code edition — só o Copilot edition
- **NÃO** adicionar features novas aos prompts
- **NÃO** mudar o MANUAL.md (é compartilhado; os nomes dos comandos não mudam)
- **NÃO** testar em VS Code real neste momento (validação manual posterior)

## Technical Context

### Estrutura atual (copilot/)
```
copilot/github/
  instructions/vibeflow/vibeflow.instructions.md  ← OK (subdirs suportados)
  prompts/vibeflow/vibeflow-*.prompt.md            ← PROBLEMA (subdir)
  agents/vibeflow/architect.agent.md               ← PROBLEMA (subdir)
  skills/vibeflow/spec-driven-dev/SKILL.md         ← PROBLEMA (nesting extra)
```

### Estrutura alvo
```
copilot/github/
  instructions/vibeflow/vibeflow.instructions.md   ← mantém
  prompts/vibeflow-analyze.prompt.md               ← flat, com frontmatter
  prompts/vibeflow-audit.prompt.md
  prompts/vibeflow-discover.prompt.md
  prompts/vibeflow-gen-spec.prompt.md
  prompts/vibeflow-prompt-pack.prompt.md
  prompts/vibeflow-quick.prompt.md
  prompts/vibeflow-stats.prompt.md
  prompts/vibeflow-teach.prompt.md
  agents/vibeflow-architect.agent.md               ← flat, com frontmatter
  skills/vibeflow-spec-driven-dev/SKILL.md         ← dir name = skill name
```

### YAML frontmatter esperado pelo Copilot

**Prompts:**
```yaml
---
name: 'vibeflow-analyze'
description: 'Deep-analyze codebase and build .vibeflow/ knowledge'
agent: 'vibeflow-architect'
---
```

**Agent:**
```yaml
---
name: 'vibeflow-architect'
description: 'Senior software architect and technical PM. Plans features, reviews specs, generates prompt packs. Does NOT write implementation code.'
tools:
  - 'read'
  - 'search'
  - 'web'
---
```

**Skill:**
```yaml
---
name: 'vibeflow-spec-driven-dev'
description: 'Spec-driven development methodology for planning, speccing, and auditing features.'
---
```

### Referências que precisam atualizar
- `architect.agent.md` → tabela de "Available Prompts" (paths)
- `vibeflow.instructions.md` → tabela "Where Things Live" e "Available Prompts"
- `copilot/README.md` → file tree e instruções de instalação
- `copilot/AGENTS.md` → referência ao agent path

## Open Questions

1. **Quais prompts devem rodar com `agent: 'vibeflow-architect'` vs `agent: 'agent'`?**
   Candidatos para architect: discover, analyze, gen-spec, prompt-pack, audit, stats, teach.
   Candidato para agent mode genérico: quick (porque envolve implementação).
   → Decisão: todos usam `vibeflow-architect` exceto `quick` que usa `agent` (mode genérico com acesso a edit/execute).

2. **O agent architect deve ter acesso a `edit` e `execute`?**
   Pela filosofia do Vibeflow, o architect NÃO implementa. Mas precisa de `execute`
   para rodar testes no audit. E o `quick` precisa de tudo.
   → Decisão: architect tem `read`, `search`, `web`, `execute` (para testes).
   O `quick` usa agent mode genérico que tem acesso total.
