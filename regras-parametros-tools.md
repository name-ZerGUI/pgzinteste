# 🎯 Framework de Regras para Parâmetros de Tools

Sistema universal de regras para configuração de parâmetros de tools em qualquer integração.

## 📚 **Estrutura do Documento**

Este framework contém dois tipos de regras:

### **🎨 Regras de UX/Configuração (1-9)**
**Para quem:** Usuário final configurando tools no checkpoint  
**Quando:** Durante setup/configuração da tool  
**Impacto:** Interface, interação, decisões do usuário

### **⚙️ Regras de Schema/Sistema (10-12)**
**Para quem:** Desenvolvedor definindo schema da tool  
**Quando:** Durante implementação/mapeamento da integração  
**Impacto:** Validação, processamento, comportamento interno

> 💡 **O usuário não vê as Regras 10-12 diretamente**. Elas são aplicadas automaticamente pelo sistema baseado no schema da tool.

---

## 🌳 **Árvore de Decisão: Guia Completo das 9 Regras de UX**

Esta árvore guia desenvolvedores e usuários através de todas as decisões necessárias para configurar parâmetros de tools corretamente.

### **🎯 Para Desenvolvedores: Definindo o Schema da Tool**

```
┌─────────────────────────────────────────────────────────────────┐
│ INÍCIO: Criando uma nova Tool ou Parâmetro                     │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ▼
    ┌────────────────────────────────────────┐
    │ 1️⃣ Esta tool deve aparecer na lista @? │ (REGRA 1)
    └────────┬───────────────────────┬────────┘
             │                       │
         NÃO │                       │ SIM
             │                       │
             ▼                       ▼
    ┌────────────────────┐  ┌──────────────────────────┐
    │ OCULTAR da lista @ │  │ MOSTRAR na lista @       │
    │                    │  │                          │
    │ ✓ Prefixo: get*    │  │ ✓ Prefixo: create/update │
    │ ✓ Consulta pura    │  │ ✓ Ação/Modificação       │
    │ ✓ Suporte para LLM │  │ ✓ getOrCreate (híbrida)  │
    │                    │  │                          │
    │ Invocação:         │  │ Invocação:               │
    │ → Automática       │  │ → Explícita pelo usuário │
    │ → Por dependência  │  │                          │
    └────────────────────┘  └──────────┬───────────────┘
                                       │
                                       ▼
                    ┌──────────────────────────────────────┐
                    │ 2️⃣ Configurar parâmetros da tool:   │
                    │    Para cada parâmetro...            │
                    └──────────┬───────────────────────────┘
                               │
                               ▼
              ┌────────────────────────────────────────┐
              │ 3️⃣ Este parâmetro deve ser visível    │ (REGRA 2)
              │    para o usuário configurar?          │
              └────┬───────────────────────────┬────────┘
                   │                           │
               NÃO │                           │ SIM
                   │                           │
                   ▼                           ▼
    ┌──────────────────────────┐  ┌────────────────────────────┐
    │ OCULTAR parâmetro        │  │ MOSTRAR parâmetro          │
    │                          │  │                            │
    │ Quando ocultar:          │  │ Quando mostrar:            │
    │ ✓ Dependência única      │  │ ✓ Dependência com array    │
    │   (valor singular)       │  │   (múltiplas opções)       │
    │ ✓ Parâmetro técnico      │  │ ✓ Decisão de negócio       │
    │ ✓ Valor padrão óbvio     │  │ ✓ Conteúdo customizável    │
    │                          │  │ ✓ Regras variáveis         │
    │ Exemplo:                 │  │                            │
    │ person_id (vem de        │  │ Exemplo:                   │
    │ @getOrCreatePerson que   │  │ deal_id (vem de array,     │
    │ retorna 1 pessoa)        │  │ precisa critério seleção)  │
    └──────────────────────────┘  └────────┬───────────────────┘
                                           │
                                           ▼
                        ┌──────────────────────────────────────┐
                        │ 4️⃣ Quais tipos de preenchimento     │ (REGRA 3)
                        │    permitir?                         │
                        └──────┬───────────────────────────────┘
                               │
                ┌──────────────┼──────────────┐
                │              │              │
                ▼              ▼              ▼
    ┌────────────────┐ ┌──────────────┐ ┌──────────────────┐
    │ FIXO           │ │ LLM PROMPT   │ │ DEPENDÊNCIA      │
    │                │ │              │ │                  │
    │ Usuário define │ │ LLM decide   │ │ Vem de outra tool│
    │ valores        │ │ dinamicamente│ │                  │
    │ permitidos     │ │              │ │ Obrigatório para:│
    │                │ │ Tool suporte │ │ → IDs de array   │
    │ Quando usar:   │ │ provê contexto│ │ → Valor anterior │
    │ → IDs tabelados│ │              │ │                  │
    │ → Enum fechado │ │ Quando usar: │ │ Quando usar:     │
    │ → Padrões fixos│ │ → Texto livre│ │ → Pipeline serial│
    │                │ │ → Contexto   │ │ → Relação forte  │
    └────────┬───────┘ └──────┬───────┘ └────────┬─────────┘
             │                │                   │
             │                │                   │
             ▼                ▼                   ▼
    ┌────────────────┐ ┌──────────────┐ ┌──────────────────┐
    │ 5️⃣ Multi-select│ │ Enum Values? │ │ Preview de campos│
    │ permitido?     │ │ (opcional)   │ │ disponíveis?     │
    │ (REGRA 4)      │ │              │ │ (REGRA 5E)       │
    └────┬───────────┘ └──────────────┘ └─────────┬────────┘
         │                                         │
         ▼                                         ▼
    SE SIM:                               ┌──────────────────┐
    → Campo instrução                     │ SE Dependência OU│
      OBRIGATÓRIO                         │ LLM com support: │
    → Validação impede                    │                  │
      salvar sem instrução                │ Mostrar preview: │
                                          │ • id             │
                                          │ • status         │
                                          │ • value          │
                                          │ • created_at     │
                                          │                  │
                                          │ Campo LIVRE para │
                                          │ critério/instrução│
                                          └──────────────────┘
                               │
                               ▼
              ┌────────────────────────────────────────┐
              │ 6️⃣ Definir dependências e validações  │ (REGRA 5)
              └────┬───────────────────────────────────┘
                   │
        ┌──────────┼──────────┬─────────────┐
        │          │          │             │
        ▼          ▼          ▼             ▼
    ┌──────┐  ┌───────┐  ┌────────┐  ┌──────────┐
    │ Tool │  │ Tool  │  │ Params │  │ Tool     │
    │ obri-│  │ condi-│  │ relacio│  │ suporte  │
    │ gatór│  │ cional│  │ nados  │  │ (info)   │
    │ ia   │  │       │  │        │  │          │
    │      │  │       │  │        │  │          │
    │ 🔴   │  │ Pode  │  │ stage  │  │ ℹ️ LLM   │
    │ Erro │  │ usar  │  │ depende│  │ terá     │
    │ se   │  │ depen-│  │ de     │  │ contexto │
    │ não  │  │ dência│  │ pipeline│ │ via tool │
    │ ativa│  │ ou não│  │        │  │          │
    └──────┘  └───────┘  └────────┘  └──────────┘
                   │
                   ▼
              ┌────────────────────────────────┐
              │ 7️⃣ Definir obrigatoriedade    │ (REGRA 7)
              └────┬───────────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
        ▼                     ▼
    ┌──────────┐      ┌──────────────┐
    │ REQUIRED │      │ OPTIONAL     │
    │          │      │              │
    │ Badge *  │      │ Toggle       │
    │ Validação│      │ Colapsável   │
    │ bloqueio │      │ Pode omitir  │
    └──────────┘      └──────────────┘
                   │
                   ▼
              ┌────────────────────────────────┐
              │ 8️⃣ Configurar instruções       │ (REGRA 8)
              └────┬───────────────────────────┘
                   │
        ┌──────────┴───────────┐
        │                      │
        ▼                      ▼
    ┌─────────────┐    ┌──────────────┐
    │ Por         │    │ Gerais       │
    │ Parâmetro   │    │ (da tool)    │
    │             │    │              │
    │ Específica  │    │ Contexto     │
    │ do campo    │    │ geral        │
    │             │    │ execução     │
    │ Ex:         │    │              │
    │ "Extrair    │    │ Ex:          │
    │ título da   │    │ "Cliente VIP,│
    │ conversa"   │    │ priorizar"   │
    └─────────────┘    └──────────────┘
                   │
                   ▼
              ┌────────────────────────────────┐
              │ 9️⃣ Documentar no schema JSON   │ (REGRA 9)
              └────┬───────────────────────────┘
                   │
                   ▼
    ┌──────────────────────────────────────────┐
    │ Schema completo com:                     │
    │ • name, display_name, type               │
    │ • required, visible                      │
    │ • allowed_input_types                    │
    │ • fixed_config (multi_select, api, etc)  │
    │ • llm_config (support_tool, enum, etc)   │
    │ • dependencies                           │
    │ • parameter_relationships                │
    │ • validation                             │
    │ • criticality (Regra 10 - Sistema)       │
    │ • nullable_behavior (Regra 11 - Sistema) │
    └──────────────────────────────────────────┘
```

