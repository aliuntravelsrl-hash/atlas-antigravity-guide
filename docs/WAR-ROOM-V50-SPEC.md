# SPEC: War Room v5.0 — Sistema de Reportes ATLAS
**Versión:** 1.0 | **Fecha:** 25 Jul 2026
**Propietario:** Director Aldo Hilario
**Ejecutor:** Antigravity (frontend) + ATLAS-TECH (backend ✅ listo)
**Componente actual:** `src/components/marketing/WarRoomV41.jsx`
**Ruta:** `/warroom`
**Estado backend:** ✅ RPCs creadas y listas

---

## CONTEXTO — EL PROBLEMA ACTUAL

El War Room v4.1 tiene:
- Logs **hardcodeados** en `useState([])` → no reflejan realidad
- Engines **hardcodeados** → no conectados a n8n ni Supabase
- Sin visión de los 4 owners del ecosistema
- Hermes Ops y Hermes-QA reportan a `logs_operativos` pero esos datos nunca llegan al panel

**El Director necesita:** abrir `/warroom` y ver en 5 segundos si el ecosistema está sano.

---

## ARQUITECTURA DE REPORTES — 4 OWNERS

```
DIRECTOR      (A) — decisiones, contratos, credenciales
ATLAS-TECH    (B) — BD, RPCs, WFs, arquitectura
SWARM         (C) — Hermes Ops, QA, Commercial, Marketing, Ariadne, Intel
ANTIGRAVITY   (D) — frontend, repos, builds, UI
```

### Flujo de reportes de los agentes:

```
Hermes Ops    → logs_operativos (origen: 'hermes-ops')
Hermes QA     → logs_operativos (origen: 'hermes-qa')  
               + WF-HERMES-QA-BRIEFING-DIARIO (Telegram 8AM)
Hermes Comm.  → logs_operativos (origen: 'hermes-commercial')
ATLAS-TECH    → logs_operativos (origen: 'atlas-tech')
n8n           → logs_operativos (origen: 'wf-*')

TODOS → logs_operativos → War Room v5.0 (tiempo real)
```

---

## DISEÑO DEL WAR ROOM v5.0

### SECCIÓN 1 — HEADER (mantener estilo v4.1)
```
War Room v5.0  [🟢 ECOSISTEMA SANO / 🔴 N ALERTAS]
Centro de monitoreo operacional — refresh automático cada 60s
[Última actualización: hace X segundos]
```

### SECCIÓN 2 — 4 TARJETAS OWNER (NUEVO)
```
Fuente: get_warroom_task_summary() → RPC Supabase ✅

┌─────────────────┐ ┌─────────────────┐
│   DIRECTOR  (A) │ │  ATLAS-TECH (B) │
│   14 tareas     │ │   20 tareas     │
│   🔴 1 crítica  │ │   🔴 1 crítica  │
│   ⚠️ 10 altas   │ │   ⚠️ 15 altas   │
└─────────────────┘ └─────────────────┘
┌─────────────────┐ ┌─────────────────┐
│    SWARM    (C) │ │ ANTIGRAVITY (D) │
│   23 tareas     │ │    7 tareas     │
│   ⚪ 0 críticas │ │   ⚪ 0 críticas │
│   ⚠️ 11 altas   │ │   ⚠️ 5 altas    │
└─────────────────┘ └─────────────────┘

Click en tarjeta → expande lista de tareas del owner
```

