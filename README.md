# atlas-antigravity-guide

> **START HERE** — Guía oficial de rehidratación y contexto operativo para Antigravity.

---

## 📋 Índice

| Archivo | Propósito |
|---|---|
| [`CONTEXT.md`](./CONTEXT.md) | Identidad, infraestructura, repos, estado actual y reglas de desarrollo |
| [`ROADMAP_B2B_PAGOS.md`](./ROADMAP_B2B_PAGOS.md) | Integración APIs B2B (Ratehawk, TBO) y pasarelas de pago (AZUL) |
| [`docs/MAPA_GPS.md`](./docs/MAPA_GPS.md) | **SSOT de rutas de código** — qué tocar, qué no, estado operativo por componente |
| [`docs/ATLAS_TASKS_MISSION_CONTROL.md`](./docs/ATLAS_TASKS_MISSION_CONTROL.md) | **Mesa de Tareas** — schema, RPC, reglas de negocio y estado del swarm |
| [`docs/ATLAS_ARQUITECTURA_COGNITIVA_CORE1_MOTOR2_v1.md`](./docs/ATLAS_ARQUITECTURA_COGNITIVA_CORE1_MOTOR2_v1.md) | Blueprint Core 1 + Motor 2: flujos, RPCs, relaciones Supabase |
| [`docs/PLAN_EJECUCION_CORE1_ATLAS_v1.md`](./docs/PLAN_EJECUCION_CORE1_ATLAS_v1.md) | Plan de ejecución con cirugías quirúrgicas de frontend |
| [`docs/WALKTHROUGH_RESCATE_VISUAL_FASE1.md`](./docs/WALKTHROUGH_RESCATE_VISUAL_FASE1.md) | Bitácora y walkthrough del Rescate Visual de la Fase 1 (Norte/Sureste) |
| [`docs/WALKTHROUGH_CONTEXTO_OPERATIVO.md`](./docs/WALKTHROUGH_CONTEXTO_OPERATIVO.md) | Bitácora de sesiones — innovaciones UI, saneamiento Wyndham, selector de resorts |
| [`docs/AUDITORIA_CRM_INTERNO.md`](./docs/AUDITORIA_CRM_INTERNO.md) | CRM Pipeline Kanban — `/crm/pipeline`, tablas, flujos |
| [`docs/AUDITORIA_CRM_DASHBOARD.md`](./docs/AUDITORIA_CRM_DASHBOARD.md) | CRM Dashboard — métricas, recibos PDF, crm_deals |
| [`docs/AUDITORIA_RELACION_CRM_BOOKING.md`](./docs/AUDITORIA_RELACION_CRM_BOOKING.md) | Integridad relacional CRM-Booking — booking_id null en producción |
| [`docs/AUDITORIA_MAPA_BOOKINGS.md`](./docs/AUDITORIA_MAPA_BOOKINGS.md) | Mapa y auditoría del motor de reservas |
| [`docs/MIGRACION_PAGOS_SUPABASE.md`](./docs/MIGRACION_PAGOS_SUPABASE.md) | Migración del esquema de pagos — atlas_payments, SEV0 service_role |
| [`docs/PLAN_SCHEMA_AUDITORIA_BOOKINGS.md`](./docs/PLAN_SCHEMA_AUDITORIA_BOOKINGS.md) | Plan de auditoría del schema de bookings |
| [`docs/OVERVIEW_FRONTEND_V2.md`](./docs/OVERVIEW_FRONTEND_V2.md) | Vista general del frontend v2 |

---

## ⚙️ Reglas de uso

- Leer `CONTEXT.md` **completo** al inicio de cada sesión antes de tocar cualquier archivo.
- Consultar `docs/MAPA_GPS.md` antes de modificar cualquier componente o tabla.
- Para crear o consultar tareas del swarm: ver `docs/ATLAS_TASKS_MISSION_CONTROL.md`.
- Este repo es el **único** contexto válido para Antigravity. Ignorar cualquier otro repo legacy.
- Toda actualización de estado la hace el Director o ATLAS-TECH tras cada sesión.

---

*Mantenido por: Director Aldo Hilario | Aliun Travel SRL | República Dominicana*
*Última actualización de índice: Computer (Perplexity) — 17 JUL 2026*
