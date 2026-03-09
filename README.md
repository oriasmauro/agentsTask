# El Secreto de sus Ojos — Arquitectura Agent2Agent (A2A)

Tarea: implementar un sistema multi-agente que analice incoherencias de una película y devuelva un JSON estructurado, aplicando todo lo visto en clase.

**Película elegida:** El Secreto de sus Ojos (2009) — Juan José Campanella

## Descripción

Cuatro agentes especializados analizan las incoherencias narrativas, morales y jurídicas de la película. Cada agente tiene su propio rol, sistema prompt y acceso a tools con datos concretos de la trama.

```
Sacerdote  →  Crítico  →  Meta-crítico  →  Juez (veredicto final)
```

| Agente | Rol | Output |
|---|---|---|
| Padre Ignacio | Perspectiva moral y teológica | Texto |
| Crítico | Análisis narrativo y jurídico | Texto |
| Meta-crítico | Detecta sesgos del Crítico | Texto |
| Juez | Consolida todo y emite veredicto | `VeredictoChatbot` (JSON) |

## Conceptos aplicados

### Fundamentos LangChain
- ✅ `SystemMessage`, `HumanMessage`, `AIMessage`, `ToolMessage` — historial de conversación multi-turno
- ✅ `PromptTemplate` — plantillas de prompts parametrizadas
- ✅ `FewShotPromptTemplate` + `FewShotChatMessagePromptTemplate` — few-shot con chain-of-thought (Thought/Response)
- ✅ `StrOutputParser`, `PydanticOutputParser`, `DatetimeOutputParser`, `BooleanOutputParser`, `OutputFixingParser` — parseo y autocorrección de salidas
- ✅ `with_structured_output` + schemas Pydantic (`BaseModel`, `Field`) — JSON tipado y validado
- ✅ **Chatbot con memoria propia** — clase con historial interno que acumula `SystemMessage` / `HumanMessage` / `AIMessage` entre turnos

### Agentes con Tools
- ✅ `@tool` + `bind_tools` — definición y conexión de herramientas al LLM
- ✅ **Proto-agente manual** — loop ReAct en Python puro: `llm_con_tools → ejecutar tool_calls → reinvocar` hasta respuesta final (sin LangGraph)
- ✅ `MessagesState` — estado tipado que acumula el historial de mensajes en el grafo
- ✅ `ToolNode` + `tools_condition` — nodo de ejecución de tools y edge condicional (continuar/terminar)
- ✅ `StateGraph` (LangGraph) — grafo con nodos, edges condicionales y visualización Mermaid
- ✅ `stream_mode="updates"` — ejecución paso a paso para ver qué nodo se activa en cada momento

### Memoria y Checkpointing
- ✅ `create_react_agent` — agente ReAct prebuilt de LangGraph con checkpointer integrado
- ✅ `InMemorySaver` — checkpointing para memoria persistente entre turnos
- ✅ **Threads aislados** — mismo agente, distintos `thread_id` → contextos independientes
- ✅ **State inspection & time travel** — `get_state()` y `get_state_history()` para inspeccionar snapshots del grafo

### MultiAgentes
- ✅ **Patrón Supervisor** — agente central (Juez) que delega a sub-agentes especializados envueltos como tools
- ✅ Sub-agentes: `agente_sacerdote`, `agente_critico`, `agente_metacritico` — cada uno con su propio prompt y tools

### Protocolo A2A (Agent2Agent)
- ✅ **A2A vs MCP** — A2A (Google/Linux Foundation) conecta agente↔agente en horizontal; MCP (Anthropic) conecta agente↔herramientas en vertical; no compiten, se complementan
- ✅ **Agent Cards** — tarjeta de presentación de cada agente (nombre, url, version, capabilities, skills) siguiendo el estándar Google/Linux Foundation 2025
- ✅ **`RegistroA2A`** — simula el descubrimiento de agentes vía `/.well-known/agent-card.json` con métodos `registrar`, `descubrir`, `listar`, `enviar_mensaje`
- ✅ **Protocolo `message/send`** — JSON-RPC 2.0 con ciclo de vida de Tasks (`submitted → working → completed`)
- ✅ **`dataclass` `Task`** + **`TypedDict` `EstadoAnalisisA2A`** — unidad de trabajo entre agentes y estado tipado del grafo A2A
- ✅ **Orquestador A2A con `StateGraph`** — grafo con nodos por agente: `descubrimiento → sacerdote → crítico → meta-crítico → veredicto`
- ✅ **Streaming paso a paso** — `grafo_a2a.stream()` con `stream_mode="updates"` para ver el log de comunicaciones A2A en tiempo real

## Instalación

```bash
cd secreto_de_sus_ojos

python -m venv .venv
source .venv/bin/activate

pip install langchain-classic langchain-openai langchain-core langgraph python-dotenv a2a-sdk uvicorn httpx

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