### **👤 Para Usuários: Configurando Tool no Checkpoint**

```
┌─────────────────────────────────────────────────────────────────┐
│ INÍCIO: Usuário seleciona @tool da lista                       │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ▼
    ┌────────────────────────────────────────┐
    │ Modal abre com parâmetros visíveis     │
    │ (apenas os que passaram pela Regra 2)  │
    └────────────────┬───────────────────────┘
                     │
                     ▼
        ┌────────────────────────────────┐
        │ Para cada PARÂMETRO visível:   │
        └────────┬───────────────────────┘
                 │
                 ▼
    ┌─────────────────────────────────────────┐
    │ 1️⃣ Escolher tipo de preenchimento:      │ (REGRA 3 + 6)
    └────┬────────────────────────┬───────────┘
         │                        │
         ▼                        ▼
    ┌──────────┐            ┌──────────┐
    │ FIXO     │            │ LLM      │
    └────┬─────┘            └────┬─────┘
         │                       │
         ▼                       ▼
    ┌──────────────────┐   ┌──────────────────┐
    │ UI mostra:       │   │ UI mostra:       │
    │                  │   │                  │
    │ ☑ Lista valores  │   │ ✎ Text area      │
    │   disponíveis    │   │   instrução      │
    │                  │   │                  │
    │ Multi-select?    │   │ [+] Enum Values  │
    │ → SE SIM:        │   │     (opcional)   │
    │   ⚠️ Campo       │   │                  │
    │   instrução      │   │ ℹ️ Tool suporte  │
    │   OBRIGATÓRIO    │   │   será chamada   │
    │                  │   │   em runtime     │
    └────┬─────────────┘   └────┬─────────────┘
         │                      │
         └──────────┬───────────┘
                    │
                    ▼
    ┌───────────────────────────────────────┐
    │ 2️⃣ SE parâmetro tem DEPENDÊNCIA:     │ (REGRA 5D)
    └────┬──────────────────────────────────┘
         │
         ▼
    ┌──────────────────────────────────────┐
    │ Tipo = Dependência OBRIGATÓRIO       │
    │ (sem opção Fixo/LLM)                 │
    │                                      │
    │ ℹ️ Preview de campos disponíveis:   │
    │    • id (number)                     │
    │    • status (enum): open, won, lost  │
    │    • value (number)                  │
    │    • created_at (datetime)           │
    │                                      │
    │ Critério para seleção:               │
    │ ┌────────────────────────────────┐   │
    │ │ Deal com status "open" e valor │   │
    │ │ maior que 1000                 │   │
    │ └────────────────────────────────┘   │
    │                                      │
    │ 💡 Use os campos disponíveis acima  │
    │    para criar sua instrução livre    │
    └──────────────────────────────────────┘
                    │
                    ▼
    ┌───────────────────────────────────────┐
    │ 2.1️⃣ SE tipo LLM com tool suporte:   │ (REGRA 5E)
    └────┬──────────────────────────────────┘
         │
         ▼
    ┌──────────────────────────────────────┐
    │ Também mostra preview de campos!     │
    │                                      │
    │ ℹ️ Tool de suporte:                 │
    │    @getAllExistingPipelines          │
    │                                      │
    │ ℹ️ Campos disponíveis:              │
    │    • id (number)                     │
    │    • name (string)                   │
    │    • stages (array)                  │
    │                                      │
    │ Instrução para LLM:                  │
    │ ┌────────────────────────────────┐   │
    │ │ Usar pipeline que contenha     │   │
    │ │ "VIP" no campo name            │   │
    │ └────────────────────────────────┘   │
    │                                      │
    │ 💡 Você pode usar: id, name, stages │
    └──────────────────────────────────────┘
                    │
                    ▼
    ┌───────────────────────────────────────┐
    │ 3️⃣ Validações automáticas:           │ (REGRA 7)
    └────┬──────────────────────────────────┘
         │
         ▼
    ┌──────────────────────────────────────┐
    │ • Campos obrigatórios (*) preenchidos│
    │ • Multi-select tem instrução         │
    │ • Tool dependente está ativa         │
    │ • Parâmetros relacionados válidos    │
    │                                      │
    │ ✅ Tudo OK → Botão "Salvar" ativo   │
    │ ❌ Falta algo → Botão desabilitado  │
    └──────────────────────────────────────┘
                    │
                    ▼
    ┌───────────────────────────────────────┐
    │ 4️⃣ Instruções gerais (opcional):     │ (REGRA 8)
    └────┬──────────────────────────────────┘
         │
         ▼
    ┌──────────────────────────────────────┐
    │ Text area no final do modal:         │
    │                                      │
    │ "Contexto geral para esta tool"     │
    │                                      │
    │ Ex: "Cliente é VIP, priorizar"       │
    │     "Criar apenas se valor > 1000"   │
    └──────────────────────────────────────┘
                    │
                    ▼
    ┌───────────────────────────────────────┐
    │ SALVAR CONFIGURAÇÃO                   │
    │ → Tool ativa no checkpoint            │
    │ → Pronta para execução em runtime     │
    └───────────────────────────────────────┘
```

### **⚡ Fluxo de Runtime (Execução)**

```
┌─────────────────────────────────────────┐
│ Checkpoint é acionado na conversa       │
└────────────────┬────────────────────────┘
                 │
                 ▼
    ┌────────────────────────────────┐
    │ Sistema carrega tools ativas   │
    └────────┬───────────────────────┘
             │
             ▼
    ┌────────────────────────────────┐
    │ Para cada parâmetro tipo LLM:  │
    │ → Chamar tool de suporte       │
    │ → Injetar contexto             │
    └────────┬───────────────────────┘
             │
             ▼
    ┌────────────────────────────────┐
    │ Para dependências:             │
    │ → Invocar tool GET automática  │
    │ → SE array: aplicar critério   │
    │ → SE único: usar valor direto  │
    └────────┬───────────────────────┘
             │
             ▼
    ┌────────────────────────────────┐
    │ LLM processa com:              │
    │ • Contexto da conversa         │
    │ • Dados de tools suporte       │
    │ • Instruções do usuário        │
    │ • Valores fixos permitidos     │
    └────────┬───────────────────────┘
             │
             ▼
    ┌────────────────────────────────┐
    │ Validações finais:             │
    │ • Normalização (campo crítico) │
    │ • Null handling                │
    │ • Tipo de dados correto        │
    └────────┬───────────────────────┘
             │
             ▼
    ┌────────────────────────────────┐
    │ Executar tool na API externa   │
    │ ✅ Sucesso                      │
    └────────────────────────────────┘
```

### **🎯 Decisões Rápidas por Cenário**

| Cenário | Regra Principal | Decisão |
|---------|----------------|---------|
| Tool começa com `get` sem side effects | 1 | ❌ Não mostrar na lista @ |
| Tool `create`, `update`, `delete` | 1 | ✅ Mostrar na lista @ |
| Parâmetro vem de tool que retorna 1 valor | 2 | ❌ Ocultar parâmetro |
| Parâmetro vem de tool que retorna array | 2, 5D | ✅ Mostrar com campo livre + preview |
| ID que precisa contexto dinâmico | 3, 5E | LLM + tool suporte + preview campos |
| ID com valores tabelados conhecidos | 3 | Fixo + lista carregada |
| Usuário seleciona múltiplos valores fixos | 4 | Campo instrução obrigatório |
| Parâmetro obrigatório na API | 7 | Badge * + validação |
| Texto livre gerado pela LLM | 3, 8 | LLM Prompt + instrução específica |
| Dependência com array | 5D | Campo LIVRE (não opções pré-definidas) |
| LLM com support tool | 5E | Preview de campos disponíveis |

---

## 👁️ **Regra 1: Visibilidade de Tools no Checkpoint**

Define quais tools aparecem na lista `@` para o usuário configurar explicitamente.

### **Critérios para OCULTAR tool da lista @:**

**A. Tools de Consulta/Leitura (Padrão GET)**
- ✅ Tools que começam com `get` ou `getAll`
- ✅ Apenas leem dados, não modificam
- ✅ São invocadas automaticamente como dependências

**Exemplos que NÃO aparecem:**
- `@getAllExistingDealsFromPerson` → invocada quando @updateDeal precisa de deal_id
- `@getDealWithCompleteInfo` → invocada quando @updateDeal precisa de dados completos
- `@getAllExistingPipelines` → invocada quando tipo=LLM em pipeline_id
- `@getActivitiesFromDeal` → invocada quando outra tool precisa de atividades

