# Vibeflow — Copilot Edition

Arquivos de configuração do Vibeflow para **GitHub Copilot**.

## O que está incluído

```
copilot/
├── AGENTS.md                                     → raiz do repo (append se já existir)
├── copilot-instructions.md                       → snippet de append (opcional)
└── github/                                       → mapeia para .github/ no repo destino
    ├── instructions/vibeflow/
    │   └── vibeflow.instructions.md              → instruções completas
    ├── agents/vibeflow/
    │   └── architect.agent.md                    → persona do Architect
    ├── prompts/vibeflow/
    │   ├── vibeflow-analyze.prompt.md
    │   ├── vibeflow-audit.prompt.md
    │   ├── vibeflow-discover.prompt.md
    │   ├── vibeflow-gen-spec.prompt.md
    │   ├── vibeflow-prompt-pack.prompt.md
    │   ├── vibeflow-quick.prompt.md
    │   ├── vibeflow-stats.prompt.md
    │   └── vibeflow-teach.prompt.md
    └── skills/vibeflow/
        └── spec-driven-dev/
            └── SKILL.md
```

Tudo fica namespaced dentro de `vibeflow/` — não bagunça pastas existentes do projeto.

## Instalação

### Repo novo (sem nada configurado)

```bash
# Na raiz do repo destino
cp copilot/AGENTS.md .
cp -r copilot/github/ .github/
```

Remova a nota de instrução do topo do `AGENTS.md` (o bloco entre `>` e `---`).

### Repo que JÁ tem AGENTS.md e/ou copilot-instructions.md

#### Passo 1 — Copie as pastas (seguro, não sobrescreve nada)

```bash
cp -r copilot/github/instructions/ .github/instructions/
cp -r copilot/github/agents/       .github/agents/
cp -r copilot/github/prompts/      .github/prompts/
cp -r copilot/github/skills/       .github/skills/
```

Como tudo está dentro de subpastas `vibeflow/`, não sobrescreve nenhum arquivo existente.

#### Passo 2 — copilot-instructions.md (opcional)

O Copilot carrega automaticamente os arquivos de `.github/instructions/`.
O `vibeflow.instructions.md` que você copiou já será lido.

Se quiser ser explícito, veja o snippet em `copilot-instructions.md` e adicione ao seu existente.

#### Passo 3 — AGENTS.md

Se já existe, faça append do conteúdo abaixo do `---` do `AGENTS.md` ao final do seu.
Se não existe, copie direto (removendo a nota do topo).

### O que conflita e o que não conflita

| Arquivo / Pasta | Pode conflitar? | O que fazer |
|-----------------|----------------|-------------|
| `.github/instructions/vibeflow/` | Não | Copia direto |
| `.github/agents/vibeflow/` | Não | Copia direto |
| `.github/prompts/vibeflow/` | Não | Copia direto |
| `.github/skills/vibeflow/` | Não | Copia direto |
| `.github/copilot-instructions.md` | Talvez | Append opcional |
| `AGENTS.md` | **Sim** | Append ao existente |

## Após a instalação

1. Use o prompt `vibeflow-analyze` para gerar a pasta `.vibeflow/`.
2. Adicione `.vibeflow/` ao git.
3. Para features novas, comece pelo `vibeflow-discover`.

Veja o [MANUAL.md](../MANUAL.md) para a documentação completa.
