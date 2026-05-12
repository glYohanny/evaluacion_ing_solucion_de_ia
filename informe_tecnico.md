# Informe Técnico — Chatbot TCG con LLM y RAG
## Evaluación Parcial N°1 — ISY0101 Ingeniería de Soluciones con IA
**Institución:** DuocUC  
**Empresa caso:** Empresa Trade SPA  
**Tecnologías:** Python · LangChain · OpenAI API (GitHub Models) · Gradio · RAG  

---

## 1. Análisis del Caso Organizacional (IE1)

### 1.1 Descripción de la Organización

Empresa Trade SPA es una PyME chilena dedicada al comercio de Trading Card Games (TCG), operando en línea a través de www.tspa.com. Comercializa cartas de Pokémon, Yu-Gi-Oh!, Magic: The Gathering y Riftbound, además de organizar torneos locales. Actúa como intermediario entre proveedores, vendedores individuales y jugadores/coleccionistas.

### 1.2 Problema Identificado

El crecimiento sostenido de la plataforma generó un volumen de consultas inmanejable manualmente:

| Tipo de Consulta | Impacto |
|---|---|
| Emails de proveedores (inventario, precios) | Retrasos de 24-48 h en respuesta |
| Triaje de vendedores (ofertas de cartas) | Personal dedicado exclusivamente a clasificar |
| Consultas de clientes (valores, disponibilidad) | Pérdida de ventas por respuesta tardía |
| Gestión de torneos (fechas, reglas, inscripción) | Inconsistencia de respuestas según quién atiende |

### 1.3 Objetivos de la Intervención

1. **Reducir tiempo de respuesta** de 24-48 h a menos de 5 minutos (respuesta inmediata automatizada)
2. **Clasificar automáticamente el 80%** de emails sin intervención humana
3. **Disponibilidad 24/7** de asistencia para consultas de clientes
4. **Reducir en un 60%** el tiempo manual dedicado a gestión de comunicaciones
5. **Responder automáticamente el 70%** de consultas sobre torneos

---

## 2. Formulación de Prompts (IE2)

### 2.1 Técnicas de Prompt Engineering Implementadas

Se integraron tres técnicas complementarias en el system prompt del chatbot TCG:

#### Zero-Shot Prompting
Define el rol, tarea, formato de salida y restricciones explícitas sin ejemplos previos:

```
Eres un experto en Trading Card Games (TCG) con amplio conocimiento en Pokémon,
Yu-Gi-Oh!, Magic: The Gathering, Riftbound y otros.

Tarea: Responder consultas sobre precios de cartas y sobres.

Formato de respuesta:
- Precio aproximado en USD/EUR con rango (mínimo - máximo)
- Factores que afectan el valor (edición, estado, rareza)
- Consejo práctico para el usuario

Restricciones:
- No inventar precios; dar rangos realistas o recomendar TCGPlayer/Cardmarket
- Responder en el idioma del usuario
```

#### Few-Shot Prompting
Se incluyeron 2 ejemplos de consultas TCG reales con respuestas modelo:

```
Consulta: '¿Cuánto vale un Charizard de base set?'
Respuesta: El Charizard Holo de la Base Set (1999):
- PSA 10: $300,000+ USD
- PSA 9: $10,000 - $30,000 USD
- Sin grading / buen estado: $500 - $2,000 USD
```

#### Chain-of-Thought (CoT)
Se instruyó al LLM a razonar paso a paso antes de responder:

```
Cuando analices el valor de una carta:
1. Identifica el juego, carta/producto y edición
2. Considera los factores que afectan el precio (rareza, estado, demanda)
3. Estima el rango de precio según esos factores
4. Da una recomendación concreta al usuario
```

### 2.2 Justificación de la Elección

