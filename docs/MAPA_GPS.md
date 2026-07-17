# 🗺️ MAPA GPS — Arquitectura de Código del Ecosistema ATLAS
**Actualizado:** 17 de Julio de 2026 | **SSOT de Rutas y Estado Operativo**
**Última actualización por:** Computer (Perplexity) — Sesión 17 JUL 2026 — Antigravity en time-out Google

Este documento consolida el mapa de archivos físicos, repositorios, ubicaciones de red, estados operativos y restricciones de modificación para todos los componentes del ecosistema ATLAS.

---

## 🖥️ 1. Repositorio: `atlas-booking-frontend-v2` (Web Pública)
* **Propósito:** Portal de cara al cliente final para reservas de hoteles y excursiones (`aliuntravelsrl.com`).
* **Stack:** Vite 6 + React 18 + React Router DOM v6 + Tailwind CSS.

| Concepto / Frente | Ruta en Código | Estado Operativo | Restricción / Qué NO tocar |
|---|---|---|---|
| **Banner Principal (Hero)** | `src/components/Hero.jsx` | **✅ Operativo** (Parallax Dinámico + Imagen Familiar) | **NO TOCAR** la imagen local `/hero-family.jpg` ni la animación del scroll de framer-motion. |
| **Catálogo de Destinos (Buscador)** | `src/pages/DestinationsPage.jsx` | **✅ Operativo** (Parallax Dinámico + Imagen Pareja) | **NO TOCAR** la imagen local `/destinos-hero.jpg` ni la animación del scroll de framer-motion. |
| **Ficha de Destino (Individual)** | `src/pages/DestinationPage.jsx` | **✅ Operativo** (Parallax Dinámico + Imágenes de Destinos) | **NO TOCAR** el mapeo estático `DESTINATION_METADATA` de imágenes de fondo (ej: `/samana-hero.jpg` para Samaná). |
| **Catálogo General & Búsqueda** | `src/pages/HotelCatalogPage.jsx`<br>`src/pages/HotelsPage.jsx` | **✅ Operativo** | Cargar datos desde Supabase filtrando por estado activo. |
| **Carrusel de Destinos Populares** | `src/components/CategoryCards.jsx` | **✅ Operativo** (Migrado a Swiper 3D Coverflow) | **NO TOCAR** el fondo degradado claro para mantener consistencia visual con el Home y el estilo ShareTrip. |
| **Carrusel de Ofertas Especiales** | `src/components/OffersSection.jsx` | **✅ Operativo** (Migrado a Swiper lineal con loop infinito) | **NO TOCAR** la lógica de duplicación si las ofertas son < 6 para evitar fallas del loop en producción. |
| **Ficha del Hotel** | `src/pages/HotelFullPage.jsx`<br>`src/pages/HotelPage.jsx`<br>`src/pages/HotelDetailPage.jsx` | **✅ Operativo** (Fase 1 completada) | **NO TOCAR** el renderizador de galerías de fotos. Debe usar proxy HTTPS `images.weserv.nl`. |
| **Flujo de Reserva de Hotel** | `src/pages/booking/RoomSelection.jsx`<br>`src/pages/booking/GuestDetails.jsx`<br>`src/pages/booking/ReviewBooking.jsx`<br>`src/pages/booking/Confirm.jsx` | **✅ Operativo** (Flujo nativo conectado a Supabase) | **NO TOCAR** la lógica de cálculo de precios del lado del cliente en `ReviewBooking.jsx`; debe basarse en el RPC `calcular_cotizacion`. |
| **Catálogo de Excursiones** | `src/pages/ExcursionsPage.jsx`<br>`src/pages/ExcursionStandalonePage.jsx` | **✅ Operativo** (Solo catálogo visual) | **PENDIENTE:** Conectar el checkout final al RPC `funnel_excursiones`. |
| **Gestión de Ofertas** | `src/pages/oferta/OfertaDetalle.jsx`<br>`src/pages/reserva/ReservaOferta.jsx` | **✅ Operativo** | **NO TOCAR** el flag de origen `source = 'atlas_offer'` en el INSERT del booking. |
| **Pasarela de Pagos & Checkout** | `src/pages/checkout/CheckoutPage.jsx`<br>`src/pages/checkout/CheckoutRedirectHandler.jsx` | **⚠️ En espera (RNC)** | **REGLA SEV0:** Prohibido utilizar `service_role` key de Supabase. El cobro y verificación se delega a n8n. |

