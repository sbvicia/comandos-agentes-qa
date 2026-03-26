# bug-report-etapa-desenvolvimento

> Fluxo para **etapa de desenvolvimento**: igual ao `bug-report-roboteasy` (Jira Bug, campos, sprint AVALIAR, anexos), com **tarefa pai obrigatória** no campo **Pai** da Jira. Disparo: `/bug-report-etapa-desenvolvimento`.

## CONFIGURAÇÃO JIRA (editar uma vez neste arquivo)

| Chave | Valor |
|--------|--------|
| `PROJECT_KEY` | `NS` |
| `SPRINT_AVALIAR_ID` | `1006` — ID API do sprint **AVALIAR** (Futuro, **Produto - 2026**, espaço NS); board Jira Software **`49`** |
| `PRIORIDADE` | **Média** — fixa em todas as issues criadas por este comando |

### Tarefa pai (campo **Pai** na Jira) — obrigatório neste comando

Bugs de **etapa de desenvolvimento** devem **sempre** ficar ligados a uma **tarefa pai** na Jira (campo **Pai**, como «Selecionar pai» — ex. **NS-953**, **NS-666**). **Não** criar Bug filho sem essa ligação confirmada.

**Formatos aceites** (case-insensitive no rótulo; a chave Jira normaliza-se em **MAIÚSCULAS**, ex. `NS-666`):

- `tarefa pai: NS-666`
- `Tarefa pai: ns-666`
- `pai: NS-666`
- `Pai NS-666`
- `parent: NS-666` (alternativa em inglês)

Pode estar na **primeira linha** ou em qualquer linha isolada; o assistente extrai com regex do tipo: `(?:tarefa\s*pai|pai|parent)\s*:?\s*(NS-\d+)` (ajustar ao padrão de chave do projeto **NS**).

**Variável:** `{{TAREFA_PAI}}` = chave normalizada, ex. `NS-666`.

**Regras (ordem rigorosa):**

1. **Sem chave na mensagem:** **interromper** o comando. Responder só com aviso claro e pedido explícito: *«Indique a tarefa pai no formato `tarefa pai: NS-XXX` e reenvie o bug.»* — **não** executar Fase A, **não** executar Fase B, **não** criar issue.
2. **Com chave + MCP Jira disponível:** chamar **obrigatoriamente** `jira_issues` com `action`: **`get`** e `issueKey`: `{{TAREFA_PAI}}` **antes** de qualquer outra etapa (antes da Fase A e antes do `create`).
   - Se o **`get` falhar** (issue inexistente, sem permissão, chave inválida, erro da API): **interromper** o comando. **Sinalizar** em destaque (ex.: título **«Tarefa pai não encontrada»** ou **«Não foi possível validar a tarefa pai»**), indicar a chave que foi tentada (`{{TAREFA_PAI}}`) e o motivo em linguagem simples. **Pedir de novo** a chave: *«Confirme a chave da tarefa pai no Jira e reenvie com `tarefa pai: NS-XXX` + descrição do bug.»* — **não** gerar bug report completo, **não** chamar `create_metadata`, **não** criar issue, **não** enviar anexos.
3. **Com chave + MCP Jira indisponível:** **não** criar issue no Jira (não é possível validar o pai). **Interromper** a parte Jira; explicar que sem MCP não há validação da tarefa pai; pedir reexecutar o comando quando o MCP estiver ativo **ou** enviar nova mensagem só com `tarefa pai: NS-XXX` corrigida após ativar MCP. Opcional: entregar só uma linha de lembrete — **não** simular sucesso na Jira.
4. **Só após `get` com sucesso:** prosseguir para Fase A (markdown) e Fase B (`create` com **sempre** `"parent": { "key": "{{TAREFA_PAI}}" }` em `customFields`, junto com sprint e demais campos).
5. No `jira_issues` `create`, incluir **`"parent": { "key": "{{TAREFA_PAI}}" }`** em `customFields`. **Proibido** criar o Bug **propositadamente** sem `parent` para “corrigir depois”. Se o `create` falhar **só** no campo `parent`, pode tentar **uma** vez `jira_issues` `update` com o mesmo `parent`; se continuar a falhar, **sinalizar**, **não** insistir em atalhos que deixem o bug sem pai, e orientar o utilizador (tipo de issue pai / regras do projeto).
6. Ao gerar o **{{TEXTO_BRUTO}}** para as secções `###`, **remover** as linhas que só declaram a tarefa pai.

