# **AGENTS.md**

## **Projeto**

Aplicativo web simples para acompanhamento de treino de musculação/mobilidade.

Objetivo principal:

- Executar treinos em um celular Android.
- Funcionar totalmente offline.
- Ser distribuído como um único arquivo HTML.
- Não depender de backend.
- Não depender de frameworks externos (React, Vue, Angular etc.).
- Manter simplicidade e legibilidade do código.

Arquivo principal:

- `serie-2026.html`

### **Estrutura de treinos**

O aplicativo contém dois modos do mesmo programa: **treino curto** e **treino longo**. O curto é usado em dias de pedalada forte ou menor disponibilidade; o longo é a sessão completa.

---

## **Filosofia**

Prioridades em ordem:

1. Confiabilidade do cronômetro
2. Facilidade de uso durante o treino
3. Compatibilidade com Android
4. Simplicidade do código
5. Aparência visual

Uma mudança visual nunca deve comprometer o funcionamento do cronômetro.

---

## **Restrições Arquiteturais**

### **Arquivo único**

Todo o aplicativo deve permanecer em um único arquivo HTML contendo:

- HTML
- CSS
- JavaScript

Não criar:

- build steps
- npm
- webpack
- vite
- typescript
- transpilers

---

### **Sem backend**

Não utilizar:

- servidores
- bancos de dados
- APIs externas obrigatórias

O aplicativo deve funcionar sem internet após carregado.

---

### **Estado do aplicativo**

Evitar renderizações completas da interface.

Não utilizar padrões do tipo:

```javascript
render()
app.innerHTML = ...
```

após cliques em botões de treino.

Motivo:

Versões anteriores apresentaram bugs de cronômetro porque elementos eram recriados durante timers ativos.

Preferir:

- atualização pontual do DOM
- manipulação direta dos elementos existentes

---

## **Cronômetros**

Existem dois tipos de contagem:

### **Série**

Iniciada ao clicar:

```
▶ Série N
```

### **Descanso**

Disparado ao finalizar a série no modal:

```
Finalizar
```

Requisitos:

- botão ativo deve piscar
- cronômetro deve mudar para a cor do botão ativo
- após chegar a zero deve continuar em números negativos
- formato atual:

```
40
0
-1
```

Nunca:

```
00:30
00:00
```

### **Decisão sobre o formato**

O formato numérico atual deve ser mantido. Os treinos usam 40 segundos de série e 30 segundos de descanso; mesmo 90 segundos continua aceitável nesse formato. Só revisar essa decisão se o programa passar a exigir durações maiores.

---

## **Layout**

### **Dashboard**

O Dashboard é a tela inicial e pode ser aberto a qualquer momento. Ele mostra o total de treinos, o último modo concluído, os últimos sete dias, a distribuição entre curto e longo e o botão de exportação CSV. Ele deve permanecer separado dos containers de treino para não interferir nos cronômetros.

### **Fonte**

Utilizar:

```
Outfit
```

em toda a aplicação.

Inclui:

- títulos
- texto
- botões
- observações

---

### **Estrutura dos exercícios**

Cada exercício deve conter:

- nome (com badge numerado)
- reps
- carga
- observações

Opcionalmente:

- timer
- séries
- lista de movimentos

---

### **Exercícios de mobilidade**

Não possuem:

- cronômetro
- séries
- botões

São exibidos apenas como lista.

---

### **Exercícios normais**

Possuem:

- cronômetro em modal
- 3 séries
- botão iniciar para cada série

O botão `Finalizar` do modal encerra a série, dispara o descanso visual e marca a série como concluída. Os botões de série devem possuir largura uniforme.

### **Botão de série concluída (.done)**

Quando o usuário finaliza uma série no modal:

- O botão "Série N" correspondente recebe:
  - `disabled = true`
  - classe `.done` adicionada
  - texto alterado para "Feito"
- A classe `.done` aplica opacidade reduzida e cor cinza, indicando visualmente que a série foi concluída.
- A mesma série não pode ser reiniciada após o descanso.

---

## **Persistência**

Implementada via `localStorage`.

Os campos `contenteditable` (reps, carga, observações) são salvos automaticamente no `input` e restaurados ao carregar a página.

### **Chaves**

Cada campo usa a chave `treino_{modo}_{campo}_{indice}`, onde:

- `{modo}` = `curto` ou `longo`
- `{campo}` = `reps`, `load` ou `obs`
- `{indice}` = posição do exercício no array (0-based)

