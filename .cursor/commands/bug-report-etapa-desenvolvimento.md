# bug-report-etapa-desenvolvimento

exemplo de disparo: `/bug-report-etapa-desenvolvimento: ao guardar a rotina o painel fica em branco; reproduz sempre com ficheiro XLSX > 5 MB anexado ao passo 3. {NS-985}`

> Fluxo para **etapa de desenvolvimento**: criar **Bug** na Jira a partir do texto do utilizador. **Diferença do** `bug-report-roboteasy`: aquele coloca o bug na fila/sprint **AVALIAR** (Futuro) — **correto para bugs gerais**. **Este** comando coloca o bug na **sprint de desenvolvimento atual** (na UI, o mesmo tipo de sprint que a tarefa em curso — ex. **022026**, não **AVALIAR**), alinhado à tarefa que o utilizador indica; **obrigatório** **ligar** ao trabalho de desenvolvimento: ligação em que o **bug bloqueia** a tarefa cuja chave veio no disparo (`{{TAREFA_PAI}}` em `{NS-XXX}`) — na Jira, a **tarefa referenciada fica bloqueada pelo bug** (ela **is blocked by** o bug). **Sem** prioridade fixa por regra deste fluxo. **Resposta de sucesso:** frase fixa com link e chave da tarefa do disparo (ver **Resposta ao utilizador**).



## Formato de disparo (obrigatório)



```text

/bug-report-etapa-desenvolvimento: {texto livre com todo o conteúdo do bug} {CHAVE_DA_TAREFA_PAI}

```



- **Depois dos dois pontos** (`:`), primeiro vem o **texto** que descreve o bug (detalhes, passos, imagens referenciadas por caminho, etc.).

- **Em seguida**, na **mesma mensagem**, a **tarefa pai** (chave da tarefa que o **bug vai bloquear** após criar o link) deve aparecer **entre chaves**, no formato de chave Jira, **exemplos:** `{NS-985}`, `{ns-985}` (normalizar para maiúsculas ao usar na API e na resposta, ex.: `NS-985`). **`{{TAREFA_PAI}}`** é sempre esta chave extraída do disparo.



**Exemplo (estrutura exata):**



```text

/bug-report-etapa-desenvolvimento: aqui vou mandar tudo que vai ser para construção do bug como detalhes passo a passo, imagens e etc. {NS-985}

```



**Extração de `{{TAREFA_PAI}}`:**



- Procurar na mensagem completa (incluindo linhas após o comando) por **chaves Jira dentro de `{}`**: padrão sugerido `(?i)\{\s*([A-Z][A-Z0-9]*-\d+)\s*\}`.

- Se existir **mais de uma** ocorrência válida, usar a **última** como `{{TAREFA_PAI}}` — tarefa que o bug deve **bloquear** (costuma ser o sufixo do disparo, como no exemplo).

- **`{{TEXTO_BRUTO}}`** = o texto do bug **sem** a ocorrência `{…}` escolhida para `{{TAREFA_PAI}}` (e sem o prefixo `/bug-report-etapa-desenvolvimento:` se estiver na mesma linha).



### Regra inicial — tarefa pai ausente



Se **não** existir nenhuma chave válida entre `{}` na mensagem de disparo:



- **Interromper** imediatamente (sem Fase A, sem Jira `create`, sem anexos).

- Responder **exatamente** com **uma linha**, sem texto extra:



```text

> Está faltando a tarefa pai desse bug que você quer criar

```



---



## CONFIGURAÇÃO JIRA (editar uma vez neste arquivo)



| Chave | Valor |

|--------|--------|

| `PROJECT_KEY` | `NS` |

| `JIRA_BOARD_ID` | `49` — board Jira Software de referência (mesmo contexto que `bug-report-roboteasy`; útil para documentação e consultas Agile manuais). |