**Como funciona a invocação automática:**

```
Usuário configura apenas @updateDeal:

┌────────────────────────────────┐
│ @updateDeal                    │
│                                │
│ #deal_id - tipo: Dependência   │
│                                │
│ Fonte: Consultar deals         │
│ ┌────────────────────────────┐ │
│ │ Critério para seleção:     │ │
│ │ ┌────────────────────────┐ │ │
│ │ │ Deal com status open   │ │ │
│ │ │ e valor > 1000         │ │ │
│ │ └────────────────────────┘ │ │
│ └────────────────────────────┘ │
└────────────────────────────────┘

Em runtime:
1. Sistema invoca @getAllExistingDealsFromPerson
2. Retorna: [deal1, deal2, deal3]
3. LLM aplica critério: "status open e valor > 1000"
4. Seleciona: deal2
5. @updateDeal usa deal_id=2

✅ Usuário não viu @getAllExistingDealsFromPerson
✅ Invocação foi automática
✅ Configuração de seleção fica onde faz sentido
```

**B. Tools de Suporte para LLM**
- ✅ Fornecem contexto para LLM em runtime
- ✅ Chamadas automaticamente quando parâmetro tipo=LLM
- ✅ Exemplo: `@getAllExistingPipelines` para popular contexto

### **Critérios para MOSTRAR tool na lista @:**

**A. Tools de Ação/Modificação**
- ✅ Criam, atualizam ou deletam dados
- ✅ Verbos: `create`, `update`, `delete`, `add`, `remove`
- ✅ Usuário DEVE configurar explicitamente

**Exemplos:**
- `@createDeal` → aparece
- `@updateDeal` → aparece
- `@createNote` → aparece
- `@createDealActivity` → aparece

**B. Tools Híbridas (GET com Efeito Colateral)**
- ✅ Começam com `get` MAS podem modificar dados
- ✅ Exemplo: `@getOrCreatePerson` → cria se não existir
- ✅ Usuário DEVE configurar porque tem impacto

### **Padrão de Naming como Indicador:**

| Prefixo | Visível? | Categoria | Invocação |
|---------|----------|-----------|-----------|
| `get`, `getAll` | ❌ NÃO | Consulta Auto | Por dependência |
| `create`, `add` | ✅ SIM | Ação | Explícita |
| `update`, `edit` | ✅ SIM | Ação | Explícita |
| `delete`, `remove` | ✅ SIM | Ação | Explícita |
| `getOrCreate` | ✅ SIM | Híbrida | Explícita |

### **Exceções ao Padrão GET:**

```
SE tool começa com "get" E:
  → Cria recursos (getOrCreate) → MOSTRAR
  → Tem side effects → MOSTRAR
  → Modifica estado → MOSTRAR

SENÃO:
  → OCULTAR e invocar automaticamente
```

### **Schema da Tool:**

```json
{
  "tool": "@getAllExistingDealsFromPerson",
  "naming_pattern": "getAll",
  "category": "query_auto",
  "visible_in_checkpoint": false,
  "auto_invoke": true,
  "invoked_by": [
    {
      "tool": "@updateDeal",
      "parameter": "deal_id",
      "when": "type=dependency"
    }
  ]
}
```

---

## 📐 **Regra 2: Visibilidade do Parâmetro**

### Critérios para **OCULTAR** parâmetro do usuário:

**A. Parâmetro de Dependência Única (Valor Singular)**
- ✅ Ocultar quando o valor vem de tool que retorna **UM único resultado**
- ✅ Não há decisão ou critério de seleção a ser configurado
- ✅ Sistema resolve automaticamente

**Exemplos:**
- `person_id` no `@createDeal` → vem de `@getOrCreatePerson`
  - Tool retorna: 1 pessoa
  - Não há escolha a fazer
  - ✅ **OCULTAR**

- `note_id` no `@updateNote` → vem de `@createNote`
  - Tool retorna: 1 nota criada
  - Não há escolha a fazer
  - ✅ **OCULTAR**

**Contra-exemplos (NÃO ocultar):**
- `deal_id` no `@updateDeal` → vem de `@getAllExistingDealsFromPerson`
  - Tool retorna: **ARRAY** [deal1, deal2, deal3]
  - Usuário PRECISA definir critério: "último", "status open", etc
  - ❌ **MOSTRAR** como tipo Dependência obrigatório

### **Comparação Visual:**

```
┌─────────────────────────────────────────────────────────────┐
│ CASO 1: Valor Único - OCULTAR parâmetro                    │
├─────────────────────────────────────────────────────────────┤
│ Tool: @createDeal                                           │
│ Parâmetro: person_id                                        │
│                                                              │
│ Dependência:                                                │
│ → @getOrCreatePerson                                        │
│ → Retorna: { id: 123, name: "João" }  ← ÚNICO VALOR        │
│                                                              │
│ Decisão: ✅ OCULTAR                                         │
│ Razão: Não há escolha a fazer, sempre usa o valor único    │
│                                                              │
│ UI: Usuário não vê person_id                                │
│     Sistema resolve automaticamente                         │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ CASO 2: Array de Valores - MOSTRAR parâmetro               │
├─────────────────────────────────────────────────────────────┤
│ Tool: @updateDeal                                           │
│ Parâmetro: deal_id                                          │
│                                                              │
│ Dependência:                                                │
│ → @getAllExistingDealsFromPerson                            │
│ → Retorna: [                            ← ARRAY (múltiplos) │
│      {id: 1, status: "open", value: 500},                   │
│      {id: 2, status: "won", value: 2000},                   │
│      {id: 3, status: "open", value: 1500}                   │
│    ]                                                         │
│                                                              │
│ Decisão: ❌ MOSTRAR (tipo Dependência obrigatório)          │
│ Razão: Usuário PRECISA escolher qual deal usar              │
│                                                              │
│ UI: Mostra deal_id com:                                     │
│     - Tipo: Dependência (único tipo permitido)              │
│     - Preview de campos disponíveis                          │
│     - Opções de seleção (primeiro/último/critério)          │
│     - Campo de instrução se critério personalizado           │
└─────────────────────────────────────────────────────────────┘
```

**B. Parâmetro Técnico/Sistema**
- ✅ Ocultar parâmetros puramente técnicos sem valor de configuração
- Exemplos:
  - `get: true` no `@getAllExistingPipelines` → sempre true, sem sentido expor
  - Flags booleanas fixas (ex: `busy: true` no createDealActivity)

**C. Parâmetros com Valor Padrão Inteligente**
- ✅ Ocultar se houver valor padrão que funciona em 95%+ dos casos
- ⚠️ MAS: permitir modo avançado para expor se necessário

### Critérios para **MOSTRAR** parâmetro:

**A. Requer Decisão de Negócio**
- ✅ Sempre mostrar quando usuário precisa escolher estratégia/comportamento
- Exemplos:
  - `pipeline_id` → qual funil usar?
  - `stage_id` → qual estágio inicial?
  - `currency` → qual moeda?

**B. Conteúdo Customizável**
- ✅ Sempre mostrar campos de texto/conteúdo que a LLM deve gerar
- Exemplos:
  - `title`, `content`, `subject`, `description`
  - Usuário define **como** a LLM deve gerar

**C. Regras de Negócio Variáveis**
- ✅ Mostrar quando o comportamento muda conforme contexto de negócio
- Exemplos:
  - `value` (valor do deal) → pode ter regra fixa ou dinâmica
  - `expected_close_date` → pode seguir padrão da empresa

---

## 🎚️ **Regra 3: Tipos de Preenchimento Permitidos**

### **Matriz de Decisão:**

| Tipo de Dado | Fixo | LLM Prompt | Dependência | Justificativa |
|--------------|------|-----------|-------------|---------------|
| **ID singular** (person_id quando única fonte) | ❌ | ❌ | ✅ ONLY | Dependência OBRIGATÓRIA. Ocultar parâmetro. Valor único sem escolha. |
| **ID com múltiplas opções** (deal_id, pipeline_id) | ✅ | ✅ | ✅ | Fixo: lista carregada. LLM: contexto dinâmico. Dependência: tool GET com seleção |
| **Texto livre** (title, content, description) | ❌ | ✅ | ❌ | Sempre gerado dinamicamente. Fixo só se for template literal |
| **Enum fechado** (status, currency) | ✅ | ✅ | ❌ | Fixo: lista hardcoded. LLM: interpreta texto ("moeda brasileira" → BRL) |
| **Data/Hora** | ✅ | ✅ | ❌ | Fixo para horários padrão, LLM para "daqui 3 dias às 14h" |
| **Valor numérico** (value, probability) | ✅ | ✅ | ❌ | Fixo para valores tabelados, LLM para extrair da conversa |
| **Array de objetos** (attendees) | ✅ | ✅ | ❌ | Fixo para lista padrão, LLM para adicionar mais baseado em contexto |
| **Booleano** | ✅ | ❌ | ❌ | Toggle simples, sem necessidade de LLM |

