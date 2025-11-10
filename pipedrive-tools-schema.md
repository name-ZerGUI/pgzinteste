# 🔧 Pipedrive Tools - Schema Completo v2.3

Schemas de todas as tools do Pipedrive seguindo o **Framework de Regras para Parâmetros** (v3.7).

**Data:** 2025-11-10  
**Integração:** Pipedrive  
**Total de Tools:** 10

---

## 💡 **Nota Importante sobre Exemplos Visuais**

Nos exemplos visuais deste documento:

✅ **Quando há APENAS 1 tipo permitido** (`allowed_input_types` com 1 elemento):
- **NÃO** aparece seletor de tipo na UI
- Campos são mostrados **diretamente**
- Exemplo: `allowed_input_types: ["llm"]` → mostra campo de instrução diretamente

❌ **Quando há MÚLTIPLOS tipos** (`allowed_input_types` com 2+ elementos):
- **APARECE** seletor de tipo `[Fixo | LLM Prompt]`
- Usuário escolhe o tipo antes de configurar
- Exemplo: `allowed_input_types: ["llm", "fixed"]` → mostra seletor

**Referência:** Regra 3 do Framework - "UI deve mostrar campos diretamente quando há apenas 1 tipo"

---

## 📚 **Como Ler Este Documento**

### **Estrutura de Cada Tool**

```
1. Categoria e Visibilidade (visível @ ou automática)
2. Metadados da Tool (nome, descrição, dependências)
3. Schema de Parâmetros
   ├─ #parametro1 (JSON schema completo)
   │  └─ Exemplo Visual (como aparece na UI)
   ├─ #parametro2
   └─ #parametro3
```

### **Cores e Símbolos**
- 🔴 **Campo Crítico** - Requer normalização automática (chave de busca)
- 🟡 **Obrigatório** - API exige este campo
- ⚪ **Opcional** - Pode ser deixado vazio
- 🔗 **Dependência** - Vem de outra tool
- ℹ️ **Informação** - Contexto adicional

---

## 📖 **Glossário de Campos do Schema**

Cada parâmetro usa estes campos:

| Campo | Descrição | Exemplo |
|-------|-----------|---------|
| **Identificação** |||
| `name` | Nome técnico do parâmetro | `pipeline_id` |
| `display_name` | Nome mostrado na UI | `"Pipeline"` |
| `help_text` | Texto auxiliar ao lado do nome | `"Selecione o(s) pipeline(s)"` |
| `type` | Tipo de dado da API | `string`, `number`, `boolean`, `array` |
| `format` | Formato específico | `email`, `phone`, `date`, `time` |
| **Visibilidade** |||
| `required` | Obrigatório pela API? | `true` / `false` |
| `visible` | PODE aparecer na UI? | `true` = sim, `false` = nunca |
| `show_by_default` | Aparece de cara? | `true` = visível, `false` = botão "Adicionar" |
| `is_critical_field` | Precisa normalização? | `true` = chave de busca (phone) |
| **Tipos de Preenchimento** |||
| `allowed_input_types` | Tipos permitidos | `["fixed"]`, `["llm"]`, `["dependency"]` |
| `default_type` | Tipo padrão | `"fixed"` |
| **Tipo Fixo** |||
| `fixed_config` | Configurações tipo Fixo | `{ multi_select, api_endpoint, ... }` |
| `fixed_config.multi_select` | Permite múltiplos valores? | `true` / `false` |
| `fixed_config.api_endpoint` | Como carregar valores | `{ method: "GET", url: "..." }` |
| `fixed_config.enum_values` | Valores hardcoded | `[{value, label}, ...]` |
| `fixed_config.instruction` | Config instrução multi-select | `{ required_when_multi: true }` |
| **Tipo LLM** |||
| `llm_config` | Configurações tipo LLM | `{ instruction_hint, placeholder, ... }` |
| `llm_config.instruction_hint` | Label do campo instrução | `"Como a LLM deve..."` |
| `llm_config.placeholder` | Exemplo no placeholder | `"Ex: Extrair nome..."` |
| `llm_config.support_tool` | Tool que dá contexto | `"@getAllPipelines"` |
| **Tipo Dependência** |||
| `dependency_config` | Config tipo Dependência | `{ source_tool, output_type, ... }` |
| `dependency_config.source_tool` | Tool que fornece valor | `"@getAllDeals"` |
| `dependency_config.output_type` | Retorna único ou array? | `"single_value"` / `"array"` |
| `dependency_config.show_fields_preview` | Mostrar campos disponíveis? | `true` / `false` |
| `dependency_config.available_fields` | Lista de campos | `[{name, type, description}, ...]` |
| **Relações** |||
| `dependencies` | Tools que devem estar ativas | `[{tool, type, field}, ...]` |
| `parameter_relationships` | Relação com outros params | `[{depends_on_parameter, ...}, ...]` |
| **Validação** |||
| `validation` | Regras de validação | `{ min, max, format, ... }` |
| `validation.min` / `max` | Valor mínimo/máximo | `0`, `100` |
| `validation.error_message` | Mensagem de erro | `"Campo obrigatório"` |
| **Normalização** |||
| `normalization` | Normalização automática | `{ enabled, rules }` |
| `normalization.rules` | Regras aplicadas | `["remove_whitespace", ...]` |
| **Null/Empty** |||
| `nullable_behavior` | Como tratar vazios | `{ empty_converts_to, send_when_null }` |
| `nullable_behavior.send_when_null` | Enviar quando null? | `true` / `false` |
| **UI** |||
| `ui_indicators` | Indicadores visuais | `{ badge, warning, hidden }` |
| `ui_indicators.badge` | Badge a mostrar | `"🔴 Campo Crítico"` |
| `ui_indicators.warning` | Aviso importante | `"Formato incorreto causa duplicatas"` |
| `ui_indicators.hidden` | Oculto permanentemente? | `true` / `false` |

**Legenda Rápida:**
- 🔴 = Campo crítico (requer normalização)
- ⚪ = Opcional
- \* = Obrigatório
- 🔗 = Dependência de outra tool
- ℹ️ = Informação adicional

---

## 📋 **Índice de Tools**

### **🎯 Tools Visíveis (Aparecem na lista @)**

