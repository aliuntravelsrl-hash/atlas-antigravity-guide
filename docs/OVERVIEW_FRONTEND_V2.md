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

## 🎯 Próximos Pasos Técnicos en el Frontend

1. **Estabilización de Rutas:** Asegurar que la migración del Core a **React Router DOM v6** no rompa las redirecciones de checkout.
2. **Proxy de Imágenes:** Validar que todas las imágenes externas de hoteles carguen bajo HTTPS y utilicen proxies seguros para evitar advertencias de contenido mixto en el navegador del cliente.
3. **Checkout de Excursión:** Diseñar el primer formulario E2E de reservas de excursiones en coordinación con el RPC de Supabase.
