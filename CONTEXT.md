# 🧠 CONTEXT — Antigravity (ATLAS Ecosistema)
**Actualizado:** 13 JUL 2026 | SSOT Cognitivo y Técnico

Guía de rehidratación rápida de contexto operativo y memoria de estado para Antigravity al iniciar cualquier sesión en el ecosistema ATLAS.

---

## 🤖 1. Identidad y Rol Operativo

- **Nombre:** Antigravity — Agente Técnico Externo Soberano de ATLAS.
- **Identidad Operativa:** Reporta directamente a Aldo Hilario (Director) e integra las pautas cognitivas de **CEO-2** (Director Operativo IA del ecosistema).
- **Idioma de trabajo:** Español obligatorio.
- **Foco:** Terminal, Docker, repositorios, deploys y cableados de infraestructura.
- **Canal de comunicación oficial:** Chatwoot (Kommo CRM degradado). NUNCA interactúa con clientes finales; es ingeniería pura y operativa.
- **Regla Doctrinal:** Cualquier base de datos local es efímera. **Supabase** es el único cerebro operativo de ATLAS. Toda transacción que no termine reflejada en Supabase es un error crítico.

---

## 🖥️ 2. Infraestructura y Redes

| Componente | Dirección / Detalle | Propósito / Rol |
|---|---|---|
| **Supabase** | Project ID: `oyihiyivdhfxpyiwnmqk` | SSOT absoluto (Cerebro operativo) |
| **n8n** | `https://n8n-n8n.xaruuo.easypanel.host` | Puente de automatizaciones y ruteo |
| **OpenClaw UI** | `https://musculo-aliun.tail02a864.ts.net/` | Interfaz (Acceso exclusivo vía Tailscale VPN) |
| **VPS 1** | `72.61.12.170` | EasyPanel, n8n, Gotenberg y frontends |
| **VPS 2** | `2.24.198.231` | Swarm v3.0 (5 agentes Hermes: ops, commercial, marketing, data, qa) |
| **VPS 3** | *Pendiente Contratar (Boston Hostinger)* | Requerido para stack de ComfyUI + WAN 2.2 |
| **Tailscale IPs** | desktop=`100.102.102.116` · musculo=`100.100.201.7` · note20=`100.110.83.1` | VPN interna segura |

---

## 📂 3. Repositorios bajo responsabilidad de Antigravity

- `atlas-booking-frontend-v2` - Web pública de reservas (`aliuntravelsrl.com`).
- `atlas-admin-v2` - Panel de administración (`atlas.aliuntravelsrl.com`).
- `atlas-sales-mcp` - Servidor MCP de herramientas de ventas.
- `atlas-cableados` - Documentación de infraestructura y redes.
- `atlas-marketing-v2` - Motor de ofertas de marketing (en construcción).
- `aliun-rrhh-v2` - Doctrina y SOUL.md de agentes Hermes.
- `musculo-vps` - Scripts y docker-compose de VPS2.
- `atlas-antigravity-guide` - **Este repo** — contexto y guía de Antigravity (SSOT de contexto).

---

## 🌳 4. Las 8 Ramas del Árbol Maestro de Conceptos (Notion)

### RAMA 1 — Estructura Institucional
- **Referencia Notion:** `37b293f46b2481b6be89d31a754d1f5e` (Visión Enterprise)
- **Estado:** Parcialmente implementada.
- **Foco:** Formalizar a **Ariadne** como fuente financiera oficial (CFO) y definir cobertura del rol legal (CLO).

### RAMA 2 — Stack Técnico (ATLAS Ecosystem)
- **Referencia Notion:** `atlas-cableados` + `RRHH-IA v3.0`
- **Estado:** Operativo con **Swarm v3.0 (Modelo Dispatcher)**.
- **Flujo:** `ATLAS-TECH` orquesta → `atlas_tasks` (autorizado_por) → agentes ejecutan → `logs_operativos` → Director solo atiende escalaciones.
- **Correcciones recientes:** Eliminación del bucle *Telegram Watchdog* en hermes-commercial y purga de 9 crons no autorizados en hermes-ops.

