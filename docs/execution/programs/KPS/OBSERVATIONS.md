# KPS — Observations

> Observaciones técnicas locales al programa KPS.
> Diferente de las OBS del Curator (atlas-curator-office/OBSERVATIONS.md) —
> estas son observaciones de implementación, no de arquitectura institucional.
> Si una observación local escala a nivel ecosistema, el Curator la promueve como OBS-NNN.

---

## Plantilla

```
## KOBS-NNN — [Título corto]
**Fecha:** YYYY-MM-DD · **Sprint:** N · **Observado por:** nombre
**Severidad:** info | warn | critica

### Observación
Descripción concreta y verificable.

### Impacto en KPS
Cómo afecta al programa.

### Acción
Ninguna / Ver DEC-KPS-NNN / Escalar a Curator
```

---

## KOBS-001 — sync_debt no disponible como autoridad
**Fecha:** 2026-08-03 · **Sprint:** 0 · **Observado por:** ATLAS-TECH
**Severidad:** warn

### Observación
El campo `sync_debt` está definido en el Projection Model (UI-OVR-002) pero su autoridad operativa (`sync_obligations`, `sync_debt` en Supabase) aún no existe. Prerequisito: `PROMOTION-CONSEQUENCE-STUDY-001`.

### Impacto en KPS
Sprints 1–3: `sync_debt` se trata como campo opcional (`null`). Sprint 4 bloqueado hasta autoridad certificada.

### Acción
Restricción R1 incorporada en UI-OVR-002. Sin acción adicional hasta resultado de `PROMOTION-CONSEQUENCE-STUDY-001`.

---

## KOBS-002 — Cadenas de trazabilidad parciales deben proyectarse explícitamente
**Fecha:** 2026-08-03 · **Sprint:** 0 · **Observado por:** ATLAS-TECH
**Severidad:** info

### Observación
El Projection Runtime no debe ocultar cadenas de resolución incompletas. Estados válidos: `complete`, `partial`, `missing_evidence`, `not_yet_instrumented`.

### Impacto en KPS
Sprint 2: el consumidor del contrato OVR debe implementar los 4 estados de cadena visualmente. Es un mecanismo de observación institucional, no un bug a corregir.

### Acción
Incorporado como Restricción R2 en UI-OVR-002. Antigravity implementa en Sprint 2.

---