### Mapeamento `###` (markdown) → campos na tarefa Jira (Bug / NS)

O relatório gerado na **Fase A** usa títulos `### Título`. **Após** gerar o markdown, **antes** do `jira_issues` `create`, **separar o corpo por tema**: cada bloco começa na linha `### NomeDoTema` e termina imediatamente antes do próximo `### ` (ou fim do texto). O **conteúdo** enviado a cada campo Jira é **só o texto desse tema** — **sem** repetir a linha do título `### …` no campo.

| Título exato no markdown (`### …`) | Campo na Jira (UI) | Onde enviar na API / MCP |
|-----------------------------------|--------------------|---------------------------|
| `### Descrição` | Descrição | parâmetro `description` (string; o servidor MCP converte para ADF no campo principal) |
| `### Passos para Reproduzir` | Passos para Reproduzir | `customFields.customfield_10827` |
| `### Comportamento Atual` | Comportamento Atual | `customFields.customfield_10828` |
| `### Comportamento Esperado` | Comportamento Esperado | `customFields.customfield_10134` |
| `### Critérios de Aceitação` | Critérios de Aceitação | `customFields.customfield_10829` |

**Regras de parsing:**

- Respeitar **exatamente** os títulos acima (incluindo acentos e maiúsculas como no modelo).
- Fazer **trim** do corpo de cada secção; não incluir linhas em branco excessivas só por causa do separador.
- O **Resumo** (`summary`) da issue continua a ser **uma linha** derivada da secção **Descrição** (primeira frase ou síntese), não o título `###`.

**ADF nos custom fields (obrigatório):** na API Jira, os quatro campos da tabela (exceto a Descrição principal via parâmetro `description`) **não** aceitam string simples — exigem **Atlassian Document Format (ADF)**. Em `customFields`, o valor de cada `customfield_*` deve ser um **objeto ADF** (`{ "type": "doc", "version": 1, "content": [ … ] }`), não texto cru.

- Converter o **markdown do corpo** da secção para ADF de forma conservadora: parágrafos (`paragraph` + `text`), quebras de linha com `hardBreak` quando fizer sentido, listas numeradas com `orderedList` / `listItem`, bullets com `bulletList` / `listItem`.
- Se a conversão falhar ou a API devolver erro de validação noutro formato, **fallback:** colocar o relatório completo só em `description` (string) e avisar o utilizador para copiar manualmente para os blocos da Jira — mas **tentar primeiro** o mapeamento + ADF.

Se `create_metadata` indicar **outros** ids para estes rótulos (mudança de projeto), preferir os ids devolvidos pelo metadata em vez da tabela fixa acima.

### Imagens / prints → anexos na Jira

O utilizador pode **anexar imagens ao chat** na mesma mensagem que dispara `/bug-report-etapa-desenvolvimento` (colar print, arrastar ficheiro, etc.). O assistente deve:

1. **Identificar** todos os ficheiros de imagem disponíveis para esta mensagem: caminhos absolutos indicados pelo ambiente (ex.: ficheiros guardados em `workspaceStorage`, `assets`, etc.), ou caminhos que o próprio utilizador tenha colado no texto.
2. **Só depois** da issue existir: usar o MCP **`user-jira`**, ferramenta **`jira_attachments`**, com:
   - `action`: **`upload`**
   - `issueKey`: chave devolvida pelo `jira_issues` `create` (ex.: `NS-123`)
   - `filePath`: **caminho absoluto** no disco da máquina onde o Cursor corre (Windows: ex. `C:\Users\...\image.png`). O servidor MCP lê o ficheiro localmente.
   - `fileName` (opcional): nome legível na Jira (ex. `erro-tela-inicial.png`, `passo-2.png`); se omitir, usa o nome do ficheiro.