### SECCIÓN 3 — CABLES / INTEGRACIONES (NUEVO)
```
Fuente: consultas directas a Supabase

Cuadrícula de luces 2×N:
┌─────────────────────────┬─────────────────────────┐
│ 🟢 Meta CAPI            │ 🔴 Google CAPI (gclid)  │
│ último: hace 5 min      │ pendiente activar        │
├─────────────────────────┼─────────────────────────┤
│ 🟢 Firecrawl Intel      │ 🟢 WF-QA Briefing 8AM   │
│ viernes 9AM             │ diario activo            │
├─────────────────────────┼─────────────────────────┤
│ 🟢 payment_ledger       │ 🟢 hotel_knowledge       │
│ último: hace 2h         │ 150 entradas activas     │
├─────────────────────────┼─────────────────────────┤
│ 🟡 Geniall proveedor    │ ⚪ TBO Holidays           │
│ último: 15 Mar 2025     │ pendiente onboarding     │
├─────────────────────────┼─────────────────────────┤
│ 🟢 Chatwoot             │ 🟢 n8n                   │
│ conectado               │ 14 WFs activos           │
└─────────────────────────┴─────────────────────────┘

Lógica de semáforos:
  🟢 = dato recibido en período esperado
  🟡 = dato antiguo (>umbral configurable)
  🔴 = nunca enviado / error / pendiente activar
  ⚪ = no aplica aún / pendiente configurar
```

### SECCIÓN 4 — LOG EN TIEMPO REAL (reemplazar hardcode)
```
Fuente: supabase
  .from('logs_operativos')
  .select('nivel, origen, evento, mensaje, created_at')
  .order('created_at', { ascending: false })
  .limit(50)

Filtros del usuario:
  [Todos] [INFO] [WARN] [ERROR] [CRITICAL]
  [Todos] [atlas-tech] [hermes-ops] [hermes-qa] [hermes-commercial] [swarm]

Cada log row:
  [timestamp] [badge nivel] [origen] mensaje
  CRITICAL → fondo rojo suave
  ERROR    → fondo amber suave  
  WARN     → borde amber
  INFO     → normal
  SUCCESS  → borde emerald
```

### SECCIÓN 5 — SSOT HEALTH (mantener ✅ ya funciona)
```
Mantener igual que v4.1:
  Hoteles activos · Habitaciones · Tarifas · Temporadas
  RAG Context Cache
Solo añadir: crm_leads total · payment_ledger entries
```

### SECCIÓN 6 — HERMES REPORTER (NUEVO — panel derecho)
```
Fuente: logs_operativos WHERE created_at > NOW() - INTERVAL '24h'
Agrupado por: origen

Mini-card por agente activo:
┌──────────────────────────────────┐
│ 🤖 hermes-ops                   │
│ Último reporte: hace 2h         │
│ Reportes hoy: 12                │
│ Errores: 0 🟢                   │
└──────────────────────────────────┘
┌──────────────────────────────────┐
│ 🔍 hermes-qa                    │
│ Último: hace 8h (briefing 8AM)  │
│ Reportes hoy: 3                 │
│ Errores: 0 🟢                   │
└──────────────────────────────────┘
```

---

## RPCS BACKEND ✅ LISTAS (Antigravity solo consume)

| RPC | Qué devuelve |
|-----|-------------|
| `get_warroom_task_summary()` | 4 owners · cuellos · total pendiente |
| `get_payment_ledger_breakdown()` | estado contable |
| `get_meta_catalog_feed()` | 91 hoteles para catálogo |

**Tabla directa para logs:**
```javascript
// Antigravity usa esto directo — no hay RPC adicional
const { data } = await supabase
  .from('logs_operativos')
  .select('nivel, origen, evento, mensaje, created_at, resuelto')
  .order('created_at', { ascending: false })
  .limit(50);
```

**Tabla directa para cables:**
```javascript
// Cable: Meta CAPI
const { data: capiLog } = await supabase
  .from('crm_capi_logs')
  .select('sent_at, status')
  .order('sent_at', { ascending: false })
  .limit(1);

// Cable: Firecrawl Intel
const { data: intel } = await supabase
  .from('competitive_intel')
  .select('scrapeado_at')
  .order('scrapeado_at', { ascending: false })
  .limit(1);

// Cable: payment_ledger
const { data: ledger } = await supabase
  .from('payment_ledger')
  .select('created_at')
  .order('created_at', { ascending: false })
  .limit(1);

// Cable: hotel_knowledge
const { count: hkCount } = await supabase
  .from('hotel_knowledge')
  .select('id', { count: 'exact', head: true })
  .eq('activo', true);

// Cable: n8n WFs activos (estático por ahora — pendiente endpoint)
// Valor: 14 WFs activos (actualizar manualmente hasta tener API)
```

