# CODE_REVIEW_QA

## Disparo

- Formato esperado — o **número / chave da tarefa** é **sempre fornecido pelo utilizador** ao disparar (não inventar nem adivinhar):

  `quero fazer o code review > {ns-721}`

- **`{ns-721}`** é só **exemplo**: no teu pedido vais indicar a **chave real** (ex.: `NS-721`). Podes usar chaves `{}` ou só o texto da chave após `>`. O agente **usa esse valor como entrada** para procurar no Jira (podes **normalizar** capitalização para a API, ex.: `NS-721`, mas a origem do valor é **o que o utilizador escreveu**).
- Objetivo: **garantir qualidade** do código entregue — **padrão do projeto**, **escalabilidade**, **segurança**, **respeito pela arquitetura** e **lente QA** (aceite, testes, testabilidade) — via Jira → painel **Desenvolvimento** → **pull request** (quando existir ligação clara) **ou** **commits** → diff no Bitbucket → revisão estruturada abaixo, **sem** substituir CI, SAST nem revisão humana de segurança.

## Caminho até ao diff no Bitbucket (visão de conjunto)

Ordem **fixa** — passos **1–5** identificam o **alvo da revisão** (preferencialmente **um PR** ligado à issue) e obtêm o **diff** correspondente; **6–11** são a revisão técnica.

| Fase | Passo | O quê |
|------|--------|--------|
| Jira | 1 | Chave dada pelo utilizador |
| Jira | 2 | Localizar tarefa (MCP); se falhar → `não localizei a tarefa` |
| Jira | 3 | Painel **Desenvolvimento** — identificar **pull request(s)** e **commit(s)** associados (ex.: **«N pull request(s)»**, **«N commit(s)»**, ou diálogo **Desenvolvimento de [CHAVE]** com separadores **Pull requests** / **Commits**; MCP em paralelo se devolver ligações) |
| Jira → BB | 4 | **Prioridade PR:** se existir **exatamente um** pull request ligado à entrega da issue (aberto ou merged), esse PR é o **alvo** — **não** substituir por “só o último commit” salvo **pedido explícito** do utilizador para rever um commit concreto. **Vários PRs** sem critério óbvio (ex.: mesma relevância, empate de datas): **parar** e pedir ao utilizador **qual PR** (ou link). **Sem PR** (ou integração sem PR visível): usar fluxo **commits** — **um commit** → alvo; **vários** → comparar **data** (e hash/mensagem); **mais recente** → único alvo |
| Jira → BB | 5 | **Se alvo = PR:** obter o **diff do PR** no Bitbucket (MCP `getPullRequestDiff`, ou abrir o PR → vista **diff** na UI). **Se alvo = commit:** abrir **diff desse commit** (ex.: **Visualizar** na linha do commit, hash clicável, ou MCP) |
| Revisão | 6–11 | Diff do **alvo** (PR **ou** commit) + contexto pré-mudança + síntese + validação final |

**Nota:** sem automação de browser, **3–5** são UI do utilizador ou dados/URL vindos de MCP; o agente trabalha com **diff colado**, **screenshot**, **ficheiros no workspace** ou **API**, conforme disponível.

**Regra do alvo único:** o **resultado final** é **um** code review sobre **um** diff — o do **PR escolhido** **ou** o do **commit escolhido**. **Não** fundir várias revisões completas (vários PRs ou vários commits) numa só mensagem.

**Quando o fluxo é por commits** (sem PR ou por escolha explícita): com **2 ou mais** commits na lista, **registar** (hash curto, data, primeira linha da mensagem) **só para decidir o mais atual**; **obter e rever unicamente o diff desse commit** (por **data** na UI Jira). O hash escolhido **não** vai na resposta ao utilizador **salvo** menção **mínima** dentro de **Alterações possíveis** ou **Pacotes impactados** quando for **estritamente** necessário. Se **não** for possível ordenar por data (empate ou dados em falta), **parar** e pedir ao utilizador qual commit rever.

**Quando o fluxo é por PR:** o diff do PR já agrega as alterações típicas da entrega; **não** é obrigatório rever **adicionalmente** cada commit do PR como revisões separadas neste comando.