**Categoria: Ação**
1. [createDeal](#1-createdeal-) - Criar novo deal
2. [updateDeal](#2-updatedeal-) - Atualizar deal existente
3. [createNote](#3-createnote-) - Criar nova nota
4. [updateNote](#4-updatenote-) - Atualizar nota existente
5. [createDealActivity](#5-createdealactivity-) - Criar atividade

**Categoria: Híbrida**
6. [getOrCreatePerson](#6-getorcreateperson-) - Buscar ou criar pessoa

**Categoria: Consulta Visível**
7. [getDealWithCompleteInfo](#7-getdealwithcompleteinfo-) - Buscar deal completo
8. [getActivitiesFromDeal](#8-getactivitiesfromdeal-) - Buscar atividades

### **⚙️ Tools Automáticas (Ocultas)**

**Categoria: Suporte LLM**
9. [getAllExistingPipelines](#9-getallexistingpipelines-) - Contexto para LLM

**Categoria: Consulta de Dependência**
10. [getAllExistingDealsFromPerson](#10-getallexistingdealsfromperson-) - Busca automática

---

## 1. createDeal 💼 

### **Categoria e Visibilidade**

```json
{
  "category": "action",
  "visible_in_checkpoint": true,
  "user_configurable": true,
  "icon": "💼",
  "show_in_list": true
}
```

### **Metadados da Tool**

```json
{
  "name": "createDeal",
  "display_name": "Criar novo deal",
  "description": "Cria um novo deal no Pipedrive com as informações fornecidas.",
  "integration": "pipedrive",
  "dependencies": [
    {
      "tool": "getOrCreatePerson",
      "type": "required",
      "reason": "Precisa do person_id para associar o deal"
    }
  ]
}
```

### **Schema de Parâmetros**

#### **#title**

```json
{
  "name": "title",
  "display_name": "Título do deal",
  "help_text": "Crie um título descritivo",
  "type": "string",
  
  "required": true,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Como a LLM deve gerar o título?",
    "placeholder": "Ex: Usar nome da pessoa + tipo de produto mencionado",
    "default_instruction": "Criar título usando o nome da pessoa ou empresa",
    "support_tool": null
  },
  
  "validation": {
    "min_length": 1,
    "error_message": "Título é obrigatório"
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ Título do deal * — Crie um título      │
│                    descritivo          │
│                                        │
│ Instrução para LLM:                    │
│ ┌────────────────────────────────────┐ │
│ │ Criar título usando o nome da      │ │
│ │ pessoa + tipo de produto           │ │
│ └────────────────────────────────────┘ │
│                                        │
│ 💡 Dica: Use informações da conversa  │
│    para criar um título descritivo    │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (LLM) -    │
│    campos mostrados diretamente       │
└────────────────────────────────────────┘
```

---

#### **#person_id**

```json
{
  "name": "person_id",
  "display_name": "Pessoa",
  "type": "number",
  
  "required": true,
  "visible": false,
  "show_by_default": false,
  "is_critical_field": true,
  
  "allowed_input_types": ["dependency"],
  "default_type": "dependency",
  
  "dependencies": [
    {
      "tool": "getOrCreatePerson",
      "field": "id",
      "type": "required",
      "output_type": "single_value",
      "auto_resolve": true
    }
  ],
  
  "validation": {
    "required": true,
    "error_message": "🔴 Tool @getOrCreatePerson deve estar ativa"
  },
  
  "ui_indicators": {
    "hidden": true,
    "dependency_badge": "🔗 Resolvido por @getOrCreatePerson"
  }
}
```

**Exemplo Visual:**

```
❌ NÃO APARECE NA UI

Razão: Regra 2A - Dependência única (valor singular)
→ @getOrCreatePerson retorna sempre 1 pessoa
→ Não há escolha a fazer
→ Sistema resolve automaticamente
```

---

#### **#pipeline_id**

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
      "url": "/api/integrations/pipedrive/pipelines",
      "trigger": "on_modal_open",
      "frontend_call": true,
      "response_mapping": {
        "value_field": "id",
        "label_field": "name"
      }
    },
    "instruction": {
      "required_when_multi": true,
      "placeholder": "Quando usar cada pipeline selecionado?",
      "validation": "required_if_multiple_selected"
    }
  },
  
  "validation": {
    "min_selections": 0,
    "error_message": "Selecione ao menos um pipeline ou deixe vazio para usar o padrão"
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

**Exemplo Visual - Single Select:**

```
┌────────────────────────────────────────┐
│ Pipeline — Selecione o(s) pipeline(s)  │
│ ⚪ Opcional                            │
│                                        │
│ Valores disponíveis:                   │
│ ☐ 1 - Pipeline Vendas                 │
│ ☑ 2 - Pipeline VIP                    │ ← selecionou 1
│ ☐ 3 - Pipeline Inbound                │
│                                        │
│ ✅ Será enviado: Pipeline VIP (2)     │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (Fixo) -   │
│    lista mostrada diretamente         │
└────────────────────────────────────────┘
```

**Exemplo Visual - Multi-Select:**

```
┌────────────────────────────────────────┐
│ Pipeline — Selecione o(s) pipeline(s)  │
│ ⚪ Opcional                            │
│                                        │
│ Valores disponíveis:                   │
│ ☑ 1 - Pipeline Vendas                 │ ← selecionou
│ ☑ 2 - Pipeline VIP                    │ ← selecionou
│ ☐ 3 - Pipeline Inbound                │
│                                        │
│ ⚠️ Você selecionou múltiplos valores   │
│                                        │
│ Instrução (obrigatória): *             │
│ ┌────────────────────────────────────┐ │
│ │ Se cliente mencionar "premium" ou  │ │
│ │ "vip", usar Pipeline VIP (2).      │ │
│ │ Caso contrário, usar Pipeline      │ │
│ │ Vendas (1).                        │ │
│ └────────────────────────────────────┘ │
│                                        │
│ ✅ LLM escolherá entre os valores     │
│    selecionados usando a instrução    │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (Fixo) -   │
│    lista mostrada diretamente         │
└────────────────────────────────────────┘
```

---

#### **#stage_id**

```json
{
  "name": "stage_id",
  "display_name": "Estágio",
  "type": "number",
  "required": false,
  "visible": true,
  "show_by_default": false,
  "is_critical_field": false,
  
  "allowed_input_types": ["fixed"],
  "default_type": "fixed",
  
  "fixed_config": {
    "multi_select": false,
    "api_endpoint": {
      "method": "GET",
      "url": "/api/integrations/pipedrive/stages",
      "trigger": "on_pipeline_selected",
      "frontend_call": true,
      "params_from": ["pipeline_id"],
      "response_mapping": {
        "value_field": "id",
        "label_field": "name"
      }
    }
  },
  
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
  ],
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ #stage_id                              │
│ ⚪ Opcional                            │
│ 🔗 Relacionado a: Pipeline VIP         │
│                                        │
│ ⏳ Carregando estágios...              │
│                                        │
│ Valores disponíveis:                   │
│ ☐ 101 - Contato inicial               │
│ ☑ 102 - Qualificação                  │ ← selecionou
│ ☐ 103 - Proposta                      │
│ ☐ 104 - Negociação                    │
│                                        │
│ ✅ Será enviado: Qualificação (102)   │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (Fixo) -   │
│    lista mostrada diretamente         │
└────────────────────────────────────────┘
```

---

#### **#user_id**

```json
{
  "name": "user_id",
  "display_name": "Responsável (owner)",
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
      "url": "/api/integrations/pipedrive/users",
      "trigger": "on_modal_open",
      "frontend_call": true,
      "response_mapping": {
        "value_field": "id",
        "label_field": "name"
      }
    },
    "instruction": {
      "required_when_multi": true,
      "placeholder": "Quando usar cada responsável?"
    }
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ #user_id                               │
│ ⚪ Opcional                            │
│                                        │
│ Valores disponíveis:                   │
│ ☑ 10 - João Silva (vendedor)          │
│ ☐ 11 - Maria Santos (gerente)         │
│ ☐ 12 - Pedro Costa (SDR)              │
│                                        │
│ ✅ Será enviado: João Silva (10)      │
│                                        │
│ 💡 Se não definir, usa o usuário que  │
│    criou o deal ou padrão da conta    │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (Fixo) -   │
│    lista mostrada diretamente         │
└────────────────────────────────────────┘
```

---

#### **#value**

```json
{
  "name": "value",
  "display_name": "Valor",
  "type": "number",
  "required": false,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Como a LLM deve extrair/calcular o valor?",
    "placeholder": "Ex: Extrair valor mencionado na conversa"
  },
  
  "validation": {
    "min": 0,
    "error_message": "Valor deve ser maior ou igual a zero"
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false,
    "zero_is_valid": true
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ #value                                 │
│ ⚪ Opcional                            │
│                                        │
│ Instrução para LLM:                    │
│ ┌────────────────────────────────────┐ │
│ │ Extrair valor mencionado pelo      │ │
│ │ cliente na conversa                │ │
│ └────────────────────────────────────┘ │
│                                        │
│ 💡 Se não definir, deal fica sem      │
│    valor (0 ou null)                  │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (LLM) -    │
│    campo mostrado diretamente         │
└────────────────────────────────────────┘
```

---

#### **#status**

```json
{
  "name": "status",
  "display_name": "Status",
  "type": "enum",
  "required": false,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["fixed"],
  "default_type": "fixed",
  
  "fixed_config": {
    "multi_select": false,
    "enum_values": [
      { "value": "open", "label": "Aberto (open)" },
      { "value": "won", "label": "Ganho (won)" },
      { "value": "lost", "label": "Perdido (lost)" }
    ],
    "hardcoded": true,
    "default_value": "open"
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ #status                                │
│ ⚪ Opcional                            │
│                                        │
│ Valores disponíveis:                   │
│ ☑ open - Aberto                       │
│ ☐ won - Ganho                         │
│ ☐ lost - Perdido                      │
│                                        │
│ ✅ Será enviado: open                 │
│                                        │
│ 💡 Se não definir, usa "open" por     │
│    padrão                             │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (Fixo) -   │
│    enum validado pela API             │
└────────────────────────────────────────┘
```

---

#### **#probability**

```json
{
  "name": "probability",
  "display_name": "Probabilidade",
  "help_text": "Estimativa de fechamento (0-100%)",
  "type": "number",
  "required": false,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Como a LLM deve estimar a probabilidade?",
    "placeholder": "Ex: Baseado no engajamento do cliente"
  },
  
  "validation": {
    "min": 0,
    "max": 100,
    "error_message": "Probabilidade deve estar entre 0 e 100"
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false,
    "zero_is_valid": true
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ #probability                           │
│ ⚪ Opcional                            │
│                                        │
│ Instrução para LLM:                    │
│ ┌────────────────────────────────────┐ │
│ │ Estimar probabilidade baseado no   │ │
│ │ engajamento e interesse do cliente │ │
│ └────────────────────────────────────┘ │
│                                        │
│ ✅ Validação: 0-100%                  │
│                                        │
│ 💡 Se não definir, usa probabilidade  │
│    padrão do estágio                  │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (LLM) -    │
│    campo mostrado diretamente         │
└────────────────────────────────────────┘
```

---

## 2. updateDeal ✏️

### **Categoria e Visibilidade**

```json
{
  "category": "action",
  "visible_in_checkpoint": true,
  "user_configurable": true,
  "icon": "✏️",
  "show_in_list": true
}
```

### **Metadados da Tool**

```json
{
  "name": "updateDeal",
  "display_name": "Atualizar deal existente",
  "description": "Atualiza os dados de um deal existente no Pipedrive, como título, responsável, estágio, valor, e outros campos relevantes.",
  "integration": "pipedrive",
  "dependencies": [
    {
      "tool": "getOrCreatePerson",
      "type": "required",
      "reason": "Necessário para identificar a pessoa"
    },
    {
      "tool": "getAllExistingDealsFromPerson",
      "type": "required",
      "reason": "Necessário para identificar qual deal atualizar"
    }
  ]
}
```

### **Schema de Parâmetros**

#### **#deal_id** 🔴 CRÍTICO

```json
{
  "name": "deal_id",
  "display_name": "Deal",
  "type": "number",
  "required": true,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": true,
  
  "allowed_input_types": ["dependency"],
  "default_type": "dependency",
  
  "dependency_config": {
    "source_tool": "getAllExistingDealsFromPerson",
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
        "values": ["open", "won", "lost", "deleted"],
        "description": "Status atual do deal"
      },
      {
        "name": "value",
        "type": "number",
        "description": "Valor monetário do deal"
      },
      {
        "name": "currency",
        "type": "string",
        "description": "Moeda (BRL, USD, EUR)"
      },
      {
        "name": "stage_id",
        "type": "number",
        "description": "ID do estágio atual"
      },
      {
        "name": "stage_current",
        "type": "string",
        "description": "Nome do estágio atual"
      },
      {
        "name": "pipeline_id",
        "type": "number",
        "description": "ID do pipeline"
      },
      {
        "name": "person_id",
        "type": "number",
        "description": "ID da pessoa associada"
      },
      {
        "name": "created_at",
        "type": "datetime",
        "description": "Data de criação"
      },
      {
        "name": "updated_at",
        "type": "datetime",
        "description": "Última atualização"
      }
    ],
    "selection_strategy": {
      "type": "llm_with_criteria",
      "prompt_required": true,
      "prompt_hint": "Use os campos disponíveis acima para criar sua instrução"
    }
  },
  
  "dependencies": [
    {
      "tool": "getAllExistingDealsFromPerson",
      "type": "required",
      "auto_invoke": true
    }
  ],
  
  "validation": {
    "required": true,
    "error_message": "🔴 Critério de seleção é obrigatório quando existem múltiplos deals"
  },
  
  "ui_indicators": {
    "badge": "🔴 Campo Crítico",
    "help_text": "Campo usado para identificar qual deal será atualizado"
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────────────────┐
│ #deal_id * 🔴 Campo Crítico                        │
│                                                    │
│ 🔗 Depende de: @getAllExistingDealsFromPerson     │
│ ⚠️ Esta tool retorna múltiplos deals               │
│                                                    │
│ ℹ️ Campos disponíveis do Deal:                    │
│ ┌────────────────────────────────────────────────┐ │
│ │ • id (number) - ID único do deal               │ │
│ │ • title (string) - Título do deal              │ │
│ │ • status (enum) - open, won, lost, deleted     │ │
│ │ • value (number) - Valor monetário             │ │
│ │ • currency (string) - BRL, USD, EUR            │ │
│ │ • stage_id (number) - ID do estágio            │ │
│ │ • stage_current (string) - Nome do estágio     │ │
│ │ • pipeline_id (number) - ID do pipeline        │ │
│ │ • created_at (datetime) - Data de criação      │ │
│ │ • updated_at (datetime) - Última atualização   │ │
│ └────────────────────────────────────────────────┘ │
│                                                    │
│ Critério para seleção: *                          │
│ ┌────────────────────────────────────────────────┐ │
│ │ Deal com status "open" e que esteja no         │ │
│ │ estágio "Negociação" (stage_current)           │ │
│ └────────────────────────────────────────────────┘ │
│                                                    │
│ 💡 Use os campos disponíveis acima para criar     │
│    sua instrução precisa                          │
│                                                    │
│ 🔴 Campo usado para identificar qual deal será    │
│    atualizado                                     │
│                                                    │
│ ℹ️ Apenas 1 tipo permitido (Dependência) -        │
│    campos mostrados diretamente                   │
└────────────────────────────────────────────────────┘
```

---

#### **#title**

```json
{
  "name": "title",
  "display_name": "Título",
  "type": "string",
  "required": false,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Como a LLM deve gerar o novo título?",
    "placeholder": "Ex: Atualizar apenas se cliente mudar de empresa",
    "support_tool": null
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

---

#### **#user_id**

```json
{
  "name": "user_id",
  "display_name": "Responsável (owner)",
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
      "url": "/api/integrations/pipedrive/users",
      "trigger": "on_modal_open",
      "frontend_call": true,
      "response_mapping": {
        "value_field": "id",
        "label_field": "name"
      }
    },
    "instruction": {
      "required_when_multi": true,
      "placeholder": "Quando transferir para cada responsável?"
    }
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ #user_id                               │
│ ⚪ Opcional                            │
│                                        │
│ Valores disponíveis:                   │
│ ☑ 10 - João Silva (vendedor)          │
│ ☑ 11 - Maria Santos (gerente)         │
│ ☐ 12 - Pedro Costa (SDR)              │
│                                        │
│ Instrução (obrigatória): *             │
│ ┌────────────────────────────────────┐ │
│ │ Se deal for de valor alto (>5000), │ │
│ │ transferir para Maria Santos (11). │ │
│ │ Caso contrário, João Silva (10).   │ │
│ └────────────────────────────────────┘ │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (Fixo) -   │
│    lista mostrada diretamente         │
└────────────────────────────────────────┘
```

---

#### **#stage_id**

```json
{
  "name": "stage_id",
  "display_name": "Estágio",
  "type": "number",
  "required": false,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["fixed"],
  "default_type": "fixed",
  
  "fixed_config": {
    "multi_select": false,
    "api_endpoint": {
      "method": "GET",
      "url": "/api/integrations/pipedrive/stages",
      "trigger": "on_modal_open",
      "frontend_call": true,
      "response_mapping": {
        "value_field": "id",
        "label_field": "name"
      }
    }
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

---

#### **#value**

```json
{
  "name": "value",
  "display_name": "Valor",
  "type": "number",
  "required": false,
  "visible": true,
  "show_by_default": false,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Como a LLM deve extrair/calcular o valor?",
    "placeholder": "Ex: Extrair valor mencionado na conversa"
  },
  
  "validation": {
    "min": 0,
    "error_message": "Valor deve ser maior ou igual a zero"
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false,
    "zero_is_valid": true
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ #value                                 │
│ ⚪ Opcional                            │
│                                        │
│ Instrução para LLM:                    │
│ ┌────────────────────────────────────┐ │
│ │ Extrair valor mencionado pelo      │ │
│ │ cliente na conversa                │ │
│ └────────────────────────────────────┘ │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (LLM) -    │
│    campo mostrado diretamente         │
└────────────────────────────────────────┘
```

---

#### **#currency**

```json
{
  "name": "currency",
  "display_name": "Moeda",
  "type": "string",
  "required": false,
  "visible": true,
  "show_by_default": false,
  "is_critical_field": false,
  
  "allowed_input_types": ["fixed"],
  "default_type": "fixed",
  
  "fixed_config": {
    "multi_select": false,
    "enum_values": [
      { "value": "BRL", "label": "Real Brasileiro (BRL)" },
      { "value": "USD", "label": "Dólar Americano (USD)" },
      { "value": "EUR", "label": "Euro (EUR)" },
      { "value": "GBP", "label": "Libra Esterlina (GBP)" }
    ],
    "hardcoded": true
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ #currency                              │
│ ⚪ Opcional                            │
│                                        │
│ Valores disponíveis:                   │
│ ☑ BRL - Real Brasileiro               │
│ ☐ USD - Dólar Americano               │
│ ☐ EUR - Euro                          │
│ ☐ GBP - Libra Esterlina               │
│                                        │
│ ✅ Será enviado: BRL                  │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (Fixo) -   │
│    enum validado pela API             │
└────────────────────────────────────────┘
```

---

#### **#status**

```json
{
  "name": "status",
  "display_name": "Status",
  "type": "enum",
  "required": false,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["fixed"],
  "default_type": "fixed",
  
  "fixed_config": {
    "multi_select": true,
    "enum_values": [
      { "value": "open", "label": "Aberto (open)" },
      { "value": "won", "label": "Ganho (won)" },
      { "value": "lost", "label": "Perdido (lost)" },
      { "value": "deleted", "label": "Deletado (deleted)" }
    ],
    "hardcoded": true,
    "instruction": {
      "required_when_multi": true,
      "placeholder": "Quando usar cada status?"
    }
  },
  
  "parameter_relationships": [
    {
      "depends_on_parameter": "lost_reason",
      "type": "conditional",
      "behavior": {
        "show_when": "status === 'lost'",
        "required_when_shown": true
      }
    }
  ],
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

---

#### **#lost_reason**

```json
{
  "name": "lost_reason",
  "display_name": "Motivo da perda",
  "type": "string",
  "required": false,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Como a LLM deve extrair o motivo?",
    "placeholder": "Ex: Extrair motivo mencionado pelo cliente",
    "support_tool": null
  },
  
  "parameter_relationships": [
    {
      "depends_on_parameter": "status",
      "type": "conditional",
      "behavior": {
        "show_only_when": "status === 'lost'",
        "required_when_shown": true
      },
      "ui": {
        "hidden_message": "Aparece apenas quando status = lost",
        "required_indicator": "* Obrigatório quando status é 'lost'"
      }
    }
  ],
  
  "validation": {
    "required_if": "status === 'lost'",
    "error_message": "Motivo da perda é obrigatório quando status é 'lost'"
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

**Exemplo Visual (quando status = lost):**

```
┌────────────────────────────────────────┐
│ #lost_reason *                         │
│ 🟡 Obrigatório (status é 'lost')       │
│                                        │
│ Instrução para LLM:                    │
│ ┌────────────────────────────────────┐ │
│ │ Extrair o motivo da perda          │ │
│ │ mencionado pelo cliente            │ │
│ └────────────────────────────────────┘ │
│                                        │
│ ⚠️ Aparece apenas quando status = lost│
│                                        │
│ ℹ️ Apenas 1 tipo permitido (LLM) -    │
│    campo mostrado diretamente         │
└────────────────────────────────────────┘
```

---

#### **#expected_close_date**

```json
{
  "name": "expected_close_date",
  "display_name": "Data esperada de fechamento",
  "type": "string",
  "format": "date",
  "required": false,
  "visible": true,
  "show_by_default": false,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Como a LLM deve definir a data?",
    "placeholder": "Ex: Usar data mencionada pelo cliente ou calcular +30 dias",
    "format_note": "Sistema normaliza automaticamente para YYYY-MM-DD"
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ #expected_close_date                   │
│ ⚪ Opcional                            │
│                                        │
│ Instrução para LLM:                    │
│ ┌────────────────────────────────────┐ │
│ │ Usar data mencionada pelo cliente  │ │
│ │ ou calcular +30 dias               │ │
│ └────────────────────────────────────┘ │
│                                        │
│ ⚙️ Sistema normaliza para YYYY-MM-DD  │
│    automaticamente                     │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (LLM) -    │
│    campo mostrado diretamente         │
└────────────────────────────────────────┘
```

---

#### **#probability**

```json
{
  "name": "probability",
  "display_name": "Probabilidade",
  "help_text": "Estimativa de fechamento (0-100%)",
  "type": "number",
  "required": false,
  "visible": true,
  "show_by_default": false,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Como a LLM deve estimar a probabilidade?",
    "placeholder": "Ex: Baseado no engajamento do cliente"
  },
  
  "validation": {
    "min": 0,
    "max": 100,
    "error_message": "Probabilidade deve estar entre 0 e 100"
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false,
    "zero_is_valid": true
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ #probability                           │
│ ⚪ Opcional                            │
│                                        │
│ Instrução para LLM:                    │
│ ┌────────────────────────────────────┐ │
│ │ Estimar probabilidade baseado no   │ │
│ │ engajamento e interesse do cliente │ │
│ └────────────────────────────────────┘ │
│                                        │
│ ✅ Validação: 0-100%                  │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (LLM) -    │
│    campo mostrado diretamente         │
└────────────────────────────────────────┘
```

---

## 3. createNote 📝

### **Categoria e Visibilidade**

```json
{
  "category": "action",
  "visible_in_checkpoint": true,
  "user_configurable": true,
  "icon": "📝",
  "show_in_list": true
}
```

### **Metadados da Tool**

```json
{
  "name": "createNote",
  "display_name": "Criar nova nota",
  "description": "Cria uma nova nota para um deal específico no Pipedrive.",
  "integration": "pipedrive",
  "dependencies": [
    {
      "tool": "getOrCreatePerson",
      "type": "required"
    },
    {
      "tool": "getAllExistingDealsFromPerson",
      "type": "required"
    }
  ]
}
```

### **Schema de Parâmetros**

#### **#deal_id**

```json
{
  "name": "deal_id",
  "display_name": "Deal",
  "type": "number",
  "required": true,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": true,
  
  "allowed_input_types": ["dependency"],
  "default_type": "dependency",
  
  "dependency_config": {
    "source_tool": "getAllExistingDealsFromPerson",
    "source_field": "id",
    "output_type": "array",
    "selection_required": true,
    "show_fields_preview": true,
    "available_fields": [
      { "name": "id", "type": "number" },
      { "name": "title", "type": "string" },
      { "name": "status", "type": "enum", "values": ["open", "won", "lost"] },
      { "name": "stage_current", "type": "string" }
    ],
    "selection_strategy": {
      "type": "llm_with_criteria",
      "prompt_required": true,
      "prompt_hint": "Qual deal receberá a nota?"
    }
  }
}
```

---

#### **#content**

```json
{
  "name": "content",
  "display_name": "Conteúdo da nota",
  "type": "string",
  "required": true,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "O que a nota deve conter?",
    "placeholder": "Ex: Resumir pontos principais da conversa",
    "support_tool": null
  },
  
  "validation": {
    "min_length": 1,
    "error_message": "Conteúdo é obrigatório"
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ #content *                             │
│ 🟡 Obrigatório                         │
│                                        │
│ Instrução para LLM:                    │
│ ┌────────────────────────────────────┐ │
│ │ Criar um resumo executivo da       │ │
│ │ conversa incluindo:                │ │
│ │ - Necessidades identificadas       │ │
│ │ - Objeções mencionadas             │ │
│ │ - Próximos passos acordados        │ │
│ └────────────────────────────────────┘ │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (LLM) -    │
│    campo mostrado diretamente         │
└────────────────────────────────────────┘
```

---

## 4. updateNote ✏️📝

### **Categoria e Visibilidade**

```json
{
  "category": "action",
  "visible_in_checkpoint": true,
  "user_configurable": true,
  "icon": "✏️",
  "show_in_list": true
}
```

### **Metadados da Tool**

```json
{
  "name": "updateNote",
  "display_name": "Atualizar nota existente",
  "description": "Atualiza o conteúdo de uma nota existente no Pipedrive.",
  "integration": "pipedrive",
  "dependencies": [
    {
      "tool": "getOrCreatePerson",
      "type": "required"
    },
    {
      "tool": "getAllExistingDealsFromPerson",
      "type": "required"
    },
    {
      "tool": "getDealWithCompleteInfo",
      "type": "required",
      "reason": "Necessário para obter as notas existentes do deal"
    }
  ]
}
```

### **Schema de Parâmetros**

#### **#note_id**

```json
{
  "name": "note_id",
  "display_name": "Nota",
  "type": "number",
  "required": true,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": true,
  
  "allowed_input_types": ["dependency"],
  "default_type": "dependency",
  
  "dependency_config": {
    "source_tool": "getDealWithCompleteInfo",
    "source_field": "notes[].id",
    "output_type": "array",
    "selection_required": true,
    "show_fields_preview": true,
    "available_fields": [
      {
        "name": "id",
        "type": "number",
        "description": "ID da nota"
      },
      {
        "name": "content",
        "type": "string",
        "description": "Conteúdo atual da nota"
      }
    ],
    "selection_strategy": {
      "type": "llm_with_criteria",
      "prompt_required": true,
      "prompt_hint": "Qual nota deve ser atualizada?"
    }
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────────────────┐
│ #note_id * 🔴 Campo Crítico                        │
│                                                    │
│ 🔗 Depende de: @getDealWithCompleteInfo           │
│                                                    │
│ ℹ️ Campos disponíveis da Nota:                    │
│ ┌────────────────────────────────────────────────┐ │
│ │ • id (number) - ID da nota                     │ │
│ │ • content (string) - Conteúdo atual            │ │
│ └────────────────────────────────────────────────┘ │
│                                                    │
│ Critério para seleção: *                          │
│ ┌────────────────────────────────────────────────┐ │
│ │ A nota mais recente (última criada)            │ │
│ └────────────────────────────────────────────────┘ │
│                                                    │
│ ℹ️ Apenas 1 tipo permitido (Dependência) -        │
│    campos mostrados diretamente                   │
└────────────────────────────────────────────────────┘
```

---

#### **#content**

```json
{
  "name": "content",
  "display_name": "Novo conteúdo",
  "type": "string",
  "required": true,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Como atualizar o conteúdo?",
    "placeholder": "Ex: Adicionar informações da nova conversa ao conteúdo existente"
  }
}
```

---

## 5. createDealActivity 📅

### **Categoria e Visibilidade**

```json
{
  "category": "action",
  "visible_in_checkpoint": true,
  "user_configurable": true,
  "icon": "📅",
  "show_in_list": true
}
```

### **Metadados da Tool**

```json
{
  "name": "createDealActivity",
  "display_name": "Criar atividade para deal",
  "description": "Cria uma nova atividade (reunião) para um deal específico no Pipedrive.",
  "integration": "pipedrive",
  "dependencies": [
    {
      "tool": "getOrCreatePerson",
      "type": "required"
    },
    {
      "tool": "getAllExistingDealsFromPerson",
      "type": "required"
    }
  ]
}
```

### **Schema de Parâmetros**

#### **#subject**

```json
{
  "name": "subject",
  "display_name": "Assunto",
  "type": "string",
  "required": true,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Como gerar o assunto da atividade?",
    "placeholder": "Ex: Reunião de alinhamento com [nome da pessoa]"
  }
}
```

---

#### **#deal_id**

```json
{
  "name": "deal_id",
  "display_name": "Deal",
  "type": "number",
  "required": true,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": true,
  
  "allowed_input_types": ["dependency"],
  "default_type": "dependency",
  
  "dependency_config": {
    "source_tool": "getAllExistingDealsFromPerson",
    "source_field": "id",
    "output_type": "array",
    "selection_required": true,
    "show_fields_preview": true,
    "available_fields": [
      { "name": "id", "type": "number" },
      { "name": "title", "type": "string" },
      { "name": "status", "type": "enum", "values": ["open", "won", "lost"] }
    ]
  }
}
```

---

#### **#due_date**

```json
{
  "name": "due_date",
  "display_name": "Data",
  "type": "string",
  "format": "date",
  "required": true,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Como definir a data?",
    "placeholder": "Ex: Usar data mencionada pelo cliente",
    "format_note": "Sistema normaliza para YYYY-MM-DD automaticamente"
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ #due_date *                            │
│ 🟡 Obrigatório                         │
│                                        │
│ Instrução para LLM:                    │
│ ┌────────────────────────────────────┐ │
│ │ Usar data mencionada pelo cliente  │ │
│ └────────────────────────────────────┘ │
│                                        │
│ ⚙️ Sistema normaliza para YYYY-MM-DD  │
│    automaticamente                     │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (LLM) -    │
│    campo mostrado diretamente         │
└────────────────────────────────────────┘
```

---

#### **#due_time**

```json
{
  "name": "due_time",
  "display_name": "Horário",
  "type": "string",
  "format": "time",
  "required": true,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Como definir o horário?",
    "placeholder": "Ex: Usar horário mencionado ou sugerir 14:00",
    "format_note": "Sistema normaliza para HH:mm automaticamente"
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ #due_time *                            │
│ 🟡 Obrigatório                         │
│                                        │
│ Instrução para LLM:                    │
│ ┌────────────────────────────────────┐ │
│ │ Usar horário mencionado pelo       │ │
│ │ cliente. Se não mencionar,         │ │
│ │ sugerir 14:00                      │ │
│ └────────────────────────────────────┘ │
│                                        │
│ ⚙️ Sistema normaliza para HH:mm       │
│    automaticamente                     │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (LLM) -    │
│    campo mostrado diretamente         │
└────────────────────────────────────────┘
```

---

#### **#duration**

```json
{
  "name": "duration",
  "display_name": "Duração",
  "type": "string",
  "format": "time",
  "required": false,
  "visible": true,
  "show_by_default": false,
  "is_critical_field": false,
  
  "allowed_input_types": ["fixed"],
  "default_type": "fixed",
  
  "fixed_config": {
    "input_type": "time",
    "format": "HH:mm",
    "default_value": "01:00",
    "presets": [
      { "value": "00:30", "label": "30 minutos" },
      { "value": "01:00", "label": "1 hora" },
      { "value": "01:30", "label": "1h30" },
      { "value": "02:00", "label": "2 horas" }
    ]
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ #duration                              │
│ ⚪ Opcional                            │
│                                        │
│ Duração padrão:                        │
│ ☐ 00:30 - 30 minutos                  │
│ ☑ 01:00 - 1 hora                      │
│ ☐ 01:30 - 1h30                        │
│ ☐ 02:00 - 2 horas                     │
│                                        │
│ ✅ Será enviado: 01:00                │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (Fixo) -   │
│    lista mostrada diretamente         │
└────────────────────────────────────────┘
```

---

#### **#note**

```json
{
  "name": "note",
  "display_name": "Nota interna",
  "type": "string",
  "required": false,
  "visible": true,
  "show_by_default": false,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "O que incluir na nota interna?",
    "placeholder": "Ex: Contexto da conversa para o vendedor",
    "help_text": "Nota visível apenas internamente (não enviada ao lead)"
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

---

#### **#public_description**

```json
{
  "name": "public_description",
  "display_name": "Descrição pública",
  "type": "string",
  "required": false,
  "visible": true,
  "show_by_default": false,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "O que incluir na descrição pública?",
    "placeholder": "Ex: Agenda e pauta da reunião",
    "help_text": "Descrição visível para o lead"
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

---

#### **#attendees**

```json
{
  "name": "attendees",
  "display_name": "Participantes",
  "type": "array",
  "item_type": "object",
  "required": true,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Quem deve ser incluído como participante?",
    "placeholder": "Ex: Adicionar a pessoa do deal + vendedor responsável",
    "default_instruction": "Incluir a pessoa associada ao deal e outros participantes mencionados"
  },
  
  "validation": {
    "min_items": 1,
    "error_message": "Pelo menos um participante é obrigatório"
  }
}
```

**Exemplo Visual:**

```
┌────────────────────────────────────────┐
│ #attendees *                           │
│ 🟡 Obrigatório                         │
│                                        │
│ Instrução para LLM:                    │
│ ┌────────────────────────────────────┐ │
│ │ Adicionar sempre:                  │ │
│ │ 1. A pessoa associada ao deal      │ │
│ │ 2. O vendedor responsável          │ │
│ │ 3. Outros participantes mencionados│ │
│ │    na conversa                     │ │
│ └────────────────────────────────────┘ │
│                                        │
│ ✅ Validação: Mínimo 1 participante   │
│                                        │
│ ℹ️ Apenas 1 tipo permitido (LLM) -    │
│    campo mostrado diretamente         │
└────────────────────────────────────────┘
```

---

## 6. getOrCreatePerson 👤

### **Categoria e Visibilidade**

```json
{
  "category": "hybrid",
  "visible_in_checkpoint": true,
  "user_configurable": true,
  "icon": "👤",
  "show_in_list": true,
  "reason": "Tool híbrida - 'get' mas com side effect (create)"
}
```

### **Metadados da Tool**

```json
{
  "name": "getOrCreatePerson",
  "display_name": "Buscar ou criar pessoa",
  "description": "Busca uma pessoa no Pipedrive pelo telefone. Se não encontrar, cria uma nova com as informações do cliente.",
  "integration": "pipedrive",
  "dependencies": []
}
```

### **Schema de Parâmetros**

#### **#fullname**

```json
{
  "name": "fullname",
  "display_name": "Nome completo",
  "type": "string",
  "required": true,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Como extrair o nome?",
    "placeholder": "Ex: Extrair nome completo da conversa",
    "default_instruction": "Extrair nome completo do cliente da conversa"
  }
}
```

---

#### **#email**

```json
{
  "name": "email",
  "display_name": "Email",
  "type": "string",
  "format": "email",
  "required": false,
  "visible": true,
  "show_by_default": false,
  "is_critical_field": false,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Como extrair o email?",
    "placeholder": "Ex: Extrair email da conversa",
    "null_handling": {
      "instruction": "Se não encontrar email, retorne explicitamente NULL",
      "parse_strategy": "convert_string_null_to_real_null",
      "fallback_value": null
    }
  },
  
  "validation": {
    "format": "email",
    "error_message": "Formato de email inválido"
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

---

#### **#phone** 🔴 CRÍTICO

```json
{
  "name": "phone",
  "display_name": "Telefone",
  "help_text": "Número do cliente",
  "type": "string",
  "format": "phone",
  
  "required": true,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": true,
  
  "allowed_input_types": ["llm"],
  "default_type": "llm",
  
  "llm_config": {
    "instruction_hint": "Como extrair o telefone?",
    "placeholder": "Ex: Extrair telefone da conversa",
    "default_instruction": "Extrair telefone do cliente. Use lead_phone das variáveis se disponível."
  },
  
  "validation": {
    "format": "e164_international",
    "error_message": "Telefone é obrigatório e será normalizado automaticamente"
  },
  
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
    "warning": "Campo usado para buscar pessoas existentes. Sistema normaliza formato automaticamente para padrão internacional (+5511987654321)."
  }
}
```

**Exemplo Visual:**

```
┌─────────────────────────────────────────────────┐
│ #phone * 🔴 Campo Crítico                       │
│                                                 │
│ Instrução para LLM:                             │
│ ┌─────────────────────────────────────────────┐ │
│ │ Extrair telefone da conversa.               │ │
│ │ Se disponível, usar lead_phone das          │ │
│ │ variáveis da campanha.                      │ │
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
│                                                 │
│ ℹ️ Apenas 1 tipo permitido (LLM) -             │
│    campo mostrado diretamente                  │
└─────────────────────────────────────────────────┘
```

---

## 7. getDealWithCompleteInfo 🔍

### **Categoria e Visibilidade**

```json
{
  "category": "query_visible",
  "visible_in_checkpoint": true,
  "user_configurable": true,
  "icon": "🔍",
  "show_in_list": true,
  "read_only": true,
  "reason": "Consulta que usuário pode querer executar explicitamente"
}
```

### **Metadados da Tool**

```json
{
  "name": "getDealWithCompleteInfo",
  "display_name": "Buscar deal com informações completas",
  "description": "Busca um deal específico no Pipedrive com informações completas incluindo notas, atividades, histórico de mudanças e dados do responsável.",
  "integration": "pipedrive",
  "dependencies": [
    {
      "tool": "getOrCreatePerson",
      "type": "required"
    },
    {
      "tool": "getAllExistingDealsFromPerson",
      "type": "required"
    }
  ]
}
```

### **Schema de Parâmetros**

#### **#deal_id**

```json
{
  "name": "deal_id",
  "display_name": "Deal",
  "type": "number",
  "required": true,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": true,
  
  "allowed_input_types": ["dependency"],
  "default_type": "dependency",
  
  "dependency_config": {
    "source_tool": "getAllExistingDealsFromPerson",
    "source_field": "id",
    "output_type": "array",
    "selection_required": true,
    "show_fields_preview": true,
    "available_fields": [
      { "name": "id", "type": "number" },
      { "name": "title", "type": "string" },
      { "name": "status", "type": "enum", "values": ["open", "won", "lost"] },
      { "name": "value", "type": "number" }
    ]
  }
}
```

---

## 8. getActivitiesFromDeal 🔍

### **Categoria e Visibilidade**

```json
{
  "category": "query_visible",
  "visible_in_checkpoint": true,
  "user_configurable": true,
  "icon": "🔍",
  "show_in_list": true,
  "read_only": true
}
```

### **Metadados da Tool**

```json
{
  "name": "getActivitiesFromDeal",
  "display_name": "Buscar atividades de um deal",
  "description": "Busca as atividades associadas a um deal específico no Pipedrive.",
  "integration": "pipedrive",
  "dependencies": [
    {
      "tool": "getOrCreatePerson",
      "type": "required"
    },
    {
      "tool": "getAllExistingDealsFromPerson",
      "type": "required"
    }
  ]
}
```

### **Schema de Parâmetros**

#### **#deal_id**

```json
{
  "name": "deal_id",
  "display_name": "Deal",
  "type": "number",
  "required": true,
  "visible": true,
  "show_by_default": true,
  "is_critical_field": true,
  
  "allowed_input_types": ["dependency"],
  "default_type": "dependency",
  
  "dependency_config": {
    "source_tool": "getAllExistingDealsFromPerson",
    "source_field": "id",
    "output_type": "array",
    "selection_required": true,
    "show_fields_preview": true,
    "available_fields": [
      { "name": "id", "type": "number" },
      { "name": "title", "type": "string" },
      { "name": "status", "type": "enum", "values": ["open", "won", "lost"] }
    ]
  }
}
```

---

## 9. getAllExistingPipelines ⚙️

### **Categoria e Visibilidade**

```json
{
  "category": "support_llm",
  "visible_in_checkpoint": false,
  "user_configurable": false,
  "show_in_list": false,
  "trigger": "runtime_for_llm_context",
  "reason": "Tool de suporte - fornece contexto para LLM automaticamente"
}
```

### **Metadados da Tool**

```json
{
  "name": "getAllExistingPipelines",
  "display_name": "Buscar todos os pipelines existentes",
  "description": "Busca os pipelines existentes no Pipedrive com seus respectivos ids e stages. Chamada automaticamente quando parâmetro tipo LLM precisa de contexto de pipelines.",
  "integration": "pipedrive",
  "dependencies": [],
  "provides_context_for": [
    {
      "tool": "createDeal",
      "parameter": "pipeline_id",
      "when": "type=llm"
    },
    {
      "tool": "updateDeal",
      "parameter": "pipeline_id",
      "when": "type=llm"
    }
  ]
}
```

### **Schema de Parâmetros**

#### **#get**

```json
{
  "name": "get",
  "display_name": "Buscar",
  "type": "boolean",
  "required": true,
  "visible": false,
  "show_by_default": false,
  "is_critical_field": false,
  
  "allowed_input_types": ["fixed"],
  "default_type": "fixed",
  
  "fixed_config": {
    "hardcoded_value": true,
    "reason": "Parâmetro técnico sem valor de configuração"
  },
  
  "ui_indicators": {
    "hidden": true,
    "reason": "Regra 2B - Parâmetro técnico/sistema"
  }
}
```

**Como usuário vê:**

```
❌ NÃO APARECE NA UI DO CHECKPOINT

Esta tool é invocada automaticamente em runtime.

Exemplo: Quando pipeline_id tipo=LLM é configurado,
o sistema chama getAllExistingPipelines para fornecer
contexto à LLM.

Usuário vê apenas o indicador:

┌────────────────────────────────────────┐
│ #pipeline_id                           │
│                                        │
│ ℹ️ LLM terá acesso automático aos     │
│    pipelines via                       │
│    @getAllExistingPipelines            │
│                                        │
│ Instrução para LLM:                    │
│ ┌────────────────────────────────────┐ │
│ │ [instrução configurada pelo        │ │
│ │  usuário]                          │ │
│ └────────────────────────────────────┘ │
└────────────────────────────────────────┘
```

---

## 10. getAllExistingDealsFromPerson ⚙️

### **Categoria e Visibilidade**

```json
{
  "category": "query_dependency",
  "visible_in_checkpoint": false,
  "user_configurable": false,
  "show_in_list": false,
  "trigger": "dependency_for_action_tool",
  "reason": "Tool de dependência - invocada automaticamente"
}
```

### **Metadados da Tool**

```json
{
  "name": "getAllExistingDealsFromPerson",
  "display_name": "Buscar todos os deals existentes de uma pessoa",
  "description": "Busca os deals existentes no Pipedrive para uma pessoa específica. Invocada automaticamente quando tool de ação precisa de deal_id.",
  "integration": "pipedrive",
  "dependencies": [
    {
      "tool": "getOrCreatePerson",
      "type": "required"
    }
  ],
  "invoked_by": [
    {
      "tool": "updateDeal",
      "parameter": "deal_id"
    },
    {
      "tool": "createNote",
      "parameter": "deal_id"
    },
    {
      "tool": "updateNote",
      "parameter": "deal_id"
    },
    {
      "tool": "createDealActivity",
      "parameter": "deal_id"
    },
    {
      "tool": "getDealWithCompleteInfo",
      "parameter": "deal_id"
    },
    {
      "tool": "getActivitiesFromDeal",
      "parameter": "deal_id"
    }
  ]
}
```

### **Schema de Parâmetros**

#### **#person_id**

```json
{
  "name": "person_id",
  "display_name": "Pessoa",
  "type": "number",
  "required": true,
  "visible": false,
  "show_by_default": true,
  "is_critical_field": true,
  
  "allowed_input_types": ["dependency"],
  "default_type": "dependency",
  
  "dependencies": [
    {
      "tool": "getOrCreatePerson",
      "field": "id",
      "type": "required",
      "auto_resolve": true
    }
  ],
  
  "ui_indicators": {
    "hidden": true
  }
}
```

---

#### **#pipeline_id**

```json
{
  "name": "pipeline_id",
  "display_name": "Filtrar por pipeline",
  "type": "number",
  "required": false,
  "visible": false,
  "show_by_default": false,
  "is_critical_field": false,
  
  "allowed_input_types": ["fixed"],
  "default_type": "fixed",
  
  "ui_indicators": {
    "hidden": true,
    "reason": "Filtro opcional - não exposto ao usuário"
  },
  
  "nullable_behavior": {
    "empty_converts_to": null,
    "send_when_null": false
  }
}
```

**Como usuário vê:**

```
❌ NÃO APARECE NA UI DO CHECKPOINT

Esta tool é invocada automaticamente quando:
- Usuário configura @updateDeal, @createNote, etc.
- Parâmetro deal_id precisa ser resolvido

Exemplo: Usuário configura apenas @updateDeal

┌────────────────────────────────────────────────────┐
│ @updateDeal                                        │
│                                                    │
│ #deal_id *                                         │
│                                                    │
│ 🔗 Fonte: Consultar deals da pessoa               │
│                                                    │
│ Critério para seleção:                            │
│ ┌────────────────────────────────────────────────┐ │
│ │ Deal com status open e valor > 1000            │ │
│ └────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────┘

Em runtime:
1. Sistema invoca @getAllExistingDealsFromPerson
2. Retorna: [deal1, deal2, deal3]
3. LLM aplica critério: "status open e valor > 1000"
4. Seleciona: deal2
5. @updateDeal usa deal_id=2

✅ Usuário não viu @getAllExistingDealsFromPerson
✅ Invocação foi automática
```

---

## 📊 **Resumo de Categorias**

### **Tools Visíveis (7 tools)**

| Tool | Categoria | Ícone | Dependências |
|------|-----------|-------|--------------|
| **createDeal** | Ação | 💼 | getOrCreatePerson |
| **updateDeal** | Ação | ✏️ | getOrCreatePerson, getAllExistingDealsFromPerson |
| **createNote** | Ação | 📝 | getOrCreatePerson, getAllExistingDealsFromPerson |
| **updateNote** | Ação | ✏️📝 | getOrCreatePerson, getAllExistingDealsFromPerson, getDealWithCompleteInfo |
| **createDealActivity** | Ação | 📅 | getOrCreatePerson, getAllExistingDealsFromPerson |
| **getOrCreatePerson** | Híbrida | 👤 | (nenhuma) |
| **getDealWithCompleteInfo** | Consulta | 🔍 | getOrCreatePerson, getAllExistingDealsFromPerson |
| **getActivitiesFromDeal** | Consulta | 🔍 | getOrCreatePerson, getAllExistingDealsFromPerson |

### **Tools Automáticas (2 tools)**

| Tool | Categoria | Invocação |
|------|-----------|-----------|
| **getAllExistingPipelines** | Suporte LLM | Quando pipeline_id tipo=LLM |
| **getAllExistingDealsFromPerson** | Dependência | Quando deal_id necessário |

---

## 🎯 **Padrões Identificados**

### **Parâmetros Críticos 🔴**

- **phone** (getOrCreatePerson) - chave de busca, normalização automática
- **deal_id** (updateDeal, createNote, etc.) - identificação de recurso
- **note_id** (updateNote) - identificação de recurso

### **Dependências Comuns**

```
Fluxo típico:

1. @getOrCreatePerson (sempre primeiro)
   └─ Retorna: person_id (valor único)

2. @getAllExistingDealsFromPerson (auto-invocada)
   └─ Usa: person_id
   └─ Retorna: [deals] (array)

3. @updateDeal / @createNote / etc. (configurada pelo usuário)
   └─ Usa: person_id (oculto, resolvido automaticamente)
   └─ Usa: deal_id (visível, requer critério de seleção)
```

### **Parâmetros Relacionados**

```
pipeline_id → stage_id (contextual)
  → stage_id desabilitado até pipeline_id ser selecionado
  → stage_id carrega opções baseado no pipeline selecionado

status = 'lost' → lost_reason (condicional)
  → lost_reason aparece apenas quando status é 'lost'
  → lost_reason torna-se obrigatório quando visível
```

---

## 🔗 **Mapa de Dependências**

```
┌─────────────────────────────────────────────────────┐
│ LAYER 1: Base                                       │
├─────────────────────────────────────────────────────┤
│ 👤 getOrCreatePerson (híbrida - visível)            │
│    └─ Retorna: person_id (valor único)              │
└─────────────────────────────────────────────────────┘
                          │
            ┌─────────────┴─────────────┐
            │                           │
            v                           v
┌─────────────────────────┐  ┌─────────────────────────┐
│ LAYER 2: Consultas      │  │ LAYER 2: Suporte        │
├─────────────────────────┤  ├─────────────────────────┤
│ ⚙️ getAllExistingDeals  │  │ ⚙️ getAllExistingPipe   │
│    FromPerson           │  │    lines (suporte LLM)  │
│    (dependência)        │  │                         │
│    └─ array de deals    │  │    └─ contexto LLM     │
└─────────────────────────┘  └─────────────────────────┘
            │
            ├────────────┬────────────┬────────────┐
            │            │            │            │
            v            v            v            v
┌───────────────┐ ┌───────────┐ ┌───────────┐ ┌──────────┐
│ LAYER 3: Ações│ │           │ │           │ │          │
├───────────────┤ ├───────────┤ ├───────────┤ ├──────────┤
│ 💼 createDeal │ │ ✏️ update │ │ 📝 create │ │ 📅 create│
│               │ │    Deal   │ │    Note   │ │ Activity │
└───────────────┘ └─────┬─────┘ └───────────┘ └──────────┘
                        │
                        v
                  ┌──────────────┐
                  │ 🔍 getDeal   │
                  │ WithComplete │
                  │ Info         │
                  └──────┬───────┘
                         │
                         v
                  ┌──────────────┐
                  │ ✏️📝 update  │
                  │    Note      │
                  └──────────────┘
```

---

**Versão:** 2.3  
**Data:** 2025-11-10  
**Framework:** v3.7  
**Integração:** Pipedrive  
**Status:** Completo e pronto para implementação

---

## 📝 Changelog

### **v2.3** - 2025-11-10 - Reformulação de Criticidade → Visibilidade Padrão + Padronização

**Mudanças Principais:**
- ❌ Removido campo `criticality` (critical/important/complementary) de todos os parâmetros
- ✅ Adicionado campo `show_by_default` (boolean) - controla se aparece de cara ou no botão "Adicionar"
- ✅ Mantido campo `visible` (boolean) - controla se PODE aparecer
- ✅ Novo campo `is_critical_field` (boolean) - apenas para normalização automática
- ✅ Adicionado campo `help_text` nos parâmetros principais

**Padronização de Schemas:**
- ✅ **Ordem padronizada** de todos os campos seguindo o glossário
- ✅ **Ordem correta:** `visible` → `show_by_default` → `is_critical_field` → `allowed_input_types` → `default_type`
- ✅ **Correção:** `help_text` movido de `ui_indicators` para nível raiz
- ✅ **Correção:** `ui_indicators.help_text` renomeado para `ui_indicators.warning`
- ✅ **Aplicado em:** Todos os 10 tools e 50+ parâmetros

**Parâmetros Atualizados:**

**createDeal:**
- `title`: + `help_text: "Crie um título descritivo"`, `show_by_default: true`, `is_critical_field: false`
- `pipeline_id`: + `help_text: "Selecione o(s) pipeline(s)"`, `show_by_default: true`, `is_critical_field: false`
- `person_id`: `show_by_default: false` (não importa, visible=false), `is_critical_field: true`
- `probability`: + `help_text: "Estimativa de fechamento (0-100%)"`, `show_by_default: false`, `is_critical_field: false`

**Atualização em massa:**
- critical → `show_by_default: true`, `is_critical_field: true`
- important → `show_by_default: true`, `is_critical_field: false`
- complementary → `show_by_default: false`, `is_critical_field: false`

**Exemplos Visuais:** Atualizados para mostrar `help_text` ao lado do `display_name`

**Referência:** Framework v3.7 - Regras 10 e 11

---

### **v2.2** - 2025-11-10 - Remoção de Enum Values para Tipo LLM

**Mudança:**
- ❌ Removido campo `enum_values` de `llm_config`
- ✅ Parâmetros do tipo LLM agora têm liberdade total para gerar valores baseado em contexto

**Parâmetro Atualizado:**
- **`title`** em `createDeal`: Removido `enum_values: []` do `llm_config`

**Impacto:**
- ✅ LLM tem mais flexibilidade para criar títulos contextuais
- ✅ Redução de complexidade no schema
- ✅ Alinhado com Framework v3.6

**Justificativa:**  
Se há necessidade de restringir valores, usar tipo `["fixed"]` com `enum_values` ao invés de tentar limitar a LLM.

**Referência:** Framework v3.6 - "Remoção de Enum Values para Tipo LLM"

---

### **v2.1** - 2025-11-10 - Simplificação de Tipos + Parâmetros Adicionais

**Mudanças Críticas:**
- ❌ Removidos múltiplos tipos `["llm", "fixed"]` de 7 parâmetros
- ✅ Cada parâmetro agora tem apenas UM tipo baseado no caso de uso real (90%+)
- ✅ **Adicionados 4 parâmetros opcionais importantes ao createDeal**

**Parâmetros Adicionados ao createDeal:**

1. **`user_id`** - Responsável/Owner (opcional)
   - Tipo: Fixo (lista de usuários)
   - Se não definir, usa usuário que criou ou padrão da conta
   - Permite criar deal já atribuído ao responsável correto
   
2. **`value`** - Valor monetário (opcional)
   - Tipo: LLM
   - Se não definir, deal fica sem valor
   - Evita precisar fazer update depois para adicionar valor
   
3. **`status`** - Status do deal (opcional)
   - Tipo: Fixo (enum: open, won, lost)
   - Se não definir, usa "open" por padrão
   - Permite criar deal já em status específico
   
4. **`probability`** - Probabilidade de fechamento (opcional)
   - Tipo: LLM (0-100%)
   - Se não definir, usa probabilidade padrão do estágio
   - Facilita estimativa desde o início

**Benefício:** Permite criar deals mais completos em 1 chamada de API ao invés de precisar fazer update depois.

---

**Parâmetros Atualizados (Simplificação de Tipos):**

**updateDeal:**
1. `value` - `["llm", "fixed"]` → `["llm"]`
   - Raramente fixo, varia por produto mencionado na conversa
2. `expected_close_date` - `["llm", "fixed"]` → `["llm"]`
   - Raramente fixo, varia por disponibilidade do cliente
3. `probability` - `["llm", "fixed"]` → `["llm"]`
   - Raramente fixo, varia por engajamento do cliente

**createDealActivity:**
4. `due_date` - `["llm", "fixed"]` → `["llm"]`
   - Raramente fixo, varia por disponibilidade do cliente
5. `due_time` - `["llm", "fixed"]` → `["llm"]`
   - Raramente fixo, varia por disponibilidade do cliente
6. `duration` - `["llm", "fixed"]` → `["fixed"]` ⚠️
   - Empresa tem presets padrão (30min, 1h, 1h30, 2h)
   - LLM não agrega valor (sem contexto suficiente)
7. `attendees` - `["llm", "fixed"]` → `["llm"]`
   - Raramente fixo, varia por quem foi mencionado na conversa

**Impacto UX:**
- ✅ Seletor de tipo removido (interface mais limpa)
- ✅ Campos mostrados diretamente
- ✅ Menos decisões para o usuário
- ✅ Alinhado com Framework v3.5

**Referência:** Regra 3E do Framework - "Quando NÃO Permitir Múltiplos Tipos"

---

### **v2.0** - 2025-11-07 - Versão Inicial
- Schema completo de 10 tools do Pipedrive
- Baseado no Framework v3.4

