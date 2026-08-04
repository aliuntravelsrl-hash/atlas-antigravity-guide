# KPS — Decisions Log

> Registro de decisiones de implementación tomadas durante el programa.
> Cada decisión incluye: contexto, opciones consideradas, decisión adoptada, evidencia.
> Inmutable una vez registrada — si se revierte, se registra como nueva decisión.

---

## Plantilla

```
## DEC-KPS-NNN — [Título corto]
**Fecha:** YYYY-MM-DD
**Sprint:** N
**Tomada por:** Antigravity / Director / Curator

### Contexto
Qué situación lo motivó.

### Opciones consideradas
1. Opción A
2. Opción B

### Decisión adoptada
Opción X — razón.

### Evidencia
- Link o referencia a archivo en evidence/
```

---

## DEC-KPS-001 — Estructura escalable docs/execution/programs/
**Fecha:** 2026-08-03
**Sprint:** 0 (Bootstrap)
**Tomada por:** Director (recomendación) + computer-perplexity (implementación)

### Contexto
La instrucción original especificaba `docs/execution/KPS/`. Al anticipar 15-20 programas futuros (CRM, Hospitality, Marketing, Intelligence, etc.), un directorio `execution/` plano se volvería inmanejable.

### Opciones consideradas
1. `docs/execution/KPS/` — estructura plana (instrucción original)
2. `docs/execution/programs/KPS/` — con índice general y portfolio (recomendación Director)

### Decisión adoptada
Opción 2 — `docs/execution/programs/KPS/` con `README.md` + `portfolio.md` en el nivel `execution/`. Evita una migración futura cuando el portfolio crezca. Decisión tomada antes del primer commit.

### Evidencia
- `docs/execution/README.md` — índice general
- `docs/execution/portfolio.md` — estado de programas
- Instrucción del Director en sesión 2026-08-03 21:04 UTC-4

---
