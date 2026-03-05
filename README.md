# El Secreto de sus Ojos — Arquitectura Agent2Agent (A2A)

Tarea del curso: implementar un sistema multi-agente que analice incoherencias de una película y devuelva un JSON estructurado, aplicando todo lo visto en clase.

**Película elegida:** El Secreto de sus Ojos (2009) — Juan José Campanella

## Descripción

Cuatro agentes especializados analizan las incoherencias narrativas, morales y jurídicas de la película. Cada agente tiene su propio rol, sistema prompt y acceso a tools con datos concretos de la trama.

```
Sacerdote  →  Crítico  →  Meta-crítico  →  Juez (JSON final)
```

| Agente | Rol | Output |
|---|---|---|
| Padre Ignacio | Perspectiva moral y teológica | Texto |
| Crítico | Análisis narrativo y jurídico | Texto |
| Meta-crítico | Detecta sesgos del Crítico | Texto |
| Juez | Consolida todo | `VeredictoChatbot` (JSON) |

## Conceptos aplicados

- ✅ `SystemMessage`, `HumanMessage`, `AIMessage`, `ToolMessage` — historial de conversación multi-turno
- ✅ `PromptTemplate` — plantillas de prompts parametrizadas
- ✅ `FewShotPromptTemplate` + `FewShotChatMessagePromptTemplate` — few-shot con chain-of-thought (Thought/Response)
- ✅ `StrOutputParser`, `PydanticOutputParser`, `DatetimeOutputParser`, `BooleanOutputParser`, `OutputFixingParser` — parseo y autocorrección de salidas
- ✅ `with_structured_output` + schemas Pydantic (`BaseModel`, `Field`) — JSON tipado y validado
- ✅ `@tool` + `bind_tools` — definición y conexión de herramientas al LLM
- ✅ ReAct loop manual — ciclo `llm_con_tools → ejecutar tool_calls → reinvocar` hasta respuesta final
- ✅ `StateGraph` (LangGraph) — grafo con nodos, edges condicionales y visualización Mermaid
- `create_react_agent` — agente ReAct prebuilt de LangGraph
- `InMemorySaver` — checkpointing para memoria persistente entre turnos
- Patrón Supervisor (multi-agente) — agente central que delega a sub-agentes especializados como tools
- Protocolo A2A (Agent2Agent) — Agent Cards, descubrimiento de agentes y comunicación entre frameworks

## Instalación

```bash
# Clonar y entrar al proyecto
cd secreto_de_sus_ojos

# Crear entorno virtual
python -m venv .venv
source .venv/bin/activate

# Instalar dependencias
pip install langchain-classic langchain-openai langchain-core langgraph python-dotenv

# Configurar API key
cp .env.example .env
# Editar .env y pegar tu OPENAI_API_KEY
```

## Uso

Abrir `notebook.ipynb` en Jupyter o VS Code y ejecutar las celdas en orden.

## Estructura

```
.
├── notebook.ipynb   # Notebook principal con los 4 agentes
├── .env             # API key (no commitear)
├── .env.example     # Plantilla de variables de entorno
└── .venv/           # Entorno virtual (no commitear)
```
