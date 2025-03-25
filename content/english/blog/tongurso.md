---
title: "How I created AI-generated trivia questions"
meta_title: "How I created AI-generated trivia questions"
description: "How I created AI-generated trivia questions"
date: 2025-03-25T05:00:00Z
image: "/images/blog/tongurso/tongurso.jpg"
categories: ["AI"]
author: "Fernando Souto"
tags: ["vibe code", "Agents", "Cursor","LLM", "Langchain"]
draft: false
---

Just last week, an old teammate hit me with the question: "How can I use AI to generate random trivia questions?" At the same time, I was prepping a presentation for my colleagues at DEUS, so I thought—why not turn this into a real example? And boom! The result? An AI-powered trivia generator that effortlessly creates engaging, dynamic questions! What started as a simple inquiry became a full-blown project—challenge accepted, mission accomplished! 

## Project overview
The Trivia Question Generator API is an exciting **FastAPI-based** application that brings trivia to life with AI-powered question generation across various categories! Built with Clean Architecture principles, it ensures a modular, scalable, and maintainable codebase. But here’s the coolest part—both the code and content are AI-generated! I leveraged **Cursor** AI to supercharge development, while **Azure OpenAI** handles question generation and validation, making this project a true fusion of AI and software engineering magic! 

![Frontend](/images/blog/tongurso/api.png)

In the beginning, I kicked things off with a simple **Jupyter Notebook**, allowing me to iterate quickly without worrying about heavy dependencies or deployments. This lightweight approach let me refine the core solution at lightning speed! Once I had a solid foundation, I supercharged the development with **Vibe Coding and Cursor**, transforming it into a well-structured, clean architecture-powered repository. The result? A rock-solid, AI-driven trivia generator built for scalability and maintainability!


## Main features
![Frontend](/images/blog/tongurso/graph.png)
1. **Trivia Question Generation:** Utilizes Azure OpenAI to produce trivia questions and answers.​
2. **Question refelect:** Ensures the quality and accuracy of generated questions.​
3. **Clean Architecture Implementation:** Promotes separation of concerns and scalability.​
4. **Tiny frontend:** Small HTML, CSS and Javascript UI generated with AI
5. **FastAPI:** Use API to connect with the multi agent

## How Vibe Coding with Cursor enhanced development

**Vibe Coding** is next-level development—it's coding in the flow state, where AI becomes your ultimate coding companion! Imagine effortlessly bringing ideas to life, with real-time AI-powered suggestions, refactoring magic, and instant problem-solving. No more getting stuck—just smooth, creative, and lightning-fast development! Whether you're brainstorming, debugging, or refining, **Vibe Coding** turns coding into a seamless, hyper-efficient, and even fun experience. With tools like Cursor, it's like having a genius co-pilot, helping you build cleaner, better, and smarter—at the speed of thought!

After a few thrilling iterations with **Cursor**, the application took shape into a well-structured masterpiece! In the beginning, it was just a simple notebook that you can see in the repository, but as I refined the design and leveraged Cursor AI superpowers, it evolved into a clean, scalable, and powerful architecture. What started as a basic setup quickly transformed into a rock-solid repository—all thanks to the magic of AI-driven development!
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

## Conclusion
The Trivia Question Generator API is a perfect showcase of how Clean Architecture and modern development tools can come together to build something powerful! By integrating **Azure OpenAI** and harnessing the magic of **Vibe Coding with Cursor**, this project balances both functionality and maintainability like a pro.

One quick pro tip about Vibe Coding—while it’s an absolute game-changer, if you’re not careful, you might find yourself in an AI loop, overcomplicating things and creating a messy structure. So stay mindful, keep control, and enjoy this next-level way of coding! Embrace the AI, but steer the ship!

You can also check out the AI-generated web version of the project—built automatically by an AI-powered frontend! It's like magic, but real. Go explore and have fun! Enjoy!

![Frontend](/images/blog/tongurso/frontend.png)


For more details, visit the [**GitHub repo - ficiverson/trivia-quiz-ai-clean**](https://github.com/ficiverson/trivia-quiz-ai-clean)