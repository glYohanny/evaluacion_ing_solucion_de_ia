# Asistente de IA para Empresa Trade SPA (Empresa Fictisia)

## 1. Nombre y Descripción de la Organización

### Identificación Organizacional
- **Nombre:** Empresa Trade SPA
- **Rubro:** Comercio de Trading Card Games (TCG)
- **Tamaño:** PyME (pequeña-mediana empresa)
- **Ubicación:** En línea - www.tspa.com
- **Contexto General:** Empresa especializada en la compra, venta e intermediación de cartas coleccionables de diferentes juegos (Pokémon, Magic, Yu-Gi-Oh, etc.). Además, organiza y dirige torneos TCG de forma regular, actuando como hub central para la comunidad gaming local. Operación enfocada en atender proveedores, vendedores individuales, jugadores y clientes finales de forma simultánea.

---

## 2. Identificación y Descripción del Problema/Desafío

### Problema Principal
Empresa Trade SPA enfrenta un volumen creciente de comunicaciones que demandan gestión manual:
- **Gestión de emails** de proveedores con múltiples consultas sobre inventario, precios y disponibilidad
- **Triaje de solicitudes** de vendedores que desean ofrecer cartas (requiere clasificación, evaluación de calidad)
- **Consultas de clientes** en la plataforma web que buscan artículos específicos y requieren orientación
- **Gestión de torneos:** Consultas sobre fechas, regulaciones, registro de participantes, resultados, premios

### Impacto Actual
- ⏱️ **Respuestas lentas:** Retrasos en atender proveedores y clientes
- 💼 **Carga operativa:** Personal dedicado exclusivamente a clasificar y responder emails
- 🔍 **Inconsistencia:** Diferentes calidades de respuesta según quién atiende
- 💰 **Pérdida de oportunidades:** Clientes no encontraban productos o no obtenían respuesta a tiempo

### Relevancia
En el comercio de TCG, la velocidad de respuesta es crítica. Los clientes buscan rápidamente cartas específicas, y los proveedores necesitan respuestas ágiles para mantener relaciones comerciales.

---

## 3. Objetivos de la Intervención

### Objetivos Medibles

1. **Reducir tiempo de respuesta**
   - Objetivo: De 24-48 horas a respuesta inmediata (< 5 minutos)
   - Métrica: % de consultas respondidas automáticamente en la primera interacción

2. **Aumentar clasificación automática de solicitudes**
   - Objetivo: 80% de emails clasificados correctamente sin intervención manual
   - Métrica: Precisión de clasificación (proveedores vs. vendedores vs. clientes)

3. **Mejorar experiencia del cliente**
   - Objetivo: Disponibilidad 24/7 de asistencia en búsqueda de productos
   - Métrica: Satisfacción del cliente (encuestas post-interacción)

4. **Reducir carga operativa**
   - Objetivo: Disminuir 60% del tiempo manual dedicado a emails
   - Métrica: Horas/mes ahorradas en gestión manual

5. **Optimizar gestión de torneos**
   - Objetivo: Responder automáticamente 70% de consultas sobre torneos (fechas, reglamentos, inscripción)
   - Métrica: % de preguntas sobre torneos resueltas sin intervención humana

---

## 4. Datos Disponibles o que se Pueden Obtener

### Datos Actuales
- 📧 **Base de emails históricos:** Conversaciones previas con proveedores, vendedores y clientes
- 📦 **Catálogo de productos:** Inventario de cartas (nombre, set, rareza, condición, precio)
- 👥 **Base de clientes:** Preferencias de búsqueda, historial de compras
- 💎 **Descripción de cartas:** Especificaciones técnicas, valoraciones, tendencias de mercado
- 📝 **FAQs:** Preguntas frecuentes sobre funcionamiento del sitio, envíos, devoluciones
- 🏆 **Base de torneos:** Calendario de eventos, reglas de juego, historial de participantes, rankings

### Datos que se Pueden Simular
- Ejemplos de emails de proveedores
- Solicitudes de vendedores con descartables de cartas
- Consultas de clientes buscando artículos específicos
- Consultas de jugadores sobre torneos, reglas y participación

---

## 5. Restricciones o Requerimientos Particulares

### Limitaciones Técnicas
- 🌐 Integración con sistema de emails existente
- 🔌 Compatibilidad con plataforma web www.tspa.com
- ⚡ Disponibilidad 24/7 sin interrupciones

### Requerimientos de Negocio
- ✅ Escalabilidad: Capaz de manejar múltiples conversaciones simultáneas
- 🔒 Privacidad: Proteger información de clientes y datos comerciales
- 📊 Trazabilidad: Registrar todas las interacciones para auditoría
- 💬 Tono: Profesional pero amable, adaptado a cada segmento (proveedores vs. clientes)

### Limitaciones Normativas
- Cumplimiento con normativas de protección de datos personales
- Claridad sobre cuándo interviene humano vs. IA
- Políticas de devolución y garantía según legislación local

---

## 6. Motivación para el Uso de Agentes de IA, LLMs y RAG

