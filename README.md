# claude-skills

Marketplace de plugins para Claude Code.

## Instalação

```
/plugin marketplace add alealvescoelho/claude-skills
/plugin install changelog-update@claude-skills
/plugin install develop-review@claude-skills
/plugin install develop-description@claude-skills
```

## Plugins

| Plugin | O que faz |
| --- | --- |
| `changelog-update` | Atualiza o `CHANGELOG.md` existente no padrão do projeto e gera `CHANGELOG-TECHNICAL-{versão}.md` (devs) e `CHANGELOG-CASUAL-{versão}.md` (usuários), definindo a versão por SemVer. |
| `develop-review` | Revisa a branch atual contra uma branch base e gera relatório Markdown em português com problemas por severidade (crítico, aviso, sugestão) e sugestões de testes. Baseada na `pr-code-review` de [Tainmat/skill_claude_tainmat](https://github.com/Tainmat/skill_claude_tainmat). |
| `develop-description` | Gera a descrição de PR/commit em português a partir do diff da branch, com as seções "O que foi feito", "Arquivos alterados", "Motivação" e "Como testar". |

## Adicionar uma nova skill

1. Crie `plugins/<nome>/.claude-plugin/plugin.json` e `plugins/<nome>/skills/<nome>/SKILL.md`.
2. Registre o plugin em `.claude-plugin/marketplace.json`.