| `SPRINT_ATUAL_FALLBACK_ID` | *(opcional)* ID API numérico da **sprint ativa de desenvolvimento** quando a tarefa pai **não** trouxer sprint em `customfield_10020`. Na UI costuma aparecer como código tipo **022026** / **032026** (secção diferente de **AVALIAR**). **Atualizar** quando mudar a sprint ativa. **Nunca** usar aqui o ID do sprint **AVALIAR** (`1006` — esse é só do `bug-report-roboteasy`). |



**Não aplicar neste comando:** sprint **AVALIAR** (`SPRINT_AVALIAR_ID` / `1006` do `bug-report-roboteasy`). **Não** deixar o campo Sprint vazio por omissão: o bug de desenvolvimento deve ir para a **sprint atual** (ver secção **Sprint — desenvolvimento atual** abaixo). **Prioridade:** não forçar **Média** só por este fluxo — usar default do projeto ou o que `create_metadata` exigir.



### Tarefa de referência (`{{TAREFA_PAI}}`) e ligação **o bug bloqueia a tarefa**



- **`{{TAREFA_PAI}}`** vem **obrigatoriamente** do disparo, no formato `{NS-XXX}` — é a tarefa (no texto do fluxo, «tarefa pai») que deve ficar **bloqueada pelo bug** depois do link.

- **Obrigatório (com MCP):** antes de criar o bug, `jira_issues` **`get`** com `issueKey`: `{{TAREFA_PAI}}`. Se falhar → interromper, sinalizar claramente (tarefa não encontrada / sem permissão), **não** criar issue. A resposta do `get` serve para **(1)** validar a tarefa e **(2)** obter a **Sprint** já atribuída à tarefa de desenvolvimento (`customfield_10020`), quando existir, para replicar no bug novo.

- **Sem MCP Jira:** não criar issue; explicar que não é possível cumprir o fluxo.



### Sprint — desenvolvimento atual (obrigatório; não é AVALIAR)



- **Objetivo:** o bug criado por este comando deve ficar na **mesma sprint de desenvolvimento** que o trabalho em curso — como no campo **Sprint** da tarefa no ecrã (ex. **022026**), **não** na fila **AVALIAR** (isso fica reservado ao `bug-report-roboteasy`).

- **Procedimento preferido:** após o **`get`** em `{{TAREFA_PAI}}`, ler **`customfield_10020`** (Sprint) na issue pai. Se vier **um ou mais IDs** de sprint, usar **o mesmo valor** (formato que a API aceitar: inteiro ou array de IDs, como em `bug-report-roboteasy` para `customfield_10020`) no `jira_issues` **`create`** do bug. Assim o bug entra na sprint onde a tarefa pai já está.

- **Se a tarefa pai não tiver sprint** (`customfield_10020` vazio ou ausente): usar **`SPRINT_ATUAL_FALLBACK_ID`** da tabela de configuração, **se** estiver preenchido. Se também não houver fallback → interromper antes do `create` com mensagem clara: é preciso sprint de desenvolvimento (pai com sprint ou ID de fallback na tabela).

- **Nunca** neste fluxo: preencher sprint com o ID **AVALIAR** (`1006`) salvo instrução explícita do utilizador **nesta conversa** (caso raro).

- **Nota técnica:** o campo Sprint na API costuma ser **`customfield_10020`**; confirmar com `create_metadata` se o projeto usar outro id.

- **Manutenção do fallback:** para preencher ou atualizar **`SPRINT_ATUAL_FALLBACK_ID`**, usar o ID numérico do sprint ativo (ex.: `GET /rest/agile/1.0/board/{JIRA_BOARD_ID}/sprint?state=active` ou URL do sprint com parâmetro `sprint=…`), **não** o nome texto **022026** na API.



**Após** criar o bug com sucesso (chave `{{BUG_KEY}}`, URL `{{BUG_URL}}`):



- O **bug** deve **bloquear** `{{TAREFA_PAI}}`. Na UI Jira: na tarefa `{{TAREFA_PAI}}` aparece que ela **is blocked by** o bug; na issue do bug, o link descreve-se como o bug **blocks** a outra.

