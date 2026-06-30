# 📋 PLAN DE EJECUCIÓN: Quién Hace Qué
## Basado en ATLAS_ARQUITECTURA_COGNITIVA_CORE1_MOTOR2_v1.md

**Fecha:** 26 de marzo de 2026  
**Referencia:** [Arquitectura Cognitiva v1](https://github.com/aliuntravelsrl-hash/aliun_agente/blob/main/docs/atlas/ATLAS_ARQUITECTURA_COGNITIVA_CORE1_MOTOR2_v1.md)

---

## 🟢 BLOQUE A: Lo que Perplexity Computer ejecuta AHORA

Estas tareas son 100% SQL/Supabase + documentación. No tocan código frontend. Tú solo tienes que aprobar y yo ejecuto.

| # | Tarea | Herramienta | Tiempo | Riesgo |
|---|---|---|---|---|
| A1 | Agregar `hotel_id` FK a `marketing_offers` | SQL en Supabase | 2 min | Cero — columna nueva |
| A2 | Poblar `hotel_id` desde `hotel_slug` existente | SQL en Supabase | 1 min | Cero — solo UPDATE |
| A3 | Crear índice en `hotel_id` | SQL en Supabase | 1 min | Cero |
| A4 | Agregar columnas `source`, `offer_id`, `atlas_offer_id`, `date_range_id` a `bookings` | SQL en Supabase | 2 min | Cero — columnas nuevas, IF NOT EXISTS |
| A5 | Crear RPC `calcular_margen` (Escudo Financiero compartido) | SQL en Supabase | 5 min | Cero — función nueva |
| A6 | Crear RPC `consultar_disponibilidad` | SQL en Supabase | 5 min | Cero — función nueva |
| A7 | Crear bucket `hotel-images` en Storage | Dashboard Supabase | 3 min | Cero — recurso nuevo |
| A8 | Normalizar `gallery_data` esquema viejo → nuevo (todos los hoteles) | SQL en Supabase | 3 min | Bajo — solo normaliza claves JSONB |
| A9 | Limpiar ofertas duplicadas SEMANA SANTA | SQL en Supabase | 1 min | Cero — despublicar duplicados |
| A10 | Subir documentación actualizada a GitHub | GitHub API | 5 min | Cero |

**Total Bloque A: ~30 minutos** — Ejecutable ahora mismo por Computer.

---

## 🟡 BLOQUE B: Lo que tú (Director) decide/ejecuta

| # | Tarea | Qué necesitas decidir | Dónde |
|---|---|---|---|
| B1 | Multiplicadores de temporada | ¿Alta = 1.3? ¿Media = 1.15? ¿Baja = 1.0? Dame los números | Perplexity te prepara el SQL |
| B2 | Crear bucket Storage | Click en Supabase Dashboard → Storage → New Bucket (si no quieres que use la API) | Dashboard |
| B3 | Fees por método de pago | ¿Visa 3.5%? ¿PayPal 4.5%? ¿Transferencia 0%? Confirma | Para el RPC calcular_margen |
| B4 | ¿Cuáles ofertas eliminar/despublicar? | Las 2 SEMANA SANTA duplicadas + "Test" — ¿confirmas? | Perplexity ejecuta |
| B5 | Revisar instrucciones quirúrgicas antes de enviar a Horizons | Leer Bloque C abajo, aprobar o modificar | Documento |

---

## 🔴 BLOQUE C: Cirugías Explícitas para Horizons

Cada cirugía es UN cambio, EN UN archivo, con instrucciones de copiar-pegar. Sin ambigüedad.

---

### CIRUGÍA 1: Precio real en tarjetas de hotel (reemplazar $199 hardcoded)

**Archivo:** El componente que renderiza las tarjetas de hotel en `/destinos/punta-cana` (probablemente `HotelCard.jsx` o similar en `src/components/`)

**Buscar esta línea (o similar):**
```jsx
DESDE $199 / noche
```

**Reemplazar con:**
```jsx
DESDE ${hotel.min_price || 'Consultar'} / noche
```

**Y agregar en el query de carga de hoteles** (donde se hace el `supabase.from('hotels_master').select(...)`):

Agregar una subconsulta o campo calculado. La forma más simple:
```javascript
// Después de cargar los hoteles, para cada uno obtener el precio mínimo:
const { data: hotels } = await supabase
  .from('hotels_master')
  .select('*, rooms(rates(price_per_night))')
  .eq('zone', zone)
  .eq('is_active', true);

// En el renderizado, calcular min_price:
const min_price = Math.min(
  ...hotel.rooms.flatMap(r => r.rates.map(rt => rt.price_per_night))
);
```

**Criterio de éxito:** Los hoteles muestran precios diferentes entre sí (no todos $199).

**NO TOCAR:** Nada más en la tarjeta. Solo el precio.

---

### CIRUGÍA 2: Conectar widget de búsqueda al RPC

**Archivo:** El componente del buscador "Reserva tu Estadía" en la página de destinos (probablemente en `src/pages/DestinoDetalle.jsx` o `src/components/SearchWidget.jsx`)

**Buscar:** El formulario con campos LLEGADA, SALIDA, ADULTOS, NIÑOS y el botón de búsqueda.

**El botón actualmente:** No hace nada o tiene un `onClick` vacío/placeholder.

**Reemplazar el onClick del botón con:**
```javascript
const handleSearch = async () => {
  const { data, error } = await supabase.rpc('calcular_cotizacion', {
    hotel_name_query: hotelName,  // o el slug del destino
    check_in: checkIn,            // del datepicker
    check_out: checkOut,          // del datepicker
    adults: adults,               // del input
    children: children            // del input
  });
  
  if (data) {
    setSearchResults(data);  // mostrar resultados debajo
  }
};
```

**Criterio de éxito:** Al buscar fechas + pax, aparecen resultados con precios reales debajo del widget.

**NO TOCAR:** El diseño visual del widget. Solo conectar la lógica.

---

### CIRUGÍA 3: JOIN de marketing_offers con hotels_master en la landing de ofertas

**Archivo:** El componente que carga las ofertas en `/destinos/ofertas` (probablemente `OffersPage.jsx` o `MarketingOffers.jsx`)

**Buscar la consulta Supabase actual** (algo como):
```javascript
supabase.from('marketing_offers').select('*')
```

**Reemplazar con:**
```javascript
supabase.from('marketing_offers').select(`
  *,
  offer_date_ranges(*),
  hotels_master!hotel_id(
    name, stars, zone, 
    gallery_urls, gallery_data,
    description
  )
`).eq('is_published', true)
 .gte('valid_until', new Date().toISOString())
```

**En el renderizado de cada tarjeta**, usar:
- `offer.hotels_master.name` en vez del nombre hardcoded
- `offer.hotels_master.stars` para las estrellas
- `offer.hotels_master.zone` para la ubicación
- `offer.hotels_master.gallery_data[0]?.url` para la foto (en vez de `provider_image_url` si es null)

**Criterio de éxito:** Las tarjetas de oferta muestran foto real del hotel desde `hotels_master`, no imagen placeholder.

**NO TOCAR:** El layout de las tarjetas. Solo la fuente de datos.

---

### CIRUGÍA 4: Detalle de oferta muestra galería y "Acerca del hotel"

**Archivo:** El componente de detalle de oferta (`OfferDetail.jsx` o similar)

**Buscar la consulta actual** y reemplazar el `select` con:
```javascript
supabase.from('marketing_offers').select(`
  *,
  offer_date_ranges(*),
  hotels_master!hotel_id(
    name, slug, stars, zone,
    description, long_description,
    gallery_urls, gallery_data,
    restaurants_data, services_data,
    amenities, policies,
    about_image, hero_video
  )
`).eq('id', offerId).single()
```

**Agregar debajo del bloque "Lo que incluye":**
```jsx
{offer.hotels_master && (
  <>
    {/* Galería */}
    {offer.hotels_master.gallery_data?.length > 0 && (
      <div className="grid grid-cols-2 md:grid-cols-3 gap-2 mt-6">
        {offer.hotels_master.gallery_data.map((img, i) => (
          <img 
            key={i} 
            src={img.url} 
            alt={img.title || offer.hotels_master.name}
            className="rounded-lg w-full h-48 object-cover"
          />
        ))}
      </div>
    )}
    
    {/* Acerca del hotel */}
    {offer.hotels_master.long_description && (
      <div className="mt-6">
        <h3 className="font-semibold text-lg">Acerca del Hotel</h3>
        <p className="text-gray-600 mt-2">{offer.hotels_master.long_description}</p>
      </div>
    )}
  </>
)}
```

**Criterio de éxito:** La página de oferta muestra fotos reales del hotel y descripción.

**NO TOCAR:** El sidebar de precio, las fechas, el botón Reservar. Solo agregar contenido visual.

---

### CIRUGÍA 5: Crear ruta /reserva/oferta/:id (Formulario de reserva)

**Esta es la cirugía más grande. Solo cuando las cirugías 1-4 estén verificadas.**

**Crear archivo nuevo:** `src/pages/ReservaOferta.jsx`

**Registrar ruta en** `App.jsx` o `main.jsx`:
```jsx
<Route path="/reserva/oferta/:id" element={<ReservaOferta />} />
```

**El componente debe:**
1. Cargar la oferta + rango seleccionado (viene de query param `?range=`)
2. Mostrar resumen (hotel, fechas, precio)
3. Formulario: nombre titular, cédula/pasaporte, teléfono, email
4. Botón "Confirmar Reserva"
5. Al confirmar → INSERT en `bookings` con `source = 'atlas_offer'`

**NO TOCAR:** Nada del flujo de pago aún. Solo captura de datos + creación de booking.

---

## 📅 CRONOGRAMA SUGERIDO

| Día | Computer | Director | Horizons |
|---|---|---|---|
| **Hoy (26 Mar)** | Ejecuta Bloque A completo (SQL + Storage + GitHub) | Aprueba multiplicadores y fees | — |
| **27 Mar** | Revisa resultados, prepara tests | Envía Cirugía 1 y 2 a Horizons | Ejecuta Cirugía 1 (precio $199) |
| **28 Mar** | Valida que Cirugía 1 funcione en web | Aprueba/rechaza | Ejecuta Cirugía 2 (buscador) |
| **29-30 Mar** | Valida Cirugía 2 | Envía Cirugía 3 y 4 | Ejecuta Cirugía 3-4 (JOIN + galería) |
| **31 Mar - 2 Abr** | Valida todo | — | Ejecuta Cirugía 5 (ruta reserva) |
| **3-5 Abr** | Test E2E completo | Valida flujo como usuario | Fixes si hay bugs |

---

## ⚠️ REGLAS PARA ENVIAR CIRUGÍAS A HORIZONS

1. **UNA cirugía por mensaje.** No enviar 3 a la vez.
2. **No enviar la siguiente hasta verificar la anterior.** Computer valida en la web.
3. **Cada cirugía tiene "Criterio de éxito"** — Horizons no declara éxito sin cumplirlo.
4. **"NO TOCAR"** es sagrado. Si tocan algo fuera de la cirugía, se rechaza.
5. **Computer audita después de cada entrega** — inspección directa de la web.

---

*Plan basado en auditoría real del schema Supabase y capacidades verificadas*  
*Perplexity Computer — Aliun Travel SRL*