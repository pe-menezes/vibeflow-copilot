# Vibeflow

**Spec-driven development** — defina o que construir antes de codar.

Vibeflow separa quem pensa de quem implementa. O Architect (você + IA) define specs, toma decisões e corta escopo. O Coding Agent recebe um prompt pack auto-contido e implementa seguindo os padrões reais do projeto.

## O pipeline

```
discover → analyze → gen-spec → prompt-pack → implement → audit
```

## Edições disponíveis

| Edição | Pasta | Status |
|--------|-------|--------|
| **GitHub Copilot** | [`copilot/`](copilot/) | Disponível |
| **Claude Code** | [`claude-code/`](claude-code/) | Disponível |

Cada edição adapta os mesmos prompts e metodologia ao formato do agente.
O conteúdo da metodologia é o mesmo — o que muda é a estrutura de arquivos.

### Claude Code

O Claude Code usa um sistema de plugins baseado em git. O repo de distribuição
(marketplace) é mantido separado:

**Repo de distribuição:** [pe-menezes/vibeflow](https://github.com/pe-menezes/vibeflow)

Instale via Claude Code:
```
/install-plugin pe-menezes/vibeflow
```

O source of truth dos arquivos Claude Code está em `claude-code/` neste repo.

### GitHub Copilot

Copie os arquivos para o seu repo. Veja [`copilot/README.md`](copilot/README.md) para instruções de instalação.

## Manual

Veja o [MANUAL.md](MANUAL.md) para a documentação completa de todos os comandos, fluxos e a metodologia.