## Início do fluxo (obrigatório nesta ordem)

### 1. Chave fornecida pelo utilizador

- O **número da tarefa** vem **do pedido** no formato acima: o token após **`>`** (remover `{}` só se forem decoração — o significado é: **a chave que o utilizador deu**).
- **Não** há “extração” de outra fonte: sem chave no disparo, **parar** e pedir: `quero fazer o code review > {CHAVE}` com a **CHAVE** explícita.

### 2. Jira via MCP

- Usar o **servidor MCP do Jira** configurado no Cursor (**identificador típico:** `user-jira`) para **localizar a tarefa** com a chave **que o utilizador forneceu**.
- Se **não** localizar a tarefa no Jira (inexistente, sem permissão, MCP sem resultado válido): **travar o comando** e **não** avançar para os passos seguintes. **Responder ao utilizador** com a frase:

  **não localizei a tarefa**

### 3. Painel **Desenvolvimento** — PR e commits

- Na issue encontrada, trabalhar com a informação exposta pelo Jira/MCP (campos, ligações de desenvolvimento, integrações Bitbucket, etc.).
- Identificar **pull request(s)** e **commit(s)** associados à tarefa. Exemplos de UI:
  - **«1 pull request»** / **«N pull request(s)»** no painel **Desenvolvimento**; separador **Pull requests** no diálogo de desenvolvimento; ou
  - **«1 commit»** / **«N commits»**; separador **Commits** no diálogo **Desenvolvimento de [CHAVE]**.
- Se o MCP devolver IDs de PR, URLs ou lista de commits com datas e hashes, usar em paralelo com a UI.

### 4. Escolher o **alvo** — PR em primeiro lugar

- **4a — Pull request (prioridade)**  
  - **Um único** PR ligado à entrega da issue → esse PR é o **alvo** → passo **5** (diff do PR).  
  - **Dois ou mais** PRs: se um for claramente o da feature (ex.: título com a chave da issue, branch dedicada) e os outros forem ruído, podes escolher esse **internamente** com critério conservador; caso contrário **parar** e pedir ao utilizador **qual PR** rever (ou o link do PR).  
  - **Pedido explícito** do utilizador para rever **um commit** ou **um hash** → esse commit passa a **alvo** mesmo existindo PR (modo excecional).
- **4b — Só commits (sem PR utilizável)**  
  - **Um único** commit na lista (ou só uma entrada relevante): esse é o **alvo** → passo **5**.  
  - **Dois ou mais commits**: **comparar** data (e hash/mensagem); o **mais recente** no tempo é o **único** alvo. **Não** agregar revisões de vários commits numa só mensagem. Empate ou dados ambíguos → **parar** e pedir ao utilizador qual hash rever.  
- **Internamente:** registar alvo (tipo PR vs. commit, id/hash, datas). **Não** incluir essa linha na resposta ao utilizador (só os quatro blocos).

### 5. Obter o **diff** no Bitbucket (PR ou commit)

- **Alvo = PR:** usar MCP Bitbucket (`getPullRequestDiff` / `getPullRequestPatch` conforme disponível) com `workspace`, `repo_slug` e `pull_request_id`, **ou** abrir na UI o PR → vista **Diff**. **Objetivo:** ficheiros alterados + diff **desse PR** (o conjunto que a equipa revê antes do merge).
- **Alvo = commit:** na linha do commit, abrir **Visualizar** / hash no Bitbucket → diff **desse** commit.
- Se o diff **não** estiver acessível ao agente, **instruir o utilizador** (UI): para PR — abrir o link do PR e o separador de diff; para commit — no separador Commits, **Visualizar** (ou hash) na linha do commit escolhido.

---

## Formato obrigatório da resposta ao utilizador

