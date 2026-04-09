---
title: "La babel de los agentes de IA: Skills universales con skills-forge"
meta_title: "La babel de los agentes de IA: Skills universales con skills-forge"
description: "Cómo pasar de copiar y pegar system prompts a una capa de skills universal y ejecutable para agentes de IA."
date: 2026-04-08T05:00:00Z
image: "/images/blog/babel/skills-forge.png"
categories: ["AI"]
language: "es"
author: "Fernando Souto"
tags: ["AI", "Architecture", "MCP", "DevOps", "Python"]
draft: false
---

**Audio del post:**

<audio controls>
  <source src="/audios/babel_es.mp3" type="audio/mp3">
  Tu navegador no soporta el elemento de audio.
</audio>

El panorama de la IA es un mapa fragmentado de jardines amurallados. Creas un Custom GPT para tu equipo de producto, un Gemini Gem para tus investigadores y una skill de Claude Code para tus ingenieros. Cada uno vive en aislamiento — sus propias instrucciones, su propio formato, su propio ciclo de vida. Cuando el conocimiento subyacente cambia, actualizas tres cosas en lugar de una.

**Esto es el equivalente moderno de la Edad Oscura en manuales de instrucciones.** En [skills-forge](https://github.com/ficiverson/skills-forge), creo en un futuro diferente: **una skill, cualquier plataforma.** Una capa de distribución y ejecución universal para competencias de IA — construida con los mismos principios de ingeniería con los que construimos software.

---

## La idea central: skills como binarios distribuibles

El cambio fundamental que introduce skills-forge es tratar las skills de IA de la misma forma en que el software moderno trata los paquetes. Una skill no es un documento que pegas en una interfaz de chat. Es un **binario versionado, validado y distribuible** — un `.skillpack` — que se crea una sola vez, se publica en un registry y se instala en cualquier plataforma.

Esto refleja lo que npm hizo por JavaScript o pip por Python. La diferencia es que el artefacto transporta comportamiento ejecutable de IA en lugar de código.

```
Crear una vez  →  Empaquetar en .skillpack  →  Publicar en registry  →  Instalar en cualquier plataforma
```

El formato es Markdown puro (`SKILL.md`), siguiendo el estándar abierto **Agent Skills**. Cualquier herramienta agéntica que hable ese estándar — Claude Code, Gemini CLI, OpenAI Codex, VS Code Copilot — puede consumirlo directamente. Sin traducción. Sin conversión. El mismo archivo.

---

## Primeros pasos: de cero a instalado

```bash
pip install skills-forge

# 1. Scaffolding
skills-forge create \
  --name python-tdd \
  --category development \
  --description "Usar para TDD con Python. Disparadores: pytest, test-first, red-green-refactor." \
  --emoji 🔴

# 2. Lint — detecta descripciones vagas, violaciones de presupuesto de tokens y referencias rotas
skills-forge lint output_skills/development/python-tdd

# 3. Instalar en todas las herramientas soportadas a la vez
skills-forge install output_skills/development/python-tdd --target all
```

La instalación crea un enlace simbólico a tu archivo fuente. Edita tu `SKILL.md` y el cambio es recogido instantáneamente por cualquier herramienta de agente que apunte a él — sin necesidad de reinstalar.

---

## Por qué esto lo cambia todo para las organizaciones

El verdadero poder de skills-forge no está en la CLI. Está en lo que ocurre cuando la combinas con **registries de skills** — repositorios versionados de binarios `.skillpack` desde los que toda tu organización instala.

Piensa en ello como el registry privado de npm de tu empresa, pero para capacidades de IA.

### Una única fuente de verdad para todos los equipos

Hoy en día, tu equipo de seguridad mantiene un prompt de "Codificación Segura" en Claude. Tu equipo de DevOps tiene una versión ligeramente distinta en Copilot. Tu equipo de QA tiene una tercera en Gemini. Cuando cambia un patrón de vulnerabilidad, tres personas tienen que actualizar tres cosas en tres sitios distintos — y siempre hay alguien que se olvida.

Con un registry de skills, existe un único `secure-coding.skillpack` canónico publicado en `v1.2.0`. Cada desarrollador, en cada herramienta, instala desde la misma fuente:

```bash
skills-forge install https://registry.tuempresa.com/packs/secure-coding-1.2.0.skillpack
```

Cuando el equipo de seguridad publica `v1.3.0`, la actualización está disponible para todos de inmediato. **Cero desviación. Cero inconsistencia.**

### Múltiples registries para distintos niveles de confianza

Las organizaciones raramente operan con un único registry monolítico de paquetes. El mismo modelo aplica a las skills:

- Un **registry público** (como [skill-registry](https://ficiverson.github.io/skill-registry/)) distribuye skills abiertas y mantenidas por la comunidad — herramientas de propósito general para redactar posts en LinkedIn, hacer grooming de sprints o probar APIs.
- Un **registry interno privado** alberga tu conocimiento propietario: el checklist de onboarding que tu equipo de RRHH refinó durante tres años, el framework de revisión de arquitectura que usan tus ingenieros principales, el playbook de respuesta a incidentes que escribió tu equipo de SRE.
- Un **registry por equipo** da autonomía a los squads individuales para publicar e iterar rápidamente sin tocar el catálogo de toda la empresa.

Cada registry es simplemente un repositorio git con un `index.json` y un directorio `packs/`. Skills-forge publica en él e instala desde él usando los mismos comandos, independientemente del registry al que apuntes.

```bash
# Publicar en tu registry privado
skills-forge publish ./ai-eng-evaluator-1.0.0.skillpack \
  --registry ~/code/internal-skill-registry \
  --base-url https://raw.githubusercontent.com/tuorg/skills/main \
  --push

# Instalar desde el registry público
skills-forge install https://raw.githubusercontent.com/ficiverson/skill-registry/main/packs/testing/backend-api-tester-1.0.0.skillpack
```

### Integridad y gobernanza integradas

Cada skillpack publicado lleva un checksum SHA256 en el índice del registry. Antes de la instalación, skills-forge verifica que el binario no ha sido alterado. Esto no es opcional — es el comportamiento por defecto.

```json
{
  "name": "secure-coding",
  "version": "1.2.0",
  "sha256": "a3f9b12c...",
  "platforms": ["claude", "gemini", "codex", "vscode", "agents"],
  "export_formats": ["system-prompt", "gpt-json", "gem-txt", "bedrock-xml", "mcp-server"]
}
```

Los campos `platforms` y `export_formats` indican — a ti y a la interfaz del registry — exactamente qué soporta cada skill. Tu equipo de infraestructura puede construir automatización de instalación con total confianza: si una skill está en el registry, está verificada, versionada y es compatible.

---

## Export: un pack, cinco plataformas

Un `.skillpack` no es solo para herramientas de agente en línea de comandos. El comando `export` lo renderiza en cinco formatos nativos de plataforma desde una única fuente:

```bash
# Para el campo de system-prompt de cualquier interfaz de chat
skills-forge export ./evaluation-1.0.0.skillpack --format system-prompt

# Para Custom GPTs de OpenAI
skills-forge export ./evaluation-1.0.0.skillpack --format gpt-json

# Para Gemini Gems de Google
skills-forge export ./evaluation-1.0.0.skillpack --format gem-txt

# Para agentes de AWS Bedrock
skills-forge export ./evaluation-1.0.0.skillpack --format bedrock-xml

# Un servidor MCP de Python autocontenido — ejecutable al instante con uvx
skills-forge export ./evaluation-1.0.0.skillpack --format mcp-server -o ./exports/
uvx --from "mcp[cli]" mcp run ./exports/evaluation-mcp-server.py
```

Los archivos suplementarios incluidos en tu pack — documentos de referencia, payloads de ejemplo, scripts — se incrustan automáticamente en la exportación. Los autores escriben una vez; el adaptador de formato hace el resto.

### Targets de instalación multiplataforma

Para equipos que mezclan editores y herramientas de agentes, el flag `--target all` instala en todas las rutas soportadas simultáneamente:

| `--target` | Ruta global | Ruta de proyecto |
|------------|-------------|------------------|
| `claude` | `~/.claude/skills/` | `.claude/skills/` |
| `gemini` | `~/.gemini/skills/` | `.gemini/skills/` |
| `codex` | `~/.codex/skills/` | `.codex/skills/` |
| `vscode` | *(no soportado)* | `.github/skills/` |
| `agents` | `~/.agents/skills/` | `.agents/skills/` |

El target `agents` es la opción recomendada para instalaciones a nivel de proyecto en equipos que usan múltiples editores. Cualquier herramienta conforme con agentskills.io escanea `.agents/skills/` automáticamente.

---

## El modelo organizativo en una frase

> El conocimiento colectivo de IA de tu organización vive en un registry versionado y firmado. Cualquier equipo lo instala en segundos. Cualquier plataforma lo consume de forma nativa.

Este es el salto de **gestionar archivos** a **orquestar capacidades** — la misma transición que los equipos de software hicieron cuando adoptaron los gestores de paquetes. La era del agente de un solo prompt ha terminado. El futuro es una capa de skills gobernada, portable y ejecutable que escala con tu organización.

Explora el proyecto: [skills-forge en GitHub](https://github.com/ficiverson/skills-forge)  
Navega por el registry público: [Skill Registry](https://ficiverson.github.io/skill-registry/)
