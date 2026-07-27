---
title: "Open Clone Marketplace: an app store for software you actually own"
meta_title: "Open Clone Marketplace: an app store for software you actually own"
description: "A catalog of open-source clone apps that anyone can deploy to their own account in one prompt. Your data, your instance, no vendor lock-in."
date: 2026-07-25T05:00:00Z
image: "images/blog/open-market/clone-market-hero.png"
categories: ["AI"]
author: "Fernando Souto"
tags: ["open-source", "flutter", "firebase", "self-hosting", "ai", "claude"]
draft: false
---

**Audio of the post:**

<audio controls>
  <source src="/audios/clone_market_en.mp3" type="audio/mp3">
  Your browser does not support the audio element.
</audio>


Every useful app eventually asks you to pay a subscription, accept a new privacy
policy, or trust a server you don't control. What if you could take a great app,
deploy your **own** copy of it in a few minutes, and own the data forever — even
if you can't code?

That's the idea behind the **[Open Clone Marketplace](https://clonemarket.org)**:
a catalog of open-source apps *inspired by* well-known products (expense
splitting, kanban boards, habit trackers, shopping lists…) that anybody can
deploy to their own cloud account with a single prompt.

<!--more-->

## The problem: great apps, zero ownership

Self-hosting has always been the answer to vendor lock-in, but it has a brutal
onboarding cost. Even a "simple" Flutter + Firebase app expects you to install a
toolchain, create a backend project, wire up authentication, apply security
rules, build, and deploy. That's a wall most people never climb.

So the marketplace splits the problem in two:

1. **A machine-readable manifest** (`clonefest.json`) that fully describes how to
   deploy an app — tools, accounts, and every ordered step.
2. **An AI agent** (a Claude *skill*) that reads that manifest and walks a
   non-technical user through the whole thing, running the commands it can and
   explaining the clicks it can't.

The result: you press **GET** on a catalog card, Claude opens with the prompt
ready, and ~30 minutes later you have your own instance live at
`https://your-project.web.app`.

## How it fits together

Three moving parts, each intentionally boring:

- **App repos** — each one is a normal open-source GitHub repo with a
  `clonefest.json` at its root. That file is the single source of truth.
- **The catalog** — a static GitHub Pages site that reads an index
  (`apps.json`), fetches each manifest, and renders an store–style front end.
  No backend, no build step, no cost.
- **Two Claude skills** — `clone-deployer` (for people installing an app) and
  `clonefest-generator` (for authors publishing one).

Because the catalog reads manifests at runtime, adding an app is just a pull
request. There's nothing to redeploy.

## Anatomy of a `clonefest.json`

The manifest is a small, validated JSON document. All user-facing text is
bilingual (`en` required, `es` optional), and the deploy steps are typed as
either `shell` (a command the agent can run) or `manual` (something the user
does in a browser console). Here's a trimmed example:

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
  "owner": { "name": "Your Name", "github": "your-user" },
  "repo": "https://github.com/your-user/plan",
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
        "run": "git clone https://github.com/your-user/plan.git && cd plan",
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

The key rule: never ship your own secrets. No API keys, no project IDs — every
person who deploys the app brings their **own** backend. The `flutterfire
configure` step regenerates the config to point at *their* Firebase project.

## Publishing an app in three steps

### 1. Generate the manifest

You can write the JSON by hand from the
[template](https://github.com/ficiverson/open-clone-marketplace/blob/main/clonefest.template.json),
but the fastest path is the `clonefest-generator` skill: it inspects your repo
(stack, owner, git history, README steps) and produces a valid manifest.

```text
You: generate the clonefest for my repo https://github.com/your-user/plan
Claude: [reads pubspec.yaml, firebase.json, README…] → writes clonefest.json
```

### 2. Validate it locally

The schema is published, so validation is a two-liner:

```bash
npm i -g ajv-cli ajv-formats
curl -fsSLO https://clonemarket.org/spec/clonefest.schema.json
ajv validate -s clonefest.schema.json -d clonefest.json --spec=draft2020 -c ajv-formats
```

Commit `clonefest.json` to the root of your repo.

### 3. Open a pull request to the catalog

Add one entry to `apps.json`, pinned to a **commit SHA** so the reviewed
manifest can never change underneath you:

```json
{
  "apps": [
    {
      "id": "plan",
      "manifest": "https://raw.githubusercontent.com/your-user/plan/<commit-sha>/clonefest.json"
    }
  ]
}
```

CI validates the manifest against the schema, checks that the id matches, and
**lints the deploy steps for dangerous commands** (piped `curl | sh`, `sudo`,
`rm -rf`, writes outside the repo). A maintainer then reviews the steps by hand
before merge — because the `clone-deployer` skill executes them on real users'
machines, that human review is non-negotiable.

## Why the manifest matters more than the code

The interesting bet here isn't any single app — it's the format. A
`clonefest.json` turns "deploy this repo" from tribal knowledge buried in a
README into a contract a machine can execute and a human can audit. Once that
contract exists, the same agent deploys *any* app in the catalog, and the
catalog itself is just a list of URLs.

It also keeps everyone honest about ownership. Because the manifest forbids
shipping secrets and the deploy always reconfigures against the user's own
backend, "self-hosted" isn't a marketing word — it's structurally true.

## Try it

The catalog is live at **[clonemarket.org](https://clonemarket.org)** with a
handful of apps to start: expense splitting, kanban, habits and shopping lists —
all Flutter + Firebase, all MIT-licensed.

- **Deploy one:** install the `clone-deployer` skill and press GET.
- **Publish yours:** grab the `clonefest-generator` skill and open a PR.

The apps listed are independent open-source projects inspired by well-known
apps. They are not affiliated with, endorsed by, or connected to the original
products or their trademark owners.