### ¿Por qué Agentes de IA?
Los **agentes de IA** tienen capacidad de:
- Tomar decisiones secuenciales (ej: clasificar → consultar base de datos → responder)
- Manejar múltiples herramientas (email, búsqueda en catálogo, historial de cliente)
- Escalar a humanos cuando sea necesario

### ¿Por qué LLMs?
- 🧠 **Comprensión del lenguaje natural:** Entienden variantes de preguntas (ej: "tengo un pikachu holo de 1ª edición")
- 📖 **Generación de respuestas:** Crean textos contextuales y personalizados
- 🔄 **Memoria conversacional:** Mantienen contexto de conversaciones largas

### ¿Por qué RAG (Retrieval-Augmented Generation)?
- 🎯 **Precisión en datos:** Evita alucinaciones consultando base de datos de cartas
- 📚 **Información actualizada:** Accede a inventario en tiempo real
- 💼 **Contexto empresarial:** Proporciona precios, disponibilidad y políticas específicas

### Arquitectura Propuesta
```
Email/Consulta en Web
        ↓
   [Agente IA]
   ├→ Clasificar tipo de consulta (RAG: políticas, torneos)
   ├→ Buscar en catálogo (RAG: base de productos)
   ├→ Consultar información de torneos (RAG: calendario, reglas)
   ├→ Consultar historial del cliente (memoria)
   └→ Generar respuesta con LLM
        ↓
   Respuesta automática o escalada a humano
```

---

## 7. Instrucciones de Ejecución

### Requisitos previos
- Python 3.11+
- Git
- Token de GitHub con acceso a GitHub Models

### 1. Clonar el repositorio
```bash
git clone <URL_REPOSITORIO>
cd evaluacion_ing_solucion_de_ia
```

### 2. Crear entorno virtual

**Windows (path corto recomendado para evitar error MAX_PATH):**
```powershell
python -m venv C:\ev_ia_venv
C:\ev_ia_venv\Scripts\activate
```

**Linux/Mac:**
```bash
python -m venv .venv
source .venv/bin/activate
```

### 3. Instalar dependencias
```bash
pip install -r requirements.txt
pip install gradio
```

### 4. Configurar variables de entorno

Crear un archivo `.env` en esta carpeta con el siguiente contenido:
```
GITHUB_TOKEN=tu_github_token_aqui
OPENAI_BASE_URL=https://models.inference.ai.azure.com
GITHUB_BASE_URL=https://models.inference.ai.azure.com
```

> El `GITHUB_TOKEN` se obtiene en https://github.com/settings/tokens (no requiere permisos especiales para GitHub Models).

### 5. Registrar kernel en Jupyter (solo si se usa el venv de path corto)
```powershell
C:\ev_ia_venv\Scripts\python -m ipykernel install --user --name ev_ia_venv --display-name "Python (ev_ia_venv)"
```

### 6. Ejecutar el notebook
```bash
jupyter notebook IA.ipynb
```

Seleccionar el kernel **"Python (ev_ia_venv)"** y ejecutar las celdas en orden:

| Sección | Celdas | Descripción |
|---|---|---|
| IL1.1 | 1-2 | Chatbot con memoria conversacional |
| IL1.2 | 3-4 | Interfaz Gradio + técnicas de prompting |
| IL1.3 | 5-8 | Sistema RAG con base de conocimiento TCG |
| IL1.4 | 9-12 | Métricas de evaluación (Faithfulness, Relevancy, Precision) |
| IE5-IE7 | 13-19 | Arquitectura, diagrama y justificación técnica |

### 7. Validar el sistema RAG
Las celdas de IL1.3 ejecutan 3 ejemplos de consultas TCG automáticamente.  
Las celdas de IL1.4 generan un reporte de evaluación con promedios de las 3 métricas.

### Solución de problemas comunes

| Error | Causa | Solución |
|---|---|---|
| `api_key must be set` | `.env` no encontrado o no cargado | Verificar que el archivo `.env` existe en la carpeta del notebook |
| `OSError: No such file or directory` | Windows MAX_PATH (260 chars) | Crear venv en `C:\ev_ia_venv\` como se indica |
| `ModuleNotFoundError: gradio` | gradio no incluido en requirements.txt | `pip install gradio` manualmente |
| Kernel no aparece en Jupyter | Kernel no registrado | Ejecutar paso 5 de estas instrucciones |

---

## 8. Estructura del Repositorio

```
evaluacion_ing_solucion_de_ia/
├── IA.ipynb              # Notebook principal con todo el código
├── informe_tecnico.md    # Informe técnico IE8/IE9 (convertir a PDF para entregar)
├── README.md             # Este archivo
├── requirements.txt      # Dependencias Python
└── .env                  # Variables de entorno (NO subir a GitHub)
```

---

## 9. Referencias y Anexos

### Tecnologías Utilizadas
- **LLM:** GPT-4o-mini (GitHub Models / Azure Inference)
- **Framework:** LangChain (gestión de agentes y memoria)
- **RAG:** Retrieval léxico sobre base de conocimiento TCG (12 documentos)
- **UI:** Gradio ChatInterface
- **Lenguaje:** Python 3.11
- **Evaluación:** Métricas RAGAS-inspired (Faithfulness, Answer Relevancy, Context Precision)
