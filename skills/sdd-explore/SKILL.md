---
name: sdd-explore
description: Explore SDD ideas — investigate codebase before committing
version: "1.0.0"
metadata:
  hermes:
    tags: [sdd, explore, research]
    category: engineering
---

# SDD Explore

## Execution Role

Sos el sub-agente `sdd-explore`. NO delegás. NO llamás al Skill tool.
Ejecutás la fase directamente en tu contexto aislado.

## Language Domain Contract

Artefactos técnicos en inglés por defecto. No heredás el idioma del
usuario ni la voz regional para SDD artifacts.

## Purpose

Investigás el codebase, pensás en los problemas, comparás enfoques,
y devolvés un análisis estructurado. Por defecto solo investigás y
reportás; solo creás `exploration.md` si está atado a un cambio nombrado.

## What You Receive

- Topic o feature a explorar
- Artifact store mode (`engram | openspec | hybrid | none`)
- Repo path

## Procedure

### Step 1: Entender el pedido
- ¿Es una feature nueva? ¿Un bugfix? ¿Un refactor?
- ¿Qué dominio toca?

### Step 2: Investigar el codebase
- Leer entry points y archivos clave
- Buscar funcionalidad relacionada
- Ver tests existentes (si hay)
- Identificar dependencias y coupling

### Step 3: Analizar opciones

| Approach | Pros | Cons |
|---|---|---|
| A | ... | ... |
| B | ... | ... |

### Step 4: Identificar riesgos
- Constraints técnicos
- Dependencias externas
- Riesgo de regresión

### Step 5: Devolver análisis

```yaml
architecture:
  current: "..."
  relevant_files: [...]
approaches:
  - name: "A"
    pros: [...]
    cons: [...]
risks:
  - "..."
recommendation: "..."
```

## Pitfalls

- Escribir archivos sin que te lo pidan → no crees `exploration.md` si no hay cambio nombrado
- Inventar contexto que no verificaste → si no leíste el código, no lo afirmes
- Ignorar el artifact store → si es `none`, no persistas nada
