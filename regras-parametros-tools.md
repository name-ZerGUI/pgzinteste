# 🎯 Framework de Regras para Parâmetros de Tools

Sistema universal de regras para configuração de parâmetros de tools em qualquer integração.

**Versão:** 3.7.1 | **Data:** 2025-11-10 | **Status:** Implementação

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
                        │ 4️⃣ Definir `allowed_input_types`    │ (REGRA 3)
                        │    (Pode ser um ou mais)             │
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
             ▼                                      ▼
    ┌────────────────┐                   ┌──────────────────┐
    │ 5️⃣ Multi-select│                   │ Preview de campos│
    │ permitido?     │                   │ disponíveis?     │
    │ (REGRA 4)      │                   │ (REGRA 5E)       │
    └────┬───────────┘                   └─────────┬────────┘
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
    │ • llm_config (support_tool, etc)         │
    │ • dependencies                           │
    │ • parameter_relationships                │
    │ • validation                             │
    │ • show_by_default (Regra 10 - Sistema)  │
    │ • is_critical_field (Regra 11 - Sistema)│
    │ • nullable_behavior (Regra 12 - Sistema)│
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
    └────────────────┬───────────────────────┘
                     │
                     ▼
        ┌────────────────────────────────┐
        │ Para cada PARÂMETRO visível:   │
        └────────┬───────────────────────┘
                 │
                 ▼
    ┌─────────────────────────────────────────┐
    │ 1️⃣ Apenas UM tipo de preenchimento     │ (REGRA 3)
    │    permitido pelo schema?               │
    └────┬───────────────────────────┬────────┘
         │                           │
      SIM│                           │ NÃO (múltiplos tipos)
         │                           │
         ▼                           ▼
┌──────────────────────────┐  ┌──────────────────────────────────┐
│ Mostrar UI do tipo       │  │ 2️⃣ Usuário escolhe o tipo:      │
│ único DIRETAMENTE        │  │    ┌──────────────────────────┐  │
│ (sem seletor de tipo)    │  │    │ [Fixo | LLM]             │  │
│                          │  │    └──────────────────────────┘  │
└───────────┬──────────────┘  └───────────┬──────────────────────┘
            │                             │
            └─────────────┬───────────────┘
                          │
                          ▼
    ┌──────────────────────────────────────────┐
    │ 3️⃣ UI se adapta ao tipo escolhido/definido:│ (REGRA 6)
    └────┬────────────────────────────┬──────────┘
         │                            │
         ▼                            ▼
    ┌──────────┐                 ┌──────────┐
    │ SE FIXO  │                 │ SE LLM   │
    └────┬─────┘                 └────┬─────┘
         │                            │
         ▼                            ▼
    ┌──────────────────┐   ┌──────────────────┐
    │ UI mostra:       │   │ UI mostra:       │
    │                  │   │                  │
    │ ☑ Lista valores  │   │ ✎ Text area      │
    │   disponíveis    │   │   instrução      │
    │                  │   │                  │
    │ Multi-select?    │   │ ℹ️ Tool suporte  │
    │ → SE SIM:        │   │   será chamada   │
    │   ⚠️ Campo       │   │   em runtime     │
    │   instrução      │   │                  │
    │   OBRIGATÓRIO    │   │                  │
    │                  │   │                  │
    └────┬─────────────┘   └────┬─────────────┘
         │                      │
         └──────────┬───────────┘
                    │
                    ▼
    ┌───────────────────────────────────────┐
    │ 4️⃣ SE parâmetro tem DEPENDÊNCIA:     │ (REGRA 5D)
    └────┬──────────────────────────────────┘
         │
         ▼
    ┌──────────────────────────────────────┐
    │ Tipo = Dependência OBRIGATÓRIO       │
    │ (UI mostrada diretamente)            │
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
    │ 4.1️⃣ SE tipo LLM com tool suporte:   │ (REGRA 5E)
    └────┬──────────────────────────────────┘
         │
         ▼
    ┌──────────────────────────────────────┐
    │ Também mostra preview de campos!     │
    │                                      │
    │ ℹ️ Tool de suporte:                 │
    │    @searchEmailTemplates             │
    │                                      │
    │ ℹ️ Campos disponíveis:              │
    │    • id (number)                     │
    │    • name (string)                   │
    │    • subject (string)                │
    │    • tags (array)                    │
    │                                      │
    │ Instrução para LLM:                  │
    │ ┌────────────────────────────────┐   │
    │ │ Usar template com a tag        │   │
    │ │ "onboarding"                   │   │
    │ └────────────────────────────────┘   │
    │                                      │
    │ 💡 Você pode usar: id, name, subject │
    └──────────────────────────────────────┘
                    │
                    ▼
    ┌───────────────────────────────────────┐
    │ 5️⃣ Validações automáticas:           │ (REGRA 7)
    └────┬──────────────────────────────────┘
         │
         ▼
    ┌──────────────────────────────────────┐
    │ • Campos obrigatórios (*) preenchidos│
    │ • Multi-select tem instrução         │
    │ • Tool dependente está ativa         │
    │ • Parâmetros relacionados válidos    │
    │                                      │
    │ ✅ Tudo OK → Botão "Salvar" ativo    │
    │ ❌ Falta algo → Botão desabilitado   │
    └──────────────────────────────────────┘
                    │
                    ▼
    ┌───────────────────────────────────────┐
    │ 6️⃣ Instruções gerais (opcional):     │ (REGRA 8)
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

**A. Tools de Consulta/Leitura Automáticas (Padrão GET)**
- ✅ Tools que começam com `get` ou `getAll`
- ✅ Apenas leem dados, não modificam
- ✅ São invocadas automaticamente como dependências ou suporte

**Exemplos que NÃO aparecem:**
- `@getAllExistingDealsFromPerson` → invocada automaticamente quando @updateDeal precisa de deal_id
- `@getAllExistingPipelines` → invocada automaticamente para contexto LLM

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

**C. Tools de Consulta Visíveis (EXCEÇÃO ao padrão GET)**
- ✅ Começam com `get` MAS usuário pode querer executar explicitamente
- ✅ Úteis para obter contexto ou informações detalhadas
- ✅ Exemplo: `@getDealWithCompleteInfo`, `@getActivitiesFromDeal`
- ✅ Aparecem com ícone 🔍 para diferenciá-las de ações

### **Padrão de Naming como Indicador:**

| Prefixo | Visível? | Categoria | Invocação |
|---------|----------|-----------|-----------|
| `get`, `getAll` | ❌ NÃO* | Consulta Auto | Por dependência |
| `get*` (exceção) | ✅ SIM | Consulta Visível 🔍 | Explícita (opcional) |
| `create`, `add` | ✅ SIM | Ação | Explícita |
| `update`, `edit` | ✅ SIM | Ação | Explícita |
| `delete`, `remove` | ✅ SIM | Ação | Explícita |
| `getOrCreate` | ✅ SIM | Híbrida | Explícita |

\* Exceção: Tools de consulta que o usuário pode querer executar explicitamente aparecem como "Consulta Visível"

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

### **Critério de Decisão (ATUALIZADO v3.4):**

