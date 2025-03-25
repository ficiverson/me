---
title: "Cómo creé un trivial generado por IA"
meta_title: "Cómo creé un trivial generado por IA"
description: "Cómo creé un trivial generado por IA"
date: 2025-03-25T05:00:00Z
image: "/images/blog/tongurso/tongurso.jpg"
categories: ["AI"]
author: "Fernando Souto"
tags: ["vibe code", "Agents", "Cursor","LLM", "Langchain"]
draft: false
---

La semana pasada, un antiguo compañero me lanzó la pregunta: "¿Cómo puedo usar IA para generar preguntas de trivial aleatorias?" Justo en ese momento, estaba preparando una presentación para mis colegas en DEUS, así que pensé: ¿por qué no convertir esto en un ejemplo real? ¡Y boom! El resultado: un generador de trivial impulsado por IA que crea preguntas dinámicas y atractivas de manera automática. Lo que comenzó como una simple pregunta, se convirtió en un proyecto completo. ¡Desafío aceptado, misión cumplida!

## Descripción del proyecto
El TrivIAl es una aplicación basada en **FastAPI** con preguntas generadas por IA en múltiples categorías. Construida con principios de Clean Architecture, garantiza un código modular, escalable y fácil de mantener. Pero aquí está la mejor parte. ¡Tanto el código como el contenido son generados por IA! Usé **Cursor AI** para potenciar el desarrollo, mientras que **Azure OpenAI** se encarga de la generación y validación de preguntas, fusionando lo mejor de la IA y la ingeniería de software en un solo proyecto.

![Frontend](/images/blog/tongurso/api.png)

Al principio, comencé con un simple **Jupyter Notebook**, lo que me permitió iterar rápidamente sin preocuparme por dependencias pesadas o despliegues. Este enfoque ligero me ayudó a refinar la solución central a la velocidad de la luz. Una vez que tuve una base sólida, llevé el desarrollo al siguiente nivel con **Vibe Coding y Cursor**, transformándolo en un repositorio bien estructurado con Clean Architecture. ¿El resultado? Un generador de trivial impulsado por IA, robusto, escalable y fácil de mantener.


## Características principales
![Frontend](/images/blog/tongurso/graph.png)
1. **Generación de preguntas de trivial:** Utiliza Azure OpenAI para crear preguntas y respuestas de trivia. 
2. **Reflexión sobre las preguntas:** Garantiza la calidad y precisión de las preguntas generadas.
3. **Implementación de Clean Architecture:** Promueve la separación de responsabilidades y la escalabilidad. 
4. **Pequeño frontend:** Interfaz ligera en HTML, CSS y JavaScript generada con IA.
5. **FastAPI:** Usa una API para conectarse con el sistema multiagente.

## Cómo Vibe Coding con Cursor mejoró el desarrollo

**Vibe Coding** es desarrollo de otro nivel: es programar en estado de flujo, donde la IA se convierte en tu mejor compañero de código. Imagina dar vida a tus ideas sin esfuerzo, con sugerencias en tiempo real, refactorización mágica y soluciones instantáneas. ¡Adiós a los bloqueos! Aquí todo fluye de manera rápida, creativa y sin fricciones. Ya sea que estés brainstormeando, depurando o refinando código, **Vibe Coding** convierte la programación en una experiencia hipereficiente, fluida y hasta divertida. Con herramientas como **Cursor**, es como tener un copiloto genial que te ayuda a escribir código más limpio, mejor y más inteligente a la velocidad del pensamiento.

Después de algunas iteraciones emocionantes con **Cursor**, la aplicación tomó forma y se convirtió en una obra maestra bien estructurada. Al principio, todo comenzó con un simple notebook, que aún puedes ver en el repositorio. Pero a medida que fui refinando el diseño y aprovechando los superpoderes de **Cursor AI**, evolucionó en una arquitectura limpia, escalable y potente. Lo que comenzó como una configuración básica rápidamente se transformó en un repositorio sólido, ¡Todo gracias a la magia del desarrollo impulsado por IA!
```python
app/
├── domain/
│   ├── di/           # Dependency injection
│   ├── entities/     # Domain entities
│   ├── interfaces/   # Abstract interfaces
│   └── use_cases/    # Business logic
├── infrastructure/
│   └── repositories/ # Implementation of interfaces
└── presentation/
    ├── api/          # FastAPI routes
    └── entities/     # API request/response models
```

## Conclusión
TrivIAl es una muestra perfecta de cómo Clean Architecture y las herramientas de desarrollo modernas pueden unirse para crear algo poderoso. Al integrar **Azure OpenAI** y aprovechar la magia de **Vibe Coding** con Cursor, este proyecto equilibra a la perfección tanto la funcionalidad como la mantenibilidad, ¡como un verdadero profesional!

Un consejo rápido sobre **Vibe Coding**: aunque es un cambio radical en el desarrollo, si no tienes cuidado, podrías terminar atrapado en un bucle con la IA, complicando demasiado las cosas y creando una estructura desordenada. Así que mantén el control, disfruta de esta manera de programar de siguiente nivel, ¡y navega con rumbo! Abraza la IA, pero tú diriges el barco.

También puedes echar un vistazo a la versión web generada por IA del proyecto, ¡creada automáticamente por un frontend impulsado por IA! Es como magia, pero real. Ve a explorar y diviértete. ¡Disfruta! 

![Frontend](/images/blog/tongurso/frontend.png)


Para más detalles, visita el [**GitHub repo - ficiverson/trivia-quiz-ai-clean**](https://github.com/ficiverson/trivia-quiz-ai-clean)