3. **Repetir** um `upload` por imagem, na ordem que fizer sentido (ou ordem em que apareceram).
4. Se **não** existir caminho de ficheiro acessível (só pré-visualização inline sem path): informar o utilizador que o upload MCP exige ficheiro no disco — pedir para **guardar** a imagem numa pasta (ex. área de trabalho), **anexar de novo** ao chat ou colar o **caminho completo** do ficheiro na mensagem, e então repetir o upload (ou `update` + anexos numa segunda mensagem com a chave da issue).
5. Formatos típicos: `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`.
6. Opcional na **Descrição** (texto): se houver imagens anexadas com sucesso, pode acrescentar uma frase curta no corpo da secção **Descrição** (antes do create) a referir que existem evidências em **Anexos** na issue — **não** substituir o upload; anexos são sempre via `jira_attachments` `upload`.

**Prioridade (regra fixa):** toda tarefa criada por este fluxo deve ter **Prioridade = Média** na Jira. Na chamada MCP `jira_issues` (`action`: `create`, e `update` se corrigir campos), passar sempre `priority`: **`Medium`** (nome em inglês esperado pela API; na UI costuma aparecer como **Média**). **Não** usar Highest, High, Low ou Lowest, mesmo que o texto bruto sugira outra prioridade — salvo se o utilizador **nesta mesma conversa** pedir explicitamente outro valor antes de criar a issue.

### Sprint na UI (referência fixa — sempre esta opção)

Em **todas** as execuções deste comando, o campo **Sprint** (Informações do ticket) deve corresponder à opção da lista que na interface é:

- Secção **Futuro**
- Nome do sprint: **AVALIAR** (contexto **Produto - 2026**)
- Com filtro do tipo **«Mostrar apenas sprints nesse espaço (NS)»** ativo, é a entrada **AVALIAR** desse espaço — **não** outro sprint, **não** deixar «Selecionar Sprint» vazio.

**Regra para o assistente:** em **cada** `jira_issues` `create` (e `update` se corrigir sprint), incluir **sempre** o sprint que na UI é **AVALIAR** (secção **Futuro**, mesmo nome que no dropdown com «Mostrar apenas sprints nesse espaço (NS)»). **Nunca** omitir de propósito; **nunca** usar **022026**, **032026**, **IMPORTANTE** ou outro sprint salvo instrução explícita do utilizador **nesta conversa**.

**Nota técnica:** a API não aceita o texto «AVALIAR» no campo Sprint — usa o ID **`SPRINT_AVALIAR_ID`** (`1006`) em **`customfield_10020`**. Se `create_metadata` indicar tipo **array** para Sprint, usar **`[1006]`**; se aceitar inteiro, usar **`1006`**. Se a criação falhar por formato, tentar o outro formato (inteiro ↔ array de um elemento).

**Manutenção:** se o sprint **AVALIAR** for recriado na Jira e o ID mudar, atualizar a tabela com o novo ID (descobrir via `GET /rest/agile/1.0/board/49/sprint?state=future,active` ou URL do sprint `sprint=…`).

---

## EXEMPLO DE DISPARO (COMENTADO)

# /bug-report-etapa-desenvolvimento
# tarefa pai: NS-666
# ↑ (obrigatório) linha com a chave da issue pai — depois o texto bruto do bug ({{TEXTO_BRUTO}})

# O mesmo pedido pode ir **com imagens** na mesma mensagem (anexar prints ao chat).
# {{IMAGENS_ANEXADAS}} = ficheiros de imagem com caminho absoluto quando disponível.

# Frases equivalentes para a tarefa pai:
# - "tarefa pai: NS-953"
# - "pai: ns-907"