### **Tipo DEPENDÊNCIA (Novo - Regra 1):**

Quando parâmetro precisa de valor de outra tool:

```
SE parâmetro é ID de recurso:
  → Opção: DEPENDÊNCIA
  → Sistema invoca tool GET automaticamente
  → Usuário define critério de seleção
  → Exemplo: deal_id invoca @getAllExistingDealsFromPerson
```

### **Regra de Ouro:**

```
SE tipo_dado = ID de recurso:
  → FIXO: carregar opções da API na configuração
  → LLM: contexto dinâmico via tool de suporte
  → DEPENDÊNCIA: invocar tool GET automaticamente (Regra 1)

SE tipo_dado = Texto:
  → LLM (padrão)
  → FIXO apenas se for template reutilizável

SE tipo_dado IN (Data, Número, Array):
  → FIXO ou LLM (usuário decide estratégia)
```

---

## 🔢 **Regra 4: Multi-Select**

### **Quando permitir Multi-Select:**

**A. Tipo de Dado é Array Nativo**
- ✅ `attendees` em `@createDealActivity` → naturalmente múltiplos
- ✅ Se a API aceita array, permitir multi-select

**B. Lógica de Negócio Permite "Ou" / "Qualquer Um"**
- ✅ Exemplo: "Criar deal em qualquer um desses pipelines: [A, B, C]"
- ⚠️ Cuidado: validar se a API suporta isso ou se é lógica do front

**C. Enum Values para LLM**
- ✅ Quando tipo = "LLM Prompt" e usuário quer **restringir** opções da LLM
- Exemplo: `#title` com "Enum Values (opicional)"
- A LLM só pode escolher entre os valores pré-definidos

### **Comportamento de Multi-Select:**

**REGRA CRÍTICA: Multi-select SEMPRE exige campo de instrução obrigatório**

```
SE multi_select = true E tipo = "Fixo":
  → Usuário seleciona múltiplos valores da lista (checkboxes)
  → Campo "instrução" aparece OBRIGATORIAMENTE
  → Validação: não pode salvar sem preencher instrução
  → Instrução explica QUANDO usar cada valor
  → Exemplo: "Se produto for premium, usar pipeline 2; caso contrário, pipeline 1"

SE multi_select = true E tipo = "LLM Prompt":
  → Campo "Enum Values" aparece (opcional)
  → Se preenchido: LLM deve escolher UM dos valores enum
  → Instrução explica COMO escolher
  → Exemplo: "Escolher pipeline baseado no tipo de produto"
```

### **Validação de Multi-Select:**

```javascript
// Pseudo-código de validação

if (tipo === "Fixo" && selected_values.length > 1) {
  if (!instruction || instruction.trim() === "") {
    return {
      error: true,
      message: "Instrução é obrigatória quando múltiplos valores são selecionados",
      field: "instruction"
    }
  }
}

// UI deve desabilitar botão "Salvar" até instrução ser preenchida
```

### **Comportamento Visual:**

```
Single value selecionado:
  → Campo instrução: OCULTO
  → Não há escolha a fazer

Múltiplos values selecionados:
  → Campo instrução: APARECE automaticamente
  → Marcado com (*) obrigatório
  → Placeholder: "Explique quando usar cada valor selecionado"
  → Botão Salvar: DESABILITADO até preencher
```

---

## 🧩 **Regra 5: Dependências e Validações**

### **A. Dependência Obrigatória de Tool:**

```
SE parâmetro tem dependência obrigatória:
  → Marcar visualmente com badge "🔴 dependente da tool @xxx"
  → Validar antes de salvar: tool dependência está ativa?
  → Se NÃO: mostrar erro e bloquear
  → Se SIM: mostrar indicador verde
```

**Exemplo:**
- `@createDeal` mostra "🔴 dependente da tool @getOrCreatePerson"
- Se `@getOrCreatePerson` não estiver ativa → erro

### **B. Dependência Condicional:**

```
SE parâmetro PODE usar dependência MAS também aceita input manual:
  → Permitir tipo "Dependência" como terceira opção
  → Dropdown: [Fixo | LLM Prompt | De outra Tool]
  → Se escolher "De outra Tool" → selecionar qual tool e qual campo
```

**Exemplo:**
- `deal_id` pode vir de `@createDeal` (dependência) OU usuário pode fornecer ID fixo

### **C. Tool de Suporte Associada (Informativo):**

```
SE parâmetro tipo="LLM" tem tool de suporte associada:
  → Mostrar info (ℹ️ azul) 
  → Apenas informativo, não requer ação
  → "ℹ️ LLM terá acesso aos pipelines via @getAllExistingPipelines"
  
Comportamento:
  → Tool de suporte é chamada automaticamente em runtime
  → Usuário não precisa ativar ou configurar
  → Apenas saber que contexto adicional será fornecido à LLM
```

### **D. Dependência de Tool com Output Array (CRÍTICO - Integra com Regra 2):**

Quando a tool de origem retorna múltiplos resultados, o parâmetro DEVE ser visível:

**Regra de Visibilidade:**
```
SE tool de dependência retorna ARRAY:
  → Parâmetro DEVE ser VISÍVEL
  → Tipo: Dependência OBRIGATÓRIO (sem Fixo/LLM)
  → Usuário DEVE configurar critério de seleção
  
SE tool de dependência retorna VALOR ÚNICO:
  → Parâmetro pode ser OCULTO (Regra 2A)
  → Sistema resolve automaticamente
```

**Implementação:**

```json
{
  "dependency_config": {
    "source_tool": "@getAllExistingDealsFromPerson",
    "source_field": "id",
    "output_type": "array",
    "selection_required": true,
    "show_fields_preview": true,
    "available_fields": [
      {
        "name": "id",
        "type": "number",
        "description": "ID único do deal"
      },
      {
        "name": "title",
        "type": "string",
        "description": "Título do deal"
      },
      {
        "name": "status",
        "type": "enum",
        "values": ["open", "won", "lost"],
        "description": "Status atual do deal"
      },
      {
        "name": "value",
        "type": "number",
        "description": "Valor monetário do deal"
      },
      {
        "name": "stage_current",
        "type": "string",
        "description": "Nome do estágio atual"
      },
      {
        "name": "created_at",
        "type": "datetime",
        "description": "Data de criação"
      }
    ],
    "selection_strategy": {
      "type": "llm_with_criteria",
      "prompt_required": true,
      "prompt_hint": "Use os campos disponíveis acima para criar sua instrução",
      "show_available_fields": true
    }
  }
}
```

**Comportamento na UI com Preview de Campos:**

```
🔗 Depende de: @getAllExistingDealsFromPerson
⚠️ Esta tool retorna múltiplos deals

ℹ️ Campos disponíveis do Deal:
┌────────────────────────────────────┐
│ • id (number)                      │
│ • title (string)                   │
│ • status (string): open, won, lost │
│ • value (number)                   │
│ • currency (string)                │
│ • stage_id (number)                │
│ • stage_current (string)           │
│ • person_id (number)               │
│ • created_at (datetime)            │
│ • updated_at (datetime)            │
└────────────────────────────────────┘

Critério para seleção:
┌──────────────────────────────────────┐
│ ┌────────────────────────────────┐   │
│ │ Escolher deal com status open │   │
│ │ e valor maior que 1000        │   │
│ └────────────────────────────────┘   │
│                                       │
│ 💡 Você pode usar os campos:         │
│    status, value, stage_current,     │
│    created_at, etc.                  │
└──────────────────────────────────────┘

✅ Agora usuário sabe quais palavras usar!
```

**Por que isso é crítico:**
- Usuário não sabe que deal tem campo `stage_current`
- Sem ver os campos, não pode escrever instrução precisa
- Preview mostra estrutura de dados disponível
- Inclui tipos e valores possíveis (enums)

### **E. Preview de Campos para Tipo LLM com Tool de Suporte:**

**IMPORTANTE:** O preview de campos não é exclusivo do tipo "Dependência". Ele também aparece quando:
- Tipo = "LLM Prompt"
- Parâmetro tem tool de suporte associada
- Tool de suporte retorna objetos estruturados

**Exemplo: pipeline_id tipo LLM:**