Depois de executar a análise (passos **6–11** e cobertura **A–I** **internamente**), a **mensagem ao utilizador** contém **apenas** os **quatro** blocos abaixo, **nesta ordem**, com os **títulos literais** (e dois-pontos). A primeira linha visível da resposta tem de ser **`Tarefa:`** (início do primeiro bloco). **Proibido** qualquer **resumo**, **introdução** ou **contexto** antes desse primeiro bloco (ex.: localização no Jira, número do PR, repositório, data de merge, explicação de qual commit ou PR foi escolhido como alvo, síntese dos AC ou do diff). Esse contexto fica **só no raciocínio interno**. **Nada** após o último bloco.

```
Tarefa: [repetir a chave tal como o utilizador enviou no disparo, ex.: NS-721]

Pacotes alterados: [pastas/módulos/pacotes cujos ficheiros entram **diretamente** no diff analisado — PR **ou** commit, conforme o alvo nos passos **4–5**]

Pacotes impactados pela alteração: [módulos/pacotes **consumidores** ou afetados pelo novo comportamento: quem importa, quem partilha validação/estado/UI, risco de regressão; **não** copiar só a lista dos alterados se o impacto for mais largo]

Alterações possíveis no código:

1. **Título da sugestão** — O quê mudar e porquê (padrão, AC, segurança, performance, etc.).
2. **Outra sugestão** — …
(se não houver sugestões: uma linha só, ver regras abaixo)
```

**Formatação do bloco «Alterações possíveis no código» (opção A — obrigatória)**

- **Depois dos dois-pontos**, **saltar linha** e usar **lista numerada** (`1.`, `2.`, `3.`, …).
- **Cada número = uma sugestão isolada** (não juntar várias ideias no mesmo item).
- **Estrutura de cada item:** `N. **Título curto** — ` seguido de **uma ou duas frases** com *o quê* alterar e *porquê* (padrão, segurança, performance, AC, arquitetura, edge case, regressão).
- **Legibilidade:** título em negrito até o primeiro `—`; evitar parágrafos longos; sem bullet `*` dentro deste bloco (só numeração).
- **Se não houver** sugestão objetiva: **uma única linha** após os dois-pontos (sem lista numerada): `Nenhuma sugestão objetiva além do já implementado.`

**Exemplo (ilustrativo):**

```
Alterações possíveis no código:

1. **Verificar imports órfãos** — Correr `grep` por `symbolX` no repo; se existir import após remoção do export, corrigir para evitar build quebrado.
2. **Filtrar opções por tipo** — Se o AC exige só listas, restringir `getProperties` ao tipo enumerable para não poluir o select.
```

**Fim do padrão.**

### O que **não** incluir na resposta ao utilizador

- **Texto antes do primeiro bloco** — Nenhum parágrafo, título ou lista **acima** de `Tarefa:`; o retorno padrão do comando **começa logo** em `Tarefa:`.
- Tabelas **checklist** (rápida ou estendida), colunas **Sim/Não/N/A**, nem linhas do género *«Testabilidade / observabilidade — N/A — …»*.
- Frases sobre **o que o comando não faz** ou **não substitui** (SAST, CI, linter, testes automáticos, revisão humana), *«Não verificado pelo agente»*, *«Esta revisão não substitui…»*, nem **avisos meta** sobre limitações da ferramenta.
- **Conclusões** formais *Aprovar / Aprovar com ressalvas / Não aprovar*, **rastreio AC** linha-a-linha, **listagens de commits/hashes** ou **diff** completo — salvo **uma citação mínima** dentro de **Alterações possíveis** quando for estritamente necessário para justificar uma sugestão.
- Explicações sobre **como** o slash command funciona.

*(A cobertura A–I e os passos 9–11 servem para **raciocínio interno**; o utilizador só vê os quatro blocos, **sem** resumo nem preâmbulo.)*

---

## Revisão de código (objetivo: qualidade, padrão, escalabilidade, QA)

**Princípio:** revisão **objetiva** — alinhar com **regras e padrões do projeto**, não com gosto pessoal. Executar **6 → 11** na ordem e cumprir **integralmente** a secção **«Cobertura obrigatória — lacunas fechadas»** (incluindo **I — Anti-falsos-positivos**) **no raciocínio interno**. O **único** texto enviado ao utilizador é o definido em **«Formato obrigatório da resposta ao utilizador»**.

