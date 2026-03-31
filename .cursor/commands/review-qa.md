# review-qa

## Disparo

- O utilizador invoca o comando e **anexa ou indica o alvo**: ficheiros abertos, pasta, diff de PR, link, ou texto do código a rever.
- Se **não** houver código nem caminho claro, **parar** e pedir o alvo (ficheiros, branch, PR ou paste do diff).

---

## Papel: orquestração **qa-master**

Executar a análise **como o skill `qa-master`**: um único relatório fundido, **não** três relatórios separados.

### Obrigatório antes de concluir

1. **Ler** o skill de orquestração: `qa-master` — ficheiro típico: `C:\Users\Roboteasy\.cursor\skills\qa-master\SKILL.md` (ou `~/.cursor/skills/qa-master/SKILL.md` conforme o ambiente).
2. **Ler** os três skills referenciados pelo qa-master (conteúdo necessário para cumprir cada lente):
   - `code-review-rpa` → `.../skills/code-review-rpa/SKILL.md`
   - `bug-hunter-rpa` → `.../skills/bug-hunter-rpa/SKILL.md`
   - `rpa-flow` → `.../skills/rpa-flow/SKILL.md`

Se o projeto tiver cópias locais desses skills, usar os do projeto.

### Processo interno (não copiar como secções ao utilizador)

1. **Lente code-review-rpa** — logs, retry, fluxo frágil, código desnecessário, tratamento de erros.
2. **Lente bug-hunter-rpa** — null reference, condições não tratadas, API/IO, loops, race conditions; cenário em produção e severidade.
3. **Lente rpa-flow** — ordem do fluxo, pontos de falha, dependências externas, quebra de sequência, resiliência.

**Consolidação:** deduplicar achados (uma ocorrência no relatório, severidade mais alta), **não inventar** problemas, priorizar **impacto em produção** (filas, reprocessamento, falhas silenciosas, suporte cego, estado inconsistente).

---

## Contexto (sempre assumir na análise)

- Código de **RPA em C#** no ecossistema **Genesis**.
- Objetivo: automação **estável**, **resiliente** e **mantenível**.

## Foco explícito

- Falhas que podem **quebrar o robô** em runtime ou deixar o processo em estado errado.
- **Ausência ou insuficiência de logs** (rastreio, correlação, erros sem contexto).
- **Falta de retry / backoff / circuito** em chamadas externas, UI, ficheiros, rede.
- Código **frágil** (timeouts fixos inadecuados, seletores frágeis, hardcodes, suposições de ambiente).
- **Tratamento de erro inadequado** (engolir exceções, `catch` vazio, retornos ambíguos, sem compensação onde o negócio exige).

Para cada achado relevante: **exemplo concreto** (trecho, `Ficheiro.cs:linha`, ou passo do fluxo) e **impacto operacional**.

---

## Formato obrigatório da resposta ao utilizador

Responder **apenas em português**. A saída deve seguir **exatamente** esta estrutura e ordem (títulos com emoji):

### 📊 RESUMO GERAL

- **2–4 frases** executivas: o que foi analisado e o nível de risco geral.
- **Bullets** (3–7) com os temas mais importantes (estabilidade, bugs de runtime, robustez do fluxo, observabilidade).

### 🚨 RISCOS CRÍTICOS

Lista **priorizada** (crítico primeiro). Em cada item:

- **Descrição curta** do risco.
- **Impacto em produção** (fila, dados errados, crash, loop, suporte sem visibilidade, reprocessamento, inconsistência de estado, etc.).
- **Evidência**: citação de código, `Ficheiro.cs:linha`, ou passo concreto do fluxo.

**Não** ser genérico: quando o código permitir, mostrar **como** o problema se manifestaria na operação.

### 🛠 SUGESTÕES DE MELHORIA

Ações **concretas**, **ordenadas por impacto** (o que fazer primeiro):

- Endurecimento: logs estruturados, retry com política clara, validações, timeouts, idempotência, compensação, onde aplicável.
- **Sempre que fizer sentido**, incluir **exemplo de código** (C#) ou pseudo-código alinhado ao padrão do projeto.

### 🟢 PONTOS POSITIVOS

Itens objetivos já existentes no código (boas práticas observadas). **Não** repetir o detalhe dos riscos; manter secção equilibrada mas honesta (se pouco a destacar, indicar de forma sucinta).

---

## O que não fazer

- Não entregar três relatórios independentes (um por skill) como resposta principal.
- Não usar como **estrutura principal** o formato interno dos skills individuais (ex.: só «Problemas críticos / Melhorias») em detrimento das quatro secções acima.
- Não omitir exemplos práticos quando o código ou o fluxo o permitir.
