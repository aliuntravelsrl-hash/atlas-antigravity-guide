# 🧠 CONTEXT — Antigravity (ATLAS Ecosistema)
**Actualizado:** 29 JUN 2026

Guía de rehidratación rápida de contexto operativo y memoria de estado para Antigravity al iniciar cualquier sesión de pair programming en el ecosistema ATLAS.

---

## 🤖 1. Identidad y Rol Operativo

- **Nombre:** Antigravity — Agente Técnico Externo Soberano de ATLAS.
- **Foco:** Terminal, Docker, repositorios, deploys y cableados de infraestructura.
- **Director:** Aldo Hilario | Aliun Travel SRL | República Dominicana.
- **Idioma de trabajo:** Español obligatorio.
- **Límite de rol:** Antigravity NO habla con clientes finales. Es ingeniería pura.

---

## 🖥️ 2. Infraestructura

| Componente | Detalle |
|---|---|
| **Supabase** | `oyihiyivdhfxpyiwnmqk` — SSOT absoluto |
| **n8n** | `https://n8n-n8n.xaruuo.easypanel.host` |
| **VPS 1** | `72.61.12.170` — EasyPanel, n8n, frontends (expira 2027-11-21) |
| **VPS 2** | `2.24.198.231` — Docker Swarm con 5 agentes Hermes (expira 2027-04-14) |
| **Chatwoot** | `https://n8n-chatwoot.xaruuo.easypanel.host` |
| **Token Chatwoot** | `T5YDT4onGs3wve7wHf3fZXpr` |
| **MCP Server** | `https://n8n-atlas-sales-mcp.xaruuo.easypanel.host` (v1.5.0, 20 tools) |
| **OpenClaw UI** | `https://musculo-aliun.tail02a864.ts.net/` (solo vía Tailscale) |
| **Tailscale** | desktop=`100.102.102.116` · musculo=`100.100.201.7` · note20=`100.110.83.1` |

---

## 📂 3. Repositorios bajo responsabilidad de Antigravity

| Repo | Propósito |
|---|---|
| `atlas-booking-frontend-v2` | Web pública de reservas (`aliuntravelsrl.com`) |
| `atlas-admin-v2` | Panel de administración (`atlas.aliuntravelsrl.com`) |
| `atlas-sales-mcp` | Servidor MCP de herramientas de ventas |
| `atlas-cableados` | Documentación de infraestructura y redes |
| `atlas-marketing-v2` | Motor de ofertas de marketing (en construcción) |
| `aliun-rrhh-v2` | Doctrina y SOUL.md de agentes Hermes |
| `musculo-vps` | Scripts y compose de VPS2 |
| `atlas-antigravity-guide` | **Este repo** — contexto y guía de Antigravity |

---

## 🐳 4. Swarm VPS2 — 5 Agentes Hermes

| Slug | Puerto | Estado |
|---|---|---|
| `hermes-ops` | 32769 | ✅ Producción activa |
| `hermes-commercial` | 32770 | ✅ Activo |
| `hermes-marketing` | 32771 | ✅ Activo |
| `ariadne-data` | 32772 | ✅ Activo |
| `hermes-qa` | 32773 | ✅ Activo |

**Path de stacks en VPS2:** `/opt/data/atlas-hermes-v2/stacks/{slug}/`

Archivos relevantes por stack: `SOUL.md`, `REHIDRATACION.md`, `ALCANCE.md`

---

## 📈 5. Estado de avance — 11 JUL 2026

### ✅ Completado esta sesión

**Estabilización y Seguridad del Frontend (F5-SEC)**
- Actualización de seguridad del core a **Vite 6** (`vite@6.4.3`) y **React Router DOM v6** (`react-router-dom@6.30.4`) para mitigar vulnerabilidades CVE-2026-53571 y CVE-2026-40181.
- Eliminación de `public/index.html` que generaba conflictos 404 al sobreescribir el build de Vite 6.
- Estandarización de nombres de archivos e importaciones inconsistentes con case-sensitivity (ej. unificación a `errorHandler.js` y `pdfGenerator.js`).
- Creación de componentes SEO, validadores y utilidades faltantes que rompían la build (ej. `SchemaMarkup.jsx`, `RichResultsValidator.jsx`, `PerformanceMonitor.jsx`, `TravelAgencySchema.jsx`).
- Limpieza y eliminación de stubs y rutas obsoletas de cruceros (`CrucerosPage` y `CrucerosSearchPage`).
- Estabilización del workflow de GitHub Actions para despliegue automático a Hostinger (`aliuntravelsrl.com`).

**Gobernanza y Separación de Interfaces**
- Cierre arquitectónico formal de la superficie interna en la web pública (`aliuntravelsrl.com`). Desactivación de `AdminLink.jsx` y desconexión de 29 rutas administrativas de `src/App.jsx` (migradas al subdominio/red dedicada).
- Consolidación de Chatwoot como chat oficial y remoción del widget de Kommo CRM del frontend público.

