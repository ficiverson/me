---
title: "Construyendo un sistema de validación de artículos periodísticos con IA y Human-in-the-Loop"
meta_title: "Construyendo un sistema de validación de artículos periodísticos con IA y Human-in-the-Loop"
description: "Construyendo un sistema de validación de artículos periodísticos con IA y Human-in-the-Loop"
date: 2025-11-04T05:00:00Z
image: "/images/blog/middleware/editor.png"
categories: ["AI","news"]
author: "Fernando Souto"
tags: ["human in the loop","middleware", "Langchain"]
draft: false
---

**Audio del artítculo:**

<audio controls>
  <source src="/audios/middleware_es.wav" type="audio/wav">
  Your browser does not support the audio element.
</audio>

En el vertiginoso mundo del periodismo digital, los editores se enfrentan a la desalentadora tarea de revisar cientos de artículos enviados diariamente. Aunque la IA puede redactar artículos rápidamente, la supervisión humana sigue siendo crucial para mantener los estándares editoriales, garantizar la precisión de los hechos y preservar la integridad periodística. Aquí es donde el **middleware Human-in-the-Loop (HITL)** se vuelve crucial.

En esta publicación del blog, exploraremos cómo construir un sistema de aprobación de artículos periodísticos que combina la generación de contenido con IA y la supervisión editorial humana, utilizando LangChain y Azure OpenAI. Revisaremos la implementación, mostraremos ejemplos de código y discutiremos advertencias importantes.

## El problema: escalar vs calidad

Las redacciones modernas reciben un volumen abrumador de artículos enviados. Los procesos tradicionales de revisión manual no pueden escalar, pero los sistemas totalmente automatizados arriesgan publicar contenido inapropiado o inexacto. ¿La solución? Un flujo de trabajo inteligente que:

1. **La IA genera** borradores de artículos basados en temas o indicaciones
2. **Los humanos revisan** cada artículo en cuanto a calidad, precisión y ajuste editorial
3. **El refinamiento iterativo** permite la mejora colaborativa
4. **La aprobación final** garantiza que nada se publique sin consentimiento humano

## Descripción general del proyecto

La solución utiliza un middleware Human-in-the-Loop personalizado construido sobre LangChain que permite la integración perfecta de puntos de decisión humanos en flujos de trabajo automatizados. El sistema incluye:

- **Integración con Azure OpenAI**: Para la generación de artículos
- **Middleware flexible**: Puntos de entrada humanos personalizables
- **Composición de flujos de trabajo**: Encadenamiento de múltiples pasos con puertas de aprobación
- **Refinamiento iterativo**: Múltiples rondas de colaboración humano-IA

## Arquitectura: los componentes principales

#### Paso 1: Configuración e inicialización

```python
from newspaper_article_example import create_newspaper_workflow
from config import get_azure_openai_llm
from langchain.schema.runnable import RunnableLambda

# Initialize the workflow manager
llm = get_azure_openai_llm()  # Get Azure OpenAI instance
workflow_manager = create_newspaper_workflow(llm)
workflow = workflow_manager.create_article_approval_workflow()

# The workflow is now ready to use
# Input: string (article topic)
# Output: string (final approval result)
```

#### Paso 2: Generación de artículos (paso de IA)

**Qué sucede:**
1. Entrada: Cadena de texto con el tema (ej., "El mejor presidente del Real Club Deportivo A Coruña.")
2. La IA genera el contenido del artículo
3. Salida: Diccionario con los datos del artículo

```python
def generate_article_draft(topic_and_details: str) -> Dict[str, Any]:
    """
    Step 1: AI generates initial article draft
    
    INPUT STATE:
    - topic_and_details: "The best president of Real Club Deportivo A Coruña."
    
    PROCESS:
    1. Create detailed prompt for AI
    2. Call Azure OpenAI API
    3. Receive generated article content
    4. Structure data for next step
    
    OUTPUT STATE:
    {
        "topic": "The best president of Real Club Deportivo A Coruña.",
        "article": "In recent months, major tech companies...",  # Generated content
        "status": "draft",      # Initial status
        "version": 1,           # Version tracking
        "editor_notes": []      # Empty list for future notes
    }
    """
    print(f"\n📰 Generating article draft about: {topic_and_details}")
    
    # Construct detailed prompt
    prompt = f"""Write a professional news article about: {topic_and_details}

Requirements:
- Objective and factual tone
- Include key facts and context
- Follow journalistic style (who, what, when, where, why)
- 500-800 words
- Suitable for publication in a reputable newspaper
- Include a compelling headline
- Structure with clear paragraphs and sections"""
    
    if self.llm:
        try:
            # API call to Azure OpenAI
            response = self.llm.invoke(prompt)
            
            # Structure the output data
            return {
                "topic": topic_and_details,
                "article": response.content,  # AI-generated article
                "status": "draft",            # Current workflow state
                "version": 1,                 # Track versions for revisions
                "editor_notes": []            # Store editor feedback
            }
        except Exception as e:
            print(f"❌ AI generation failed: {e}")
            # Return fallback data structure (same format)
            return {
                "topic": topic_and_details,
                "article": f"Draft article about {topic_and_details} (simulated)",
                "status": "draft",
                "version": 1,
                "editor_notes": []
            }
```

