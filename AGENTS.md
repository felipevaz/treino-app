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

O aplicativo contém dois treinos independentes:

- **Treino A**
- **Treino B**

Cada treino possui seus próprios exercícios, cronômetros e persistência independente.

A alternância entre treinos é feita por abas no topo da página e preserva o estado do DOM (cronômetros e botões não são recriados).

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

Iniciado ao clicar:

```
✔ Descanso
```

Requisitos:

- botão ativo deve piscar
- cronômetro deve mudar para a cor do botão ativo
- após chegar a zero deve continuar em números negativos
- formato sempre:

```
0:30
0:00
-0:15
```

Nunca:

```
00:30
00:00
```

### **Limitação do formato**

O formato atual sempre usa `0:` prefixado (minuto único). Funciona corretamente apenas para valores entre 0 e 59 segundos.

Se no futuro um exercício usar `time` ou `rest` ≥ 60 segundos, a função `format()` precisará ser atualizada para calcular minutos reais.

---

## **Layout**

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

- nome
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

- cronômetro
- 3 séries
- botão iniciar
- botão descanso

Layout:

cronômetro acima

abaixo:

linha 1:

- Série 1
- Descanso

linha 2:

- Série 2
- Descanso

linha 3:

- Série 3
- Descanso

Botões devem possuir largura uniforme.

### **Botão de série concluída (.done)**

Quando o usuário clica em "✔ Descanso" para uma série:

- O botão "▶ Série N" correspondente recebe:
  - `disabled = true`
  - classe `.done` adicionada
  - texto alterado para "Feito"
- A classe `.done` aplica opacidade reduzida e cor cinza, indicando visualmente que a série foi concluída.
- A mesma série não pode ser reiniciada após o descanso.

---

## **Alternância entre Treinos**

### **Renderização inicial**

Ambos os treinos são renderizados no DOM durante o `init`:

```
renderWorkout("A");
renderWorkout("B");
```

Cada treino fica em um container `<div class="workout" id="workoutA">` / `<div class="workout" id="workoutB">`.

### **Mecanismo de exibição**

- Ambos os containers existem no DOM desde o carregamento inicial.
- A classe CSS `.workout` define `display: none`.
- A classe `.workout.visible` define `display: block`.
- Alternar entre treinos é puramente uma troca de classes CSS → **sem recriação de elementos**.
- Cronômetros e estado dos botões são preservados durante a alternância.

### **Abas**

Dois botões `.tab-btn` no topo, com atributo `data-workout="A"` e `data-workout="B"`.
O botão ativo recebe a classe `.tab-btn.active`.

Função `switchWorkout(key)`:
1. Atualiza classe `active` nos botões de aba
2. Alterna classe `visible` nos containers `.workout`
3. Salva preferência em `localStorage` (`treino_selected`)

### **Restauração**

No `init`, após renderizar e configurar ambos os treinos, o último treino selecionado é restaurado do `localStorage` (padrão: `"A"`).

---

## **Persistência**

Implementada via `localStorage`.

Os campos `contenteditable` (reps, carga, observações) são salvos automaticamente no `input` e restaurados ao carregar a página.

### **Chaves**

Cada campo usa a chave `treino{tipo}_{campo}_{indice}`, onde:

- `{tipo}` = `A` ou `B` (treino)
- `{campo}` = `reps`, `load` ou `obs`
- `{indice}` = posição do exercício no array (0-based)

Exemplo:

```
treinoA_reps_0   → reps do 1º exercício do Treino A (Mobilidade)
treinoB_load_2   → carga do 3º exercício do Treino B (Remada)
treinoA_obs_4    → observações do 5º exercício do Treino A
```

### **Preferência de treino**

O treino selecionado é salvo na chave `treino_selected` (valor `"A"` ou `"B"`) e restaurado ao carregar a página.

### **Função responsável**

`setupPersistence(key)` em `serie-2026.html`, chamada uma vez para cada treino:

1. Localiza o container `#workout{key}`
2. Itera todos os `.exercise` dentro dele
3. Localiza os spans `[contenteditable]` e a div `.obs[contenteditable]`
4. Restaura valores salvos do `localStorage` (se existirem)
5. Registra `input` listener para salvar alterações

Chamada no INIT após `renderWorkout()` e `bindButtons()`.

### **Observações**

- Apenas campos editáveis são persistidos (não há salvamento de estado dos botões).
- Ao recarregar a página, os botões de série voltam ao estado inicial; apenas reps, carga e observações são restaurados.
- `localStorage` é síncrono e adequado para uso offline.

---

## **Histórico**

Implementado via `treino_log` no `localStorage`. Registrado automaticamente ao concluir todas as séries de um treino.

Registrar:

- data (`date`)
- tipo do treino (`type`: A/B)
- timestamp de conclusão (`completedAt`)

Estrutura: `{ date, type, completedAt }` em array JSON.

---

## **Alternância Inteligente**

O app sempre abre no **Dashboard**. Ao clicar em "Treino A" ou "Treino B", o respectivo treino é exibido. Não há sugestão automática — o usuário escolhe qual treino fazer.

---

## **Conclusão do Treino**

O treino pode ser concluído de duas formas:

1. **Automática**: quando todas as 3 séries de todos os exercícios com cronômetro são marcadas como Feito.
2. **Manual**: botão "✅ Concluir treino" no final de cada treino — útil para registrar a conclusão mesmo que a mobilidade ou séries opcionais não tenham sido feitas.

Em ambos os casos:

- Banner verde "Treino A/B concluído! 🎉" aparece no rodapé
- Após 2 segundos, o banner desaparece e o **Dashboard** é aberto automaticamente

---

## **Dashboard**

Aba "Dashboard" na barra de navegação, ao lado de Treino A / Treino B.

### **Cartões**

| Cartão | Descrição |
|--------|-----------|
| Último treino | Tipo + data formatada |
| Total de treinos | Contagem total de sessões |
| Sequência atual | Dias consecutivos (streak) |
| Distribuição A/B | Quantos treinos de cada tipo |
| Últimos 7 dias | Bolinhas ✔ (treinou) / ✗ (não treinou) |

### **Funções**

- `showDashboard()` — ativa a tela e chama `renderDashboard()`
- `renderDashboard()` — lê `treino_log`, calcula métricas e preenche o container

---

## **Exportação**

Implementado via botão "📥 Exportar histórico (CSV)" no Dashboard. Gera download CSV com colunas `date,type,completedAt`.

Formato futuro:

- JSON

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

## **Objetivo de Longo Prazo**

Transformar o arquivo HTML em um treinador pessoal simples para Android, mantendo:

- simplicidade
- robustez
- funcionamento offline
- facilidade de edição por IA e humanos

Ver o [PLAN.md](PLAN.md) para o roadmap detalhado de evolução.