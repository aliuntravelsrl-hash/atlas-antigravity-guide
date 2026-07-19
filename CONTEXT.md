# 🧠 CONTEXT — Antigravity (ATLAS Ecosistema)
**Actualizado:** 18 JUL 2026 | SSOT Cognitivo y Técnico
**Última actualización por:** ATLAS-TECH (Claude)

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

- `atlas-booking-frontend-v2` — Web pública de reservas (`aliuntravelsrl.com`).
- `-atlas-admin-v2` — Panel de administración (`atlas.aliuntravelsrl.com`). **⚠️ El nombre tiene prefijo dash en GitHub.**
- `atlas-sales-mcp` — Servidor MCP de herramientas de ventas.
- `atlas-cableados` — Documentación de infraestructura y redes.
- `atlas-marketing-v2` — Motor de ofertas de marketing (en construcción).
- `aliun-rrhh-v2` — Doctrina y SOUL.md de agentes Hermes.
- `musculo-vps` — Scripts y docker-compose de VPS2.
- `atlas-antigravity-guide` — **Este repo** — contexto y guía de Antigravity (SSOT de contexto).

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
- **Fase 1 Rescate Visual:** ✅ Completada (Puerto Plata, Juan Dolio, Boca Chica — ver `WALKTHROUGH_RESCATE_VISUAL_FASE1.md`).
- **Foco:** Continuar rescate visual por fases. Las integraciones de APIs B2B (TBO/Tripsnstay) están en standby hasta la obtención del RNC.
- **Doctrina de Mapeo & Inventario:** Al conectar APIs de proveedores B2B en el futuro, es obligatorio implementar una capa de consolidación y mapeo de inventario (delegada en servicios tipo Giata o Gimmonix, o mediante tablas de equivalencias en Supabase) para evitar duplicidad de propiedades y discrepancias en nombres de habitaciones.
- **Modelo Comercial:** Operar bajo el modelo de *Tarifa Neta (Net Rate) con Markup* autogestionado por el motor mediante el RPC de cotizaciones.

### RAMA 4 — Motor de Excursiones
- **Referencia Notion:** `37b293f46b24804b82b5c0252406da89`
- **Estado:** Fase 5-6 operativa. Fases 3-4 bloqueadas por RNC.
- **Pendiente:** Implementar RPC `funnel_excursiones` en Supabase y validación E2E con la primera reserva real.

### RAMA 5 — Motor de Marketing (Motor de Atracción)
- **Referencia Notion:** `388293f46b2481cf8849c0f2eec84f1b` (ROADMAP Motor Atracción) y `367293f46b248128921cd2497ef86398` (R.E.P.L.I.C.A.)
- **Estado:** Fase 1 al 50%. VPS3 es el cuello de botella actual.
- **Stack:** Publicación directa vía Meta Graph API (Blotato descartado). Generación de ofertas mediante `WF-MKT-GENERATE-OFFER-v3` + Gemini.
- **Pendiente:** Reconectar token Meta (MKT-1) y construir `WF-CONTENT-PUBLISH-v1` (texto + imagen estática).

### RAMA 6 — Canal de Atención & CRM
- **Canal de Atención:** Activo con **Chatwoot** como receptor oficial.
- **CRM Interno (Pipeline Kanban):** Gestionado de forma nativa en `/crm/pipeline` sobre `crm_leads` y `crm_activities` en Supabase. Ver `AUDITORIA_CRM_INTERNO.md`.
- **CRM Dashboard (Métricas y Recibos):** Módulo analítico en `/crm/dashboard` — procesa leads, `crm_deals` y genera recibos PDF. Ver `AUDITORIA_CRM_DASHBOARD.md`.
- **Integridad Relacional CRM-Booking:** `crm_deals.booking_id` es `null` en el 100% de registros de producción — cruce aproximado por texto activo. Ver `AUDITORIA_RELACION_CRM_BOOKING.md`.
- **Vinculación automática (17 JUL):** `AdminBookingsPanel` crea o busca automáticamente en `crm_leads` al guardar datos del cliente de una reserva. Clasificación dinámica de origen (`manual` vs `web_booking`).

### RAMA 7 — Finanzas & Trazabilidad Legal
- **Referencia Notion:** `37f293f46b248113baedfe919cf20e09` y `atlas-payments-v2`
- **Estado:** **CRÍTICO / Desalineado**.
- **Acciones Urgentes:**
  1. Deprecar y eliminar `atlas_payments_flat`.
  2. Implementar nueva tabla relacional `atlas_payments` con FK obligatoria a `bookings.id`.
  3. Crear la vista `public.transactions` para auditoría contable.
  4. **FIN-002 SEV0:** Mover `service_role` key de Supabase a variables de entorno estrictas antes de activar AZUL.

### RAMA 8 — Expansión Futura
- **Foco:** Merchant of Record (MoR) completo, pasarela AZUL, catálogo dinámico B2B post-RNC.
- **IBE de Vuelos & GDS:** Prever campos PNR, segmentos de vuelo y proveedor externo en el modelo de datos.
- **Empaquetamiento Dinámico:** Precios combinados opacos (sin desglose de costo individual).

---

## ⚙️ 5. Reglas Innegociables de Desarrollo

