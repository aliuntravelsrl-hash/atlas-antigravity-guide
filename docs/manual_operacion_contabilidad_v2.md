# Manual de Operación y Mapeo de Versiones: ATLAS Finance v2.1.3
**Versión:** 2.1.3-FIN-SDD (2026-07-24)  
**Mapeo de Frente:** 
*   **F1-FRONTEND:** Panel Administrativo en `/admin/bookings` ([AdminAccountingPage.jsx](file:///C:/Users/Admin/Downloads/-atlas-admin-v2/src/pages/admin/AdminAccountingPage.jsx)).
*   **F2-DATABASE:** Esquema de Base de Datos y Capa de RPCs Transaccionales en Supabase.
*   **F3-WORKFLOWS:** Flujos de n8n (`WF-ACCOUNTING-AI-v1`, `WF-FACTURA-GRUPAL-EMAIL-v1`, `WF-VOUCHER-INDIVIDUAL-GRUPAL`).

---

## 📌 1. Lógica del Módulo Contable y Desacoplamiento (ATLAS-SDD)

El sistema financiero opera en tres capas aisladas para garantizar la inmutabilidad y auditoría de la **Meta de $200,000 USD**:

```
┌─────────────────────────┐
│     atlas_payments      │  --> Dinero recibido. Tasa de cambio (exchange_rate)
│                         │      histórica congelada en el momento del pago.
└───────────┬─────────────┘
            │
            │ Aprobación por el Director
            ▼
┌─────────────────────────┐
│     payment_ledger      │  --> Ledger contable. Registra aplicaciones totales
│                         │      y parciales inmutables. Clave de idempotencia.
└───────────┬─────────────┘
            │
            │ Reconciliación cruzada
            ▼
┌─────────────────────────┘
│        bookings         │  --> Obligaciones pactadas. Mapeo de total_currency,
│                         │      total_amount_dop y total_amount_usd.
└─────────────────────────┘
```

### A. Detección de Duplicados en el Backend
*   La RPC `atlas_register_payment` normaliza la referencia bancaria eliminando caracteres especiales y calcula el `duplicate_fingerprint` en MD5 utilizando el monto formateado canónicamente a dos decimales.
*   Si el fingerprint existe, se marca en el backend el estado `duplicate_status = 'possible_duplicate'` para alertar en el frontend.

### B. Control Concurrente
*   La RPC de aplicación `atlas_apply_payment_to_booking` implementa el bloqueo `FOR UPDATE` sobre `atlas_payments` para evitar condiciones de carrera cuando se aplican abonos simultáneos al saldo disponible de un mismo pago.

### C. Reversión Contable Compensatoria
*   No se eliminan ni modifican registros en el ledger. Las correcciones se realizan insertando un registro negativo compensatorio en `payment_ledger` vinculado al ID de la transacción original mediante `reversal_of_ledger_id` (`FIN-I-008`).

---

## 📋 2. Desglose de Tareas Pendientes para el Seguimiento

Para el correcto seguimiento de la implementación del Transaction Ledger, se detallan las siguientes subtareas secuenciales asignadas al plan de desarrollo:

*   **[ ] FIN-001-A: Schema + Migraciones SQL**
    *   *Descripción:* Ejecución de tipos ENUM contables y DDLs de base de datos para `bookings`, `atlas_payments` y la creación de `payment_ledger`.
    *   *Estado:* PENDIENTE.
*   **[ ] FIN-001-B: RPCs Transaccionales PostgreSQL**
    *   *Descripción:* Creación de las funciones seguras de base de datos `atlas_register_payment` (con fingerprint), `atlas_approve_payment`, `atlas_reject_payment` y `atlas_apply_payment_to_booking` (con `FOR UPDATE`).
    *   *Estado:* PENDIENTE.
*   **[ ] FIN-001-C: Triggers de Inmutabilidad e Invariantes**
    *   *Descripción:* Implementación de políticas RLS y triggers de Postgres para bloquear ediciones directas a campos monetarios y forzar las invariantes `FIN-I-001` a `FIN-I-009`.
    *   *Estado:* PENDIENTE.
*   **[ ] FIN-001-D: Refactorización del Dashboard Contable (get_accounting_dashboard)**
    *   *Descripción:* Reescribir la RPC del dashboard para consolidar saldos y proyecciones desde el ledger inmutable con tipo de cambio histórico congelado.
    *   *Estado:* PENDIENTE.
*   **[ ] FIN-001-E: Integración n8n y Flujos Idempotentes**
    *   *Descripción:* Ajustar prompt del `WF-ACCOUNTING-AI-v1` y crear tabla `fulfillment_events` para gatillar vouchers del Flujo F una sola vez de forma idempotente.
    *   *Estado:* PENDIENTE.
*   **[ ] FIN-001-F: Verificación de Criterios QA (Hermes-QA)**
    *   *Descripción:* Ejecución del suite de pruebas automatizadas validando las invariantes `QA-C-01` a `QA-C-12`.
    *   *Estado:* PENDIENTE.
*   **[ ] FIN-001-G: Evidencia, Puesta en Marcha y Cierre**
    *   *Descripción:* Pruebas reales con abonos de clientes en producción y certificación final.
    *   *Estado:* PENDIENTE.

---
*ATLAS-FINANCE · Aliun Travel SRL · Julio 2026*
