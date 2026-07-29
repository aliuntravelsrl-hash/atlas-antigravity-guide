# MANUAL DE OPERACIÓN Y MAPEO DE VERSIONES — COMMERCIAL INTELLIGENCE WORKSPACE (COS-CRM-v3)
**Código de Componentes:** `PipelineKanban.jsx` | `LeadDetail.jsx` | `BookingOpsPanel.jsx`
**Ruta en Admin:** `/crm/pipeline` | `/reservas`
**Ecosistema:** ATLAS (COS-v3.5)
**Última actualización:** 28 Jul 2026 (Ola 1 - Transición a Workspace)

---

## 📌 1. HISTORIAL DE VERSIONES (MAPEO DE VERSIONES)

| Versión | Fecha | Autor | Cambios Clave | Estado |
|---|---|---|---|---|
| **v2.0** | Legacy | Swarm | Kanban plano con 14 columnas. Scroll horizontal excesivo. Sin validaciones de saltos ni ledger contable. | Deprecado ❌ |
| **v3.0.0** | 28 Jul 2026 | Antigravity | **Commercial Intelligence Workspace:** 4 pestañas × 2 columnas, budget sum en DOP, reglas lógicas `isSkippingStages` y validación de `payment_ledger` para abonos. | **ACTIVO (En producción) ✅** |

---

## ⚙️ 2. LÓGICA DE NEGOCIO Y GUARDAS ARQUITECTÓNICAS (COS-v3.5)

El Workspace Comercial funciona como un dominio de producto del COS, gobernado por reglas lógicas que no dependen exclusivamente de la interfaz de usuario:

### A. Regla de Saltos de Etapa (`isSkippingStages`)
Para asegurar la coherencia del ciclo de venta, se definen niveles de fases:
1. `captacion`: `['nuevo', 'calificado', 'contactado']` (Nivel 1)
2. `comercial`: `['cotizacion_enviada', 'factura_enviada', 'negociando']` (Nivel 2)
3. `financiero`: `['validacion_pago', 'abono_recibido', 'deposito_recibido', 'saldo_pendiente']` (Nivel 3)
4. `operativo`: `['en_fulfillment', 'voucher_enviado', 'completado']` (Nivel 4)

*   **Bloqueo:** No se permite arrastrar un lead a una etapa cuyo nivel sea superior al nivel actual en más de **1 nivel** (ej. de `captacion` directamente a `financiero` u `operativo`).
*   **Excepción:** Se permite retroceder niveles o marcar un lead en etapa `perdido` (Nivel 5) en cualquier momento.

### B. Guarda de Validación Contable
Al intentar arrastrar un lead a las etapas financieras `abono_recibido`, `deposito_recibido` o `validacion_pago`:
1. El sistema verifica que el lead tenga al menos una reserva (`booking`) vinculada.
2. Consulta en tiempo real la tabla `payment_ledger` en Supabase por registros coincidentes con el `booking_id`.
3. Si el ledger no arroja registros de pago conciliados, la transición es revertida de inmediato y se le notifica al usuario en la UI.

---

## 🛠️ 3. PROCEDIMIENTO DE VINCULACIÓN DE RESERVAS (`lead_id`)

Para resolver brechas de sincronización entre el pipeline y el motor de reservas:
1. **Creación Manual:** Al crear una reserva manual en `BookingOpsPanel.jsx` (individual, grupal o de excursión), el sistema asocia de forma consistente el `lead_id` del lead seleccionado o creado en caliente.
2. **Confirmación de Reserva:** Al hacer clic en "Confirmar" en el listado de reservas, si la reserva no cuenta con un `lead_id` (nulo):
   * El sistema busca en caliente un lead por email o teléfono.
   * Si no se encuentra, inicializa un lead en stage `deposito_recibido` en Supabase.
   * Modifica e inyecta la referencia `lead_id` en la reserva de forma persistente.