**Infraestructura y Stacks VPS2 (Rehidratación Swarm)**
- Implementación del script maestro `rehidratacion-auto.sh` en `/opt/data/scripts/` de VPS 2 y configuración de wrappers `rehidratar.sh` en los 4 agentes (`hermes-commercial`, `hermes-marketing`, `ariadne-data`, `hermes-qa`).
- Sincronización y delimitación de tareas en archivos `ALCANCE.md` para cada stack y configuración de 5 cron jobs nativos que garantizan la ejecución de la rehidratación cada 30 minutos.
- Integración automática del estado online en la tabla `personal_ia` de Supabase Cloud.

**Optimización y Telemetría VPS 1**
- Configuración de la restart policy `--restart=no` en el stack local de Supabase y los 4 contenedores legacy de Chatwoot detenidos.
- Despliegue de script de recolección de métricas (disco, RAM, load) apuntando a la tabla `vps_metrics`.
- Modificación del hook `useMarketingKPIs.js` para extraer telemetría conversacional de Chatwoot en tiempo real y renderizado mediante `ChatwootTelemetry.jsx` (Recharts) en la vista de WhatsApp de `App.jsx` y `Sidebar.jsx`.

**Widget de Chat y Registro de Pasarelas**
- Remoción del widget heredado de Kommo CRM (eliminando el componente `KommoWidget.jsx` y depurando `App.jsx`).
- Registro automático del webhook oficial en Chatwoot (ID: 2) para el bot Hermes.
- Cache-Control de Chatwoot configurado con un max-age de 365 días en la cabecera.

**Mesa de Control y Reservas de Excursiones**
- Creación de `AdminExcursionBookingsPanel.jsx` para el monitoreo de reservas.
- Registro de ruta en `AppShell.jsx` y vinculación en el menú lateral de `HorizonsLayout.jsx`.
- Despliegue del compilado `atlas_admin_compilado.zip` en Hostinger y reinicio de la app Passenger.

