---
name: changelog-update
description: Atualiza o CHANGELOG.md existente do projeto seguindo o padrão já usado nele e gera dois novos arquivos em português - CHANGELOG-TECHNICAL-{versão}.md (técnico, para devs) e CHANGELOG-CASUAL-{versão}.md (linguagem simples, para usuários). Define a próxima versão por SemVer/Conventional Commits. Use sempre que o usuário pedir para gerar, criar, atualizar ou fechar o changelog, notas de versão ou release notes, falar em "CHANGELOG", "changelog técnico/casual", "o que mudou nessa versão" ou "subir versão", mesmo que não cite os nomes dos arquivos.
---

# Changelog Update

Gera as notas de uma nova versão em três saídas, sempre em **português do Brasil**:

1. `CHANGELOG.md` (já existente) — recebe a nova versão, **no mesmo padrão que o arquivo já usa**
2. `CHANGELOG-TECHNICAL-{versão}.md` — detalhado, para desenvolvedores
3. `CHANGELOG-CASUAL-{versão}.md` — linguagem simples, para quem usa o sistema

Esta skill só lê o histórico git e escreve arquivos de documentação. Não altere código, não faça commit, tag, push ou merge — o usuário decide o que fazer depois.

## Passo 1 — Definir de onde vêm as mudanças

Pergunte ao usuário (não assuma):

> As mudanças devem ser buscadas a partir da **última tag** ou de uma **branch base informada** (ex.: master, develop)?

- **Última tag:** use `git describe --tags --abbrev=0` e o intervalo `<tag>..HEAD`. Se não houver tag, avise e ofereça a opção de branch base.
- **Branch base:** pergunte qual (ex.: `master`, `develop`) e use `git merge-base HEAD <base>` como ponto de partida.

Liste os commits com `git log <início>..HEAD --no-merges --pretty=format:"%h|%s|%b|%an|%ad" --date=short`. Ignore commits de merge e use só o histórico real. Em branches de release que agregam várias features, os títulos de PR/MR (`git log --merges`) ajudam a dar contexto, mas não viram itens por si só.

## Passo 2 — Localizar e entender o CHANGELOG.md

Procure o `CHANGELOG.md` na raiz do repositório (se não achar, busque com Glob por `**/CHANGELOG.md` ignorando `node_modules`). Leia-o inteiro, ou ao menos as duas ou três versões mais recentes, e identifique o padrão:

- formato do título de versão (`## [1.2.0] - 2026-01-10`, `## v1.2.0`, etc.) e se há link de comparação no rodapé
- idioma, uso de emojis, nomes e ordem das seções (Added/Changed/Fixed, ou Novidades/Correções...)
- estilo das entradas (com escopo em negrito? com hash? com link de PR/issue?)
- onde fica a seção `Unreleased`, se existir

Reproduza esse padrão exatamente, para o diff do arquivo ficar limpo e consistente. A única exceção é se o arquivo estiver vazio ou sem padrão reconhecível: nesse caso use o formato Keep a Changelog em português.

**Se não existir CHANGELOG.md:** avise o usuário e pergunte se deve criar um (Keep a Changelog) ou gerar apenas os dois arquivos novos.

## Passo 3 — Versão atual e próxima versão (SemVer)

Versão atual, por ordem de prioridade: versão mais recente no `CHANGELOG.md` → última tag git → `package.json`. Se nada for encontrado, pergunte.

Classifique os commits pelo prefixo Conventional Commits e aplique a regra mais alta:

| Sinal | Bump |
| --- | --- |
| `BREAKING CHANGE` no corpo, ou `!` após o tipo (`feat!:`) | MAJOR |
| `feat:` | MINOR |
| `fix:`, `perf:`, `refactor:`, `docs:`, `style:`, `test:`, `chore:`, `ci:`, `build:` e commits sem prefixo | PATCH |

Commits sem prefixo convencional vão para a categoria "Outros".

**Antes de escrever qualquer arquivo**, apresente a decisão e espere confirmação, porque a versão vira nome de arquivo e título no histórico:

> **Versão atual:** `1.25.0` → **próxima:** `1.26.0` (MINOR)
> **Motivo:** N commits `feat:`, sem quebra de compatibilidade.
> Confirma? (s/n, ou informe outra versão)

