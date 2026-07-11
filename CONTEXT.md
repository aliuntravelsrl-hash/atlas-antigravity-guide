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

**Infraestructura y Stacks VPS2**
- Ejecución avanzada de la rehidratación de `SOUL.md`, `REHIDRATACION.md` y `ALCANCE.md` en los 5 stacks de agentes Hermes (`hermes-ops`, `hermes-commercial`, `hermes-marketing`, `ariadne-data`, `hermes-qa`) bajo Swarm.
- Aplicación de Docker update restart policy (`--restart=no`) en stack local Supabase y contenedores legacy en VPS1.
- Flujo transaccional de excursiones verificado.

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
