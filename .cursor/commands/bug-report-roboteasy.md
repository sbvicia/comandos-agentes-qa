# bug-report-roboteasy

## CONFIGURAÇÃO JIRA (editar uma vez neste arquivo)

| Chave | Valor |
|--------|--------|
| `PROJECT_KEY` | `NS` |
| `SPRINT_AVALIAR_ID` | `1006` — ID API do sprint **AVALIAR** (Futuro, **Produto - 2026**, espaço NS); board Jira Software **`49`** |
| `PRIORIDADE` | **Média** — fixa em todas as issues criadas por este comando |

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

O utilizador pode **anexar imagens ao chat** na mesma mensagem que dispara `/bug-report-roboteasy` (colar print, arrastar ficheiro, etc.). O assistente deve:

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

# /bug-report-roboteasy
# ↑ Em seguida o usuário cola ou digita o texto bruto a formatar ({{TEXTO_BRUTO}})

# O mesmo pedido pode ir **com imagens** na mesma mensagem (anexar prints ao chat — colar captura de ecrã ou ficheiros de imagem).
# {{IMAGENS_ANEXADAS}} = ficheiros de imagem que o Cursor associa à mensagem (normalmente com **caminho absoluto** no disco, ex.: pasta workspaceStorage/.../images/*.png).

# Frases equivalentes que o usuário pode usar:
# - "Formate para o padrão bug report roboteasy: [texto aqui]"
# - "Bug report Roboteasy a partir de: [texto aqui]"

# Regras:
# - Texto: tudo o que o utilizador escrever após o comando (ou após os dois-pontos) é {{TEXTO_BRUTO}}.
# - Imagens: anexos de imagem na **mesma** mensagem do comando (ou caminhos de ficheiro `.png` / `.jpg` / `.jpeg` / `.gif` / `.webp` que o utilizador cole explicitamente) são {{IMAGENS_ANEXADAS}}.
# - Se {{TEXTO_BRUTO}} estiver vazio ou for só ruído, peça o texto do bug em uma única mensagem curta e pare (imagens sozinhas não bastam).

---

## Objetivo

Transformar {{TEXTO_BRUTO}} em um **bug report** no padrão Roboteasy, em **português**, sem inventar fatos que não estejam no texto ou que não sejam inferência óbvia e segura a partir dele — e **criar a issue Bug no Jira** via MCP, **distribuindo cada secção `### Tema` pelo campo Jira correspondente** (Descrição, Passos para Reproduzir, Comportamento Atual, Esperado, Critérios de Aceitação), com **prioridade Média** (`Medium`) e sprint **sempre a opção AVALIAR** (Futuro / NS), aplicando **`SPRINT_AVALIAR_ID`** (`1006`) em **`customfield_10020`** em todo o `create`. Se o utilizador enviar **prints ou imagens** juntamente com o comando (**{{IMAGENS_ANEXADAS}}**), **anexar esses ficheiros à issue** depois de criada, via MCP **`jira_attachments`** (upload com caminho local).

---

## REGRAS OBRIGATÓRIAS

### Fase A — Documento

- Produzir primeiro o bug report completo no **MODELO OBRIGATÓRIO** (markdown).
- Usar **exatamente** os títulos de seção abaixo (com `###` e o nome indicado).
- Preencher cada seção com base em {{TEXTO_BRUTO}}; onde faltar detalhe, usar marcadores genéricos claros **somente** quando necessário.
- Listas em **Passos para Reproduzir**, **Comportamento Atual**, **Comportamento Esperado** e **Critérios de Aceitação** devem usar `-` ou numeração `1.` conforme o exemplo.
- Linguagem técnica clara (rotinas, módulos, laços, etc.) quando o texto original assim indicar.
- Não copiar este arquivo inteiro como resposta.

### Fase B — Jira (MCP)

- Após o markdown estar pronto, **obrigatório** criar a issue no Jira usando o servidor MCP **`user-jira`** (ferramentas `jira_search`, `jira_issues` e, quando houver imagens com path local, **`jira_attachments`**), **se** o MCP estiver disponível nesta sessão.
- Se o MCP Jira **não** estiver disponível: entregar só o markdown e uma linha a dizer que a criação no Jira não foi possível (MCP indisponível).
- **Antes do create:** partir o markdown final em secções pelos títulos `### …` conforme a tabela **Mapeamento `###` → campos na tarefa Jira** em **CONFIGURAÇÃO JIRA**. Preencher **cada** campo Jira **apenas** com o texto do tema correspondente (não juntar tudo na Descrição, salvo fallback por erro de API).
- Fluxo sugerido:
  1. `jira_search` com `action`: `create_metadata`, `projectKey`: valor de **PROJECT_KEY** deste ficheiro, `issueType`: `Bug` (confirmar ids de campo se necessário).
  2. `jira_issues` com `action`: `create`, `projectKey`, `issueType`: `Bug`, `priority`: **`Medium`** (Média — obrigatório), `summary`: **uma linha** (preferencialmente da secção **Descrição**).
  3. `description`: **apenas** o corpo da secção **### Descrição** (sem a linha `### Descrição`), em string — o MCP trata da conversão ADF do campo principal.
  4. `customFields`: incluir **obrigatoriamente** (quando a secção existir no markdown) os ADF para **Passos para Reproduzir**, **Comportamento Atual**, **Comportamento Esperado** e **Critérios de Aceitação** nos `customfield_*` da tabela de mapeamento; ver nota **ADF nos custom fields** acima.
  5. **Sprint — obrigatoriamente AVALIAR:** em `customFields`, **`customfield_10020`** = valor da tabela **`SPRINT_AVALIAR_ID`** (**`1006`**, sprint **AVALIAR** no board **49**). Formato: inteiro **`1006`** ou array **`[1006]`** conforme o que a API aceitar (ver nota na secção Sprint). Se após `create` o sprint não aparecer, `jira_issues` `update` com o mesmo `customfield_10020`.
  6. **Anexos (imagens):** se existir **{{IMAGENS_ANEXADAS}}** com caminhos válidos, após o `create` bem-sucedido chamar **`jira_attachments`** com `action`: `upload` para cada ficheiro, conforme **Imagens / prints → anexos na Jira** acima.
- Se `create_metadata` listar **outro** id de campo para Sprint ou para os blocos de texto, ajustar em conformidade.
- No final da resposta ao utilizador: **link** e chave da issue + markdown do relatório; indicar **quantos anexos** foram enviados (ou que não houve imagens com path válido); mencionar brevemente a distribuição pelos campos de texto quando o mapeamento tiver sido aplicado.

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

1. Ler {{TEXTO_BRUTO}} e identificar **{{IMAGENS_ANEXADAS}}** (anexos / caminhos de imagem na mesma mensagem).
2. Produzir o bug report completo no modelo obrigatório (Fase A).
3. Executar Fase B (Jira: secções `###` → campos + ADF + **Medium** + sprint **AVALIAR** + **upload de anexos** quando aplicável) conforme **CONFIGURAÇÃO JIRA** e **REGRAS OBRIGATÓRIAS**.
4. Na resposta: markdown do relatório + chave/link da issue + confirmação dos anexos. Preferir começar o relatório em `### Descrição` sem preâmbulos longos.
