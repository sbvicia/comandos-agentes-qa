# atualizar-docu

## CONFIGURAÇÃO (editar uma vez neste ficheiro)

| Chave | Valor |
|--------|--------|
| `SPACE_ID_GITBOOK` | ID do space no GitBook onde a doc **oficial para utilizador final** vive (ex.: Genesis **hml** = `bjbLJiYKvnlRGJBmNIt9`). Usar nas ferramentas MCP quando o utilizador **não** indicar outro `spaceId`. |
| `SPACE_ID_REFERENCIA_STUDIO` | Opcional: space **Studio** `ABFcMeFxBJhKzVOwJbRE` — usar só para **copiar tom/estrutura** de páginas homólogas quando o space Genesis ainda não tiver conteúdo naquele tópico. |
| `RAIZ_CORPUS_LOCAL` | `documentacao-genesis/` — **consulta** do que já existe no repo (exportes, índices, notas). **Não** é o destino obrigatório da saída. |

**MCP:** servidor **`user-gitbook`** — ferramentas típicas: `get_page_by_path` (`pagePath` + `spaceId`), `get_page_content` (`pageId` + `format`: `markdown`), `search_content` (`query` + `spaceId`). Se o MCP **não** estiver disponível: avisar e usar só o corpus local + instrução do utilizador, com ressalva de que o texto **não** foi confrontado com o GitBook ao vivo.

**Publicação:** o utilizador cola o resultado **manualmente** no GitBook. Este comando **não** grava no GitBook via API.

---

## EXEMPLO DE DISPARO (COMENTADO)

```text
# /atualizar-docu
# ou: /atualizar docu:
#
# Objetivo: gerar texto de documentação para UTILIZADOR FINAL, no padrão GitBook,
# para colar manualmente no editor, depois de confrontar com a página atual no GitBook.
#
# Blocos típicos:
#   {{INSTRUCAO}}  — o que queres alterar, acrescentar ou esclarecer (texto bruto)
#   {{ALVO}}       — página no GitBook (path) e/ou ficheiro no corpus local
#
# Exemplo A — página GitBook por path (recomendado):
# ---
# Explicar em duas frases como guardar o projeto e onde fica o ficheiro .roboteasy.
# ---
# GITBOOK_PATH: page
# (ou: introducao, geral/cadastro-e-login, etc. — mesmo estilo de slug/path da URL app.gitbook.com)
#
# Exemplo B — etiquetas explícitas:
# INSTRUÇÃO: Incluir passos numerados para abrir o painel de componentes.
# GITBOOK_PATH: geral/iniciar-um-projeto
# SPACE_ID: bjbLJiYKvnlRGJBmNIt9
#
# Exemplo C — corpus local como apoio (opcional):
# INSTRUÇÃO: Alinhar o glossário com a lista de objetos do Genesis.
# GITBOOK_PATH: page
# CORPUS: INICIO.md, COMPONENTES_ARVORE_STUDIO.md
#
# Se GITBOOK_PATH estiver em falta: usar search_content no space para localizar página
# pelo tema da INSTRUÇÃO; se continuar ambíguo, pedir o path ao utilizador e parar.
```

**Regras de parsing:**

- `INSTRUÇÃO:` / `GITBOOK_PATH:` / `SPACE_ID:` / `CORPUS:` — quando presentes, têm prioridade.
- Sem etiquetas: primeiro bloco substantivo = {{INSTRUCAO}}; linha final que pareça path GitBook (`algo/outro`) ou nome `*.md` em `documentacao-genesis/` = {{ALVO}}.
- `GITBOOK_PATH`: sem barra inicial; exemplos `page`, `geral/componentes`, `geral/componentes/arquivos`.
- `CORPUS:` lista opcional de ficheiros sob `RAIZ_CORPUS_LOCAL` para contexto interno (não substitui a leitura da página no GitBook).

---

## Objetivo

1. Produzir **documentação formal para utilizador final** em **português**, pronta a **copiar para o GitBook**.  
2. **Consultar o GitBook** (MCP) para obter o **conteúdo atual** da página alvo em **markdown** quando possível, e **alinhar** a alteração pedida em {{INSTRUCAO}} com o que **já está publicado** (estrutura de títulos, listas, tabelas, tom).  
3. **Consultar** `documentacao-genesis/` como **corpus** do que a equipa já tem no repo (exports, índices, mapeamentos), **sem** assumir que o GitBook está idêntico ao disco.  
4. **Entregar** na resposta o **texto completo** da secção ou da página **já formatado** no padrão da documentação existente no GitBook (ver **Padrão GitBook** abaixo), **não** substituir o fluxo manual de publicação do utilizador.