---

## ⚙️ 2. Repositorio: `-atlas-admin-v2` (Panel de Administración)
* **Propósito:** Panel operativo para agentes de Aliun Travel (`atlas.aliuntravelsrl.com`).
* **Stack:** React + Vite SPA + Supabase SDK + GitHub Actions CI/CD → Hostinger.
* **⚠️ ATENCIÓN (17 JUL 2026):** El nombre del repo en GitHub tiene prefijo dash: `-atlas-admin-v2` (no `atlas-admin-v2`).

### 🔧 Incidente 17 JUL 2026 — White Screen post-commit imágenes Wyndham
**Síntoma:** Pantalla blanca total en `/admin41` tras commit `11b21e66` ("media: agregar imágenes Wyndham Alltra Punta Cana").
**Root cause real:** Deploy parcial en Hostinger — el `index.html` del build nuevo llegó al servidor (referenciaba `index-B9GhaAvc-v3.js`) pero los archivos físicos de `/assets/` del mismo build NO sincronizaron. El catch-all del `.htaccess` devolvía `index.html` con MIME `text/html` cuando el browser pedía el bundle JS → Chrome rechaza ejecutar scripts con MIME incorrecto → pantalla blanca permanente.
**Fix aplicado (Computer/Perplexity):**
- Commit `af0e4ac7` — `public/.htaccess` corregido: `AddType application/javascript .js .mjs`, `AddType text/css .css`, cache por `FilesMatch` correcta.
- Commit `26cd90cf` — Force redeploy limpio → CI rebuildó y subió assets completos al servidor.
- **Estado post-fix:** MIME `content-type: text/javascript` ✅ — Bundle `index-0nUnIgae.js` ✅ — Dashboard operativo ✅.

**Patrón de diagnóstico para futuros incidentes iguales:**
1. Si build CI = `success` pero pantalla en blanco → NO es error de código → es deploy parcial.
2. Verificar: `curl -sI https://atlas.aliuntravelsrl.com/assets/index-*.js | grep content-type` — si responde `text/html` el archivo físico no existe en disco.
3. Fix siempre: force redeploy desde CI (editar `deploy-trigger.txt` y push).

| Concepto / Frente | Ruta en Código | Estado Operativo | Restricción / Qué NO tocar |
|---|---|---|---|
| **Dashboard Principal** | `src/components/AppShell.jsx` (ruta `/admin41`) | **✅ Operativo** (post-fix 17 JUL) | Ruta física en Hostinger: `~/domains/atlas.aliuntravelsrl.com/public_html/`. |
| **Gestor de Reservas** | `src/components/admin/AdminBookingsPanel.jsx` | **✅ Operativo** (ruta: `/admin/bookings` y `/sales/confirmaciones`) | Usa `supabaseAdmin` desde `src/lib/supabaseAdmin.js`. Fallback a anon key si `VITE_SUPABASE_SERVICE_KEY` no está en secrets. |
| **Cliente supabaseAdmin** | `src/lib/supabaseAdmin.js` | **✅ Existe** — fallback a anon key hardcodeado | `VITE_SUPABASE_SERVICE_KEY` **NO está** en los GitHub Actions secrets del workflow. Solo están `VITE_SUPABASE_URL` y `VITE_SUPABASE_ANON_KEY`. Agregar el secret para activar service_role real. |
| **Mission Control** | `src/components/MissionControlLive.jsx` (ruta `/mission`) | **✅ Operativo** (v2.8 — dual-feed Actividad Reciente) | Poll: FAST=30s, MED=60s, SLOW=300s. |
| **Mesa de Tareas** | Parte de `MissionControlLive.jsx` | **✅ Operativo** — 32 tareas activas en `atlas_tasks` | Filtros, + Nueva Tarea, badges de frente. |
| **Marketing Mission Control** | `src/pages/admin/MarketingMissionControl.jsx` (ruta `/marketing`) | **✅ Operativo** | — |
| **Ariadne Data Panel** | `src/pages/admin/AriadnePanel.jsx` (ruta `/ariadne`) | **✅ Operativo** | — |
| **War Room** | `src/components/marketing/WarRoomV41.jsx` (ruta `/warroom`) | **✅ Operativo** | — |
| **Integrity Monitor** | Ruta `/integrity` | **✅ Operativo** | — |
| **Hoteles Admin** | `src/pages/admin/AdminHotelsPage.jsx` (ruta `/admin/hotels`) | **✅ Operativo** | — |
| **Excursiones Admin** | `src/pages/admin/AdminExcursionsPage.jsx` (ruta `/admin/excursions`) | **✅ Operativo** | — |
| **Pagos por Booking** | `src/components/admin/PaymentGatewayPage.jsx` (ruta `/admin/payments/:bookingRef`) | **⚠️ En espera (RNC)** | — |
| **CRM Pipeline** | Ruta `/crm/pipeline` | **✅ Operativo** | — |
| **CRM Dashboard** | Ruta `/crm/dashboard` | **✅ Operativo** | — |
| **API Toolbox** | Ruta `/api-toolbox` | **✅ Operativo** | — |
| **Booking Ops Panel** | `src/pages/admin/BookingOpsPanel.jsx` (ruta `/dashboard26`) | **✅ Operativo** | — |

