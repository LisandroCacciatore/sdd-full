---
name: sdd-full-orchestrator
description: Full SDD pipeline — from request to verified implementation
version: "1.0.0"
metadata:
  hermes:
    tags: [sdd, orchestration, pipeline, full, qa, implement, verify]
    category: engineering
---

# SDD Full Orchestrator

## When to Use

- El usuario pide una feature completa end-to-end
- El usuario quiere el ciclo SDD completo sin cambiar de profile
- El usuario dice "implementá X con SDD completo"
- El usuario quiere llegar de request a PR verificado en un solo flujo

## Runtime constraints (Hermes v0.20.x) — leer antes de actuar

Estos cuatro puntos invalidan varios supuestos del diseño original. Respetalos
o el pipeline se rompe en silencio:

1. **TODA delegación es asíncrona.** No existe modo sincrónico. Los
   `delegate_task(...)` de este skill son **pseudocódigo de intención**, no una
   firma literal: no existe el parámetro `worktree`, y `background` está
   deprecado/ignorado. Poner `background: false` NO vuelve sincrónica la llamada.

2. **Tenés que esperar en el turno, fase por fase.** Si cerrás el turno después
   de despachar, el proceso dueño muere, los children mueren con él y el batch
   queda `state: unknown`. Esto aplica a CADA fase.

3. **Los sub-agentes comparten tu filesystem.** No hay worktree aislado. La
   implementación persiste sola (bien), pero **nunca lances dos sub-agentes que
   escriban en paralelo** — se pisarían. Solo los 4 de QA van en paralelo,
   porque solo leen.

4. **Si el batch no vuelve, recuperalo de `state.db`.** Nunca inventes un
   resultado.

### Cómo esperar un sub-agente (patrón obligatorio)

Después de cada despacho:

- Quedate en el turno. Esperá las señales de progreso (`✓ [N/M] ... (12.3s)`).
- Verificá el estado con `delegate_task action='list'` hasta que no queden
  children vivos.
- Cuando el artefacto de la fase esté **escrito en disco**, ahí sí avanzá.
- Si el mensaje consolidado no entró a la conversación, recuperalo:

```bash
python -c "
import sqlite3, json
db = r'<HERMES_HOME>/profiles/sdd-full/state.db'
con = sqlite3.connect(db)
row = con.execute('SELECT delegation_id, state, result_json FROM async_delegations ORDER BY rowid DESC LIMIT 1').fetchone()
print(row[0], row[1])
print(json.dumps(json.loads(row[2]), indent=2, ensure_ascii=False)[:6000])
"
```

`result_json` trae `{"results": [{"task_index": N, "status": "completed", "summary": "..."}]}`.

## Procedure

### Orden canónico (memorizalo, no lo reordenes)

```
explore → propose → spec → design → QA → implement → verify
```

`design` va **ANTES** del gate de QA (el QA audita el spec *y* el design).
Para scope **medio** se saltea `design`:

```
explore → propose → spec → QA → implement → verify
```

Nunca pongas QA antes de design en scope grande: el QA auditaría un spec sin
su diseño y te va a pedir el `design.md` que todavía no existe.

### Paso 0: Preflight

1. Verificá SDD init (`odd/` o `openspec/`). Si no existe, preguntá si
   inicializarlo antes de seguir.
2. Determiná el artifact store: `engram | openspec | hybrid | none`.
3. Verificá que el repo esté limpio (`git status --short`). Si hay cambios sin
   commitear, avisá al usuario: la implementación va a escribir encima.

### Paso 1: Routing inicial

| Scope | Pipeline |
|---|---|
| Trivial (typo, config) | Directo. No uses SDD |
| Medio (feature acotada) | explore → propose → spec → QA → implement → verify |
| Grande (feature completa) | explore → propose → spec → design → QA → implement → verify |

### Paso 2: Cadena de especificación (SECUENCIAL)

Cada fase espera a la anterior. `explore` alimenta `propose`, `propose` alimenta
`spec`, `spec` alimenta `design`.

| Orden | Sub-agente | Input | Artefacto de salida |
|---|---|---|---|
| 2.1 | `sdd-explore` | topic + repo | análisis (opcional `exploration.md`) |
| 2.2 | `sdd-propose` | resultado de explore | `proposal.md` |
| 2.3 | `sdd-spec` | resultado de propose | `spec.md` |
| 2.4 | `sdd-design` *(solo scope grande)* | propose + spec | `design.md` |

Para cada fase, el `context` del delegate lleva el **Role Prompt** del
sub-agente correspondiente (copialo de su SKILL.md) más el resultado de la fase
anterior. Después de cada despacho: **esperá** (ver patrón obligatorio) y
**verificá que el artefacto existe en disco** antes de la fase siguiente.

**Checkpoint:** si `spec.md` no existe, PARÁS. No sigas a QA sin spec.

### Paso 3: QA gate (PARALELO — 4 lectores)

Lanzá los 4 en un solo batch:

| Sub-agente | Qué audita |
|---|---|
| `qa-criteria` | acceptance criteria testeables/medibles |
| `qa-edge-cases` | edge cases no cubiertos |
| `qa-risk` | riesgos técnicos y de negocio |
| `qa-consistency` | coherencia spec ↔ proposal ↔ design |

Esperá los 4, integrá y asigná veredicto:

| Veredicto | Acción |
|---|---|
| **PASS** | Continuar al Paso 4 |
| **PASS_WITH_CONCERNS** | Si los concerns son altos → Loop 1. Si son medios/bajos → continuar |
| **FAIL** | **Loop 1** |

**Loop 1 (QA → spec → QA), máximo 2 iteraciones:**

```
iteracion = 0
while iteracion < 2:
    re-despachar sdd-spec con el spec actual + los hallazgos de QA
    esperar y verificar que spec.md se reescribió
    re-despachar los 4 QA (paralelo)
    esperar e integrar
    if veredicto in (PASS, PASS_WITH_CONCERNS con concerns medios/bajos):
        break
    iteracion += 1
else:
    PARAR y reportar: "no converge tras 2 iteraciones — intervención humana"
```

Persistí el resultado en `qa-report.md` junto al spec.

### Paso 4: Implementación (SECUENCIAL — escritor único)

Un solo sub-agente `sdd-apply`, y **nadie más escribiendo en paralelo**.

El `context` incluye: el spec completo, el design (si existe), el `qa-report.md`,
y el path del repo. El sub-agente escribe código real y corre tests por batch.

**Checkpoint:** verificá `git status --short` y `git diff --stat`. Si no hay
cambios esperados, PARÁS y reportá.

**Loop 2 (verify → implement → verify), máximo 2 iteraciones** — se evalúa en el
Paso 5 y vuelve acá.

### Paso 5: Verify gate

Un solo sub-agente `sdd-verify`: mapea cada acceptance criterion del spec contra
un test real en la implementación, y corre la suite.

| Veredicto | Condición |
|---|---|
| **PASS** | 100% de criterios cubiertos y todos los tests pasan |
| **FAIL** | Algún criterio sin cobertura, o tests fallando |

**Loop 2:** si FAIL y quedan iteraciones, volvé al Paso 4 pasando los hallazgos
de verify al `sdd-apply`, después re-verify. Máximo 2 iteraciones; si no
converge, PARÁS y reportá.

**Guard:** si verify reporta **>5 hallazgos críticos**, PARÁS y pedí intervención
humana — probablemente el problema está en el spec, no en el código.

Persistí `verify-report.md` junto al spec.

### Paso 6: Reporte final y PR

Si verify PASS:

1. `git diff --stat` para el resumen de cambios.
2. Si el repo tiene convención de PR, prepará el comando (`gh pr create`); no
   pushees sin confirmación del usuario.
3. Reportá:

```
## SDD Full Pipeline: <change-name> — COMPLETADO

### Artefactos generados
- proposal.md · spec.md · design.md (si aplica)
- qa-report.md · verify-report.md

### Pipeline
- Explore: ✓
- Propose: ✓
- Spec: ✓
- Design: ✓ (si aplica)
- QA: PASS_WITH_CONCERNS (0 críticos, N altos resueltos)
- Implement: N archivos, M tests pasando
- Verify: PASS (N/N criterios cubiertos)

### Cambios
| Archivo | Cambio |
|---|---|

### Próximo paso
- Revisar el diff: `git diff`
- Crear el PR: `git push origin <branch>` + `gh pr create`
```

## Comandos disponibles

| Comando | Qué hace |
|---|---|
| `/sdd-full <change>` | Pipeline completo: explore → verify |
| `/sdd-full-status [change]` | Estado del pipeline activo |
| `/sdd-full-resume <change>` | Retoma un pipeline pausado |
| `/sdd-full-abort <change>` | Aborta y deja el estado actual |

## Pitfalls

- Implementar sin QA PASS → el spec no está listo, vas a reescribir
- Verify sin correr tests → verify es una mentira
- Loop infinito sin límite → máximo 2 iteraciones por loop
- **Cerrar el turno después de despachar** → el child muere y perdés el resultado
- **Asumir que `background: false` vuelve sincrónica la delegación** → no existe
- **Lanzar dos escritores en paralelo** → sin worktree aislado, se pisan
- Ignorar checkpoints → los errores se acumulan silenciosamente
- Reportar sin evidencia → cada fase necesita su artefacto en disco

## Verification checklist

- [ ] Preflight completado (SDD init + repo limpio)
- [ ] Explore → Propose → Spec corrieron en orden y en espera
- [ ] `spec.md` existe en disco antes del QA gate
- [ ] QA gate pasó (PASS o PASS_WITH_CONCERNS con concerns menores)
- [ ] `qa-report.md` persistido
- [ ] Implement corrió solo, sin escritores paralelos
- [ ] `git diff --stat` muestra los cambios esperados
- [ ] Verify pasó con todos los criterios cubiertos
- [ ] `verify-report.md` persistido
- [ ] Reporte final con artefactos y diff