## Passo 4 — Atualizar o CHANGELOG.md

Insira a nova versão no topo (logo abaixo do cabeçalho do arquivo / acima da versão mais recente, ou convertendo `Unreleased` se o padrão for esse), usando o padrão identificado no Passo 2. Preserve todo o conteúdo antigo, apenas acrescente. Atualize links de comparação do rodapé se o arquivo os tiver. Use a data de hoje.

Se a versão já existir no arquivo (reexecução), atualize a entrada em vez de duplicá-la.

## Passo 5 — Gerar `CHANGELOG-TECHNICAL-{versão}.md`

Salve na raiz do projeto. Se o arquivo já existir, sobrescreva com o conteúdo atualizado.

```markdown
# CHANGELOG TÉCNICO — v{versão}

> **Tipo de versão:** MAJOR / MINOR / PATCH
> **Motivo:** {decisão SemVer em 1–2 frases}
> **Data:** {dd/mm/aaaa}
> **Base de comparação:** {tag ou branch base}
> **Branch de origem:** {branch atual}

---

## 💥 Quebras de Compatibilidade
- **{escopo}:** {descrição} (`{hash 7 chars}`)

## ✨ Novas Funcionalidades
## 🐛 Correções de Bug
## ⚡ Melhorias de Performance
## ♻️ Refatorações
## 🔧 Manutenção e Infraestrutura   (chore, ci, build, docs, style, test)
## 🔀 Outros

---

## Resumo
| Categoria | Quantidade |
| --- | --- |
| ... | X |
| **Total de commits** | X |
```

Omita seções vazias. Com escopo: `**auth:** adiciona validação de token expirado (`abc1234`)`; sem escopo, só a descrição. Descreva o que mudou com precisão técnica (módulos, comportamento), indo além de copiar a mensagem do commit quando o diff deixar claro algo mais útil.

## Passo 6 — Gerar `CHANGELOG-CASUAL-{versão}.md`

Salve na raiz. Aqui o leitor é quem usa o sistema, não quem o desenvolve. Por isso entram **apenas** mudanças que a pessoa percebe:

| Tipo | Incluir? |
| --- | --- |
| `feat`, `fix`, `BREAKING CHANGE` | Sim |
| `perf` | Só se perceptível (algo ficou mais rápido) |
| `style` | Só se mudar a aparência de telas |
| `refactor`, `chore`, `docs`, `test`, `ci`, `build` | Não |

Escreva em linguagem simples, na perspectiva de quem usa: o que ela pode fazer agora ou parou de ver. Sem jargão (`hook`, `endpoint`, `refactor`), sem hash. Traduza escopos técnicos para o nome da tela ou funcionalidade quando der para saber, e agrupe mudanças da mesma tela.

| Commit | Texto casual |
| --- | --- |
| `feat(equipamentos): adiciona filtro por subgrupo` | Agora é possível filtrar equipamentos por subgrupo na tela de Equipamentos |
| `fix(login): corrige erro com matrícula inválida` | Corrigido o erro que aparecia ao tentar entrar com uma matrícula inválida |

```markdown
# O que há de novo — v{versão}

> **Data:** {dd/mm/aaaa}

## ✨ Novidades
- ...

## 🐛 Correções
- ...

## ⚡ Melhorias
- ...

## ⚠️ Mudanças Importantes
- ...
```

Omita seções vazias. Se nada for visível ao usuário, escreva `> Nenhuma alteração visível para o usuário nesta versão.`

## Passo 7 — Informar o resultado

Resuma em poucas linhas: versão (`anterior` → `nova`, tipo do bump), o que foi alterado no `CHANGELOG.md` e os dois arquivos criados, com o total de commits e de itens voltados ao usuário. Lembre que nada foi commitado nem taggeado.

## Casos de borda

- Branch atual é `master`/`main` e a base é ela mesma: avise que não haverá diferença e confirme a branch correta.
- Nenhum commit no intervalo: informe e não gere arquivos.
- Commits sem padrão convencional: classifique como PATCH/"Outros" e, se forem muitos, sugira ao usuário revisar a classificação.
