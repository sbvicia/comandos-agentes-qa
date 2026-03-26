# qa-module

## EXEMPLO DE DISPARO (COMENTADO)

# /qa-module Excel
# ↑ "Excel" é o NOME DO MÓDULO ({{MODULO}})

# /qa-module Excel | Excel — window_excel_* (39 componentes: abrir, ler, escrever, formatar, CSV, etc)
# ↑ "Excel" = módulo
# ↑ Tudo após "|" = detalhes do módulo ({{DETALHES}})

# Regra:
# - O primeiro valor após /qa-module é sempre o {{MODULO}}
# - Tudo que vier após "|" será considerado como {{DETALHES}}

---

## DEFINIÇÃO DE MÓDULO

"MÓDULO" refere-se a um conjunto de componentes relacionados a uma funcionalidade específica dentro do sistema.

Um módulo pode representar:
- Um grupo de ações (ex: Excel, Email, Browser)
- Um conjunto de componentes técnicos (ex: window_excel_*)
- Uma funcionalidade do sistema (ex: Login, Pagamentos, API)

IMPORTANTE:
- Um módulo NÃO é apenas uma tela isolada
- Um módulo é composto por múltiplos componentes reutilizáveis
- Os componentes devem ser identificados dentro de:
  - front-studio
  - rpa-agent

Sempre considere variações de nomenclatura (prefixos, sufixos e padrões).

---

Extraia o {{MODULO}} do comando.

Se {{DETALHES}} não forem informados, solicite antes de continuar.

---

Você é um QA. Olhe para este projeto como um QA especializado em testes automatizados.

REGRAS OBRIGATÓRIAS:
- Nunca pule etapas
- Sempre siga a ordem dos passos
- O fluxo deve ir até o Passo 5 obrigatoriamente
- Não interromper a execução no meio do fluxo
- Sempre entregar todos os passos completos na resposta
- Se pular qualquer etapa, a resposta será inválida

---

## Passo 1 — Rastreamento de componentes

Rastreie dentro de:
- front-studio
- rpa-agent

Os módulos relacionados a: **{{MODULO}}**

Detalhes adicionais do módulo:
{{DETALHES}}

Objetivo:
- Identificar TODOS os componentes relacionados ao módulo
- Listar e numerar todos os componentes encontrados

---

## Passo 2 — Escrita de casos de teste (DOCUMENTAÇÃO)

Leia o padrão no arquivo:
TEST-CASES.md

ATENÇÃO:
- Neste passo, NÃO gerar código
- Apenas documentação

PADRÃO OBRIGATÓRIO DE COBERTURA:

- Gerar casos de teste INDIVIDUAIS para cada componente identificado
- Para cada componente, criar:
  - 1 cenário positivo
  - 1 cenário negativo

- NÃO agrupar cenários por tipo de ação
- NÃO resumir ou reduzir quantidade de casos
- É OBRIGATÓRIO exibir todos os casos de teste na resposta
- NÃO apenas salvar em arquivo

TIPO DE CENÁRIOS NEGATIVOS:

- Os cenários negativos devem focar em comportamento funcional do sistema (runtime)
- Exemplos:
  - caminho inválido
  - arquivo inexistente
  - permissões insuficientes
  - entrada inválida

- NÃO criar cenários negativos focados em implementação técnica
  (ex: validação de C#, métodos internos, codegen, etc)

NÃO FAZER PERGUNTAS:

- Não solicitar confirmação do usuário
- Não oferecer opções
- Executar diretamente seguindo as regras definidas


Use a seguinte feature APENAS como REFERÊNCIA de padrão de escrita:
Feature: Abrir navegador (`browse_open`)

IMPORTANTE:

- Essa feature NÃO deve ser replicada literalmente
- Ela serve apenas como EXEMPLO de estrutura
- Os casos devem ser baseados no módulo **{{MODULO}}**

- Os casos de teste DEVEM seguir padrão BDD:
  - Given (Dado)
  - When (Quando)
  - Then (Então)

- NÃO criar tabelas técnicas
- NÃO validar código C#
- NÃO focar em implementação

Os casos devem descrever:
- comportamento funcional
- entrada e saída esperada
- regras de negócio

---

## Passo 3 — Automação

OBJETIVO DESTE PASSO:
- Implementar testes automatizados com base nos casos de teste criados no Passo 2

IMPORTANTE:
- Neste passo, É OBRIGATÓRIO gerar código
- NÃO permanecer apenas em análise ou explicação

ANTES de implementar os testes:

- Cada teste automatizado deve ser derivado diretamente de um cenário BDD do Passo 2
- NÃO criar testes genéricos, estruturais ou de codegen
- NÃO abstrair os testes em validações indiretas
- Analisar o projeto `genesis-qa`
- Identificar como os testes já estão estruturados
- Entender:
  - padrão de organização dos arquivos
  - convenção de nomes
  - estrutura dos testes
  - uso de mocks, stubs ou helpers
  - bibliotecas/frameworks utilizados

- Seguir EXATAMENTE o padrão existente (NÃO inventar novo padrão)

---

Implementação:

- Criar os testes seguindo fielmente o padrão identificado no projeto
- Manter consistência com os testes já existentes
- Garantir código limpo e reutilizável
- Garantir correspondência direta entre:
  - caso de teste (Passo 2)
  - teste automatizado

- Utilizar mocks/stubs conforme padrão já utilizado no projeto

---

## Passo 4 — Execução

- Rodar os testes
- Identificar falhas
- Corrigir problemas
- Reexecutar até 100% sucesso

---

## Passo 5 — Validação final

- Garantir que TODOS os componentes de **{{MODULO}}** foram cobertos
- Verificar se não há gaps de teste
- Validar consistência entre:
  - Casos de teste
  - Testes automatizados
- Validar que a quantidade de testes corresponde a:
  (número de componentes x 2 cenários)
- Garantir que não existem testes duplicados
- Garantir que todos os cenários possuem assertivas válidas