**Perguntas-chave:**
1. *"A API valida/rejeita valores fora de uma lista fechada?"*
2. *"O valor é único por contexto ou pode ser reutilizado?"*
3. *"É um ID de recurso que precisa existir na API?"*

---

### **Matriz de Decisão:**

| Tipo de Dado | Fixo | LLM | Dep | API Valida? | Justificativa |
|--------------|------|-----|-----|-------------|---------------|
| **ID de recurso pré-existente** (pipeline_id, stage_id, user_id, org_id) | ✅ ONLY | ❌ | ❌ | ✅ Sim | API só aceita IDs existentes. LLM não pode inventar. Lista carregada da API. |
| **ID de dependência singular** (person_id quando tool retorna único valor) | ❌ | ❌ | ✅ ONLY | ✅ Sim | Ocultar parâmetro. Valor único resolvido automaticamente por dependência. |
| **ID de dependência com array** (deal_id quando tool retorna múltiplos) | ❌ | ❌ | ✅ ONLY | ✅ Sim | Tipo Dependência visível. Usuário define critério de seleção entre múltiplos. |
| **Enum validado pela API** (status, currency, priority) | ✅ ONLY | ❌ | ❌ | ✅ Sim | API REJEITA valores fora do enum. Valor deve ser exato. Lista fechada. |
| **String livre sem validação** (lost_reason, description) | ❌ | ✅ ONLY | ❌ | ❌ Não | API ACEITA qualquer string. Conteúdo único por contexto. Não há lista. |
| **Texto livre gerado** (title, content, note, subject) | ❌ | ✅ ONLY | ❌ | ❌ Não | API ACEITA qualquer string. Sempre gerado dinamicamente. Cada contexto é único. |
| **Texto extraído de contexto** (fullname, email, phone) | ❌ | ✅ ONLY | ❌ | ❌ Não* | API ACEITA qualquer string (*valida formato). Extraído da conversa. Sem variáveis. |
| **Data contextual** (due_date, due_time, expected_close_date) | ❌ | ✅ ONLY | ❌ | ❌ Não | Varia por conversa. "Usar data mencionada pelo cliente". Raramente é fixa. |
| **Valor contextual** (value do deal, probability) | ❌ | ✅ ONLY | ❌ | ❌ Não | Varia por conversa e engajamento. Extraído do contexto. Raramente é fixo. |
| **Participantes contextuais** (attendees) | ❌ | ✅ ONLY | ❌ | ❌ Não | Varia por conversa. "Pessoa do deal + outros mencionados". Raramente é fixo. |
| **Duração padrão com presets** (duration) | ✅ ONLY | ❌ | ❌ | ❌ Não | Valores padrão da empresa (30min, 1h, 2h). LLM não agrega valor. |
| **Booleano** | ✅ ONLY | ❌ | ❌ | ❌ Não | Toggle simples (true/false). Não precisa de LLM. |

> ⚠️ **IMPORTANTE**: A prática mostrou que **raramente** faz sentido ter múltiplos tipos (`["llm", "fixed"]`). Na dúvida, escolha **apenas um tipo** baseado no caso de uso real.

---

### **Regras Específicas por Tipo:**

#### **A. IDs de Recursos Pré-Existentes**

```
Exemplos: pipeline_id, stage_id, user_id, org_id

allowed_input_types: ["fixed"]  ← APENAS FIXO

❌ LLM NÃO FAZ SENTIDO porque:
  → LLM não pode "inventar" um ID que não existe no sistema
  → Mesmo com support tool, a escolha é entre valores fixos
  → Não há interpretação semântica útil
  → Apenas seleção de dropdown

✅ Fluxo correto:
  1. Frontend carrega lista da API (GET /pipelines)
  2. Usuário seleciona 1 ou mais valores permitidos
  3. Se multi-select: adiciona campo de instrução para quando usar cada um
  4. Em runtime: LLM escolhe ENTRE os valores pré-selecionados
```

#### **B. Enums e Validação de API**

**Critério-chave:** O que a API ACEITA ou REJEITA?

**B.1. Enum Validado pela API (Apenas "Fixo")**

```
Definição: API REJEITA qualquer valor fora do conjunto predefinido

Exemplos: 
  - status: enum("open", "won", "lost", "deleted")
  - currency: enum("BRL", "USD", "EUR", "GBP")
  - priority: enum("high", "medium", "low")
  - type: enum("call", "meeting", "task", "email")

allowed_input_types: ["fixed"]  ← APENAS FIXO

❌ LLM NÃO FAZ SENTIDO porque:
  → API valida e rejeita valores inválidos
  → Não há interpretação possível - valor deve ser EXATO
  → LLM não pode "criar" novos valores do enum
  → Erro 400/422 se enviar valor fora da lista

✅ Fluxo correto:
  1. Frontend carrega enum da API ou usa hardcoded
  2. Usuário seleciona 1 ou múltiplos valores permitidos
  3. Se multi-select: instrução de quando usar cada um
  4. Em runtime: LLM escolhe ENTRE os valores pré-selecionados

Como identificar:
  ✓ Documentação da API diz: "enum", "allowed values", "one of"
  ✓ API retorna erro se valor não está na lista
  ✓ Valores são códigos/identificadores técnicos
```

**B.2. String Livre Sem Validação (Apenas "LLM")**

```
Definição: API ACEITA qualquer string, sem lista predefinida

Exemplos:
  - lost_reason: string (motivo da perda em texto livre)
  - description: string (descrição livre)
  - note: string (nota/observação livre)
  - custom_field: string (campo customizado)

allowed_input_types: ["llm"]  ← APENAS LLM

❌ FIXO NÃO FAZ SENTIDO porque:
  → Não há lista fechada de valores
  → Cada contexto é único e requer texto específico
  → Valor fixo literal seria repetitivo e sem utilidade

✅ Apenas LLM com instrução:
  "Extrair motivo da perda mencionado pelo cliente"
  "Criar descrição detalhada do problema relatado"

Como identificar:
  ✓ Documentação diz: "string", "text", "free text"
  ✓ API aceita qualquer valor de string
  ✓ Não há lista de valores válidos
  ✓ Conteúdo varia por contexto de negócio
```

**B.3. Enum Sugerido (NÃO USAR - Removido em v3.5)**

```
❌ REMOVIDO: Anteriormente permitia ["llm", "fixed"] para valores sugeridos

✅ DECISÃO v3.5: Escolher UM tipo dominante:
  → SE há valores restritos importantes → usar ["fixed"] com multi-select
  → SE valores variam muito por contexto → usar ["llm"]

Justificativa:
  → Múltiplos tipos complicam UX sem agregar valor
  → 90%+ dos casos usa apenas um tipo
  → Melhor ter interface simples e consistente
```

#### **C. Texto Livre Gerado**

```
Exemplos: title, content, note, description, subject

allowed_input_types: ["llm"]  ← APENAS LLM

❌ FIXO NÃO FAZ SENTIDO porque:
  → Cada contexto requer texto diferente
  → Valor fixo literal seria repetitivo

✅ Apenas LLM com instrução:
  "Criar título usando: [regra de negócio]"
```