**Flujo de datos:**
```python
Input: "The best president of Real Club Deportivo A Coruña."
  ↓
[AI Processing]
  ↓
Output: {
  "topic": "The best president of Real Club Deportivo A Coruña.",
  "article": "...",
  "status": "draft",
  "version": 1,
  "editor_notes": []
}
```

#### Paso 3: Verificación de calidad (paso automatizado)

**Qué sucede:**
1. Entrada: Diccionario de datos del artículo
2. Verificaciones automatizadas (recuento de palabras, citas, etc.)
3. Salida: Mismo diccionario con advertencias de calidad añadidas


```python
def quality_check(data: Dict[str, Any]) -> Dict[str, Any]:
    """
    Step 2: Automated quality checks
    
    INPUT STATE:
    {
        "topic": "...",
        "article": "In recent months...",
        "status": "draft",
        "version": 1,
        "editor_notes": []
    }
    
    PROCESS:
    1. Extract article content
    2. Run quality metrics:
       - Word count
       - Presence of quotes
       - Presence of numbers
       - Sentence count
    3. Generate warnings if issues found
    
    OUTPUT STATE:
    {
        "topic": "...",
        "article": "...",
        "status": "draft",
        "version": 1,
        "editor_notes": [],
        "quality_warnings": ["Article may need quotes from sources"]  # NEW
    }
    """
    article = data['article']
    
    # Calculate quality metrics
    checks = {
        'word_count': len(article.split()),
        'has_quotes': '"' in article or "'" in article,
        'has_numbers': any(char.isdigit() for char in article),
        'sentence_count': article.count('.') + article.count('!') + article.count('?')
    }
    
    # Generate warnings based on checks
    warnings = []
    if checks['word_count'] < 300:
        warnings.append("Article may be too short")
    if checks['word_count'] > 1500:
        warnings.append("Article may be too long")
    if not checks['has_quotes']:
        warnings.append("Article may need quotes from sources")
    if checks['sentence_count'] < 10:
        warnings.append("Article may need more sentences")
    
    # Add warnings to data (preserving all existing data)
    if warnings:
        data['quality_warnings'] = warnings
        print(f"\n⚠️ Quality Warnings: {', '.join(warnings)}")
    
    return data  # Same structure, with warnings added
```

**Flujo de datos:**
```python
Input: {article: "...", status: "draft", ...}
  ↓
[Quality Checks]
  ↓
Output: {article: "...", status: "draft", quality_warnings: [...], ...}
```

#### Paso 4: Revisión del editor (paso Human-in-the-Loop)

**Qué sucede:**
1. Entrada: Datos del artículo con advertencias de calidad
2. **PAUSA**: El editor humano revisa el artículo
3. El humano toma una decisión (aprobar/rechazar/revisar)
4. Salida: Datos actualizados con la decisión del editor


