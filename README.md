# sdd-full — Pipeline SDD de extremo a extremo

**Distribución de perfil Hermes que lleva un pedido desde el análisis inicial hasta la implementación verificada, con el ciclo SDD completo: explore → propose → spec → design → QA → implement → verify.**

El orquestador no implementa: **coordina**. Mantiene un hilo delgado y delega todo el trabajo real a sub-agentes especializados, fase por fase, con un artefacto verificable como condición de avance.

---

## Qué resuelve

El problema no es que falten herramientas para escribir código, sino que se puede escribir código antes de saber qué hay que construir. Este pipeline invierte el orden y lo hace ejecutable:

| Fase | Qué produce |
|---|---|
| `sdd-explore` | Intake y descubrimiento del problema real |
| `sdd-propose` | Propuesta de cambio: intención, alcance, impacto |
| `sdd-spec` | Spec con requisitos y escenarios |
| `sdd-design` | Diseño técnico y enfoque de arquitectura |
| QA (`qa-criteria` · `qa-edge-cases` · `qa-risk` · `qa-consistency`) | Auditoría del spec **antes** de implementar |
| `sdd-apply` | Implementación |
| `sdd-verify` | Verificación contra los criterios de aceptación |

El pipeline es **secuencial con checkpoints**: cada fase deja un artefacto que alimenta la siguiente. No se lanzan todas juntas.

## Instalación

```bash
hermes profile install github.com/LisandroCacciatore/sdd-full
```

Variables de entorno requeridas (declaradas en `distribution.yaml`): `HERMES_MODEL` y `DEEPSEEK_API_KEY`.

## Restricciones de runtime — el valor menos obvio del repo

Este perfil documenta, dentro de las propias skills, cuatro restricciones que invalidan supuestos habituales de diseño. Están escritas porque rompieron el pipeline en la práctica:

1. **Toda la delegación es asíncrona.** No existe modo sincrónico: poner `background: false` no vuelve sincrónica la llamada.
2. **Hay que esperar en el turno, fase por fase.** Si el orquestador cierra el turno después de despachar, el proceso dueño muere y los hijos con él; el batch queda en estado desconocido.
3. **Los sub-agentes comparten el filesystem.** No hay worktree aislado, así que nunca se lanzan dos sub-agentes que escriban en paralelo. Los cuatro de QA sí van en paralelo, porque solo leen.
4. **Si el batch no vuelve, se recupera de `state.db`.** Nunca se inventa un resultado.

Esa documentación es la diferencia entre "un pipeline que funciona en la demo" y uno que otro puede instalar.

## Qué incluye

| Componente | Rol |
|---|---|
| `SOUL.md` | Persona del orquestador: coordinador de alto nivel, hilo delgado |
| `skills/sdd-full-orchestrator/` | Coordinación de fases, checkpoints y recuperación de batches |
| `skills/sdd-{explore,propose,spec,design,apply,verify}/` | Las 6 fases del ciclo |
| `skills/qa-{criteria,edge-cases,risk,consistency}/` | Los 4 QA en paralelo sobre el spec |
| `config.yaml` | Delegación: 4 hijos, 40 min por fase, 80 turnos de agente |
| `cron/sdd-pipeline-status.yaml` | Reporte diario 11:00 del estado de los pipelines activos |
| `.no-bundled-skills` | No hereda el catálogo de skills de Hermes: el perfil trae los suyos |

## Relación con spec-qa

`spec-qa` es la auditoría de QA sola, para cuando el spec ya existe. `sdd-full` es el ciclo completo e incluye esos mismos cuatro QA como una fase intermedia. Se pueden usar por separado o encadenados.

## Estado

Distribución en uso y en evolución. Aplica SDD a sí misma: las restricciones de runtime de arriba son el resultado de correr el pipeline y encontrarse con ellas.
