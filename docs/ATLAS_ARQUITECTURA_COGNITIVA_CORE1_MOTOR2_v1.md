# 🧠 ARQUITECTURA COGNITIVA: Core 1 + Atlas Motor 2
## Lógica de Integración — Mejores Prácticas

**Fecha:** 25 de marzo de 2026  
**Director:** Aldo Hilario — Aliun Travel SRL  
**Auditor/Arquitecto:** Perplexity Computer  
**Fuente:** Auditoría directa del schema Supabase (`oyihiyivdhfxpyiwnmqk`)

---

## 1. EL CONCEPTO CENTRAL

Tienes UN solo inventario de hoteles (la propiedad es la misma), pero DOS formas de venderlo:

```
                    ┌─────────────────────────┐
                    │    BÓVEDA ESTÁTICA       │
                    │  (Extracción anual de    │
                    │   proveedores locales)   │
                    │                          │
                    │  hotels_master (116+)    │
                    │  rooms (habitaciones)    │
                    │  rates (596 tarifas)     │
                    │  seasons (7 temporadas)  │
                    │  gallery / about / etc.  │
                    └────────┬────────┬────────┘
                             │        │
                    ┌────────┘        └────────┐
                    │                          │
            ┌───────▼───────┐          ┌───────▼───────┐
            │   CORE 1      │          │   ATLAS       │
            │  Motor de     │          │   Motor 2     │
            │  Búsqueda     │          │               │
            │               │          │  Estructura A │
            │ /destinos/    │          │  (Contado)    │
            │  punta-cana   │          │               │
            │               │          │  Estructura B │
            │ Buscar hotel  │          │  (Bloques)    │
            │ por fechas    │          │               │
            │ + pax         │          │ /ofertas/     │
            │               │          │  last-minute  │
            │ Precio =      │          │  flash-sale   │
            │ rates +       │          │  paquetes     │
            │ seasons       │          │  early-bird   │
            │ (dinámico)    │          │  grupos       │
            └───────┬───────┘          └───────┬───────┘
                    │                          │
                    └──────────┬───────────────┘
                               │
                    ┌──────────▼──────────┐
                    │  ESCUDO FINANCIERO  │
                    │  (Motor Contable)   │
                    │                     │
                    │  Margen mín. 15%    │
                    │  Fee por método     │
                    │  Costo operativo    │
                    │  Riesgo             │
                    │                     │
                    │  COMPARTIDO por     │
                    │  ambos motores      │
                    └─────────────────────┘
```

---

## 2. PRINCIPIO: "MISMA PROPIEDAD, DIFERENTE VITRINA"

La regla de oro es simple:

> **Los datos del hotel son compartidos. Lo que cambia es cómo se empaqueta y vende.**

| Dato | ¿Compartido? | Dónde vive |
|---|---|---|
| Nombre, ubicación, estrellas | ✅ Compartido | `hotels_master` |
| Galería de fotos | ✅ Compartido | `hotels_master.gallery` / `gallery_urls` |
| "Acerca del hotel" | ✅ Compartido | `hotels_master.description` / `long_description` |
| Restaurantes | ✅ Compartido | `hotels_master.restaurants_data` |
| Servicios / amenidades | ✅ Compartido | `hotels_master.services_data` / `amenities` |
| Habitaciones | ✅ Compartido | `rooms` (tabla relacional) |
| Tarifas base (extracción anual) | ✅ Compartido | `rates` (tabla relacional) |
| Temporadas | ✅ Compartido | `seasons` (tabla relacional) |
| Políticas de cancelación | ✅ Compartido | `hotels_master.cancellation_policy` / `policies` |
| **Precio de venta al público** | ❌ Diferente | Core 1: calculado dinámico / Atlas: `marketing_offers.final_price` |
| **Tipo de oferta** | ❌ Diferente | Atlas: `marketing_offers.offer_type` |
| **Stock / cupos** | ❌ Diferente | Atlas: `offer_date_ranges.stock_total` |
| **Método de pago** | ❌ Diferente | Atlas: `marketing_offers.payment_method` |
| **Descuento negociado** | ❌ Diferente | Atlas: `atlas_offers.additional_discount_*` |
| **Margen / fee / riesgo** | ✅ Misma fórmula | Escudo Financiero (triggers + RPCs) |

---

## 3. DISEÑO DE RELACIONES (Lo más simple posible)