---

## LÓGICA DE SEMÁFOROS

```javascript
function getCableStatus(lastTimestamp, expectedIntervalHours) {
  if (!lastTimestamp) return 'red';        // nunca recibido
  const hours = (Date.now() - new Date(lastTimestamp)) / 3600000;
  if (hours <= expectedIntervalHours) return 'green';
  if (hours <= expectedIntervalHours * 2) return 'yellow';
  return 'red';
}

// Configuración por cable:
cables = [
  { name: 'Meta CAPI',      table: 'crm_capi_logs',    field: 'sent_at',        interval: 24   },
  { name: 'Firecrawl',      table: 'competitive_intel', field: 'scrapeado_at',   interval: 168  }, // semanal
  { name: 'payment_ledger', table: 'payment_ledger',   field: 'created_at',     interval: 168  },
  { name: 'Hotel Knowledge',table: 'hotel_knowledge',  field: null, count: true, min: 150       },
  { name: 'Geniall',        table: 'bookings',         field: 'created_at',     interval: 720  }, // mensual
]
```

---

## INSTRUCCIONES TÉCNICAS PARA ANTIGRAVITY

### 1. Renombrar/duplicar componente
```
WarRoomV41.jsx → WarRoomV50.jsx
Actualizar import en la ruta /warroom
```

### 2. Estructura de estados
```javascript
const [taskSummary, setTaskSummary] = useState(null);
const [logs, setLogs] = useState([]);
const [cables, setCables] = useState([]);
const [agentReports, setAgentReports] = useState([]);
const [ssotHealth, setSsotHealth] = useState({...}); // mantener
const [lastRefresh, setLastRefresh] = useState(null);
const [logFilter, setLogFilter] = useState({ nivel: 'all', origen: 'all' });
```

### 3. Auto-refresh
```javascript
useEffect(() => {
  loadAll();
  const interval = setInterval(loadAll, 60000); // 60 segundos
  return () => clearInterval(interval);
}, []);
```

### 4. Reglas de diseño
- Mantener paleta dark (slate-950/900/800) del v4.1 — NO cambiar colores base
- Badge owner_type: director=blue · atlas-tech=violet · swarm=emerald · antigravity=amber
- Logs CRITICAL → `bg-red-950/30 border-red-800`
- Logs ERROR → `bg-amber-950/30 border-amber-800`
- Semáforo verde → `text-emerald-400` | amarillo → `text-amber-400` | rojo → `text-red-400`
- Refresh indicator: dot pulsante en el header

### 5. Rutas — NO TOCAR
```
/warroom → solo se actualiza el componente interno
Las rutas del menú sidebar no se modifican en este sprint
```

---

## CRITERIOS DE ACEPTACIÓN (QA)

- [ ] Los 4 owner cards muestran datos reales de atlas_tasks
- [ ] Los logs son reales de logs_operativos (no hardcodeados)
- [ ] Filtro por nivel y origen funciona sin recargar página
- [ ] Cada cable tiene semáforo correcto según su umbral
- [ ] Auto-refresh cada 60s sin parpadeo de pantalla
- [ ] SSOT Health sigue funcionando (no regresión)
- [ ] Agente reporter muestra actividad de las últimas 24h

---

## PENDIENTE PARA COMPUTER (antes de ejecutar)

**Validar:**
1. ¿Los umbrales de los semáforos son correctos?
   - Meta CAPI: ¿cuándo el verde se vuelve amarillo?
   - Geniall: ¿mensual o por reserva?
   
2. ¿El panel de Hermes Reporter debe tener botón "Ver historial completo"?

3. ¿Las tarjetas de owner son clickeables para ver el detalle de tareas?

---

*ATLAS-TECH · War Room v5.0 Spec · 25 Jul 2026*
*Conecta con: WarRoomV41.jsx · logs_operativos · atlas_tasks · crm_capi_logs*
