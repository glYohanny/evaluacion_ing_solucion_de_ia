# Informe Técnico EP2 — Agente TCG Funcional
## ISY0101 Ingeniería de Soluciones con IA — DuocUC 2025
**Empresa caso:** Empresa Trade SPA | **Tecnologías:** Python · LangChain · GPT-4o-mini · Gradio

---

## 1. Diseño e Implementación del Agente (IE1, IE2)

### 1.1 Arquitectura del Agente

El agente TCG de Empresa Trade SPA se construye sobre **LangChain AgentExecutor** con `create_openai_tools_agent`, implementando el ciclo **ReAct** (Reason + Act) de forma automática. El agente recibe una consulta, razona sobre qué herramienta usar, la ejecuta, observa el resultado y genera la respuesta final.

### 1.2 Herramientas Configuradas (IE1)

Se definieron **3 herramientas especializadas** usando el decorador `@tool` de LangChain:

| Herramienta | Función | Cuándo la usa el agente |
|---|---|---|
| `consultar_precio_carta` | Busca precio de una carta específica en la KB | Consultas directas de valor ("¿cuánto vale X?") |
| `buscar_informacion_tcg` | RAG léxico sobre 12 documentos TCG | Consultas generales de dominio ("¿cómo funciona el grading?") |
| `calcular_valor_coleccion` | Estima valor de múltiples cartas | Consultas de inventario ("tengo X, Y y Z, ¿cuánto vale?") |

El uso del decorador `@tool` permite que LangChain infiera el JSON Schema automáticamente desde el docstring y los type hints, eliminando definición manual de esquemas.

### 1.3 Framework Utilizado (IE2)

**LangChain** fue seleccionado como framework principal por:
- **Escalabilidad**: Abstracción `AgentExecutor` maneja el loop ReAct sin código manual
- **Compatibilidad**: Compatible con GitHub Models API (Azure Inference) sin modificaciones
- **Extensibilidad**: Las herramientas son intercambiables; la memoria es conectable via interfaz `BaseChatMessageHistory`

```python
agent = create_openai_tools_agent(llm, tools, agent_prompt)
executor = AgentExecutor(agent=agent, tools=tools, memory=memoria, verbose=True, max_iterations=5)
```

### 1.4 Diagrama de Orquestación (IE7)

