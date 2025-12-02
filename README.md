# Agente Calculadora + Búsqueda con LangChain y DeepSeek

Un agente inteligente que decide automáticamente qué herramienta usar para responder tus preguntas.

## Tabla de Contenidos

- [Descripción](#descripción)
- [Características](#características)
- [Requisitos](#requisitos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Uso](#uso)
- [Cómo Funciona](#cómo-funciona)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Ejemplos](#ejemplos)
- [Troubleshooting](#troubleshooting)

---

## Descripción

Este proyecto implementa un **agente inteligente** usando LangChain con DeepSeek como LLM. El agente puede:

- Realizar cálculos matemáticos
- Buscar información actualizada en la web
- Consultar datos enciclopédicos en Wikipedia

**Lo interesante**: El agente **decide por sí mismo** qué herramienta usar basándose en tu pregunta.

---

## Características

| Herramienta | Descripción | Ejemplo de uso |
|-------------|-------------|----------------|
| Calculadora | Operaciones matemáticas complejas | "¿Cuánto es 15% de 1250?" |
| Búsqueda Web | Información actual de internet | "¿Cuál es la cotización del dólar hoy?" |
| Wikipedia | Datos enciclopédicos | "¿Quién fue Marie Curie?" |

---

## Requisitos

- Python 3.9+
- Una API key de DeepSeek ([obtener aquí](https://platform.deepseek.com/))
- (Opcional) API key de SerpAPI para búsquedas web

---

## Instalación

### 1. Clonar o crear el proyecto

```bash
cd agenteCalculadora
```

### 2. Crear entorno virtual

```bash
python -m venv venv

# En macOS/Linux:
source venv/bin/activate

# En Windows:
venv\Scripts\activate
```

### 3. Instalar dependencias

```bash
pip install -r requirements.txt
```

### 4. Configurar variables de entorno

```bash
# Copiar el archivo de ejemplo
cp .env.example .env

# Editar con tus API keys
nano .env  # o usa tu editor preferido
```

---

## Configuración

### Archivo `.env`

```env
# API de DeepSeek (REQUERIDO)
DEEPSEEK_API_KEY=tu-api-key-de-deepseek

# API de SerpAPI para búsquedas web (OPCIONAL)
SERPAPI_API_KEY=tu-api-key-de-serpapi

# Configuración del agente
AGENT_VERBOSE=true
AGENT_MAX_ITERATIONS=10
```

### ¿Cómo obtener las API keys?

#### DeepSeek
1. Ve a [platform.deepseek.com](https://platform.deepseek.com/)
2. Crea una cuenta
3. Ve a "API Keys" y genera una nueva

#### SerpAPI (opcional, para búsquedas)
1. Ve a [serpapi.com](https://serpapi.com/)
2. Crea una cuenta gratuita (100 búsquedas/mes gratis)
3. Copia tu API key del dashboard

---

## Uso

### Modo Interactivo

```bash
python main.py
```

Esto abre una sesión interactiva donde puedes hacer preguntas:

```
🤖 Agente Calculadora + Búsqueda
================================
Escribe 'salir' para terminar

Tu pregunta: ¿Cuánto es 25 multiplicado por 16?

🔍 El agente está pensando...

💭 Thought: Necesito usar la calculadora para esta operación
🔧 Action: calculator
📥 Input: 25 * 16
📤 Output: 400

✅ Respuesta: El resultado de 25 multiplicado por 16 es 400.
```

### Uso Programático

```python
from src.agents.calculator_agent import CalculatorSearchAgent

# Crear el agente
agent = CalculatorSearchAgent()

# Hacer una pregunta
response = agent.run("¿Cuál es la raíz cuadrada de 144?")
print(response)
```

---

## Cómo Funciona

### El Patrón ReAct

El agente usa el patrón **ReAct** (Reasoning + Acting):

```
1. PENSAMIENTO → El agente analiza qué necesita hacer
2. ACCIÓN      → Elige y ejecuta una herramienta
3. OBSERVACIÓN → Ve el resultado de la herramienta
4. REPETIR     → Hasta tener suficiente información
5. RESPUESTA   → Genera la respuesta final
```

### Ejemplo de Razonamiento

**Pregunta**: "¿Cuántos años tiene el presidente de Francia?"

```
💭 Thought: Necesito buscar quién es el presidente de Francia
           y luego calcular su edad.

🔧 Action: web_search
📥 Input: "presidente de Francia 2024"
📤 Output: Emmanuel Macron es el presidente de Francia...

💭 Thought: Emmanuel Macron nació el 21 de diciembre de 1977.
           Necesito calcular su edad actual.

🔧 Action: calculator
📥 Input: 2024 - 1977
📤 Output: 47

💭 Thought: Ya tengo toda la información necesaria.

✅ Final Answer: Emmanuel Macron, el presidente de Francia,
                tiene 47 años (nacido en 1977).
```

---

## Estructura del Proyecto

```
agenteCalculadora/
│
├── docs/
│   └── ARCHITECTURE.md     # Documentación de arquitectura
│
├── src/
│   ├── agents/
│   │   └── calculator_agent.py   # Agente principal
│   │
│   ├── tools/
│   │   ├── calculator.py         # Herramienta calculadora
│   │   ├── web_search.py         # Herramienta de búsqueda
│   │   └── wikipedia_tool.py     # Herramienta Wikipedia
│   │
│   ├── config/
│   │   └── settings.py           # Configuración centralizada
│   │
│   └── utils/
│       └── helpers.py            # Funciones auxiliares
│
├── tests/                        # Tests unitarios
├── main.py                       # Punto de entrada
├── requirements.txt              # Dependencias
├── .env.example                  # Ejemplo de configuración
└── README.md                     # Este archivo
```

---

## Ejemplos

### Cálculos Matemáticos

```
❓ ¿Cuánto es el 15% de 1,500?
✅ El 15% de 1,500 es 225.

❓ Si tengo 3 pizzas de 8 porciones cada una, ¿cuántas porciones tengo?
✅ Tienes 24 porciones en total (3 × 8 = 24).

❓ ¿Cuál es la raíz cuadrada de 256?
✅ La raíz cuadrada de 256 es 16.
```

### Búsquedas de Información

```
❓ ¿Quién ganó el último mundial de fútbol?
✅ Argentina ganó el Mundial de Qatar 2022...

❓ ¿Cuál es la capital de Australia?
✅ La capital de Australia es Canberra...
```

### Consultas Combinadas

```
❓ ¿Cuántos años han pasado desde que se fundó Apple?
✅ Apple fue fundada el 1 de abril de 1976.
   Han pasado 48 años desde su fundación (2024 - 1976 = 48).
```

---

## Troubleshooting

### Error: "API key not found"
```bash
# Asegúrate de que el archivo .env existe y tiene las keys
cat .env
```

### Error: "Module not found"
```bash
# Reinstalar dependencias
pip install -r requirements.txt
```

### Error: "Rate limit exceeded"
- Espera unos minutos antes de hacer más consultas
- DeepSeek tiene límites según tu plan

### El agente no usa la herramienta correcta
- Las herramientas tienen descripciones que guían al LLM
- Puedes ajustar las descripciones en `src/tools/`

---

## Contribuir

1. Fork el repositorio
2. Crea una rama (`git checkout -b feature/nueva-herramienta`)
3. Commit tus cambios (`git commit -m 'Agrega nueva herramienta'`)
4. Push a la rama (`git push origin feature/nueva-herramienta`)
5. Abre un Pull Request

---

## Licencia

MIT License - Usa este código como quieras.

---

## Recursos Adicionales

- [Documentación de LangChain](https://python.langchain.com/docs/)
- [API de DeepSeek](https://platform.deepseek.com/api-docs/)
- [Patrón ReAct (Paper)](https://arxiv.org/abs/2210.03629)

---

*Creado con fines educativos para aprender sobre agentes inteligentes con LangChain.*