#### **D. Texto Extraído de Contexto**

```
Exemplos: fullname, email, phone (dados de identificação)

allowed_input_types: ["llm"]  ← APENAS LLM

❌ FIXO NÃO DISPONÍVEL porque:
  → Sistema de variáveis não implementado ainda
  → Não há {{lead_name}}, {{contact_email}}, etc
  → Futuramente pode ser adicionado quando houver variáveis

✅ Apenas LLM para extração:
  → "Extrair nome da conversa"
  → "Identificar email do cliente"
  → "Extrair telefone da conversa"
```

---

#### **E. Quando NÃO Permitir Múltiplos Tipos**

⚠️ **REGRA PRÁTICA CRÍTICA**: Na dúvida, escolha **apenas um tipo**.

Múltiplos tipos (`["llm", "fixed"]`) raramente fazem sentido porque:

**1. Datas e Horários**
```
❌ EVITAR: ["llm", "fixed"] para due_date, due_time, expected_close_date

POR QUÊ:
  → Datas/horários são contextuais (disponibilidade do cliente)
  → Raramente a empresa tem "data fixa padrão" para reuniões
  → Valor fixo só faria sentido se SEMPRE fosse o mesmo (ex: "sempre 14h")
  → Na prática, isso não acontece

✅ USAR: ["llm"] apenas
  → "Usar data/horário mencionado pelo cliente"
  → "Se não mencionar, sugerir amanhã às 14h"
```

**2. Valores Monetários e Probabilidades**
```
❌ EVITAR: ["llm", "fixed"] para value, probability

POR QUÊ:
  → Valor do deal varia por produto/serviço mencionado na conversa
  → Probabilidade varia por engajamento do cliente
  → Valor fixo só faria sentido se empresa vende APENAS 1 produto pelo mesmo preço
  → Se há regra fixa de probabilidade, deve estar configurada no CRM/API, não na tool

✅ USAR: ["llm"] apenas
  → "Extrair valor mencionado pelo cliente"
  → "Estimar probabilidade baseado no engajamento"
```

**3. Participantes/Attendees**
```
❌ EVITAR: ["llm", "fixed"] para attendees

POR QUÊ:
  → Participantes variam por conversa (quem foi mencionado?)
  → Lista fixa só faria sentido se SEMPRE fossem as mesmas 2-3 pessoas
  → Na prática, varia muito

✅ USAR: ["llm"] apenas
  → "Adicionar pessoa do deal + outros mencionados na conversa"
```

**4. Duração (EXCEÇÃO - Apenas Fixo)**
```
✅ USAR: ["fixed"] apenas para duration

POR QUÊ:
  → Empresa tem padrões (30min, 1h, 1h30, 2h)
  → Raramente cliente especifica duração exata
  → LLM não agrega valor ("reunião rápida" = 30min? 45min?)
  → Melhor ter presets claros

❌ EVITAR: ["llm"]
  → Não há contexto suficiente para LLM decidir
```

**REGRA DE OURO ATUALIZADA:**
```
SE você está pensando em permitir múltiplos tipos, pergunte:

1. O valor é REALMENTE fixo em 90%+ dos casos?
   → SIM: apenas ["fixed"]
   → NÃO: apenas ["llm"]

2. O valor é REALMENTE contextual em 90%+ dos casos?
   → SIM: apenas ["llm"]
   → NÃO: apenas ["fixed"]

3. Oscila 50/50 entre fixo e contextual?
   → RARO: revise o caso de uso
   → Provavelmente um dos dois domina
   → Escolha o dominante

⚠️ Se ainda está em dúvida: escolha ["llm"]
   → Mais flexível
   → Usuário pode instruir comportamento fixo via instrução
   → Ex: "Sempre usar 14h" na instrução LLM
```

---

### **Tipo DEPENDÊNCIA (Regra 1):**

Quando parâmetro vem de outra tool:

```
SE valor vem de tool e retorna VALOR ÚNICO:
  → Tipo: Dependência (ocultar parâmetro)
  → Exemplo: person_id sempre vem de @getOrCreatePerson (único resultado)

SE valor vem de tool e retorna ARRAY:
  → Tipo: Dependência (mostrar com critério de seleção)
  → Exemplo: deal_id vem de @getAllExistingDealsFromPerson (múltiplos deals)
```

---

### **Tabela de Decisão Rápida:**

| Campo | API Valida? | Lista Fechada? | Valores Únicos por Contexto? | Tipo Permitido |
|-------|-------------|----------------|------------------------------|----------------|
| `status` | ✅ Sim (rejeita inválidos) | ✅ Sim | ❌ Não | `["fixed"]` |
| `currency` | ✅ Sim (rejeita inválidos) | ✅ Sim | ❌ Não | `["fixed"]` |
| `priority` | ✅ Sim (rejeita inválidos) | ✅ Sim | ❌ Não | `["fixed"]` |
| `pipeline_id` | ✅ Sim (IDs existentes) | ✅ Sim | ❌ Não | `["fixed"]` |
| `stage_id` | ✅ Sim (IDs existentes) | ✅ Sim | ❌ Não | `["fixed"]` |
| `user_id` | ✅ Sim (IDs existentes) | ✅ Sim | ❌ Não | `["fixed"]` |
| `duration` | ❌ Não (time format) | ✅ Sim (presets) | ❌ Não | `["fixed"]` |
| `lost_reason` | ❌ Não (aceita qualquer) | ❌ Não | ✅ Sim | `["llm"]` |
| `note` | ❌ Não (aceita qualquer) | ❌ Não | ✅ Sim | `["llm"]` |
| `description` | ❌ Não (aceita qualquer) | ❌ Não | ✅ Sim | `["llm"]` |
| `title` | ❌ Não (aceita qualquer) | ❌ Não | ✅ Sim | `["llm"]` |
| `fullname` | ❌ Não (aceita qualquer) | ❌ Não | ✅ Sim | `["llm"]` |
| `email` | ❌ Não (formato validado) | ❌ Não | ✅ Sim | `["llm"]` |
| `phone` | ❌ Não (formato validado) | ❌ Não | ✅ Sim | `["llm"]` |
| `value` | ❌ Não (number) | ❌ Não | ✅ Sim | `["llm"]` |
| `probability` | ❌ Não (number) | ❌ Não | ✅ Sim | `["llm"]` |
| `due_date` | ❌ Não (date format) | ❌ Não | ✅ Sim | `["llm"]` |
| `due_time` | ❌ Não (time format) | ❌ Não | ✅ Sim | `["llm"]` |
| `expected_close_date` | ❌ Não (date format) | ❌ Não | ✅ Sim | `["llm"]` |
| `attendees` | ❌ Não (array) | ❌ Não | ✅ Sim | `["llm"]` |

---

### **Fluxograma de Decisão:**

