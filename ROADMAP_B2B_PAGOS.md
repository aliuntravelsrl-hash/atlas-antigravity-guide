# ROADMAP — Integración B2B y Pasarelas de Pago
**Actualizado:** 13 JUL 2026

## Visión

Transformar Aliun Travel de agencia con catálogo propio a **OTA híbrida** con inventario global en tiempo real, pasarelas de pago locales e internacionales, y motor de comparación multi-proveedor.

**Modelo comercial:** tarifa neta de proveedor + markup propio = precio al cliente.

> [!WARNING]
> **BLOQUEANTE EXTERNO PRINCIPAL (RNC):**
> La contratación e integración de la pasarela **AZUL** y del proveedor prepago **TBO / Tripsnstay** están completamente bloqueadas a la espera de la emisión del RNC oficial de Aliun Travel SRL.
> El único frente B2B que puede ejecutarse de forma inmediata en el Sandbox es **Ratehawk Sandbox** (pago en destino, no requiere pasarela transaccional activa de inmediato).

---

## Estado actual vs destino

| Dimensión | Hoy | Destino |
|---|---|---|
| Inventario | Core1 (13 hoteles con tarifas dinámicas) + Core2 (79 hoteles activos en Supabase) | Core1 + Core2 + APIs B2B (Ratehawk, TBO, GoGlobal) |
| Pago | Transferencia / depósito manual | AZUL (tarjeta) + pago en destino (Ratehawk) |
| Booking | Manual por operador | Automático vía API proveedor |
| Voucher | Gotenberg (soberano) | Gotenberg + localizador del proveedor |

---

## Proveedores identificados

### Pago en destino (cliente paga en el hotel)
| Proveedor | API | Estado |
|---|---|---|
| Ratehawk | REST, bien documentada | **P1 — Iniciar Sandbox** (Único canal activo para pruebas) |

### Prepago en el sistema
| Proveedor | API | Estado |
|---|---|---|
| TBO | REST | *Bloqueado:* Esperando RNC para credenciales |
| GoGlobal | REST | Evaluación |

### Pasarelas de pago locales
| Pasarela | Tipo | Estado |
|---|---|---|
| AZUL | Tarjeta local/internacional RD | *Bloqueado:* Esperando RNC para afiliación comercial |
| Carnet | Tarjeta local RD | Evaluación |

---

## Arquitectura de adaptadores (Desarrollo propio)

```
Web frontend
    ↓
ATLAS Aggregation Layer (n8n + Supabase)
    ↓
┌─────────────────────────┐
│ Adaptador Ratehawk      │
│ Adaptador TBO           │  → Schema único ATLAS
│ Adaptador GoGlobal      │
└─────────────────────────┘
```

**Schema normalizado ATLAS:**
```json
{
  "hotel_id": "string",
  "hotel_name": "string",
  "room_type": "string",
  "board_basis": "all_inclusive | breakfast | ep | map",
  "price_net": 0.00,
  "price_markup": 0.00,
  "price_client": 0.00,
  "currency": "USD",
  "refundable": true,
  "cancellation_policy": {},
  "check_in": "YYYY-MM-DD",
  "check_out": "YYYY-MM-DD",
  "provider": "ratehawk | tbo | goglobal",
  "book_token": "string",
  "payment_type": "prepaid | pay_at_destination"
}
```

---

## Flujo de pago por tipo de reserva

```
Cliente selecciona habitación
    ↓
¿Proveedor admite pago en destino?

  SÍ (Ratehawk)                NO (TBO, etc.) [Bloqueado por RNC]
    ↓                                ↓
  Reserva confirmada           Pasarela AZUL
  Sin cobro ahora                    ↓
  Cliente paga en hotel          Cobro en tarjeta
                                     ↓
                               Book automático al proveedor
```

---

## Roadmap de ejecución

### Fase 0 — Prerrequisitos (Sandbox)
- [x] Obtener documentación API Ratehawk
- [/] Validar credenciales API Ratehawk (sandbox)
- [ ] Solicitar cuenta merchant AZUL (*Bloqueado por RNC*)
- [ ] Solicitar credenciales API TBO (*Bloqueado por RNC*)

### Fase 1 — Ratehawk MVP (Sandbox - En progreso)
- [ ] Construir adaptador `ratehawk-search` en n8n
- [ ] Normalizar respuesta al schema ATLAS unificado
- [ ] RPC `buscar_disponibilidad_api(destino, check_in, check_out, pax)` en Supabase
- [ ] Markup engine: neto + % = precio cliente

### Fase 2 — Pasarela AZUL (Post-RNC)
- [ ] Integrar SDK AZUL en checkout Horizons
- [ ] Flujo: habitación → datos tarjeta → cobro AZUL → book API
- [ ] Prueba E2E con credenciales de prueba de AZUL

---

## Preguntas abiertas para el Director

1. **Markup de Ratehawk:** ¿El porcentaje de markup para pago en destino será fijo o variable según categoría de hotel?
2. **Moneda:** ¿Operamos la API de Ratehawk únicamente en USD o activamos conversión dinámica a DOP para el mercado local utilizando la tabla `exchange_rates` de Supabase?
