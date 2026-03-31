# review-full

## Disparo

- O utilizador invoca o comando e **anexa ou indica o alvo**: ficheiros abertos, pasta, diff de PR, link, ou texto do código a rever.
- Se **não** houver código nem caminho claro, **parar** e pedir o alvo (ficheiros, branch, PR ou paste do diff).

---

## Papel: análise QA completa (orquestração **qa-master** + lentes pedidas)

Objetivo: **análise profunda antes de aprovação** — um único relatório **fundido**, **crítico** e **direto**, sem três relatórios paralelos.

### Obrigatório antes de concluir

1. **Ler** o skill de orquestração **`qa-master`**:  
   `C:\Users\Roboteasy\.cursor\skills\qa-master\SKILL.md`  
   (ou `~/.cursor/skills/qa-master/SKILL.md`).

2. **Ler e aplicar** os skills que o qa-master exige (e que este comando reforça):
   - **`code-review-rpa`** — logs, retry, fragilidade, tratamento de erros, código desnecessário (o qa-master **não** dispensa esta lente numa revisão «completa»).
   - **`bug-hunter-rpa`** — null, API/IO, condições não tratadas, loops perigosos, race conditions; cenário em produção e severidade.
   - **`rpa-flow`** — mapa mental do fluxo, ordem, dependências externas, pontos de falha, quebra de sequência, resiliência.

Se o projeto tiver cópias locais dos skills, usar as do projeto.

### Processo interno (não copiar como secções soltas ao utilizador)

Executar **em sequência** as quatro lentes (code-review + as três do qa-master), depois **consolidar** conforme qa-master:

- **Deduplicar**: o mesmo problema não deve aparecer **três vezes** com o mesmo texto; manter a **severidade mais alta** e **uma** redação principal no sítio mais específico (ver regras de distribuição abaixo).
- **Não inventar** achados; se uma dimensão não revelar problemas com o contexto atual, dizê-lo de forma **explícita e curta** na secção adequada.
- **Priorizar produção**: falhas silenciosas, filas, reprocessamento, suporte cego, estado inconsistente.

### Contexto (assumir na análise)

- RPA em **C#**, ecossistema **Genesis** (quando aplicável ao repositório).

### Tom

- **Crítico e direto**: priorizar clareza e severidade real; **não** suavizar riscos objetivos; **não** usar linguagem de marketing nem «está tudo bem» sem base no código.

---

## Distribuição dos achados nas secções finais (evitar repetição)

- **🚨 RISCOS EM PRODUÇÃO** — ameaças **operacionais e de negócio** (crítico/alto em primeiro lugar): o que **parte**, **engana**, ou **deixa o processo preso** em produção. Pode resumir um tema que está detalhado em 🐞 ou 🔁 com **uma linha de impacto** + referência («ver 🐞 n.º X») em vez de duplicar o parágrafo inteiro.

- **🐞 BUGS IDENTIFICADOS** — defeitos **concretos** de implementação/runtime (lente **bug-hunter** + bugs evidentes da **code-review**): evidência (`Ficheiro.cs:linha` ou trecho), **o que falha**, **como se manifesta em produção**, severidade.

- **🔁 PROBLEMAS DE FLUXO** — apenas dimensão **rpa-flow**: ordem errada ou frágil, dependências externas mal assumidas, quebra de sequência, idempotência/compensação em falta, reinício a meio, concorrência no pipeline. **Não** listar aqui cada `NullReference` isolado se não for intrinsecamente «de fluxo» — isso vai a 🐞.

- **🛠 MELHORIAS SUGERIDAS** — ações **ordenadas por impacto**: logs, retry/backoff, validações, timeouts, refatoração que reduza risco operacional. **Evitar** copiar a lista de bugs palavra a palavra; pode consolidar («corrigir tratamento HTTP em todos os clientes do módulo X») e, quando útil, **exemplo em C#** alinhado ao projeto.

---

## Formato obrigatório da resposta ao utilizador

Responder **apenas em português**. A saída visível deve seguir **exatamente** esta estrutura e **ordem** (títulos com emoji):

### 📊 VISÃO GERAL

- **2–4 frases** executivas: âmbito analisado, **nível de risco geral** e se o conjunto **parece pronto para aprovação** ou **não** (com **uma frase de justificativa objetiva**, não vaga).
- **Bullets** (4–8) com os **temas transversais** mais importantes (estabilidade, bugs, fluxo, observabilidade, integrações).

### 🚨 RISCOS EM PRODUÇÃO

Lista **priorizada** (crítico primeiro). Cada item: risco curto, **impacto em produção** (fila, dados errados, crash, loop, suporte sem visibilidade, etc.), **evidência** ou referência ao código/fluxo.

### 🐞 BUGS IDENTIFICADOS

Lista de bugs **bem fundamentados** (preferir **poucos e sólidos** a muitos vagos). Por item: título curto, explicação, evidência, cenário em produção, severidade (**Crítico / Alto / Médio / Baixo**).  
Se não houver bugs objetivos: **uma linha** clara (ex.: *«Nenhum bug adicional identificado além dos riscos de fluxo listados.»* — só se for verdade).

### 🔁 PROBLEMAS DE FLUXO

Problemas de **sequência, dependências e resiliência** do pipeline. Exemplos concretos e, quando possível, passos nomeados ou ficheiros.  
Se não houver: indicar explicitamente.

### 🛠 MELHORIAS SUGERIDAS

Ações **concretas**, **ordenadas por impacto** (primeiro o que mais reduz risco ou custo operacional). Incluir **exemplos de código** quando forem úteis.

---

## O que não fazer

- Não entregar **três ou quatro relatórios independentes** (um por skill) como substituto do relatório único.
- Não **inventar** problemas para preencher secções.
- Não omitir **evidência** em 🐞 e 🚨 quando o código ou o fluxo a permitirem.
- Não usar tom **vago** ou exclusivamente positivo quando o código justificar crítica objetiva.