### 🔐 Configuración de Deploy (CI/CD)
- **Workflow:** `.github/workflows/deploy.yml` → GitHub Actions → SCP → Hostinger
- **Build time:** ~60-75 segundos
- **Secrets configurados:** `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`, `HOSTINGER_SSH_KEY`, `HOSTINGER_PORT`, `HOSTINGER_HOST`, `HOSTINGER_USER`
- **Secret FALTANTE:** `VITE_SUPABASE_SERVICE_KEY` — sin esto `supabaseAdmin` cae al anon key (funcional pero sin bypass RLS real)
- **Deploy trigger manual:** Editar `deploy-trigger.txt` en raíz y push → dispara CI completo

### 📁 Archivos clave de configuración
- `public/.htaccess` → se copia a `dist/.htaccess` en el build de Vite → **es el que llega al servidor**
- `.htaccess` (raíz) → referencia solo, NO es el que Vite despliega
- `vite.config.js` → configuración del bundler
- `src/lib/supabaseClient.js` → cliente anon (export nombrado `supabase`)
- `src/lib/customSupabaseClient.js` → cliente custom (también export `supabase`)
- `src/lib/supabaseAdmin.js` → cliente service_role con fallback a anon

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

### 📋 atlas_tasks — Estado Swarm (post-remap 07 JUL 2026)
| Owner | asignado_tipo | Tareas pendientes |
|-------|--------------|-------------------|
| Hermes Ops | swarm | 13 |
| Hermes Commercial | swarm | 8 |
| Hermes Marketing | swarm | 4 |
| Ariadne Data | swarm | 2 |
| Director | antigravity | 5 |
| **TOTAL** | — | **32** |

**Tareas reasignadas a Hermes Ops (antes Computer):** ATL-001 (Auth), ATL-003 (n8n owners), OPS-002 (service_role key), OPS-003 (forense git).

---

## ☁️ 4. Supabase: `oyihiyivdhfxpyiwnmqk` (Cerebro Operativo)
* **Propósito:** SSOT Absoluto de Datos y Reglas de Integridad.
* **Región:** us-east-1 | **Estado:** ACTIVE_HEALTHY

| Componente | Tabla / RPC / Vista | Estado Operativo | Restricción / Qué NO tocar |
|---|---|---|---|
| **Esquema de Pagos** | Tabla: `public.atlas_payments` | **✅ Normalizada** (Clave FK obligatoria) | **PROHIBIDO** insertar registros planos sin FK válida a `bookings.id`. |
| **Histórico Legacy** | Tabla: `public.atlas_payments_flat` | **❌ Deprecada** | No usar para nuevos desarrollos. Se purgará tras la migración. |
| **Vista Contable** | Vista: `public.transactions` | **✅ Operativo** (SELECT de atlas_payments) | Usada para el cuadre contable y auditoría de pasarelas. |
| **Escudo Financiero** | RPC: `calcular_margen` | **✅ Operativo** | Calcula markup y neto de manera centralizada. |
| **Cotizador Core** | RPC: `calcular_cotizacion` | **✅ Operativo** | Utilizado por el buscador principal del frontend. |
| **Funnel Excursiones** | RPC: `funnel_excursiones` | **⚠️ Pendiente** | Integrará el inventario local con los estados de reserva de excursión. |
| **Atlas Tasks v2** | Tabla: `public.atlas_tasks` | **✅ Operativo** (+6 columnas migradas, RLS SELECT activa) | RPC: `rpc_crear_tarea`. Política SELECT anon + authenticated. |
| **Logs Operativos** | Tabla: `public.logs_operativos` | **✅ Operativo** | Feed de Actividad Reciente en Mission Control. |
| **CRM Leads** | Tabla: `public.crm_leads` | **✅ Operativo** | Vinculación relacional a `bookings.lead_id`. Creación automática en `AdminBookingsPanel`. |
| **CRM Deals** | Tabla: `public.crm_deals` | **⚠️ Parcial** | `crm_deals.booking_id` es `null` en el 100% de registros de producción. Ver `AUDITORIA_RELACION_CRM_BOOKING.md`. |
| **Migraciones aplicadas (sesión 06-07 JUL)** | — | — | `atlas_tasks_v2_migration`, `atlas_tasks_select_anon`, `rls_audit_fix_all_unprotected_tables`, `atlas_tasks_remap_owners_swarm_real_0707` |