### 6. Obter o conteúdo a revisar + contexto da issue

- No Bitbucket: vista de **ficheiros alterados** + **diff** do **alvo** definido no passo **4–5** — **PR** (diff do pull request) **ou** **commit** (diff desse commit), conforme o caso (ou equivalente via MCP).
- **Em paralelo (MCP / campos da tarefa):** extrair **descrição** e **critérios de aceite** (ou equivalente). Se **não** existirem no Jira, usar **só internamente** — **não inventar** AC; **não** escrever na resposta ao utilizador frases do género *«AC não disponíveis»*.
- **Convenção visual:** **verde** / `+` = código **novo** (entra); **vermelho** / `-` = código **antigo** (sai).
- Se o diff **não** estiver acessível ao agente (só URL privada), **parar** e pedir ao utilizador: colar o diff, anexar screenshot legível, ou abrir o **repositório local** no Cursor no **commit** ou **branch** que o utilizador indicar.

### 7. Identificar pacote(s) / área alterada

- A partir da **lista de paths** do **diff analisado** (PR ou commit), indicar **que pacote(s), pasta(s) ou módulo(s)** estão em causa (ex.: `src/store/module-project/`, `front-studio/...`).
- Se o diff tocar **várias áreas**, tratar **cada área** nos passos seguintes (ou agrupar por coerência).

### 8. Baseline — padrão da pasta **antes** da mudança

- **Antes** de julgar o diff, **consultar o código existente** na(s) mesma(s) pasta(s) / pacote(s) no **workspace** (repositório local aberto no Cursor):
  - Ler ficheiros vizinhos, convenções de **nomenclatura**, **estrutura de pastas**, padrões de **imports**, **camadas** (store, serviços, componentes), **tratamento de erros**, estilo já usado ali.
- **Branch / estado de referência:** preferir o **pai** do commit de merge / **base do PR** (`git show` / merge-base no repo local), **development**, ou o que o utilizador indicar; se só existir o branch da feature no disco, ler o contexto atual e usar o diff para inferir o “antes” nas linhas removidas.
- Se **não** houver repo local: basear-se no **diff completo** + trechos que o utilizador fornecer; **não** inventar ficheiros não mostrados; **não** explicar na resposta ao utilizador que “faltou repo local” (cumprir os quatro blocos com o que o diff permitir).

### 9. Analisar a mudança no código

- Relacionar o diff com o **baseline** do passo **8** e com as **regras obrigatórias** da secção **«Cobertura obrigatória — lacunas fechadas»** (QA, correção/edge cases, performance, segurança operacional, código agentico quando aplicável, **e anti-falsos-positivos I** antes de emitir sugestões genéricas).
- A **lógica nova** segue o **padrão** da pasta alterada? A **escrita** (formatação, abstrações, nomes) respeita a **regra implícita** do sítio onde o código vive?
- Identificar **riscos objetivos**: regressão em consumidores do mesmo módulo, mutações partilhadas, APIs públicas alteradas.

### 10. Síntese interna (não reproduzir como secções separadas na resposta)

Construir **mentalmente** (ou em raciocínio interno) antes do passo **11**:

1. **Efeito** da mudança (comportamento / técnico) e **lista de ficheiros-chave**.
2. **Mapeamento AC ↔ diff** (coberto / parcial / não visível) — **só** para informar o texto de **Alterações possíveis** quando houver lacuna face aos AC; **não** listar AC na saída.
3. **Merge vs outros commits** (secção **F**): se o alvo for **commit** de **merge** e existirem outros commits na mesma issue, a análise pode estar **incompleta**; se o alvo for **PR**, o diff do PR cobre em geral a entrega — refletir lacunas **apenas** em **Alterações possíveis** ou **Pacotes impactados** se for relevante (sem tabela de commits na resposta).

Esta síntese **alimenta** o bloco **Alterações possíveis no código** e o refinamento de **Pacotes impactados**.

### 11. Validação final (interna → convergir para os quatro blocos)

Aplicar **sem checklist na saída**:

