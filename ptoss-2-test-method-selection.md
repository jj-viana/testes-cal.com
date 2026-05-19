# PTOSS-2: Seleção de Métodos para Testes Unitários

Este documento registra a seleção inicial de métodos para a atividade PTOSS-2, considerando uma equipe com seis integrantes. Para cada função, foram mapeadas a funcionalidade, as técnicas de caixa-preta e caixa-branca aplicáveis, os principais casos de teste, os branches esperados e a justificativa da escolha.

## 1. `getPrefetchMonthCount`

Arquivo: `packages/features/bookings/Booker/utils/getPrefetchMonthCount.ts`

| Item | Descrição |
| --- | --- |
| Funcionalidade | Determina se o calendário deve buscar dois meses de disponibilidade ou não, considerando layout, estado do booker, meses exibidos e prefetch já ativo. |
| Técnica caixa-preta usada | Particionamento de equivalência por layout (`COLUMN_VIEW`, `WEEK_VIEW`, `MONTH_VIEW`), por estado (`selecting_date`, `selecting_time`, `booking`), por relação entre meses (iguais, diferentes, inválidos) e por `prefetchNextMonth` (`true`/`false`). |
| Técnica caixa-branca usada | Cobertura de branches e MC/DC na decisão composta `!isWeekView && isSelectingTime && !prefetchNextMonth`. |
| Casos de teste | Meses iguais retornam `undefined`; meses diferentes em `COLUMN_VIEW` retornam `2`; `WEEK_VIEW` sempre retorna `undefined`; `MONTH_VIEW` com `selecting_time` e `prefetchNextMonth=false` retorna `2`; `MONTH_VIEW` com `selecting_time` e `prefetchNextMonth=true` retorna `undefined`; mês `NaN` ou infinito retorna `undefined`. |
| Branches cobertos | `!isDifferentMonth`; `isColumnView`; decisão composta para month view; retorno final `undefined`. |
| Justificativa | É uma função pequena, pura e com regras condicionais claras. Permite demonstrar bem a diferença entre testar comportamento esperado e garantir cobertura estrutural de uma condição composta. |

## 2. `getAvailabilityFromSchedule`

Arquivo: `packages/lib/availability.ts`

| Item | Descrição |
| --- | --- |
| Funcionalidade | Converte uma agenda semanal em uma lista de disponibilidades agrupadas por horários iguais. |
| Técnica caixa-preta usada | Particionamento de equivalência por tipo de agenda: vazia, dias com mesmo horário, dias com horários diferentes e múltiplos horários no mesmo dia. |
| Técnica caixa-branca usada | Cobertura de branches no `findIndex`: quando um horário já existe na disponibilidade e quando ainda não existe. |
| Casos de teste | Agenda vazia retorna `[]`; segunda a sexta com `09:00-17:00` retorna uma disponibilidade com `days: [1, 2, 3, 4, 5]`; dias com manhã/tarde separados geram grupos diferentes; mesmo horário em dias não consecutivos é agrupado no mesmo objeto. |
| Branches cobertos | `findIndex !== -1`; `findIndex === -1`; `filteredTimes.forEach`; redução em dias sem horários. |
| Justificativa | Boa função para mostrar caixa-preta baseada no comportamento esperado do domínio: agrupar disponibilidades por horário. A caixa-branca ajuda a perceber que é preciso testar tanto a criação de um novo grupo quanto o reaproveitamento de grupo existente. |

## 3. `parseTimeString`

Arquivo: `packages/features/schedules/components/ScheduleComponent.tsx`