```
╔══════════════════════════════════════════════════════════════════════╗
║          ORQUESTACIÓN DEL AGENTE TCG — Empresa Trade SPA            ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  ┌──────────┐  consulta  ┌──────────────────────────────────────┐   ║
║  │ USUARIO  │──────────▶│           AGENTEXECUTOR               │   ║
║  └──────────┘           │  ┌────────────────────────────────┐   │   ║
║       ▲                 │  │  create_openai_tools_agent      │   │   ║
║       │ respuesta       │  │  GPT-4o-mini  temperatura=0.3   │   │   ║
║       │                 │  └────────────────┬───────────────┘   │   ║
║       │                 │                   │ selecciona tool    │   ║
║       │                 │  ┌────────────────▼───────────────┐   │   ║
║       │                 │  │         TOOL SELECTOR          │   │   ║
║       │                 │  └────┬──────────┬────────────┬───┘   │   ║
║       │                 │       │          │            │        │   ║
║       │                 │  ┌────▼───┐ ┌───▼────┐ ┌────▼────┐   │   ║
║       │                 │  │ precio │ │  RAG   │ │  valor  │   │   ║
║       │                 │  │ carta  │ │  TCG   │ │ colec.  │   │   ║
║       │                 │  └────┬───┘ └───┬────┘ └────┬────┘   │   ║
║       │                 │       └──────────┴────────────┘        │   ║
║       │                 │                  │ resultado            │   ║
║       │                 │  ┌───────────────▼────────────────┐   │   ║
║       │                 │  │   KNOWLEDGE BASE TCG (12 docs) │   │   ║
║       │                 │  └────────────────────────────────┘   │   ║
║       │                 │  ┌────────────────────────────────┐   │   ║
║       │                 │  │  ConversationBufferWindowMemory│   │   ║
║       │                 │  │  k=5 | memoria de corto plazo  │   │   ║
║       │                 │  └────────────────────────────────┘   │   ║
║       │                 └──────────────────┬─────────────────────┘   ║
║       └──────────────────────────────────── ┘                        ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 2. Configuración de Memoria (IE3, IE4)

### 2.1 Memoria de Corto Plazo — ConversationBufferWindowMemory (IE3)

Se implementa `ConversationBufferWindowMemory(k=5)` que mantiene las últimas 5 interacciones usuario-agente en memoria activa. La ventana deslizante garantiza:
- **Continuidad**: El agente recuerda el nombre del usuario, preferencias declaradas y consultas previas
- **Control de tokens**: La ventana k=5 impide que el historial crezca ilimitadamente
- **Coherencia**: Permite referencias anafóricas ("y si las mando a gradear, cuánto más valdría?")

**Evidencia de funcionamiento** (extracto del notebook):
```
[Turno 1] Usuario: Hola! Me llamo Carlos y colecciono cartas de Pokemon
[Turno 5] Usuario: Gracias! Recuerdas como me llamo?
[Turno 5] Agente: ¡Claro, Carlos! ...
```

### 2.2 Recuperación de Contexto Semántico — RAG como herramienta (IE4)

La herramienta `buscar_informacion_tcg` actúa como capa de **recuperación semántica de largo plazo**: el agente accede a la knowledge base TCG de EP1 (12 documentos especializados) cuando necesita contexto de dominio que no está en la memoria conversacional.

**Flujo de recuperación semántica:**
```
Consulta → análisis de keywords → scoring léxico → top-3 docs → contexto enriquecido
```

La arquitectura permite migrar este componente a embeddings vectoriales (FAISS/ChromaDB) para recuperación verdaderamente semántica en producción.

---

## 3. Planificación y Toma de Decisiones (IE5, IE6)

### 3.1 Esquema de Planificación (IE5)

El agente implementa una arquitectura **Plan-and-Execute** con 4 fases secuenciadas:

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  1. CLASIF. │───▶│ 2. RECUPERAR│───▶│ 3. SINTETIZ.│───▶│ 4. RESPONDER│
│  tipo de    │    │ herramienta │    │ contexto +  │    │ con recomen-│
│  consulta   │    │ apropiada   │    │ razonamiento│    │ dación      │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

El planificador LLM genera explícitamente los pasos antes de ejecutarlos, mejorando la coherencia en tareas multi-etapa (ej: valorar colección + recomendar canal de venta + estimar ganancia).

### 3.2 Toma de Decisiones Adaptativas (IE6)

Se demostraron 4 escenarios con condiciones distintas:

| Escenario | Input | Decisión del agente | Herramienta |
|---|---|---|---|
| **Consulta directa** | "¿Cuánto vale un Black Lotus?" | Busca precio específico | `consultar_precio_carta` |
| **Fuera de dominio** | "¿Cuál es el precio del dólar?" | Declina amablemente | Ninguna (respuesta directa) |
| **Consulta compleja** | "Tengo Charizard, Blue-Eyes y Black Lotus" | Multi-herramienta | `calcular_valor_coleccion` + `buscar_informacion_tcg` |
| **Límite de conocimiento** | "¿Cuánto vale una carta de Riftbound?" | Indica incertidumbre + contexto disponible | `consultar_precio_carta` |

El agente ajusta su comportamiento según las condiciones del entorno: usa herramientas cuando hay información disponible, responde directamente cuando no es necesario, y declina apropiadamente cuando la consulta está fuera del dominio TCG.

---

## 4. Justificación de Componentes (IE8)

| Decisión | Alternativa considerada | Justificación técnica |
|---|---|---|
| `create_openai_tools_agent` vs ReAct manual | Implementación ReAct desde cero (EP1/IL2.1) | El framework maneja parsing, retry y tool calling de forma robusta; menor superficie de error en producción |
| `ConversationBufferWindowMemory(k=5)` vs Buffer completo | `ConversationBufferMemory` sin límite | La ventana k=5 controla el uso de tokens; conversaciones TCG típicas no requieren más de 5 turnos de contexto |
| Temperatura 0.3 vs 0.7 | Default LangChain (0.7) | Los precios de cartas son datos factuales; temperatura baja reduce alucinaciones en información numérica |
| RAG como herramienta vs RAG always-on | Inyectar contexto en cada prompt | Como herramienta, el agente decide cuándo recuperar; evita contaminar el prompt con contexto irrelevante |
| LangChain vs CrewAI | Multi-agente con CrewAI | Para agente único con múltiples herramientas, LangChain es más directo; CrewAI agrega complejidad innecesaria en esta fase |

---

## 5. Flujos de Trabajo (IE9)

### Flujo 1: Consulta de precio simple
```
Usuario: "¿Cuánto vale un Charizard de 1999?"
    ↓