Exemplo:

```
treino_longo_reps_0   → reps do 1º exercício longo (Mobilidade)
treino_curto_load_2   → carga do 3º exercício curto (Flexão)
treino_longo_obs_7    → observações do 8º exercício longo (Hanging Knee Raise)
```

### **Função responsável**

`setupPersistence()` em `serie-2026.html`, chamada uma vez no `init`:

1. Localiza o container do modo (`#workout-curto` ou `#workout-longo`)
2. Itera todos os `.exercise` dentro dele
3. Localiza os spans `[contenteditable]` e a div `.obs[contenteditable]`
4. Restaura valores salvos do `localStorage` (se existirem)
5. Registra `input` listener para salvar alterações

Chamada no INIT após `renderWorkout(modo)` e `bindButtons(modo)`.

### **Observações**

- Apenas campos editáveis são persistidos (não há salvamento de estado dos botões).
- Ao recarregar a página, os botões de série voltam ao estado inicial; apenas reps, carga e observações são restaurados por modo.
- `localStorage` é síncrono e adequado para uso offline.

---

## **Histórico**

Implementado via `treino_log` no `localStorage`. Registrado automaticamente ao concluir todas as séries do treino.

Registrar:

- data (`date`)
- tipo do treino (`type`: `curto` ou `longo`)
- timestamp de conclusão (`completedAt`)

Estrutura: `{ date, type, completedAt }` em array JSON. Registros legados `A` são tratados como `longo` e `B` como `curto`.

---

## **Conclusão do Treino**

O treino pode ser concluído de duas formas:

1. **Automática**: quando todas as 3 séries de todos os exercícios com cronômetro são marcadas como Feito.
2. **Manual**: botão "Concluir treino" no final — útil para registrar a conclusão mesmo que a mobilidade ou séries opcionais não tenham sido feitas.

Em ambos os casos:

- Overlay de celebração "Treino concluído!" aparece após a conclusão
- Após 5 segundos, o overlay desaparece

---

## **Exportação**

Botão "Exportar histórico (CSV)" disponível no Dashboard (export via `exportHistory()`). Gera download CSV com colunas `date,type,completedAt`.

---

## **Estilo de Código**

Preferências:

- JavaScript simples (ES6)
- funções pequenas
- nomes explícitos
- comentários curtos

Evitar:

- abstrações excessivas
- classes complexas
- dependências desnecessárias

---

## **Processo para Alterações**

Antes de modificar o aplicativo:

1. Identificar o comportamento atual.
2. Confirmar se a alteração afeta timers.
3. Confirmar se a alteração afeta layout mobile.
4. Preservar compatibilidade offline.
5. Evitar regressões nos cronômetros.

Mudanças em timers devem ser tratadas como alterações de alto risco.

---

## **Design Visual**

### **Paleta**

```css
--bg: #0b1120;        /* fundo profundo */
--card: #0f1a2e;      /* cartão azul-escuro sutil */
--text: #e5e7eb;      /* texto principal */
--muted: #94a3b8;     /* texto secundário */
--neg: #ef4444;       /* negativo/erro */
--tab: #1e293b;       /* aba inativa */
--accent: #f59e0b;    /* destaque (streak) */
--border: #1e293b;    /* borda de cartões */
```

### **Componentes**

| Componente | Detalhe |
|------------|---------|
| Exercise card | `--card` + `1px solid --border` |
| Badge numerado | Círculo verde `#22c55e` de 28px no título |
| Mobilidade | Lista com chips `#020617` e bullet `○` |
| Timer | Fundo `#020617` separado, `font-size: 3em` |
| Botões | `box-shadow` + `:active` com `scale(0.96)` |
| Histórico | Exportação CSV ao final da página |
| Transições | `fadeIn` (opacity + translateY) no banner |

---

## **Registro de revisão**

`TREINOS.md` mantém, fora do HTML, o registro humano dos exercícios, da ordem, do objetivo de cada modo e das decisões de revisão. Ele não é carregado pelo navegador e não substitui o arquivo HTML distribuível. Se houver divergência, o comportamento implementado no HTML é a referência operacional até uma sincronização deliberada.

## **Objetivo de Longo Prazo**

Transformar o arquivo HTML em um treinador pessoal simples para Android, mantendo:

- simplicidade
- robustez
- funcionamento offline
- facilidade de edição por IA e humanos

Ver o [PLAN.md](PLAN.md) para o roadmap detalhado de evolução.