```python
def editor_review(data: Dict[str, Any]) -> Dict[str, Any]:
    """
    Step 3: Human editor reviews the article
    
    INPUT STATE:
    {
        "topic": "...",
        "article": "...",
        "status": "draft",
        "version": 1,
        "editor_notes": [],
        "quality_warnings": [...]
    }
    
    PROCESS:
    1. Display article and metadata to editor
    2. PAUSE: Wait for human input
    3. Process editor decision:
       - "approve" → status = "approved"
       - "reject" → status = "rejected"
       - "revise" → status = "needs_revision"
    4. Collect additional information (notes, feedback)
    
    OUTPUT STATE (if approved):
    {
        "topic": "...",
        "article": "...",
        "status": "approved",           # CHANGED
        "version": 1,
        "editor_notes": ["Approval notes: Looks good"],  # UPDATED
        "editor_decision": "approved",   # NEW
        "quality_warnings": [...]
    }
    
    OUTPUT STATE (if needs revision):
    {
        "topic": "...",
        "article": "...",
        "status": "needs_revision",     # CHANGED
        "version": 1,
        "editor_notes": ["Revision feedback: Add more statistics"],  # UPDATED
        "editor_decision": "revise",     # NEW
        "revision_feedback": "Add more statistics",  # NEW
        "quality_warnings": [...]
    }
    """
    # Display article for review
    print(f"\n{'='*60}")
    print(f"📝 ARTICLE FOR EDITORIAL REVIEW")
    print(f"{'='*60}")
    print(f"Topic: {data['topic']}")
    print(f"Version: {data['version']}")
    print(f"Word Count: {len(data['article'].split())}")
    
    if 'quality_warnings' in data:
        print(f"\n⚠️ Quality Warnings: {', '.join(data['quality_warnings'])}")
    
    # Show full article content
    print(f"\n{'='*60}")
    print("ARTICLE CONTENT:")
    print(f"{'='*60}")
    print(data['article'])
    print(f"{'='*60}")
    
    # HUMAN INPUT POINT - Execution pauses here
    print(f"\n🤖 Editorial Decision Required:")
    print("Options:")
    print("  - 'approve': Accept article for publication")
    print("  - 'reject': Reject article")
    print("  - 'revise': Request revisions with feedback")
    
    # This is where the workflow pauses and waits for human input
    decision = self.middleware.get_human_input("Editor decision: ").strip().lower()
    
    # Process decision and update state
    if decision == 'approve':
        data['status'] = 'approved'
        data['editor_decision'] = 'approved'
        # Optional: Collect additional notes
        notes = self.middleware.get_human_input(
            "Optional editor notes (or press Enter to skip): "
        ).strip()
        if notes:
            data['editor_notes'].append(f"Approval notes: {notes}")
    
    elif decision == 'reject':
        data['status'] = 'rejected'
        data['editor_decision'] = 'rejected'
        reason = self.middleware.get_human_input("Rejection reason: ").strip()
        data['rejection_reason'] = reason
        data['editor_notes'].append(f"Rejection reason: {reason}")
    
    else:  # revise
        data['status'] = 'needs_revision'
        data['editor_decision'] = 'revise'
        # Collect revision feedback
        feedback = self.middleware.get_human_input("Revision feedback: ").strip()
        data['revision_feedback'] = feedback
        data['editor_notes'].append(f"Revision feedback: {feedback}")
    
    return data  # Return updated state
```

**Flujo de datos (ruta de aprobación):**
```python
Input: {article: "...", status: "draft", ...}
  ↓
[HUMAN REVIEW - PAUSES HERE]
  ↓ Editor types: "approve"
  ↓
Output: {article: "...", status: "approved", editor_decision: "approved", ...}
```
**Flujo de datos (ruta de revisión):**
```python
Input: {article: "...", status: "draft", ...}
  ↓
[HUMAN REVIEW - PAUSES HERE]
  ↓ Editor types: "revise"
  ↓ Editor provides: "Add more statistics"
  ↓
Output: {
  article: "...",
  status: "needs_revision",
  revision_feedback: "Add more statistics",
  ...
}
```

#### Paso 5: Revisión por IA (paso condicional)

**Qué sucede:**
1. Entrada: Datos del artículo (solo procesa si status = "needs_revision")
2. La IA revisa el artículo basándose en la retroalimentación
3. Salida: Artículo actualizado con nuevo número de versión


```python
def ai_revision(data: Dict[str, Any]) -> Dict[str, Any]:
    """
    Step 4: AI revises article based on editor feedback
    
    INPUT STATE (if revision needed):
    {
        "topic": "...",
        "article": "Original article content...",
        "status": "needs_revision",
        "version": 1,
        "revision_feedback": "Add more statistics"
    }
    
    PROCESS:
    1. Check if revision is needed (status = "needs_revision")
    2. If not needed, return data unchanged
    3. If needed:
       - Create revision prompt with original article + feedback
       - Call AI to generate revised version
       - Update article content
       - Increment version number
    
    OUTPUT STATE:
    {
        "topic": "...",
        "article": "Revised article with statistics added...",  # UPDATED
        "status": "revised",     # CHANGED
        "version": 2,            # INCREMENTED
        "revision_feedback": "Add more statistics",
        "last_feedback": "Add more statistics"  # NEW
    }
    """
    # Conditional processing: only revise if needed
    if data['status'] != 'needs_revision':
        return data  # Pass through unchanged
    
    print(f"\n🤖 AI revising article based on editor feedback...")
    
    # Create comprehensive revision prompt
    revision_prompt = f"""Original article topic: {data['topic']}

Current article:
{data['article']}

Editor feedback: {data['revision_feedback']}

Please revise the article based on the feedback above. 
Maintain journalistic standards and incorporate the editor's suggestions.
Keep the same length and style."""
    
    if self.llm:
        try:
            # Call AI for revision
            response = self.llm.invoke(revision_prompt)
            
            # Update article content
            data['article'] = response.content
            data['version'] += 1  # Increment version
            data['status'] = 'revised'
            data['last_feedback'] = data['revision_feedback']
            
            print(f"✅ Article revised (Version {data['version']})")
        except Exception as e:
            print(f"❌ Revision failed: {e}")
            data['status'] = 'revision_failed'
    
    return data
```

