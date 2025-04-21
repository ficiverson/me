---
title: "Jugando con MCP y los buses de Coruña"
meta_title: "Jugando con MCP y los buses de Coruña"
description: "Jugando con MCP y los buses de Coruña"
date: 2025-04-20T05:00:00Z
image: "/images/blog/buses/intro.jpeg"
categories: ["AI"]
author: "Fernando Souto"
tags: ["MCP","tools","LLM","Bus"]
draft: false
---

Llevo unas semanas jugando con el protocolo **Model Context Protocol (MCP)**. Hay ya bastante material por ahí, pero quería probarlo con un caso sencillo y real: **los autobuses de A Coruña**.

Mientras aprendía, pensaba en alguna pequeña idea para cacharrear y se me ocurrió algo como el clásico "¿cuándo llega mi bus?".
No es lo más revolucionario del mundo, pero es perfecto para experimentar con un sistema de datos en tiempo real que pueda usar la LLM. Y claro, ¿por qué no integrar **Bus Coruña** directamente en un entorno MCP para que un LLM pueda consultarlo?

En este ejemplo se le pregunta al modelo:

**"¿Cuándo sale el siguiente bus del Abente y Lago"**

Y devuelve directamente los tiempos de llegada desde el backend de Bus Coruña.

[Conversación completa en Claude](https://claude.ai/share/6dbf72ea-921c-40b6-a593-4c6c8dcf95c5)
![Screenshot de Claude Desktop app](/images/blog/buses/calude-conversation.png)

En este ejemplo se le pregunta al modelo:

**"Estoy en Riazor, ¿cómo llego a San Andrés?"**

Lo más interesante es que a priori lo monté para solo decir los datos de una parada pero la IA es capaz de hacer itinerarios sin que yo lo hubiera programado dado que busca origen y destino automágicamente invocando varias veces las tools para su cálculo.

[Conversación completa en Claude](https://claude.ai/share/dccbf7e7-66ed-4492-b8bc-a140cff63eae)
![Screenshot de Claude Desktop app](/images/blog/buses/claude-conversation-complex.png)

## Detalles técnicos

- El MCP server está en **Python**, usando [fastmcp](https://github.com/jlowin/fastmcp).

```python
from fastmcp import FastMCP, Context
import os
import json
import httpx
import difflib
import copy
import time

# Create an MCP server
mcp = FastMCP("bus-finder")

@mcp.tool()
def get_bus_timetable(stop: int) -> dict:
    """Get a bus timetable for a given stop number"""
    return {}

@mcp.tool()
def get_stop_code_by_location(location: str) -> dict:
    """Return stop code(s) given a location by searching all JSON files in the stops directory. Uses fuzzy matching for similar names."""
    return {}

if __name__ == "__main__":
    print("Starting MCP server...")
    mcp.run()
```

- Tool para conseguir el código de parada **get_stop_code_by_location**.

```python
@mcp.tool()
def get_stop_code_by_location(location: str) -> dict:
    """Return stop code(s) given a location by searching all JSON files in the stops directory. Uses fuzzy matching for similar names."""
    BASE_DIR = os.path.dirname(os.path.abspath(__file__))
    stops_dir = os.path.join(BASE_DIR, "stops")
    results = []
    threshold = 0.7  # Similarity threshold for fuzzy matching
    location_lower = location.lower()
    for filename in os.listdir(stops_dir):
        if filename.endswith('.json'):
            filepath = os.path.join(stops_dir, filename)
            try:
                with open(filepath, 'r') as f:
                    data = json.load(f)
                for direction in data.get('directions', []):
                    for stop in direction.get('stops', []):
                        stop_name_lower = stop['name'].lower()
                        # Substring match (legacy behavior)
                        if location_lower in stop_name_lower:
                            results.append({
                                'code': stop['code'],
                                'name': stop['name'],
                                'file': filename,
                                'match_type': 'substring',
                                'similarity': 1.0
                            })
                        else:
                            # Fuzzy match
                            similarity = difflib.SequenceMatcher(None, location_lower, stop_name_lower).ratio()
                            if similarity >= threshold:
                                results.append({
                                    'code': stop['code'],
                                    'name': stop['name'],
                                    'file': filename,
                                    'match_type': 'fuzzy',
                                    'similarity': similarity
                                })
            except Exception as e:
                continue
    if not results:
        return {"error": f"No stop found for location: {location}"}
    # Optionally, sort by similarity (descending)
    results.sort(key=lambda x: x['similarity'], reverse=True)
    return {"matches": results}
```

- Tool para conseguir los tiempos de llegada **get_bus_timetable**.

```python
@mcp.tool()
def get_bus_timetable(stop: int) -> dict:
    """Get a bus timetable for a given stop number"""
    try:
        response = httpx.get(
            f"https://busapi/dato={stop}&func=0&_={int(time.time() * 1000)}",
            timeout=60
        )
        response.raise_for_status()
        try:
            timetable = response.json()
        except Exception as e:
            timetable = {"error": f"Error parsing JSON: {e}", "raw": response.text}
    except Exception as e:
        timetable = {"error": f"Error during API analysis: {e}"}
    return map_line_numbers_to_friendly_names(timetable)
```


- Ejemplo de contenido de las líneas de bus 

```json
{
  "line": "1",
  "directions": [
    {
      "from": "Abente y Lago",
      "to": "Castrillón",
      "stops": [
        { "code": 523, "name": "Abente y Lago" },
        { "code": 598, "name": "Avenida Porto, A Terraza" },
        { "code": 5, "name": "Pza. de Ourense" },
        { "code": 6, "name": "Linares Rivas, 26" },
        { "code": 7, "name": "Primo de Rivera, 1" },
        { "code": 270, "name": "Pza. da Palloza" },
        { "code": 271, "name": "Av. do Exército, Casa do Mar" },
        { "code": 272, "name": "Av. do Exército, 16" },
        { "code": 416, "name": "Av. do Exército, 44" },
        { "code": 524, "name": "Av. do Exército, 68" },
        { "code": 525, "name": "Os Castros, Av. da Concordia" },
        { "code": 64, "name": "Av. da Concordia, 14" },
        { "code": 65, "name": "Av. da Concordia, 50" },
        { "code": 66, "name": "Av. da Concordia, 72" },
        { "code": 67, "name": "Abegondo, 3" },
        { "code": 68, "name": "Pza. de Pablo Iglesias" }
      ]
    },
    {
      "from": "Castrillón",
      "to": "Abente y Lago",
      "stops": [
        { "code": 68, "name": "Pza. de Pablo Iglesias" },
        { "code": 69, "name": "Av. da Concordia, 188" },
        { "code": 70, "name": "Av. de Monelos, 141" },
        { "code": 71, "name": "Av. de Monelos, 103" },
        { "code": 72, "name": "Av. de Monelos, 45" },
        { "code": 73, "name": "Cabaleiros, 33" },
        { "code": 74, "name": "Cabaleiros, Estación Autobuses" },
        { "code": 75, "name": "Concepción Arenal, 21" },
        { "code": 41, "name": "Costa da Palloza, 5" },
        { "code": 23, "name": "Primo de Rivera, A Palloza" },
        { "code": 24, "name": "Primo de Rivera, viaduto" },
        { "code": 25, "name": "Pza. de Ourense" },
        { "code": 597, "name": "Avenida Porto, entrada parking" },
        { "code": 523, "name": "Abente y Lago" }
      ]
    }
  ]
}
```

## El poder de dotar de tools a tus LLMs

Lo que empezó como un pequeño experimento para consultar los tiempos de llegada de los buses en una parada concreta de A Coruña, acabó revelando algo mucho más interesante.


**Lo más sorprendente** fue ver cómo el modelo, sin haber sido programado explícitamente para ello, es capaz de **deducir rutas completas** buscando automáticamente origen y destino. Esto demuestra el verdadero potencial de combinar herramientas bien definidas con modelos de lenguaje: no hace falta codificar cada posible interacción si se diseñan bien los inputs y outputs.

Además:

- MCP facilita la conexión entre el modelo y APIs externas de forma limpia y estructurada.
- La IA puede aprovechar esa estructura para tomar decisiones y razonar de forma emergente.
- Con muy poco código puedes montar un asistente útil con acceso a datos en tiempo real.

Repositorio en Github: [Github](https://github.com/ficiverson/mcp-bus-coruna)