AgentExecutor: Razón → "Necesito precio específico → usar consultar_precio_carta"
    ↓
consultar_precio_carta("Charizard 1999")
    ↓
Knowledge Base → ["Charizard Holo Base Set 1999: USD 500-2000. PSA 10: USD 300000+..."]
    ↓
GPT-4o-mini: Sintetiza respuesta con rango de precios y consejo práctico
    ↓
Usuario: "Un Charizard Holo de 1999 vale entre USD 500 y USD 2000 sin grading..."
```

### Flujo 2: Conversación prolongada con memoria
```
Turno 1: "Me llamo Carlos, tengo Pokémon"         → memoria registra "Carlos, Pokémon"
Turno 2: "¿Cuánto vale mi Charizard?"             → recupera precio + contexto memoria
Turno 3: "¿Y si lo grado con PSA?"                → contexto previo + nueva herramienta
Turno 4: "¿Dónde vendo en Chile?"                 → mantiene contexto Charizard
Turno 5: "¿Recuerdas mi nombre?"                  → responde "Carlos" desde memoria k=5
```

### Flujo 3: Plan-and-Execute para tarea compleja
```
Objetivo: "Quiero invertir en cartas Yu-Gi-Oh antiguas para vender en Chile"
    ↓
[Planner] Genera plan:
  Paso 1: Identificar cartas Yu-Gi-Oh valiosas [consultar_precio_carta]
  Paso 2: Evaluar opción de grading [buscar_informacion_tcg]
  Paso 3: Identificar canales de venta en Chile [buscar_informacion_tcg]
  Paso 4: Sintetizar recomendación de inversión
    ↓
[Executor] Ejecuta cada paso con herramientas correspondientes
    ↓
Respuesta integral con análisis, datos y recomendación
```

---

## 6. Conclusiones Técnicas (IE10)

La implementación del Agente TCG EP2 demuestra una evolución significativa respecto al chatbot RAG de EP1: mientras EP1 implementa recuperación y generación como pipeline lineal, EP2 introduce **agencia real** —el sistema decide autónomamente qué información recuperar, cuándo razonar sin herramientas y cómo adaptar su comportamiento según el contexto.

El patrón Plan-and-Execute resuelve una limitación fundamental de los agentes puramente reactivos: la tendencia a tomar acciones localmente óptimas pero globalmente subóptimas. Al generar un plan explícito primero, el agente mantiene coherencia en tareas multi-etapa como valoración de colecciones completas o asesoría de inversión.

La arquitectura modular (herramientas intercambiables, memoria conectable, LLM reemplazable) permite a Empresa Trade SPA escalar la solución incrementalmente: añadir nuevas herramientas (API de precios en tiempo real, integración con MercadoLibre), migrar la memoria a Redis para persistencia entre sesiones, y eventualmente implementar orquestación multi-agente con CrewAI para flujos más complejos.

---

## 7. Referencias (APA)

- Chase, H. (2022). *LangChain*. GitHub. https://github.com/langchain-ai/langchain
- Yao, S., et al. (2022). *ReAct: Synergizing Reasoning and Acting in Language Models*. arXiv. https://arxiv.org/abs/2210.03629
- OpenAI. (2024). *Function Calling Guide*. https://platform.openai.com/docs/guides/function-calling
- Langchain. (2024). *AgentExecutor — LangChain Documentation*. https://python.langchain.com/docs/modules/agents/
- Wang, L., et al. (2023). *A Survey on Large Language Model based Autonomous Agents*. arXiv. https://arxiv.org/abs/2308.11432
- Lewis, P., et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*. NeurIPS. https://arxiv.org/abs/2005.11401

---

## 8. Declaración de Uso de IA

Se utilizó Claude Code (Anthropic) como asistente de programación para estructurar el código del agente y generar el diagrama ASCII de orquestación. Todas las decisiones técnicas, justificaciones y análisis fueron elaborados por el equipo. Las conclusiones son redactadas sin apoyo de IA según normativa DuocUC: https://bibliotecas.duoc.cl/ia