```
┌─────────────────────────────────────────────────────────┐
│ Qual tipo de dado esse parâmetro é?                     │
└──────────────┬──────────────────────────────────────────┘
               │
       ┌───────┴───────┐
       │ É um ID?      │
       └───────┬───────┘
               │
        ┌──────┴──────┐
        │ SIM         │ NÃO
        │             │
        v             v
  ┌─────────┐   ┌──────────────────────┐
  │ ID vem  │   │ API tem lista        │
  │ de API? │   │ fechada de valores?  │
  └────┬────┘   └──────────┬───────────┘
       │                   │
    ✅ SIM            ┌────┴────┐
       │             │ SIM     │ NÃO
       v             │         │
  ["fixed"]          v         v
                ["fixed"]  ┌────────────────────────┐
                           │ Tem valores padrão     │
                           │ (presets) fixos?       │
                           │ Ex: duração (30m, 1h)  │
                           └──────┬─────────────────┘
                                  │
                           ┌──────┴──────┐
                           │ SIM         │ NÃO
                           │             │
                           v             v
                      ["fixed"]      ┌────────────────┐
                                     │ Conteúdo varia │
                                     │ por contexto   │
                                     │ da conversa?   │
                                     └──────┬─────────┘
                                            │
                                         ✅ SIM
                                            │
                                            v
                                       ["llm"]

⚠️ IMPORTANTE: Múltiplos tipos ["llm", "fixed"] foram REMOVIDOS
   → Na prática, sempre há um tipo dominante
   → Escolha o tipo baseado no caso de uso real (90%+ dos casos)
```

---

### **Regra de Ouro (ATUALIZADA v3.5):**

```
1. ID de recurso PRÉ-EXISTENTE (pipeline_id, stage_id, user_id)
   → API só aceita IDs que existem
   → allowed_input_types: ["fixed"]

2. ID de DEPENDÊNCIA (vem de outra tool)
   → allowed_input_types: ["dependency"]
   → Ocultar se valor único, mostrar se array

3. ENUM VALIDADO pela API (status, currency, priority)
   → API REJEITA valores fora da lista
   → allowed_input_types: ["fixed"]

4. STRING LIVRE sem validação (lost_reason, description)
   → API ACEITA qualquer string
   → Conteúdo único por contexto
   → allowed_input_types: ["llm"]

5. TEXTO GERADO (title, content, note, subject)
   → API ACEITA qualquer string
   → Sempre dinâmico e contextual
   → allowed_input_types: ["llm"]

6. TEXTO EXTRAÍDO (fullname, email, phone)
   → API ACEITA qualquer string (com validação de formato)
   → Extraído da conversa
   → allowed_input_types: ["llm"]
   → Sistema de variáveis não disponível ainda

7. DATA/HORA CONTEXTUAL (due_date, due_time, expected_close_date)
   → API ACEITA datas válidas
   → Varia por disponibilidade do cliente na conversa
   → allowed_input_types: ["llm"]
   → Raramente é fixo (90%+ dos casos varia)

8. VALORES CONTEXTUAIS (value, probability)
   → API ACEITA numbers
   → Varia por produto mencionado / engajamento do cliente
   → allowed_input_types: ["llm"]
   → Raramente é fixo (90%+ dos casos varia)

9. PARTICIPANTES CONTEXTUAIS (attendees)
   → API ACEITA arrays
   → Varia por quem foi mencionado na conversa
   → allowed_input_types: ["llm"]
   → Raramente é fixo (90%+ dos casos varia)

10. VALORES COM PRESETS FIXOS (duration)
    → API ACEITA time format
    → Empresa tem padrões claros (30min, 1h, 1h30, 2h)
    → allowed_input_types: ["fixed"]
    → LLM não agrega valor (sem contexto suficiente)

11. BOOLEANO
    → Toggle simples
    → allowed_input_types: ["fixed"]

⚠️ REGRA CRÍTICA: Múltiplos tipos ["llm", "fixed"] foram REMOVIDOS
   → Na prática, sempre há um tipo que domina (90%+ dos casos)
   → Escolha o tipo dominante
   → Se ainda está em dúvida: escolha ["llm"] (mais flexível)
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
  → LLM escolhe baseado na instrução e contexto
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

**Exemplo: `template_id` para buscar um template de email:**

```
┌────────────────────────────────────────┐
│ template_id                            │
│ tipo: LLM Prompt                       │
│                                        │
│ ℹ️ Tool de suporte:                   │
│    @searchEmailTemplates               │
│                                        │
│ ℹ️ Campos disponíveis do Template:     │
│ ┌────────────────────────────────────┐ │
│ │ • id (number)                      │ │
│ │ • name (string)                    │ │
│ │ • subject (string)                 │ │
│ │ • tags (array)                     │ │
│ └────────────────────────────────────┘ │
│                                        │
│ Instrução para LLM:                    │
│ ┌────────────────────────────────────┐ │
│ │ Usar o template que tenha a tag   │ │
│ │ "boas-vindas" e mencione "premium"│ │
│ │ no campo subject                   │ │
│ └────────────────────────────────────┘ │
│                                        │
│ 💡 Você pode usar: id, name, subject, │
│    tags                                │
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
  ✅ Preview de ESTRUTURA (campos) se há tool de suporte
  ❌ NÃO mostrar lista de valores fixos
```

**Exemplo de configuração COM tool de suporte:**

```
Usuário configura template_id tipo LLM:

┌────────────────────────────────┐
│ tipo: LLM Prompt               │
│                                │
│ ℹ️ Tool de suporte:            │
│    @searchEmailTemplates       │
│                                │
│ ℹ️ Campos disponíveis:         │
│ ┌────────────────────────────┐ │
│ │ • id (number)              │ │
│ │ • name (string)            │ │
│ │ • subject (string)         │ │
│ │ • tags (array)             │ │
│ └────────────────────────────┘ │
│                                │
│ Instrução para LLM:            │
│ ┌────────────────────────────┐ │
│ │ Usar o template que tenha │ │
│ │ a tag "onboarding"        │ │
│ └────────────────────────────┘ │
│                                │
│ 💡 Você pode usar: id, name,  │
│    subject, tags               │
└────────────────────────────────┘

Em runtime:
→ Sistema chama @searchEmailTemplates com a instrução do usuário como critério de busca
→ Retorna: [{id:101, name:"Boas-vindas Padrão", subject:"Bem-vindo!", tags:["onboarding"]}, {id:102, name:"Boas-vindas VIP", subject:"Bem-vindo ao VIP!", tags:["onboarding", "vip"]}]
→ LLM recebe dados + instrução do usuário
→ LLM analisa: "template com a tag 'onboarding'"
→ LLM pode refinar a escolha com o contexto da conversa e retornar o ID mais apropriado (ex: 102 se o cliente for VIP)
→ Retorna: 102
→ ✅ COM chamada de tool de suporte
```

**Por que preview de campos é crítico:**
- Sem preview: usuário não sabe que pode usar `name`, `subject`, `tags` etc
- Com preview: pode escrever instruções precisas usando a estrutura real
- Exemplo: "template com a tag 'cancelamento' e que não tenha a tag 'vip'" (sabe que campo `tags` existe)

**Quando tool de suporte é chamada:**

```
SE parâmetro tipo = "LLM Prompt":
  → Verificar se existe tool de suporte associada
  → Exemplo: template_id → @searchEmailTemplates
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

