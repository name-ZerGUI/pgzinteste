# 🔧 Pipedrive Tools - Schema de Configuração

Documentação completa de todas as tools visíveis do Pipedrive com schemas e visualizações de UI.

**Versão:** 1.0  
**Data:** 2025-11-06  
**Integração:** Pipedrive CRM

---

## 📋 **Índice de Tools**

### **Tools Visíveis (aparecem na lista @):**

1. [@getOrCreatePerson](#1-getorcreateperson) - Buscar ou criar pessoa
2. [@createDeal](#2-createdeal) - Criar novo deal
3. [@updateDeal](#3-updatedeal) - Atualizar deal existente
4. [@createNote](#4-createnote) - Adicionar nota ao deal
5. [@updateNote](#5-updatenote) - Atualizar nota existente
6. [@createDealActivity](#6-createdealactivity) - Criar atividade no deal

### **Tools Ocultas (invocadas automaticamente):**
- `@getAllExistingPipelines` - Chamada em runtime quando tipo=LLM em pipeline_id
- `@getAllExistingDealsFromPerson` - Chamada quando deal_id tipo=Dependência
- `@getDealWithCompleteInfo` - Chamada quando necessário enriquecer deal
- `@getActivitiesFromDeal` - Chamada quando necessário listar atividades

---

## 1️⃣ **@getOrCreatePerson**

### **Descrição:**
Busca uma pessoa no Pipedrive pelo telefone. Se não encontrar, cria uma nova com as informações fornecidas.

### **Categoria:** Híbrida (GET com efeito colateral)
### **Visível na lista @:** ✅ Sim (pode criar pessoa)
### **Dependências:** ❌ Nenhuma

---

### **Schema Completo:**

```json
{
  "tool": "@getOrCreatePerson",
  "integration": "pipedrive",
  "category": "hybrid",
  "visible_in_checkpoint": true,
  "display_config": {
    "title": "Buscar ou Criar Pessoa",
    "description": "Busca pessoa no Pipedrive pelo telefone. Se não existir, cria nova.",
    "icon": "👤",
    "color": "#4CAF50"
  },
  
  "parameters": [
    {
      "name": "fullname",
      "display_name": "Nome Completo",
      "type": "string",
      "required": true,
      "visible": true,
      "criticality": "important",
      
      "allowed_input_types": ["llm", "fixed"],
      "default_type": "llm",
      
      "llm_config": {
        "default_prompt": "Extrair o nome completo da pessoa (nome + sobrenome). Se não houver sobrenome, usar apenas o nome disponível.",
        "examples": ["João Silva", "Maria Santos"]
      },
      
      "fixed_config": {
        "placeholder": "Ex: {{lead_name}} ou {{contact_fullname}}",
        "allow_variables": true
      },
      
      "validation": {
        "min_length": 2,
        "recommended_min_words": 2,
        "warning_message": "Nome completo recomendado para melhor identificação"
      }
    },
    
    {
      "name": "email",
      "display_name": "Email",
      "type": "string",
      "required": false,
      "visible": true,
      "criticality": "complementary",
      
      "allowed_input_types": ["llm", "fixed"],
      "default_type": "llm",
      
      "llm_config": {
        "default_prompt": "Extrair email da conversa. Se não houver email disponível, retornar null.",
        "examples": ["joao@empresa.com", "maria.silva@gmail.com"],
        "null_handling": {
          "instruction": "Se não encontrar email, retorne NULL",
          "parse_strategy": "convert_string_null_to_real_null"
        }
      },
      
      "fixed_config": {
        "placeholder": "Ex: {{lead_email}}",
        "allow_variables": true
      },
      
      "validation": {
        "format": "email",
        "allow_null": true,
        "allow_empty": true,
        "error_message": "Email inválido. Use formato: usuario@dominio.com"
      },
      
      "nullable_behavior": {
        "empty_converts_to": null,
        "send_when_null": false
      }
    },
    
    {
      "name": "phone",
      "display_name": "Telefone",
      "type": "string",
      "required": true,
      "visible": true,
      "criticality": "critical",
      
      "allowed_input_types": ["llm", "fixed"],
      "default_type": "llm",
      
      "llm_config": {
        "default_prompt": "Extrair telefone e converter para formato internacional (+5511987654321). Remover espaços, parênteses e hífens. Adicionar +55 se for número brasileiro sem código de país.",
        "examples": ["+5511987654321", "+5521976543210"],
        "critical_note": "Formato incorreto causará duplicação de pessoas no CRM"
      },
      
      "fixed_config": {
        "placeholder": "Ex: {{lead_phone}}",
        "auto_normalize": true,
        "allow_variables": true
      },
      
      "validation": {
        "format": "e164_international",
        "regex": "^\\+[1-9]\\d{1,14}$",
        "error_message": "Telefone deve estar no formato internacional: +5511987654321"
      },
      
      "normalization": {
        "enabled": true,
        "auto_apply": true,
        "show_preview": true,
        "rules": [
          "remove_whitespace",
          "remove_special_chars",
          "add_country_code_if_missing"
        ]
      },
      
      "ui_indicators": {
        "badge": "🔴 Campo Crítico",
        "help_text": "Este campo é usado para buscar pessoas existentes. Formato incorreto criará duplicatas!",
        "show_normalized_preview": true
      }
    }
  ],
  
  "output": {
    "provides": ["person_id", "person"],
    "structure": {
      "id": "number",
      "name": "string",
      "email": "string",
      "phone": "string"
    }
  }
}
```

---

### **Visualização da UI:**

```
╔══════════════════════════════════════════════════════════╗
║  👤 @getOrCreatePerson                      [ ✅ ON ]    ║
╠══════════════════════════════════════════════════════════╣
║  Busca pessoa no Pipedrive pelo telefone. Se não        ║
║  existir, cria nova.                                     ║
╠══════════════════════════════════════════════════════════╣
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #fullname * ℹ️                                   ⌄  │ ║
║  │                                                      │ ║
║  │ tipo                                                 │ ║
║  │ ┌──────────────┐ ┌──────────────┐                  │ ║
║  │ │ LLM Prompt ▼│ │ Fixo      ▼ │                  │ ║
║  │ └──────────────┘ └──────────────┘                  │ ║
║  │                                                      │ ║
║  │ Instrução para LLM:                                 │ ║
║  │ ┌──────────────────────────────────────────────┐   │ ║
║  │ │ Extrair o nome completo da pessoa (nome +   │   │ ║
║  │ │ sobrenome). Se não houver sobrenome, usar   │   │ ║
║  │ │ apenas o nome disponível.                    │   │ ║
║  │ └──────────────────────────────────────────────┘   │ ║
║  │                                                      │ ║
║  │ 💡 Exemplo: "João Silva"                            │ ║
║  │ ⚠️ Nome completo recomendado para melhor           │ ║
║  │    identificação                                     │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #email (opcional) ℹ️                             ⌄  │ ║
║  │                                                      │ ║
║  │ tipo: LLM Prompt                                    │ ║
║  │                                                      │ ║
║  │ Instrução para LLM:                                 │ ║
║  │ ┌──────────────────────────────────────────────┐   │ ║
║  │ │ Extrair email da conversa. Se não houver    │   │ ║
║  │ │ email disponível, retornar null.             │   │ ║
║  │ └──────────────────────────────────────────────┘   │ ║
║  │                                                      │ ║
║  │ ⚪ Campo opcional - não será enviado se vazio       │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #phone * 🔴 Campo Crítico ℹ️                     ⌄  │ ║
║  │                                                      │ ║
║  │ tipo: LLM Prompt                                    │ ║
║  │                                                      │ ║
║  │ Instrução para LLM:                                 │ ║
║  │ ┌──────────────────────────────────────────────┐   │ ║
║  │ │ Extrair telefone e converter para formato   │   │ ║
║  │ │ internacional (+5511987654321). Remover     │   │ ║
║  │ │ espaços, parênteses e hífens.                │   │ ║
║  │ └──────────────────────────────────────────────┘   │ ║
║  │                                                      │ ║
║  │ 🔄 Normalização automática:                         │ ║
║  │ Input: (11) 98765-4321                              │ ║
║  │ ↓                                                    │ ║
║  │ ✅ Enviará: +5511987654321                          │ ║
║  │                                                      │ ║
║  │ ⚠️ Este campo é usado para buscar pessoas          │ ║
║  │    existentes. Formato incorreto criará duplicatas! │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
╠══════════════════════════════════════════════════════════╣
║  Instruções Gerais:                                      ║
║  ┌──────────────────────────────────────────────────┐   ║
║  │ Sempre buscar primeiro antes de criar. Se a     │   ║
║  │ pessoa já existir, não duplicar.                 │   ║
║  └──────────────────────────────────────────────────┘   ║
╠══════════════════════════════════════════════════════════╣
║  [ Cancelar ]                              [ Salvar ]    ║
╚══════════════════════════════════════════════════════════╝
```

---

## 2️⃣ **@createDeal**

### **Descrição:**
Cria um novo deal no Pipedrive com as informações fornecidas.

### **Categoria:** Ação
### **Visível na lista @:** ✅ Sim
### **Dependências:** 🔴 @getOrCreatePerson (obrigatória)

---

### **Schema Completo:**

```json
{
  "tool": "@createDeal",
  "integration": "pipedrive",
  "category": "action",
  "visible_in_checkpoint": true,
  "display_config": {
    "title": "Criar Deal",
    "description": "Cria um novo deal no Pipedrive",
    "icon": "💼",
    "color": "#FF6B35"
  },
  
  "dependencies": [
    {
      "tool": "@getOrCreatePerson",
      "type": "required",
      "provides": "person_id",
      "error_message": "A tool @getOrCreatePerson deve estar ativa para criar um deal"
    }
  ],
  
  "parameters": [
    {
      "name": "person_id",
      "display_name": "ID da Pessoa",
      "type": "number",
      "required": true,
      "visible": false,
      "visibility_reason": "dependency_unique_value",
      "criticality": "important",
      
      "allowed_input_types": ["dependency"],
      
      "dependency_config": {
        "source_tool": "@getOrCreatePerson",
        "source_field": "id",
        "output_type": "single",
        "auto_resolve": true
      }
    },
    
    {
      "name": "title",
      "display_name": "Título do Deal",
      "type": "string",
      "required": true,
      "visible": true,
      "criticality": "important",
      
      "allowed_input_types": ["llm"],
      "default_type": "llm",
      
      "llm_config": {
        "default_prompt": "Gerar título usando o nome completo da pessoa. Formato: \"Deal - [Nome]\"",
        "examples": [
          "Deal - João Silva",
          "Deal - Empresa XYZ Ltda"
        ]
      },
      
      "validation": {
        "min_length": 1,
        "max_length": 255,
        "error_message": "Título é obrigatório e deve ter no máximo 255 caracteres"
      },
      
      "help_text": "O título aparecerá na lista de deals do Pipedrive"
    },
    
    {
      "name": "pipeline_id",
      "display_name": "Pipeline",
      "type": "number",
      "required": false,
      "visible": true,
      "criticality": "important",
      
      "allowed_input_types": ["fixed", "llm"],
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
        "instruction": {
          "required_when_multi": true,
          "placeholder": "Explique quando usar cada pipeline selecionado",
          "validation": "required_if_multiple_selected"
        }
      },
      
      "llm_config": {
        "default_prompt": "Identificar o pipeline apropriado baseado no contexto do produto/serviço mencionado",
        "examples": [
          "Pipeline de Vendas",
          "Pipeline VIP"
        ],
        "support_tool": {
          "tool": "@getAllExistingPipelines",
          "trigger": "runtime",
          "context_injection": "Pipelines disponíveis: {data}"
        }
      },
      
      "default_behavior": "Se não especificado, o Pipedrive usará o pipeline padrão da conta",
      "help_text": "Pipeline é o funil de vendas onde o deal será criado"
    },
    
    {
      "name": "stage_id",
      "display_name": "Estágio Inicial",
      "type": "number",
      "required": false,
      "visible": true,
      "criticality": "complementary",
      
      "allowed_input_types": ["fixed", "llm"],
      "default_type": "fixed",
      
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
            "loading_message": "Carregando estágios do pipeline...",
            "relationship_indicator": "🔗 Relacionado a: {pipeline_id.name}"
          }
        }
      ],
      
      "fixed_config": {
        "multi_select": false,
        "api_endpoint": {
          "method": "GET",
          "url": "/api/pipedrive/stages",
          "trigger": "on_pipeline_select",
          "frontend_call": true,
          "params": {
            "pipeline_id": "{pipeline_id}"
          },
          "response_mapping": {
            "value_field": "id",
            "label_field": "name"
          }
        }
      },
      
      "llm_config": {
        "allow_enum_restriction": true,
        "default_prompt": "Iniciar no primeiro estágio do pipeline selecionado",
        "context_injection": "Pipeline selecionado: {{pipeline_id.name}}"
      },
      
      "default_behavior": "Se não especificado, inicia no primeiro estágio do pipeline",
      "help_text": "Estágio representa a fase inicial do deal no processo de vendas"
    }
  ],
  
  "output": {
    "provides": ["deal_id", "deal"],
    "available_for_dependencies": true,
    "structure": {
      "id": "number",
      "title": "string",
      "pipeline_id": "number",
      "stage_id": "number",
      "person_id": "number",
      "status": "string"
    }
  }
}
```

---

### **Visualização da UI:**

```
╔══════════════════════════════════════════════════════════╗
║  💼 @createDeal                             [ ✅ ON ]    ║
╠══════════════════════════════════════════════════════════╣
║  Cria um novo deal no Pipedrive                          ║
╠══════════════════════════════════════════════════════════╣
║                                                           ║
║  🔴 Dependente da tool: @getOrCreatePerson               ║
║                                                           ║
╠══════════════════════════════════════════════════════════╣
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #title * ℹ️                                      ⌄  │ ║
║  │                                                      │ ║
║  │ tipo: LLM Prompt (único tipo permitido)             │ ║
║  │                                                      │ ║
║  │ Instrução para LLM:                                 │ ║
║  │ ┌──────────────────────────────────────────────┐   │ ║
║  │ │ Gerar título usando o nome completo da      │   │ ║
║  │ │ pessoa. Formato: "Deal - [Nome]"             │   │ ║
║  │ └──────────────────────────────────────────────┘   │ ║
║  │                                                      │ ║
║  │ 💡 Exemplo: "Deal - João Silva"                     │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #pipeline_id ℹ️                                  ⌄  │ ║
║  │                                                      │ ║
║  │ 🟡 Recomendado: @getAllExistingPipelines para      │ ║
║  │    contexto LLM                                      │ ║
║  │                                                      │ ║
║  │ tipo                                                 │ ║
║  │ ┌──────────────┐ ┌──────────────┐                  │ ║
║  │ │ Fixo       ▼│ │ LLM Prompt ▼│                  │ ║
║  │ └──────────────┘ └──────────────┘                  │ ║
║  │                                                      │ ║
║  │ ┌─ Tipo FIXO ─────────────────────────────────┐   │ ║
║  │ │                                              │   │ ║
║  │ │ Valores disponíveis:                        │   │ ║
║  │ │ ☑ 1 - Pipeline Vendas                       │   │ ║
║  │ │ ☑ 2 - Pipeline VIP                          │   │ ║
║  │ │ ☐ 3 - Pipeline Inbound                      │   │ ║
║  │ │                                              │   │ ║
║  │ │ ⚠️ Múltiplos valores - instrução obrigatória│   │ ║
║  │ │                                              │   │ ║
║  │ │ Instrução *:                                 │   │ ║
║  │ │ ┌──────────────────────────────────────┐    │   │ ║
║  │ │ │ Se cliente mencionar "premium" ou   │    │   │ ║
║  │ │ │ "vip", usar Pipeline VIP. Caso      │    │   │ ║
║  │ │ │ contrário, Pipeline Vendas.          │    │   │ ║
║  │ │ └──────────────────────────────────────┘    │   │ ║
║  │ │                                              │   │ ║
║  │ └──────────────────────────────────────────────┘   │ ║
║  │                                                      │ ║
║  │ ┌─ Tipo LLM PROMPT ────────────────────────────┐   │ ║
║  │ │                                              │   │ ║
║  │ │ ℹ️ Tool de suporte:                         │   │ ║
║  │ │    @getAllExistingPipelines                 │   │ ║
║  │ │                                              │   │ ║
║  │ │ ℹ️ Campos disponíveis do Pipeline:          │   │ ║
║  │ │ ┌──────────────────────────────────────┐    │   │ ║
║  │ │ │ • id (number)                        │    │   │ ║
║  │ │ │ • name (string)                      │    │   │ ║
║  │ │ │ • stages (array)                     │    │   │ ║
║  │ │ │ • deal_probability (boolean)         │    │   │ ║
║  │ │ └──────────────────────────────────────┘    │   │ ║
║  │ │                                              │   │ ║
║  │ │ Instrução para LLM:                          │   │ ║
║  │ │ ┌──────────────────────────────────────┐    │   │ ║
║  │ │ │ Usar pipeline que contenha "VIP"    │    │   │ ║
║  │ │ │ no campo name                        │    │   │ ║
║  │ │ └──────────────────────────────────────┘    │   │ ║
║  │ │                                              │   │ ║
║  │ │ 💡 Você pode usar: id, name, stages,        │   │ ║
║  │ │    deal_probability                          │   │ ║
║  │ └──────────────────────────────────────────────┘   │ ║
║  │                                                      │ ║
║  │ 💡 Se não fornecido, usa pipeline padrão da conta  │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #stage_id ℹ️                                     ⌄  │ ║
║  │                                                      │ ║
║  │ 🔗 Relacionado a: Pipeline Vendas                   │ ║
║  │                                                      │ ║
║  │ tipo: Fixo                                          │ ║
║  │                                                      │ ║
║  │ Valores disponíveis:                                │ ║
║  │ ┌──────────────────────────────────────────────┐   │ ║
║  │ │ ○ 1 - Qualificação                           │   │ ║
║  │ │ ● 2 - Proposta Enviada                       │   │ ║
║  │ │ ○ 3 - Negociação                             │   │ ║
║  │ │ ○ 4 - Fechamento                             │   │ ║
║  │ └──────────────────────────────────────────────┘   │ ║
║  │                                                      │ ║
║  │ 💡 Se não fornecido, usa primeiro estágio           │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
╠══════════════════════════════════════════════════════════╣
║  Instruções Gerais:                                      ║
║  ┌──────────────────────────────────────────────────┐   ║
║  │ Criar deal apenas se valor mencionado for        │   ║
║  │ superior a R$ 1.000. Priorizar velocidade.       │   ║
║  └──────────────────────────────────────────────────┘   ║
╠══════════════════════════════════════════════════════════╣
║  [ Cancelar ]                              [ Salvar ]    ║
╚══════════════════════════════════════════════════════════╝
```

---

## 3️⃣ **@updateDeal**

### **Descrição:**
Atualiza um deal existente no Pipedrive com novos dados.

### **Categoria:** Ação
### **Visível na lista @:** ✅ Sim
### **Dependências:** 🟡 @getAllExistingDealsFromPerson (quando deal_id tipo=Dependência)

---

### **Schema Completo:**

```json
{
  "tool": "@updateDeal",
  "integration": "pipedrive",
  "category": "action",
  "visible_in_checkpoint": true,
  "display_config": {
    "title": "Atualizar Deal",
    "description": "Atualiza dados de um deal existente no Pipedrive",
    "icon": "✏️",
    "color": "#2196F3"
  },
  
  "parameters": [
    {
      "name": "deal_id",
      "display_name": "ID do Deal",
      "type": "number",
      "required": true,
      "visible": true,
      "criticality": "important",
      
      "allowed_input_types": ["dependency"],
      "default_type": "dependency",
      
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
          },
          {
            "name": "updated_at",
            "type": "datetime",
            "description": "Data da última atualização"
          }
        ],
        "selection_strategy": {
          "type": "llm_choose",
          "options": [
            {
              "value": "first",
              "label": "Primeiro deal da lista"
            },
            {
              "value": "last",
              "label": "Último deal (mais recente)"
            },
            {
              "value": "by_criteria",
              "label": "Por critério personalizado",
              "requires_prompt": true,
              "prompt_hint": "Use os campos disponíveis acima"
            }
          ]
        }
      }
    },
    
    {
      "name": "title",
      "display_name": "Título",
      "type": "string",
      "required": false,
      "visible": true,
      "criticality": "complementary",
      
      "allowed_input_types": ["llm", "fixed"],
      "default_type": "llm",
      
      "llm_config": {
        "default_prompt": "Atualizar título do deal se necessário"
      },
      
      "fixed_config": {
        "placeholder": "Novo título do deal"
      },
      
      "validation": {
        "max_length": 255
      }
    },
    
    {
      "name": "stage_id",
      "display_name": "Mover para Estágio",
      "type": "number",
      "required": false,
      "visible": true,
      "criticality": "important",
      
      "allowed_input_types": ["fixed", "llm"],
      "default_type": "fixed",
      
      "fixed_config": {
        "api_endpoint": {
          "method": "GET",
          "url": "/api/pipedrive/stages",
          "trigger": "on_modal_open",
          "frontend_call": true
        }
      },
      
      "llm_config": {
        "default_prompt": "Mover para estágio apropriado baseado no status da conversa",
        "allow_enum_restriction": true
      }
    },
    
    {
      "name": "value",
      "display_name": "Valor do Deal",
      "type": "number",
      "required": false,
      "visible": true,
      "criticality": "important",
      
      "allowed_input_types": ["llm", "fixed"],
      "default_type": "llm",
      
      "llm_config": {
        "default_prompt": "Extrair valor monetário mencionado na conversa"
      },
      
      "fixed_config": {
        "placeholder": "Ex: 5000"
      }
    },
    
    {
      "name": "status",
      "display_name": "Status",
      "type": "enum",
      "required": false,
      "visible": true,
      "criticality": "important",
      
      "allowed_input_types": ["fixed", "llm"],
      "default_type": "fixed",
      
      "fixed_config": {
        "values": [
          {"value": "open", "label": "Aberto"},
          {"value": "won", "label": "Ganho"},
          {"value": "lost", "label": "Perdido"}
        ]
      },
      
      "llm_config": {
        "default_prompt": "Determinar status baseado no resultado da conversa",
        "allow_enum_restriction": true,
        "enum_values": ["open", "won", "lost"]
      }
    },
    
    {
      "name": "lost_reason",
      "display_name": "Motivo da Perda",
      "type": "string",
      "required": false,
      "visible": true,
      "criticality": "complementary",
      
      "allowed_input_types": ["llm"],
      "default_type": "llm",
      
      "parameter_relationships": [
        {
          "depends_on_parameter": "status",
          "type": "conditional",
          "show_when": "status === 'lost'",
          "becomes_required_when": "status === 'lost'"
        }
      ],
      
      "llm_config": {
        "default_prompt": "Extrair motivo da perda mencionado pelo cliente"
      }
    }
  ],
  
  "output": {
    "provides": ["deal"],
    "structure": {
      "id": "number",
      "title": "string",
      "status": "string",
      "value": "number"
    }
  }
}
```

---

### **Visualização da UI:**

```
╔══════════════════════════════════════════════════════════╗
║  ✏️ @updateDeal                             [ ✅ ON ]    ║
╠══════════════════════════════════════════════════════════╣
║  Atualiza dados de um deal existente no Pipedrive        ║
╠══════════════════════════════════════════════════════════╣
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #deal_id * ℹ️                                    ⌄  │ ║
║  │                                                      │ ║
║  │ tipo: Dependência (único tipo permitido)            │ ║
║  │                                                      │ ║
║  │ 🔗 Depende de: @getAllExistingDealsFromPerson       │ ║
║  │ ⚠️ Esta tool retorna múltiplos deals                │ ║
║  │                                                      │ ║
║  │ ┌─ ℹ️ Campos disponíveis do Deal ──────────────┐   │ ║
║  │ │ • id (number)                                │   │ ║
║  │ │ • title (string)                             │   │ ║
║  │ │ • status (enum): open, won, lost             │   │ ║
║  │ │ • value (number)                             │   │ ║
║  │ │ • stage_current (string)                     │   │ ║
║  │ │ • created_at (datetime)                      │   │ ║
║  │ │ • updated_at (datetime)                      │   │ ║
║  │ └──────────────────────────────────────────────┘   │ ║
║  │                                                      │ ║
║  │ Critério para seleção:                              │ ║
║  │ ┌────────────────────────────────────────────┐     │ ║
║  │ │ Escolher deal com status open e valor     │     │ ║
║  │ │ maior que 1000                             │     │ ║
║  │ └────────────────────────────────────────────┘     │ ║
║  │                                                      │ ║
║  │ 💡 Use os campos disponíveis acima                 │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #title (opcional) ℹ️                             ⌄  │ ║
║  │                                                      │ ║
║  │ tipo: LLM Prompt                                    │ ║
║  │                                                      │ ║
║  │ Instrução:                                          │ ║
║  │ ┌──────────────────────────────────────────────┐   │ ║
║  │ │ Atualizar título do deal se necessário       │   │ ║
║  │ └──────────────────────────────────────────────┘   │ ║
║  │                                                      │ ║
║  │ ⚪ Campo opcional                                   │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #status (opcional) ℹ️                            ⌄  │ ║
║  │                                                      │ ║
║  │ tipo: Fixo                                          │ ║
║  │                                                      │ ║
║  │ ○ Aberto (open)                                     │ ║
║  │ ● Ganho (won)                                       │ ║
║  │ ○ Perdido (lost)                                    │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #lost_reason * ℹ️ (aparece se status=lost)      ⌄  │ ║
║  │                                                      │ ║
║  │ tipo: LLM Prompt                                    │ ║
║  │                                                      │ ║
║  │ Instrução:                                          │ ║
║  │ ┌──────────────────────────────────────────────┐   │ ║
║  │ │ Extrair motivo da perda mencionado           │   │ ║
║  │ └──────────────────────────────────────────────┘   │ ║
║  │                                                      │ ║
║  │ ⚠️ Obrigatório quando status = "lost"              │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
║  [ + Mostrar parâmetros avançados ]                      ║
║                                                           ║
╠══════════════════════════════════════════════════════════╣
║  [ Cancelar ]                              [ Salvar ]    ║
╚══════════════════════════════════════════════════════════╝
```

---

## 4️⃣ **@createNote**

### **Descrição:**
Cria uma nota para um deal específico no Pipedrive.

### **Categoria:** Ação
### **Visível na lista @:** ✅ Sim
### **Dependências:** 🟡 @createDeal ou @getAllExistingDealsFromPerson (para deal_id)

---

### **Schema Completo:**

```json
{
  "tool": "@createNote",
  "integration": "pipedrive",
  "category": "action",
  "visible_in_checkpoint": true,
  "display_config": {
    "title": "Criar Nota",
    "description": "Adiciona uma nota a um deal no Pipedrive",
    "icon": "📝",
    "color": "#FFC107"
  },
  
  "parameters": [
    {
      "name": "deal_id",
      "display_name": "ID do Deal",
      "type": "number",
      "required": true,
      "visible": true,
      "criticality": "important",
      
      "allowed_input_types": ["dependency"],
      "default_type": "dependency",
      
      "dependency_config": {
        "possible_sources": [
          {
            "tool": "@createDeal",
            "field": "id",
            "label": "Deal recém-criado",
            "output_type": "single"
          },
          {
            "tool": "@getAllExistingDealsFromPerson",
            "field": "id",
            "label": "Deal existente",
            "output_type": "array",
            "requires_selection": true
          }
        ]
      }
    },
    
    {
      "name": "content",
      "display_name": "Conteúdo da Nota",
      "type": "string",
      "required": true,
      "visible": true,
      "criticality": "important",
      
      "allowed_input_types": ["llm"],
      "default_type": "llm",
      
      "llm_config": {
        "default_prompt": "Criar um resumo da conversa com os pontos principais discutidos",
        "examples": [
          "Cliente interessado em produto X. Mencionou orçamento de R$5000.",
          "Reunião agendada para próxima semana. Cliente quer demonstração."
        ]
      },
      
      "validation": {
        "min_length": 1,
        "error_message": "Conteúdo da nota é obrigatório"
      }
    }
  ],
  
  "output": {
    "provides": ["note_id", "note"],
    "structure": {
      "id": "number",
      "content": "string",
      "deal_id": "number"
    }
  }
}
```

---

### **Visualização da UI:**

```
╔══════════════════════════════════════════════════════════╗
║  📝 @createNote                             [ ✅ ON ]    ║
╠══════════════════════════════════════════════════════════╣
║  Adiciona uma nota a um deal no Pipedrive                ║
╠══════════════════════════════════════════════════════════╣
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #deal_id * ℹ️                                    ⌄  │ ║
║  │                                                      │ ║
║  │ tipo: Dependência (único tipo permitido)            │ ║
║  │                                                      │ ║
║  │ Deal vem de:                                        │ ║
║  │ ● @createDeal (deal recém-criado)                   │ ║
║  │ ○ @getAllExistingDealsFromPerson (deal existente)  │ ║
║  │                                                      │ ║
║  │ ✅ Usando deal criado anteriormente no checkpoint   │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #content * ℹ️                                    ⌄  │ ║
║  │                                                      │ ║
║  │ tipo: LLM Prompt (único tipo permitido)             │ ║
║  │                                                      │ ║
║  │ Instrução para LLM:                                 │ ║
║  │ ┌──────────────────────────────────────────────┐   │ ║
║  │ │ Criar um resumo da conversa com os pontos   │   │ ║
║  │ │ principais discutidos. Incluir:              │   │ ║
║  │ │ - Interesse do cliente                       │   │ ║
║  │ │ - Orçamento mencionado                       │   │ ║
║  │ │ - Próximos passos                            │   │ ║
║  │ │                                              │   │ ║
║  │ │                                              │   │ ║
║  │ │                                              │   │ ║
║  │ └──────────────────────────────────────────────┘   │ ║
║  │                                                      │ ║
║  │ 💡 Exemplo: "Cliente interessado em produto X..."  │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
╠══════════════════════════════════════════════════════════╣
║  Instruções Gerais:                                      ║
║  ┌──────────────────────────────────────────────────┐   ║
║  │ Sempre adicionar nota após interação importante. │   ║
║  └──────────────────────────────────────────────────┘   ║
╠══════════════════════════════════════════════════════════╣
║  [ Cancelar ]                              [ Salvar ]    ║
╚══════════════════════════════════════════════════════════╝
```

---

## 5️⃣ **@updateNote**

### **Descrição:**
Atualiza o conteúdo de uma nota existente no Pipedrive.

### **Categoria:** Ação
### **Visível na lista @:** ✅ Sim
### **Dependências:** 🔴 @createNote (para note_id)

---

### **Schema Completo:**

```json
{
  "tool": "@updateNote",
  "integration": "pipedrive",
  "category": "action",
  "visible_in_checkpoint": true,
  "display_config": {
    "title": "Atualizar Nota",
    "description": "Atualiza conteúdo de nota existente",
    "icon": "✏️📝",
    "color": "#FF9800"
  },
  
  "parameters": [
    {
      "name": "note_id",
      "display_name": "ID da Nota",
      "type": "number",
      "required": true,
      "visible": false,
      "visibility_reason": "dependency_unique_value",
      "criticality": "important",
      
      "allowed_input_types": ["dependency"],
      
      "dependency_config": {
        "source_tool": "@createNote",
        "source_field": "id",
        "output_type": "single",
        "auto_resolve": true
      }
    },
    
    {
      "name": "content",
      "display_name": "Novo Conteúdo",
      "type": "string",
      "required": true,
      "visible": true,
      "criticality": "important",
      
      "allowed_input_types": ["llm"],
      "default_type": "llm",
      
      "llm_config": {
        "default_prompt": "Atualizar nota com informações adicionais da conversa"
      },
      
      "validation": {
        "min_length": 1
      }
    }
  ],
  
  "output": {
    "provides": ["note"],
    "structure": {
      "id": "number",
      "content": "string"
    }
  }
}
```

---

## 6️⃣ **@createDealActivity**

### **Descrição:**
Cria uma atividade (meeting) para um deal específico no Pipedrive.

### **Categoria:** Ação
### **Visível na lista @:** ✅ Sim
### **Dependências:** 🟡 @createDeal ou @getAllExistingDealsFromPerson (para deal_id)

---

### **Schema Completo:**

```json
{
  "tool": "@createDealActivity",
  "integration": "pipedrive",
  "category": "action",
  "visible_in_checkpoint": true,
  "display_config": {
    "title": "Criar Atividade",
    "description": "Agenda uma atividade (meeting) para um deal",
    "icon": "📅",
    "color": "#9C27B0"
  },
  
  "parameters": [
    {
      "name": "deal_id",
      "display_name": "ID do Deal",
      "type": "number",
      "required": true,
      "visible": true,
      "criticality": "important",
      
      "allowed_input_types": ["dependency"],
      "default_type": "dependency",
      
      "dependency_config": {
        "possible_sources": [
          {
            "tool": "@createDeal",
            "field": "id",
            "output_type": "single"
          },
          {
            "tool": "@getAllExistingDealsFromPerson",
            "field": "id",
            "output_type": "array",
            "requires_selection": true
          }
        ]
      }
    },
    
    {
      "name": "subject",
      "display_name": "Nome da Atividade",
      "type": "string",
      "required": true,
      "visible": true,
      "criticality": "important",
      
      "allowed_input_types": ["llm", "fixed"],
      "default_type": "llm",
      
      "llm_config": {
        "default_prompt": "Criar nome curto e objetivo para a atividade",
        "examples": [
          "Reunião de Apresentação",
          "Call de Follow-up",
          "Demonstração do Produto"
        ]
      }
    },
    
    {
      "name": "due_date",
      "display_name": "Data",
      "type": "string",
      "required": true,
      "visible": true,
      "criticality": "important",
      
      "allowed_input_types": ["llm", "fixed"],
      "default_type": "llm",
      
      "llm_config": {
        "default_prompt": "Calcular data no formato YYYY-MM-DD. Interpretar linguagem natural como 'amanhã', 'próxima segunda', 'daqui 3 dias'",
        "examples": ["2025-11-07", "2025-11-10"]
      },
      
      "fixed_config": {
        "input_type": "date",
        "placeholder": "YYYY-MM-DD"
      },
      
      "validation": {
        "format": "date",
        "regex": "^\\d{4}-\\d{2}-\\d{2}$"
      }
    },
    
    {
      "name": "due_time",
      "display_name": "Horário",
      "type": "string",
      "required": true,
      "visible": true,
      "criticality": "important",
      
      "allowed_input_types": ["llm", "fixed"],
      "default_type": "llm",
      
      "llm_config": {
        "default_prompt": "Extrair horário no formato HH:mm. Se não mencionado, sugerir horário comercial (14:00)",
        "examples": ["14:00", "09:30", "16:00"]
      },
      
      "fixed_config": {
        "input_type": "time",
        "placeholder": "HH:mm"
      },
      
      "validation": {
        "format": "time",
        "regex": "^([0-1][0-9]|2[0-3]):[0-5][0-9]$"
      }
    },
    
    {
      "name": "duration",
      "display_name": "Duração",
      "type": "string",
      "required": false,
      "visible": true,
      "criticality": "complementary",
      
      "allowed_input_types": ["llm", "fixed"],
      "default_type": "fixed",
      
      "llm_config": {
        "default_prompt": "Extrair duração no formato HH:mm"
      },
      
      "fixed_config": {
        "predefined_values": [
          {"value": "00:30", "label": "30 minutos"},
          {"value": "01:00", "label": "1 hora"},
          {"value": "01:30", "label": "1 hora e 30 min"},
          {"value": "02:00", "label": "2 horas"}
        ]
      }
    },
    
    {
      "name": "note",
      "display_name": "Nota Interna",
      "type": "string",
      "required": false,
      "visible": true,
      "criticality": "complementary",
      
      "allowed_input_types": ["llm"],
      "default_type": "llm",
      
      "llm_config": {
        "default_prompt": "Criar nota interna com contexto para quem fará a call"
      }
    },
    
    {
      "name": "attendees",
      "display_name": "Participantes",
      "type": "array",
      "required": true,
      "visible": true,
      "criticality": "important",
      
      "allowed_input_types": ["llm", "fixed"],
      "default_type": "llm",
      
      "llm_config": {
        "default_prompt": "Incluir a pessoa do deal e outros participantes mencionados. Formato: [{name, email}]",
        "array_handling": "merge_with_person"
      },
      
      "fixed_config": {
        "predefined_values": [
          {
            "value": "person_from_deal",
            "label": "Pessoa do deal (obrigatório)"
          }
        ],
        "allow_add_more": true
      }
    }
  ],
  
  "output": {
    "provides": ["activity_id", "activity"],
    "structure": {
      "id": "number",
      "subject": "string",
      "due_date": "string",
      "due_time": "string"
    }
  }
}
```

---

### **Visualização da UI:**

```
╔══════════════════════════════════════════════════════════╗
║  📅 @createDealActivity                     [ ✅ ON ]    ║
╠══════════════════════════════════════════════════════════╣
║  Agenda uma atividade (meeting) para um deal             ║
╠══════════════════════════════════════════════════════════╣
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #deal_id * ℹ️                                    ⌄  │ ║
║  │                                                      │ ║
║  │ tipo: Dependência                                   │ ║
║  │ Deal vem de: @createDeal                            │ ║
║  │ ✅ Usando deal criado no checkpoint                 │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #subject * ℹ️                                    ⌄  │ ║
║  │                                                      │ ║
║  │ tipo: LLM Prompt                                    │ ║
║  │                                                      │ ║
║  │ Instrução:                                          │ ║
║  │ ┌──────────────────────────────────────────────┐   │ ║
║  │ │ Criar nome curto e objetivo para atividade  │   │ ║
║  │ └──────────────────────────────────────────────┘   │ ║
║  │                                                      │ ║
║  │ 💡 Ex: "Reunião de Apresentação"                    │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #due_date * ℹ️                                   ⌄  │ ║
║  │                                                      │ ║
║  │ tipo: LLM Prompt                                    │ ║
║  │                                                      │ ║
║  │ Instrução:                                          │ ║
║  │ ┌──────────────────────────────────────────────┐   │ ║
║  │ │ Calcular data no formato YYYY-MM-DD.        │   │ ║
║  │ │ Interpretar: "amanhã", "próxima segunda",   │   │ ║
║  │ │ "daqui 3 dias"                               │   │ ║
║  │ └──────────────────────────────────────────────┘   │ ║
║  │                                                      │ ║
║  │ 💡 Ex: "2025-11-07"                                 │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #due_time * ℹ️                                   ⌄  │ ║
║  │                                                      │ ║
║  │ tipo: LLM Prompt                                    │ ║
║  │                                                      │ ║
║  │ Instrução:                                          │ ║
║  │ ┌──────────────────────────────────────────────┐   │ ║
║  │ │ Extrair horário no formato HH:mm. Se não    │   │ ║
║  │ │ mencionado, sugerir 14:00                    │   │ ║
║  │ └──────────────────────────────────────────────┘   │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #duration (opcional) ℹ️                          ⌄  │ ║
║  │                                                      │ ║
║  │ tipo: Fixo                                          │ ║
║  │                                                      │ ║
║  │ ○ 30 minutos (00:30)                                │ ║
║  │ ● 1 hora (01:00)                                    │ ║
║  │ ○ 1 hora e 30 min (01:30)                           │ ║
║  │ ○ 2 horas (02:00)                                   │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
║  ┌─────────────────────────────────────────────────────┐ ║
║  │ #attendees * ℹ️                                  ⌄  │ ║
║  │                                                      │ ║
║  │ tipo: LLM Prompt                                    │ ║
║  │                                                      │ ║
║  │ Instrução:                                          │ ║
║  │ ┌──────────────────────────────────────────────┐   │ ║
║  │ │ Incluir pessoa do deal + outros             │   │ ║
║  │ │ participantes mencionados                    │   │ ║
║  │ └──────────────────────────────────────────────┘   │ ║
║  │                                                      │ ║
║  │ ✅ Pessoa do deal incluída automaticamente          │ ║
║  └─────────────────────────────────────────────────────┘ ║
║                                                           ║
║  [ + Mostrar parâmetros avançados ]                      ║
║                                                           ║
╠══════════════════════════════════════════════════════════╣
║  [ Cancelar ]                              [ Salvar ]    ║
╚══════════════════════════════════════════════════════════╝
```

---

## 📊 **Tabela Resumo de Todas as Tools**

| Tool | Visível @ | Categoria | Parâmetros Visíveis | Dependências | Output |
|------|-----------|-----------|---------------------|--------------|--------|
| @getOrCreatePerson | ✅ | Híbrida | 3 (fullname, email, phone) | ❌ Nenhuma | person_id |
| @createDeal | ✅ | Ação | 3 (title, pipeline_id, stage_id) | 🔴 @getOrCreatePerson | deal_id |
| @updateDeal | ✅ | Ação | 6 (deal_id, title, stage_id, value, status, lost_reason) | 🟡 @getAllExistingDealsFromPerson | deal |
| @createNote | ✅ | Ação | 2 (deal_id, content) | 🟡 @createDeal ou @getAllExisting... | note_id |
| @updateNote | ✅ | Ação | 1 (content) | 🔴 @createNote | note |
| @createDealActivity | ✅ | Ação | 7 (deal_id, subject, due_date, due_time, duration, note, attendees) | 🟡 @createDeal ou @getAllExisting... | activity_id |

---

## 🔄 **Fluxos Típicos de Uso**

### **Fluxo 1: Criar Deal Completo com Atividade**

```
1. @getOrCreatePerson
   ↓ fornece: person_id
   
2. @createDeal
   ↓ usa: person_id (oculto)
   ↓ fornece: deal_id
   
3. @createNote
   ↓ usa: deal_id (dependência de @createDeal)
   
4. @createDealActivity
   ↓ usa: deal_id (dependência de @createDeal)
```

### **Fluxo 2: Atualizar Deal Existente**

```
1. @getOrCreatePerson
   ↓ fornece: person_id
   
2. @updateDeal
   ↓ invoca automaticamente: @getAllExistingDealsFromPerson
   ↓ usuário define critério de seleção
   ↓ atualiza deal selecionado
```

---

**Versão:** 1.0  
**Data:** 2025-11-06  
**Status:** Documentação Completa  
**Baseado em:** regras-parametros-tools.md v3.1

