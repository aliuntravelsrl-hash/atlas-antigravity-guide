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

## 📋 2. Desglose de Tareas de Ejecución (ATLAS-SDD SPEC-002 v1.0.2)

Para la implementación física del modelo financiero y la alineación comercial, se detallan las siguientes tareas y dependencias oficiales del Quality Gate de Hermes-QA:

*   **[x] AUDIT-001: Corrección de Métricas Contables Temporales**
    *   *Descripción:* Solucionar el bug visual que igualaba facturación a caja en React, implementando el cálculo y consolidación monetaria en caliente local (tasa 60.5) en `AdminAccountingPage.jsx`.
    *   *Estado:* COMPLETADO.
*   **[ ] FIN-DB-001: Implementación de Base de Datos Canónica**
    *   *Descripción:* Ejecución de los scripts SQL DDL y RPCs de la SPEC contable base en Supabase (tipos enums, alteración de `bookings`, creación de `atlas_payments` y `payment_ledger`).
    *   *Estado:* PENDIENTE.
*   **[ ] CRM-FIN-001: Contrato de Sincronización Backend**
    *   *Descripción:* Crear el trigger de base de datos en Supabase para sincronizar automáticamente el lead comercial a `'deposito_recibido'` tras una inserción exitosa en `payment_ledger`.
    *   *Estado:* PENDIENTE.
*   **[ ] FIN-UI-001: Dashboard de Caja y Conciliación**
    *   *Descripción:* Diseñar las vistas A (Caja Recibida en `atlas_payments`) y B (Ledger de Aplicación en `payment_ledger`) en React, mostrando saldos y conversiones en base a la tasa histórica inmutable de base de datos.
    *   *Estado:* PENDIENTE.
*   **[ ] CRM-UI-001: Kanban Canónico del CRM**
    *   *Descripción:* Alinear las 7 columnas del Kanban en React a los estados de Supabase y bloquear la transición manual a `'deposito_recibido'`, dejándola en modo de sólo lectura automática de base de datos.
    *   *Estado:* PENDIENTE.

---
*ATLAS-FINANCE · Aliun Travel SRL · Julio 2026*