### **📖 Glossário de Campos do Schema**

| Campo | Tipo | Quem Define | Descrição |
|-------|------|-------------|-----------|
| **Identificação** ||||
| `name` | string | Desenvolvedor | Nome técnico do parâmetro (ex: `pipeline_id`) |
| `display_name` | string | Desenvolvedor | Nome amigável mostrado na UI (ex: "Pipeline") |
| `help_text` | string | Desenvolvedor | Texto auxiliar ao lado do nome (ex: "Selecione o(s) pipeline(s)") |
| `type` | string | API | Tipo de dado: `string`, `number`, `boolean`, `array`, `enum` |
| `format` | string | API | Formato específico: `email`, `phone`, `date`, `time`, `datetime` |
| **Visibilidade e Obrigatoriedade** ||||
| `required` | boolean | API | Se é obrigatório pela API externa |
| `visible` | boolean | Desenvolvedor | Se PODE aparecer na UI (false = nunca aparece) |
| `show_by_default` | boolean | Desenvolvedor | Se aparece de cara ou no botão "Adicionar" |
| `is_critical_field` | boolean | Desenvolvedor | Se precisa normalização automática (chave de busca) |
| **Tipos de Preenchimento** ||||
| `allowed_input_types` | array | Desenvolvedor | Tipos permitidos: `["fixed"]`, `["llm"]`, `["dependency"]` |
| `default_type` | string | Desenvolvedor | Tipo padrão quando há múltiplas opções |
| **Configuração Tipo Fixo** ||||
| `fixed_config` | object | Desenvolvedor | Configurações para tipo Fixo |
| `fixed_config.multi_select` | boolean | Desenvolvedor | Se permite selecionar múltiplos valores |
| `fixed_config.api_endpoint` | object | Desenvolvedor | Como carregar valores da API |
| `fixed_config.enum_values` | array | Desenvolvedor | Valores fixos quando não vem da API |
| `fixed_config.hardcoded` | boolean | Desenvolvedor | Se valores são hardcoded no frontend |
| `fixed_config.instruction` | object | Desenvolvedor | Config do campo instrução (quando multi-select) |
| **Configuração Tipo LLM** ||||
| `llm_config` | object | Desenvolvedor | Configurações para tipo LLM |
| `llm_config.instruction_hint` | string | Desenvolvedor | Label do campo de instrução |
| `llm_config.placeholder` | string | Desenvolvedor | Placeholder com exemplo |
| `llm_config.default_instruction` | string | Desenvolvedor | Instrução pré-preenchida |
| `llm_config.support_tool` | string | Desenvolvedor | Tool que fornece contexto (ex: `@getAllPipelines`) |
| `llm_config.null_handling` | object | Desenvolvedor | Como tratar valores nulos da LLM |
| **Configuração Tipo Dependência** ||||
| `dependency_config` | object | Desenvolvedor | Config para tipo Dependência |
| `dependency_config.source_tool` | string | Desenvolvedor | Tool que fornece o valor (ex: `@getAllDeals`) |
| `dependency_config.source_field` | string | Desenvolvedor | Campo da tool fonte (ex: `id`) |
| `dependency_config.output_type` | string | Desenvolvedor | `single_value` ou `array` |
| `dependency_config.selection_required` | boolean | Desenvolvedor | Se usuário precisa definir critério |
| `dependency_config.show_fields_preview` | boolean | Desenvolvedor | Se mostra preview dos campos disponíveis |
| `dependency_config.available_fields` | array | Desenvolvedor | Lista de campos disponíveis com tipos |
| **Dependências e Relacionamentos** ||||
| `dependencies` | array | Desenvolvedor | Tools que devem estar ativas |
| `parameter_relationships` | array | Desenvolvedor | Relação com outros parâmetros da mesma tool |
| **Validação** ||||
| `validation` | object | Desenvolvedor | Regras de validação |
| `validation.min` | number | Desenvolvedor | Valor mínimo |
| `validation.max` | number | Desenvolvedor | Valor máximo |
| `validation.min_length` | number | Desenvolvedor | Tamanho mínimo string |
| `validation.format` | string | Desenvolvedor | Validação de formato (email, phone, etc) |
| `validation.error_message` | string | Desenvolvedor | Mensagem de erro customizada |
| **Normalização** ||||
| `normalization` | object | Desenvolvedor | Regras de normalização automática |
| `normalization.enabled` | boolean | Desenvolvedor | Se normalização está ativa |
| `normalization.auto_apply` | boolean | Sistema | Se aplica automaticamente |
| `normalization.rules` | array | Desenvolvedor | Lista de regras (ex: `remove_whitespace`) |
| **Null/Empty Handling** ||||
| `nullable_behavior` | object | Desenvolvedor | Como tratar valores vazios |
| `nullable_behavior.empty_converts_to` | any | Desenvolvedor | Converter string vazia para que? |
| `nullable_behavior.send_when_null` | boolean | Desenvolvedor | Enviar parâmetro quando null? |
| `nullable_behavior.zero_is_valid` | boolean | Desenvolvedor | Zero é valor válido? (numbers) |
| **UI Indicators** ||||
| `ui_indicators` | object | Desenvolvedor | Indicadores visuais na UI |
| `ui_indicators.badge` | string | Desenvolvedor | Badge a mostrar (ex: "🔴 Campo Crítico") |
| `ui_indicators.warning` | string | Desenvolvedor | Aviso importante ao usuário |
| `ui_indicators.hidden` | boolean | Desenvolvedor | Se está oculto mesmo quando visible=false |
| `ui_indicators.dependency_badge` | string | Desenvolvedor | Badge de dependência (ex: "🔗 Resolvido por @tool") |

---

### **Exemplo de Schema Completo:**

```json
{
  "name": "pipeline_id",
  "display_name": "Pipeline",
  "help_text": "Selecione o(s) pipeline(s)",
  "type": "number",
  
  "required": false,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["fixed"],
  "default_type": "fixed",
  
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

## 🎯 **Regra 10: Visibilidade Padrão dos Parâmetros**

**Audiência:** Desenvolvedor definindo schema  
**Aplicação:** Automática pelo sistema  
**Visível ao usuário:** Quais campos aparecem de cara vs. botão "Adicionar"

Define quais parâmetros aparecem imediatamente na UI e quais ficam ocultos até o usuário adicionar.

### **Dois Níveis de Controle:**

```
visible: false
  → NUNCA aparece (nem em "Adicionar parâmetros opcionais")
  → Exemplo: person_id (resolvido por dependência única)
  
visible: true + show_by_default: false
  → PODE aparecer, mas começa oculto
  → Disponível em "Adicionar parâmetros opcionais"
  → Exemplo: probability, currency, expected_close_date
  
visible: true + show_by_default: true
  → Aparece de cara na UI principal
  → Exemplo: pipeline_id, stage_id, title
  
required: true
  → SEMPRE aparece (forçado)
  → Ignora show_by_default
  → Exemplo: title, subject, deal_id
