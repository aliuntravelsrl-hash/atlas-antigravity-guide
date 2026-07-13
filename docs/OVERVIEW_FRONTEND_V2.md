# Overview — atlas-booking-frontend-v2 (Web Pública)
**Actualizado:** 13 de Julio de 2026 | **Foco:** Web Pública de Reservas (`aliuntravelsrl.com`)

Este documento ordena y mapea los frentes de trabajo y ramas de desarrollo del Árbol Maestro de Conceptos que impactan o interactúan directamente con la aplicación de frontend **`atlas-booking-frontend-v2`**.

---

## 🗺️ Mapa de Relaciones por Ramas

```
┌─────────────────────────────────────────────────────────┐
│              atlas-booking-frontend-v2                  │
│               (Vite 6 + React Router v6)                │
└────────────────────────────┬────────────────────────────┘
                             │
       ┌─────────────────────┼─────────────────────┐
       ▼                     ▼                     ▼
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│    RAMA 3    │      │    RAMA 4    │      │    RAMA 7    │
│  Inventario  │      │  Excursiones │      │  Finanzas &  │
│  de Hoteles  │      │   y Funnel   │      │  Trazabilidad│
└──────────────┘      └──────────────┘      └──────────────┘
```

---

## 🌳 Ramas Vinculadas al Frontend

### 1. RAMA 3 — Motor de Hoteles (Inventario & Tarifas)
* **Objetivo:** Mostrar tarifas dinámicas y contenido enriquecido real para los 79 hoteles activos en Supabase.
* **Componentes de Frontend Afectados:**
  - Tarjetas de catálogo de hoteles (reemplazo definitivo del precio de placeholder `$199` por `min_price` real).
  - Buscador general ("Reserva tu Estadía") en la página de destinos, conectado al RPC de cotizaciones.
  - Vistas de detalle de hotel: Galerías fotográficas (de hasta 23 slides cargados mediante proxy HTTPS `images.weserv.nl`), menús de restaurantes, servicios y políticas de cancelación.
* **Frentes Activos:**
  - **Fase 2 de Rescate Visual:** Saneamiento y carga de imágenes en Supabase para complejos de Samaná, La Romana y Punta Cana.
  - **Room Picker B2B:** Preparación de la interfaz con pestañas o tarjetas por proveedor (Ratehawk vs TBO) que muestre los badges `PAGO EN DESTINO` y `PREPAGO`.

### 2. RAMA 4 — Motor de Excursiones (Catálogo & Checkout)
* **Objetivo:** Integrar la compra de excursiones directamente en la experiencia de usuario.
* **Componentes de Frontend Afectados:**
  - Landing page del catálogo de excursiones.
  - Funnel de checkout de excursión conectado al RPC `funnel_excursiones`.
* **Frentes Activos:**
  - Mockups y prototipado del flujo de selección de fechas, horarios y pax para excursiones locales.

### 3. RAMA 5 — Motor de Marketing (Landing de Ofertas)
* **Objetivo:** Redirección de campañas y visualización de ofertas especiales de temporada.
* **Componentes de Frontend Afectados:**
  - Landing de ofertas especiales (`/destinos/ofertas`).
  - Ruta de detalle de oferta (`/reserva/oferta/:id`) con bloque "Acerca del Hotel" y galerías reales usando JOIN contra `hotels_master` en Supabase.

### 4. RAMA 7 — Finanzas & Trazabilidad Legal (Checkout & Pagos)
* **Objetivo:** Garantizar transacciones financieras seguras e integradas contablemente.
* **Componentes de Frontend Afectados:**
  - Formulario de checkout y cobro de reservas.
  - Integración de pasarela de tarjetas (AZUL) en el checkout de Horizons (Post-RNC).
* **Frentes Activos / Reglas de Seguridad (SEV0):**
  - **Seguridad de la base de datos:** El frontend de reservas solo puede consumir Supabase a través del cliente público (`anon` key).
  - **Prohibición de service_role:** NUNCA utilizar la clave `service_role` en código React.
  - **Normalización de transacciones:** Toda llamada a pagos al confirmar una reserva debe crear registros en `atlas_payments` vinculados a un ID de reserva no nulo (`booking_id`). Las transacciones planas e independientes están deprecadas.