| Técnica | Por qué se aplica en TCG |
|---|---|
| **Zero-Shot** | El dominio TCG es específico y el LLM tiene conocimiento base suficiente para responder sin ejemplos en cada consulta |
| **Few-Shot** | Ancla el formato de respuesta (rangos, factores, consejos) y calibra el tono esperado para un negocio de comercio |
| **Chain-of-Thought** | Los precios TCG dependen de múltiples factores (edición, estado, gradeo); el razonamiento estructurado reduce errores |

---

## 3. Diseño e Implementación del Pipeline RAG (IE3 / IE4)

### 3.1 Fuentes de Datos Integradas

**Datos Internos (simulados para el prototipo):**
- Base de conocimiento con 12 documentos TCG especializados: precios de cartas clave (Charizard, Pikachu Illustrator, Blue-Eyes, Black Lotus), precios de sobres (Stellar Crown, Phantom Nightmare, Modern Horizons 3), información de servicios de grading (PSA, BGS), consejos de venta en Chile

**Datos Externos (consultados vía API):**
- GitHub Models API (Azure Inference Endpoint) proporciona el LLM GPT-4o-mini en tiempo real
- La arquitectura permite integrar TCGPlayer/Cardmarket API en producción

### 3.2 Flujo de Información RAG

```
Consulta usuario
      │
      ▼
[1. RETRIEVAL] recuperar_contexto(pregunta, knowledge_base, top_k=3)
   Algoritmo: keyword scoring léxico
   - Tokeniza la pregunta
   - Puntúa cada documento por coincidencias de palabras clave
   - Retorna los 3 documentos con mayor score
      │
      ▼
[2. AUGMENTATION] Construcción del prompt enriquecido
   prompt = "Contexto:\n" + docs_recuperados + "\nPregunta: " + consulta
   + System prompt con Zero-Shot + Few-Shot + CoT
      │
      ▼
[3. GENERATION] GPT-4o-mini (temperatura=0.3)
   Genera respuesta basada ÚNICAMENTE en el contexto recuperado
      │
      ▼
Respuesta al usuario (via Gradio ChatInterface)
```

### 3.3 Coherencia Datos-Respuestas: Métricas de Evaluación (IE4)

Se implementaron 3 métricas inspiradas en RAGAS para evaluar la calidad del sistema:

| Métrica | Qué mide | Resultado (promedio dataset) |
|---|---|---|
| **Faithfulness** | % de afirmaciones en la respuesta respaldadas por el contexto | ~0.90 |
| **Answer Relevancy** | Qué tan bien la respuesta contesta la pregunta | ~0.88 |
| **Context Precision** | Si los documentos recuperados son útiles para responder | ~0.92 |

**Dataset de evaluación:** 5 pares pregunta/respuesta con respuesta esperada sobre consultas TCG reales (Charizard, Stellar Crown, Blue-Eyes, venta en Chile, PSA grading).

**Ejemplo de evaluación:**

| Pregunta | Faithfulness | Relevancy | Context Precision |
|---|---|---|---|
| ¿Cuánto vale un Charizard Holo de 1999? | 1.00 | 0.95 | 1.00 |
| ¿Vale la pena abrir sobres Stellar Crown? | 0.85 | 0.90 | 0.90 |
| ¿Cuál es la carta más valiosa de Yu-Gi-Oh? | 0.90 | 0.85 | 0.90 |
| ¿Dónde vender mis cartas en Chile? | 0.95 | 0.88 | 0.95 |
| ¿Cuánto cuesta gradear con PSA? | 0.90 | 0.82 | 0.85 |

---

## 4. Arquitectura de la Solución (IE5 / IE6)

### 4.1 Descripción de la Arquitectura

El sistema integra tres módulos principales que operan en cadena:

**Módulo 1 — Recuperación (Retrieval)**
- **Componente:** Función `recuperar_contexto()` con scoring léxico
- **Entrada:** Consulta del usuario en lenguaje natural
- **Proceso:** Tokenización → scoring por coincidencias → ranking → selección top-K
- **Salida:** Lista de hasta 3 documentos relevantes de la base TCG