# Regras:
# - **{{TAREFA_PAI}}** obrigatória; sem ela → **interromper** tudo e pedir `tarefa pai: NS-XXX`.
# - Com pai na mensagem: validar com `jira_issues` **get** antes de Fase A; se pai **não existir** → **interromper**, **sinalizar**, pedir **nova** chave PAI (não criar issue, não bug report completo).
# - **{{TEXTO_BRUTO}}** = resto da mensagem **sem** as linhas que só declaram a tarefa pai.
# - Imagens: mesma mensagem ou caminhos explícitos → {{IMAGENS_ANEXADAS}} (só após pai validado).
# - Se {{TEXTO_BRUTO}} estiver vazio ou for só ruído (após remover pai), pedir o texto do bug e parar (após pai válido).

---

## Objetivo

Extrair **{{TAREFA_PAI}}**, **validar** com `jira_issues` **get**; **se a tarefa pai não existir ou o get falhar — interromper**, sinalizar e pedir de novo a chave **Pai**. Só depois de validação bem-sucedida: transformar **{{TEXTO_BRUTO}}** em bug report Roboteasy (português) e **criar Bug filho** via MCP **sempre** com **Pai** = `{{TAREFA_PAI}}`, campos `###`, **prioridade Média**, sprint **AVALIAR** (`1006`), e anexos quando existirem.

---

## REGRAS OBRIGATÓRIAS

### Fase A — Documento

- Executar **somente depois** de `{{TAREFA_PAI}}` estar **validada** com sucesso pelo `jira_issues` `get` (quando o MCP estiver disponível).
- Produzir o bug report completo no **MODELO OBRIGATÓRIO** (markdown), usando como fonte apenas **{{TEXTO_BRUTO}}** (já sem a linha da tarefa pai).
- Usar **exatamente** os títulos de seção abaixo (com `###` e o nome indicado).
- Preencher cada seção com base em {{TEXTO_BRUTO}}; onde faltar detalhe, usar marcadores genéricos claros **somente** quando necessário.
- Listas em **Passos para Reproduzir**, **Comportamento Atual**, **Comportamento Esperado** e **Critérios de Aceitação** devem usar `-` ou numeração `1.` conforme o exemplo.
- Linguagem técnica clara (rotinas, módulos, laços, etc.) quando o texto original assim indicar.
- Não copiar este arquivo inteiro como resposta.

### Fase B — Jira (MCP)

- **Pré-requisito:** `{{TAREFA_PAI}}` presente **e** `jira_issues` **`get`** com **sucesso** nesta sessão. Sem isso, **não** há Fase B.
- Após o markdown estar pronto, **obrigatório** criar a issue no Jira usando o servidor MCP **`user-jira`** (ferramentas `jira_search`, `jira_issues`, **`jira_attachments`** se houver imagens).
- Se o MCP Jira **não** estiver disponível **antes** da validação do pai: **interromper** conforme secção **Tarefa pai** (não criar issue). Se cair indisponível **depois** da Fase A (caso raro), entregar o markdown e declarar que a criação/anexos não foram executados.
- **Antes do create:** partir o markdown final em secções pelos títulos `### …` conforme a tabela **Mapeamento `###` → campos na tarefa Jira** em **CONFIGURAÇÃO JIRA**.
- Fluxo sugerido:
  1. `jira_search` com `action`: `create_metadata`, `projectKey`: valor de **PROJECT_KEY** deste ficheiro, `issueType`: `Bug`.
  2. `jira_issues` com `action`: `create`, `projectKey`, `issueType`: `Bug`, `priority`: **`Medium`**, `summary`: **uma linha** (da secção **Descrição**).
  3. `description`: corpo da secção **### Descrição** apenas (string).
  4. `customFields`: ADF nos `customfield_10827`, `10828`, `10134`, `10829` quando aplicável; **`customfield_10020`** = sprint **1006** (ou `[1006]`); **`"parent": { "key": "{{TAREFA_PAI}}" }`**.
  5. Se o `create` falhar no `parent`, tentar `update` só com `parent`; se persistir, informar o utilizador (pode ser restrição de tipo de issue pai ou projeto).
  6. **Anexos:** `jira_attachments` `upload` por imagem após sucesso do create.