### Modelo actual en Supabase (lo que ya tienes):

```
hotels_master (116+)
  ├── rooms (FK: hotel_id → hotels_master.id)
  │     └── rates (FK: room_id → rooms.id, season_id → seasons.id)
  ├── seasons (⚠️ hotel_id es NULL — son globales)
  │
  ├── [JSONB embebido] gallery, rates, seasons, restaurants, services, policies
  │
  ├── marketing_offers (FK implícita: hotel_slug → hotels_master.slug)
  │     └── offer_date_ranges (FK: offer_id → marketing_offers.id)
  │
  └── atlas_offers (FK implícita: hotel_slug → hotels_master.slug)
        └── atlas_quotes (existe pero vacía)
```

### Modelo objetivo (lo que necesitas):

```
hotels_master (SSOT — Bóveda estática)
  │
  │  Datos compartidos por ambos motores:
  │  name, slug, zone, stars, description, gallery_urls,
  │  restaurants_data, services_data, amenities, policies
  │
  ├── rooms (habitaciones — compartido)
  │     │
  │     └── rates (tarifas base — extracción anual)
  │           │
  │           └── seasons (FK: season_id — temporadas estáticas)
  │
  │  ════════════════════════════════════════════════
  │  CORE 1 (Búsqueda dinámica por fechas + PAX)
  │  ════════════════════════════════════════════════
  │  
  │  Flujo: Usuario busca → RPC consulta rates + seasons
  │         → calcula precio → aplica Escudo Financiero
  │         → muestra resultado → usuario reserva
  │
  │  RPCs: calcular_cotizacion (✅ existe)
  │        consultar_disponibilidad (❌ falta crear)
  │
  │  ════════════════════════════════════════════════
  │  ATLAS Motor 2 (Ofertas empaquetadas)
  │  ════════════════════════════════════════════════
  │
  ├── marketing_offers (vitrina de ofertas)
  │     │  hotel_slug → referencia a hotels_master
  │     │  offer_type: last_minute | flash_sale | paquete | early_bird | grupo
  │     │  original_price ← viene de rates (extracción)
  │     │  final_price ← calculado con descuento
  │     │  base_cost ← costo proveedor (para margen)
  │     │  
  │     │  ⭐ CLAVE: original_price se calcula desde rates
  │     │     NO se inventa. La oferta "descuenta" sobre
  │     │     el precio que ya existe en rates.
  │     │
  │     └── offer_date_ranges (fechas + stock por oferta)
  │           check_in, check_out, nights
  │           stock_total, stock_sold, stock_held
  │
  ├── atlas_offers (ofertas B2B / negociación directa / bloques)
  │     │  hotel_slug → referencia a hotels_master  
  │     │  total_amount ← calculado desde rates + descuento
  │     │  additional_discount_* ← negociación
  │     │
  │     └── atlas_quotes → atlas_quote_items
  │           (cotizaciones formales)
  │
  └── bookings (reservas confirmadas — compartida ambos motores)
        │  ⭐ CLAVE: Una reserva puede originarse de Core 1
        │     O de Atlas. El campo `source` lo diferencia.
        │
        └── source: 'core1_search' | 'atlas_offer' | 'atlas_block' | 'agent_ia'
```

---

## 4. LAS SUB-PAGES DE OFERTAS: CÓMO CONSUMEN DATOS

Las categorías de la landing `/destinos/ofertas`:

| Tab | `offer_type` | Descripción | Datos que necesita |
|---|---|---|---|
| **Todas** | `*` | Todas las ofertas publicadas | `marketing_offers` WHERE `is_published = true` |
| **Last Minute** | `last_minute` | Ofertas de último momento | Mismo filtro + `offer_type = 'last_minute'` |
| **Flash Sale** | `flash_sale` | Descuentos relámpago | Mismo filtro + `offer_type = 'flash_sale'` |
| **Paquetes** | `paquete` | Paquetes todo incluido | Mismo filtro + `offer_type = 'paquete'` |
| **Early Bird** | `early_bird` | Reserva anticipada | Mismo filtro + `offer_type = 'early_bird'` |
| **Grupos** | `grupo` | Ofertas para grupos | Mismo filtro + `offer_type = 'grupo'` |

### Cada tarjeta de oferta necesita mostrar:

```
┌─────────────────────────────────────────┐
│  [FOTO HOTEL] ← hotels_master.gallery   │
│                                         │
│  SEMANA SANTA 2026                      │
│  ← marketing_offers.title               │
│                                         │
│  ⭐⭐⭐⭐⭐ Occidental Caribe              │
│  ← hotels_master.stars + name           │
│                                         │
│  Punta Cana                             │
│  ← hotels_master.zone                   │
│                                         │
│  $980 → $882 USD  (-10%)               │
│  ← marketing_offers.original_price      │
│     marketing_offers.final_price        │
│     marketing_offers.discount_percentage│
│                                         │
│  [Ver Fechas y Reservar]                │
└─────────────────────────────────────────┘
```

### La consulta SQL/Supabase para cada tab:

```javascript
// Consulta optimizada para la landing de ofertas
const { data: offers } = await supabase
  .from('marketing_offers')
  .select(`
    *,
    offer_date_ranges(*),
    hotels_master!inner(
      name, stars, zone, gallery_urls, 
      description, restaurants_data, 
      services_data, amenities, policies
    )
  `)
  .eq('is_published', true)
  .gte('valid_until', new Date().toISOString())
  .eq('offer_type', selectedTab)  // filtro por tab
  .order('created_at', { ascending: false });
```

**Problema actual:** `marketing_offers` usa `hotel_slug` (TEXT) como referencia, no un FK real a `hotels_master.id`. Para que el JOIN funcione, necesitas:

```sql
-- Opción A: Agregar FK por slug (simple)
-- No se puede hacer FK a un campo no-unique directamente.
-- Usar slug como lookup manual en el código.

-- Opción B (RECOMENDADA): Agregar hotel_id a marketing_offers
ALTER TABLE marketing_offers ADD COLUMN hotel_id UUID REFERENCES hotels_master(id);

-- Poblar el hotel_id desde el slug existente:
UPDATE marketing_offers mo
SET hotel_id = hm.id
FROM hotels_master hm
WHERE mo.hotel_slug = hm.slug;

-- Ahora el JOIN funciona nativamente en Supabase:
-- .select('*, hotels_master!hotel_id(*)')
```

---

## 5. EL ESCUDO FINANCIERO: MOTOR CONTABLE COMPARTIDO

Ambos motores deben pasar por la misma validación financiera:

```
                    ┌──────────────────────────────────────┐
                    │        ESCUDO FINANCIERO             │
                    │     (Compartido Core 1 + Atlas)      │
                    │                                      │
                    │  INPUTS:                             │
                    │  ├─ sell_price (precio al cliente)   │
                    │  ├─ base_cost (costo proveedor)      │
                    │  ├─ payment_method (visa/transfer)   │
                    │  ├─ fee_rate (% comisión método)     │
                    │  └─ operational_cost                 │
                    │                                      │
                    │  CÁLCULOS:                           │
                    │  ├─ gross_margin = sell - cost        │
                    │  ├─ payment_fee = sell × fee_rate    │
                    │  ├─ risk_cost = sell × risk_weight   │
                    │  ├─ net_margin = gross - fee - risk  │
                    │  └─ margin_pct = net / sell × 100    │
                    │                                      │
                    │  REGLA DURA:                         │
                    │  ├─ margin_pct >= 15% → ✅ APROBADO  │
                    │  └─ margin_pct <  15% → ❌ BLOQUEADO │
                    │                                      │
                    │  OUTPUTS (se guardan en la tabla     │
                    │  correspondiente):                   │
                    │  ├─ Core 1  → bookings.*             │
                    │  ├─ Atlas A → marketing_offers.*     │
                    │  └─ Atlas B → atlas_offers.*         │
                    └──────────────────────────────────────┘
```

### Implementación como RPC reutilizable:

