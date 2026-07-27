---
title: "Open Clone Marketplace: una tienda de apps que de verdad son tuyas"
meta_title: "Open Clone Marketplace: una tienda de apps que de verdad son tuyas"
description: "Un catálogo de apps clónicas open source que cualquiera puede desplegar en su propia cuenta con un solo prompt. Tus datos, tu instancia, sin dependencia de proveedores."
date: 2026-07-25T05:00:00Z
image: "images/blog/open-market/clone-market-hero-es.png"
categories: ["AI"]
author: "Fernando Souto"
tags: ["open-source", "flutter", "firebase", "self-hosting", "ia", "claude"]
draft: false
---

**Audio del post:**

<audio controls>
  <source src="/audios/clone_market_es.mp3" type="audio/mp3">
  Your browser does not support the audio element.
</audio>


Toda app útil acaba pidiéndote una suscripción, cambiándote la política de
privacidad o metiendo tus datos en un servidor que no controlas. ¿Y si pudieras
coger una buena app, desplegar **tu propia** copia en unos minutos y ser dueño
de tus datos para siempre… aunque no sepas programar?

Esa es la idea detrás del **[Open Clone Marketplace](https://clonemarket.org)**:
un catálogo de apps open source *inspiradas en* productos conocidos (división de
gastos, tableros kanban, seguimiento de hábitos, listas de la compra…) que
cualquiera puede desplegar en su propia cuenta cloud con un único prompt.

<!--more-->

## El problema: grandes apps, cero propiedad

El autoalojamiento (self-hosting) siempre ha sido la respuesta a la dependencia
de proveedores, pero tiene un coste de arranque brutal. Incluso una app "simple"
de Flutter + Firebase te pide instalar un toolchain, crear un proyecto de
backend, configurar la autenticación, aplicar reglas de seguridad, compilar y
desplegar. Es un muro que la mayoría nunca escala.

Así que el marketplace parte el problema en dos:

1. **Un manifest legible por máquinas** (`clonefest.json`) que describe por
   completo cómo desplegar una app: herramientas, cuentas y cada paso en orden.
2. **Un agente de IA** (una *skill* de Claude) que lee ese manifest y guía a un
   usuario no técnico por todo el proceso, ejecutando los comandos que puede y
   explicando los clics que no.

El resultado: pulsas **OBTENER** en una ficha del catálogo, Claude se abre con el
prompt listo, y unos 30 minutos después tienes tu propia instancia online en
`https://tu-proyecto.web.app`.

## Cómo encaja todo

Tres piezas, cada una intencionadamente aburrida:

- **Repos de apps** — cada uno es un repo open source normal de GitHub con un
  `clonefest.json` en la raíz. Ese archivo es la única fuente de verdad.
- **El catálogo** — un sitio estático en GitHub Pages que lee un índice
  (`apps.json`), descarga cada manifest y pinta un front-end estilo tienda de apps.
  Sin backend, sin build, sin coste.
- **Dos skills de Claude** — `clone-deployer` (para quien instala una app) y
  `clonefest-generator` (para quien publica una).

Como el catálogo lee los manifests en tiempo de ejecución, añadir una app es
solo un pull request. No hay nada que volver a desplegar.

## Anatomía de un `clonefest.json`

El manifest es un documento JSON pequeño y validado. Todo el texto de cara al
usuario es bilingüe (`en` obligatorio, `es` opcional), y los pasos de despliegue
son de tipo `shell` (un comando que el agente puede ejecutar) o `manual` (algo
que el usuario hace en una consola del navegador). Un ejemplo recortado:

```json
{
  "$schema": "https://clonemarket.org/spec/clonefest.schema.json",
  "clonefest": "1.0",
  "id": "plan",
  "name": "Plan",
  "clone_of": "Kanban boards",
  "category": "productivity",
  "tagline": {
    "en": "Kanban boards for your team. Your data, your Firebase.",
    "es": "Tableros kanban para tu equipo. Tus datos, tu Firebase."
  },
  "owner": { "name": "Tu Nombre", "github": "tu-usuario" },
  "repo": "https://github.com/tu-usuario/plan",
  "license": "MIT",
  "platforms": ["web", "android", "ios"],
  "stack": {
    "frontend": { "framework": "flutter", "language": "dart" },
    "backend": { "provider": "firebase", "services": ["auth", "firestore", "hosting"] }
  },
  "deploy": {
    "target": "firebase-hosting",
    "estimated_minutes": 30,
    "steps": [
      { "id": "clone", "type": "shell",
        "run": "git clone https://github.com/tu-usuario/plan.git && cd plan",
        "title": { "en": "Clone the repository", "es": "Clona el repositorio" } },
      { "id": "firebase-project", "type": "manual",
        "url": "https://console.firebase.google.com",
        "title": { "en": "Create a Firebase project and enable Auth + Firestore",
                   "es": "Crea un proyecto Firebase y activa Auth + Firestore" } },
      { "id": "deploy", "type": "shell", "run": "firebase deploy --only hosting",
        "title": { "en": "Deploy to Firebase Hosting", "es": "Despliega a Firebase Hosting" } }
    ],
    "verify": {
      "en": "Open your web.app URL, create a board and drag a card between columns.",
      "es": "Abre tu URL web.app, crea un tablero y arrastra una tarjeta entre columnas."
    }
  }
}
```

La regla clave: nunca incluyas tus propios secretos. Ni claves de API ni IDs de
proyecto — cada persona que despliega la app aporta su **propio** backend. El
paso `flutterfire configure` regenera la configuración para que apunte a *su*
proyecto de Firebase.

## Publicar una app en tres pasos

### 1. Genera el manifest

Puedes escribir el JSON a mano a partir de la
[plantilla](https://github.com/ficiverson/open-clone-marketplace/blob/main/clonefest.template.json),
pero el camino más rápido es la skill `clonefest-generator`: inspecciona tu repo
(stack, autor, historial de git, pasos del README) y produce un manifest válido.

```text
Tú: genera el clonefest de mi repo https://github.com/tu-usuario/plan
Claude: [lee pubspec.yaml, firebase.json, README…] → escribe clonefest.json
```

### 2. Valídalo en local

El schema está publicado, así que validar son dos líneas:

```bash
npm i -g ajv-cli ajv-formats
curl -fsSLO https://clonemarket.org/spec/clonefest.schema.json
ajv validate -s clonefest.schema.json -d clonefest.json --spec=draft2020 -c ajv-formats
```

Haz commit del `clonefest.json` en la raíz de tu repo.

### 3. Abre un pull request al catálogo

Añade una entrada a `apps.json`, pineada a un **commit SHA** para que el manifest
revisado no pueda cambiar por debajo:

```json
{
  "apps": [
    {
      "id": "plan",
      "manifest": "https://raw.githubusercontent.com/tu-usuario/plan/<commit-sha>/clonefest.json"
    }
  ]
}
```

La CI valida el manifest contra el schema, comprueba que el id coincide y
**lintea los pasos de despliegue en busca de comandos peligrosos** (`curl | sh`,
`sudo`, `rm -rf`, escrituras fuera del repo). Después, una persona revisa los
pasos a mano antes del merge: como la skill `clone-deployer` los ejecuta en las
máquinas de usuarios reales, esa revisión humana es innegociable.

## Por qué el manifest importa más que el código

La apuesta interesante aquí no es ninguna app concreta, es el formato. Un
`clonefest.json` convierte el "despliega este repo" —conocimiento tribal
enterrado en un README— en un contrato que una máquina puede ejecutar y una
persona puede auditar. Una vez que ese contrato existe, el mismo agente despliega
*cualquier* app del catálogo, y el catálogo en sí es solo una lista de URLs.

Además mantiene a todos honestos sobre la propiedad. Como el manifest prohíbe
incluir secretos y el despliegue siempre reconfigura contra el backend del
usuario, "autoalojado" no es una palabra de marketing: es cierto por
construcción.

## Pruébalo

El catálogo está online en **[clonemarket.org](https://clonemarket.org)** con un
puñado de apps para empezar: división de gastos, kanban, hábitos y listas de la
compra — todas Flutter + Firebase, todas con licencia MIT.

- **Despliega una:** instala la skill `clone-deployer` y pulsa OBTENER.
- **Publica la tuya:** coge la skill `clonefest-generator` y abre un PR.

Las apps listadas son proyectos independientes open source inspirados en apps
conocidas. No están afiliadas, respaldadas ni conectadas con los productos
originales ni con sus marcas.