- Coerência com **AC** e com o **baseline** (passo **8**).
- **Sugestões objetivas** (mais limpo, seguro, performático, alinhado à arquitetura) → entram **só** em **Alterações possíveis no código**, na **lista numerada (opção A)** definida no formato obrigatório.
- **Segurança operacional**, **edge cases**, **performance**, **testes em falta**: se forem achados, **um item numerado por achado** (`N. **Título** — motivo`); **não** usar linhas *N/A* nem tabelas.

**Proibido** na mensagem ao utilizador: checklists, conclusões *Aprovar/Não aprovar*, frases de limitação do comando (ver **«O que não incluir»** acima).

---

## Cobertura obrigatória — lacunas fechadas

Cada bloco tem **Contexto** (porque existe), **Objetivo** (o que deve ganhar o utilizador) e **Regras obrigatórias** (o agente **não** pode ignorar).

### A — Lente QA (aceite, testes, testabilidade)

- **Contexto:** O nome do comando é **CODE_REVIEW_QA**; revisões só técnicas deixam fugir o que equipas QA esperam da leitura de código.
- **Objetivo:** Garantir **rastreabilidade** entre o que a issue pede e o que o diff mostra, e **sinalizar** falta de testes ou de pontos de verificação — **na saída**, só via **Alterações possíveis** ou **Pacotes impactados** (sem listar AC nem *N/A*).
- **Regras obrigatórias:**
  1. No passo **6**, obter **AC** da issue via MCP; se inexistentes, usar isso **internamente** (sem frase de limitação na resposta ao utilizador).
  2. **Não** concluir “coberto” sem **ligação objetiva** diff ↔ AC; lacunas → **uma sugestão objetiva** em **Alterações possíveis** (ex.: *falta cobrir o AC X porque Y não aparece no diff*).
  3. Falta de testes ou de observabilidade → **sugestão objetiva** em **Alterações possíveis** (risco para QA), **não** linha *N/A* em checklist.

### B — Correção e casos-limite

- **Contexto:** Checklists industriais tratam **edge cases** à parte; só “faz sentido?” deixa o agente saltar verificações.
- **Objetivo:** Forçar **pergunta explícita** sobre entradas e falhas quando o código **lida com dados externos, UI, I/O ou estado partilhado**.
- **Regras obrigatórias:**
  1. Se o diff tocar **validação de input**, **API**, **ficheiros**, **rede**, **async** ou **estado global**: no passo **9**, considerar **null/vazio**, **limites** ou **falha**; se houver lacuna objetiva, **Alterações possíveis** com motivo.
  2. Se **nenhum** desses eixos se aplica ao diff, **nada** a acrescentar por este bloco (sem declarar *N/A* na resposta).

### C — Performance

- **Contexto:** O comando original não pedia revisão de **custo** de execução; isso é comum em code review profissional.
- **Objetivo:** Detetar **riscos objetivos** (ex.: loops com I/O, queries em render, listas sem paginação).
- **Regras obrigatórias:**
  1. Se o diff tocar **consultas a BD**, **chamadas HTTP em série**, **renders/listas grandes** ou **loops**: avaliar risco; se existir, **Alterações possíveis** com motivo objetivo.
  2. **Não** alegar problemas de performance **sem** indício no diff ou no baseline (evitar fantasia).

### D — Segurança operacional (além de XSS / injection)

- **Contexto:** XSS/injection não cobrem **segredos**, **authz** nem **vazamento** em logs/erros.
- **Objetivo:** Espelhar itens que **SAST e políticas** costumam apanhar e que o LLM **pode** referenciar se visíveis no diff.
- **Regras obrigatórias:**
  1. Procurar no diff **credenciais, API keys, tokens** em claro → se suspeita, **Alterações possíveis** (remover/rotação), **sem** repetir segredo completo.
  2. Se o diff tocar **autenticação/autorização** e faltar verificação objetiva de permissão, **Alterações possíveis** com motivo.
  3. **Logs / erros** com dados sensíveis ou stack interna → **Alterações possíveis** se visível no diff.
  4. **Não** afirmar “está seguro” na resposta; **não** listar “não há problema” como meta-frase — só sugestões quando houver achado objetivo.