```

### **Quando usar show_by_default: true**

✅ **Mostrar de cara quando:**
- Parâmetro é importante para configurar (mesmo sendo opcional)
- Decisão de negócio comum (pipeline, stage, responsável)
- Melhora UX ter visível desde o início

**Exemplos:**
- `pipeline_id` - opcional, mas importante definir
- `stage_id` - opcional, mas útil configurar
- `user_id` - opcional, mas comum atribuir responsável

### **Quando usar show_by_default: false**

✅ **Ocultar inicialmente quando:**
- Parâmetro é avançado ou raramente usado
- Caso de uso específico (não 90%+ dos casos)
- UX mais limpa sem ele

**Exemplos:**
- `probability` - útil, mas avançado
- `currency` - tem padrão, raramente muda
- `expected_close_date` - opcional e específico
- `note` - complementar, não essencial

### **Schema Atualizado:**

```json
{
  "name": "probability",
  "display_name": "Probabilidade",
  "help_text": "Estimativa de fechamento (0-100%)",
  "type": "number",
  
  "required": false,
  "visible": true,
  "show_by_default": false,
  
  "allowed_input_types": ["llm"]
}
```

### **Comportamento na UI:**

**Parâmetros Visíveis (show_by_default: true):**

```
┌────────────────────────────────────────┐
│ @createDeal                            │
├────────────────────────────────────────┤
│                                        │
│ #title *                               │
│ ┌────────────────────────────────────┐ │
│ │ ...                                │ │
│ └────────────────────────────────────┘ │
│                                        │
│ #pipeline_id                           │
│ ┌────────────────────────────────────┐ │
│ │ ...                                │ │
│ └────────────────────────────────────┘ │
│                                        │
│ #stage_id                              │
│ ┌────────────────────────────────────┐ │
│ │ ...                                │ │
│ └────────────────────────────────────┘ │
│                                        │
├────────────────────────────────────────┤
│ [+] Adicionar parâmetros opcionais (3) │
└────────────────────────────────────────┘
```

**Modal "Adicionar Parâmetros" (show_by_default: false):**

```
┌────────────────────────────────────────┐
│ Parâmetros Opcionais Disponíveis:     │
│                                        │
│ ☐ probability                          │
│    Estimativa de fechamento (0-100%)  │
│                                        │
│ ☐ currency                             │
│    Moeda do deal                       │
│                                        │
│ ☐ expected_close_date                  │
│    Data esperada de fechamento         │
│                                        │
│ [✓ Adicionar Selecionados]            │
└────────────────────────────────────────┘
```

### **Matriz de Decisão Completa:**

| Parâmetro | `visible` | `show_by_default` | `required` | Resultado na UI |
|-----------|-----------|-------------------|-----------|-----------------|
| `person_id` | `false` | - | `false` | ❌ NUNCA aparece |
| `title` | `true` | - | `true` | ✅ Visível + * obrigatório |
| `pipeline_id` | `true` | `true` | `false` | ✅ Visível sem * |
| `probability` | `true` | `false` | `false` | ⚪ Oculto → botão "Adicionar" |

### **Lógica de Implementação:**

```javascript
function shouldShowParameter(param) {
  // Se não é visível, nunca mostra
  if (param.visible === false) {
    return false;
  }
  
  // Se é obrigatório, sempre mostra
  if (param.required === true) {
    return true;
  }
  
  // Se é opcional, depende da flag
  return param.show_by_default === true;
}

function getAvailableToAdd(params) {
  return params.filter(p => 
    p.visible === true &&
    p.required === false &&
    p.show_by_default === false &&
    !p.currently_added
  );
}
```

---

## 🔴 **Regra 11: Campos Críticos**

**Audiência:** Desenvolvedor definindo schema  
**Aplicação:** Automática pelo sistema  
**Visível ao usuário:** Badge "🔴 Campo Crítico" + texto explicativo

Define quais campos são críticos por (1) serem chaves de busca que precisam normalização automática OU (2) serem identificadores para operações importantes/destrutivas.

### **Quando marcar como crítico:**

```
is_critical_field: true

✅ TIPO 1: Chaves de Busca (requerem normalização)
  → Campo usado para buscar/identificar recursos existentes
  → Formato incorreto causa duplicação de dados
  → Exemplos: phone, email
  → Sistema aplica normalização automática
  
✅ TIPO 2: Identificadores de Operações (sem normalização)
  → IDs usados para identificar recurso em operações destrutivas/importantes
  → Formato incorreto pode atualizar/deletar recurso errado
  → Exemplos: deal_id (updateDeal), note_id (updateNote)
  → Sistema não normaliza (já é ID correto), mas marca como crítico na UI
  
❌ NÃO usar quando:
  → Campo é importante mas não afeta integridade de dados
  → Campo não é usado para buscar nem identificar recursos
  → Exemplo: title, value, stage_id
```

### **Comportamento por Tipo:**

**TIPO 1: Chave de Busca (com normalização)**

```json
{
  "name": "phone",
  "is_critical_field": true,
  
  "normalization": {
    "enabled": true,
    "auto_apply": true,
    "rules": [
      "remove_whitespace",
      "remove_special_chars",
      "add_country_code_if_missing",
      "format_e164"
    ]
  },
  
  "ui_indicators": {
    "badge": "🔴 Campo Crítico",
    "warning": "Campo usado para buscar pessoas existentes. Sistema normaliza formato automaticamente."
  }
}
```

**TIPO 2: Identificador de Operação (sem normalização)**

```json
{
  "name": "deal_id",
  "is_critical_field": true,
  
  "normalization": {
    "enabled": false
  },
  
  "ui_indicators": {
    "badge": "🔴 Campo Crítico",
    "warning": "Campo usado para identificar qual deal será atualizado. Certifique-se de que o critério de seleção está correto."
  }
}
```

### **Exemplo Visual:**

```
┌─────────────────────────────────────────────────┐
│ #phone * 🔴 Campo Crítico                       │
│                                                 │
│ Instrução para LLM:                             │
│ ┌─────────────────────────────────────────────┐ │
│ │ Extrair telefone da conversa                │ │
│ └─────────────────────────────────────────────┘ │
│                                                 │
│ 🔴 Campo usado para buscar pessoas existentes. │
│    Sistema normaliza formato automaticamente   │
│    para padrão internacional (+5511987654321). │
│                                                 │
│ ⚙️ Normalizações aplicadas automaticamente:    │
│    • Remove espaços e caracteres especiais     │
│    • Adiciona código do país se ausente        │
│    • Formata para padrão E.164                 │
└─────────────────────────────────────────────────┘
```

### **Exemplos de Campos Críticos:**

**TIPO 1 (com normalização):**
- `phone` - chave de busca para pessoa, requer normalização E.164
- `email` - chave de busca, requer normalização lowercase/trim

**TIPO 2 (sem normalização):**
- `deal_id` - identificador para updateDeal (operação destrutiva)
- `note_id` - identificador para updateNote (operação de atualização)

### **Diferença: Crítico vs. Obrigatório**

```
required: true
  → API EXIGE o campo
  → Validação: não pode ser vazio
  → Badge: * Obrigatório

is_critical_field: true
  → Campo é CHAVE DE BUSCA/IDENTIFICAÇÃO
  → Sistema aplica normalização automática
  → Badge: 🔴 Campo Crítico
  → Pode ser required ou optional