```
┌────────────────────────────────────────┐
│ pipeline_id                            │
│ tipo: LLM Prompt                       │
│                                        │
│ ℹ️ Tool de suporte:                   │
│    @getAllExistingPipelines            │
│                                        │
│ ℹ️ Campos disponíveis do Pipeline:    │
│ ┌────────────────────────────────────┐ │
│ │ • id (number)                      │ │
│ │ • name (string)                    │ │
│ │ • stages (array)                   │ │
│ │ • deal_probability (boolean)       │ │
│ └────────────────────────────────────┘ │
│                                        │
│ Instrução para LLM:                    │
│ ┌────────────────────────────────────┐ │
│ │ Usar pipeline que contenha "VIP"  │ │
│ │ no campo name                      │ │
│ └────────────────────────────────────┘ │
│                                        │
│ 💡 Você pode usar: id, name, stages,  │
│    deal_probability                    │
└────────────────────────────────────────┘
```

**Diferença entre Dependência vs LLM com Preview:**

| Aspecto | Tipo Dependência | Tipo LLM com Support |
|---------|------------------|----------------------|
| **Quando usar** | Tool retorna array E não tem Fixo/LLM | Parâmetro pode ser Fixo ou LLM |
| **Preview** | ✅ Sempre mostra | ✅ Mostra apenas se tipo=LLM |
| **Chamada tool** | ✅ Sempre em runtime | ✅ Somente quando tipo=LLM |
| **Tipo Fixo** | ❌ Não disponível | ✅ Frontend chama API direto |
| **Exemplo** | deal_id (só Dependência) | pipeline_id (Fixo OU LLM) |

**Regra Universal para Preview:**
```
Mostrar preview de campos SE:
  (tipo = "Dependência") OU 
  (tipo = "LLM Prompt" E existe support_tool)

Não mostrar preview SE:
  tipo = "Fixo" 
  → Valores já são pré-selecionados pelo usuário
```

### **F. Relacionamento entre Parâmetros da Mesma Tool:**

Quando um parâmetro depende de outro **dentro da mesma tool**:

```json
{
  "name": "stage_id",
  "parameter_relationships": [
    {
      "depends_on_parameter": "pipeline_id",
      "type": "contextual",
      "behavior": {
        "disabled_until_filled": true,
        "data_source_filter": "pipeline_id",
        "dynamic_loading": true,
        "cascade_clear": true
      },
      "ui": {
        "disabled_message": "Selecione um pipeline primeiro",
        "loading_message": "Carregando estágios...",
        "relationship_indicator": "🔗 Relacionado a: {pipeline_id.name}"
      }
    }
  ]
}
```

**Tipos de Relacionamento:**

1. **Contextual** - B só faz sentido após A estar preenchido
   - Exemplo: `stage_id` depende de `pipeline_id`

2. **Exclusivo** - A e B são mutuamente exclusivos
   - Exemplo: "Usar pipeline padrão" OU "Selecionar customizado"

3. **Condicional** - B só aparece se A tiver valor específico
   - Exemplo: `lost_reason` só aparece se `status = "lost"`

4. **Derivado** - B é calculado automaticamente de A
   - Exemplo: `expected_close_date = due_date + 30 dias`

---

## 🎨 **Regra 6: Campos Condicionais**

### **A. Tipo = "Fixo":**

**Conceito:** Usuário seleciona valores permitidos de uma lista carregada da API. Em runtime, LLM escolhe ENTRE esses valores.

```
Fluxo na UI de configuração:

1. CARREGAR OPÇÕES (Configuração - Frontend)
   ✅ Frontend chama API diretamente (HTTP, não tool)
   ✅ Exemplo: GET /pipedrive/pipelines
   ✅ Popula dropdown com opções disponíveis
   
2. USUÁRIO SELECIONA (Configuração - Frontend)
   ✅ Usuário marca quais valores permitir
   ✅ Pode selecionar múltiplos (multi-select)
   
3. SE MULTI-SELECT (Configuração - Frontend)
   ✅ OBRIGATÓRIO: Campo de instrução aparece
   ✅ Usuário DEVE explicar quando usar cada valor
   ✅ Sem instrução = erro de validação
   
4. EXECUÇÃO (Runtime - Backend)
   ❌ NÃO chama API novamente
   ❌ NÃO chama tool de suporte
   ✅ LLM recebe apenas valores pré-selecionados + instrução
   ✅ LLM escolhe baseado na conversa
```

**Exemplo - Single Select:**

```
Na configuração:

┌────────────────────────────────┐
│ #pipeline_id - tipo: Fixo      │
│                                │
│ Valores disponíveis:           │
│ ☐ 1 - Pipeline Vendas          │
│ ☑ 2 - Pipeline VIP             │ ← selecionou 1
│ ☐ 3 - Pipeline Inbound         │
└────────────────────────────────┘

Campo de instrução: NÃO aparece (só 1 valor)

Em runtime:
→ LLM recebe: ["2-Pipeline VIP"]
→ LLM usa sempre esse valor
→ Não há escolha a fazer
```

**Exemplo - Multi-Select:**

```
Na configuração:

┌────────────────────────────────┐
│ #pipeline_id - tipo: Fixo      │
│                                │
│ Valores disponíveis:           │
│ ☑ 1 - Pipeline Vendas          │ ← selecionou
│ ☑ 2 - Pipeline VIP             │ ← selecionou
│ ☐ 3 - Pipeline Inbound         │
│                                │
│ ⚠️ Você selecionou múltiplos   │
│    valores. Explique quando    │
│    usar cada um:               │
│                                │
│ Instrução (obrigatória):       │
│ ┌────────────────────────────┐ │
│ │ Se cliente mencionar       │ │
│ │ "premium" ou "vip", usar   │ │
│ │ Pipeline VIP (2).          │ │
│ │ Caso contrário, usar       │ │
│ │ Pipeline Vendas (1).       │ │
│ └────────────────────────────┘ │
└────────────────────────────────┘

✅ Validação: Instrução é obrigatória quando multi-select
❌ Não pode salvar sem preencher

Em runtime:
→ LLM recebe: ["1-Pipeline Vendas", "2-Pipeline VIP"]
→ LLM recebe instrução de escolha
→ LLM analisa conversa
→ LLM escolhe: 2 (Pipeline VIP)
→ ✅ SEM chamar API ou tool
```

**Para enums conhecidos (status, currency):**
- Valores são hardcoded no frontend (não precisa API)
- Usuário pode desmarcar opções que não quer permitir
- Exemplo: status → permite apenas [open, won], desmarca [lost, deleted]
- Multi-select: mesma regra de instrução obrigatória

### **B. Tipo = "LLM Prompt":**

**Conceito:** LLM decide livremente baseado em contexto. Sistema chama tool de suporte para fornecer dados em runtime.

```
Mostrar na UI de configuração:
  ✅ Text area "descrição/instrução" para LLM
  ✅ Campo "Enum Values (opcional)" - restringir escolhas
  ✅ Botão [+] para adicionar enum values opcionais
  ✅ Preview de ESTRUTURA (campos) se há tool de suporte
  ❌ NÃO mostrar lista de valores fixos
```

**Exemplo de configuração COM tool de suporte:**

```
Usuário configura pipeline_id tipo LLM:

┌────────────────────────────────┐
│ tipo: LLM Prompt               │
│                                │
│ ℹ️ Tool de suporte:            │
│    @getAllExistingPipelines    │
│                                │
│ ℹ️ Campos disponíveis:         │
│ ┌────────────────────────────┐ │
│ │ • id (number)              │ │
│ │ • name (string)            │ │
│ │ • stages (array)           │ │
│ └────────────────────────────┘ │
│                                │
│ Instrução para LLM:            │
│ ┌────────────────────────────┐ │
│ │ Usar pipeline que contenha│ │
│ │ "VIP" no campo name       │ │
│ └────────────────────────────┘ │
│                                │
│ 💡 Você pode usar: id, name,  │
│    stages                      │
│                                │
│ Enum Values (opcional):        │
│ (vazio)                        │
└────────────────────────────────┘

Em runtime:
→ Sistema chama @getAllExistingPipelines
→ Retorna: [{id:1, name:"Vendas"}, {id:2, name:"VIP"}, {id:3, name:"Inbound"}]
→ LLM recebe dados + instrução do usuário
→ LLM analisa: "pipeline com 'VIP' no name"
→ LLM encontra: {id:2, name:"VIP"}
→ Retorna: 2
→ ✅ COM chamada de tool de suporte
```

**Por que preview de campos é crítico:**
- Sem preview: usuário não sabe que pode usar `name`, `stages`, etc
- Com preview: pode escrever instruções precisas usando a estrutura real
- Exemplo: "pipeline com mais de 3 stages" (sabe que campo `stages` existe)

**Comportamento do Enum Values (opcional):**

```
SE enum_values está vazio:
  → LLM pode escolher QUALQUER valor retornado pela tool de suporte
  → Liberdade total dentro dos dados disponíveis

SE enum_values tem itens:
  → LLM DEVE escolher apenas entre os valores da lista
  → Exemplo: ["Pipeline VIP", "Pipeline Vendas"] 
  → LLM não pode escolher "Pipeline Inbound" mesmo que exista
  → Sistema adiciona restrição na instrução
```

