# KPS — Kernel Projection System
## Execution Workspace · Expediente KPS-EXP-001-v3

---

| Campo | Valor |
|-------|-------|
| Código de programa | `KPS` |
| Expediente | `KPS-EXP-001-v3` |
| Orden de autorización | `UI-OVR-002` |
| Estado | **ACTIVO — Sprint 1 en curso** |
| Ejecutor | Antigravity |
| Sandbox | `/mission-next` (a crear en Sprint 1) |
| Régimen | Architectural Freeze · Relocate, not Rewrite |
| Director | ALIUN TRAVELSRL ✅ AUTORIZADO |
| Curator Office | ATLAS-TECH ✅ CERTIFICADO |
| Antigravity | ⬜ PENDIENTE acuse de recibo |
| Fecha de emisión | 2026-08-03 |

---

## Objeto

Rediseño de la capa de presentación del COS bajo la denominación **Kernel Projection System (KPS)**.
Dentro del COS, el término "UI" queda sustituido por KPS. El término "UI" continúa existiendo fuera del COS.

## Principio rector

> **Relocate, not Rewrite** — los componentes certificados se mueven, nunca se reescriben.

## Plan de 7 Sprints

| Sprint | Nombre | Estado |
|--------|--------|--------|
| 1 | Esqueleto de Proyección — Layout Agnóstico | 🔵 EN CURSO |
| 2 | Reubicación de Componentes + Consumidor OVR | ⬜ Pendiente |
| 3 | Navegación Dimensional Dinámica | ⬜ Pendiente |
| 4 | Sincronización Memoria Operativa ⚠️ | ⬜ Requiere aprobación Director |
| 5 | Validación comparativa | ⬜ Pendiente |
| 6 | Certificación Curator | ⬜ Pendiente |
| 7 | Switch controlado a producción | ⬜ Pendiente |

## Restricciones constitucionales

- **R1** — `sync_debt` es metadato previsto, autoridad no disponible hasta `PROMOTION-CONSEQUENCE-STUDY-001`
- **R2** — Projection Runtime debe representar cadenas parciales/ausentes (`complete|partial|missing_evidence|not_yet_instrumented`)
- **R3** — Sprint 1 crea el Sandbox `/mission-next` desde cero — no hay infraestructura previa
- **R4** — Política Single Active Editor per Artifact — ver `INFRA-WF-LOCK-001`

## Lo que Antigravity NO puede hacer

- Modificar tablas de Supabase directamente
- Modificar workflows activos en n8n
- Escribir o modificar el Projection Model (solo consumirlo)
- Tocar producción antes del Switch del Sprint 7
- Iniciar Sprint 4 sin aprobación explícita del Director

## Archivos de este workspace

| Archivo | Uso |
|---------|-----|
| `EXECUTION_LOG.md` | Bitácora diaria — una entrada por sesión de trabajo |
| `DECISIONS.md` | Decisiones de implementación con contexto y evidencia |
| `OBSERVATIONS.md` | Observaciones técnicas locales al programa KPS |
| `MILESTONES.md` | Hitos de Sprint — completado/pendiente/bloqueado |
| `evidence/` | Artefactos verificables: capturas, JSONs, logs |

## Referencias cruzadas

| Ref | Descripción |
|-----|-------------|
| `OBS-021` | Sync Debt — commit c32b17b2 |
| `OBS-022` | Riesgo concurrencia WF n8n — commit 7ac153ab |
| `OBS-023` | 5 capacidades cognitivas / plan 3 carriles — commit be8eeb20 |
| `INFRA-WF-LOCK-001` | Protocolo Single Active Editor |
| `PROMOTION-CONSEQUENCE-STUDY-001` | Prerequisito autoridad sync_debt |
| `KPS-FOUNDATION-001` | Nacimiento oficial KPS — `UI-OVR-002` |

---
*Creado: 2026-08-03 · computer-perplexity · Autorizado por UI-OVR-002*
