# 🎯 Framework de Regras para Parâmetros de Tools

Sistema universal de regras para configuração de parâmetros de tools em qualquer integração.

**Versão:** 3.8.0  
**Data:** 2025-11-13  
**Status:** Pronto para implementação


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

## ⚡ **Fluxo de Runtime (Execução)**

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
    │ Para cada parâmetro:           │
    │ 1. Resolver DEPENDÊNCIAS       │
    │    - Invocar tool GET          │
    │    - Se array, aplicar critério│
    │ 2. Chamar SUPPORT TOOLS (p/ LLM)│
    │    - Injetar contexto extra    │
    └────────┬───────────────────────┘
             │
             ▼
    ┌────────────────────────────────┐
    │ LLM processa com:              │
    │ • Contexto da conversa         │
    │ • Dados das tools de suporte   │
    │ • Instruções do usuário        │
    │ • Valores fixos permitidos     │
    └────────┬───────────────────────┘
             │
             ▼
    ┌────────────────────────────────┐
    │ Validações Finais do Sistema:  │
    │ • Normalização (campos críticos)│
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

---

## 👁️ **Regra 1: Visibilidade de Tools no Checkpoint**

Define quais tools aparecem na lista `@` para o usuário configurar explicitamente. A regra geral é: **actions são visíveis, queries são ocultas e automáticas**.

| Prefixo | Visível na Lista @? | Categoria | Invocação | Exemplo |
|---|---|---|---|---|
| `create`, `update`, `delete`, `add` | ✅ **SIM** | Ação | Explícita pelo usuário | `@createDeal` |
| `getOrCreate` | ✅ **SIM** | Híbrida | Explícita pelo usuário | `@getOrCreatePerson` |
| `get*` (comum) | ❌ **NÃO** | Consulta Automática | Por dependência ou suporte | `@getAllExistingDealsFromPerson` |
| `get*` (exceção) | ✅ **SIM** (com ícone 🔍) | Consulta Visível | Explícita pelo usuário | `@getDealWithCompleteInfo` |

### **Critérios:**

-   **MOSTRAR (Ações e Híbridas):** Tools que modificam dados (`create`, `update`, `delete`) ou que podem modificar (`getOrCreate`) devem ser configuradas pelo usuário.
-   **OCULTAR (Consultas Automáticas):** Tools que apenas leem dados (`get`, `getAll`) são, por padrão, invocadas automaticamente pelo sistema para satisfazer uma dependência ou para fornecer contexto a um parâmetro LLM. O usuário não as vê nem as configura diretamente.
-   **MOSTRAR (Consulta Visível - Exceção):** Ocasionalmente, uma tool `get` pode ser útil para o usuário executar diretamente (ex: para verificar o status de algo). Nesses casos, ela pode ser marcada como visível e aparecerá com um ícone de busca (🔍) para diferenciá-la das ações.

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
                        │ 4️⃣ Definir `input_type`             │ (REGRA 3)
                        │    (v3.8.0: APENAS UM tipo)          │
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
    │ Schema completo v3.8.0 com:              │
    │ • name, display_name, type               │
    │ • required, visible                      │
    │ • input_type (v3.8.0: único)             │
    │ • config (v3.8.0: unificado)             │
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

Define se um parâmetro deve ser configurável pelo usuário ou resolvido automaticamente pelo sistema.

### Critérios para Ocultar (Resolver Automaticamente):
- **Dependência com Valor Único:** Ocultar o parâmetro se seu valor vem de uma tool que retorna **um único resultado**. Não há decisão para o usuário tomar.
  - **Exemplo:** `person_id` em `@createDeal` vem de `@getOrCreatePerson`, que retorna apenas uma pessoa. O sistema usa o ID automaticamente.
- **Parâmetros Técnicos:** Ocultar campos internos do sistema que não têm valor de configuração para o usuário (ex: flags booleanas fixas).

