---
name: develop-review
description: Revisa as mudanças da branch atual contra uma branch base (usando o merge-base) e gera um relatório Markdown em português com problemas classificados por severidade (CRÍTICO, AVISO, SUGESTÃO) e sugestões de testes. Use sempre que o usuário pedir code review, revisão de PR/MR, "revisar meu código", "checar bugs da branch", "o que pode quebrar nessa PR" ou quiser uma análise das alterações antes de abrir ou aprovar um pull request, mesmo sem citar o nome da skill.
---

# Develop Review

Baseada na skill `pr-code-review` de [Tainmat/skill_claude_tainmat](https://github.com/Tainmat/skill_claude_tainmat), com a branch base e os tipos de arquivo configuráveis.

## Objetivo

Revisar todas as mudanças da branch atual em relação a uma branch base, identificando bugs, problemas de performance e violações de boas práticas. O relatório é escrito em português do Brasil e salvo como `.md` na raiz do projeto. Esta skill só lê o repositório e escreve o relatório: não altera código, não faz commit nem push.

## Passo 1 — Definir a base e o escopo

Se o usuário não informou, pergunte qual é a **branch base** (sugira `develop`, ou `main`/`master` se `develop` não existir: confira com `git branch -a`). Os **tipos de arquivo** padrão são `.ts`, `.tsx`, `.js` e `.jsx`; se o projeto usar outra linguagem (veja as extensões em `git diff --name-only`), use as extensões dela e avise o usuário.

Se a branch atual for a própria base, avise: não haverá diferença para revisar.

## Passo 2 — Merge-base e diff

```bash
git merge-base HEAD <base>
```

O merge-base é o ponto exato em que a branch saiu da base. Compará-lo com `HEAD` garante que só o trabalho da branch seja revisado, sem commits que chegaram depois na base.

```bash
git diff <merge-base> HEAD --name-only -- '*.ts' '*.tsx' '*.js' '*.jsx'
git diff <merge-base> HEAD --unified=5 -- '*.ts' '*.tsx' '*.js' '*.jsx'
```

Se não houver arquivos no escopo, informe e pare.

## Passo 3 — Ranges de linhas

Em cada hunk `@@ -115,10 +115,12 @@`, a parte `+a,b` indica que, a partir da linha `a`, `b` linhas foram alteradas; o range é `a` até `a+b-1`. Se `b` for 0 ou 1, é a linha `a` sozinha. Una hunks a até 3 linhas de distância.

Formato: vários ranges `115-125 | 150-180`; linha única `151` (nunca `200-200`).

## Passo 4 — Contexto

Leia cada arquivo alterado por inteiro, ou as seções relevantes. Sem entender o que a função faz, como as linhas alteradas entram no fluxo e se as dependências são bem usadas, a revisão vira palpite.

## Passo 5 — Análise

Analise as linhas alteradas e o que elas afetam.

**Bugs (CRÍTICO ou AVISO)**
- erros de lógica; acesso a `null`/`undefined` sem guarda; condições de corrida em código assíncrono
- mutação direta de estado (React/Redux); dependências faltando em `useEffect`/`useCallback`/`useMemo`
- listeners, timers ou subscrições sem cleanup; falta de tratamento de erro em chamadas de API e parse de JSON
- `==` onde `===` seria correto; variáveis usadas antes de atribuição
- falhas de segurança (injeção, dados sensíveis expostos, entrada sem validação)

**Performance (AVISO ou SUGESTÃO)**
- objetos, arrays ou funções recriados a cada render sem memoização; re-renders evitáveis
- laços aninhados O(n²) com alternativa simples; chamadas de API duplicadas ou dentro de laços
- import de biblioteca inteira para usar um submódulo; regex recriada em laço ou render

**Boas práticas (AVISO ou SUGESTÃO)**
- tipos ausentes, excesso de `any` ou `as unknown as X`; props não tipadas
- funções/componentes com mais de ~100 linhas sem divisão clara; código duplicado
- nomes pouco descritivos; constantes mágicas; `console.log` esquecido; comentários desatualizados
- `useEffect` complexo que deveria ser custom hook

## Passo 6 — Relatório

Salve em `code-review-<nome-da-branch>.md` na raiz do projeto (troque `/` por `-` no nome da branch) e informe o caminho ao usuário. Use esta estrutura:

```markdown
# Relatório de Code Review

| Campo       | Valor                   |
|-------------|-------------------------|
| Branch      | <branch>                |
| Branch base | <base>                  |
| Merge-base  | <hash>                  |
| Data        | <dd/mm/aaaa>            |

---

## Resumo das Alterações
<Parágrafo sobre o que a PR faz: funcionalidade, bug corrigido, refatoração.>

---

## Arquivos Alterados

| # | Arquivo | Linhas alteradas |
|---|---------|------------------|
| 1 | `src/modules/Auth/Login.tsx` | 45-67 \| 120 |

**Total:** X arquivo(s) | Y hunk(s)

---

## Problemas Encontrados

### 🔴 CRÍTICO

#### #1 — <Título curto>

**Arquivo:** `caminho` — Linhas: 45-52

**Problema:** <o que se observa no código>

**Por que é um problema:** <impacto: o que pode dar errado, em que cenário>

**Sugestão de correção:** <como corrigir; se for código, só o trecho relevante>

---

### 🟡 AVISO
...

### 🔵 SUGESTÃO
...

---

## Resumo Final

| Severidade | Quantidade |
|------------|------------|
| 🔴 Crítico | X |
| 🟡 Aviso   | Y |
| 🔵 Sugestão| Z |
| **Total**  | W |

> Sem críticos: ✅ A PR pode ser aprovada com atenção aos avisos listados.
> Com críticos: ❌ A PR requer correções antes de ser aprovada.

---

## Sugestões de Testes

### O que deve ser testado (funcionalidades implementadas)
- [ ] <Cenário feliz: ação e resultado esperado>

### O que pode ter quebrado (regressões)
- [ ] <Funcionalidade preexistente afetada, mesmo que indiretamente, e como verificar>
```

Omita as seções de severidade que estiverem vazias.

## Critérios

- **Foco no diff:** revise só o que foi adicionado ou modificado. Só aponte código antigo se a mudança criar uma interação problemática com ele.
- **Contexto manda:** antes de reportar, considere se o padrão pode ser intencional naquele projeto.
- **Sem falsos positivos:** é melhor poucos problemas reais que muitos duvidosos. Na dúvida, omita. Sem problemas reais, escreva `✅ APROVADO — nenhum problema encontrado.` e não invente sugestões para preencher.
- **Testes práticos:** cenários que um QA ou o próprio dev execute em poucos minutos.
- **Severidades:** `CRÍTICO` = pode causar bug, crash, falha de segurança ou perda de dados; `AVISO` = degradação de performance, comportamento inesperado em edge cases ou violação de padrão do projeto; `SUGESTÃO` = legibilidade/manutenção sem impacto funcional imediato.