```sql
-- Motor Contable: Función compartida por Core 1 y Atlas
CREATE OR REPLACE FUNCTION public.calcular_margen(
  p_sell_price NUMERIC,
  p_base_cost NUMERIC,
  p_payment_method TEXT DEFAULT 'transferencia',
  p_operational_cost NUMERIC DEFAULT 5.00
)
RETURNS JSONB AS $$
DECLARE
  v_fee_rate NUMERIC;
  v_gross_margin NUMERIC;
  v_payment_fee NUMERIC;
  v_risk_cost NUMERIC;
  v_net_margin NUMERIC;
  v_margin_pct NUMERIC;
  v_approved BOOLEAN;
BEGIN
  -- Fee según método de pago
  v_fee_rate := CASE p_payment_method
    WHEN 'visa' THEN 0.035       -- 3.5%
    WHEN 'tarjeta' THEN 0.035    -- 3.5%
    WHEN 'paypal' THEN 0.045     -- 4.5%
    WHEN 'transferencia' THEN 0  -- 0%
    WHEN 'efectivo' THEN 0       -- 0%
    ELSE 0.03                    -- default 3%
  END;

  v_gross_margin := p_sell_price - p_base_cost;
  v_payment_fee := p_sell_price * v_fee_rate;
  v_risk_cost := p_sell_price * 0.02; -- 2% riesgo base
  v_net_margin := v_gross_margin - v_payment_fee - v_operational_cost - v_risk_cost;
  v_margin_pct := (v_net_margin / NULLIF(p_sell_price, 0)) * 100;
  v_approved := v_margin_pct >= 15;

  RETURN jsonb_build_object(
    'sell_price', p_sell_price,
    'base_cost', p_base_cost,
    'gross_margin', ROUND(v_gross_margin, 2),
    'payment_fee', ROUND(v_payment_fee, 2),
    'risk_cost', ROUND(v_risk_cost, 2),
    'operational_cost', p_operational_cost,
    'net_margin', ROUND(v_net_margin, 2),
    'margin_percentage', ROUND(v_margin_pct, 2),
    'approved', v_approved,
    'payment_method', p_payment_method,
    'rejection_reason', CASE 
      WHEN NOT v_approved THEN 'Margen neto ' || ROUND(v_margin_pct, 1) || '% es menor al mínimo 15%'
      ELSE NULL
    END
  );
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

---

## 6. FLUJO COGNITIVO COMPLETO

### Core 1: Búsqueda → Cotización → Reserva

```
USUARIO                    WEB                      SUPABASE
  │                         │                          │
  │  Buscar Punta Cana      │                          │
  │  10-13 Abril, 2 adultos │                          │
  │ ─────────────────────── >│                          │
  │                         │  SELECT hotels_master     │
  │                         │  WHERE zone = 'Punta Cana'│
  │                         │  JOIN rooms + rates       │
  │                         │  WHERE fechas match       │
  │                         │ ──────────────────────── >│
  │                         │                          │
  │                         │< ──── hotels + precios ──│
  │                         │                          │
  │  [Lista de hoteles      │                          │
  │   con PRECIOS REALES]   │                          │
  │< ───────────────────────│                          │
  │                         │                          │
  │  Click "Ver detalles"   │                          │
  │ ─────────────────────── >│                          │
  │                         │  SELECT hotels_master     │
  │  [Página de hotel:      │  gallery, about,          │
  │   galería, info,        │  restaurants, rooms,      │
  │   habitaciones,         │  rates para esas fechas   │
  │   precios por room]     │ ──────────────────────── >│
  │< ───────────────────────│< ──── datos completos ───│
  │                         │                          │
  │  Click "Reservar"       │                          │
  │ ─────────────────────── >│                          │
  │                         │  RPC calcular_margen()    │
  │                         │  ¿Margen >= 15%?          │
  │                         │ ──────────────────────── >│
  │                         │< ── ✅ Aprobado ─────────│
  │                         │                          │
  │  [Formulario de datos   │                          │
  │   del viajero]          │                          │
  │  → Nombre, cédula, etc. │                          │
  │ ─────────────────────── >│                          │
  │                         │  INSERT bookings          │
  │                         │  source = 'core1_search'  │
  │                         │ ──────────────────────── >│
  │                         │                          │
  │  [Confirmación +        │  n8n → WhatsApp/Email    │
  │   instrucciones pago]   │  → Voucher               │
  │< ───────────────────────│< ────────────────────────│