### Critérios para Mostrar (Requer Configuração):
- **Dependência com Array de Valores:** Mostrar o parâmetro se a tool de origem retorna **múltiplos resultados**. O usuário precisa definir um critério para selecionar o item correto.
  - **Exemplo:** `deal_id` em `@updateDeal` vem de `@getAllExistingDealsFromPerson`, que retorna uma lista de deals. O usuário deve especificar qual deal atualizar (ex: "o mais recente com status 'open'").
- **Decisão de Negócio:** Sempre mostrar campos que exigem uma decisão estratégica ou de negócio do usuário (ex: `pipeline_id`, `stage_id`, `currency`).
- **Conteúdo Customizável:** Sempre mostrar campos de texto livre que a LLM irá gerar (ex: `title`, `content`, `description`).

---

## 🎚️ **Regra 3: Tipos de Preenchimento Permitidos**

> **Simplificação v3.8:** Cada parâmetro agora possui apenas **um** `input_type` (`fixed`, `llm`, ou `dependency`), eliminando a complexidade de permitir múltiplos tipos.

A escolha do tipo de input depende de como a API externa valida o dado e se o valor é estático ou dinâmico.

### **Matriz de Decisão Rápida:**

| Tipo de Dado | API Valida? | Tipo de Input | Justificativa |
|---|---|---|---|
| **ID de recurso pré-existente** (ex: `pipeline_id`, `user_id`) | ✅ Sim (só aceita IDs válidos) | `fixed` | A LLM não pode "inventar" um ID. O valor deve ser selecionado de uma lista carregada da API. |
| **ID de dependência** (ex: `deal_id` de `getAllDeals`) | ✅ Sim (resolvido por outra tool) | `dependency` | O valor é o resultado de outra tool. O sistema resolve a dependência automaticamente. |
| **Enum validado pela API** (ex: `status`, `currency`) | ✅ Sim (rejeita valores inválidos) | `fixed` | A API só aceita valores de uma lista restrita. A seleção deve ser exata. |
| **Texto/String livre** (ex: `title`, `note`, `lost_reason`) | ❌ Não (aceita qualquer string) | `llm` | O conteúdo é dinâmico e gerado/extraído do contexto da conversa pela LLM. |
| **Data/Hora/Valor contextual** (ex: `due_date`, `value`) | ❌ Não (valida formato, não valor) | `llm` | Quase sempre extraído dinamicamente da conversa (ex: "marcar para amanhã", "valor de R$5000"). |
| **Valores com presets fixos** (ex: `duration`) | ❌ Não (valida formato) | `fixed` | A empresa geralmente tem durações padrão (30min, 1h). Uma lista fixa é mais prática. |
| **Booleano** (ex: `done: true`) | ✅ Sim | `fixed` | Uma simples seleção entre `true` e `false`. |

---

### **Regras Específicas por Tipo:**

#### **A. Tipo `fixed`**

Usado quando os valores permitidos vêm de uma **lista finita e conhecida**, seja de uma API ou de regras de negócio.

- **Como funciona:** O usuário seleciona um ou mais valores permitidos durante a configuração. Em tempo de execução, a LLM escolhe **entre as opções pré-selecionadas** com base na instrução fornecida.
- **Exemplos:** `pipeline_id`, `stage_id`, `user_id`, `status`, `currency`, `duration`.
- **Multi-Select:** Se múltiplos valores são permitidos (ex: "usar pipeline A ou B"), é **obrigatório** fornecer uma instrução clara de quando usar cada um.

#### **B. Tipo `llm`**

Usado para **qualquer valor que precise ser extraído ou gerado dinamicamente** a partir do contexto da conversa.

- **Como funciona:** O usuário fornece uma instrução para a LLM (ex: "Extrair o e-mail do cliente da conversa"). O sistema pode, opcionalmente, usar uma `support_tool` para buscar dados que ajudem a LLM a tomar uma decisão (ex: buscar templates de e-mail).
- **Exemplos:** `title`, `description`, `note`, `fullname`, `email`, `phone`, `value`, `due_date`.
- **Preview de Campos:** Se uma `support_tool` é usada, a UI mostra os campos que a tool retorna (ex: `id`, `name`, `subject` de um template de e-mail) para ajudar o usuário a escrever uma instrução mais precisa.