**Quando tool de suporte é chamada:**

```
SE parâmetro tipo = "LLM Prompt":
  → Verificar se existe tool de suporte associada
  → Exemplo: pipeline_id → @getAllExistingPipelines
  → Chamar tool ANTES de chamar LLM
  → Injetar resultado no contexto da LLM
  
SE não existe tool de suporte:
  → LLM trabalha apenas com instrução do usuário
  → Exemplo: "title" - não precisa buscar dados externos
```

### **C. Tipo = "Dependência":**

**Conceito:** Valor vem exclusivamente de outra tool que retorna array/múltiplos resultados. Usuário define critério para seleção via instrução livre.

```
Mostrar na UI de configuração:
  ✅ Indicação da tool de dependência
  ✅ Campo de texto LIVRE para instrução de critério
  ✅ Preview de campos disponíveis da tool de suporte
  ❌ Não mostrar tipos "Fixo" ou "LLM Prompt"
  ❌ Não mostrar opções pré-definidas (primeiro/último)
  ❌ Não chamar API para listar valores
```

**Exemplo visual:**

```
┌────────────────────────────────────────┐
│ deal_id                                │
│ tipo: Dependência ⚡                   │
│                                        │
│ ℹ️ Tool de suporte:                   │
│    @getAllExistingDealsFromPerson      │
│                                        │
│ ℹ️ Campos disponíveis do Deal:        │
│ ┌────────────────────────────────────┐ │
│ │ • id (number)                      │ │
│ │ • status (enum): open, won, lost   │ │
│ │ • value (number)                   │ │
│ │ • title (string)                   │ │
│ │ • stage_id (number)                │ │
│ └────────────────────────────────────┘ │
│                                        │
│ Critério para seleção:                │
│ ┌────────────────────────────────────┐ │
│ │ Deal com status "open" e valor     │ │
│ │ maior que R$ 1000                  │ │
│ └────────────────────────────────────┘ │
│                                        │
│ 💡 Use os campos disponíveis          │
└────────────────────────────────────────┘
```

**Por que campo livre (não opções pré-definidas):**
- Flexibilidade total: usuário define qualquer critério
- Combina múltiplos campos: "status open AND value > 1000"
- Suporta lógica complexa: "título contém 'VIP' OR valor > 5000"
- Preview de campos garante que usuário saiba o que usar

---

## 🔐 **Regra 7: Obrigatoriedade**

### **Parâmetro Obrigatório:**

```
SE parâmetro é required=true na API:
  → Marcar com asterisco (*) ou indicador visual
  → Validar antes de salvar
  → Se tipo="LLM" → garantir que descrição não está vazia
  → Se tipo="Fixo" → garantir que valor foi selecionado
  → Se tipo="Dependência" → garantir que tool fonte está ativa
```

### **Parâmetro Opcional:**

```
SE parâmetro é optional:
  → Permitir deixar em branco
  → Adicionar toggle "Incluir este parâmetro?"
  → Se desativado → não enviar para API
```

**Otimização UX:**

```
Parâmetros opcionais podem iniciar colapsados:
  → Mostrar apenas obrigatórios por padrão
  → Botão "Mostrar parâmetros avançados [+]"
```

---

## 📋 **Regra 8: Instruções Gerais vs Instruções por Parâmetro**

### **A. Separação de Responsabilidades (CRÍTICO):**

**🚫 Usuário NÃO configura lógica técnica:**
```
❌ Errado: "Converter telefone para formato internacional (+5511987654321)"
❌ Errado: "Remover espaços, parênteses e hífens"
❌ Errado: "Extrair horário no formato HH:mm"
❌ Errado: "Normalizar para maiúsculas/minúsculas"

✅ Correto: "Extrair telefone da conversa"
✅ Correto: "Usar horário mencionado pelo cliente"
✅ Correto: "Extrair nome completo"
```

**Responsabilidade do Sistema (Backend/API):**
- Normalização de dados (formato, case, encoding)
- Validação técnica (regex, tipos, ranges)
- Conversão de formatos (datas, moedas, telefones)
- Sanitização e segurança

**Responsabilidade do Usuário (Configuração):**
- **ONDE** encontrar o dado no contexto
- **QUAL** lógica de negócio aplicar
- **QUANDO** usar cada valor (se multi-select)

### **B. Instruções Gerais (da tool):**

```
Localização: Final do modal
Finalidade: Contexto geral para execução da tool
Exemplos:
  - "Esta é uma venda quente, priorizar"
  - "Cliente é VIP, usar pipeline premium"
  - "Deal deve ser criado apenas se valor > R$1000"
```

### **C. Descrição (por parâmetro):**

```
Localização: Dentro de cada card de parâmetro
Finalidade: Instruir especificamente aquele campo

SE tipo="Fixo" E multi_select=true:
  → Descrição = lógica condicional
  → "Se produto for A, usar pipeline 1..."

SE tipo="LLM Prompt":
  → Descrição = instruções de extração/geração
  → "Extrair telefone da conversa"  (não o formato!)
  → "Usar data mencionada pelo cliente"  (não HH:mm!)
```

---

## 🌐 **Regra 9: Metadata de Parâmetros (Schema Universal)**

Para aplicar essas regras a **qualquer integração**, cada parâmetro deve ter este schema:

```json
{
  "name": "pipeline_id",
  "display_name": "Pipeline",
  "type": "number",
  "required": true,
  "visible": true,
  
  "allowed_input_types": ["fixed", "llm"],
  "default_type": "fixed",
  
  "criticality": "important",
  
  "fixed_config": {
    "multi_select": true,
    "api_endpoint": {
      "method": "GET",
      "url": "/api/pipedrive/pipelines",
      "trigger": "on_modal_open",
      "frontend_call": true,
      "response_mapping": {
        "value_field": "id",
        "label_field": "name"
      }
    },
    "selected_values": [
      {"value": "1", "label": "Pipeline Vendas"},
      {"value": "2", "label": "Pipeline VIP"}
    ],
    "instruction": {
      "required_when_multi": true,
      "value": "Se cliente mencionar premium ou vip, usar Pipeline VIP. Caso contrário, Pipeline Vendas.",
      "placeholder": "Explique quando usar cada valor selecionado",
      "validation": "required_if_multiple_selected"
    }
  },
  
  "llm_config": {
    "allow_enum_restriction": true,
    "default_prompt": "Qual pipeline usar baseado no contexto da conversa?",
    "examples": ["Pipeline Vendas", "Pipeline VIP"],
    "support_tool": {
      "tool": "@getAllExistingPipelines",
      "trigger": "runtime",
      "context_injection": "Pipelines disponíveis: {data}",
      "show_fields_preview": true,
      "available_fields": [
        {
          "name": "id",
          "type": "number",
          "description": "ID único do pipeline"
        },
        {
          "name": "name",
          "type": "string",
          "description": "Nome do pipeline"
        },
        {
          "name": "stages",
          "type": "array",
          "description": "Lista de estágios do pipeline"
        }
      ]
    }
  },
  
  "dependencies": [
    {
      "tool": "@getOrCreatePerson",
      "type": "required",
      "field": "person_id"
    }
  ],
  
  "parameter_relationships": [
    {
      "depends_on_parameter": "other_param",
      "type": "contextual",
      "behavior": {
        "disabled_until_filled": true
      }
    }
  ],
  
  "validation": {
    "min": 1,
    "error_message": "Pipeline é obrigatório"
  },
  
  "normalization": {
    "enabled": false
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

---

# ⚙️ **REGRAS DE SCHEMA/SISTEMA**

> **As regras abaixo (10-12) são para definição do SCHEMA da tool pelo desenvolvedor.**  
> O usuário final NÃO configura essas regras - elas são aplicadas automaticamente.

---

## 🔴 **Regra 10: Criticidade de Parâmetros**

**Audiência:** Desenvolvedor definindo schema  
**Aplicação:** Automática pelo sistema  
**Visível ao usuário:** Apenas os efeitos (badges, validações, normalizações)

Classificar parâmetros por impacto no sistema:

### **Nível 1: Crítico (🔴)**

**Características:**
- Usado como chave de busca/identificação
- Erro causa duplicação ou perda de dados
- Formato incorreto compromete integridade

**Exemplos:**
- `phone` - usado para buscar pessoa existente
- `email` - quando usado como identificador único
- `deal_id` - quando usado para operações destrutivas

**Tratamento Especial:**

```json
{
  "criticality": "critical",
  "validation": {
    "required": true,
    "format": "e164_international",
    "error_message": "⚠️ Formato incorreto causará duplicatas no CRM"
  },
  "normalization": {
    "enabled": true,
    "auto_apply": true,
    "rules": [
      "remove_whitespace",
      "remove_special_chars",
      "add_country_code_if_missing"
    ]
  },
  "ui_indicators": {
    "badge": "🔴 Campo Crítico",
    "help_text": "Campo usado para buscar pessoas existentes. Sistema normaliza formato automaticamente."
  }
}
```

**Comportamento na UI:**

```
┌─────────────────────────────────────┐
│ #phone * 🔴 Campo Crítico           │
│                                     │
│ tipo: LLM Prompt                    │
│                                     │
│ Instrução para LLM:                 │
│ ┌─────────────────────────────────┐ │
│ │ Extrair telefone da conversa    │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ⚠️ Campo usado para buscar pessoas │
│    existentes. Sistema normaliza   │
│    formato automaticamente.         │
└─────────────────────────────────────┘
```

### **Nível 2: Importante (🟡)**

**Características:**
- Afeta comportamento principal da tool
- Erro causa falha na operação mas é reversível
- Validação importante mas não crítica

**Exemplos:**
- `title` - obrigatório mas não afeta busca
- `person_id` - validado por dependência
- `pipeline_id` - opcional com fallback

**Tratamento:**

```json
{
  "criticality": "important",
  "validation": {
    "required": true,
    "min_length": 1
  },
  "ui_indicators": {
    "badge": "* Obrigatório"
  }
}
```

### **Nível 3: Complementar (⚪)**

**Características:**
- Enriquece informação mas não é essencial
- Erro não quebra a operação
- Geralmente opcional

**Exemplos:**
- `currency` - tem valor padrão
- `probability` - opcional
- `description` - complementar

**Tratamento:**

```json
{
  "criticality": "complementary",
  "required": false,
  "ui_indicators": {
    "badge": "Opcional",
    "collapsible": true
  }
}
```

---

## 📭 **Regra 11: Null e Empty Handling**

**Audiência:** Desenvolvedor definindo schema  
**Aplicação:** Automática pelo sistema  
**Visível ao usuário:** Apenas indicadores ("⚪ Não será enviado")

Define como tratar valores vazios e nulos em parâmetros opcionais.

### **Estratégia por Tipo de Dado:**

```javascript
SE tipo = "string":
  → empty_string → converter para null antes de enviar
  → null → não enviar parâmetro na API call
  → whitespace only → tratar como empty