1. **Priorización:** No abrir frentes nuevos si no empujan directamente el cash flow o solucionan un bloqueo operativo inmediato.
2. **Supabase FK:** NUNCA insertar registros en la tabla de pagos que no estén asociados mediante clave foránea a un booking ID existente.
3. **Seguridad (SEV0):** Prohibido almacenar claves `service_role` o tokens de API en repositorios de código o frontend. Deben inyectarse mediante variables de entorno en el backend/n8n.
4. **Higiene de Swarm:** No se permiten crons en los agentes Hermes que no estén explícitamente autorizados en `aliun-rrhh-v2`.
5. **Validación Visual en Sandbox (Crítica):** Antes de cualquier despliegue en producción, todo cambio estructural visual debe ser ejecutado y testeado en el Sandbox. El desarrollador debe mostrar la previsualización al Director y recibir aprobación explícita.
6. **Despliegue de `-atlas-admin-v2` — CI/CD AUTOMÁTICO vía GitHub Actions (actualizado 17 JUL 2026):**
   - **Estado actual:** El deploy es 100% automático. Cualquier `git push` a `main` dispara el workflow `.github/workflows/deploy.yml` que: instala dependencias → build Vite → empaqueta `dist/` en tar.gz → sube por SCP a Hostinger → extrae y aplica permisos → valida el deploy.
   - **NO se debe** compilar localmente ni subir por SCP manualmente — el CI lo hace solo.
   - **Build time:** ~60-75 segundos. Verificar en GitHub Actions antes de concluir que hubo un error.
   - **Force redeploy:** Para forzar un rebuild sin cambio de código, editar `deploy-trigger.txt` en la raíz del repo y hacer push.
   - **Secrets configurados en GitHub Actions:** `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`, `HOSTINGER_SSH_KEY`, `HOSTINGER_PORT`, `HOSTINGER_HOST`, `HOSTINGER_USER`.
   - **Secret FALTANTE:** `VITE_SUPABASE_SERVICE_KEY` — sin esto `supabaseAdmin` opera con anon key (funcional pero sin bypass RLS real). Pendiente agregar en GitHub → Settings → Secrets.
   - **Patrón de pantalla blanca post-deploy:** Si build CI = `success` pero pantalla blanca → es deploy parcial (MIME mismatch). Verificar con `curl -sI https://atlas.aliuntravelsrl.com/assets/index-*.js | grep content-type`. Si responde `text/html` en lugar de `text/javascript` → hacer force redeploy. Ver historial de incidente 17 JUL en `MAPA_GPS.md`.

---

## 📋 6. Estado de Agentes Externos (17 JUL 2026)

| Agente | Estado | Notas |
|--------|--------|-------|
| **Antigravity** | ✅ Activo | Operativo. Sesión 18 Jul completada: OPS-01, OPS-004, B-3, Wyndham Alltra saneado. |
| **Computer (Perplexity)** | ✅ Activo | Cubre incidentes de infraestructura y documentación mientras Antigravity está en timeout. Ficha: `aliun-rrhh-v2/personnel/AP-EXT-01_COMPUTER_PERPLEXITY.md` |
| **ATLAS-TECH (Claude Sonnet 4.6)** | ✅ Activo | Desktop. Orquesta el swarm vía `atlas_tasks`. |

---


---

## 📋 7. Protocolo de Cierre de Sesión — OBLIGATORIO

Al terminar cada sesión, antes de cerrar, ejecutar en Supabase:

### Paso 1 — Marcar tareas completadas
Los 3 campos son OBLIGATORIOS — sin ellos la tarea no cuenta como hecha:

```sql
UPDATE atlas_tasks
SET 
  estado        = 'completado',
  fecha_completado = NOW(),
  resultado     = '[qué se hizo exactamente]',
  cerrado_por   = 'antigravity',
  evidencia_url = '[commit SHA / URL / WF ID que prueba la ejecución]'
WHERE codigo = '[CODIGO]';
```

**Ejemplos de evidencia_url válida:**
- Commit: `https://github.com/aliuntravelsrl-hash/atlas-admin-v2/commit/42d44ccf`
- WF n8n: `https://n8n-n8n.xaruuo.easypanel.host/workflow/X5BnnlKKqgssQUNI`
- CI run: `https://github.com/aliuntravelsrl-hash/atlas-admin-v2/actions`
- Notion walkthrough: URL de la página donde documentaste el trabajo

> ⚠️ Una tarea marcada sin `cerrado_por` y `evidencia_url` se considera
> no verificada. Mission Control no la contará como realmente completada.

### Paso 2 — Registrar sesión en logs_operativos
```sql
INSERT INTO logs_operativos (nivel, origen, evento, mensaje, resuelto)
VALUES ('INFO', 'antigravity', 'SESION_CIERRE',
  '[resumen: qué completaste, qué dejaste pendiente]', true);
```

### Paso 3 — Avisar al Director
Comunicar al Director el resumen de lo completado para que ATLAS-TECH sincronice Mission Control.

> ⚠️ Sin este protocolo, Mission Control no refleja el estado real del sistema y el Director debe sincronizar manualmente.

---

## 📅 8. Schedule de Tareas Activo (18 Jul 2026)

| Código | Tarea | Prioridad | Sprint |
|--------|-------|-----------|--------|
| **AG-001** | Adoptar protocolo de cierre en sesiones | Alta | Esta semana |
| **B-5** | Fix abonos DOP interpretados como USD en BookingOpsPanel | Alta | Esta semana |
| **FIN-003** | Panel conciliación contable en Mission Control | Alta | Sprint-1 |
| **F3-MKT-UI-001** | Marketing + Ariadne datos reales ampliados | Alta | Backlog |
| **I-7** | Botón PDF cotización desde RoomSelection | Baja | Este mes |

*Mantenido por: Director Aldo Hilario | Aliun Travel SRL | República Dominicana*
*Última actualización: ATLAS-TECH — 18 JUL 2026*