#### **C. Tipo `dependency`**

Usado quando o valor de um parâmetro é o **resultado direto de outra tool**.

- **Como funciona:** O sistema invoca a tool de origem automaticamente.
  - Se a tool retorna um **valor único** (ex: `person_id` de `@getOrCreatePerson`), o parâmetro é **oculto** do usuário e o valor é usado diretamente.
  - Se a tool retorna um **array de valores** (ex: múltiplos deals de `@getAllExistingDealsFromPerson`), o parâmetro é **visível**, e o usuário deve fornecer um critério em linguagem natural para que a LLM selecione o item correto do array (ex: "Deal com status 'open' e valor maior que 1000").
- **Preview de Campos:** Para dependências de array, a UI sempre mostra os campos disponíveis do objeto retornado para ajudar a escrever o critério de seleção.

---

## 🔢 **Regra 4: Multi-Select**

Aplica-se apenas a parâmetros do tipo `fixed`.

**REGRA CRÍTICA:** Se o usuário selecionar múltiplos valores permitidos, um campo de **instrução se torna obrigatório**.

- **Exemplo:** Para `pipeline_id`, se o usuário selecionar "Pipeline Vendas" e "Pipeline VIP", ele **deve** preencher o campo de instrução, explicando quando a LLM deve escolher cada um (ex: "Se o cliente mencionar 'premium' ou 'vip', usar Pipeline VIP. Caso contrário, usar Pipeline Vendas.").
- **Validação:** A UI deve impedir que a configuração seja salva se múltiplos valores forem selecionados sem uma instrução.

---

## 🧩 **Regra 5: Dependências e Relacionamentos**

Define como os parâmetros se relacionam com outras tools ou com outros parâmetros.

### Tipos de Dependência:

1.  **Dependência entre Tools:** Uma tool de ação (ex: `@createDeal`) pode depender de uma tool de consulta (ex: `@getOrCreatePerson`) para obter um valor. A UI deve indicar essa dependência e validar se a tool necessária está ativa.

2.  **Relacionamento entre Parâmetros:** O valor ou a visibilidade de um parâmetro pode depender de outro na mesma tool. A UI deve ser dinâmica.
    -   **Exemplo:** O campo `stage_id` só deve ser habilitado e carregar suas opções *após* um `pipeline_id` ter sido selecionado.

### Preview de Campos: Ajudando o Usuário

**REGRA CRÍTICA DE UX:** Para que o usuário possa escrever instruções de critério eficazes, ele precisa saber quais dados estão disponíveis.

O sistema deve mostrar um **preview dos campos retornados** pela tool de suporte sempre que:
-   O `input_type` for `dependency` e a `source_tool` retornar um array.
-   O `input_type` for `llm` e o parâmetro tiver uma `support_tool` associada.

**Exemplo de UI para uma dependência de array:**
```
🔗 Depende de: @getAllExistingDealsFromPerson
⚠️ Esta tool retorna múltiplos deals. Defina um critério para a seleção.

ℹ️ Campos disponíveis do Deal:
  • id (number)
  • title (string)
  • status (string): open, won, lost
  • value (number)
  • created_at (datetime)

Critério para seleção:
┌──────────────────────────────────┐
│ Deal com status "open" e valor   │
│ maior que 1000                   │
└──────────────────────────────────┘
```
Isso capacita o usuário a escrever critérios precisos (ex: `status = "open" AND value > 1000`) porque ele vê a estrutura de dados real.

---

## 🔐 **Regra 6: Obrigatoriedade**

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

## 📋 **Regra 7: Instruções Gerais vs Instruções por Parâmetro**

### **A. Separação de Responsabilidades (CRÍTICO):**