**Flujo de datos:**
```python
Input: {status: "needs_revision", article: "...", revision_feedback: "..."}
  ↓
[Check: status == "needs_revision"?]
  ↓ YES
[AI Revision]
  ↓
Output: {status: "revised", article: "[REVISED]", version: 2, ...}
```

#### Paso 6: Aprobación final (paso de decisión)

**Qué sucede:**
1. Entrada: Datos del artículo con estado final
2. Procesamiento basado en el estado (aprobado/rechazado/revisado)
3. Salida: Cadena de texto con resultado final


```python
def final_approval(data: Dict[str, Any]) -> str:
    """
    Step 5: Final editorial approval
    
    INPUT STATE (if approved):
    {
        "topic": "...",
        "article": "...",
        "status": "approved",
        "version": 1,
        "editor_notes": [...]
    }
    
    INPUT STATE (if revised):
    {
        "topic": "...",
        "article": "[Revised content]",
        "status": "revised",
        "version": 2,
        "revision_feedback": "..."
    }
    
    PROCESS:
    - If approved: Return approval message with article
    - If rejected: Return rejection message
    - If revised: Show revised article and ask for final decision
    - If revision failed: Return error message
    
    OUTPUT: Final result string
    """
    if data['status'] == 'approved':
        # Direct approval path
        return f"""✅ ARTICLE APPROVED FOR PUBLICATION!

Topic: {data['topic']}
Version: {data['version']}
Editor Notes: {', '.join(data['editor_notes']) if data['editor_notes'] else 'None'}

{'='*60}
FINAL ARTICLE:
{'='*60}
{data['article']}
{'='*60}"""
    
    elif data['status'] == 'rejected':
        return f"""❌ ARTICLE REJECTED

Topic: {data['topic']}
Reason: {data.get('rejection_reason', 'Not specified')}
Editor Notes: {', '.join(data['editor_notes'])}"""
    
    elif data['status'] == 'revised':
        # Show revised article for final review
        print(f"\n{'='*60}")
        print(f"📝 REVISED ARTICLE FOR FINAL REVIEW")
        print(f"{'='*60}")
        print(f"Topic: {data['topic']}")
        print(f"Version: {data['version']}")
        print(f"Previous Feedback: {data.get('revision_feedback', 'N/A')}")
        print(f"\n{'='*60}")
        print("REVISED ARTICLE:")
        print(f"{'='*60}")
        print(data['article'])
        print(f"{'='*60}")
        
        # Another human input point
        final_decision = self.middleware.get_human_input(
            "Final decision (approve/reject): "
        ).strip().lower()
        
        if final_decision == 'approve':
            return f"""✅ ARTICLE APPROVED AFTER REVISION!

Topic: {data['topic']}
Final Version: {data['version']}

{'='*60}
FINAL ARTICLE:
{'='*60}
{data['article']}
{'='*60}"""
        else:
            reason = self.middleware.get_human_input("Final rejection reason: ").strip()
            return f"""❌ ARTICLE REJECTED AFTER REVISION

Topic: {data['topic']}
Reason: {reason}"""
    
    elif data['status'] == 'revision_failed':
        return f"""❌ ARTICLE REVISION FAILED

Topic: {data['topic']}
The AI was unable to revise the article. Please review manually."""
    
    return f"⏳ Article status: {data['status']}"
```

#### Composición completa del flujo de trabajo

Ahora veamos cómo todos estos pasos se componen juntos:

**Flujo de datos completo:**
```python
# Step 1: Create workflow steps
workflow = (
    RunnableLambda(generate_article_draft)    # Step 1: AI generates
    | RunnableLambda(quality_check)            # Step 2: Automated checks
    | RunnableLambda(editor_review)            # Step 3: Human review (PAUSES)
    | RunnableLambda(ai_revision)              # Step 4: Conditional AI revision
    | RunnableLambda(final_approval)           # Step 5: Final processing
)

# Step 2: Execute workflow
result = workflow.invoke("The best president of Real Club Deportivo A Coruña.")
```

## Advertencias y consideraciones importantes

### Advertencia 1: Alucinaciones de IA y verificación de hechos

**Problema**: El contenido generado por IA puede contener errores factuales o alucinaciones.

**Solución**: 
- Siempre requerir verificación de hechos humana antes de la publicación
- Usar IA para borradores, no para verificación final
- Implementar requisitos de citación de fuentes

```python
def fact_check_reminder(data):
    """Remind editor to fact-check"""
    print("\n⚠️ IMPORTANT: Please verify all facts, figures, and claims before approving.")
    print("Check sources, dates, names, and statistics.")
    return data

# Add to workflow
workflow = (
    RunnableLambda(generate_article)
    | RunnableLambda(fact_check_reminder)
    | RunnableLambda(editor_review)
)
```