**Rescate Visual y Enriquecimiento de Hoteles (Health Score 7/7 — 🟢 READY TO SELL)**
Se completó la depuración de placeholders genéricos, inyección de fotos reales (CDN de Booking/Expedia/Scene7) y normalización de datos (habitaciones, temporadas, tarifas 2026, políticas y restaurantes) en Supabase para los siguientes complejos:
1.  **Lopesan Caoba Lagoon Resort, Spa & Casino** (`20f915c7-8e2e-41ab-9ace-fec759b64f51`) — 10 fotos reales (laguna, spa, teatro), 6 servicios (Cenote Lagoon, Panchito Club), 5 restaurantes (Inari, La Bohème), habitaciones y Temporada 2026 activa.
2.  **Be Live Collection Marien** (`332657c0-a75b-4580-8a77-55f058ff01ec`) — 8 fotos oficiales, 16 categorías de habitaciones con interiores del estándar real (Majestic/Catalonia/Meliá), 11 restaurantes/bares oficiales con fotos reales del dominio (Mylos, Oralé, Rodizio) libres de TripAdvisor.
3.  **Sunscape Coco Punta Cana** (`8bbc304d-fb21-4ee2-a78c-bceb90583c4f`) — 12 fotos reales, 9 categorías de habitaciones con precios y fotos del dominio oficial, tarifario 2026 y 13 restaurantes/bares oficiales.
4.  **Sunscape Dominicus La Romana** (`2080f7c9-2ffb-4ff0-9781-d8341b866765`) — 15 fotos de alta resolución, 6 categorías de habitaciones reales con tarifas activas, 15 restaurantes/bares y políticas 2026.
5.  **Bahia Principe Explorer Legend** / *Fantasia Punta Cana* (`2490ce7a-02fe-495c-a9cb-3f0565a4bd2a`) — 21 fotos únicas (sin duplicados CDN), 3 categorías con fotos de interior reales de la web oficial, 5 restaurantes reales (Garden Circus, Il Paradiso) y metadatos de alias de mapeo API correctos.
6.  **Bahia Principe Luxury Esmeralda** (`db00338f-9404-4cbf-a2fb-98479ab47bed`) — 20 fotos reales, 2 categorías de habitaciones (incluyendo Swim Up), 9 restaurantes reales (Naan, Taíno, Yurta) y tarifas 2026.
7.  **Bahia Principe Grand Aquamarine** (`d54473c3-9964-458c-a26c-72a0c1407e10`) — Creado desde cero. 19 fotos únicas, 2 habitaciones oficiales, 5 restaurantes y tarifas 2026.
8.  **Bahia Principe Luxury Ambar** (`c84e6f0d-90e2-4eec-a19a-3bbb9c0ea9e0`) — 20 fotos de alta resolución, 2 habitaciones con fotos del interior, 6 restaurantes reales con descripciones e imágenes reales y tarifas 2026.
9.  **Bahia Principe Grand Punta Cana** (`2db11109-511d-47c2-9ed3-1c31f6392b70`) — 20 fotos desduplicadas (sin hashes del CDN), 2 categorías con fotos interiores reales, 5 restaurantes reales de la propiedad.
10. **Bahia Principe Grand La Romana** (`d0383dcd-47c9-48d8-bed8-c8012c3f9a1a`) — 20 fotos desduplicadas de la hermosa bahía y el resort, 2 categorías reales, 3 restaurantes y tarifas 2026.
11. **Barceló Bávaro Palace** (`2d49fd1f-0de0-4c61-bdd9-f7bdb5542ab5`) — 20 fotos desduplicadas y de alta resolución (Scene7/Expedia), depuración de 17 categorías corruptas antiguas y consolidación de las 8 categorías de habitaciones oficiales con tarifas.
12. **Occidental Caribe** (`20f915c7-8e2e-41ab-9ace-fec759b64f51`) — Identificación de campos extendidos del frontend (cuisine, schedule, location, dress_code). Inyección de fotos gastronómicas oficiales (`OCARI_GAST_`) en Supabase y verificación de 7 restaurantes.
13. **Occidental Punta Cana** — Inyección de 8 habitaciones (10 assets) y 13 restaurantes/bares/snacks (16 assets) con el prefijo `OPCANA_`. Saneamiento y reconstrucción de la galería general en Supabase (libre de Booking y CORS).
14. **Barceló Bávaro Beach (Adults Only)** — Mapeo de 12 habitaciones y 5 restaurantes/bares utilizando Scene7 oficial Barceló con el prefijo `BBAVB_`. Galería saneada y reconstruida libre de CORS y crash de video.
15. **Coral Costa Caribe Beach Resort** (`6bccee79-9f3a-4f68-9873-a9d3f33722b8`) — 9 restaurantes/bares oficiales y 8 habitaciones saneadas con fotos reales interiores de su dominio.
16. **Emotions by Hodelpa Juan Dolio** (`6a8ca8ea-7193-4adf-81ec-a1a423d961fb`) — 10 restaurantes/bares oficiales refinados desde `/restaurantes.html` (Ergo's, Amici, Orégano, Lolita, Bambú, Bígaro, etc.) y 19 habitaciones saneadas con fotos oficiales reales de `media.booking-channel.com` (CDN Hodelpa).


---

## 📋 6. Pendientes críticos — próxima sesión

### P1 — Resolución de Conflicto en `atlas_payments`
Corregir discrepancias entre la estructura plana obsoleta (v1.0 con depósitos 30/70 hardcoded) y la estructura normalizada relacional de producción (con vinculación FK a `bookings.id` y flexibilidad de montos/monedas).

### P2 — Fase Inicial de Adaptadores B2B (Ratehawk Sandbox)
Comenzar la Fase 0/1 del roadmap de B2B: validación de credenciales sandbox de Ratehawk, creación del adaptador de búsqueda en n8n y normalización al esquema común ATLAS.

### P3 — Auditoría de Webhooks e Integraciones Post-Cierre Admin
Inspeccionar webhooks de Kommo y flujos de n8n para asegurar que no intenten invocar endpoints de administración deshabilitados de la web principal y que se redirijan al flujo relacional correspondiente.

### P4 — Continuación de Rescate Visual de Hoteles (Supabase)
Mantener la migración de fotos de hoteles maestros con imágenes rotas hacia el bucket `hotel-media` y actualizar `gallery_urls` en Supabase.

---

## ⚙️ 7. Reglas innegociables

| Regla | Detalle |
|---|---|
| **Supabase** | Verificar registros existentes ANTES de cualquier inyección |
| **Docker** | NUNCA eliminar volúmenes persistentes |
| **Infraestructura** | Todo cambio de red o Docker → documentar en `atlas-cableados` |
| **Tasa de cambio** | Siempre dinámica desde tabla `exchange_rates` en Supabase |
| **Identidad de agentes** | No resolver contradicciones de identidad sin confirmación del Director |
| **Legacy** | Ignorar repos y docs anteriores a este repo. Este es el SSOT de contexto |

---

## 🔗 8. Referencias rápidas

| Recurso | Valor |
|---|---|
| Supabase project | `oyihiyivdhfxpyiwnmqk` |
| Notion War Room root | `346293f4-6b24-811a-9c56-ddf2ce0fd7a7` |
| WF Notificación reserva (Telegram) | chatId `683265740` |
| WF Excursion doc webhook | `aliun-excursion-doc` |
| WF IDs activos | ATLAS-SALES=`0ps4wRmBFXcAy0u2` · COTIZACION=`Da46ZVQGRpdgaI02` · VOUCHER=`XwCARt2qk6U-dvz_RgYy7` |
