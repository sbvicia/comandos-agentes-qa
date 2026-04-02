# documenta-testes

## Disparo (formato esperado)

Exemplos válidos:

- `documenta testes da demanda NS-988`
- `documenta testes/ NS-721 correção Excel`
- URL Jira: `https://roboteasy.atlassian.net/browse/NS-959`

O agente deve **extrair a chave Jira** com o padrão `\b[A-Z]{1,10}-\d+\b` (ex.: `NS-988`, `ns-988` → maiúsculas).

### Gate obrigatório (travar se falhar)

- Se **não** existir **nenhuma** chave Jira no texto do disparo (nem URL com `/browse/PROJ-123`), **não** gravar no Jira.
- **Responder apenas** pedindo o disparo correto.

---

## Objetivo

1. **Validar** a tarefa Jira (`jira_issues` `get`).
2. **Cruzar** com **`rpa-agent`** e **`front-studio`** no workspace, quando existirem.
3. Produzir **dois níveis em conjunto** (obrigatório os dois) — **registo principal no Jira** (qualquer pessoa com acesso à issue vê o plano **sem** depender de login noutra ferramenta):

   - **Cenário de teste** — agrupador temático: identificador (ex. `S1`, `S2`), **prioridade** (Crítico → Baixo), **nome** e **1–2 frases** do que se valida (âmbito / risco).
   - **Casos de teste** — sob cada cenário, **pelo menos um** caso **executável**, com: **ID** (ex. `CT-S1-01`), **Pré-condições**, **Passos** (numerados), **Resultado esperado**, **Dados** (se aplicável), **Área** (ex.: front-studio, rpa-agent, integração).

4. **Gravar o plano completo** no campo Jira **Implementação → Testes realizados** (via `jira_issues` **`update`** e `customFields`, valor em **ADF**). **Não** publicar o corpo do plano em **comentários** da issue.

5. **Kiwi TCMS (obrigatório):** criar no tenant, na mesma execução, o **plano de testes** e os **casos** correspondentes ao plano do Jira, via MCP **`user-kiwi-tcms`** (consultar descritores das tools e schema antes de invocar). O tenant exige utilizador autenticado: se existir tool de autenticação no servidor, **chamar primeiro**; se a sessão for inválida ou a API falhar, **não** dar o comando como concluído — responder com o erro, pedir correção de credenciais/sessão e **repetir** o Kiwi após o utilizador resolver. Incluir no Jira **sempre** um parágrafo com `Plano de testes (Kiwi):` + URL do plano — **sem** retirar o detalhe executável do Jira. **Ordem fixa no bloco que se anexa:** o parágrafo `Plano de testes (Kiwi):` + link deve ser o **primeiro** nó desse bloco (logo após separador mínimo face ao conteúdo já existente, se houver); **nunca** colocar o link Kiwi apenas no fim do plano.

**Conteúdo já preenchido em Testes realizados:** obrigatório **ler** no **`get`** o valor atual; o `update` envia **existente + separação mínima + novo bloco** (regra de **não sobrescrever**), salvo pedido explícito do utilizador nessa mensagem para substituir tudo. O **novo bloco** começa por `Plano de testes (Kiwi):` + URL e só depois segue cabeçalho *documenta-testes*, contexto, cenários e casos.

**Não** usar Confluence. **Não** usar sub-tarefas. **Não** alterar **Critérios de Aceitação**, **descrição** nem outros campos **salvo** o campo **Testes realizados** (e pedido explícito noutra mensagem para outros campos).

---

## Campo Jira «Testes realizados» (Implementação)

| Label na UI (Roboteasy) | Onde aparece | API |
|-------------------------|--------------|-----|
| **Testes realizados** | Informações principais → **Implementação** → *Testes Realizados* | `customFields.customfield_*` |

**Resolver o ID do `customfield_*`:**

1. Na resposta do `jira_issues` **`get`**, se existir mapeamento **`names`**, localizar **«Testes realizados»** / **«Testes Realizados»**.
2. **Fallback** (projeto **NS**): **`customfield_10831`**.