| Item | Descrição |
| --- | --- |
| Funcionalidade | Converte uma string de horário em `Date`, respeitando formato 12h ou 24h, ou retorna `null` para entradas inválidas. |
| Técnica caixa-preta usada | Análise de valor limite e particionamento por formato: entradas vazias, formato 24h, formato 12h, horários mínimos/máximos válidos e valores inválidos. |
| Técnica caixa-branca usada | Cobertura de branches: entrada vazia, parsing inválido, validação de hora/minuto e retorno válido. |
| Casos de teste | `""` e `"   "` retornam `null`; `"00:00"` é válido; `"23:59"` é válido; `"24:00"` é inválido; `"16:60"` é inválido; `"4:05pm"` é válido; `"12:00am"` vira `00:00`; `"12:00pm"` vira `12:00`; `"invalid"` retorna `null`. |
| Branches cobertos | `!input.trim()`; `timeFormat === 12`; `!parsed.isValid()`; validação `hours > 23`; validação `minutes > 59`; retorno com `Date`. |
| Justificativa | Excelente para análise de valor limite, porque horários têm fronteiras naturais: `00:00`, `23:59`, `24:00`, minuto `59` e minuto `60`. |

## 4. `computeEffectiveStateAcrossTeams`

Arquivo: `packages/features/feature-opt-in/lib/computeEffectiveState.ts`

| Item | Descrição |
| --- | --- |
| Funcionalidade | Calcula se uma feature opt-in está efetivamente habilitada considerando estado global, organização, times, usuário e política permissiva ou strict. |
| Técnica caixa-preta usada | Tabela de decisão com combinações de estados: global desligado, organização desligada, time habilitado/desabilitado, usuário habilitado/desabilitado e política `strict` ou `permissive`. |
| Técnica caixa-branca usada | Cobertura de branches e MC/DC nas decisões compostas, como `userEnabled && !hasExplicitEnablementAboveUser` e `!userEnabled && !hasExplicitEnablementAboveUser`. |
| Casos de teste | `globalEnabled=false` sempre desabilita; `orgState=disabled` desabilita; política `strict` com qualquer time desabilitado bloqueia; política `strict` com usuário habilitado mas sem org/time habilitado bloqueia; política permissiva com usuário habilitado habilita; política permissiva com todos os times desabilitados bloqueia; ausência de habilitação explícita retorna desabilitado. |
| Branches cobertos | `!globalEnabled`; `orgDisabled`; `policy === "strict"`; `anyTeamDisabled`; `userEnabled && !hasExplicitEnablementAboveUser`; `userDisabled`; `!hasExplicitEnablementAboveUser`; `allTeamsDisabled`; retorno habilitado. |
| Justificativa | É uma das melhores funções para a atividade porque permite uma tabela de decisão robusta e uma análise estrutural rica. Também mostra claramente a precedência entre regras. |

## 5. `intersect`

Arquivo: `packages/features/schedules/lib/date-ranges.ts`

| Item | Descrição |
| --- | --- |
| Funcionalidade | Calcula os intervalos de tempo comuns entre múltiplas listas de disponibilidade. |
| Técnica caixa-preta usada | Particionamento por relação entre intervalos: listas vazias, sem interseção, interseção parcial, intervalo contido, múltiplos usuários e intervalos encostados. |
| Técnica caixa-branca usada | Cobertura de branches no loop principal: sem ranges, saída antecipada, interseção encontrada, interseção não encontrada, avanço de `commonIndex` e avanço de `userIndex`. |
| Casos de teste | `[]` retorna `[]`; `[[]]` retorna `[]`; uma lista única retorna ela mesma; dois intervalos sem sobreposição retornam `[]`; dois intervalos com sobreposição retornam apenas a parte comum; intervalo totalmente contido retorna o menor; intervalos que apenas encostam, com fim igual ao início, não geram interseção. |
| Branches cobertos | `!ranges.length`; `commonAvailability.length === 0`; `intersectStartValue < intersectEndValue`; `commonRange.endValue <= userRange.endValue`; ramo `else` que avança `userIndex`. |
| Justificativa | Boa função para análise de valor limite com datas e horários. A fronteira mais importante é quando dois intervalos encostam, pois o código exige `start < end`, não `start <= end`. |

