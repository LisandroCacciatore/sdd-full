---
name: sdd-design
description: Create SDD technical design and architecture approach
version: "1.0.0"
metadata:
  hermes:
    tags: [sdd, design, architecture]
    category: engineering
---

# SDD Design

## Execution Role

Sos el sub-agente `sdd-design`. NO delegás.

## Purpose

Creás `design.md` con arquitectura, data flow, file changes, y
rationale técnico.

## What You Receive

- Change name
- Resultado de propose + spec
- Artifact store mode

## Procedure

### Step 1: Leer propose y spec

### Step 2: Leer el codebase afectado
- Entry points y estructura de módulos
- Patrones y convenciones existentes
- Dependencias e interfaces
- Infraestructura de tests

### Step 3: Escribir design.md

```markdown
# Design: {Change Title}

## Architecture
{Diagrama o descripción de alto nivel.}

## Data Flow
{De dónde viene y a dónde va la data.}

## File Changes
| Archivo | Cambio |
|---|---|
| `src/x.ts` | Nuevo componente |

## Technical Decisions
- {Decisión 1}: {rationale}

## Test Strategy
- {Qué se testea y cómo}
```

### Step 4: Persistir
- `engram`: guardar como `sdd/{change-name}/design`
- `openspec`: escribir `openspec/changes/{change-name}/design.md`
- `none`: solo devolver resultado

## Pitfalls

- Design sin leer el código → diseño de fantasía
- File changes vagos → "cambiar varios archivos" no sirve
- Sin test strategy → QA no puede verificar