Enviar **ADF** (`{ "type": "doc", "version": 1, "content": [ … ] }`). Para **acumular**: concatenar no array `content` do documento existente um separador (ex. parágrafo com espaço) e os novos nós, **nessa ordem**: primeiro o parágrafo Kiwi (link), depois o resto do plano da execução atual.

---

## Limite de volume

- **Máximo 20 casos de teste** no total (soma de todos os `CT-...`).
- **Cenários:** o necessário para cobrir a demanda com qualidade.

---

## Formato do conteúdo (campo Testes realizados no Jira)

Dentro de cada **execução** do comando, a ordem dos nós do **bloco novo** (ou do documento inteiro se o campo estiver vazio) é:

1. **Primeiro (obrigatório):** parágrafo `Plano de testes (Kiwi):` + link (ADF com `link` no URL) ao plano criado no Kiwi nessa execução.
2. Cabeçalho: *«Plano de testes (documenta-testes)»* + data ou build se o utilizador indicou.
3. Secção **Contexto**: resumo da issue + nota se **rpa-agent/front-studio** não estão no workspace.
4. Para **cada cenário**: título (`S1 — Crítico — …`), descrição curta, depois os **casos**.
5. Cada **caso de teste**: linha com **`[ ]`** + ID + título; **Pré**, **Passos** numerados, **Esperado**, **Área** (parágrafos ADF ou lista simples; evitar estruturas ADF muito aninhadas se a API falhar).

```text
Plano de testes (Kiwi): https://…/plan/…/

Plano de testes (documenta-testes) — [data]

Contexto: …

S1 — Crítico — [nome do cenário]
Breve descrição do cenário (âmbito).

[ ] CT-S1-01 — [título curto]
Pré: …
Passos: 1. … 2. …
Esperado: …
Área: …
```

**Limite de tamanho:** se existente + novo exceder a API, **truncar** o bloco **novo** (nunca cortar o existente sem pedido explícito) ou avisar.

---

## Uso obrigatório de MCP

- **Jira:** `jira_issues` (`get`, **`update`** com `customFields` para **Testes realizados** — plano **completo** em ADF; no bloco novo, **primeiro** o parágrafo Kiwi com link, depois o resto).
- **Kiwi:** `user-kiwi-tcms` em **toda** execução deste comando — criar/atualizar plano e casos alinhados ao plano do Jira (respeitar limite de 20 casos no Jira; replicar estrutura no Kiwi de forma coerente).
- **Não** usar `jira_comments` **`add`** para o plano (comentários só para falhas técnicas ou avisos, se necessário).

---

## Resposta ao utilizador

- Chave da issue validada.
- Confirmação de **`update`** no campo **Testes realizados** (`customfield_*`).
- Contagem: **N cenários**, **M casos de teste** (M ≤ 20).
- Prioridade máxima coberta (cenário).
- Confirmação Kiwi: **URL do plano** no TCMS e referência no Jira (`Plano de testes (Kiwi):` …) no **início** do bloco de plano anexado. Se o Kiwi estiver bloqueado por autenticação ou erro de API, indicar o bloqueio e o que falta fazer — **não** apresentar o fluxo como concluído sem plano no Kiwi.

---

## O que não fazer

- **Não** gravar **apenas** um link no Jira em detrimento do plano executável (o Kiwi não substitui o Jira para equipas sem acesso ao tenant).
- **Não** colocar o parágrafo `Plano de testes (Kiwi):` + link **só no fim** do bloco novo; deve ser **sempre o primeiro** conteúdo desse bloco (após separador do existente, se aplicável).
- **Não** concluir o comando **sem** plano criado no Kiwi (salvo falha técnica documentada após tentativa com MCP e orientação ao utilizador).
- **Não** escrever **só** cenários sem **casos de teste** concretos sob cada um.
- **Não** escrever **só** casos soltos sem **cenários** que os agrupem.
- **Não** gravar o plano em **comentários** Jira.
- **Não** gravar em **Critérios de Aceitação** (nem `customfield_10829` nem equivalente).
- **Não** ultrapassar **20 casos de teste** no total.
- **Não** criar sub-tarefas nem usar Confluence.