SE tipo = "number":
  → empty/null → não enviar parâmetro
  → 0 → ENVIAR (zero é valor válido)
  → NaN → não enviar

SE tipo = "array":
  → [] → não enviar parâmetro
  → null → não enviar parâmetro
  → [null] → limpar e não enviar

SE tipo = "boolean":
  → undefined → não enviar parâmetro
  → null → não enviar parâmetro
  → false → ENVIAR (false é valor válido!)
```

### **Configuração por Parâmetro:**

```json
{
  "name": "email",
  "type": "string",
  "required": false,
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false,
    "send_when_empty_string": false,
    "trim_whitespace": true
  }
}
```

### **Para LLM:**

```json
{
  "llm_config": {
    "null_handling": {
      "instruction": "Se não encontrar email, retorne explicitamente a palavra NULL",
      "parse_strategy": "convert_string_null_to_real_null",
      "fallback_value": null,
      "validation": "reject_if_invalid_email"
    }
  }
}
```

### **Comportamento na UI:**

```
Campo opcional vazio:
  → Indicador: "⚪ Este campo não será enviado"

Campo opcional preenchido:
  → Indicador: "✅ Será enviado: [valor]"

Campo opcional com valor inválido:
  → Indicador: "⚠️ Valor inválido - não será enviado"
```

### **Tabela de Conversão:**

| Input | Tipo String | Tipo Number | Tipo Array | Tipo Boolean |
|-------|-------------|-------------|------------|--------------|
| `""` | null → não envia | não envia | não envia | não envia |
| `" "` | null → não envia | não envia | não envia | não envia |
| `null` | não envia | não envia | não envia | não envia |
| `undefined` | não envia | não envia | não envia | não envia |
| `0` | envia "0" | envia 0 ✅ | não envia | não envia |
| `false` | envia "false" | não envia | não envia | envia false ✅ |
| `[]` | não aplica | não aplica | não envia | não aplica |

---

## 🏷️ **Regra 12: Categorias de Tools (Integração com Regra 1)**

**Audiência:** Desenvolvedor definindo schema  
**Aplicação:** Automática pelo sistema  
**Visível ao usuário:** Apenas resultado (quais tools aparecem na lista @)

> **Esta regra consolida e expande a Regra 1 (Visibilidade de Tools) com detalhes de implementação**

Separar tools por função para melhor organização da UX.

**Resumo de Categorias (baseado na Regra 1):**

| Categoria | Visível @ | Padrão Naming | Quando Executa |
|-----------|-----------|---------------|----------------|
| **Ação** | ✅ SIM | create, update, delete, add | Explícito (usuário configura) |
| **Consulta Auto** | ❌ NÃO | get, getAll | Por dependência (Regra 5) |
| **Suporte LLM** | ❌ NÃO | getAll* | Runtime quando tipo=LLM (Regra 6B) |
| **Híbrida** | ✅ SIM | getOrCreate | Explícito (tem efeito colateral) |

### **Categoria A: Tools de Ação (User-Facing)**

**Características:**
- Executam operações no sistema externo (criar, atualizar, deletar)
- Usuário configura explicitamente
- Aparecem na lista de @tools do checkpoint
- Têm parâmetros configuráveis visíveis

**Exemplos:**
- `@createDeal`
- `@updateDeal`
- `@createNote`
- `@createDealActivity`

**Configuração:**

```json
{
  "tool": "@createDeal",
  "category": "action",
  "user_configurable": true,
  "show_in_checkpoint_menu": true,
  "icon": "💼",
  "requires_user_setup": true
}
```

**Comportamento:**
- Aparece na lista quando usuário digita `@`
- Abre modal de configuração completo
- Salva configuração no checkpoint

### **Categoria B: Tools de Suporte (Runtime Context)**

**Características:**
- Fornecem contexto para LLM em runtime
- Chamadas automaticamente quando parâmetro tipo = "LLM Prompt"
- NÃO aparecem na lista de @tools do checkpoint
- Não têm configuração visível para usuário

**Exemplos:**
- `@getAllExistingPipelines`
- `@getAllUsers` (hipotético)
- `@getCurrencies` (hipotético)

**Configuração:**

```json
{
  "tool": "@getAllExistingPipelines",
  "category": "support",
  "user_configurable": false,
  "auto_execute": "when_needed",
  "show_in_checkpoint_menu": false,
  "trigger": "runtime_for_llm_context",
  "cache_duration": 300,
  "provides_context_for": [
    {
      "tool": "@createDeal",
      "parameter": "pipeline_id",
      "when_type": "llm"
    },
    {
      "tool": "@updateDeal", 
      "parameter": "pipeline_id",
      "when_type": "llm"
    }
  ]
}
```

**Comportamento em Runtime:**

```
Usuário configurou pipeline_id como "LLM Prompt":

1. Checkpoint é executado
2. Sistema identifica: pipeline_id precisa de contexto
3. Sistema chama @getAllExistingPipelines
4. Recebe: [{id:1, name:"Vendas"}, {id:2, name:"VIP"}]
5. Injeta no contexto da LLM:
   "Pipelines disponíveis: 1-Vendas, 2-VIP"
6. LLM executa com contexto enriquecido
7. LLM retorna: 2

✅ Tudo automático, usuário não vê a tool de suporte
```

**Comportamento se tipo = "Fixo":**

```
Usuário configurou pipeline_id como "Fixo":
→ Valores já foram definidos pelo usuário na configuração
→ Tool de suporte NÃO é chamada
→ LLM escolhe apenas entre valores pré-definidos
```

**Cache:**
- Resultado é cacheado por 5 minutos
- Se múltiplos parâmetros usam mesma tool, chama apenas 1x
- Cache é por sessão/conversa

### **Categoria C: Tools de Consulta (Query)**

**Características:**
- Buscam informações sem modificar dados
- Usuário pode querer ou não usar
- Aparecem na lista mas com indicador diferente
- Úteis para enriquecer contexto

**Exemplos:**
- `@getDealWithCompleteInfo`
- `@getActivitiesFromDeal`
- `@getAllExistingDealsFromPerson`

**Configuração:**

```json
{
  "tool": "@getDealWithCompleteInfo",
  "category": "query",
  "user_configurable": true,
  "show_in_checkpoint_menu": true,
  "icon": "🔍",
  "recommended_after": ["@createDeal"],
  "read_only": true
}
```

**Comportamento:**
- Aparece na lista com ícone 🔍
- Pode ser sugerida automaticamente:
  ```
  💡 Sugestão: Use @getDealWithCompleteInfo para enriquecer o deal criado
  ```
- Não modifica dados, apenas lê

### **Indicadores Visuais por Categoria:**

```
Lista de @tools disponíveis (o que usuário vê):

