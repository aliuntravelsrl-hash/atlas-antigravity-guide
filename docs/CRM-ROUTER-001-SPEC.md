# CRM-ROUTER-001 — Event Router spec
**Fecha:** 27 Jul 2026 | **Status:** READY FOR EXECUTION

## Webhook único de entrada

`POST /webhook/crm-lead-stage-changed` (n8n)

escuchador_crm.py llama aquí con el payload del crm_event_log.

## Router: webhook → handler → agente

```javascript
// Nodo Switch en n8n — routing por webhook field
switch(payload.webhook) {
  case 'WH-1': → WF-HERMES-COMMERCIAL-CALIFICADO
  case 'WH-2': → WF-QA-FOLLOWUP-COTIZACION
  case 'WH-3': → WF-HERMES-COMMERCIAL-OBJECION
  case 'WH-4': → WF-HERMES-FINANZAS-SALDO
  case 'WH-5': → WF-HERMES-OPS-FULFILLMENT
  case 'WH-6': → WF-QA-POSTVENTA
  default:     → logs_operativos (warning sin handler)
}
```

## Handlers por webhook

| WH | Stage trigger | Handler | Acción | Agente |
|----|-------------|---------|--------|--------|
| WH-1 | calificado | WF-HERMES-COMMERCIAL-CALIFICADO | Hermes saluda + registra en crm_activities | Hermes Commercial |
| WH-2 | cotizacion_enviada | WF-QA-FOLLOWUP-COTIZACION | Espera T+2h/24h/48h → si sin resp → WhatsApp | Hermes QA |
| WH-3 | negociando | WF-HERMES-COMMERCIAL-OBJECION | Framework objeción + escalar >48h | Hermes Commercial |
| WH-4 | saldo_pendiente | WF-HERMES-FINANZAS-SALDO | Recordatorio pago + fecha límite | Hermes Finanzas |
| WH-5 | en_fulfillment | WF-HERMES-OPS-FULFILLMENT | Solicitar datos pasajeros + emitir voucher | Hermes Ops |
| WH-6 | completado | WF-QA-POSTVENTA | Review T+2h del viaje | Hermes QA |

## Payload que recibe el router

```json
{
  "event_id": "uuid",
  "event_type": "lead.stage_changed",
  "lead_id": "uuid",
  "old_stage": "cotizacion_enviada",
  "new_stage": "negociando",
  "webhook": "WH-3",
  "occurred_at": "timestamp",
  "source": "escuchador_crm_v2"
}
```

## Primer handler a implementar: WH-2 (CRM-FOLLOWUP-001)

```
cotizacion_enviada → WH-2 → WF-QA-FOLLOWUP-COTIZACION
  ↓
Esperar T+2h
  ↓
¿Hay respuesta en crm_activities del lead?
  ├── SÍ → no hacer nada
  └── NO → WhatsApp contextual
             "Hola [nombre], ¿tuviste oportunidad de revisar la cotización?"
             + crm_activities registro (tipo: followup_automatico)
  ↓
Repetir T+24h → T+48h si sigue sin respuesta
```

## Implementación en n8n

Crear WF: `WF-CRM-EVENT-ROUTER-v1`

Nodos:
1. Webhook trigger: POST /webhook/crm-lead-stage-changed
2. Switch por `body.webhook` → 6 ramas
3. Cada rama: HTTP Request al agente correspondiente O lógica directa
4. Error handler: si webhook desconocido → log warning
5. Mark processed: PATCH crm_event_log status=processed

*ATLAS-TECH · CRM-ROUTER-001 · 27 Jul 2026*