---

## 🎯 Mapa Quirúrgico de Código (GPS)

Este mapa detalla la correspondencia exacta de cada frente técnico con los archivos físicos de la aplicación `atlas-booking-frontend-v2`, su estado actual y restricciones operativas.

| Frente / Concepto | Ruta de Archivo Físico | Estado Actual | Reglas Específicas / Qué NO tocar |
|---|---|---|---|
| **Catálogo General & Búsqueda** (Rama 3) | `src/pages/HotelCatalogPage.jsx`<br>`src/pages/HotelsPage.jsx` | **✅ Operativo** | Cargar datos desde Supabase filtrando por estado activo. |
| **Ficha de Hotel y Menús** (Rama 3) | `src/pages/HotelFullPage.jsx`<br>`src/pages/HotelPage.jsx`<br>`src/pages/HotelDetailPage.jsx` | **✅ Operativo** (Health Score 7/7 en complejos de la Fase 1) | **NO TOCAR** la lógica de filtrado de restaurantes; debe unificarse según los registros del hotel principal. |
| **Flujo de Reserva de Hotel** (Rama 3) | `src/pages/booking/RoomSelection.jsx`<br>`src/pages/booking/GuestDetails.jsx`<br>`src/pages/booking/ReviewBooking.jsx`<br>`src/pages/booking/Confirm.jsx` | **✅ Operativo** (Flujo nativo conectado a Supabase) | **NO TOCAR** la lógica de cálculo de precios del lado del cliente en `ReviewBooking.jsx`; debe basarse en el RPC `calcular_cotizacion`. |
| **Catálogo de Excursiones** (Rama 4) | `src/pages/ExcursionsPage.jsx`<br>`src/pages/ExcursionStandalonePage.jsx` | **✅ Operativo** (Visual y filtros de zona) | Mantener consistencia con el diseño de grid de excursiones. |
| **Ficha y Detalle de Excursión** (Rama 4) | `src/pages/ExcursionDetailPage.jsx` | **✅ Operativo** (Visual) | **PENDIENTE:** Conectar el botón de reserva al RPC de checkout de excursiones. |
| **Detalle de Oferta de Marketing** (Rama 5) | `src/pages/oferta/OfertaDetalle.jsx` | **✅ Operativo** (Carga datos dinámicos) | Hacer el JOIN con `hotels_master` para asegurar imágenes reales. |
| **Flujo de Reserva por Oferta** (Rama 5) | `src/pages/oferta/OfertaBooking.jsx`<br>`src/pages/reserva/ReservaOferta.jsx`<br>`src/pages/reserva/CheckoutOferta.jsx`<br>`src/pages/reserva/OfertaConfirm.jsx` | **✅ Operativo** | **NO TOCAR** el flag `source = 'atlas_offer'` al registrar la reserva en Supabase. |
| **Pasarela de Pagos & Checkout** (Rama 7) | `src/pages/checkout/CheckoutPage.jsx`<br>`src/pages/checkout/CheckoutRedirectHandler.jsx` | **⚠️ En espera (Bloqueado por RNC)** | **REGLA SEV0:** Prohibido utilizar la key `service_role` de Supabase. El flujo de aprobación debe completarse mediante un Webhook de n8n. |
| **Mis Reservas (Cliente)** (Rama 7) | `src/pages/mis-reservas.jsx` | **✅ Operativo** | Consultas limitadas a través de RLS por UID autenticado del usuario. |

---

## ⚙️ Directivas para Modificación Quirúrgica

1. **Inspección Previa:** Antes de editar cualquier archivo de la tabla anterior, validar en `src/App.jsx` cómo está declarada su ruta.
2. **Higiene de Contenido Mixto:** Todas las URLs de imágenes inyectadas deben usar protocolo HTTPS o ser procesadas por el proxy `images.weserv.nl`.
3. **Validación del Build:** Tras modificar cualquier archivo de React, es obligatorio correr `npm run build` localmente en la carpeta del frontend para asegurar que el bundler (Vite 6) no falle debido a importaciones rotas o problemas de sintaxis.
