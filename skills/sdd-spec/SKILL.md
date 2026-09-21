---
name: sdd-spec
description: Write SDD specifications with requirements and scenarios
version: "1.0.0"
metadata:
  hermes:
    tags: [sdd, spec, requirements]
    category: engineering
---

# SDD Spec

## Execution Role

Sos el sub-agente `sdd-spec`. NO delegás.

## Purpose

Escribís `spec.md` con requirements y scenarios testeables.

## What You Receive

- Change name
- Resultado de propose
- Artifact store mode

## Procedure

### Step 1: Leer propose
Leé el proposal para entender intent y scope.

### Step 2: Escribir spec.md

```markdown
# Spec: {Change Title}

## Requirements

### R1: {nombre}
**Descripción:** ...
**Acceptance Criteria:**
- [ ] criterio testeable 1
- [ ] criterio testeable 2

### R2: {nombre}
...

## Scenarios

### S1: {nombre}
**Given** ...
**When** ...
**Then** ...

## Edge Cases
- {edge case 1}
- {edge case 2}
```

### Step 3: Persistir
- `engram`: guardar como `sdd/{change-name}/spec`
- `openspec`: escribir `openspec/changes/{change-name}/specs/spec.md`
- `none`: solo devolver resultado

## Pitfalls

- Acceptance criteria no testeables → "funciona bien" no es criterio
- Ignorar edge cases → QA los va a encontrar
- Requirements vagos → el spec es un contrato, no una idea
