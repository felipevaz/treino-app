# Plano de Evolução — Treino App

## Objetivo

Evoluir o aplicativo de **executor de treino** para um **companheiro de treino** que ajude a manter consistência ao longo do tempo, sem perder a simplicidade offline de arquivo único.

---

## Visão Geral

Três telas alternadas via show/hide (mesmo princípio do A/B atual), preservando o DOM e os cronômetros:

| Tela | Função |
|------|--------|
| **Treino** | Execução dos treinos A/B (como hoje) |
| **Dashboard** | Métricas de consistência e motivação visual |
| **Histórico** | (futuro) Lista cronológica de sessões |

Nenhuma funcionalidade nova interfere nos cronômetros ou na execução do treino.

---

## 1. Registro de Execução

### Quando marcar como concluído

O treino é considerado concluído quando **todas as 3 séries de todos os exercícios com cronômetro** estiverem marcadas como Feito (`.done`). Exercícios de mobilidade são opcionais — não bloqueiam a conclusão.

### Gatilho

Assim que a última série do último exercício receber "✔ Descanso":

1. Mostrar banner **"Treino A concluído! 🎉"** (some após 3 s)
2. Registrar sessão no `localStorage`
3. Oferecer botão "Ver Dashboard"

### Tratamento de repetições

Se o usuário refizer o mesmo treino no mesmo dia, o registro anterior é sobrescrito (apenas a última conclusão do dia conta).

---

## 2. Estrutura dos Dados Persistidos

Tudo em `localStorage`, chave única para histórico:

```
treino_log → JSON Array
```

```json
[
  { "date": "2026-06-20", "type": "A", "completedAt": "2026-06-20T07:10:00" },
  { "date": "2026-06-21", "type": "B", "completedAt": "2026-06-21T07:05:00" }
]
```

Tamanho: ~150 bytes/sessão. 2 anos de treino (~500 sessões) = ~75 KB — folgado no limite de 5 MB do `localStorage`.

### Chaves existentes (mantidas)

- `treinoA_reps_0`, `treinoA_load_0`, `treinoA_obs_0`, etc.
- `treino_selected`

---

## 3. Dashboard

Cartões simples, empilhados verticalmente no mobile:

| Cartão | Cálculo |
|--------|---------|
| **Último treino** | Última entrada do `treino_log` |
| **Total de treinos** | `treino_log.length` |
| **Sequência atual** | Dias consecutivos sem falha (1 treino/dia) |
| **Distribuição A/B** | Contagem por tipo no log |
| **Últimos 7 dias** | Bolinhas ✔ (treinou) / ✗ (não treinou) |

### Regra da sequência (streak)

1. Ordenar `treino_log` por data decrescente
2. Contar dias consecutivos a partir de hoje
3. Se hoje ainda não foi feito, considerar streak a partir de ontem
4. Um dia sem treino zera a streak

### Atualização

Dashboard lê do `treino_log` sempre que a tela é exibida. Não precisa de atualização em tempo real.

---

## 4. Fluxo de Uso

```
1. Abre o app
   ├→ Tela Treino com última aba usada
   └→ Aba "Dashboard" disponível no topo

2. Treinar:
   a. Verifica o treino do dia (A ou B)
   b. Executa séries normalmente
   c. Último Descanso → banner de conclusão + registro

3. Dashboard:
   a. Aba "Dashboard" → cartões com estatísticas
   b. Volta para "Treino" quando quiser

4. Dia seguinte:
   Sugestão automática: se fez A ontem, mostra B
```

### Alternância inteligente

Baseado no `treino_log`: se o último foi A, sugere B, e vice-versa. Elimina a decisão matinal.

---

## 5. Etapas de Implementação

### ✅ Etapa 1 — Registro de conclusão (implementado)
- Detectar quando todas as séries de todos os exercícios estão Feito
- Salvar `treino_log` no `localStorage`
- Banner de conclusão com auto-dismiss (4 s)
- Ignorar mobilidade na contagem

### ✅ Etapa 2 — Alternância inteligente (implementado)
- Ao carregar, verificar último treino no log
- Sugerir o outro automaticamente
- Padrão: A se não houver histórico

### ✅ Etapa 3 — Dashboard (implementado)
- Container `#screen-dashboard` (show/hide via classe `.screen`)
- Aba "Dashboard" na barra de navegação
- Cartões: último treino, total, streak, distribuição A/B, últimos 7 dias
- Botão "Exportar histórico (CSV)"

### ✅ Etapa 4 — Exportação (implementado)
- Botão no Dashboard → download CSV via Blob

---

## 6. Riscos

| Risco | Mitigação |
|-------|-----------|
| Dashboard recriar elementos e afetar timers | Dashboard é tela separada sem cronômetros — show/hide seguro |
| Histórico consumir muito `localStorage` | 2 anos = ~75 KB, irrisório |
| Sequência confusa se treinar 2x no dia | Contar 1 treino/dia; múltiplos no mesmo dia atualizam sem contar dobro |
| Usuário fechar app antes de concluir | Estado preservado no DOM; pode retomar; não há conclusão parcial |
