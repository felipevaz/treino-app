# Plano de Evolução — Treino App

## Princípio

O aplicativo `serie-2026.html` é a referência operacional. A documentação descreve o que está implementado no HTML; o registro em `TREINOS.md` serve para revisar periodicamente a composição dos treinos antes de alterar o app.

O produto continua sendo um arquivo HTML único, offline, sem backend e sem frameworks externos.

## Estado atual

- Dashboard é a tela inicial.
- Há dois modos: `Treino curto` e `Treino longo`.
- O treino curto tem 6 exercícios, incluindo mobilidade e Suitcase Carry.
- O treino longo tem 9 exercícios, incluindo mobilidade, pernas, empurrar, puxar, hinge e core.
- Cada exercício normal tem três séries de 40 segundos, com 30 segundos de descanso visual na linha superior.
- Hanging Knee Raise é o exercício principal de flexão/controle da pelve do core.
- Pallof Press e Suitcase Carry cumprem funções complementares de estabilidade.
- Around the world não é obrigatório no treino principal; permanece uma opção da rotina de mobilidade/coordenação.
- Rotação externa do antebraço e Sleeper Stretch ficam fora do treino principal, na rotina matinal.
- Campos de repetições, carga e observações são persistidos por modo em `localStorage`.
- O histórico é persistido em `treino_log` e pode ser exportado pelo Dashboard em CSV.

## Cronômetro

O cronômetro atual deve ser preservado. A interface exibe algarismos simples, como `40`, `0` e `-1`, com contagem negativa depois do fim. Não há exercício atual acima de um minuto; mesmo 90 segundos continua compatível com a decisão atual. Não alterar o formato sem uma necessidade real.

A lógica do modal e o DOM dos treinos são áreas de alto risco. Mudanças de tela devem apenas alternar visibilidade; não recriar cards enquanto houver timer ativo.

## Modos de treino

### Treino curto

Usado em dias de pedalada forte, fadiga elevada ou pouco tempo. Mantém mobilidade, parte superior, core e estabilidade sem exigir o bloco completo de pernas.

### Treino longo

Usado em dias leves ou com melhor recuperação. Inclui o trabalho de pernas e cadeia posterior, além do conjunto completo de puxar, empurrar e core.

O aplicativo não decide automaticamente a intensidade da pedalada. A escolha do modo permanece manual para evitar uma regra falsa baseada apenas em quilometragem.

## Dashboard

O Dashboard deve permanecer disponível como tela inicial e ser acessível durante o uso. Ele lê o `treino_log` ao ser exibido e apresenta:

- total de treinos;
- último modo concluído;
- atividade dos últimos sete dias;
- distribuição entre curto e longo;
- exportação CSV.

O Dashboard não deve controlar nem reiniciar timers.

## Histórico e compatibilidade

O formato atual é:

```json
{ "date": "2026-07-14", "type": "longo", "completedAt": "2026-07-14T07:10:00.000Z" }
```

Registros antigos com `type: "A"` são tratados como `longo`; registros com `type: "B"` são tratados como `curto`. O app continua sobrescrevendo a conclusão do mesmo dia.

## Registro separado dos treinos

`TREINOS.md` é o registro humano para revisão de exercícios, ordem, volume e critérios de uso. Ele não é carregado pelo navegador e não substitui os dados embutidos no HTML. Depois de cada revisão, o HTML deve ser atualizado e testado; se houver divergência, o comportamento do app é a referência até a próxima sincronização deliberada.

## Próximas evoluções

1. Observar o uso real dos modos curto/longo antes de criar mais variações.
2. Melhorar o Dashboard sem alterar o modal do timer.
3. Considerar uma indicação explícita de “pedalada forte” apenas se isso simplificar a escolha manual.
4. Avaliar progressão de carga do levantamento terra KB e do Suitcase Carry.
5. Manter around the world como opção separada, sem torná-lo requisito do treino.

## Riscos

| Risco | Mitigação |
|---|---|
| Timer interrompido por navegação | alternar classes de visibilidade; não re-renderizar treinos durante a execução |
| Dados antigos desaparecerem | fallback das chaves antigas para o modo longo |
| Curto virar cópia incompleta do longo | revisar os padrões cobertos em `TREINOS.md` |
| Dashboard crescer demais | manter métricas simples e leitura direta do histórico |
| Registro separado divergir do app | usar o HTML como referência operacional e sincronizar após revisões |
