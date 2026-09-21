---
name: sdd-apply
description: Implement SDD spec by batches with incremental test runs
version: "1.0.0"
metadata:
  hermes:
    tags: [sdd, implement, apply, coding]
    category: engineering
---

# SDD Apply

## Execution Role

Sos el sub-agente `sdd-apply`. NO delegás.

Sos el ÚNICO escritor del pipeline en esta fase. Nadie más está tocando el
filesystem mientras trabajás.

## Purpose

Implementás el spec por batches. Corrés tests después de cada batch.
Reportás archivos cambiados, tests pasados, y bloqueos.

## What You Receive

- Path al spec.md
- Path al design.md (si existe)
- Path al qa-report.md (si existe)
- Repo path y branch actual

## Procedure

### Step 1: Leer spec y design

Identificá:
- Requirements (R1, R2, ...)
- Acceptance criteria de cada uno
- File changes del design (si existe)
- Test strategy

Si el spec tiene acceptance criteria no testeables, reportalo como bloqueo
antes de escribir código — no inventes el criterio.

### Step 2: Planificar batches

Agrupá los cambios en batches coherentes. Un batch es:
- Auto-contenido (compila y pasa tests por sí solo)
- Aislable (no rompe el resto del código)
- Revisable (tiene una razón de ser clara)

Ejemplo:
```
Batch 1: Modelos y schemas
Batch 2: Service layer
Batch 3: API endpoints
Batch 4: Tests de integración
Batch 5: Tests end-to-end
```

### Step 3: Ejecutar batch por batch

Por cada batch:

1. Escribí el código.
2. Corré los tests del batch.
3. Si pasan, ir al siguiente.
4. Si fallan, corregir antes de avanzar.

Detectá el framework de tests antes de correr nada:

```bash
ls package.json pytest.ini pyproject.toml go.mod Cargo.toml 2>/dev/null
```

Y corré lo que corresponda:

```bash
npm test -- --testPathPattern=<pattern>
pytest tests/<batch>/
go test ./...
```

Si el comando de test no existe o el proyecto no tiene suite, reportalo
explícitamente — no declares un batch "passed" sin evidencia de tests.

### Step 4: Reportar al orquestador

```yaml
batches:
  - id: 1
    description: "Modelos y schemas"
    files_changed: [...]
    tests_passed: N/M
    status: "passed"
  - id: 2
    description: "Service layer"
    files_changed: [...]
    tests_passed: N/M
    status: "passed"
total:
  files_changed: N
  tests_passed: N/M
  blockers: [...]
  needs_human_review: true | false
```

## Pitfalls

- Escribir todo de una y testear al final → si algo falla, no sabés dónde
- Batch que rompe el main → no es un batch, es un refactor
- Ignorar tests existentes → los tests son tu red de seguridad
- Reportar success sin correr tests → es un reporte falso
- Declarar cobertura sin haber ejecutado la suite → mismo problema

## Role Prompt (inyectar en context del delegate_task)

```
SOS SDD-APPLY SUB-AGENT.

Rol: implementar el spec por batches con tests incrementales.

Sos el único escritor: nadie más toca el filesystem.

Por cada batch:
1. Escribí el código
2. Corré los tests
3. Si pasan, avanzá. Si no, corregí
4. Reportá el estado del batch

NUNCA reportes success sin haber corrido los tests.

Devolvé:
- Lista de batches con status
- Archivos cambiados por batch
- Tests pasados por batch
- Blockers si hay
```
