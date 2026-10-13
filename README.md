# claude-skills

Marketplace de plugins para Claude Code.

## Instalação

```
/plugin marketplace add alealvescoelho/claude-skills
/plugin install changelog-update@claude-skills
```

## Plugins

| Plugin | O que faz |
| --- | --- |
| `changelog-update` | Atualiza o `CHANGELOG.md` existente no padrão do projeto e gera `CHANGELOG-TECHNICAL-{versão}.md` (devs) e `CHANGELOG-CASUAL-{versão}.md` (usuários), definindo a versão por SemVer. |

## Adicionar uma nova skill

1. Crie `plugins/<nome>/.claude-plugin/plugin.json` e `plugins/<nome>/skills/<nome>/SKILL.md`.
2. Registre o plugin em `.claude-plugin/marketplace.json`.