- Na API REST Jira v3, tipo **Blocks**: a issue que **bloqueia** vai em **outwardIssue**; a issue **bloqueada** vai em **inwardIssue**. Para **«o bug bloqueia `{{TAREFA_PAI}}`»**: **`outwardIssue`** = `{{BUG_KEY}}` (bug), **`inwardIssue`** = `{{TAREFA_PAI}}` (tarefa do disparo). Se a instância inverter nomes nos metadados do tipo de link, ajustar **inward/outward** mantendo o significado: **bloqueador = bug**, **bloqueada = `{{TAREFA_PAI}}`**.

- **Se o MCP `user-jira` não expuser criação de issue links:** criar o bug, fazer upload de anexos se aplicável; na **resposta** usar **mesmo assim** o modelo de sucesso abaixo (link + `{{TAREFA_PAI}}`) e, se necessário, **uma frase curta** a pedir que confirme o link manualmente na Jira até existir ferramenta MCP para `issueLink`. **Não** devolver o relatório markdown completo como «entrega» principal.



**Opcional:** se `create_metadata` ou política do projeto **exigirem** também o campo **Pai** (`parent`), preencher `"parent": { "key": "{{TAREFA_PAI}}" }` em `customFields` **além** do link **Blocks** (bug bloqueia tarefa), quando ambos forem compatíveis com o tipo **Bug**.



Ao gerar conteúdo para as secções `###`, **remover** do texto fonte o token `{CHAVE}` da tarefa pai.



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



O utilizador pode **anexar imagens ao chat** na mesma mensagem que dispara o comando (colar print, arrastar ficheiro, etc.). O assistente deve:



1. **Identificar** todos os ficheiros de imagem disponíveis para esta mensagem: caminhos absolutos indicados pelo ambiente, ou caminhos que o utilizador tenha colado no texto.

2. **Só depois** da issue existir: usar o MCP **`user-jira`**, ferramenta **`jira_attachments`**, com `action`: **`upload`**, `issueKey`: chave do bug criado, `filePath`: caminho absoluto, `fileName` opcional.

3. **Repetir** um `upload` por imagem.

4. Se **não** existir caminho de ficheiro acessível: informar brevemente (fora da frase fixa de sucesso — em caso de sucesso parcial, ver secção **Resposta ao utilizador**).

5. Formatos típicos: `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`.



---



## EXEMPLO DE DISPARO (COMENTADO)



```text

/bug-report-etapa-desenvolvimento: descrição longa do bug… {NS-985}

```



- **`{NS-985}`** obrigatório (ou equivalente `PROJECT-N`).

- Sem `{…}` com chave Jira → mensagem fixa de falta de tarefa pai (secção acima).

- Imagens na mesma mensagem → upload após `create`.



---



## Objetivo



Extrair **`{{TAREFA_PAI}}`** das chaves `{}`; se ausente, mensagem fixa e **fim**. Validar com `get` quando possível; **a partir do `get`**, definir sprint do bug = sprint da tarefa pai (`customfield_10020`) ou **`SPRINT_ATUAL_FALLBACK_ID`**. Transformar **`{{TEXTO_BRUTO}}`** em bug report (Fase A), criar **Bug** (Fase B) na **sprint de desenvolvimento atual** (**não** AVALIAR), **sem** prioridade Média forçada, **criar ligação** em que o **bug bloqueia** `{{TAREFA_PAI}}`, anexos se houver. **Responder** com a **frase fixa** da secção **Resposta ao utilizador** (link + chave `{{TAREFA_PAI}}` do disparo).



---



## REGRAS OBRIGATÓRIAS



### Fase A — Documento



- Executar **somente depois** de existir `{{TAREFA_PAI}}` na mensagem e, se MCP disponível, **`get`** com sucesso.