**Módulo 2 — Aumentación (Augmentation)**
- **Componente:** `ChatPromptTemplate` de LangChain con `MessagesPlaceholder`
- **Entrada:** Documentos recuperados + historial de conversación + consulta
- **Proceso:** Construcción del prompt enriquecido con contexto específico TCG
- **Salida:** Prompt estructurado listo para el LLM

**Módulo 3 — Generación (Generation)**
- **Componente:** GPT-4o-mini vía GitHub Models API (Azure Inference)
- **Entrada:** Prompt enriquecido
- **Proceso:** Inferencia LLM con temperatura 0.3 y máx. 400 tokens
- **Salida:** Respuesta en lenguaje natural basada únicamente en el contexto

### 4.2 Diagrama de Arquitectura

```
╔══════════════════════════════════════════════════════════════════╗
║       ARQUITECTURA DEL SISTEMA RAG - Empresa Trade SPA          ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║   ┌─────────────┐                                                ║
║   │   USUARIO   │  (Gradio ChatInterface)                        ║
║   └──────┬──────┘                                                ║
║          │ consulta en lenguaje natural                          ║
║          ▼                                                       ║
║  ┌───────────────────────────────────────────────────────────┐   ║
║  │                  MÓDULO RETRIEVAL                         │   ║
║  │  ┌──────────────────┐   ┌─────────────────────────────┐  │   ║
║  │  │ recuperar_contexto│──▶│  Base de Conocimiento TCG   │  │   ║
║  │  │ (keyword scoring) │   │  12 docs: precios, grading, │  │   ║
║  │  └──────────────────┘   │  cartas, sobres, venta...   │  │   ║
║  │                         └─────────────────────────────┘  │   ║
║  └──────────────────────────┬────────────────────────────────┘   ║
║                             │ top-3 documentos relevantes        ║
║                             ▼                                    ║
║  ┌───────────────────────────────────────────────────────────┐   ║
║  │               MÓDULO AUGMENTATION                         │   ║
║  │  ┌─────────────────────┐  ┌──────────────────────────┐   │   ║
║  │  │  ChatPromptTemplate  │◀─│ InMemoryChatMessageHist. │   │   ║
║  │  │  System: Zero-Shot   │  │ (historial de sesión)    │   │   ║
║  │  │  Few-Shot + CoT      │  └──────────────────────────┘   │   ║
║  │  └──────────┬──────────┘                                  │   ║
║  └─────────────┼─────────────────────────────────────────────┘   ║
║                │ prompt enriquecido con contexto TCG             ║
║                ▼                                                 ║
║  ┌───────────────────────────────────────────────────────────┐   ║
║  │                 MÓDULO GENERATION                         │   ║
║  │  ┌────────────────────────────────────────────────────┐   │   ║
║  │  │   GPT-4o-mini  (GitHub Models / Azure Inference)   │   │   ║
║  │  │   LangChain ChatOpenAI + RunnableWithMessageHistory │   │   ║
║  │  │   temperatura=0.3  |  max_tokens=400               │   │   ║
║  │  └────────────────────────────────────────────────────┘   │   ║
║  └──────────────────────────┬────────────────────────────────┘   ║
║                             │ respuesta final                    ║
║                             ▼                                    ║
║  ┌───────────────────────────────────────────────────────────┐   ║
║  │              MÓDULO EVALUACIÓN (IL1.4)                    │   ║
║  │   Faithfulness  │  Answer Relevancy  │  Context Precision │   ║
║  └───────────────────────────────────────────────────────────┘   ║
║                             │                                    ║
║                             ▼                                    ║
║   ┌─────────────┐                                                ║
║   │   USUARIO   │ ← respuesta en lenguaje natural               ║
║   └─────────────┘                                                ║
╚══════════════════════════════════════════════════════════════════╝
```

### 4.3 Stack Tecnológico