### E — Limite do agente (expectativa)

- **Contexto:** A revisão LLM **não** substitui CI/SAST/humano — mas isso **não** deve aparecer como texto na resposta ao utilizador (ver formato obrigatório).
- **Objetivo:** Expectativas corretas **sem** poluir a saída.
- **Regras obrigatórias:**
  1. **Não** incluir na resposta frases genéricas de limitação do comando.
  2. **Nunca** afirmar que *«o CI passou»* sem o utilizador o ter dito ou ferramenta com resultado; se relevante para uma sugestão, integrar **só** em **Alterações possíveis** (ex.: *validar com pipeline após alterar X*).

### F — Heurística merge / commit isolado (quando o alvo **não** é um PR)

- **Contexto:** Quando o fluxo cai no **commit** (sem PR ou exceção explícita), o último commit pode ser **só merge**; o trabalho funcional pode estar noutro hash da mesma issue. Quando o alvo é **PR**, o diff do PR é em geral o referencial correto e **não** se aplica o mesmo problema do “só diff do merge commit”.
- **Objetivo:** Evitar **falsa conclusão** de que o trabalho da tarefa foi todo revisto quando se rever **apenas** um commit de merge **sem** passar pelo PR.
- **Regras obrigatórias:**
  1. Se o alvo da revisão for **PR** (passos **4a–5**): tratar o diff do PR como cobertura principal da entrega; **não** sugerir “rever outro commit por ser merge” **só** por existir commit de merge no histórico da issue, salvo indício objetivo de que o PR não contém toda a entrega.
  2. Se o alvo for **commit** (fluxo **4b**): classificar a **mensagem** como **merge** se coincidir com padrões típicos: `Merge branch`, `Merge pull request`, `Merged in` (lista não exaustiva — julgamento conservador). Se for **merge** **e** existir **outro** commit na **mesma lista Jira** (ou PR não usado): **internamente** considerar que o diff desse commit pode **não** cobrir todo o trabalho; se aplicável, **uma** sugestão objetiva em **Alterações possíveis** ou **Pacotes impactados** (ex.: preferir o diff do PR ligado à issue, ou rever o commit funcional X). **Sem** listagem tabular de commits na resposta.
  3. **Não** emitir **conclusão** *Aprovar/Não aprovar* na saída; riscos de análise incompleta traduzem-se em **sugestão objetiva** nos blocos permitidos.

### G — Ferramentas determinísticas e CI

- **Contexto:** Boas práticas recomendam **camada** LLM + linter/SAST; o agente no Cursor **não** substitui o pipeline salvo integração explícita.
- **Objetivo:** Não **assumir** verde de CI.
- **Regras obrigatórias:**
  1. **Nunca** afirmar que *«o CI passou»* sem o utilizador o ter dito ou sem ferramenta com resultado.
  2. Se o utilizador disser que **CI falhou** ou **linter falhou**: incluir **Alterações possíveis** a corrigir o que o utilizador indicou, **sem** meta-frase sobre o que o comando não faz.

### H — Código agentico / alta autonomia (quando aplicável)

- **Contexto:** PRs que adicionam **agentes**, **ferramentas desconhecidas**, **execução automática** ou **chamadas a APIs externas** têm riscos extra (objetivos, permissões).
- **Objetivo:** Garantir **uma passagem** explícita quando o diff **claramente** toca esse domínio.
- **Regras obrigatórias:**
  1. Se paths ou texto do diff indicarem **agent**, **MCP**, **tool call**, **workflow autónomo**, **cron**, **webhook** ou equivalente: no passo **9**, avaliar permissões, dados e superfície de ataque; achados → **Alterações possíveis**.
  2. Se o diff **não** toca esse domínio, nada a acrescentar por **H** na resposta.

### I — Anti-falsos-positivos (acordo de equipa — Roboteasy Studio / depuração)