**🚫 Usuário NÃO configura lógica técnica:**
```
❌ Errado: "Converter telefone para formato internacional (+5511987654321)"
✅ Correto: "Extrair telefone da conversa"
```

-   **Responsabilidade do Sistema:** Normalização, validação e conversão de formatos de dados.
-   **Responsabilidade do Usuário:** Definir a lógica de negócio (ex: "onde" encontrar o dado, "quando" usar um valor vs. outro).

### **B. Instruções Gerais (da tool):**

-   **Localização:** Final do modal da tool.
-   **Finalidade:** Fornecer um contexto geral para a execução da tool.
-   **Exemplo:** "Cliente é VIP, usar pipeline premium" ou "Criar apenas se valor > R$1000".

### **C. Instruções por Parâmetro:**

-   **Localização:** Dentro de cada parâmetro.
-   **Finalidade:** Instruir especificamente como preencher aquele campo.
-   **Exemplo:** "Extrair telefone da conversa" ou "Usar data mencionada pelo cliente".

---

## 🌐 **Regra 8: Metadata de Parâmetros (Schema Universal)**

Para aplicar essas regras a **qualquer integração**, cada parâmetro deve ter um schema JSON bem definido, conforme detalhado no glossário abaixo. Este schema é a "fonte da verdade" que dita todo o comportamento da UI e do sistema.

### **📖 Glossário de Campos do Schema (ATUALIZADO v3.8.0)**

> **⚠️ ATENÇÃO:** Este glossário reflete a simplificação radical do schema v3.8.0.
> - `input_type` é agora um campo único.
> - `config` é um objeto unificado.
> - A configuração de `dependency` foi drasticamente reduzida.

| Campo | Tipo | Descrição |
|---|---|---|
| **`name`** | string | Nome técnico do parâmetro (ex: `pipeline_id`). |
| **`display_name`** | string | Nome amigável mostrado na UI (ex: "Pipeline"). |
| **`help_text`** | string | Texto auxiliar na UI (ex: "Selecione o(s) pipeline(s)"). |
| **`type`** | string | Tipo de dado da API (`string`, `number`, `boolean`, `array`). |
| **`required`** | boolean | Se é obrigatório pela API externa. |
| **`visible`** | boolean | Se PODE aparecer na UI (se `false`, nunca aparece). |
| **`show_by_default`** | boolean | Se aparece de cara ou no botão "Adicionar". |
| **`is_critical_field`**| boolean | Se precisa de normalização automática (ex: `email`, `phone`). |
| **`input_type`** | string | Tipo de preenchimento: `fixed`, `llm`, ou `dependency`. |
| **`config`** | object | Objeto de configuração que varia conforme `input_type`. |

*(Para detalhes completos sobre o objeto `config`, relacionamentos, validações e outros campos do schema, consulte a implementação de referência).*

---

# ⚙️ **REGRAS DE SCHEMA/SISTEMA**

> **As regras abaixo (9-12) são para definição do SCHEMA da tool pelo desenvolvedor.**
> O usuário final NÃO configura essas regras - elas são aplicadas automaticamente.

---

## 🎯 **Regra 9: Visibilidade Padrão dos Parâmetros**

Controla quais parâmetros aparecem imediatamente e quais ficam em um menu de "parâmetros opcionais".

-   **`visible: false`**: O parâmetro **nunca** aparece na UI. Usado para dependências de valor único resolvidas automaticamente.
-   **`visible: true` + `show_by_default: false`**: O parâmetro está disponível, mas fica oculto por padrão no menu "Adicionar parâmetros opcionais". Ideal para campos avançados ou raramente usados (`probability`, `currency`).
-   **`visible: true` + `show_by_default: true`**: O parâmetro aparece por padrão na UI. Ideal para campos importantes, mesmo que opcionais (`pipeline_id`, `stage_id`).
-   **`required: true`**: O parâmetro **sempre** aparece e é obrigatório, ignorando as flags acima.