## 6. `subtract`

Arquivo: `packages/features/schedules/lib/date-ranges.ts`

| Item | Descrição |
| --- | --- |
| Funcionalidade | Remove intervalos excluídos de uma lista de intervalos base, retornando os pedaços restantes. |
| Técnica caixa-preta usada | Particionamento por tipo de sobreposição: nenhuma sobreposição, exclusão antes, exclusão depois, exclusão total, exclusão no início, exclusão no fim e exclusão no meio. |
| Técnica caixa-branca usada | Cobertura de branches: `break`, `continue`, criação de trecho anterior à exclusão, atualização de `currentStart` e criação do trecho restante final. |
| Casos de teste | Excluded range antes do source mantém source inteiro; excluded depois do source mantém source inteiro; excluded cobrindo tudo retorna `[]`; excluded no meio divide em dois intervalos; excluded no início retorna apenas o final; excluded no fim retorna apenas o começo; múltiplas exclusões retornam múltiplos pedaços. |
| Branches cobertos | `excludedRange.start >= sourceEnd`; `excludedRange.end <= currentStart`; `excludedRange.start > currentStart`; `excludedRange.end > currentStart`; `sourceEnd > currentStart`. |
| Justificativa | Muito adequada para combinar caixa-preta e caixa-branca, porque a especificação funcional é intuitiva e os branches internos correspondem diretamente aos tipos de sobreposição entre intervalos. |

## Resumo da Distribuição

| Integrante | Função sugerida | Força principal |
| --- | --- | --- |
| Rodrigo | `getPrefetchMonthCount` | MC/DC e regras condicionais |
| Eduardo | `getAvailabilityFromSchedule` | Agrupamento funcional e branches simples |
| Anderson | `parseTimeString` | Análise de valor limite |
| Joaquim | `computeEffectiveStateAcrossTeams` | Tabela de decisão e MC/DC |
| Bruno | `intersect` | Interseção de intervalos |
| John | `subtract` | Subtração de intervalos e cobertura de branches |

## Observação

A seleção combina funções pequenas e testáveis com funções de maior riqueza estrutural. Isso ajuda a demonstrar a complementaridade entre caixa-preta e caixa-branca sem transformar o trabalho em uma análise excessivamente grande ou dependente de banco de dados, UI ou serviços externos.

## O Que Ainda Falta Para a Atividade

Esta seleção de métodos resolve apenas a primeira decisão do trabalho: quais funções serão analisadas e testadas. Para atender completamente ao enunciado da PTOSS-2, ainda falta produzir as evidências, implementar ou complementar testes, aplicar TDD em uma mudança real e consolidar tudo no relatório.

| Necessidade da atividade | Situação atual | O que falta fazer |
| --- | --- | --- |
| Escolher métodos em quantidade mínima igual ao número de integrantes | Seis funções foram selecionadas para seis integrantes. | Confirmar se a equipe aceita a distribuição e se cada integrante ficará responsável por uma função. |
| Projetar testes caixa-preta | Técnicas e casos principais foram mapeados neste documento. | Transformar os casos em uma tabela mais detalhada por função, com entradas, saída esperada e técnica aplicada. |
| Projetar testes caixa-branca | Branches principais foram identificados. | Conferir a cobertura real após executar os testes e ajustar os casos para cobrir branches não exercitados. |
| Evidenciar MC/DC | Há bons candidatos: `getPrefetchMonthCount` e `computeEffectiveStateAcrossTeams`. | Montar uma tabela MC/DC mostrando como cada condição independente altera o resultado da decisão. |
| Implementar testes unitários | Algumas funções já possuem testes no projeto. | Verificar os testes existentes, complementar lacunas e garantir que cada integrante tenha contribuição rastreável. |
| Gerar métricas de cobertura | Ainda não feito neste documento. | Executar testes com cobertura e salvar o relatório/print ou saída do terminal. |
| Demonstrar complementaridade entre caixa-preta e caixa-branca | A justificativa inicial está descrita por função. | Escrever uma análise comparando casos funcionais com os branches realmente cobertos. |
| Aplicar TDD | Ainda não definido. | Escolher uma melhoria pequena, criar teste falhando, implementar o mínimo e refatorar. |
| Documentar Red, Green e Refactor | Ainda não feito. | Fazer commits separados ou registrar evidências do teste falhando, passando e da refatoração. |
| Produzir análise crítica | Ainda não feito. | Escrever reflexão sobre testabilidade, dificuldades, limitações das técnicas e impacto do TDD. |
| Preparar relatório técnico | Ainda não feito. | Montar o relatório com introdução, projeto, planejamento, técnicas, testes, cobertura, TDD, análise crítica e conclusão. |
| Preparar repositório final | Parcial. | Incluir instruções de execução, testes implementados, relatórios de cobertura e histórico de commits. |