### RAMA 3 — Motor de Hoteles
- **Estado:** Operativo (79 hoteles activos, 25 tarifas manuales).
- **Foco:** Continuar rescate visual por fases. Las integraciones de APIs B2B (TBO/Tripsnstay) están en standby hasta la obtención del RNC.
- **Doctrina de Mapeo & Inventario (Anatomía Notion):** Al conectar APIs de proveedores B2B en el futuro, es obligatorio implementar una capa de consolidación y mapeo de inventario (delegada en servicios tipo Giata o Gimmonix, o mediante tablas de equivalencias en Supabase) para evitar duplicidad de propiedades y discrepancias en nombres de habitaciones (Room Mapping).
- **Modelo Comercial:** Operar bajo el modelo de *Tarifa Neta (Net Rate) con Markup* autogestionado por el motor mediante el RPC de cotizaciones, garantizando el control de márgenes e integridad del pricing.

### RAMA 4 — Motor de Excursiones
- **Referencia Notion:** `37b293f46b24804b82b5c0252406da89`
- **Estado:** Fase 5-6 operativa. Fases 3-4 bloqueadas por RNC.
- **Pendiente:** Implementar RPC `funnel_excursiones` en Supabase y validación E2E con la primera reserva real.

### RAMA 5 — Motor de Marketing (Motor de Atracción)
- **Referencia Notion:** `388293f46b2481cf8849c0f2eec84f1b` (ROADMAP Motor Atracción) y `367293f46b248128921cd2497ef86398` (R.E.P.L.I.C.A.)
- **Estado:** Fase 1 al 50%. VPS3 es el cuello de botella actual.
- **Stack:** Publicación directa vía Meta Graph API (Blotato descartado). Generación de ofertas mediante `WF-MKT-GENERATE-OFFER-v3` + Gemini.
- **Pendiente:** Reconectar token Meta (MKT-1) y construir `WF-CONTENT-PUBLISH-v1` (texto + imagen estática).

### RAMA 6 — Canal de Atención
- **Estado:** Activo con **Chatwoot** como receptor oficial.

### RAMA 7 — Finanzas & Trazabilidad Legal
- **Referencia Notion:** `37f293f46b248113baedfe919cf20e09` (Estrategia Financiera) y `atlas-payments-v2`
- **Estado:** **CRÍTICO / Desalineado**.
- **Discrepancia detectada:** Registros planos en la tabla de pagos antigua (`atlas_payments_flat`) sin FK a `bookings.id`.
- **Doctrina Merchant of Record (MoR) (Anatomía Notion):** Aliun Travel actúa técnicamente como MoR del ecosistema de reservas. Por lo tanto, el "Payment API Handshake" orquestado por n8n debe garantizar trazabilidad financiera fidedigna y vinculación dura a través de claves foráneas con las reservas (`bookings.id`).
- **Acciones Urgentes:**
  1. Deprecar y eliminar `atlas_payments_flat`.
  2. Implementar nueva tabla relacional `atlas_payments` con vinculación de clave foránea obligatoria a `bookings.id` (transacciones sin FK prohibidas).
  3. Crear la vista `public.transactions` para auditoría contable.
  4. **FIN-002 SEV0:** Mover `service_role` key de Supabase a variables de entorno estrictas antes de activar AZUL.

### RAMA 8 — Expansión Futura
- **Foco:** Merchant of Record (MoR) completo, pasarela AZUL, e integración de catálogo dinámico B2B post-RNC.

---

## ⚙️ 5. Reglas Innegociables de Desarrollo

1. **Priorización:** No abrir frentes nuevos si no empujan directamente el cash flow o solucionan un bloqueo operativo inmediato.
2. **Supabase FK:** NUNCA insertar registros en la tabla de pagos que no estén asociados mediante clave foránea a un booking ID existente.
3. **Seguridad (SEV0):** Prohibido almacenar claves `service_role` o tokens de API en repositorios de código o frontend. Deben inyectarse mediante variables de entorno en el backend/n8n.
4. **Higiene de Swarm:** No se permiten crons en los agentes Hermes que no estén explícitamente autorizados en `aliun-rrhh-v2`.
5. **Validación Visual en Sandbox (Crítica):** Antes de cualquier despliegue en producción, todo cambio estructural visual debe ser ejecutado y testeado en el Sandbox. El desarrollador debe mostrar la estructura HTML o previsualización del cambio al Director para evaluar la experiencia visual y recibir su aprobación explícita.

---

*Mantenido por: Director Aldo Hilario | Aliun Travel SRL | República Dominicana*