```

### Atlas Motor 2: Oferta → Selección → Reserva

```
USUARIO                    WEB                      SUPABASE
  │                         │                          │
  │  /destinos/ofertas      │                          │
  │  Tab: "Last Minute"     │                          │
  │ ─────────────────────── >│                          │
  │                         │  SELECT marketing_offers  │
  │                         │  + offer_date_ranges      │
  │                         │  + hotels_master (JOIN)   │
  │                         │  WHERE is_published       │
  │                         │  AND offer_type           │
  │                         │ ──────────────────────── >│
  │                         │< ── ofertas + hotel info ─│
  │                         │                          │
  │  [Tarjetas con foto     │  ⭐ La FOTO y el ABOUT   │
  │   del hotel, precio,    │  vienen de hotels_master  │
  │   descuento, fechas]    │  (no de marketing_offers) │
  │< ───────────────────────│                          │
  │                         │                          │
  │  Click oferta           │                          │
  │ ─────────────────────── >│                          │
  │                         │  [Detalle: galería,       │
  │                         │   hotel info, precio      │
  │                         │   con descuento,          │
  │                         │   fechas disponibles]     │
  │                         │                          │
  │  Seleccionar fecha      │                          │
  │  Click "Reservar"       │                          │
  │ ─────────────────────── >│                          │
  │                         │  RPC calcular_margen()    │
  │                         │  sell = final_price       │
  │                         │  cost = base_cost         │
  │                         │  method = payment_method  │
  │                         │ ──────────────────────── >│
  │                         │< ── ✅ Aprobado ─────────│
  │                         │                          │
  │  [Formulario viajero]   │                          │
  │ ─────────────────────── >│                          │
  │                         │  INSERT bookings          │
  │                         │  source = 'atlas_offer'   │
  │                         │  UPDATE stock_sold +1     │
  │                         │ ──────────────────────── >│
  │                         │                          │
  │  [Confirmación]         │  n8n → notificación      │
  │< ───────────────────────│< ────────────────────────│
```

---

## 7. RELACIÓN: DATOS ESTÁTICOS vs DATOS DINÁMICOS

```
┌───────────────────────────────────────────────────────────┐
│                  CAPA ESTÁTICA (HOY)                      │
│                  Extracción anual de proveedores           │
│                                                           │
│  hotels_master    → Nombre, zona, estrellas, galería,     │
│                     descripción, restaurantes, servicios, │
│                     amenidades, políticas                 │
│                                                           │
│  rooms            → Tipos de habitación, capacidad,       │
│                     fotos de habitación                   │
│                                                           │
│  rates            → Tarifa base por habitación/año        │
│                     (adult_rate, child_rate, valid_from/to)│
│                                                           │
│  seasons          → Alta, Media, Baja (fechas fijas,      │
│                     multiplicador por temporada)          │
│                                                           │
│  📌 FRECUENCIA: 1 vez/año + ajustes trimestrales         │
│  📌 FUENTE: Contratos directos con hoteles               │
│  📌 SIN API: Datos manuales / extracción masiva          │
└───────────────────────────────────────────────────────────┘
                            │
                            │ AMBOS motores leen de aquí
                            │
┌───────────────────────────────────────────────────────────┐
│                  CAPA DINÁMICA (ATLAS)                    │
│                  Gestión comercial en tiempo real          │
│                                                           │
│  marketing_offers → Ofertas empaquetadas con descuento,   │
│                     tipo (last_minute, flash_sale, etc.),  │
│                     vigencia, stock, método de pago        │
│                                                           │
│  offer_date_ranges→ Fechas específicas + cupos por oferta │
│                                                           │
│  atlas_offers     → Ofertas B2B / bloques negociados      │
│                                                           │
│  📌 FRECUENCIA: Diaria / bajo demanda                    │
│  📌 FUENTE: Director (Admin Atlas)                       │
│  📌 SIN API: Validación manual con proveedor             │
└───────────────────────────────────────────────────────────┘
                            │
                            │ FUTURO: API de proveedores
                            │
┌───────────────────────────────────────────────────────────┐
│                  CAPA API (FUTURO)                        │
│                  Proveedores globales                     │
│                                                           │
│  TBO / OperaHotel / Otros conectores XML/JSON            │
│                                                           │
│  Disponibilidad en tiempo real                           │
│  Tarifas dinámicas                                       │
│  Confirmación instantánea                                │
│                                                           │
│  📌 REQUIERE: Carta de solicitud técnica                 │
│  📌 IMPACTO: rates se actualiza en tiempo real           │
│               seasons puede tener override del proveedor  │
│               stock se valida al momento                  │
└───────────────────────────────────────────────────────────┘
```

---

## 8. ACCIONES CONCRETAS PARA HABILITAR ESTA ARQUITECTURA

### Paso 1: Agregar `hotel_id` (FK real) a `marketing_offers` (5 min)

```sql
-- Agregar columna FK
ALTER TABLE marketing_offers 
ADD COLUMN hotel_id UUID REFERENCES hotels_master(id);