### Advertencia 2: Sesgos y estándares editoriales

**Problema**: Los modelos de IA pueden reflejar sesgos de entrenamiento o no coincidir con el estilo editorial.

**Solución**:
- Proporcionar pautas de estilo claras en los prompts
- Los editores humanos deben hacer cumplir los estándares editoriales
- Revisar regularmente la salida de IA para detectar sesgos

```python
def generate_with_style_guide(topic):
    style_guide = """
    Editorial Guidelines:
    - Objective, third-person perspective
    - No opinion or speculation
    - Balanced representation of all sides
    - Verify all claims with sources
    - Follow AP Style guidelines
    """
    
    prompt = f"{style_guide}\n\nWrite an article about: {topic}"
    return llm.invoke(prompt)
```

### Advertencia 3: Costes y limites de presupuesto

**Problema**: Las llamadas a la API de Azure OpenAI pueden ser costosas.

**Solución**:
- Implementar caché para temas similares
- Establecer límites de tokens apropiadamente
- Monitorear el uso y costes de la API

```python
import functools
from functools import lru_cache

@lru_cache(maxsize=100)
def cached_article_generation(topic_hash: str, prompt: str):
    """Cache article generations to reduce API calls"""
    # Only generate if not cached
    return llm.invoke(prompt)

# Use caching
def generate_article_smart(topic):
    topic_hash = hash(topic)
    prompt = f"Write article about: {topic}"
    return cached_article_generation(topic_hash, prompt)
```

### Advertencia 4: Seguridad y privacidad

**Problema**: El contenido de los artículos puede contener información sensible.

**Solución**:
- Nunca registrar el contenido completo de los artículos en producción
- Implementar controles de acceso
- Asegurar conexiones seguras a la API

```python
import logging

# Don't log full articles
def safe_logging(data):
    logger.info(f"Article generated for topic: {data['topic']}")
    logger.info(f"Article length: {len(data['article'])} characters")
    # Don't log: logger.info(f"Article content: {data['article']}")  # NO!
    return data
```

### Advertencia 5: Tiempos de espera del flujo de trabajo

**Problema**: Los pasos de aprobación humana pueden tardar indefinidamente.

**Solución**:
- Implementar mecanismos de tiempo de espera
- Enviar recordatorios para aprobaciones pendientes
- Rechazar automáticamente después del tiempo de espera

```python
import time
from datetime import datetime, timedelta

def approval_with_timeout(data, timeout_hours=24):
    """Approval step with timeout"""
    start_time = datetime.now()
    timeout = timedelta(hours=timeout_hours)
    
    print(f"⏰ Approval timeout: {timeout_hours} hours")
    
    decision = input("👤 Editor decision: ").strip()
    
    # In production, use async/await with timeout
    # if datetime.now() - start_time > timeout:
    #     data['status'] = 'timeout_rejected'
    
    return data
```

### Advertencia 6: Control de versiones

**Problema**: Múltiples revisiones pueden llevar a confusión sobre qué versión es la actual.

**Solución**:
- Mantener un historial de versiones claro
- Almacenar todas las iteraciones
- Mostrar comparación de versiones

```python
def store_version_history(data):
    """Store article versions"""
    version_history = data.get('version_history', [])
    version_history.append({
        'version': data['version'],
        'article': data['article'],
        'timestamp': datetime.now().isoformat(),
        'editor_feedback': data.get('revision_feedback', '')
    })
    data['version_history'] = version_history
    return data
```

## Resumen de mejores prácticas

1. **Siempre verificar la salida de IA**: nunca publicar sin revisión humana
2. **Implementar puertas de calidad**: verificaciones automatizadas antes de la revisión humana
3. **Mantener registros de auditoría**: registrar todas las decisiones y cambios
4. **Establecer pautas claras**: proporcionar a la IA estándares editoriales detallados
5. **Monitorear costos**: rastrear el uso de la API y optimizar los prompts
6. **Manejar errores con gracia**: implementar respaldos para fallos de API
7. **Respetar la privacidad**: no registrar contenido sensible
8. **Establecer tiempos de espera**: evitar que los flujos de trabajo se queden colgados indefinidamente

## Conclusión

Construir un sistema de aprobación de artículos con IA y human-in-the-loop proporciona el equilibrio perfecto entre automatización y control editorial. Al aprovechar la composición de flujos de trabajo de LangChain y middleware personalizado, podemos crear sistemas flexibles que:

- **Escalan** para manejar grandes volúmenes
- **Mantienen la calidad** a través de la supervisión humana
- **Permiten la colaboración** entre IA y editores
- **Aseguran la responsabilidad** a través de flujos de trabajo de aprobación

