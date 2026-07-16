# Plan de Implementación: Schema de Validación de Reservas e Historial Híbrido de Auditoría

Este documento detalla la propuesta técnica y el plan de implementación para incorporar un **Schema de Validación de Datos** previo a la emisión de vouchers y un **Flujo Híbrido de Logs de Auditoría** en el motor de reservas (`atlas-admin-v2`).

---

## 🎯 Objetivos
1. **Calidad de Emisión (Cero Errores):** Impedir que se emitan vouchers a clientes si la reserva carece de datos críticos (como el localizador del hotel/proveedor).
2. **Trazabilidad a Prueba de Disputas (Historial Híbrido):** Registrar de forma persistente cada cambio realizado en la reserva tanto en la bitácora interna del expediente (`internal_notes`) como en el monitor global de eventos de la agencia (`logs_operativos` de Supabase).

---

## 🛠️ Propuesta de Implementación

### 1. Schema de Validación de Emisión (Frontend)
Al hacer clic en **"Emitir Voucher"**, el componente validará la reserva contra el siguiente esquema de datos mínimos obligatorios:

| Campo | Restricción / Regla | Razón |
|---|---|---|
| **Localizador del Hotel (`hotel_confirmation_no`)** | No puede ser `null`, vacío o igual a `'PENDIENTE'`. | Evita que el cliente viaje sin el código de confirmación del hotel. |
| **Huésped Titular (`lead_guest_name`)** | Debe estar registrado y tener $\ge 3$ caracteres. | Garantiza la validez del titular del voucher. |
| **Estado de Reserva (`status`)** | Debe ser igual a `'confirmed'`. | Prohíbe emitir vouchers sobre reservas canceladas o temporales. |
| **Estado de Pago (`payment_status`)** | Debe ser `'paid'` o `'partial'` (no `'pending'`). | Protege las finanzas de la agencia impidiendo emitir reservas no cobradas. |

* **Comportamiento ante fallo:** El frontend bloqueará la llamada a n8n, mostrará un Toast destructivo con la lista detallada de errores y guiará al usuario a corregir los datos antes de emitir.

---

### 2. Flujo Híbrido de Logs de Auditoría
Cada vez que el administrador guarde cambios en la reserva (Ficha Cliente o Ficha Proveedor/Estado):

* **A. Bitácora Local (`bookings.internal_notes`):**
  * Se concatenará un log formateado en texto plano al campo `internal_notes` de la reserva (ej. `"\n[16/07 11:14] Modificado por director: Localizador=XYZ, Estado=Confirmada"`).
  * **Beneficio:** Cualquiera que abra el modal de detalle de la reserva verá el historial cronológico completo de cambios en pantalla.
* **B. Auditoría de Misión Control (`logs_operativos`):**
  * Se insertará un registro estructurado en la tabla `logs_operativos` de Supabase con `nivel: 'INFO'`, `evento: 'RESERVA_MODIFICADA'` y un mensaje explicativo.
  * **Beneficio:** El cambio aparecerá automáticamente en la pestaña de sistema de **Mission Control Live** para auditoría y disputas legales.

---

## 📋 Archivos a Modificar
* [AdminBookingsPanel.jsx](file:///c:/Users/Admin/Downloads/-atlas-admin-v2/src/components/admin/AdminBookingsPanel.jsx):
  * Modificar `emitirVoucher` para aplicar la validación del Schema.
  * Modificar `handleSaveSettings` para insertar logs de auditoría al editar proveedor/estados.
  * Modificar `handleSaveClientData` para insertar logs de auditoría al editar el cliente o CRM Link.