- **Contexto:** Sugestões genéricas de code review podem **repetir** pontos já **decididos** entre produto, QA e desenvolvimento, ou **ignorar** contratos da stack (gerador + runtime), gerando **ruído** e retrabalho.
- **Objetivo:** Só propor **Alterações possíveis** nestes domínios quando houver **evidência no diff**, **contradição com AC** explícitos na issue, ou **novo risco** (novo call site, regressão, política de repo).
- **Regras obrigatórias:**
  1. **Árvore de depuração (`PropertyTreeBuilder` — raízes «Componente» / «Projeto»):** **Não** sugerir como padrão «garantir sempre duas raízes» nem «sempre mostrar o nó Componente». Posicionamento acordado: quando `relevantVariableNames` **não** gera filhos, **omitir** a secção **Componente** e manter **Projeto** (expandido). Só levantar o tema em **Alterações possíveis** se os **critérios de aceite** da issue **exigirem** explicitamente duas secções **visíveis** nesse cenário — aí propor alinhar **implementação ou AC/testes**, não «corrigir» por suposição de UX.
  2. **Secção «Projeto» vazia / `(0 Properties)`:** **Não** sugerir esconder ou condicionar «Projeto» por medo de pasta vazia **sem** indício no diff ou no comportamento descrito: na arquitetura atual **existem variáveis de projeto** mesmo quando o projeto não declara propriedades — «Projeto» **não** fica vazio na prática. Só sugerir ajuste se o diff ou reprodução mostrar **efetivamente** estado vazio ou contagem zero indevida.
  3. **Testes unitários em lógica só de visualização:** **Não** listar como sugestão **obrigatória** testes para builders de árvore / **apresentação** (ex.: `PropertyTreeBuilder`) **só** por ser código novo ou «simples». Incluir testes em **Alterações possíveis** **só** quando: política explícita do repositório o exigir, houver **regra de negócio** no builder (não só montagem de nós), ou **histórico de regressão** nessa área. Caso contrário, omitir ou mencionar **opcional** apenas se o utilizador pedir foco em testabilidade.
  4. **Null em coleções garantidas pelo gerador (`relevantVariableNames` e similares):** **Não** sugerir iteração defensiva contra `null` **só** por prudência genérica quando o **contrato** é garantido pelo **código gerado** do frontend da mesma solução e o diff **não** introduz **novos** consumidores fora desse contrato. Se o diff **adicionar** call site **não** gerado, API pública ou camada partilhada **sem** garantia documentada, aí sim avaliar null/vazio em **B**.
  5. **Formatação de `.resx` e recursos (newline final, whitespace):** **Não** sugerir correção **manual** de newline ou formatação em `.resx` / `Messages.Designer.cs` quando a equipa usa **formatação automática do Visual Studio** e desencoraja edição manual — isso gera **ruído de diff** e pode ser revertido pelo IDE. **Exceção:** strings erradas, cultura em falta, ou chaves incorretas (conteúdo funcional), não estética de fim de ficheiro.

---

## Coerência — comando funcional (checklist de revisão do próprio `.md`)

Antes de enviar a mensagem, o agente **confirma internamente** que:

- Respeitou **gates**: tarefa encontrada; **alvo** definido — **um PR** (prioridade quando aplicável) **ou** **um commit** (o **mais recente** se houver vários no fluxo por commits); diff desse alvo ou fallback local disponível.
- A cadeia **Jira → Desenvolvimento → PR (se um) ou commits → diff no Bitbucket** foi respeitada ou a limitação foi dita.
- Os passos **8–11** e a cobertura **A–I** foram aplicados **no raciocínio** (incluindo filtrar sugestões à luz de **I**).
- A **resposta ao utilizador** contém **exatamente** os **quatro** blocos do **«Formato obrigatório da resposta ao utilizador»**, **sem** violar **«O que não incluir»**, e **sem** linhas **antes** de `Tarefa:` (nem depois do último bloco).
- O bloco **Alterações possíveis no código** segue a **opção A** (lista numerada após os dois-pontos, **ou** linha única quando não houver sugestões).

Se algum pré-requisito falhar (sem diff, sem repo e sem cola do utilizador), **não simular** revisão completa — **parar** e pedir o que falta (esta mensagem de pedido **não** precisa seguir os quatro blocos).