La clave es entender que la IA es una herramienta para asistir a los editores humanos, no para reemplazarlos. Con una implementación adecuada, gestión de advertencias y mejores prácticas, puede construir sistemas robustos que mejoren los flujos de trabajo editoriales mientras mantienen la integridad periodística.

Aquí dejo un ejemplo completo de ejecución en la consola solicitando información sobre "El mejor presidente del Real Club Deportivo de A Coruña":


```python
(venv) fernando.souto@192 middleware % python main.py --newspaper-article "The best president of Real Club Deportivo A coruña"
📰 Running Newspaper Article Approval Workflow:
==================================================
✅ Azure OpenAI LLM configured successfully!

📰 Generating article draft about: The best president of Real Club Deportivo A coruña

============================================================
📝 ARTICLE FOR EDITORIAL REVIEW
============================================================
Topic: The best president of Real Club Deportivo A coruña
Version: 1
Word Count: 685

============================================================
ARTICLE CONTENT:
============================================================
**Headline:**  
Augusto César Lendoiro: Architect of Deportivo La Coruña's Golden Era

**A Coruña, Spain —** Real Club Deportivo de La Coruña, one of Spain's historic football institutions, has seen a number of presidents since its founding in 1906. Yet, among them, Augusto César Lendoiro stands out as the most transformative and celebrated leader, credited with steering the club through its most successful period in history. His presidency, spanning from 1988 to 2014, not only elevated Deportivo onto the national and European stage but also left an indelible legacy in both the city of A Coruña and Spanish football as a whole.

**Who Was Augusto César Lendoiro?**

Born in A Coruña in 1946, Lendoiro began his career outside of football, working as a teacher and later as a politician. His entry into the world of sports administration started with local clubs, but his appointment as president of Deportivo in June 1988 marked a turning point for both himself and the club. At the time, Deportivo languished in Spain’s Segunda División and faced severe financial difficulties.

**A Presidency That Changed Everything**

Lendoiro’s presidency quickly became synonymous with ambition and innovation. He inherited a club on the brink of bankruptcy, but through a series of strategic measures—including renegotiating debts, fostering local support, and attracting new sponsors—he stabilized the club’s finances. His eye for talent and willingness to take calculated risks in the transfer market soon paid off.

In the early 1990s, Lendoiro brought in legendary players such as Bebeto, Mauro Silva, Djalminha, and Fran, the latter a local icon. Under his leadership, the club achieved promotion to La Liga in 1991, setting the stage for a decade of remarkable achievements.

**Deportivo’s Golden Age**

The 1990s and early 2000s are widely regarded as Deportivo’s "Golden Era." The club finished as La Liga runners-up in 1994 and 1995, narrowly missing out on the title in a dramatic 1994 finale. The pinnacle came in the 1999-2000 season when Deportivo, under coach Javier Irureta and Lendoiro’s stewardship, won their first and only La Liga championship. It was a historic achievement for a club from Galicia, traditionally overshadowed by Spain’s larger teams.

During this period, Deportivo also claimed two Copa del Rey titles (1995, 2002) and three Spanish Super Cups (1995, 2000, 2002). On the European front, the team became regular fixtures in the UEFA Champions League, reaching the semi-finals in the 2003-04 season after a memorable comeback against AC Milan.

**Leadership Style and Legacy**

Lendoiro’s management style combined pragmatism with bold vision. He was known for his negotiating skills, often securing high-profile signings for modest fees. His ability to foster a sense of unity among players, staff, and supporters was instrumental in building a competitive and resilient squad.

Beyond silverware, Lendoiro’s legacy includes modernizing the club’s infrastructure, most notably the Riazor Stadium, and investing in youth development. His presidency also cultivated a strong connection between the club and the city of A Coruña, turning Deportivo into a symbol of regional pride.

**Challenges and Departure**

The latter years of Lendoiro’s tenure were marked by financial challenges, as the club struggled to compete with the resources of Spain’s football giants. Relegation battles and mounting debts eventually led to his resignation in January 2014, ending a 26-year presidency.

Despite these difficulties, Lendoiro is widely credited with raising Deportivo’s profile and establishing the club as a respected force in Spanish and European football. His stewardship remains the benchmark against which subsequent presidents are measured.

**Why Lendoiro Is Considered the Best**

Though Deportivo has had a number of capable presidents, none matched the impact of Lendoiro. He not only delivered unprecedented sporting success but also transformed the club’s identity and ambitions. Football historians, local fans, and former players alike consistently cite his vision, resilience, and leadership as the reasons behind the club’s greatest achievements.

**Conclusion**

As Deportivo La Coruña continues to navigate the challenges of modern football, the legacy of Augusto César Lendoiro endures. His era is remembered as a time when the club dared to dream—and succeeded—against the odds. For supporters and observers, Lendoiro remains, objectively and enduringly, the best president in the club’s storied history.
============================================================

🤖 Editorial Decision Required:
Options:
  - 'approve': Accept article for publication
  - 'reject': Reject article
  - 'revise': Request revisions with feedback

🤖 Human Input Required:
📝 Prompt: Revision feedback: 
👤 Your input: I need at the end the links of the sources used to create the content

🤖 AI revising article based on editor feedback...
✅ Article revised (Version 2)

============================================================
📝 REVISED ARTICLE FOR FINAL REVIEW
============================================================
Topic: The best president of Real Club Deportivo A coruña
Version: 2
Previous Feedback: I need at the end the links of the sources used to create the content

============================================================
REVISED ARTICLE:
============================================================
**Headline:**  
Augusto César Lendoiro: Architect of Deportivo La Coruña's Golden Era

**A Coruña, Spain —** Real Club Deportivo de La Coruña, one of Spain's historic football institutions, has seen a number of presidents since its founding in 1906. Yet, among them, Augusto César Lendoiro stands out as the most transformative and celebrated leader, credited with steering the club through its most successful period in history. His presidency, spanning from 1988 to 2014, not only elevated Deportivo onto the national and European stage but also left an indelible legacy in both the city of A Coruña and Spanish football as a whole.

**Who Was Augusto César Lendoiro?**

Born in A Coruña in 1946, Lendoiro began his career outside of football, working as a teacher and later as a politician. His entry into the world of sports administration started with local clubs, but his appointment as president of Deportivo in June 1988 marked a turning point for both himself and the club. At the time, Deportivo languished in Spain’s Segunda División and faced severe financial difficulties.

**A Presidency That Changed Everything**

Lendoiro’s presidency quickly became synonymous with ambition and innovation. He inherited a club on the brink of bankruptcy, but through a series of strategic measures—including renegotiating debts, fostering local support, and attracting new sponsors—he stabilized the club’s finances. His eye for talent and willingness to take calculated risks in the transfer market soon paid off.

In the early 1990s, Lendoiro brought in legendary players such as Bebeto, Mauro Silva, Djalminha, and Fran, the latter a local icon. Under his leadership, the club achieved promotion to La Liga in 1991, setting the stage for a decade of remarkable achievements.

**Deportivo’s Golden Age**

The 1990s and early 2000s are widely regarded as Deportivo’s "Golden Era." The club finished as La Liga runners-up in 1994 and 1995, narrowly missing out on the title in a dramatic 1994 finale. The pinnacle came in the 1999-2000 season when Deportivo, under coach Javier Irureta and Lendoiro’s stewardship, won their first and only La Liga championship. It was a historic achievement for a club from Galicia, traditionally overshadowed by Spain’s larger teams.

During this period, Deportivo also claimed two Copa del Rey titles (1995, 2002) and three Spanish Super Cups (1995, 2000, 2002). On the European front, the team became regular fixtures in the UEFA Champions League, reaching the semi-finals in the 2003-04 season after a memorable comeback against AC Milan.

**Leadership Style and Legacy**

Lendoiro’s management style combined pragmatism with bold vision. He was known for his negotiating skills, often securing high-profile signings for modest fees. His ability to foster a sense of unity among players, staff, and supporters was instrumental in building a competitive and resilient squad.

Beyond silverware, Lendoiro’s legacy includes modernizing the club’s infrastructure, most notably the Riazor Stadium, and investing in youth development. His presidency also cultivated a strong connection between the club and the city of A Coruña, turning Deportivo into a symbol of regional pride.

**Challenges and Departure**

The latter years of Lendoiro’s tenure were marked by financial challenges, as the club struggled to compete with the resources of Spain’s football giants. Relegation battles and mounting debts eventually led to his resignation in January 2014, ending a 26-year presidency.

Despite these difficulties, Lendoiro is widely credited with raising Deportivo’s profile and establishing the club as a respected force in Spanish and European football. His stewardship remains the benchmark against which subsequent presidents are measured.

**Why Lendoiro Is Considered the Best**

Though Deportivo has had a number of capable presidents, none matched the impact of Lendoiro. He not only delivered unprecedented sporting success but also transformed the club’s identity and ambitions. Football historians, local fans, and former players alike consistently cite his vision, resilience, and leadership as the reasons behind the club’s greatest achievements.

**Conclusion**

As Deportivo La Coruña continues to navigate the challenges of modern football, the legacy of Augusto César Lendoiro endures. His era is remembered as a time when the club dared to dream—and succeeded—against the odds. For supporters and observers, Lendoiro remains, objectively and enduringly, the best president in the club’s storied history.

---

**Sources**  
- https://www.rcdeportivo.es/en/club/club-history  
- https://elpais.com/deportes/2014/01/21/actualidad/1390321519_809019.html  
- https://www.lavozdegalicia.es/not
============================================================

🤖 Human Input Required:
📝 Prompt: Final decision (approve/reject): 
👤 Your input: approve

✅ Result: ✅ ARTICLE APPROVED AFTER REVISION!

Topic: The best president of Real Club Deportivo A coruña
Final Version: 2

============================================================
FINAL ARTICLE:
============================================================
**Headline:**  
Augusto César Lendoiro: Architect of Deportivo La Coruña's Golden Era

**A Coruña, Spain —** Real Club Deportivo de La Coruña, one of Spain's historic football institutions, has seen a number of presidents since its founding in 1906. Yet, among them, Augusto César Lendoiro stands out as the most transformative and celebrated leader, credited with steering the club through its most successful period in history. His presidency, spanning from 1988 to 2014, not only elevated Deportivo onto the national and European stage but also left an indelible legacy in both the city of A Coruña and Spanish football as a whole.

**Who Was Augusto César Lendoiro?**

Born in A Coruña in 1946, Lendoiro began his career outside of football, working as a teacher and later as a politician. His entry into the world of sports administration started with local clubs, but his appointment as president of Deportivo in June 1988 marked a turning point for both himself and the club. At the time, Deportivo languished in Spain’s Segunda División and faced severe financial difficulties.

**A Presidency That Changed Everything**

Lendoiro’s presidency quickly became synonymous with ambition and innovation. He inherited a club on the brink of bankruptcy, but through a series of strategic measures—including renegotiating debts, fostering local support, and attracting new sponsors—he stabilized the club’s finances. His eye for talent and willingness to take calculated risks in the transfer market soon paid off.

In the early 1990s, Lendoiro brought in legendary players such as Bebeto, Mauro Silva, Djalminha, and Fran, the latter a local icon. Under his leadership, the club achieved promotion to La Liga in 1991, setting the stage for a decade of remarkable achievements.

**Deportivo’s Golden Age**

The 1990s and early 2000s are widely regarded as Deportivo’s "Golden Era." The club finished as La Liga runners-up in 1994 and 1995, narrowly missing out on the title in a dramatic 1994 finale. The pinnacle came in the 1999-2000 season when Deportivo, under coach Javier Irureta and Lendoiro’s stewardship, won their first and only La Liga championship. It was a historic achievement for a club from Galicia, traditionally overshadowed by Spain’s larger teams.

During this period, Deportivo also claimed two Copa del Rey titles (1995, 2002) and three Spanish Super Cups (1995, 2000, 2002). On the European front, the team became regular fixtures in the UEFA Champions League, reaching the semi-finals in the 2003-04 season after a memorable comeback against AC Milan.

**Leadership Style and Legacy**

Lendoiro’s management style combined pragmatism with bold vision. He was known for his negotiating skills, often securing high-profile signings for modest fees. His ability to foster a sense of unity among players, staff, and supporters was instrumental in building a competitive and resilient squad.

Beyond silverware, Lendoiro’s legacy includes modernizing the club’s infrastructure, most notably the Riazor Stadium, and investing in youth development. His presidency also cultivated a strong connection between the club and the city of A Coruña, turning Deportivo into a symbol of regional pride.

**Challenges and Departure**

The latter years of Lendoiro’s tenure were marked by financial challenges, as the club struggled to compete with the resources of Spain’s football giants. Relegation battles and mounting debts eventually led to his resignation in January 2014, ending a 26-year presidency.

Despite these difficulties, Lendoiro is widely credited with raising Deportivo’s profile and establishing the club as a respected force in Spanish and European football. His stewardship remains the benchmark against which subsequent presidents are measured.

**Why Lendoiro Is Considered the Best**

Though Deportivo has had a number of capable presidents, none matched the impact of Lendoiro. He not only delivered unprecedented sporting success but also transformed the club’s identity and ambitions. Football historians, local fans, and former players alike consistently cite his vision, resilience, and leadership as the reasons behind the club’s greatest achievements.

**Conclusion**

As Deportivo La Coruña continues to navigate the challenges of modern football, the legacy of Augusto César Lendoiro endures. His era is remembered as a time when the club dared to dream—and succeeded—against the odds. For supporters and observers, Lendoiro remains, objectively and enduringly, the best president in the club’s storied history.

---

**Sources**  
- https://www.rcdeportivo.es/en/club/club-history  
- https://elpais.com/deportes/2014/01/21/actualidad/1390321519_809019.html  
- https://www.lavozdegalicia.es/not
============================================================
(venv) fernando.souto@192 middleware % 
```