- No final da resposta: link/chave da issue nova, **confirmar tarefa pai `{{TAREFA_PAI}}`**, quantidade de anexos, e o markdown do relatório.

---

## MODELO OBRIGATÓRIO (estrutura e ordem)

Reproduzir esta estrutura. O conteúdo entre seções vem de {{TEXTO_BRUTO}}.

### Descrição
[Parágrafo(s) explicando o problema, contexto e consequência.]

### Passos para Reproduzir
1. [Passo numerado]
2. […]
3. […]

### Comportamento Atual
- [Bullet com o que acontece hoje]
- […]

### Comportamento Esperado
- [Bullet com o que deveria acontecer]
- […]

### Critérios de Aceitação
- [Critério testável ou verificável]
- […]

---

## Referência de qualidade (não copiar literalmente)

Use apenas como guia de tom e completude — adapte sempre a {{TEXTO_BRUTO}}:

### Descrição
Existe uma inconsistência entre os módulos Varrer Dados Excel e Formatar Célula.  
Quando o contador do Varrer Dados Excel inicia em 0, o valor do identificador funciona normalmente no laço, porém se torna inválido ao ser usado no Formatar Célula, pois este módulo interpreta que a contagem de células inicia em 1 (padrão Excel).  
Como consequência, ao usar A{count} com count = 0, o sistema tenta formatar a célula "A0", que não existe.

### Passos para Reproduzir
1. Criar uma rotina contendo:
   - Ler Dados Excel  
   - Varrer Dados Excel com:
     - Valor inicial do contador: 0  
     - Identificador: count  
   - Dentro do laço, adicionar Formatar Célula usando a referência A{count}.
2. Executar a rotina.
3. Observar que, na primeira iteração, a célula gerada é A0.

### Comportamento Atual
- Varrer Dados Excel utiliza indexação começando em 0, seguindo o padrão do C#.  
- Formatar Célula utiliza indexação começando em 1, seguindo o padrão do Excel.  
- Isso gera uma célula inválida (A0) quando o contador vale 0.

### Comportamento Esperado
- Ambos os módulos devem utilizar a mesma lógica de indexação.  
- Contadores iniciados em 0 não devem gerar referências inválidas.  
- A primeira célula válida deve ser algo como A1, nunca A0.

### Critérios de Aceitação
- A rotina deve aceitar contador iniciando em 0 sem gerar erros no Formatar Célula.  
- Os módulos devem estar alinhados quanto à interpretação de índices.  
- Não deve ser possível gerar referências inválidas como A0.

---

## Execução (ordem)

1. Extrair **{{TAREFA_PAI}}**. Se ausente → **interromper**, pedir `tarefa pai: NS-XXX`, **fim** (sem Fase A/B).
2. Com MCP: `jira_issues` **`get`** em `{{TAREFA_PAI}}`. Se falhar → **interromper**, **sinalizar** («tarefa pai não encontrada» ou equivalente), mostrar chave tentada, **pedir nova chave PAI**, **fim** (sem Fase A/B, sem `create`, sem anexos).
3. Sem MCP: **interromper** fluxo Jira conforme regras da tarefa pai (não criar issue).
4. Montar **{{TEXTO_BRUTO}}** (sem linhas só da pai); **{{IMAGENS_ANEXADAS}}**.
5. Se **{{TEXTO_BRUTO}}** inválido após remover pai → pedir texto do bug e **fim**.
6. Fase A → Fase B: `create` com **parent** obrigatório + campos + sprint + anexos.
7. Resposta de sucesso: markdown + link/chave + **pai `{{TAREFA_PAI}}` confirmado** + anexos.
