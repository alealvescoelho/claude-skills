---
name: develop-description
description: "Gera uma descrição de Pull Request (ou commit) em português a partir das mudanças de código na branch atual, comparando com uma branch/base informada pelo usuário. Use esta skill sempre que o usuário pedir \"descrição do commit\", \"descrição do PR\", \"resumo das mudanças\", \"gerar PR description\", \"o que eu fiz nessa branch\", ou pedidos equivalentes para documentar o que foi alterado em um repositório git. A saída segue um template fixo com as seções \"O que foi feito\", \"Arquivos alterados\" (tabela com linhas modificadas), \"Motivação\" e \"Como testar\" (checklist inferido a partir do código)."
---

# Gerador de Descrição de PR/Commit

Esta skill analisa o diff de uma branch git em relação a uma base de comparação e gera uma descrição de PR pronta para colar no GitHub/GitLab/etc., seguindo um template fixo em português.

## Fluxo de trabalho

### 1. Confirmar o repositório e a branch atual

Rode `git status` e `git branch --show-current` no diretório do projeto (pergunte ao usuário o caminho se não estiver óbvio pelo contexto da conversa).

### 2. Perguntar a base de comparação

**Sempre pergunte ao usuário contra o que comparar** antes de gerar o diff — não assuma `main`, `master` ou `origin/<branch>` por padrão. Ofereça as opções mais prováveis com base no que `git branch -a` mostrar (ex: branch principal do repo, `origin/<branch-atual>`, ou outra branch específica), mas deixe o usuário escolher.

### 3. Obter o diff completo

Uma vez definida a base, rode:

```bash
git diff <base>...HEAD --stat        # visão geral dos arquivos e contagem de linhas
git diff <base>...HEAD                # diff completo, linha a linha
```

Se o diff for muito grande para caber no contexto de forma legível, processe arquivo por arquivo com `git diff <base>...HEAD -- <arquivo>`.

Também rode `git log <base>..HEAD --oneline` para ver as mensagens de commit da branch — elas costumam conter pistas úteis sobre a intenção das mudanças.

### 4. Analisar as mudanças

A partir do diff, identifique:
- **Funcionalidades/comportamentos novos** adicionados
- **Comportamentos alterados** (ex: mudança de lógica existente)
- **Bugs corrigidos**
- **Arquivos de configuração/infra tocados** (`.gitignore`, CI, etc.) — mencione brevemente se relevante
- Agrupe mudanças relacionadas ao mesmo recurso em um único item, em vez de listar arquivo por arquivo

### 5. Montar a tabela de arquivos alterados

Para cada arquivo no diff, liste os intervalos de linhas modificadas (hunks), no formato `inicio-fim`, separados por ` \| ` quando houver mais de um intervalo no mesmo arquivo. Use `git diff <base>...HEAD --unified=0 -- <arquivo>` para extrair os hunks com precisão (os cabeçalhos `@@ -a,b +c,d @@` indicam os intervalos no arquivo novo).

### 6. Inferir a seção "Como testar"

Gere uma checklist de passos manuais de teste **inferidos a partir do código alterado** (não deixe placeholder e não pergunte ao usuário, a menos que o diff seja ambíguo demais para inferir algo útil). Baseie-se em:
- Componentes/telas de UI alterados → passos de navegação e interação
- Novas condições/validações → cenários que disparam cada condição (caminho feliz e casos de erro)
- Mudanças em serviços/APIs → como verificar a chamada e a resposta esperada

### 7. Gerar a descrição final

Use exatamente esta estrutura de saída (markdown, em português):

```markdown
## O que foi feito
- [item 1]
- [item 2]
---
## Arquivos alterados
| Arquivo | Linhas modificadas |
|---------|-------------------|
| `caminho/arquivo1` | 10-20 |
| `caminho/arquivo2` | 5-8 \| 30-45 |
---
## Motivação
[parágrafo curto explicando o problema que existia antes e por que essa mudança resolve/melhora]
---
## Como testar
- [ ] [passo 1]
- [ ] [passo 2]
```

Regras de formatação:
- "O que foi feito" é uma lista de bullets, cada um descrevendo uma mudança funcional (não uma mudança de arquivo)
- A tabela de arquivos deve incluir **todos** os arquivos alterados no diff, mesmo os pequenos
- "Motivação" é um único parágrafo corrido, sem bullets
- "Como testar" é sempre uma checklist (`- [ ]`), com passos práticos e específicos ao que foi alterado

Siga modelo de estrutura e formatação:

[Inicio do modelo de formatação]
## O que foi feito

- Item do que foi feito
- Item do que foi feito

---

## Arquivos alterados

| Arquivo | Linhas modificadas |
|---------|-------------------|
| `.gitignore` | 47-53 |
| `src/@types/chatwoot.ts` | 79-92 |
| `src/services/ChatwootService.ts` | 714-720 \| 734-740 \| 756-771 |
| `src/views/crm/Chat/components/ChatForm/useChatFormRules.ts` | 1-5 \| 48-83 \| 439-497 \| 574-601 |

---

## Motivação

Descrição da Motivação.

---

## Como testar

- [ ] Acesse determinada tela.
- [ ] Simule uma comportamento.
[Final do modelo de formatação]

### 8. Entregar

Mostre o resultado direto na conversa (é conteúdo para o usuário copiar e colar em uma PR, não um arquivo separado — não use a ferramenta de criação de arquivos a menos que o usuário peça explicitamente para salvar em disco).