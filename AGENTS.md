# AGENTS.md — Protocolo de trabajo para agentes de IA

Este documento define cómo debe comportarse cualquier agente de IA que trabaje en este
repositorio: el repositorio de contratos de Vela Bank. Acá no hay producto ni código de servicio:
hay especificaciones que todos los servicios consumen. El objetivo es que el agente **no tenga
rienda suelta**: primero propone, después ejecuta y al final rinde cuentas.

Si una instrucción del usuario contradice este documento, el agente debe detenerse, señalar el
conflicto y pedir una decisión explícita. El humano siempre es el responsable del cambio.

---

## 1. Principios no negociables

1. **Plan primero.** No se edita ningún archivo ni se ejecuta ningún comando sin haber anunciado
   antes el objetivo y el plan.
2. **El plan es una propuesta.** El usuario puede cambiarlo, recortarlo o rechazarlo. El agente no
   defiende el plan: lo ajusta.
3. **Ejecución visible.** Durante el trabajo, el agente dice qué está haciendo, qué archivo está
   tocando y qué comando está lanzando.
4. **Cambios pequeños y revisables.** Un cambio = un propósito. El límite es el de
   `CONTRIBUTING.md` (heredado del repositorio `.github`): **≤ 5 archivos y ≤ 300 líneas por PR**.
   Si el plan lo supera, se divide en slices y el humano revisa uno antes de que el agente empiece
   el siguiente. El agente no puede ampliar ese límite por su cuenta.
5. **Cierre con resumen.** Al terminar, el agente resume cambios, comandos, decisiones, riesgos y
   próximos pasos.
6. **El humano decide.** Las decisiones de arquitectura, compatibilidad, versionado, seguridad y
   datos requieren aprobación humana.
7. **Sin secretos ni PII real.** Nunca leer, escribir, commitear ni registrar credenciales, tokens,
   claves, datos personales reales o dinero real. Los ejemplos usan datos sintéticos.
8. **Sin atribución de IA.** No agregar trailers de IA a los commits. El autor humano es el
   accountable.

---

## 2. Flujo obligatorio

El agente trabaja en cinco fases. No puede saltarse la fase de plan.

### Fase 1 — Contexto

Antes de proponer nada, el agente debe:

- leer `README.md`, el `CONTRIBUTING.md` heredado y este `AGENTS.md`;
- revisar el estado del repo (`git status`, rama actual);
- consultar la documentación de producto en el repositorio
  [`docs`](https://github.com/vela-bank/docs): [ADR-0004](https://github.com/vela-bank/docs/blob/main/docs/adr/0004-contracts-and-integration.md),
  [ADR-0008](https://github.com/vela-bank/docs/blob/main/docs/adr/0008-repo-governance-and-naming.md),
  [mapa de repositorios](https://github.com/vela-bank/docs/blob/main/docs/architecture/repo-map.md)
  y [trabajo multi-repositorio](https://github.com/vela-bank/docs/blob/main/docs/governance/multi-repo-workflow.md);
- identificar qué contratos cambian, qué servicios los consumen y si el cambio es compatible;
- revisar las decisiones pendientes en
  [`docs/decisions-pending.md`](https://github.com/vela-bank/docs/blob/main/docs/decisions-pending.md).

Si falta contexto crítico, lo dice antes de inventar.

### Fase 2 — Plan

El agente presenta un plan con esta estructura:

```markdown
## Plan propuesto

**Objetivo:** ...
**Alcance:** ...
**Fuera de alcance:** ...
**Supuestos:** ...
**Decisiones que necesito:** ...
**Contratos afectados:** ...
**Consumidores afectados:** ...
**Compatibilidad:** compatible | breaking (requiere versión nueva)
**Archivos que voy a tocar:** ...
**Comandos que voy a ejecutar:** ...
**Riesgos:** ...
**Criterios de aceptación:** ...
**Pasos:**
1. ...
```

Reglas del plan:

- Debe ser concreto: nombres de archivos, comandos, resultados esperados.
- Debe indicar qué partes son propuestas y qué partes ya están decididas.
- Si el cambio es breaking, debe proponer la nueva versión y el plan de migración. Sin ADR en
  `docs`, no se ejecuta.
- Si el usuario pidió solo un análisis, el agente **no ejecuta**: termina en el plan.
- Si el usuario pidió ejecutar, el agente puede avanzar después de presentar el plan, pero debe
  detenerse en los puntos de aprobación de la sección 5.
- Todo plan declara los slices y verifica que ninguno supere el límite de tamaño. Si un slice lo
  supera, el plan está incompleto: se divide antes de ejecutar.

### Fase 3 — Ejecución

Durante la ejecución, el agente:

- anuncia cada paso antes de hacerlo;
- informa qué archivo crea, edita o elimina;
- informa el comando exacto que va a ejecutar y por qué;
- reporta el resultado del comando, incluyendo errores y códigos de salida relevantes;
- mantiene el plan actualizado con checkboxes;
- antes de commitear o abrir un PR, mide el diff con `git diff --shortstat`; si supera el límite de
  tamaño, se detiene y propone la división;
- si el plan cambia, se detiene, explica el cambio y pide aprobación.

```text
Paso 2/5 — Añadir la consulta de estado de una transferencia.
Voy a editar: openapi/transfers-v0.yaml
Motivo: el contrato v0 no expone la consulta que necesita el frontend.
```

### Fase 4 — Verificación

Antes de cerrar, el agente debe verificar lo que hizo:

- revisar el diff o los archivos creados;
- validar el YAML y la sintaxis OpenAPI (`spectral`, `oasdiff` o el validador disponible);
- comprobar que los ejemplos no contienen datos reales ni secretos;
- comprobar que no se tocaron archivos fuera del plan;
- comprobar que la documentación relacionada en `docs` sigue siendo válida;
- revisar que los enlaces internos y externos resuelvan.

Si una validación falla, el agente lo informa y propone el siguiente paso. No oculta fallos.

### Fase 5 — Resumen

El cierre debe incluir:

```markdown
## Resumen

**Qué se hizo:** ...
**Archivos creados:** ...
**Archivos modificados:** ...
**Comandos ejecutados y resultado:** ...
**Decisiones tomadas:** ... (con enlace al ADR en `docs`)
**Decisiones pendientes:** ... (con enlace a `docs/decisions-pending.md`)
**Compatibilidad:** ...
**Riesgos o limitaciones:** ...
**Verificación:** ...
**Próximo paso propuesto:** ...
```

El resumen debe ser honesto: si algo quedó incompleto, se dice.

---

## 3. Plantilla de progreso

```text
Paso <n>/<total> — <descripción>
Voy a: <acción>
Archivo/comando: <ruta o comando>
Motivo: <por qué>
```

Las plantillas de plan y de resumen son las de las fases 2 y 5; no se duplican aquí.

---

## 4. Disciplina de archivos y comandos

### Archivos

- Antes de editar: indicar ruta exacta y motivo.
- Después de editar: indicar qué cambió y cuántas líneas aproximadas.
- No editar archivos fuera del plan sin anunciarlo.
- No borrar archivos sin aprobación explícita.
- No sobrescribir trabajo del usuario sin haberlo leído.
- No reformatear archivos completos si el cambio es puntual.
- No cambiar la versión de una especificación sin actualizar su `info.version` y el changelog.
- No romper compatibilidad sin una versión nueva y un plan de migración aprobado.

### Comandos

- Antes de ejecutar: mostrar el comando exacto y el motivo.
- Después de ejecutar: mostrar resultado, errores y código de salida.
- No ejecutar comandos destructivos sin aprobación explícita:
  `rm -rf`, `git reset --hard`, `git clean -fd`, `git push --force`.
- No ejecutar `git commit`, `git push`, apertura de PR ni despliegues si el usuario no lo pidió
  explícitamente.
- No ejecutar comandos que consuman credenciales o generen costos sin aprobación.

---

## 5. Cuándo detenerse y pedir aprobación

El agente debe detenerse antes de continuar cuando:

1. Hay una decisión de arquitectura, compatibilidad o versionado sin resolver.
2. El cambio es breaking y no existe versión nueva ni plan de migración con ADR.
3. El plan original ya no alcanza y hay que ampliar el alcance.
4. Se necesita un comando destructivo o irreversible.
5. Faltan permisos, accesos o información.
6. Hay instrucciones contradictorias entre el usuario, `CONTRIBUTING.md`, este archivo o los ADRs.
7. Un test o validación falla dos veces seguidas por la misma causa.
8. Se detecta un secreto, PII real o una vulnerabilidad.
9. El agente no puede explicar el impacto del cambio.
10. El diff supera el límite de tamaño (≤ 5 archivos, ≤ 300 líneas) o mezcla más de un propósito.
11. Un consumidor quedaría roto y no hay plan de migración.

En esos casos, el agente no improvisa: describe la situación, las opciones y la recomendación.

---

## 6. Decisiones y documentación

Este repositorio no guarda ADRs. La documentación durable vive en el repositorio
[`docs`](https://github.com/vela-bank/docs):

- Toda decisión durable se registra en un ADR de
  [`docs/adr/`](https://github.com/vela-bank/docs/tree/main/docs/adr).
- Toda decisión abierta se registra en
  [`docs/decisions-pending.md`](https://github.com/vela-bank/docs/blob/main/docs/decisions-pending.md).
- Un cambio breaking de contrato requiere un ADR nuevo en `docs` con la versión nueva y el plan de
  migración.
- Si cambia el mapa de repositorios o el flujo de integración, se actualizan
  [`docs/architecture/repo-map.md`](https://github.com/vela-bank/docs/blob/main/docs/architecture/repo-map.md)
  y [`docs/governance/multi-repo-workflow.md`](https://github.com/vela-bank/docs/blob/main/docs/governance/multi-repo-workflow.md).

Si el agente toma una decisión menor durante la ejecución, la informa en el progreso y la deja en
el resumen.

---

## 7. Reglas específicas de este repositorio

- Este repositorio es la **fuente de verdad** de las APIs y los eventos de Vela Bank.
- Las APIs síncronas viven en `openapi/` como OpenAPI 3.1.
- Los eventos viven en `events/` (AsyncAPI/JSON Schema y el sobre común).
- No hay código ejecutable, ni servicios, ni base de datos. No se crea infraestructura acá.
- No se crea una carpeta `docs/` local: los ADRs y las decisiones viven en `vela-bank/docs`.
- Versionado: `/v1` en APIs y `eventType.vN` en eventos; los tags son `v<major>.<minor>.<patch>`.
- Compatibilidad hacia atrás obligatoria dentro de la misma versión. Un cambio breaking requiere
  versión nueva y convivencia temporal.
- Flujo *contracts first*: el contrato se mergea antes que el productor y el consumidor.
- El CI valida `oasdiff` (breaking changes), `markdownlint` y enlaces cuando estén disponibles.
- Los ejemplos usan datos sintéticos; nunca PII real.
- Este archivo complementa al `CONTRIBUTING.md` heredado; no lo reemplaza.

---

## 8. Seguridad

- No leer ni escribir `.env`, secretos, tokens, claves privadas o credenciales.
- No pegar secretos en documentos, issues, PRs, commits ni logs.
- No usar datos personales reales. Solo datos sintéticos.
- No ejecutar código de origen desconocido.
- No agregar dependencias sin justificar y revisar licencia/seguridad.
- No desactivar controles de seguridad para “hacer que funcione”.
- Reportar vulnerabilidades de forma privada, según el `SECURITY.md` heredado.

---

## 9. Definition of Done para un cambio asistido por IA

Un cambio está terminado cuando:

- [ ] El plan fue anunciado y el alcance respetado.
- [ ] Los archivos tocados coinciden con el plan, o el cambio fue anunciado.
- [ ] El contrato valida (YAML/OpenAPI) y no rompe compatibilidad sin versión nueva.
- [ ] Los consumidores afectados están identificados.
- [ ] Las decisiones durables están propuestas como ADR en `docs`.
- [ ] Las decisiones abiertas están en `docs/decisions-pending.md`.
- [ ] Las validaciones disponibles pasan o el fallo está reportado.
- [ ] No hay secretos ni PII real.
- [ ] El resumen final incluye cambios, comandos, decisiones y próximos pasos.
- [ ] El cambio respeta el límite de tamaño (≤ 5 archivos, ≤ 300 líneas) o tiene una excepción
      aprobada por el humano.
- [ ] El humano revisó y aprobó el cambio.

---

## 10. Relación con otros documentos

- [`README.md`](README.md): qué es este repositorio y cómo se usa.
- `CONTRIBUTING.md`: flujo humano de ramas, commits, PRs y releases (heredado de `.github`).
- [`ADR-0004`](https://github.com/vela-bank/docs/blob/main/docs/adr/0004-contracts-and-integration.md):
  contratos e integración.
- [`Mapa de repositorios`](https://github.com/vela-bank/docs/blob/main/docs/architecture/repo-map.md):
  organización multi-repositorio.
- [`Trabajo multi-repositorio`](https://github.com/vela-bank/docs/blob/main/docs/governance/multi-repo-workflow.md):
  orden de merge y compatibilidad.
- [`docs/contracts/`](https://github.com/vela-bank/docs/tree/main/docs/contracts): especificaciones
  v0 mientras se migran a este repositorio.
