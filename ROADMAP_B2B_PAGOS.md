# ROADMAP — Integración B2B y Pasarelas de Pago
**Actualizado:** 29 JUN 2026

## Visión

Transformar Aliun Travel de agencia con catálogo propio a **OTA híbrida** con inventario global en tiempo real, pasarelas de pago locales e internacionales, y motor de comparación multi-proveedor.

**Modelo comercial:** tarifa neta de proveedor + markup propio = precio al cliente.

---

## Estado actual vs destino

| Dimensión | Hoy | Destino |
|---|---|---|
| Inventario | Core1 (13 hoteles con tarifas dinámicas) + Core2 (73+ hoteles bloque) | Core1 + Core2 + APIs B2B (Ratehawk, TBO, GoGlobal) |
| Pago | Transferencia / depósito manual | AZUL (tarjeta) + pago en destino (Ratehawk) |
| Booking | Manual por operador | Automático vía API proveedor |
| Voucher | Gotenberg (soberano) | Gotenberg + localizador del proveedor |

---

## Proveedores identificados

### Pago en destino (cliente paga en el hotel)
| Proveedor | API | Estado |
|---|---|---|
| Ratehawk | REST, bien documentada | **P1 — iniciar sandbox** |

### Prepago en el sistema
| Proveedor | API | Estado |
|---|---|---|
| TBO | REST | Pendiente credenciales |
| GoGlobal | REST | Evaluación |
| Travellanda | REST | Evaluación |

### Pasarelas de pago locales
| Pasarela | Tipo | Estado |
|---|---|---|
| AZUL | Tarjeta local/internacional RD | **P1 — solicitar merchant** |
| Carnet | Tarjeta local RD | Evaluación en paralelo |

---

## Arquitectura de adaptadores (Opción B — Desarrollo propio)

```
Web frontend
    ↓
ATLAS Aggregation Layer (n8n + Supabase)
    ↓
┌─────────────────────────┐
│ Adaptador Ratehawk      │
│ Adaptador TBO           │  → Schema único ATLAS
│ Adaptador GoGlobal      │
│ Adaptador Travellanda   │
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

**Recomendación ATLAS-TECH:** Híbrido — Ratehawk directo (P1, ~1 semana) + evaluar middleware para los demás.

---

## Flujo de pago por tipo de reserva

```
Cliente selecciona habitación
    ↓
¿Proveedor admite pago en destino?

  SÍ (Ratehawk)                NO (TBO, GoGlobal, etc.)
    ↓                                ↓
  Reserva confirmada           Pasarela AZUL / Carnet
  Sin cobro ahora                    ↓
  Cliente paga en hotel          Cobro en tarjeta
                                     ↓
                              Book automático al proveedor
```

---

## Roadmap de ejecución

### Fase 0 — Prerequisitos
- [ ] Solicitar credenciales API Ratehawk (sandbox)
- [ ] Solicitar credenciales API TBO (sandbox)
- [ ] Solicitar cuenta merchant AZUL
- [ ] Decidir middleware vs desarrollo propio para TBO + GoGlobal

### Fase 1 — Ratehawk MVP (Semanas 1-2)
- [ ] Adaptador `ratehawk-search` en n8n
- [ ] Normalizar respuesta al schema ATLAS unificado
- [ ] RPC `buscar_disponibilidad_api(destino, check_in, check_out, pax)`
- [ ] Markup engine: neto + % = precio cliente
- [ ] Mostrar resultados Ratehawk en frontend junto a Core2

### Fase 2 — Pasarela AZUL (Semanas 2-3)
- [ ] Integrar SDK AZUL en checkout Horizons
- [ ] Flujo: habitación → datos tarjeta → cobro AZUL → book API
- [ ] Manejo de errores: pago fallido, tarjeta rechazada
- [ ] Prueba E2E con tarjeta de prueba AZUL

### Fase 3 — Room Picker Multi-Proveedor (Semanas 3-4)
- [ ] UI con tabs/cards por proveedor
- [ ] Filtros: board basis, reembolsable vs no reembolsable
- [ ] Badge: PAGO EN DESTINO vs PREPAGO
- [ ] Ordenar por precio más bajo

### Fase 4 — Catálogo Dinámico (Semanas 4-5)
- [ ] Búsqueda por destino retorna hoteles de la API (no solo `hotels_master`)
- [ ] Enriquecer con metadata local si el hotel ya existe en `hotels_master`
- [ ] Cache de resultados en Supabase (TTL 6 horas)

### Fase 5 — TBO + Segundo Proveedor (Semanas 5-6)
- [ ] Adaptador TBO
- [ ] Mostrar TBO en room picker compitiendo con Ratehawk

### Fase 6 — Gestión Post-Reserva (Semanas 6-7)
- [ ] Cancelación vía API del proveedor desde admin
- [ ] Reembolso automático vía AZUL si aplica
- [ ] Voucher con localizador del proveedor (Gotenberg)

---

## Preguntas abiertas para el Director

1. **Markup:** ¿Porcentaje fijo sobre neto o variable por tipo de hotel/destino?
2. **AZUL vs Carnet:** ¿Evaluamos las dos en paralelo?
3. **Middleware:** ¿Quieres que ATLAS-TECH investigue Stays.net / Channex antes de decidir?
4. **Moneda:** ¿Precios en USD solamente o también DOP para clientes locales?
