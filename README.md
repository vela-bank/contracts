# `contracts`

Repositorio de contratos de la organización [Vela Bank](https://github.com/vela-bank).

Es la **fuente de verdad** de las APIs y los eventos del proyecto: define cómo se hablan los
servicios antes de que exista el código que los implementa. La decisión está registrada en el
[ADR-0004](https://github.com/vela-bank/docs/blob/main/docs/adr/0004-contracts-and-integration.md).

## Qué contiene

| Ruta | Contenido |
| --- | --- |
| `openapi/` | Especificaciones OpenAPI 3.1 de las APIs síncronas. |
| `events/` | Esquemas de eventos (AsyncAPI/JSON Schema) y el sobre común. |

Las especificaciones v0 todavía viven en
[`docs/contracts/`](https://github.com/vela-bank/docs/tree/main/docs/contracts) y se migran acá en
el siguiente slice de la tarea 1.4:

- `openapi/core-banking-v0.yaml`: clientes, cuentas, movimientos y endpoints internos.
- `openapi/transfers-v0.yaml`: transferencias con `Idempotency-Key`.
- `events/events-v0.md`: sobre de eventos y tipos versionados.

## Cómo se usa

Flujo *contracts first*:

```text
contracts → productor → consumidor → frontend → gitops → staging → prod-sim
```

1. El cambio de API o de evento se propone acá primero.
2. El CI valida compatibilidad y formato.
3. El productor implementa el contrato y el consumidor genera su cliente desde acá.

## Reglas de compatibilidad

- Agregar campos opcionales o endpoints: compatible.
- Eliminar o renombrar campos, cambiar tipos o semántica: **breaking**.
- Un cambio breaking requiere versión nueva (`/v2`, `eventType.v2`) y un plan de migración con ADR
  en el repositorio `docs`.
- Los eventos solo admiten cambios aditivos dentro de la misma versión.

## Estado

Esqueleto inicial. Todavía no hay CI ni las especificaciones migradas; llegan en los próximos
slices de la tarea 1.4 del
[plan de acción](https://github.com/vela-bank/docs/blob/main/docs/action-plan.md).

## Documentación relacionada

- [ADR-0004 — Contratos e integración](https://github.com/vela-bank/docs/blob/main/docs/adr/0004-contracts-and-integration.md)
- [Mapa de repositorios](https://github.com/vela-bank/docs/blob/main/docs/architecture/repo-map.md)
- [Trabajo multi-repositorio](https://github.com/vela-bank/docs/blob/main/docs/governance/multi-repo-workflow.md)

## Licencia

Apache-2.0. El archivo `LICENSE` se añade en el siguiente slice.
