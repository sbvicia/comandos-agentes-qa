# analyze-flow

## Disparo

- O utilizador invoca o comando e **anexa ou indica o alvo**: orquestração principal, ficheiros de fluxo, diagrama, documentação, ou descrição + trechos de código.
- Se faltar contexto para mapear o fluxo (só pedido verbal sem artefactos), **pedir o mínimo necessário** (entrada do processo, ficheiro de orquestração, passos críticos) **ou** analisar só o que foi dado e **indicar limitações** na resposta — alinhado ao skill `rpa-flow`.

---

## Papel: skill **rpa-flow**

Executar a análise **conforme o skill `rpa-flow`**: mapear o fluxo **end-to-end**, avaliar **resiliência**, **ordem**, **dependências externas**, **pontos de falha** e **quebra de fluxo** (estado inconsistente, reentrada, idempotência, compensação).

### Obrigatório antes de concluir

**Ler** o skill completo: ficheiro típico  
`C:\Users\Roboteasy\.cursor\skills\rpa-flow\SKILL.md`  
(ou `~/.cursor/skills/rpa-flow/SKILL.md` conforme o ambiente).

Se o projeto tiver cópia local deste skill, usar a do projeto.

---

## Objetivo

Validar se o **fluxo do robô é resiliente**: a sequência e as integrações permitem recuperar ou falhar de forma controlada, ou expõem o processo a paragens, reprocessamento incorreto ou estado «a meio» sem rastreio claro.

## Foco (alinhado ao workflow do skill)

1. **Ordem das etapas** — pré-condições entre passos N e N+1; passos transacionais mal colocados; happy path vs ramos de exceção com estado conhecido ao sair.
2. **Dependência externa** — sistemas, UI, API, BD, ficheiros, filas, credenciais, rede, horários de janela, versões; o que acontece se falharem ou ficarem lentos.
3. **Pontos de falha** — UI (timing, seletor), rede/API, IO, modais inesperados, rate limits; **sintoma em produção** e gravidade.
4. **Possibilidade de quebra de fluxo** — `catch` que segue em frente sem estado válido, flags desincronizadas, dupla reentrada, falta de idempotência, «meio processado» sem compensação; reinício a meio do lote.

**Regras do skill a respeitar internamente:**

- Distinguir **falha transitória** (retry, backoff) de **falha de negócio** (rota alternativa, notificação, DLQ).
- **Não ser genérico**: exemplos concretos (`Ficheiro.cs`, método, passo nomeado no fluxo).
- Onde a análise for por código, **citar trechos ou ficheiros** como no code review RPA.

### Processo interno (síntese — não copiar como secções finais ao utilizador)

Seguir o workflow do skill: (1) reconstruir fluxo na ordem real; (2) validar ordem; (3) marcar pontos de falha; (4) inventariar dependências externas; (5) cenários de quebra; (6) medidas de resiliência. Usar a **checklist interna** do skill (paralelização vs serialização, observabilidade por etapa, reinício seguro).

---

## Formato obrigatório da resposta ao utilizador

Responder **apenas em português**. A saída visível deve conter **exatamente** as três secções abaixo, **nesta ordem** (com os títulos indicados).

**Nota:** Podes **condensar** no início um **mapa do fluxo** muito curto (lista numerada ou mermaid breve) **só** dentro da secção 🔴 ou como **parágrafo introdutório de uma linha** antes de 🔴, se ajudar a contextualizar — não é obrigatório um bloco separado «Mapa do fluxo» se o utilizador só pediu as três zonas.

### 🔴 Pontos frágeis no fluxo

Aqui entram, com **exemplos concretos** e impacto em produção:

- Riscos de **ordem** (ex.: commit de estado antes de confirmação externa).
- **Pontos de falha** de maior gravidade (UI, API, IO, etc.) e sintoma operacional.
- **Dependências externas** que são **ponto único de falha** ou sem mitigação.
- Cenários de **quebra de fluxo** (falha após passo X, reinício, concorrência).

Usar bullets ou tabela curta quando clarifique; **evitar lista vaga** sem ligação ao contexto analisado.

### 🟡 Melhorias sugeridas

Ações **priorizadas** (do mais crítico ao desejável), alinhadas à secção de **resiliência** do skill:

- Cada item: **ação** + **benefício em produção** (menos fila presa, menos reprocessamento manual, menos estado inconsistente).
- Incluir **retry/backoff**, **checkpoint**, **validação de estado antes de avançar**, **timeout explícito**, **compensação**, **circuit breaker**, **DLQ**, **feature flag**, quando aplicável.
- Distinguir mitigação para falha **transitória** vs **de negócio**.

### 🟢 Fluxos bem estruturados

Destacar o que está **correto** ou **robusto** na sequência, nas decisões ou nas integrações (ex.: ordem lógica entre passos, separação clara de etapas, idempotência onde existe, bons pontos de log por etapa).

Se pouco houver a destacar, ser **honesto** e breve — não inventar elogios.

---

## O que não fazer

- Não substituir as três secções 🔴🟡🟢 por **apenas** a estrutura numerada 1–6 do skill como saída principal; o skill guia o **raciocínio**; o **produto** para o utilizador são as três secções acima.
- Não confundir análise de fluxo com revisão linha-a-linha só de estilo; o foco é **sequência, dependências, falhas e resiliência**.