---

## 🔴 **Regra 10: Campos Críticos**

A flag `is_critical_field: true` identifica campos que são chaves de busca ou identificadores importantes, acionando comportamentos especiais no sistema.

-   **TIPO 1: Chaves de Busca (com normalização):**
    -   **O que são:** Campos usados para encontrar registros existentes, onde o formato incorreto pode causar duplicatas (ex: `phone`, `email`).
    -   **Ação do Sistema:** Aplica regras de normalização automática (ex: formata `phone` para o padrão E.164, converte `email` para minúsculas).

-   **TIPO 2: Identificadores de Operações (sem normalização):**
    -   **O que são:** IDs usados para identificar um recurso em uma operação de atualização ou exclusão (ex: `deal_id` em `updateDeal`).
    -   **Ação do Sistema:** A UI exibe um aviso extra para o usuário confirmar que o critério de seleção está correto, prevenindo modificações no recurso errado.

---

## 📭 **Regra 11: Null e Empty Handling**

Define como o sistema deve tratar valores vazios ou nulos para parâmetros opcionais, garantindo que apenas dados válidos sejam enviados para a API.

-   **Strings:** `""` ou `"   "` são convertidos para `null` e, por padrão, o parâmetro não é enviado.
-   **Numbers:** `0` é considerado um valor válido e é enviado. `null` ou `undefined` não são enviados.
-   **Booleans:** `false` é um valor válido e é enviado. `null` ou `undefined` não são enviados.
-   **Arrays:** `[]` (array vazio) não é enviado.

O comportamento detalhado pode ser ajustado no schema através da propriedade `nullable_behavior`.

---

## 🏷️ **Regra 12: Categorias de Tools (Integração com Regra 1)**

Esta regra expande a Regra 1, detalhando as categorias que definem a visibilidade e o comportamento de uma tool.

| Categoria | Visível na Lista @? | Descrição |
|---|---|---|
| **Ação** | ✅ **SIM** | Executa operações de escrita (`create`, `update`, `delete`). O usuário deve configurar explicitamente. |
| **Híbrida** | ✅ **SIM** | Busca um recurso e o cria se não existir (`getOrCreate`). Requer configuração do usuário. |
| **Consulta Visível** | ✅ **SIM** (com 🔍) | Tool de leitura (`get`) que o usuário pode querer executar manualmente para consulta. |
| **Consulta de Dependência**| ❌ **NÃO** | Busca dados que são pré-requisito para outra tool. Invocada automaticamente. |
| **Suporte LLM** | ❌ **NÃO** | Fornece contexto para um parâmetro do tipo `llm`. Invocada automaticamente. |

---

## 📊 **Índice de Regras**

### 🎨 **Regras de UX/Configuração (Usuário)**

1.  **Visibilidade de Tools** - Quais tools aparecem na lista @.
2.  **Visibilidade do Parâmetro** - Quando mostrar ou ocultar um parâmetro.
3.  **Tipos de Preenchimento** - `fixed`, `llm` ou `dependency`.
4.  **Multi-Select** - Como lidar com múltiplos valores fixos.
5.  **Dependências e Relacionamentos** - Relações entre tools e parâmetros.
6.  **Obrigatoriedade** - Tratamento de campos `required`/`optional`.
7.  **Instruções** - Diferença entre instrução geral e por parâmetro.
8.  **Metadata Schema** - A estrutura JSON que define um parâmetro.

### ⚙️ **Regras de Schema/Sistema (Desenvolvedor)**

9.  **Visibilidade Padrão** - `show_by_default` para uma UI mais limpa.
10. **Campos Críticos** - Normalização automática com `is_critical_field`.
11. **Null e Empty Handling** - Como tratar valores nulos ou vazios.
12. **Categorias de Tools** - Classificação de tools para controlar o comportamento.

---

**Versão:** 3.8.0  
**Data:** 2025-11-13  
**Status:** Pronto para implementação