- Produzir o bug report completo no **MODELO OBRIGATÓRIO** (markdown), usando como fonte apenas **`{{TEXTO_BRUTO}}`** (já sem o `{CHAVE}` da tarefa pai).

- O markdown serve para **mapear campos** na criação; **não** entregar o markdown completo ao utilizador como resultado final em caso de sucesso.



### Fase B — Jira (MCP)



- **Pré-requisito:** `{{TAREFA_PAI}}` presente; com MCP, **`get`** com sucesso.

- `jira_search` `create_metadata` para `PROJECT_KEY`, `issueType`: `Bug`.

- `jira_issues` `create`: `projectKey`, `issueType`: `Bug`, `summary`, `description`, `customFields` com ADF nos campos mapeados **e** **`customfield_10020`** = sprint obtida do **`get`** da tarefa pai (ou fallback da tabela); **não** usar sprint **AVALIAR** (`1006`) por defeito. **Não** definir `priority` salvo obrigatoriedade explícita no metadata.

- **`parent`:** só se o projeto/tipo exigir; caso contrário, a obrigação principal é a **ligação Blocks** com `{{TAREFA_PAI}}`: o **bug bloqueia** a tarefa que o utilizador indicou no disparo (trabalho de desenvolvimento ligado ao bug).

- Após `create`: **issue link** tipo **Blocks** com **bloqueador** = bug novo, **bloqueada** = `{{TAREFA_PAI}}` (via API/issueLink quando existir mecanismo).

- Anexos: `jira_attachments` `upload` por imagem.



### Resposta ao utilizador



- **Sucesso:** **exatamente uma linha** no formato (substituir placeholders pelos valores reais):

```text

Criado o bug {link do bug}, esse bug bloqueia a tarefa {tarefa pai}

```

  - **`{link do bug}`** = URL completo da issue do bug na Jira (ex.: `https://…/browse/NS-456`).

  - **`{tarefa pai}`** = chave **`{{TAREFA_PAI}}`** tal como extraída e normalizada do disparo (ex.: `NS-985`), **a mesma** que estava entre `{}` na mensagem do utilizador.

  - **Sem** markdown do relatório, **sem** contagem de anexos na mesma linha. **Salvo** falha crítica em anexos ou impossibilidade de criar o link: pode acrescentar **uma** frase curta **na linha seguinte** se for essencial para o utilizador agir.

- **Erro** (tarefa pai ausente): apenas `> Está faltando a tarefa pai desse bug que você quer criar`.

- **Outros erros:** mensagem clara e curta; não simular sucesso.



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



1. Extrair **`{{TAREFA_PAI}}`** de `{CHAVE}` na mensagem. Se ausente → `> Está faltando a tarefa pai desse bug que você quer criar` → **fim**.

2. Com MCP: `jira_issues` **`get`** em `{{TAREFA_PAI}}`. Se falhar → interromper, sinalizar, **fim**. Da resposta, extrair **`customfield_10020`** (Sprint) para usar no `create` do bug; se vazio, usar **`SPRINT_ATUAL_FALLBACK_ID`** ou interromper conforme secção **Sprint — desenvolvimento atual**.

3. Sem MCP na parte Jira → não criar issue; **fim**.

4. Montar **`{{TEXTO_BRUTO}}`** (sem o token `{…}` da pai); imagens como antes.

5. Se **`{{TEXTO_BRUTO}}`** vazio ou só ruído → pedir texto do bug e **fim**.

6. Fase A → Fase B: `create` com **sprint = pai** (`customfield_10020` do `get` em `{{TAREFA_PAI}}`) ou **`SPRINT_ATUAL_FALLBACK_ID`**; **não** AVALIAR; **issue link** em que o **bug bloqueia** `{{TAREFA_PAI}}`; anexos.

7. Resposta: uma linha no modelo **Criado o bug** + URL + **, esse bug bloqueia a tarefa** + **`{{TAREFA_PAI}}`** (chave do disparo, normalizada).


