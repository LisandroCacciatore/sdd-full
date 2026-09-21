---
name: sdd-propose
description: Create SDD change proposal with intent, scope, approach
version: "1.0.0"
metadata:
  hermes:
    tags: [sdd, propose, proposal]
    category: engineering
---

# SDD Propose

## Execution Role

Sos el sub-agente `sdd-propose`. NO delegás.

## Purpose

Tomás el análisis de exploración y producís `proposal.md` con intent,
scope, y approach.

## What You Receive

- Change name
- Resultado de explore
- Artifact store mode

## Procedure

### Step 1: Leer explore
Si el artifact store es `engram`, leé `sdd/{change-name}/explore`.
Si es `openspec`, leé el filesystem.

### Step 2: Escribir proposal.md

```markdown
# Proposal: {Change Title}

## Intent
{Qué problema resolvemos. Por qué este cambio es necesario.}

## Scope
{Qué entra y qué no entra.}

## Approach
{Enfoque técnico de alto nivel.}

## Risks
{Riesgos identificados y mitigaciones.}
```

### Step 3: Persistir
- `engram`: guardar como `sdd/{change-name}/proposal`
- `openspec`: escribir `openspec/changes/{change-name}/proposal.md`
- `hybrid`: ambos
- `none`: solo devolver resultado

## Pitfalls

- Proposal sin intent claro → el "por qué" es lo más importante
- Scope infinito → acotá explícitamente qué NO entra
- Ignorar riesgos → siempre hay al menos uno
