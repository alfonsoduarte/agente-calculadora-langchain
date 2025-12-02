# Arquitectura del Agente Calculadora + Búsqueda

## Índice
1. [Introducción](#introducción)
2. [¿Qué es un Agente?](#qué-es-un-agente)
3. [Arquitectura General](#arquitectura-general)
4. [Componentes del Sistema](#componentes-del-sistema)
5. [Flujo de Ejecución](#flujo-de-ejecución)
6. [Patrones de Diseño](#patrones-de-diseño)
7. [Diagrama de Secuencia](#diagrama-de-secuencia)

---

## Introducción

Este documento describe la arquitectura de un **Agente Inteligente** construido con LangChain que puede:
- Realizar cálculos matemáticos
- Buscar información en la web
- Consultar Wikipedia

El agente utiliza **DeepSeek** como modelo de lenguaje (LLM) a través de su API compatible con OpenAI.

---

## ¿Qué es un Agente?

### Definición Simple
Un **agente** es un sistema que usa un LLM como "cerebro" para decidir qué acciones tomar. A diferencia de una cadena simple (chain), el agente puede:

1. **Razonar** sobre el problema
2. **Elegir** qué herramienta usar
3. **Observar** el resultado
4. **Decidir** si necesita más acciones
5. **Responder** cuando tiene suficiente información

```
┌─────────────────────────────────────────────────────────────┐
│                         AGENTE                               │
│  ┌─────────┐    ┌──────────────┐    ┌─────────────────┐     │
│  │  Input  │───▶│   LLM        │───▶│  Herramientas   │     │
│  │ Usuario │    │  (DeepSeek)  │    │  - Calculadora  │     │
│  └─────────┘    │              │    │  - Búsqueda     │     │
│                 │  "Cerebro"   │◀───│  - Wikipedia    │     │
│                 └──────────────┘    └─────────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

### Analogía
Piensa en el agente como un **asistente humano**:
- Si le preguntas "¿Cuánto es 25 * 4?", usa la calculadora
- Si le preguntas "¿Quién ganó el último mundial?", busca en internet
- Si le preguntas "¿Quién fue Einstein?", consulta una enciclopedia

---

## Arquitectura General

```
agenteCalculadora/
│
├── docs/                          # Documentación
│   └── ARCHITECTURE.md            # Este archivo
│
├── src/                           # Código fuente
│   ├── __init__.py
│   ├── agents/                    # Lógica del agente
│   │   ├── __init__.py
│   │   └── calculator_agent.py    # Agente principal
│   │
│   ├── tools/                     # Herramientas disponibles
│   │   ├── __init__.py
│   │   ├── calculator.py          # Herramienta de cálculo
│   │   ├── web_search.py          # Búsqueda web
│   │   └── wikipedia_tool.py      # Consulta Wikipedia
│   │
│   ├── config/                    # Configuración
│   │   ├── __init__.py
│   │   └── settings.py            # Variables y constantes
│   │
│   └── utils/                     # Utilidades
│       ├── __init__.py
│       └── helpers.py             # Funciones auxiliares
│
├── tests/                         # Pruebas
├── main.py                        # Punto de entrada
├── requirements.txt               # Dependencias
├── .env.example                   # Ejemplo de variables de entorno
└── README.md                      # Documentación principal
```

---

## Componentes del Sistema

### 1. LLM (Large Language Model) - DeepSeek

```python
# El LLM es el "cerebro" del agente
# Usamos DeepSeek a través de la API compatible con OpenAI

from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="deepseek-chat",
    openai_api_base="https://api.deepseek.com",
    openai_api_key="tu-api-key"
)
```

**Responsabilidades:**
- Entender la pregunta del usuario
- Decidir qué herramienta usar
- Interpretar resultados
- Formular la respuesta final

### 2. Herramientas (Tools)

Las herramientas son **funciones** que el agente puede invocar:

| Herramienta | Propósito | Cuándo se usa |
|-------------|-----------|---------------|
| `Calculator` | Operaciones matemáticas | "¿Cuánto es 15% de 200?" |
| `WebSearch` | Información actual | "¿Cuál es el clima hoy?" |
| `Wikipedia` | Datos enciclopédicos | "¿Quién inventó la bombilla?" |

```python
# Ejemplo de definición de herramienta
@tool
def calculator(expression: str) -> str:
    """Evalúa expresiones matemáticas."""
    return str(eval(expression))
```

### 3. Agente (Agent)

El agente **orquesta** todo el sistema:

```python
from langchain.agents import create_react_agent

agent = create_react_agent(
    llm=llm,
    tools=[calculator, web_search, wikipedia],
    prompt=prompt_template
)
```

### 4. Executor

El executor **ejecuta** el bucle del agente:

```python
from langchain.agents import AgentExecutor

executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True  # Muestra el razonamiento
)
```

---

## Flujo de Ejecución

### Patrón ReAct (Reasoning + Acting)

El agente sigue el patrón **ReAct**:

```
1. THOUGHT (Pensamiento)
   "Necesito calcular 25 * 4 para responder"

2. ACTION (Acción)
   Usar herramienta: calculator
   Input: "25 * 4"

3. OBSERVATION (Observación)
   Resultado: 100

4. THOUGHT (Pensamiento)
   "Ya tengo la respuesta"

5. FINAL ANSWER (Respuesta Final)
   "El resultado de 25 * 4 es 100"
```

### Diagrama de Flujo

```
┌──────────────┐
│   Usuario    │
│  hace una    │
│  pregunta    │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│    Agente    │
│   analiza    │
│  la pregunta │
└──────┬───────┘
       │
       ▼
┌──────────────┐     ┌─────────────────┐
│  ¿Necesita   │────▶│   Ejecutar      │
│ herramienta? │ Sí  │  herramienta    │
└──────┬───────┘     └────────┬────────┘
       │ No                   │
       │                      ▼
       │             ┌─────────────────┐
       │             │   Observar      │
       │             │   resultado     │
       │             └────────┬────────┘
       │                      │
       │◀─────────────────────┘
       │
       ▼
┌──────────────┐
│  ¿Tiene      │
│  suficiente  │────▶ Volver a analizar
│  información?│ No
└──────┬───────┘
       │ Sí
       ▼
┌──────────────┐
│  Respuesta   │
│    final     │
└──────────────┘
```

---

## Patrones de Diseño

### 1. Strategy Pattern
Cada herramienta implementa una estrategia diferente para resolver problemas.

### 2. Chain of Responsibility
El agente pasa la consulta a través de diferentes herramientas hasta encontrar una respuesta.

### 3. Observer Pattern
El executor observa las acciones del agente y registra cada paso.

---

## Diagrama de Secuencia

```
Usuario          Agente          LLM(DeepSeek)      Herramientas
   │                │                  │                  │
   │  "¿Cuánto es   │                  │                  │
   │   15% de 80?"  │                  │                  │
   │───────────────▶│                  │                  │
   │                │  Analizar query  │                  │
   │                │─────────────────▶│                  │
   │                │                  │                  │
   │                │  Thought: Usar   │                  │
   │                │  calculadora     │                  │
   │                │◀─────────────────│                  │
   │                │                  │                  │
   │                │  Action: calc    │                  │
   │                │  Input: 80*0.15  │                  │
   │                │─────────────────────────────────────▶│
   │                │                  │                  │
   │                │                  │    Result: 12    │
   │                │◀─────────────────────────────────────│
   │                │                  │                  │
   │                │  Observation     │                  │
   │                │─────────────────▶│                  │
   │                │                  │                  │
   │                │  Final Answer    │                  │
   │                │◀─────────────────│                  │
   │                │                  │                  │
   │  "El 15% de    │                  │                  │
   │   80 es 12"    │                  │                  │
   │◀───────────────│                  │                  │
```

---

## Consideraciones de Diseño

### ¿Por qué DeepSeek?

1. **Costo-efectivo**: Más económico que GPT-4
2. **API compatible**: Usa el mismo formato que OpenAI
3. **Buen rendimiento**: Excelente en razonamiento

### ¿Por qué LangChain?

1. **Abstracción**: Simplifica la creación de agentes
2. **Modularidad**: Fácil agregar/quitar herramientas
3. **Ecosystem**: Gran cantidad de integraciones

### Mejores Prácticas Implementadas

- **Separación de responsabilidades**: Cada herramienta hace una cosa
- **Configuración centralizada**: Todo en `config/settings.py`
- **Logging**: Para debugging y monitoreo
- **Manejo de errores**: Graceful degradation

---

## Próximos Pasos

1. **Agregar más herramientas**: API del clima, traductor, etc.
2. **Memoria**: Recordar conversaciones anteriores
3. **Multi-agente**: Varios agentes especializados trabajando juntos
4. **Streaming**: Respuestas en tiempo real

---

*Documento creado como guía didáctica para entender la arquitectura de agentes con LangChain.*