## Próximos Passos

1. Confirmar a distribuição das seis funções entre os integrantes.
2. Para cada função, transformar os casos sugeridos em uma tabela de teste com: identificador, técnica, entrada, saída esperada, classe de equivalência ou valor limite, branch esperado e observação.
3. Rodar os testes existentes relacionados às funções escolhidas para entender a base atual.
4. Complementar os testes que ainda não cobrem os casos planejados, mantendo os arquivos de teste já existentes quando possível.
5. Rodar cobertura para os arquivos/funções selecionados.
6. Comparar o planejamento com a cobertura real e registrar lacunas encontradas.
7. Escolher uma melhoria pequena para TDD.
8. Executar o ciclo TDD com evidências:
   - Red: teste criado e falhando;
   - Green: implementação mínima passando;
   - Refactor: melhoria mantendo os testes aprovados.
9. Escrever a análise crítica da equipe.
10. Consolidar o relatório técnico e as instruções de execução no repositório.

## Melhor Caminho Recomendado

O caminho mais seguro é tratar o trabalho em três frentes pequenas: planejamento, implementação dos testes e documentação das evidências. Isso evita que a equipe comece implementando testes sem conseguir explicar depois qual técnica foi aplicada.

### 1. Planejamento dos Testes

Antes de alterar código, cada integrante deve criar uma tabela detalhada para sua função. Um modelo simples:

| ID | Função | Técnica | Entrada | Saída esperada | Classe/limite/branch | Justificativa |
| --- | --- | --- | --- | --- | --- | --- |
| TC-01 | `parseTimeString` | Valor limite | `"23:59"`, `24` | `Date` com 23h59 | Limite superior válido | Garante aceitação do maior horário válido no formato 24h. |
| TC-02 | `parseTimeString` | Valor limite | `"24:00"`, `24` | `null` | Primeiro valor inválido após o limite | Garante rejeição fora do intervalo permitido. |

Essa tabela deve ser feita para todas as seis funções. Ela será a base da rastreabilidade entre funcionalidade e testes implementados.

### 2. Implementação dos Testes

Priorizar testes unitários puros, sem banco de dados, rede ou UI. A ordem recomendada é:

1. `getPrefetchMonthCount`
2. `parseTimeString`
3. `getAvailabilityFromSchedule`
4. `intersect`
5. `subtract`
6. `computeEffectiveStateAcrossTeams`

Essa ordem começa pelas funções mais diretas e deixa a tabela de decisão mais complexa para depois, quando a equipe já tiver padronizado a forma dos testes.

Quando existirem testes prontos no projeto, a equipe deve avaliar se eles já cobrem os casos planejados. Se não cobrirem, o melhor caminho é complementar os testes existentes em vez de criar arquivos paralelos sem necessidade.

### 3. Evidências de Cobertura

Para a atividade, não basta dizer que os testes passaram. É importante guardar evidências como:

- saída do comando de testes;
- relatório de cobertura;
- tabela comparando branches planejados e branches cobertos;
- prints ou trechos do relatório, se necessário para a entrega;
- commits mostrando a evolução dos testes.

Comandos úteis no projeto:

```bash
TZ=UTC yarn test
yarn type-check:ci --force
yarn biome check --write .
```

Se a equipe for rodar apenas testes específicos, usar o comando de teste do workspace apontando para os arquivos alterados. Depois, antes da entrega final, rodar pelo menos os testes relevantes e o type check.

### 4. Sugestão Para a Parte de TDD

A parte de TDD deve ser pequena e isolada. Evitar funcionalidades grandes, schema de banco ou mudanças que afetem muitos pacotes.

Bons candidatos:

| Opção | Ideia | Por que é boa para TDD |
| --- | --- | --- |
| `parseTimeString` | Aceitar variações simples de entrada, como espaços ao redor de um horário válido. | É uma mudança pequena, fácil de testar e com comportamento claro. |
| `getAvailabilityFromSchedule` | Garantir comportamento explícito para dias vazios entre dias com disponibilidade. | Mantém a função pura e fácil de validar. |
| `subtract` | Adicionar ou documentar comportamento para exclusões adjacentes ao intervalo base. | Trabalha com limites e tem baixo acoplamento. |

O fluxo recomendado de commits para evidenciar TDD:

```text
test: add failing test for selected behavior
fix: implement selected behavior
refactor: simplify selected behavior
```

No relatório, registrar para cada etapa:

| Etapa | Evidência esperada |
| --- | --- |
| Red | Teste novo falhando, com mensagem de erro ou print/saída do terminal. |
| Green | Implementação mínima e teste passando. |
| Refactor | Pequena melhoria no código, mantendo todos os testes passando. |

### 5. Estrutura Recomendada do Relatório

O relatório pode seguir exatamente a estrutura pedida no enunciado:

1. Introdução
2. Descrição do projeto Cal.com
3. Planejamento dos testes
4. Descrição das técnicas utilizadas
5. Testes desenvolvidos
6. Métricas de cobertura
7. Processo de TDD
8. Análise crítica
9. Conclusão

Na seção de testes desenvolvidos, usar uma subseção por função. Em cada subseção, incluir:

- objetivo da função;
- casos caixa-preta;
- casos caixa-branca;
- tabela MC/DC quando aplicável;
- cobertura obtida;
- lacunas ou limitações.

### 6. Rastreabilidade

Para facilitar a correção, cada teste deve apontar para a função e para a técnica aplicada. A equipe pode usar identificadores como:

| ID | Função | Técnica | Teste implementado |
| --- | --- | --- | --- |
| GPMC-01 | `getPrefetchMonthCount` | Particionamento de equivalência | `returns 2 for column view with different months` |
| GPMC-02 | `getPrefetchMonthCount` | MC/DC | `returns undefined when prefetchNextMonth is true in month view` |
| PTS-01 | `parseTimeString` | Valor limite | `returns a date for 23:59 in 24-hour format` |
| SUB-01 | `subtract` | Particionamento por sobreposição | `splits a source range when excluded range is in the middle` |

Essa rastreabilidade deve aparecer no relatório ou em um documento auxiliar.

## Checklist Final Antes da Entrega

- [ ] Seis funções confirmadas, uma por integrante.
- [ ] Tabelas de casos de teste completas.
- [ ] Testes caixa-preta implementados.
- [ ] Testes caixa-branca implementados.
- [ ] MC/DC documentado para pelo menos uma decisão complexa.
- [ ] Cobertura coletada e registrada.
- [ ] Parte de TDD realizada com evidências Red, Green e Refactor.
- [ ] Análise crítica escrita.
- [ ] Instruções de execução adicionadas ao repositório.
- [ ] Relatório técnico finalizado.
- [ ] Commits organizados e com mensagens claras.