Pode ter ambos:
  required: true + is_critical_field: true
  → Exemplo: phone em getOrCreatePerson
```

---

## 📭 **Regra 12: Null e Empty Handling**

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

## 🏷️ **Regra 13: Categorias de Tools (Integração com Regra 1)**

**Audiência:** Desenvolvedor definindo schema  
**Aplicação:** Automática pelo sistema  
**Visível ao usuário:** Apenas resultado (quais tools aparecem na lista @)

> **Esta regra consolida e expande a Regra 1 (Visibilidade de Tools) com detalhes de implementação**

Separar tools por função para melhor organização da UX.

**Resumo de Categorias (baseado na Regra 1):**

| Categoria | Visível @ | Padrão Naming | Quando Executa |
|-----------|-----------|---------------|----------------|
| **Ação** | ✅ SIM | `create`, `update`, `delete`, `add` | Explícito pelo usuário | `@createDeal` |
| **Consulta Visível** | ✅ SIM | `get*` | Explícito pelo usuário | `@getDealWithCompleteInfo` |
| **Híbrida** | ✅ SIM | `getOrCreate*` | Explícito pelo usuário | `@getOrCreatePerson` |
| **Consulta de Dependência** | ❌ NÃO | `get*`, `getAll*` | Por dependência de outra tool | `@getAllExistingDealsFromPerson` |
| **Suporte LLM** | ❌ NÃO | `getAll*` | Em runtime para contexto LLM | `@getAllExistingPipelines` |

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

### **Categoria B: Tools Híbridas (User-Facing)**

**Características:**
- Padrão de nome `get*` mas com potencial de escrita (`getOrCreate*`).
- Aparecem na lista @ porque podem modificar dados, exigindo configuração explícita do usuário.
- O sistema verifica se o recurso existe; se não, cria um novo.

**Exemplos:**
- `@getOrCreatePerson`

**Configuração:**

```json
{
  "tool": "@getOrCreatePerson",
  "category": "hybrid",
  "user_configurable": true,
  "show_in_checkpoint_menu": true,
  "icon": "👤"
}
```

### **Categoria C: Consulta Visível (User-Facing)**

**Características:**
- Buscam informações sem modificar dados (`read-only`).
- O usuário pode querer executá-las explicitamente para obter contexto ou verificar informações.
- Aparecem na lista @ com um ícone de "busca" (🔍).

**Exemplos:**
- `@getDealWithCompleteInfo`
- `@getActivitiesFromDeal`

**Configuração:**

```json
{
  "tool": "@getDealWithCompleteInfo",
  "category": "query_visible",
  "user_configurable": true,
  "show_in_checkpoint_menu": true,
  "icon": "🔍",
  "read_only": true
}
```

### **Categoria D: Suporte LLM (Automática)**

**Características:**
- Fornecem contexto para LLM em runtime.
- Chamadas automaticamente quando um parâmetro do tipo "LLM Prompt" precisa de dados externos.
- **NÃO** aparecem na lista de @tools do checkpoint.
- Não têm configuração visível para usuário.

**Exemplos:**
- `@getAllExistingPipelines`
- `@getAllUsers` (hipotético)

**Configuração:**

```json
{
  "tool": "@getAllExistingPipelines",
  "category": "support_llm",
  "user_configurable": false,
  "show_in_checkpoint_menu": false,
  "trigger": "runtime_for_llm_context",
  "provides_context_for": [
    {
      "tool": "@searchEmailTemplates",
      "parameter": "template_id",
      "note": "Exemplo genérico - pipeline_id usa tipo fixed (carrega via API frontend)"
    }
  ]
}
```

### **Categoria E: Consulta de Dependência (Automática)**

**Características:**
- Buscam dados que são pré-requisito para outra tool.
- São invocadas automaticamente quando uma tool de "Ação" precisa de um ID ou outra informação.
- **NÃO** aparecem na lista de @tools do checkpoint.
- O usuário não as configura diretamente, mas sim o critério de seleção no parâmetro da tool principal.

**Exemplos:**
- `@getAllExistingDealsFromPerson` (chamada por `@updateDeal` para obter o `deal_id`)

**Configuração:**

```json
{
  "tool": "@getAllExistingDealsFromPerson",
  "category": "query_dependency",
  "user_configurable": false,
  "show_in_checkpoint_menu": false,
  "trigger": "dependency_for_action_tool",
  "invoked_by": [
    {
      "tool": "@updateDeal",
      "parameter": "deal_id"
    }
  ]
}
```

### **Indicadores Visuais por Categoria:**

```
Lista de @tools disponíveis (o que usuário vê):

[AÇÕES]
  💼 @createDeal - Criar novo deal
  ✏️ @updateDeal - Atualizar deal existente
  📝 @createNote - Adicionar nota
  👤 @getOrCreatePerson - Buscar ou criar pessoa

[CONSULTAS]
  🔍 @getDealWithCompleteInfo - Buscar detalhes do deal
  🔍 @getActivitiesFromDeal - Listar atividades

[AUTOMÁTICAS (OCULTAS)]
  → @getAllExistingPipelines (chamada em runtime para LLM)
  → @getAllExistingDealsFromPerson (chamada por dependência)
```

**Como usuário sabe que tool de suporte existe?**

```
Na configuração do parâmetro tipo "LLM":

┌────────────────────────────────┐
│ #template_id                   │
│ tipo: LLM Prompt               │
│                                │
│ ℹ️ LLM terá acesso automático │
│    aos templates de e-mail     │
│    via @searchEmailTemplates   │
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
   └─ NÃO → visible: false (Regra 2)
       → Exemplo: person_id (dependência única)
   └─ SIM → visible: true, continue...

2. Deve aparecer de cara ou no botão "Adicionar"? (Regra 10 - Sistema)
   └─ De cara → show_by_default: true
       → Exemplo: pipeline_id, stage_id, user_id
   └─ Oculto → show_by_default: false
       → Exemplo: probability, currency, note

3. É campo crítico (chave de busca)? (Regra 11 - Sistema)
   └─ SIM → is_critical_field: true
       → Adicionar normalização automática
       → Exemplo: phone, email, deal_id
   └─ NÃO → is_critical_field: false
       → Exemplo: title, value, stage_id

4. Como tratar null/empty? (Regra 12 - Sistema)
   └─ Definir nullable_behavior no schema
   └─ Sistema aplica automaticamente

5. Qual categoria da tool? (Regra 13 - Sistema)
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

10. **Visibilidade Padrão** - Controla se parâmetro aparece de cara ou no botão "Adicionar"
11. **Campos Críticos** - Identificar campos que requerem normalização automática
12. **Null e Empty Handling** - Como tratar valores vazios
13. **Categorias de Tools** - Integração com Regra 1 (Ação/Consulta/Suporte/Híbrida)

---

**Versão:** 3.7.1  
**Data:** 2025-11-10  
**Status:** Pronto para implementação