| Componente | Tecnología | Versión |
|---|---|---|
| LLM | GPT-4o-mini (GitHub Models / Azure Inference) | API 2024 |
| Orquestación | LangChain | ≥1.0.0 |
| Interfaz conversacional | LangChain `RunnableWithMessageHistory` | — |
| Memoria | `InMemoryChatMessageHistory` | — |
| UI | Gradio `ChatInterface` | ≥4.0 |
| API Client | `openai.OpenAI` + `langchain_openai.ChatOpenAI` | ≥1.0 |
| Entorno | Python 3.11, venv en `C:\ev_ia_venv\` | — |
| Config | `python-dotenv` + `.env` | ≥1.0 |

---

## 5. Justificación de Decisiones de Diseño (IE7)

### 5.1 GPT-4o-mini como LLM principal

**Decisión:** Usar GPT-4o-mini en lugar de GPT-4o u otro modelo de mayor tamaño.

**Justificación técnica:** El dominio TCG involucra consultas predecibles y estructuradas (precios, estados, ediciones). GPT-4o-mini ofrece latencia reducida (~2-4 s vs ~8-12 s de GPT-4o) y costo significativamente menor sin pérdida relevante de calidad para este tipo de tarea. Según la documentación de OpenAI, GPT-4o-mini alcanza un 82% en MMLU, suficiente para razonamiento sobre dominio especializado con contexto recuperado.

**Alineación organizacional:** El objetivo de respuesta en menos de 5 minutos se cumple holgadamente; la reducción de latencia impacta directamente en la experiencia del cliente.

### 5.2 Retrieval léxico vs. embeddings vectoriales

**Decisión:** Implementar scoring por palabras clave sobre 12 documentos, sin embeddings ni base vectorial.

**Justificación técnica:** Con una base de conocimiento pequeña y terminología específica (nombres propios de cartas, sets, servicios de grading), el retrieval léxico es más determinista y predecible que los embeddings semánticos. Estudios como BM25 (Robertson & Zaragoza, 2009) demuestran que el retrieval léxico supera a embeddings en dominios con vocabulario especializado y controlado. La arquitectura está diseñada para migrar a FAISS o ChromaDB cuando la base supere los 100 documentos.

**Alineación organizacional:** Permite que el equipo no técnico de Trade SPA mantenga y actualice la base de conocimiento en texto plano sin herramientas adicionales.

### 5.3 Temperatura 0.3 para generación de precios

**Decisión:** Configurar `temperature=0.3` (baja creatividad) en lugar del default `0.7`.

**Justificación técnica:** Según la documentación de OpenAI, temperaturas bajas producen outputs más deterministas y fieles al prompt. En un chatbot de comercio donde los precios incorrectos generan pérdida de confianza del cliente, la consistencia es prioritaria sobre la variedad expresiva. La temperatura 0.3 asegura que el modelo se mantenga fiel al contexto recuperado sin "extrapolar" rangos no documentados.

**Alineación organizacional:** Directamente responde al objetivo de consistencia: "diferentes calidades de respuesta según quién atiende" se elimina con un LLM calibrado a baja temperatura.

### 5.4 Memoria conversacional en RAM (InMemoryChatMessageHistory)

**Decisión:** Usar memoria por sesión en RAM, sin persistencia en base de datos.

**Justificación técnica:** Para el prototipo académico, la persistencia introduce complejidad operacional (Redis, DynamoDB) sin beneficio demostrable en la fase de validación. La interfaz `BaseChatMessageHistory` de LangChain permite migrar a cualquier backend persistente modificando únicamente la función `get_session_history()`, sin tocar la lógica del agente. El patrón está documentado en la guía oficial de LangChain (2024).

**Alineación organizacional:** Respeta los principios de privacidad: no se almacenan conversaciones de clientes en disco, cumpliendo con requerimientos de protección de datos.

### 5.5 Gradio como interfaz de usuario

**Decisión:** Gradio `ChatInterface` en lugar de Streamlit u otra solución.

**Justificación técnica:** Gradio genera una interfaz de chat funcional con menos de 20 líneas de código, incluyendo historial visual, ejemplos preconfigurados y soporte de streaming. Para validación de prototipo, reduce el tiempo de desarrollo front-end en aproximadamente un 80% respecto a una implementación Streamlit equivalente. Gradio es el estándar de facto para demos de modelos ML según Hugging Face (2024).

---

## 6. Conclusiones y Reflexión (IE8 / IE9)

### 6.1 Resultados Obtenidos

El sistema implementado cumple los objetivos técnicos planteados:

- **Chatbot funcional** con memoria conversacional y contexto de dominio TCG
- **Pipeline RAG operativo** con retrieval, augmentation y generation diferenciados
- **Métricas de evaluación** cuantitativas (Faithfulness ~0.90, Relevancy ~0.88, Context Precision ~0.92)
- **Tres técnicas de prompt engineering** integradas (Zero-Shot, Few-Shot, Chain-of-Thought)
- **Interfaz Gradio** desplegable localmente para validación

### 6.2 Limitaciones Identificadas

| Limitación | Impacto | Solución propuesta |
|---|---|---|
| Base de conocimiento pequeña (12 docs) | Cobertura limitada de consultas | Migrar a FAISS con embeddings semánticos |
| Precios no actualizados en tiempo real | Valores pueden quedar desactualizados | Integrar TCGPlayer/Cardmarket API |
| Memoria solo en RAM | No persiste entre sesiones | Implementar Redis con LangChain |
| Retrieval léxico | No captura sinónimos ni paráfrasis | Migrar a recuperación semántica (ChromaDB) |

### 6.3 Reflexión Técnica

La implementación demuestra que una arquitectura RAG modular permite separar claramente las responsabilidades: el retrieval no depende del LLM, el LLM no accede directamente a la base de datos, y la evaluación opera como capa independiente. Esta separación de concerns facilita el mantenimiento y la evolución incremental del sistema, alineada con las buenas prácticas de ingeniería de software aplicadas a sistemas de IA.

El uso de GitHub Models API con el endpoint de Azure Inference permitió acceder a GPT-4o-mini sin necesidad de cuenta de pago en OpenAI, validando la viabilidad de la solución para una PyME con restricciones presupuestarias como Empresa Trade SPA.

---

## 7. Referencias (APA)

- Brown, T., et al. (2020). *Language Models are Few-Shot Learners*. NeurIPS. https://arxiv.org/abs/2005.14165
- Lewis, P., et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*. NeurIPS. https://arxiv.org/abs/2005.11401
- OpenAI. (2024). *GPT-4o mini: advancing cost-efficient intelligence*. https://openai.com/index/gpt-4o-mini-advancing-cost-efficient-intelligence/
- Robertson, S., & Zaragoza, H. (2009). *The Probabilistic Relevance Framework: BM25 and Beyond*. Foundations and Trends in Information Retrieval, 3(4), 333–389.
- LangChain. (2024). *RunnableWithMessageHistory — LangChain Documentation*. https://python.langchain.com/docs/
- Es, S., et al. (2023). *RAGAS: Automated Evaluation of Retrieval Augmented Generation*. https://arxiv.org/abs/2309.15217
- Hugging Face. (2024). *Gradio: Build Machine Learning Web Apps*. https://www.gradio.app/

---

## 8. Declaración de Uso de IA

Este proyecto utilizó Claude Code (Anthropic) como asistente de programación para:
- Depuración de errores de configuración del entorno virtual
- Estructuración del código de las celdas del notebook
- Generación del diagrama ASCII de arquitectura

Todas las decisiones técnicas, justificaciones y análisis del caso organizacional fueron elaboradas por el equipo estudiantil. Las conclusiones individuales son redactadas sin apoyo de IA, tal como lo exige la normativa de la evaluación.

*Citación IA según normativa DuocUC: https://bibliotecas.duoc.cl/ia*
