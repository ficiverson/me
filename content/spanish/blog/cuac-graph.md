---
title: "Del audio al conocimiento: cómo convertimos el arquivo sonoro de CUAC FM en un grafo consultable"
meta_title: "Del audio al conocimiento: cómo convertimos el arquivo sonoro de CUAC FM en un grafo consultable"
description: "Cómo transformar años de radio comunitaria en un grafo de conocimiento consultable y reutilizable con IA."
date: 2026-04-22T05:00:00Z
image: "/images/blog/cuac-graph/cuac-graph.png"
categories: ["AI"]
language: "es"
author: "Fernando Souto"
tags: ["AI", "Neo4j", "GraphRAG", "Python", "RAG", "LLM","Radio comunitaria"]
draft: false
---

**Audio del post:**

<audio controls>
  <source src="/audios/cuac-graph-es.mp3" type="audio/mp3">
  Your browser does not support the audio element.
</audio>

Si una radio comunitaria acumula años de programas, entrevistas, debates y memoria local, el mayor riesgo no es perder el audio: es no poder encontrarlo.

Ese fue exactamente el punto de partida de este proyecto en CUAC FM: transformar un archivo sonoro histórico en algo **navegable, consultable y reutilizable** con IA, sin perder el contexto comunitario que le da sentido.

## El problema real: mucho valor, poca accesibilidad

Un arquivo sonoro no es solo una colección de podcasts. Es memoria cultural, política y social de una ciudad.

El problema es que, aunque todo esté publicado, responder preguntas concretas sigue siendo difícil:

- "¿En qué episodio se habló de Inés Rey?"
- "¿Qué dijeron sobre el Deportivo en el último programa?"
- "¿Qué programas trataron este poesía durante 1996?"

Sin una capa de estructura, el audio queda "encerrado" en miles de minutos de contenido.

## La idea: tratar la radio como un grafo de conocimiento

La solución fue modelar el archivo con una lógica de grafo:

- `Programme` (programa)
- `Episode` (episodio)
- `Segment` (fragmento temporal del audio transcrito)
- `Entity` (personas, organizaciones, lugares, conceptos)
- `Topic` (temas)

Y conectarlo con relaciones explícitas:

- `Programme -> PRODUCED -> Episode`
- `Episode -> CONTAINS -> Segment`
- `Segment -> DISCUSSES -> Entity`
- `Segment -> ABOUT -> Topic`

Cuando representas así el archivo, dejas de tener solo "texto indexado" y pasas a tener **contexto estructurado**.

## Qué aporta el grafo frente a una búsqueda tradicional

Una búsqueda por palabras clave sirve para cosas simples, pero falla con preguntas reales de conversación.

El grafo añade tres ventajas clave:

1. **Contexto relacional**  
   No solo encuentra palabras, entiende qué entidad aparece en qué episodio, en qué programa y en qué parte exacta.

2. **Consultas más precisas**  
   Permite combinar restricciones de programa, fechas, episodios, entidades y temas.

3. **Mejor recuperación para chat**  
   Si encima lo combinas con recuperación híbrida (vectorial + keyword + señales de grafo), el resultado es más robusto.

## Del áudio a la respuesta: pipeline de alto nivel

Este flujo resume cómo lo estamos resolviendo:

1. Ingesta de episodios y segmentación temporal.
2. Transcripción y normalización del contenido.
3. Extracción de entidades/temas.
4. Carga en Neo4j como grafo de conocimiento.
5. Recuperación híbrida para cada pregunta.
6. Respuesta en chat con fuentes y trazabilidad.

El objetivo no es "que la IA improvise bonito", sino que responda con soporte en el archivo y con referencias verificables.

## Por qué esto importa en una radio comunitaria

En un medio comunitario, la tecnología no debería sustituir la voz local: debería **hacerla más accesible**.

Este enfoque permite:

- Reutilizar mejor años de producción radiofónica.
- Facilitar investigación y documentación.
- Mejorar la experiencia de oyentes que llegan por primera vez.
- Dar nueva vida a contenidos históricos sin rehacer trabajo editorial.

En otras palabras: convertir un archivo pasivo en una infraestructura viva de memoria.

## Lo que ya se ve en resultados

En nuestras evaluaciones internas del retriever (dataset amplio de preguntas), se observan mejoras en recuperación frente a la versión base, especialmente en cobertura de keywords y acierto global de top-k.

Aun así, hay trabajo pendiente: el mayor reto no está solo en recuperar mejor, sino en **mantener contexto conversacional** en preguntas de seguimiento ("ese programa", "ese tema", "lo anterior").

Esa es la siguiente frontera: memoria de conversación + compactación útil sin perder entidades clave.

## IA en CUAC FM: experimentar con propósito

Este proyecto encaja con una línea más amplia de experimentación en CUAC FM alrededor de IA y radio comunitaria.  
Si quieres contexto adicional, aquí tienes dos enlaces:

- Archivo sonoro de CUAC FM: [https://cuacfm.org/arquivo-sonoro/](https://cuacfm.org/arquivo-sonoro/)
- IA en CUAC FM: [https://cuacfm.org/ia/](https://cuacfm.org/ia/)
- Referencia: [https://cuacfm.org/novas/cuac-fm-intelixencia-artificial-transcricion-podcasts-arquivo-historico/](https://cuacfm.org/novas/cuac-fm-intelixencia-artificial-transcricion-podcasts-arquivo-historico/)

## Cierre

Para mí, esta es la parte más bonita del proyecto: no va de "poner IA porque sí".  

Va de preservar memoria colectiva, hacerla consultable y devolverla a la comunidad en forma de utilidad real.

Si tienes una hemeroteca, un archivo de podcasts o años de audio sin explotar, el salto no es solo técnico: es editorial y cultural.  

Pasas de almacenar contenidos a construir conocimiento.

