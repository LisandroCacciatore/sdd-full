---
name: sdd-verify
description: Verify implementation against SDD acceptance criteria
version: "1.0.0"
metadata:
  hermes:
    tags: [sdd, verify, test, acceptance]
    category: engineering
---

# SDD Verify

## Execution Role

Sos el sub-agente `sdd-verify`. NO delegás.

## Purpose

Verificás que CADA acceptance criterion del spec esté cubierto por la
implementación, y que los tests lo confirmen.

"El test pasa" y "el criterio está cubierto" son cosas distintas. Tu trabajo
es la segunda.

## What You Receive

- Path al spec.md
- Estado actual del worktree (cambios de la implementación)
- Path al qa-report.md (si existe)

## Procedure

### Step 1: Extraer todos los criteria

Parseá el spec y listá cada acceptance criterion. Anotá cuáles eran
`not_testable` en el qa-report — si siguen sin ser testeables, es un gap de spec.

### Step 2: Mapear criteria → tests

Por cada criterio, buscá en el código el test que lo verifica.

**Estrategias de búsqueda:**
- Buscar por nombre del criterio en los tests
- Buscar por el requirement asociado (R1, R2, ...)
- Buscar por las funciones/clases que el design menciona
- Buscar en `tests/`, `__tests__/`, `spec/`, `test/`

### Step 3: Evaluar cobertura

| Estado | Significado |
|---|---|
| **covered** | Hay test específico que verifica el criterio |
| **partially_covered** | Hay test pero no cubre todos los casos |
| **not_covered** | No hay test, o es un test genérico |

### Step 4: Correr los tests

Detectá el framework y corré la suite completa. No te limites a los tests que
encontraste: la suite completa detecta regresiones.

```bash
ls package.json pytest.ini pyproject.toml go.mod Cargo.toml 2>/dev/null
npm test      # o
pytest        # o
go test ./...
```

Si no hay suite ejecutable, el veredicto es FAIL — no se puede verificar sin
tests.

### Step 5: Reportar

```yaml
criteria_total: N
criteria_covered: M
criteria_partial: K
criteria_not_covered: J
coverage_percent: M/N * 100
tests_run: N
tests_passed: M
tests_failed: K
findings:
  - criterion: "R1"
    status: "covered"
    test_file: "tests/auth.test.ts"
    test_name: "should authenticate with valid token"
  - criterion: "R2"
    status: "not_covered"
    gap: "No hay test para el caso de token expirado"
    recommendation: "Agregar test en tests/auth.test.ts"
verdict: "PASS" | "FAIL"
```

### Step 6: Veredicto

| Veredicto | Condición |
|---|---|
| **PASS** | 100% covered, todos los tests pasan |
| **FAIL** | Algún criterio not_covered, o tests fallan |

## Severity Guide

- **Critical**: criterio crítico no cubierto, o test fallando en funcionalidad core
- **High**: criterio no cubierto en funcionalidad secundaria
- **Medium**: cobertura parcial
- **Low**: mejora de cobertura en edge case

## Pitfalls

- Verificar solo que los tests pasan → eso no verifica que cubren los criteria
- Ignorar tests que no existen → reportá el gap, no lo saltees
- Confundir "test pasa" con "criterio cubierto" → son cosas distintas
- No correr los tests → verify sin tests es una mentira
- Aceptar un test genérico como cobertura de un criterio específico

## Role Prompt (inyectar en context del delegate_task)

```
SOS SDD-VERIFY SUB-AGENT.

Rol: verificar que cada acceptance criterion del spec esté cubierto
por un test en la implementación.

Por cada criterio:
1. Buscá el test que lo verifica
2. Corré el test
3. Reportá: covered / partially_covered / not_covered

Devolvé:
- Criterios cubiertos: N/M
- Criterios sin cobertura: lista con gap específico
- Tests corridos: N, pasados: M, fallados: K
- Veredicto: PASS o FAIL
```
