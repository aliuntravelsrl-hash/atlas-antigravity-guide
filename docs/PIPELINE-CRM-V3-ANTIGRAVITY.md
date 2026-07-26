# PIPELINE CRM v3 — SPEC PARA ANTIGRAVITY
**Versión:** 1.0 | **Fecha:** 26 Jul 2026
**STATUS:** ✅ APPROVED — Computer + ATLAS-TECH + Director
**Componente actual:** `src/components/marketing/PipelineKanban.jsx`
**Ruta:** `/crm/pipeline`

> Backend 100% listo. Solo construir el frontend.

---

## 1. PROBLEMA ACTUAL

```
❌ 14 stages en una sola fila → scroll horizontal
❌ No cabe en el tamaño de pantalla del admin
❌ "cotizado" no mapea a 'cotizacion_enviada' en BD
❌ Columna 'deposito_recibido' no aparece
❌ Valor monetario (budget) no visible por columna
```

---

## 2. DISEÑO — 4 TABS, 2 COLUMNAS c/u

```
┌─────────────────────────────────────────────────────┐
│ [CAPTACIÓN]  [COMERCIAL]  [FINANCIERO]  [OPERATIVO] │
│  tab activa en negrita + línea de color             │
└─────────────────────────────────────────────────────┘

Cada tab = 2 columnas que ocupan 100% del ancho.
Sin scroll horizontal. Sin columnas vacías ocultas.
```

### TAB 1 — CAPTACIÓN
```
┌──────────────────────┬──────────────────────────────┐
│ ENTRANTE             │ CALIFICADO                   │
│ stage: nuevo         │ stage: calificado/contactado │
│ 980 leads            │ N leads                      │
│ —                    │ RD$ valor estimado           │
│ (sin valor — nuevo)  │                              │
│                      │ WEBHOOK WH-1:                │
│                      │ Hermes Commercial activa     │
└──────────────────────┴──────────────────────────────┘

FILOSOFÍA:
  ENTRANTE = llegó al sistema (Chatwoot, web, Meta Ads)
  CALIFICADO = completó E3: destino + fechas + personas
               (≥65% del perfil de reserva)
```

### TAB 2 — COMERCIAL
```
┌──────────────────────┬──────────────────────────────┐
│ COTIZADO             │ NEGOCIANDO                   │
│ stage: cotizacion_   │ stage: negociando            │
│        enviada       │                              │
│ N leads              │ N leads                      │
│ RD$ valor total      │ RD$ valor total              │
│                      │                              │
│ WEBHOOK WH-2:        │ WEBHOOK WH-3:                │
│ QA-followup 24h      │ Vendedor objeción / escalado │
└──────────────────────┴──────────────────────────────┘
```

### TAB 3 — FINANCIERO
```
┌──────────────────────┬──────────────────────────────┐
│ ABONO RECIBIDO       │ SALDO PENDIENTE              │
│ stage: abono_        │ stage: saldo_pendiente       │
│        recibido      │                              │
│ N leads              │ N leads                      │
│ RD$ abonado          │ RD$ saldo pendiente          │
│                      │                              │
│ AUTO: SPEC-003       │ WEBHOOK WH-4:                │
│ payment_ledger > 0   │ Hermes Finanzas cobra        │
└──────────────────────┴──────────────────────────────┘
```

### TAB 4 — OPERATIVO
```
┌──────────────────────┬──────────────────────────────┐
│ EN FULFILLMENT       │ COMPLETADO                   │
│ stage: en_           │ stage: completado            │
│       fulfillment    │                              │
│ N leads              │ N leads                      │
│       + voucher_     │ RD$ total facturado          │
│         enviado      │                              │
│                      │                              │
│ WEBHOOK WH-5:        │ WEBHOOK WH-6:                │
│ Hermes Ops: datos    │ QA post-venta T+2h           │
│ pasajeros + voucher  │                              │
└──────────────────────┴──────────────────────────────┘

PERDIDO → lista separada (botón o tab opcional)
  No ocupa espacio en el kanban principal
```

---

## 3. FUENTES DE DATOS (backend listo ✅)

### Leads por columna
```javascript
// Cada columna hace su propia query
const { data: leads } = await supabase
  .from('crm_leads')
  .select(`
    id, full_name, phone, stage,
    hotel_interest, check_in, check_out,
    adults, children, destination,
    budget_range, source, created_at, stage_updated_at,
    score, score_label,
    bookings(id, booking_reference, total_amount_usd, total_amount_dop)
  `)
  .in('stage', STAGE_CONFIG[tabId][colId].stages)
  .order('stage_updated_at', { ascending: true });
```

