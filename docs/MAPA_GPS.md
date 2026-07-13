# 🗺️ MAPA GPS — Arquitectura de Código del Ecosistema ATLAS
**Actualizado:** 13 de Julio de 2026 | **SSOT de Rutas y Estado Operativo**

Este documento consolida el mapa de archivos físicos, repositorios, ubicaciones de red, estados operativos y restricciones de modificación para todos los componentes del ecosistema ATLAS.

---

## 🖥️ 1. Repositorio: `atlas-booking-frontend-v2` (Web Pública)
* **Propósito:** Portal de cara al cliente final para reservas de hoteles y excursiones (`aliuntravelsrl.com`).
* **Stack:** Vite 6 + React 18 + React Router DOM v6 + Tailwind CSS.

| Concepto / Frente | Ruta en Código | Estado Operativo | Restricción / Qué NO tocar |
|---|---|---|---|
| **Buscador & Cotizador** | `src/pages/booking/RoomSelection.jsx`<br>`src/pages/DestinationDynamicPage.jsx` | **✅ Operativo** (Conectado a RPC `calcular_cotizacion`) | **NO TOCAR** la lógica de validación de fechas; debe respetar el huso horario local de RD. |
| **Flujo de Huéspedes** | `src/pages/booking/GuestDetails.jsx`<br>`src/pages/booking/ReviewBooking.jsx` | **✅ Operativo** | **NO TOCAR** los campos del formulario (cédula/pasaporte, email, teléfono); son requeridos por políticas hoteleras. |
| **Checkout & Pasarela** | `src/pages/checkout/CheckoutPage.jsx`<br>`src/pages/checkout/CheckoutRedirectHandler.jsx` | **⚠️ En espera (RNC)** | **REGLA SEV0:** Prohibido utilizar `service_role` key de Supabase. El cobro y verificación se delega a n8n. |
| **Ficha del Hotel** | `src/pages/HotelFullPage.jsx`<br>`src/pages/HotelPage.jsx` | **✅ Operativo** (Fase 1 completada) | **NO TOCAR** el renderizador de galerías de fotos. Debe usar proxy HTTPS `images.weserv.nl`. |
| **Catálogo de Excursiones** | `src/pages/ExcursionsPage.jsx`<br>`src/pages/ExcursionDetailPage.jsx` | **✅ Operativo** (Solo catálogo visual) | **PENDIENTE:** Conectar el checkout final al RPC `funnel_excursiones`. |
| **Gestión de Ofertas** | `src/pages/oferta/OfertaDetalle.jsx`<br>`src/pages/reserva/ReservaOferta.jsx` | **✅ Operativo** | **NO TOCAR** el flag de origen `source = 'atlas_offer'` en el INSERT del booking. |

---

## ⚙️ 2. Repositorio: `atlas-admin-v2` (Panel de Administración)
* **Propósito:** Panel operativo para agentes de Aliun Travel (`atlas.aliuntravelsrl.com`).
* **Stack:** React Panel + Supabase SDK.

| Concepto / Frente | Ruta en Código | Estado Operativo | Restricción / Qué NO tocar |
|---|---|---|---|
| **Gestor de Reservas** | `src/pages/AdminDashboardV2.jsx`<br>`src/pages/booking/BookingOpsPanel.jsx` | **✅ Operativo** | **NO TOCAR** el flujo de estados de reserva (`pending`, `confirmed`, `cancelled`). |
| **Auditoría Financiera** | `src/pages/AdminAuditPage.jsx` | **✅ Operativo** (Monitorea discrepancias) | Consume la vista `public.transactions` para cuadrar conciliación. |
| **Inventario & Tarifas** | `src/pages/AdminPricingPage.jsx`<br>`src/pages/AdminServicesSetupPage.jsx` | **✅ Operativo** (79 hoteles) | Permite la inyección manual de tarifas bloque 2026. |

---

## 🤖 3. VPS 2: Swarm v3.0 (5 Agentes Hermes)
* **Propósito:** Ejecución de automatizaciones y crons operativos internos (`2.24.198.231`).
* **Ubicación en Servidor:** `/opt/data/atlas-hermes-v2/stacks/{slug}/`
* **Orquestador:** Dispatcher (`ATLAS-TECH` orquesta → `atlas_tasks` en Supabase).

| Agente / Stack | Archivo de Configuración | Estado Operativo | Restricción / Qué NO tocar |
|---|---|---|---|
| **hermes-ops** | `stacks/hermes-ops/SOUL.md`<br>`stacks/hermes-ops/REHIDRATACION.md` | **✅ Activo** (Control de reservas) | **NO TOCAR** políticas de auto-cron. Solo crons autorizados en `aliun-rrhh-v2`. |
| **hermes-commercial** | `stacks/hermes-commercial/SOUL.md` | **✅ Activo** (Tarifas y B2B) | **Corregido:** Bucle de Telegram Watchdog eliminado. |
| **hermes-marketing** | `stacks/hermes-marketing/SOUL.md` | **✅ Activo** (Copias y ofertas) | Genera copias mediante Gemini y `WF-MKT-GENERATE-OFFER-v3`. |
| **ariadne-data** | `stacks/ariadne-data/SOUL.md` | **✅ Activo** (Fuentes financieras) | SSOT de tasas de cambio y auditoría de transacciones. |
| **hermes-qa** | `stacks/hermes-qa/SOUL.md` | **✅ Activo** (Pruebas automáticas) | Corre suites de validación contra Supabase y n8n. |

---

## ☁️ 4. Supabase: `oyihiyivdhfxpyiwnmqk` (Cerebro Operativo)
* **Propósito:** SSOT Absoluto de Datos y Reglas de Integridad.

| Componente | Tabla / RPC / Vista | Estado Operativo | Restricción / Qué NO tocar |
|---|---|---|---|
| **Esquema de Pagos** | Tabla: `public.atlas_payments` | **✅ Normalizada** (Clave FK obligatoria) | **PROHIBIDO** insertar registros planos sin FK válida a `bookings.id`. |
| **Histórico Legacy** | Tabla: `public.atlas_payments_flat` | **❌ Deprecada** | No usar para nuevos desarrollos. Se purgará tras la migración. |
| **Vista Contable** | Vista: `public.transactions` | **✅ Operativo** (SELECT de atlas_payments) | Usada para el cuadre contable y auditoría de pasarelas. |
| **Escudo Financiero** | RPC: `calcular_margen` | **✅ Operativo** | Calculates markup y neto de manera centralizada. |
| **Cotizador Core** | RPC: `calcular_cotizacion` | **✅ Operativo** | Utilizado por el buscador principal del frontend. |
| **Funnel Excursiones** | RPC: `funnel_excursiones` | **⚠️ Pendiente** | Integrará el inventario local con los estados de reserva de excursión. |

---

## ⚙️ 5. Reglas Generales de Navegación del Desarrollador (Antigravity)

1. **Consulta el GPS Primero:** Antes de realizar cualquier cambio en un componente o tabla, verifica esta guía para confirmar su ruta física y su estado de bloqueo (ej. RNC).
2. **Validación Visual en Sandbox (Obligatoria):** Antes de desplegar cualquier cambio visual o estructural de la UI en producción, se debe desarrollar en Sandbox y mostrar el HTML de previsualización al Director para su evaluación y aprobación explícita.
3. **Registro de Cambios (Cierre Verificable):** Tras culminar una sesión, actualiza las rutas o estados modificados en este archivo y en `CONTEXT.md`, crea el commit y realiza el `git push` al repositorio `atlas-antigravity-guide`.
