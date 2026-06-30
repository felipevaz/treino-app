# 需求 AI 对话记录

- 线程: thr_np7cl58h
- 更新时间: 2026-06-30T18:16:38.927Z

---

## 用户

Read the current requirement draft, identify the key questions that still need clarification, and suggest concrete additions.

> [工具] ls

> [工具] find

> [工具] read

> [工具] read

> [工具] read

> [工具] read

> [工具] read

## 需求 AI

Aqui está minha análise do **requirement.md** (`Address documentation issues`). O documento é excelente em diagnosticar os problemas de deriva documental, mas **deixa várias lacunas** que impedem a execução. Abaixo, as questões-chave e adições concretas.

---

## 🔍 Questões-chave que ainda precisam de esclarecimento

### 1. Escopo: onde esses documentos vivem?

O req cita `WORKBOOK_FIFO.md`, `DATAFLOW.md`, `SCHEMAS.md`, `B3_INCOME_EVENTS.md`, `BTG_WORKFLOW.md`, etc. — mas **nenhum desses arquivos existe neste workspace** (treino-app só tem `AGENTS.md`, `PLAN.md`, `README.md`).

**Pergunta:** Esses docs estão em outro repositório? Outro diretório fora do workspace atual? Sem saber onde estão, não é possível aplicar nenhuma das correções propostas.

### 2. Qual documento é a autoridade fonte para cada coluna conflitante?

O req identifica que `WORKBOOK_FIFO.md` difere de `SCHEMAS.md` e `DATAFLOW.md` nos mapeamentos de coluna EinAus e Payouts, e sugere que `WORKBOOK_FIFO` ("from DEV") provavelmente é o correto. Mas **não afirma categoricamente qual é a verdade** — fica em "presumably authoritative". Sem essa decisão explícita, ninguém (humano ou agente) saberá qual lado corrigir.

### 3. Quem executa e em que ordem?

As 6 recomendações estão rankeadas por alavancagem, mas não têm:
- **Responsável** (humano vs. agente CI vs. edição manual?)
- **Ordem de implementação** (o "canonical owner" é pré-requisito para o docs-lint ou vice-versa?)
- **Prazo** (tudo precisa estar feito antes de qualquer outro trabalho no sistema?)

### 4. O que acontece com a treino-app?

O req está dentro do diretório `.kunsdd/requirements/` do projeto **treino-app**, mas o conteúdo é sobre um sistema financeiro de FIFO/impostos alemães/B3 brasileiro. **Isso é intencional?** O req deveria estar num repositório diferente? Ou é um requisito cruzado (ex.: o treino-app também precisa seguir as mesmas convenções documentais)?

### 5. Métricas de sucesso: como saber que o problema acabou?

O req lista sintomas atuais, mas não define **critérios de aceitação** mensuráveis:
- "Número de mirrors conflitantes" → precisa chegar a 0.
- "docs-lint passa" → quais regras exatamente?
- "Nenhum sumário manual divergente" → como provar?

---

## 📝 Adições concretas sugeridas

### Adição 1 — Tabela de "Autoridade Canônica" (substitui a recomendação vaga #1)

Adicionar ao início da seção "How to tighten it":

