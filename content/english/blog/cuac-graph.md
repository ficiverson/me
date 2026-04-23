---
title: "From Audio to Knowledge: How We Turned CUAC FM's Sound Archive into a Queryable Knowledge Graph"
meta_title: "From Audio to Knowledge: How We Turned CUAC FM's Sound Archive into a Queryable Knowledge Graph"
description: "How to transform years of community radio into a queryable and reusable knowledge graph with AI."
date: 2026-04-22T05:00:00Z
image: "/images/blog/cuac-graph/cuac-graph-en.png"
categories: ["AI"]
language: "en"
author: "Fernando Souto"
tags: ["AI", "Neo4j", "GraphRAG", "Python", "RAG", "LLM","Community Radio"]
draft: false
---

**Audio of the post:**

<audio controls>
  <source src="/audios/cuac-graph-en.mp3" type="audio/mp3">
  Your browser does not support the audio element.
</audio>

When a community radio station accumulates years of shows, interviews, debates, and local memory, the biggest risk is not losing the audio.  
The biggest risk is not being able to find it.

That was exactly the starting point for this CUAC FM project: transform a historical sound archive into something **navigable, searchable, and reusable** with AI, without losing the community context that gives it meaning.

## The real problem: high value, low accessibility

A sound archive is not just a podcast collection. It is cultural, political, and social memory.

The challenge is that even when everything is published, answering specific questions is still hard:

- "In which episode did they talk about X?"
- "What did they say about Y in the latest show?"
- "Which programs covered this topic during 2024?"

Without a structure layer, the audio remains "locked" inside thousands of minutes of content.

## The core idea: treat radio as a knowledge graph

The solution was to model the archive with graph logic:

- `Programme`
- `Episode`
- `Segment` (time-aligned fragment of transcribed audio)
- `Entity` (people, organizations, places, concepts)
- `Topic`

Connected through explicit relationships:

- `Programme -> PRODUCED -> Episode`
- `Episode -> CONTAINS -> Segment`
- `Episode/Segment -> DISCUSSES -> Entity`
- `Episode/Segment -> ABOUT -> Topic`

When you represent the archive this way, you no longer have only indexed text.  
You get **structured context**.

## What the graph adds beyond traditional search

Keyword search works for simple cases, but it fails on real conversational questions.

The graph adds three key benefits:

1. **Relational context**  
   It does not just find words. It understands which entity appears in which episode, in which program, and at what exact point.

2. **More precise queries**  
   It allows combining constraints such as program, date, episode, entities, and topics.

3. **Better retrieval for chat**  
   Combined with hybrid retrieval (vector + keyword + graph signals), it becomes much more robust.

## From audio to answer: high-level pipeline

This is the flow we are using:

1. Episode ingestion and time segmentation.
2. Transcription and content normalization.
3. Entity/topic extraction.
4. Neo4j graph load.
5. Hybrid retrieval for each question.
6. Chat response with sources and traceability.

The goal is not "make AI sound good."  
The goal is reliable answers grounded in the archive, with verifiable references.

## Why this matters for community radio

In community media, technology should not replace local voices.  
It should **make them more accessible**.

This approach helps:

- Reuse years of radio production.
- Support research and documentation.
- Improve access for first-time listeners.
- Give historical content a second life without redoing editorial work.

In short: turn a passive archive into living memory infrastructure.

## What results are already showing

In our internal retriever evaluations (large question set), we see quality improvements versus the baseline, especially in keyword coverage and top-k retrieval accuracy.

That said, there is still work ahead: the hardest problem is not only better retrieval, but **maintaining conversational context** across follow-up questions ("that show," "that topic," "what we said before").

That is the next frontier: conversation memory + useful compaction without losing key entities and facts.

## AI at CUAC FM: experimentation with purpose

This project is part of a broader CUAC FM initiative exploring AI in community radio.  
For additional context:

- CUAC FM Sound Archive: [https://cuacfm.org/arquivo-sonoro/](https://cuacfm.org/arquivo-sonoro/)
- AI at CUAC FM: [https://cuacfm.org/ia/](https://cuacfm.org/ia/)
- The initiative reference: [https://cuacfm.org/novas/cuac-fm-intelixencia-artificial-transcricion-podcasts-arquivo-historico/](https://cuacfm.org/novas/cuac-fm-intelixencia-artificial-transcricion-podcasts-arquivo-historico/)

## Closing

For me, this is the most meaningful part of the project: it is not about "adding AI for the sake of AI."  

It is about preserving collective memory, making it queryable, and returning it to the community as real utility.

If you have a media archive, a podcast library, or years of underused audio, this shift is not only technical. 
 
It is editorial and cultural.

You move from storing content to building knowledge.



