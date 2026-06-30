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

## 📈 5. Estado de avance — 29 JUN 2026

### ✅ Completado esta sesión

**Integración Chatwoot & Hermes**
- Widget Chatwoot implementado en `atlas-booking-frontend-v2/index.html` (commit `3192a3f`)
- Widget Kommo eliminado de `public/index.html`, build compilado y desplegado a Hostinger (commit `05f5e93`)

**Registro manual y métodos de pago**
- `BookingOpsPanel.jsx` sincronizado y desplegado en `atlas.aliuntravelsrl.com`
- Lógica de divisas dinámica: `DO` → DOP locales, internacionales → USD vía `useExchangeRate.js`

**Mesa de Control de Excursiones**
- `AdminExcursionBookingsPanel.jsx` creado con monitoreo en tiempo real
- Ruta `/admin/excursion-bookings` registrada en `AppShell.jsx`
- Sidebar actualizado: "Reservas Excursiones 🌊" y "Reservas Hoteles 🏨"
- Desplegado en Hostinger (commit `6f976f7`)

---

## 📋 6. Pendientes críticos — próxima sesión

### P1 — Pruebas E2E Excursiones
Validar flujo completo: picker de reserva → Mesa de Control → emisión de voucher PDF vía webhook `WF_EXCURSION_DOC`.

### P2 — Actualizar SOUL.md en los 5 stacks VPS2
- **Fuente:** `aliun-rrhh-v2/SOUL_HERMES_OPS_v2.md`
- **Destino:** `/opt/data/atlas-hermes-v2/stacks/{slug}/SOUL.md`
- **Slugs:** `hermes-ops`, `hermes-commercial`, `hermes-marketing`, `ariadne-data`, `hermes-qa`

### P3 — Crear REHIDRATACION.md + ALCANCE.md en 4 stacks
Usar estructura de `hermes-ops` como base. Aplica a: `hermes-commercial`, `hermes-marketing`, `ariadne-data`, `hermes-qa`.

### P4 — Docker restart policy VPS1
Ejecutar `docker update --restart=no` en:
- Stack local Supabase (`n8n_supabase-*`)
- 4 contenedores Chatwoot legacy detenidos

### P5 — Rescate visual de hoteles
Continuar depuración de hoteles con imágenes rotas en Supabase → migrar galerías al bucket `hotel-media` → actualizar `gallery_urls`.

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