```markdown
### Autoridades canônicas (fato único, sem mirrors)

| Fato | Documento dono | Seção/Link |
|------|---------------|------------|
| Mapa de colunas EinAus | `WORKBOOK_FIFO.md` | §EinAus |
| Mapa de colunas Payouts | `WORKBOOK_FIFO.md` | §Payouts |
| Mapa de colunas BTG_Brokerage_Import | `WORKBOOK_FIFO.md` | §Staging |
| Status de riscos (RX-NNN) | `RISKS.md` | (seção de detalhe de cada risco; não a tabela sumário) |
| Status de tarefas (HA-NNN) | `HUMAN_ACTIONS.md` | (seção de detalhe de cada ação) |
| Contagem de sheets do FIFO | gerado automaticamente | N/A (derivado, não escrito) |
| Contagem de eventos B3 | gerado automaticamente | N/A (derivado, não escrito) |

**Regra:** Ao editar um fato, edite apenas o dono. Onde um mirror existir, substitua por `→ Consulte [WORKBOOK_FIFO.md#einaus](link)`.
```

### Adição 2 — Checklist executável de propagação de remoção

Substituir a recomendação #5 (genérica) por:

```markdown
### Checklist de remoção/depreciação (passos obrigatórios)

Ao remover uma sheet, comando, campo de modelo, ou fluxo:

- [ ] `grep -r "termo" docs/` — encontre toda ocorrência não-histórica
- [ ] Cada ocorrência: substituir por link para o doc dono OU mover para seção `## Histórico` / `(REMOVED)`
- [ ] Atualizar `INVENTORY.md` (status → "Removed" ou "Deprecated")
- [ ] Atualizar `ROADMAP.md` (se aplicável, adicionar entrada de depreciação)
- [ ] Atualizar `INDEX.md` (se roteava parser/trabalho para o termo removido)
- [ ] Adicionar termo à **denylist** do docs-lint
```

### Adição 3 — Especificação do docs-lint (implementável)

Substituir a recomendação #3 por:

```markdown
### docs-lint — regras concretas (para implementation.md no pipeline CI)

```yaml
# .github/workflows/docs-lint.yml (ou integrado ao ci.yml existente)
# Usar: markdownlint + scripts Python dedicados em scripts/docs_lint/

Regras:
1. Header obrigatório: todo `docs/*.md` deve começar com bloco:
   `Type: | Prerequisites: | Size: | Updated:`
   → Falha: arquivo sem header.
   → Pular: `.kunsdd/`, `README.md` (se explicitamente exempt).

2. Cross-references: todo `RX-NNN` / `HA-NNN` deve resolver para
   uma seção `### RX-NNN` / `### HA-NNN` no mesmo arquivo ou
   em `RISKS.md` / `HUMAN_ACTIONS.md`.
   → Falha: referência órfã.

3. Tabela sumário vs. detalhe: se arquivo tem seção de detalhes
   (`### RX-NNN`) E uma tabela sumária, o `Status` de cada linha
   da tabela deve ser igual ao campo `Lifecycle`/`Status` da seção
   de detalhe correspondente.
   → Falha: divergência (como RX-007).

4. Denylist de termos removidos: qualquer ocorrência de
   `indexFundos|_Anlagentyp|Kurs_Import|...` (fora de seção
   `## History` ou `(REMOVED)`) → falha.

5. Anchor check: links `#section-name` devem resolver para um
   heading real no mesmo arquivo.
   → Falha: âncora quebrada.

6. Updated bump: se diff de conteúdo não for apenas whitespace,
   o `Updated:` no header deve ser mais recente que o commit
   anterior do arquivo.
   → Falha (warning, não erro): esquema de changelog.
```
```

### Adição 4 — Tabela de decisão: qual coluna é a correta?

Isto resolve a ambiguidade do Tier 1 (as 3 versões de mapa de colunas):

```markdown
### Decisão: mapeamento canônico de colunas EinAus (após inserção do FelipeCode)

**Fonte:** WORKBOOK_FIFO.md (confirmado DEV, Junho 2026)

| Coluna | Campo |
|--------|-------|
| B | FelipeCode |
| C | Datum |
| D | Anteile |
| E | Kurs R$ |
| F | (vazia/não usada) |
| G | Gebühren R$ |
| H | Action |
| I | Kurs € |
| J | Gebühren € |
| ... | ... |
| AA | Anlagentyp |
| AC | Is latest |

**Ação:** DATAFLOW.md e SCHEMAS.md devem ser atualizados para refletir
este mapeamento. As seções de FIFO-workbook em SCHEMAS.md devem ser
substituídas por um link para WORKBOOK_FIFO.md#einaus.
```

### Adição 5 — Seção "Critérios de Aceitação"

Adicionar ao final do req:

```markdown
## Critérios de Aceitação

1. **Tier 1 resolvido:** Nenhum mapa de colunas EinAus, Payouts ou
   Staging existe em mais de um arquivo. Os maps residem apenas em
   WORKBOOK_FIFO.md; os demais docs linkam para lá.
2. **Tier 2 resolvido:** RISKS.md RX-007 tem mesmo status no sumário
   e no detalhe. OPERATIONS.md mostra Workflow D e F com banner de
   depreciação. INVENTORY.md reflete estado real (não "Ready for Phase 1"
   para algo já implementado e deprecado).
3. **Tier 3 resolvido:** Nenhuma ocorrência de IndexFundos, ou de
   seções duplicadas, fora de seção `(REMOVED)`/`## History`.
4. **docs-lint implementado:** pipeline CI falha se qualquer regra
   for violada.
5. **Checklist seguido:** toda remoção futura passa pelo checklist
   de propagação.
```

---

**Resumo:** O req atual é um diagnóstico brilhante, mas precisa de (a) decisões explícitas sobre qual versão é a verdade, (b) especificações implementáveis (lint rules, checklist), (c) critérios de aceitação claros, e (d) definição de onde esses documentos realmente estão. Sem isso, as recomendações continuam sendo boas intenções sem executor.
