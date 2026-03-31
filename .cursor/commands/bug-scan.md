# bug-scan

## Disparo

- O utilizador invoca o comando e **anexa ou indica o alvo**: ficheiros abertos, pasta, diff de PR, link, ou texto do código a analisar.
- Se **não** houver código nem caminho claro, **parar** e pedir o alvo (ficheiros, branch, PR ou paste do diff).

---

## Papel: skill **bug-hunter-rpa**

Executar a análise **conforme o skill `bug-hunter-rpa`**: foco em **bugs prováveis** e falhas de **runtime**, não em estilo ou refactor cosmético.

### Obrigatório antes de concluir

**Ler** o skill completo (regras e formato internos): ficheiro típico  
`C:\Users\Roboteasy\.cursor\skills\bug-hunter-rpa\SKILL.md`  
(ou `~/.cursor/skills/bug-hunter-rpa/SKILL.md` conforme o ambiente).

Se o projeto tiver cópia local deste skill, usar a do projeto.

---

## Objetivo

Encontrar **bugs que podem ocorrer em produção** em automação **RPA / C#** e integrações (HTTP, filas, UI, ficheiros).

## Foco de caça (cinco eixos — alinhados ao skill)

1. **NullReferenceException** (e acessos nulos em geral) — `?.` ausente, `FirstOrDefault`, deserialização, dicionários, retornos de API/UI após erro parcial ou timeout.
2. **Falha de API / IO externo** — HTTP sem validação de status/corpo, timeout, cancelamento; JSON assumido válido; em RPA, assumir elemento ou app sempre disponíveis.
3. **Condições não tratadas** — ramos incompletos, `default` vazio, enums não mapeados, estados de UI inesperados, listas vazias tratadas como sucesso, dados sujos / permissões / locale.
4. **Loops perigosos** — `while`/`for` sem saída clara, espera ativa sem limite, retry sem teto ou sem backoff, `WaitFor` sem timeout máximo, risco de loop infinito ou saturação.
5. **Concorrência (race condition)** — estado partilhado sem sincronização, ordem de execução assumida, UI antes do fim da operação, ficheiro/BD em uso concorrente, mistura perigosa de async/contexto.

**Regra do skill:** se num eixo **não** houver achado com o contexto atual, indicar explicitamente *«Nenhum identificado com o contexto atual.»* para esse eixo — **não inventar** problemas.

**Riscos condicionais:** quando o código não for suficiente para afirmar bug, classificar como **risco condicional** e indicar que condição externa o confirmaria (teste, log, métrica).

---

## Formato obrigatório da resposta (por cada problema)

Responder **apenas em português**.

Para **cada** bug ou risco real, usar **esta estrutura** (ordem fixa):

1. **Título curto** — tipo de bug + local (ex.: «NullReference após `GetElement` — `ProcessarPedido.cs`»).
2. **O bug (explicação)** — o que está errado no código e porquê é um defeito lógico ou de contrato (null, ramo não coberto, IO sem verificação, etc.).
3. **Evidência** — trecho citado ou `Ficheiro.cs:linha` / padrão observado (obrigatório quando houver código).
4. **Como aconteceria em produção** — narrativa **concreta**: quem/o quê, ordem, dados ou carga, que leva a crash, loop, dados errados, fila bloqueada, item duplicado, etc. (obrigatório; alinhado ao skill).
5. **Como corrigir** — passos ou **exemplo de código C#** quando fizer sentido (guardas, `null`-checks, política de retry, timeout, `lock`/semáforo, validação de resposta HTTP, etc.).
6. **Severidade** — **Crítico / Alto / Médio / Baixo**, com **uma linha** que justifique em termos de **impacto operacional**.

### Opcional no fim da resposta

- **Resumo** em 2–3 bullets apenas dos **maiores riscos** (o skill permite este fecho).

---

## O que não fazer

- Não substituir esta análise por um code review geral (logs/retry como tema principal): o foco são os **cinco eixos** e o **cenário em produção**.
- Não preferir listas longas e vagas a **poucos achados bem fundamentados**.
- Não omitir a secção **«Como corrigir»** quando houver achado objetivo.