-- Poblar desde slug existente
UPDATE marketing_offers mo
SET hotel_id = hm.id
FROM hotels_master hm
WHERE mo.hotel_slug = hm.slug;

-- Crear índice
CREATE INDEX idx_marketing_offers_hotel_id ON marketing_offers(hotel_id);
```

### Paso 2: Verificar que las páginas de oferta usen datos de `hotels_master` (Horizons)

La consulta de la página de detalle de oferta debe ser:

```javascript
const { data: offer } = await supabase
  .from('marketing_offers')
  .select(`
    *,
    offer_date_ranges(*),
    hotels_master(
      name, slug, stars, zone, 
      description, long_description,
      gallery_urls, gallery,
      restaurants_data, services_data, 
      amenities, policies,
      about_image, hero_video
    )
  `)
  .eq('id', offerId)
  .single();

// Ahora offer.hotels_master tiene todo el "about" del hotel
// Y offer.offer_date_ranges tiene las fechas disponibles
```

### Paso 3: La tabla `bookings` necesita campo `source` (5 min)

```sql
-- Si no existe ya
ALTER TABLE bookings ADD COLUMN IF NOT EXISTS source TEXT DEFAULT 'manual';
-- Valores: 'core1_search', 'atlas_offer', 'atlas_block', 'agent_ia', 'manual'

ALTER TABLE bookings ADD COLUMN IF NOT EXISTS offer_id UUID REFERENCES marketing_offers(id);
ALTER TABLE bookings ADD COLUMN IF NOT EXISTS atlas_offer_id UUID REFERENCES atlas_offers(id);
ALTER TABLE bookings ADD COLUMN IF NOT EXISTS date_range_id UUID REFERENCES offer_date_ranges(id);
```

### Paso 4: Crear RPC `calcular_margen` compartido (ya definido arriba)

### Paso 5: Para Horizons — Instrucciones de integración con la landing

```
Las sub-pages (Last Minute, Flash Sale, Paquetes, Early Bird, Grupos)
son FILTROS sobre la misma tabla marketing_offers.

CADA tarjeta de oferta debe mostrar:
- FOTO → hotels_master.gallery_urls (primer elemento) o provider_image_url
- NOMBRE HOTEL → hotels_master.name
- ESTRELLAS → hotels_master.stars
- ZONA → hotels_master.zone
- TÍTULO OFERTA → marketing_offers.title
- PRECIO → marketing_offers.original_price / final_price / discount_percentage
- STOCK → offer_date_ranges.stock_total - stock_sold

La página de DETALLE de oferta debe mostrar:
- GALERÍA COMPLETA → hotels_master.gallery_urls
- ACERCA DEL HOTEL → hotels_master.description / long_description
- RESTAURANTES → hotels_master.restaurants_data
- SERVICIOS → hotels_master.services_data / amenities
- POLÍTICAS → hotels_master.policies
- FECHAS DISPONIBLES → offer_date_ranges(*)
- PRECIO + DESCUENTO → marketing_offers.*
- BOTÓN RESERVAR → flujo de reserva
```

---

## 9. RESUMEN EJECUTIVO

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  hotels_master + rooms + rates + seasons                │
│  = BÓVEDA ESTÁTICA (una sola fuente de verdad)         │
│                                                         │
│  Core 1 LEE de la bóveda → calcula precio dinámico     │
│  Atlas  LEE de la bóveda → empaqueta en ofertas        │
│                                                         │
│  Ambos ESCRIBEN en bookings (con source diferente)     │
│  Ambos PASAN por calcular_margen (escudo financiero)   │
│                                                         │
│  La web muestra datos de hotels_master en TODAS         │
│  las páginas (galería, about, restaurantes, etc.)       │
│                                                         │
│  Futuro: API de proveedores ACTUALIZA la bóveda        │
│  (rates dinámicos), pero la estructura no cambia        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Complejidad de implementación:** BAJA — La estructura ya existe en Supabase. Solo faltan:
1. FK real (`hotel_id`) en `marketing_offers` ← 5 min SQL
2. Consultas JOIN en los componentes React ← Horizons
3. RPC `calcular_margen` compartido ← 10 min SQL
4. Campo `source` en `bookings` ← 5 min SQL

---

*Arquitectura diseñada sobre el schema real auditado de Supabase*  
*Perplexity Computer — Aliun Travel SRL*  
*25 de marzo de 2026*