**Changelog:**
- v3.7.1: **CORREÇÕES DE INCONSISTÊNCIAS**
  - ✅ Regra 1: Adicionada categoria "Consulta Visível" (exceção ao padrão GET)
  - ✅ Regra 11: Clarificado que `is_critical_field` tem 2 tipos:
    - TIPO 1: Chaves de busca (phone, email) - com normalização
    - TIPO 2: Identificadores de operações (deal_id, note_id) - sem normalização
  - ✅ Removida referência incorreta a `pipeline_id` tipo LLM em `getAllExistingPipelines`
  - ✅ Atualizada linha 206: `criticality` → `show_by_default` + `is_critical_field`
  - ✅ Tabela de padrões de naming atualizada com consulta visível
  - **Impacto:** Documentação mais precisa, sem contradições entre regras
- v3.7: **REFORMULAÇÃO DE CRITICIDADE → VISIBILIDADE PADRÃO**
  - **Motivação:** `criticality` (critical/important/complementary) estava confuso e misturava conceitos diferentes
  - **Mudanças:**
    - ❌ Removido campo `criticality` com 3 níveis (critical/important/complementary)
    - ✅ Adicionado campo `show_by_default` (boolean) - controla se aparece de cara ou no botão "Adicionar"
    - ✅ Mantido campo `visible` (boolean) - controla se PODE aparecer (person_id = false)
    - ✅ Novo campo `is_critical_field` (boolean) - apenas para normalização automática
    - ✅ Adicionado campo `help_text` - texto auxiliar ao lado do display_name
  - **Regra 10 Reescrita:** Agora trata de Visibilidade Padrão (show_by_default)
  - **Nova Regra 11:** Campos Críticos (is_critical_field) - apenas para normalização
  - **Antigas Regras 11 e 12 → 12 e 13:** Renumeradas
  - **help_text adicionado ao schema:** Permite textos auxiliares como "Selecione o(s) pipeline(s)"
  - **Impacto:** Separação clara entre:
    1. Obrigatoriedade técnica (`required`)
    2. Relevância na UX (`show_by_default`)
    3. Criticidade para integridade (`is_critical_field`)
  - **Exemplo:**
    - `pipeline_id`: `required: false`, `show_by_default: true`, `is_critical_field: false`
    - `probability`: `required: false`, `show_by_default: false`, `is_critical_field: false`
    - `person_id`: `required: false`, `visible: false`, nunca aparece
    - `phone`: `required: true`, `show_by_default: true`, `is_critical_field: true` (normalização)
- v3.6: **REMOÇÃO DE ENUM_VALUES PARA TIPO LLM**
  - **Motivação:** Enum Values restringem a liberdade da LLM sem agregar valor real
  - **Mudanças:**
    - Removido campo `enum_values` de `llm_config` em todos os schemas
    - Removida seção "C. Enum Values para LLM" da documentação
    - Removida seção "Comportamento do Enum Values (opcional)"
    - Simplificada UI de configuração do tipo LLM (apenas instrução + preview de campos)
    - Atualizado diagrama de schema completo (removido "enum" de llm_config)
    - **Árvore de Decisões atualizada:**
      - Removida caixa "Enum Values? (opcional)" do fluxo principal
      - Removida linha "[+] Enum Values (opcional)" da UI de tipo LLM
      - Removida seção "Restringir opções com Enum Values?" do guia de usuário
    - **Seção B.3 atualizada:** "Enum Sugerido" marcado como NÃO USAR (removido em v3.5)
  - **Impacto:** LLM agora tem liberdade total para gerar/escolher valores baseado em contexto e instruções
  - **Justificativa:** Se há necessidade de restringir valores, usar tipo `["fixed"]` com multi-select
- v3.5: **REMOÇÃO DE MÚLTIPLOS TIPOS - SIMPLIFICAÇÃO PRÁTICA**
  - **Regra 3 Atualizada:** Matriz de decisão agora reflete casos reais do Pipedrive
    - Removidos `["llm", "fixed"]` de Data/Hora, Valor numérico e Array de objetos
    - Adicionadas linhas específicas: Data contextual, Valor contextual, Participantes contextuais, Duração com presets
    - Cada tipo de dado agora tem apenas UM tipo permitido baseado no caso de uso dominante (90%+ dos casos)
  - **Nova Seção E:** "Quando NÃO Permitir Múltiplos Tipos" com exemplos práticos
    - Datas e horários → sempre `["llm"]` (contextuais)
    - Valores e probabilidades → sempre `["llm"]` (contextuais)
    - Participantes → sempre `["llm"]` (contextuais)
    - Duração → sempre `["fixed"]` (presets da empresa)
  - **Tabela de Decisão Rápida:** Expandida com todos os parâmetros do Pipedrive
    - Incluídos: duration, value, probability, due_date, due_time, expected_close_date, attendees
    - Todos com tipo único definido
  - **Fluxograma de Decisão:** Atualizado com nova ramificação "Tem valores padrão (presets) fixos?"
    - Remove opção de múltiplos tipos
    - Aviso crítico sobre remoção de `["llm", "fixed"]`
  - **Regra de Ouro:** Atualizada para v3.5 com 11 categorias (antes eram 8)
    - Item 7: Data/hora contextual (apenas LLM)
    - Item 8: Valores contextuais (apenas LLM)
    - Item 9: Participantes contextuais (apenas LLM)
    - Item 10: Valores com presets fixos (apenas Fixo)
    - Aviso crítico sobre remoção de múltiplos tipos
  - **Justificativa:** Análise prática mostrou que múltiplos tipos complicam UX sem agregar valor real
- v3.4: **REESTRUTURAÇÃO DA REGRA 12 PARA CONSISTÊNCIA**
  - **Corpo da Regra 12 Alinhado com o Resumo:** O texto detalhado da Regra 12 foi completamente reescrito para corresponder à tabela de 5 categorias (Ação, Híbrida, Consulta Visível, Suporte LLM, Consulta de Dependência).
  - **Contradição Removida:** A inconsistência onde `@getAllExistingDealsFromPerson` era listado como uma consulta visível foi corrigida. Agora está corretamente categorizado como uma dependência automática e oculta.
  - **Clareza Adicional:** A nova estrutura torna a distinção entre ferramentas visíveis (configuradas pelo usuário) e automáticas (invocadas pelo sistema) muito mais explícita em todo o documento.
- v3.3: **ALINHAMENTO COM DIRETRIZ DE IDS ESTÁVEIS VS. DINÂMICOS**
  - **Regra 3, 5, 6 e 9 Alinhadas:** O documento foi corrigido para seguir a regra de que IDs de recursos estáveis e pré-existentes (como `pipeline_id`) devem ser exclusivamente do tipo `Fixo`.
  - **Exemplos Corrigidos:** Os exemplos que usavam `pipeline_id` para ilustrar o tipo `LLM` com tool de suporte foram substituídos por um caso de uso mais adequado (`template_id`), que representa uma busca dinâmica em uma lista grande e variável.
  - **Schema Universal (Regra 9) Simplificado:** O schema de `pipeline_id` agora reflete corretamente `allowed_input_types: ["fixed"]` e não inclui mais a configuração `llm_config`, eliminando a inconsistência.
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