[AÇÕES]
  💼 @createDeal - Criar novo deal
  ✏️ @updateDeal - Atualizar deal existente
  📝 @createNote - Adicionar nota

[CONSULTAS]
  🔍 @getDealWithCompleteInfo - Buscar detalhes do deal
  🔍 @getActivitiesFromDeal - Listar atividades

[SUPORTE]
  → NÃO aparece na lista @
  → Chamadas automaticamente em runtime quando necessário
  → Exemplos: @getAllExistingPipelines, @getAllUsers
```

**Como usuário sabe que tool de suporte existe?**

```
Na configuração do parâmetro tipo "LLM":

┌────────────────────────────────┐
│ #pipeline_id                   │
│ tipo: LLM Prompt               │
│                                │
│ ℹ️ LLM terá acesso automático │
│    aos pipelines disponíveis   │
│    via @getAllExistingPipelines│
│                                │
│ Instrução:                     │
│ ...                            │
└────────────────────────────────┘

✅ Informativo apenas
❌ Não precisa ativar manualmente
```

---

## 🎯 **Resumo Executivo: Árvore de Decisão**

### **Para Desenvolvedor (definindo schema):**

```
1. Este parâmetro deve aparecer para o usuário?
   └─ NÃO → Se é dependência pura ou técnico (Regra 1)
   └─ SIM → Continue...

2. Qual o nível de criticidade? (Regra 9 - Sistema)
   └─ 🔴 Crítico → Chave de busca, add normalização automática
   └─ 🟡 Importante → Validação rigorosa
   └─ ⚪ Complementar → Pode ser colapsado

3. Como tratar null/empty? (Regra 10 - Sistema)
   └─ Definir nullable_behavior no schema
   └─ Sistema aplica automaticamente

4. Qual categoria da tool? (Regra 11 - Sistema)
   └─ Ação → Aparece na lista @
   └─ Suporte → Auto-executa em background
   └─ Consulta → Aparece com 🔍
```

### **Para Usuário (configurando tool):**

```
1. Que tipo de input usar para este parâmetro?
   
   └─ FIXO (Regra 2, 5A)
      → Definir manualmente os valores permitidos
      → Ex: "1-Pipeline Vendas", "2-Pipeline VIP"
      → LLM escolhe entre esses valores em runtime
      → NÃO chama tool de suporte
   
   └─ LLM PROMPT (Regra 2, 5B)
      → Escrever instrução livre para LLM
      → Ex: "Identificar pipeline baseado no produto"
      → Sistema chama tool de suporte em runtime (se houver)
      → LLM recebe dados dinâmicos + instrução
   
   └─ DEPENDÊNCIA (Regra 5D)
      → Valor vem de outra tool que retorna array
      → Ex: deal_id vem de @getAllExistingDealsFromPerson
      → Campo LIVRE para critério de seleção
      → Preview de campos disponíveis

2. Se escolhi FIXO:
   └─ Adicionar valores permitidos manualmente [+ Adicionar]
   └─ Multi-select? (Regra 4)
      → SIM → Adicionar vários + escrever lógica de escolha
      → NÃO → Apenas um valor
   └─ Escrever instrução: "Como LLM deve escolher?"

3. Se escolhi LLM:
   └─ Escrever instrução para LLM
   └─ Preview de campos disponíveis? (Regra 5E)
      → SE existe tool de suporte → Mostra campos disponíveis
      → Ajuda a escrever instruções precisas
   └─ Restringir opções com Enum Values? (Regra 4)
      → SIM → Adicionar lista restrita
      → NÃO → LLM livre (dentro dos dados disponíveis)
   └─ ℹ️ Ver info sobre tool de suporte (se existir)

4. Se é tipo DEPENDÊNCIA:
   └─ Campo instrução LIVRE obrigatório (Regra 5D)
   └─ Preview de campos disponíveis sempre mostrado
   └─ Não há opções pré-definidas (primeiro/último)
   └─ Ex: "Deal com status 'open' e valor > 1000"

5. Dependências são validadas automaticamente (Regra 5A)
   └─ UI mostra erros se tool dependente não estiver ativa

6. Parâmetros relacionados respondem dinamicamente (Regra 5F)
   └─ Ex: stages carregam após selecionar pipeline

7. Campo obrigatório ou opcional? (Regra 7)
   └─ Obrigatório (*) → Deve preencher
   └─ Opcional → Pode deixar vazio
```

> 💡 **Normalização, validação e tool de suporte acontecem automaticamente** baseado no schema (Regras 9-11).

---

## 📊 **Índice de Regras**

### 🎨 **Regras de UX/Configuração (Usuário)**

1. **Visibilidade de Tools** - Quais tools aparecem na lista @ (padrão GET oculto)
2. **Visibilidade do Parâmetro** - Quando mostrar/ocultar parâmetros
3. **Tipos de Preenchimento** - Fixo, LLM ou Dependência
4. **Multi-Select** - Quando e como permitir múltiplos valores
5. **Dependências e Validações** - Entre tools e entre parâmetros
   - 5A: Dependência obrigatória de tool
   - 5B: Dependência condicional
   - 5C: Tool de suporte associada (informativo)
   - 5D: Dependência com output array (campo livre + preview)
   - 5E: Preview de campos para LLM com tool de suporte
   - 5F: Relacionamento entre parâmetros da mesma tool
6. **Campos Condicionais** - UI por tipo de input
7. **Obrigatoriedade** - Tratamento de campos required/optional
8. **Instruções** - Gerais vs por parâmetro
9. **Metadata Schema** - Estrutura JSON universal

### ⚙️ **Regras de Schema/Sistema (Desenvolvedor)**

10. **Criticidade** - Classificação de impacto (crítico/importante/complementar)
11. **Null e Empty Handling** - Como tratar valores vazios
12. **Categorias de Tools** - Integração com Regra 1 (Ação/Consulta/Suporte/Híbrida)

---

**Versão:** 3.2  
**Data:** 2025-11-06  
**Status:** Pronto para implementação

**Changelog:**
- v3.2: **UNIVERSALIZAÇÃO DO PREVIEW DE CAMPOS**
  - **Regra 5E (NOVA):** Preview de campos também para tipo LLM com tool de suporte
    - Não é exclusivo do tipo Dependência
    - Aparece quando: tipo=LLM E existe support_tool E tool retorna objetos estruturados
    - Exemplo: pipeline_id tipo LLM mostra campos disponíveis (id, name, stages)
    - Ajuda usuário a escrever instruções precisas usando estrutura real dos dados
  - **Regra 5D Atualizada:** Campo LIVRE para critério de seleção
    - ❌ Removidas opções pré-definidas (primeiro/último da lista)
    - ✅ Apenas campo de instrução livre obrigatório
    - Flexibilidade total: "status open AND value > 1000"
    - Suporta lógica complexa combinando múltiplos campos
  - **Regra 6C Refinada:** Tipo Dependência com campo livre
    - Enfatiza instrução livre (não opções pré-selecionadas)
    - Preview de campos sempre visível
    - Sem chamada de API para listar valores
  - **Schema atualizado:** Campo `available_fields` também no `llm_config.support_tool`
  - **Árvore de Decisão atualizada:** Reflete preview universal e campo livre
- v3.1: **REFINAMENTO CRÍTICO - Dependências com Array**
  - **Regra 2A Refinada:** "Dependência Pura" → "Dependência Única (Valor Singular)"
    - ✅ OCULTAR: quando tool retorna valor ÚNICO (person_id de @getOrCreatePerson)
    - ❌ MOSTRAR: quando tool retorna ARRAY (deal_id de @getAllExistingDealsFromPerson)
  - **Regra 3:** Matriz atualizada diferenciando "ID singular" vs "ID com múltiplas opções"
  - **Regra 5D:** Preview de Campos Disponíveis (GAME CHANGER para UX)
    - Sistema mostra campos disponíveis do objeto retornado
    - Usuário vê: status, value, stage_current, created_at, etc.
    - Exemplo: "ℹ️ Campos disponíveis do Deal: • id • status • value..."
    - Permite escrever instruções precisas: "deal com status open e valor > 1000"
  - **Schema atualizado:** Campo `available_fields` com preview de estrutura de dados
- v3.0: Nova Regra 1 - Visibilidade de Tools (padrão GET oculto)
- v2.3: Correção de fluxo completo Fixo vs LLM + multi-select obrigatório
- v2.2: Correção de fluxo de tools de suporte
- v2.1: Separação entre regras de UX e Schema
- v2.0: Adição de criticidade, null handling e categorias
- v1.0: Framework inicial