---

## ⚙️ 5. Reglas Generales de Navegación del Desarrollador

1. **Consulta el GPS Primero:** Antes de realizar cualquier cambio en un componente o tabla, verifica esta guía para confirmar su ruta física y su estado de bloqueo.
2. **Validación Visual en Sandbox (Obligatoria):** Antes de desplegar cualquier cambio visual o estructural de la UI en producción, se debe desarrollar en Sandbox y mostrar el HTML de previsualización al Director para su evaluación y aprobación explícita.
3. **Registro de Cambios (Cierre Verificable):** Tras culminar una sesión, actualiza las rutas o estados modificados en este archivo y en `CONTEXT.md`, crea el commit y realiza el `git push` al repositorio `atlas-antigravity-guide`.

---

## 📋 6. Historial de Incidentes Críticos Resueltos

| Fecha | Incidente | Root Cause | Fix | SHA |
|-------|-----------|------------|-----|-----|
| 06 JUL 2026 | White screen `/admin41` | `stale_payments` RPC retornaba `null` → `.map()` sin guard → `TypeError: O.map is not a function` | `Array.isArray(data) ? data : []` guard en todos los RPCs | `9b2b8894` |
| 06 JUL 2026 | Mesa de Tareas — 0 tareas en UI | RLS SELECT faltaba en `atlas_tasks` (anon sin política) | Política SELECT anon + authenticated aplicada vía migración | — |
| 06 JUL 2026 | n8n-status webhook colgado | `responseMode` incorrecto en n8n | `responseMode: "responseNode"` — síncrono | — |
| 17 JUL 2026 | White screen `/admin41` post-imágenes Wyndham | Deploy parcial Hostinger — `index.html` nuevo llegó, assets `/` NO sincronizaron → MIME mismatch `text/html` en bundle JS | `public/.htaccess` corregido + force redeploy CI | `af0e4ac7` / `26cd90cf` |

---

## 🔑 7. Constantes y Valores Clave del Sistema

| Constante | Valor |
|-----------|-------|
| Supabase Project ID | `oyihiyivdhfxpyiwnmqk` |
| Supabase URL | `https://oyihiyivdhfxpyiwnmqk.supabase.co` |
| Supabase Anon Key | `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im95aWhpeWl2ZGhmeHB5aXdubXFrIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NjI0Mzk5NzUsImV4cCI6MjA3ODAxNTk3NX0.8jbifKF9FCExFN3PF1OeUFDVRoHyf652vMHpIgR1DSE` |
| n8n URL | `https://n8n-n8n.xaruuo.easypanel.host` |
| Admin Panel URL | `https://atlas.aliuntravelsrl.com` |
| Web Pública URL | `https://aliuntravelsrl.com` |
| Auth localStorage key | `aliun_admin_user` |
| Auth RPC | `verify_admin_user` |
| POLL_FAST | 30s |
| POLL_MED | 60s |
| POLL_SLOW | 300s |
| FrenteColor F1-FRONTEND | `#3B82F6` |
| FrenteColor F2-BACKEND-CORE | `#8B5CF6` |
| FrenteColor F3-ATRACCION | `#F59E0B` |
| FrenteColor F4-RRHH-IA | `#10B981` |
| FrenteColor F5-SEGURIDAD | `#EF4444` |

---

*Mantenido por: Director Aldo Hilario | Aliun Travel SRL | República Dominicana*
*Última sesión documentada por: Computer (Perplexity) — 17 JUL 2026 — Cobertura de Antigravity (time-out Google)*
