---
title: "Playing with MCP and the Coruña Buses"
meta_title: "Playing with MCP and the Coruña Buses"
description: "Playing with MCP and the Coruña Buses"
date: 2025-04-20T05:00:00Z
image: "/images/blog/buses/intro.jpeg"
categories: ["AI"]
author: "Fernando Souto"
tags: ["MCP","tools","LLM","Bus"]
draft: false
---

I’ve been playing around with the **Model Context Protocol (MCP)** for a few weeks now. There’s already a lot of material out there, but I wanted to try it with a simple, real-world case: **the buses in A Coruña**.

While learning, I was thinking of a small project to tinker with, and the classic “when is my bus arriving?” came to mind.
It’s not the most revolutionary idea, but it’s perfect for experimenting with a real-time data system that a LLM can use. And of course, why not integrate **Bus Coruña** directly into an MCP environment so a LLM can query it?

In this example, the model is asked:

**"When is the next bus leaving from Abente y Lago?"**

And it returns the arrival times directly from the Bus Coruña backend.

[Conversation in Claude](https://claude.ai/share/6dbf72ea-921c-40b6-a593-4c6c8dcf95c5)
![Screenshot from Claude Desktop app](/images/blog/buses/calude-conversation.png)

In this example, the model is asked:

**"I'm at Riazor, how do I get to San Andrés?"**

What’s most interesting is that I originally set it up just to provide data for a single stop, but the AI is able to generate full itineraries without me having programmed that behavior — it automatically finds the origin and destination, invoking the tools multiple times to calculate the route.

[Conversation in Claude](https://claude.ai/share/dccbf7e7-66ed-4492-b8bc-a140cff63eae)
![Screenshot from Claude Desktop app](/images/blog/buses/claude-conversation-complex.png)

## Technical details

- The MCP server is built in **Python**, using [fastmcp](https://github.com/jlowin/fastmcp).

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

- Tool to get stop code **get_stop_code_by_location**.

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

- Tool to get arrival times **get_bus_timetable**.

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


- Example of bus line content

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

## The power of equipping your LLMs with tools

What started as a small experiment to check bus arrival times at a specific stop in A Coruña ended up revealing something much more interesting.

**The most surprising part** was seeing how the model, without being explicitly programmed for it, **is able to deduce complete routes** by automatically finding the origin and destination. This shows the true potential of combining well-defined tools with language models: there’s no need to hardcode every possible interaction if inputs and outputs are well structured.

Additionally:

- MCP makes it easy to connect the model to external APIs in a clean and structured way.
- The AI can leverage that structure to make decisions and reason in an emergent manner.
- With very little code, you can build a useful assistant with access to real-time data.

GitHub repository: [Github](https://github.com/ficiverson/mcp-bus-coruna)

