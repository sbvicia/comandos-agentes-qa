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
3. Produzir **dois níveis em conjunto** (obrigatório os dois):
   - **Cenário de teste** — agrupador temático: identificador (ex. `S1`, `S2`), **prioridade** (Crítico → Baixo), **nome** e **1–2 frases** do que se valida (âmbito / risco).  
   - **Casos de teste** — sob cada cenário, **pelo menos um** caso **executável**, com: **ID** (ex. `CT-S1-01`), **Pré-condições**, **Passos** (numerados), **Resultado esperado**, **Dados** (se aplicável), **Área** (ex.: front-studio, rpa-agent, integração).  
4. **Gravar o plano no campo Jira** correspondente à UI **Implementação → Testes realizados** (área «Adicionar texto» do separador Implementação), via `jira_issues` **`update`** e `customFields`. **Não** publicar este conteúdo em **comentários** da issue.  
   - Se o campo **já tiver texto/conteúdo** na resposta do **`get`**: **não apagar nem substituir** — **acrescentar** o plano novo **a seguir ao existente**, com **um espaço** entre o fim do conteúdo antigo e o início do bloco novo (em ADF: equivalente a preservar os nós existentes e inserir separação mínima de um espaço antes dos novos nós; se um único espaço for inválido entre blocos, usar o separador mais leve que a API aceitar mantendo a regra de **não sobrescrever**).

**Não** usar Confluence. **Não** usar sub-tarefas. **Não** alterar **Critérios de Aceitação**, **descrição** nem outros campos **salvo** o campo **Testes realizados** (e pedido explícito noutra mensagem para outros campos).

---

## Campo Jira «Testes realizados» (Implementação)

| Label na UI (Roboteasy) | Onde aparece | API |
|-------------------------|--------------|-----|
| **Testes realizados** | Informações principais → **Implementação** → *Testes Realizados* | `customFields.customfield_*` |

**Resolver o ID do `customfield_*`:**

1. Na resposta do `jira_issues` **`get`**, se existir mapeamento **`names`** (id → nome do campo), localizar a chave cujo nome é **«Testes realizados»** / **«Testes Realizados»** (ou equivalente) e usar essa chave em `update`.  
2. Se **`names`** não vier ou o campo não aparecer: usar a tabela de **fallback** abaixo (atualizar na repo quando a instalação Jira tiver o ID confirmado):

| Variável / nota | Valor |
|-----------------|-------|
| `TESTES_REALIZADOS_FIELD_ID` | **`customfield_10831`** (projeto **NS** / Bug — confirmar se o projeto mudar). *Nota:* o `create_metadata` pode listar o tipo como string, mas a API costuma exigir **ADF** no `update`; se falhar com string, usar documento `{ "type": "doc", "version": 1, "content": [ … ] }`. |

Se, após `get`, o ID continuar **desconhecido** e a tabela estiver vazia: **não** fazer `update` aleatório; responder ao utilizador a pedir o **custom field id** ou a confirmar que o `get` deve incluir `names`/metadados, e entregar o **texto gerado** na resposta para colar manualmente.

**Formato do valor no `update`:** para **Testes realizados** (`customfield_10831` no NS), tratar como campo rico: enviar **ADF** (`{ "type": "doc", "version": 1, "content": [ … ] }`). O `create_metadata` pode marcar o tipo como «string», mas a validação da API pode recusar texto plano («precisa ser um documento da Atlassian»). Para **acumular** com conteúdo existente: no `get`, ler o ADF atual de `customfield_10831`, concatenar no array `content` um separador mínimo (ex. parágrafo com um espaço) e de seguida os nós do novo plano. Seguir o mesmo rigor que em `bug-report-roboteasy` / `bug-report-etapa-desenvolvimento` na secção **ADF nos custom fields**. Estruturas muito aninhadas (ex. `taskList` dentro de blocos longos) podem falhar no MCP — preferir **parágrafos** + texto `[ ] CT-…` se necessário.

---

## Limite de volume

- **Máximo 20 casos de teste** no total (soma de todos os `CT-...`), distribuídos por **cenários**.  
- Número de **cenários**: o necessário para cobrir a demanda com qualidade, tipicamente **menos** casos por cenário crítico e mais detalhe onde o risco é alto.

---

## Formato do conteúdo (campo Testes realizados)

- Cabeçalho: *«Plano de testes (documenta-testes)»* + data ou build se o utilizador indicou.  
- Secção **Contexto**: resumo da issue + nota se **rpa-agent/front-studio** não estão no workspace.  
- Para **cada cenário**: título claro (`S1 — Crítico — …`), depois os **casos** desse cenário.  
- **Lista de tarefas / controlo:** cada **caso de teste** (não o cenário isolado) deve ter **uma linha com checkbox** no início do bloco do caso — em ADF usar nós de *taskList* / *taskItem* se o render do campo o suportar; se estiver a montar texto/markup aceite pelo campo, usar `- [ ]` ou equivalente wiki.  
- **Estrutura sugerida por caso** (espelhar em parágrafos/listas ADF):

```text
S1 — Crítico — [nome do cenário]
Breve descrição do cenário (âmbito).

- [ ] CT-S1-01 — [título curto do caso]
Pré: …
Passos: 1. … 2. …
Esperado: …
Área: …
```

- **Conteúdo já preenchido:** obrigatório **ler** no **`get`** o valor atual de **Testes realizados**; o `update` envia **existente + um espaço + novo plano** (regra fixa deste comando). **Só** substituir tudo se o utilizador pedir explicitamente nessa execução.  
- **Limite de tamanho:** se **existente + novo** exceder o que a API aceita, **truncar** o bloco **novo** (nunca cortar o existente sem pedido explícito) ou avisar o utilizador; **não** apagar conteúdo pré-existente por causa do limite.

---

## Uso obrigatório de MCP

- **Jira:** `jira_issues` (`get`, **`update`** com `customFields` para Testes realizados).  
- **Não** usar `jira_comments` **`add`** para o plano de testes deste comando (comentários só para falhas técnicas ou avisos ao utilizador, se necessário).  
- Servidor típico: `user-jira`.

---

## Resposta ao utilizador

- Chave da issue validada.  
- Confirmação de **`update`** no campo **Testes realizados** (e `customfield_*` usado).  
- Contagem: **N cenários**, **M casos de teste** (M ≤ 20).  
- Prioridade máxima coberta (cenário).

---

## O que não fazer

- **Não** escrever **só** cenários sem **casos de teste** concretos sob cada um.  
- **Não** escrever **só** casos soltos sem **cenários** que os agrupem e contextualizem.  
- **Não** gravar o plano de testes em **comentários** (destino é só o campo **Implementação → Testes realizados**).  
- **Não** gravar em **Critérios de Aceitação** (nem `customfield_10829` nem equivalente).  
- **Não** ultrapassar **20 casos de teste** no total.  
- **Não** criar sub-tarefas nem usar Confluence.  
- **Não** **substituir** o campo **Testes realizados** quando já houver conteúdo (deve **acumular** com **um espaço** antes do bloco novo, salvo pedido explícito do utilizador).