---

## REGRAS OBRIGATÓRIAS

### Fase A — GitBook (MCP)

- Com `GITBOOK_PATH` e `SPACE_ID` (ou default **SPACE_ID_GITBOOK**): chamar **`get_page_by_path`** com `pagePath` e `spaceId`.  
- Se a resposta trouxer `pageId`, usar **`get_page_content`** com `format`: **`markdown`** para análise fina do corpo.  
- Se `get_page_by_path` falhar (404, path errado): tentar **`search_content`** com termos da {{INSTRUCAO}}; se ainda assim não houver página clara, **pedir o path correto** e parar.  
- **Comparar** mentalmente o pedido do utilizador com o markdown obtido: o output final deve ser **coerente** com títulos (`#`, `##`), listas, tabelas e **voz** (tu/você, impessoal) já usados nessa página.

### Fase B — Corpus local

- Ler ficheiros em `documentacao-genesis/` **só** quando {{INSTRUCAO}} ou `CORPUS:` apontar para eles, ou quando forem necessários para factos (listas de componentes, etc.).  
- O corpus **não** sobrepõe o GitBook: se houver conflito, **preferir** o GitBook como fonte da “versão oficial” em linha e indicar divergências numa nota curta ao utilizador se for relevante.

### Fase C — Texto para utilizador final

- Linguagem **clara**, orientada a **tarefas** e **resultados**, sem jargão de repo (`src/`, `npm`) salvo quando a doc já o faça para admins.  
- **Não inventar** funcionalidades que não estejam em {{INSTRUCAO}}, no conteúdo GitBook lido ou no corpus / código quando o utilizador citar explicitamente o código.  
- Incluir na resposta:
  - **Bloco principal:** markdown pronto para colar no GitBook (página completa **ou** secção a substituir, consoante o pedido — deixar isso explícito no título do bloco).  
  - **Resumo** em 2–4 bullets: o que mudou face ao que estava no GitBook.  
  - **Onde colar:** path GitBook + nome amigável da página.

### Padrão GitBook (imitar o que vier da página lida)

- Replicar **nível de cabeçalhos** e ritmo de parágrafos da página atual.  
- Se as páginas Roboteasy usarem blocos GitBook (`{% hint %}`, `{% stepper %}`, `{% content-ref %}`), **reutilizar o mesmo padrão** quando o markdown devolvido pela API os incluir; se o markdown vier “plano”, manter **markdown compatível** com GitBook (títulos, listas, links `https://docs.roboteasy.tech/...` quando fizer sentido).  
- Tabelas e listas numeradas para passos, quando a página modelo o faça.

### Fase D — Gravar no repo (opcional)

- **Só** alterar ficheiros em `documentacao-genesis/` se o utilizador pedir **explicitamente** nesta conversa (ex.: “guarda também um rascunho em INICIO.md”). Caso contrário, **não** é obrigatório escrever no disco.

### Segurança

- Não incluir tokens, credenciais ou dados pessoais no texto para o GitBook.

---

## Execução (ordem)

1. Extrair {{INSTRUCAO}}, `GITBOOK_PATH`, `SPACE_ID`, `CORPUS:` conforme **EXEMPLO DE DISPARO**.  
2. **Fase A:** obter conteúdo atual da página no GitBook (MCP).  
3. **Fase B:** ler corpus local se aplicável.  
4. **Fase C:** gerar markdown **final para utilizador**, alinhado ao padrão observado no GitBook + {{INSTRUCAO}}.  
5. **Fase D:** gravar em `documentacao-genesis/` só se pedido.  
6. Resposta: texto para colar + resumo + path da página.

---

## Referência de qualidade

- Tom semelhante à doc pública Roboteasy (ex.: [Studio — componentes](https://docs.roboteasy.tech/studio/geral/componentes)): directo, PT-BR, secções curtas.  
- O comando **atualizar-docu** complementa o **bug-report-roboteasy** (interno/Jira); este foca **manual de utilizador** e **GitBook**.