### Budget por columna
```javascript
// Suma del valor estimado visible en el header de cada columna
// Fuente: crm_leads.budget_range (en DOP)
// Si no tiene budget_range, usar total de bookings vinculados

const columnValue = leads.reduce((sum, lead) => {
  const budget = lead.budget_range ||
    lead.bookings?.[0]?.total_amount_dop || 0;
  return sum + parseFloat(budget);
}, 0);

// Mostrar en header:
// COTIZADO · 34 leads · RD$ 1,240,000
```

### Configuración de stages por tab/columna
```javascript
const STAGE_CONFIG = {
  captacion: {
    entrante:   { stages: ['nuevo'],               webhook: null,  color: 'slate'  },
    calificado: { stages: ['calificado','contactado'], webhook: 'WH-1', color: 'blue'   }
  },
  comercial: {
    cotizado:   { stages: ['cotizacion_enviada','factura_enviada'], webhook: 'WH-2', color: 'cyan'   },
    negociando: { stages: ['negociando'],           webhook: 'WH-3', color: 'yellow' }
  },
  financiero: {
    abono:      { stages: ['abono_recibido','deposito_recibido','validacion_pago'], webhook: null,  color: 'pink'   },
    saldo:      { stages: ['saldo_pendiente'],      webhook: 'WH-4', color: 'orange' }
  },
  operativo: {
    fulfillment:{ stages: ['en_fulfillment','voucher_enviado'],   webhook: 'WH-5', color: 'violet' },
    completado: { stages: ['completado'],           webhook: 'WH-6', color: 'emerald'}
  }
};
```

---

## 4. TARJETA DE LEAD

```
┌─────────────────────────────────────────┐
│ Rosanna Ogando                🟢 HOT    │ ← score_label
│ 📅 Sep 11-14 · Serenade PC             │
│ 👥 2A · RD$40,000                      │
│ ⏱ 3 días en stage                      │ ← días desde stage_updated_at
│ 📞 +1 849-407-0917                     │
└─────────────────────────────────────────┘
```

```javascript
// Días en stage
const daysInStage = Math.floor(
  (Date.now() - new Date(lead.stage_updated_at)) / 86400000
);

// Badge de alerta si lleva mucho tiempo:
// > 2 días en COTIZADO → ⚠️ amarillo
// > 3 días en NEGOCIANDO → 🔴 rojo (webhook escalamiento)
const stageAlert = getStageAlert(tabId, colId, daysInStage);
```

---

## 5. DRAG AND DROP — REGLAS

```javascript
// Al soltar un lead en una nueva columna:
const handleDrop = async (leadId, targetStage) => {

  // Regla 1: No arrastrar de CAPTACIÓN a FINANCIERO directamente
  if (isSkippingStages(currentStage, targetStage)) {
    toast.error('No puedes saltar etapas comerciales');
    return;
  }

  // Regla 2: ABONO RECIBIDO requiere payment_ledger
  if (targetStage === 'abono_recibido') {
    const { count } = await supabase
      .from('payment_ledger')
      .select('id', { count: 'exact', head: true })
      .eq('booking_id', lead.bookings?.[0]?.id);

    if (!count || count === 0) {
      toast.error('No hay abono registrado en el sistema contable');
      return;
    }
  }

  // Actualizar en BD → trigger dispara crm_event_log + pg_notify
  await supabase
    .from('crm_leads')
    .update({ stage: targetStage, stage_updated_at: new Date().toISOString() })
    .eq('id', leadId);
};
```

---

## 6. HEADER DE COLUMNA

```jsx
<div className="column-header">
  <div className="flex justify-between items-center">
    <span className="font-black uppercase text-xs tracking-wider">
      {column.label}
    </span>
    {column.webhook && (
      <span className="text-[9px] text-slate-500 border border-slate-700 px-1.5 py-0.5 rounded">
        {column.webhook}
      </span>
    )}
  </div>
  <div className="flex gap-3 mt-1 text-xs text-slate-400">
    <span>{leads.length} leads</span>
    {columnValue > 0 && (
      <span className="text-emerald-400">
        RD$ {columnValue.toLocaleString('es-DO')}
      </span>
    )}
  </div>
</div>
```

---

## 7. SELECTOR DE LEAD DETAIL — STAGES V2

En `LeadDetail.jsx` actualizar el select de stage:

```javascript
const STAGE_OPTIONS = [
  // CAPTACIÓN
  { value: 'nuevo',                label: 'Entrante',         group: 'CAPTACIÓN'  },
  { value: 'calificado',           label: 'Calificado',       group: 'CAPTACIÓN'  },
  // COMERCIAL
  { value: 'cotizacion_enviada',   label: 'Cotizado',         group: 'COMERCIAL'  },
  { value: 'factura_enviada',      label: 'Factura Enviada',  group: 'COMERCIAL'  },
  { value: 'negociando',           label: 'Negociando',       group: 'COMERCIAL'  },
  // FINANCIERO
  { value: 'validacion_pago',      label: 'Validando Pago',   group: 'FINANCIERO' },
  { value: 'abono_recibido',       label: 'Abono Recibido',   group: 'FINANCIERO' },
  { value: 'saldo_pendiente',      label: 'Saldo Pendiente',  group: 'FINANCIERO' },
  // OPERATIVO
  { value: 'en_fulfillment',       label: 'En Fulfillment',   group: 'OPERATIVO'  },
  { value: 'voucher_enviado',      label: 'Voucher Enviado',  group: 'OPERATIVO'  },
  { value: 'completado',           label: 'Completado',       group: 'OPERATIVO'  },
  // CIERRE
  { value: 'perdido',              label: 'Perdido',          group: 'CIERRE'     },
];
```

---

## 8. CAMPOS NUEVOS EN LEAD DETAIL

Añadir a `LeadDetail.jsx` los campos nuevos de `crm_leads`:

```jsx
// Sección "Expediente de Reserva"
<Field label="Destino"            value={lead.destination} />
<Field label="Operador Turístico" value={lead.operator_name} />
<Field label="No. Habitaciones"   value={lead.num_rooms} />
<Field label="No. Tour"           value={lead.tour_number} />
<Field label="No. Vuelo"          value={lead.flight_number} />
<Field label="Método de Pago"     value={lead.payment_method_pref} />
<Field label="Razón de Pérdida"   value={lead.loss_reason}
       show={lead.stage === 'perdido'} />

// Timestamps financieros (solo lectura)
<Field label="Abono Recibido"     value={fmtDate(lead.abono_recibido_at)} />
<Field label="Saldo Cobrado"      value={fmtDate(lead.saldo_cobrado_at)} />
<Field label="Voucher Enviado"    value={fmtDate(lead.voucher_enviado_at)} />
```

---

## 9. BOOKING OPS PANEL — FIX CRÍTICO

```javascript
// Al confirmar/crear una reserva → SIEMPRE guardar lead_id
// (esto cierra el CRM-SYNC: 3 reservas sin vincular)

const handleConfirmBooking = async (bookingId, foundLeadId) => {
  if (foundLeadId) {
    await supabase
      .from('bookings')
      .update({ lead_id: foundLeadId })
      .eq('id', bookingId);
    // El trigger trg_sync_booking_to_crm avanza el stage automáticamente
  }
};
```

---

## 10. CRITERIOS DE ACEPTACIÓN QA

```
□ 4 tabs visibles — sin scroll horizontal
□ Cada tab muestra exactamente 2 columnas ocupando 100% ancho
□ Header columna: nombre + N leads + RD$ valor (si > 0)
□ Badge WH-X visible en columnas con webhook
□ Tarjeta de lead: nombre + hotel + fechas + adultos + días en stage
□ Score badge: 🟢 HOT / 🟡 WARM / 🔴 COLD
□ Stage selector en LeadDetail: 12 stages canónicos con grupos
□ Campos nuevos en LeadDetail: destination, operator_name, etc.
□ Drag & drop: bloquea abono_recibido sin payment_ledger
□ Drag & drop: no permite saltar grupos de etapas
□ BookingOpsPanel: guarda lead_id al confirmar reserva
□ Tab CAPTACIÓN muestra los 980 leads en "ENTRANTE"
□ stages 'cotizacion_enviada' Y 'factura_enviada' van en "COTIZADO"
□ stages 'abono_recibido' Y 'deposito_recibido' van en "ABONO"
□ stages 'en_fulfillment' Y 'voucher_enviado' van en "FULFILLMENT"
```

---

## 11. DISEÑO — paleta

```
CAPTACIÓN  → slate/blue   (frío, captando)
COMERCIAL  → cyan/yellow  (activo, negociando)
FINANCIERO → pink/orange  (dinero)
OPERATIVO  → violet/emerald (ejecución/éxito)

Tab activa: línea inferior del color de la fase
Tab inactiva: texto slate-500
```

---

*ATLAS-TECH · Pipeline CRM v3 SPEC · 26 Jul 2026*
*Aprobado: Director Aldo Hilario · Diseño: Computer (Commercial Runtime v1)*
*STATUS: ✅ READY FOR EXECUTION*
