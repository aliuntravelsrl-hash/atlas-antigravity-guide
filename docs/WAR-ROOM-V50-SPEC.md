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


---

## ADDENDUM — CORRECCIONES DE COMPUTER
**Fecha:** 25 Jul 2026 | **Estado:** APPROVED WITH CONDITIONS
**Veredicto:** La dirección es correcta. El War Room v5.0 debe evolucionar de dashboard visual a Operational Control Plane. Las correcciones siguientes son obligatorias antes de ejecutar.

---

### CORRECCIÓN 1 🔴 — Cables: Freshness + Integrity (no solo tiempo)

El cable `payment_ledger` y otros NO deben evaluarse solo por `created_at`.
Un cable puede estar fresco pero operacionalmente roto.

**Regla:** `INTEGRIDAD > FRESHNESS`

```javascript
// INCORRECTO (versión original):
getCableStatus(lastTimestamp, expectedIntervalHours)

// CORRECTO (versión aprobada):
function resolveCableStatus({
  lastTimestamp, expectedIntervalHours,
  hasError = false, isConfigured = true, isHealthy = true,
  count = null, min = null
}) {
  if (!isConfigured) return 'gray';
  if (hasError || !isHealthy) return 'red';       // integridad primero
  if (count !== null && min !== null) {
    if (count < min) return 'yellow';
    return 'green';
  }
  if (!lastTimestamp) return 'red';
  const hours = (Date.now() - new Date(lastTimestamp)) / 3600000;
  if (hours <= expectedIntervalHours) return 'green';
  if (hours <= expectedIntervalHours * 2) return 'yellow';
  return 'red';
}
```

**Para payment_ledger:** usar `get_payment_ledger_breakdown()` no solo `MAX(created_at)`.

---

### CORRECCIÓN 2 🔴 — Sistema de semáforos basado en máquina de estados

La función de semáforo original era demasiado simple.
La RPC `get_warroom_task_summary()` ya devuelve `is_healthy`, `hasError`, etc.
El frontend **solo presenta** — el backend **decide**.

```
atlas_tasks / logs_operativos / crm_capi_logs
         ↓
get_warroom_task_summary() ← AUTORIDAD
         ↓
frontend presentation only
```

---

### CORRECCIÓN 3 🟡 — Meta CAPI: umbrales más estrictos

**Aprobado por Director:**
```javascript
// Meta CAPI — medir último envío EXITOSO, no solo cualquier evento
{
  name: 'Meta CAPI',
  expectedIntervalHours: 6,    // 🟢 0–6h
  warningAfterHours: 24,       // 🟡 6–24h
  criticalAfterHours: 48       // 🔴 >24h
}

// Firecrawl Intel — ciclo semanal
{
  name: 'Firecrawl',
  expectedIntervalHours: 168,  // 🟢 dentro del ciclo
  warningAfterHours: 336,      // 🟡 retraso moderado
  criticalAfterHours: 504      // 🔴 ciclo perdido
}

// payment_ledger — freshness + integrity
// Ver corrección #1
```

**Visualmente para Meta CAPI:**
```
🟢 Meta CAPI
Último envío exitoso: hace 12 min
Errores 24h: 0
```

---

### CORRECCIÓN 4 🟡 — Geniall: estado ⚪ hasta health check real

**ELIMINAR** `bookings.created_at` como proxy de Geniall.
Una reserva puede crearse aunque Geniall esté desconectado.

```javascript
// INCORRECTO:
{ name: 'Geniall', table: 'bookings', field: 'created_at', interval: 720 }

// CORRECTO:
{ name: 'Geniall', status: 'gray', reason: 'Sin health check implementado' }
```

**Estado correcto hasta implementar `geniall_sync_events` en logs_operativos:**
```
⚪ Geniall
Sin health check implementado
```

---

### CORRECCIÓN 5 🔴 — Owner Cards: output contractual estable

La RPC `get_warroom_task_summary()` ya está actualizada con este contrato:

```typescript
interface OwnerSummary {
  total_active: number;
  pending: number;
  in_progress: number;
  blocked: number;
  critical: number;
  high: number;
  medium: number;
  health: 'red' | 'yellow' | 'green';
}
```

**Tarjeta UI:**
```
DIRECTOR — 14 tareas activas
🔴 1 crítica
⚠️ 10 altas
⏳ 3 normales
──────────────
9 pendientes · 3 en progreso · 2 bloqueadas
[Ver detalle →] → drawer lateral
```

**Click → drawer** (no expandir inline):
Filtros en drawer: [Todas] [Críticas] [Altas] [Bloqueadas] [En progreso]

---

### CORRECCIÓN 6 🟡 — loadAll() sin solapamiento de requests

```javascript
// INCORRECTO (puede crear requests superpuestos):
useEffect(() => {
  loadAll();
  const interval = setInterval(loadAll, 60000);
  return () => clearInterval(interval);
}, []);

// CORRECTO (con lock):
const loadingRef = useRef(false);

const loadAll = async () => {
  if (loadingRef.current) return;
  loadingRef.current = true;
  try {
    await Promise.all([
      loadTaskSummary(),
      loadLogs(),
      loadCables(),
      loadAgentReports(),
      loadSsotHealth()
    ]);
    setLastRefresh(new Date());
  } finally {
    loadingRef.current = false;
  }
};
```

---

### DISEÑO FINAL APROBADO (Computer)

```
┌────────────────────────────────────────────────────────────┐
│ WAR ROOM v5.0                              🟢 HEALTHY      │
│ Última actualización: hace 12 segundos                     │
└────────────────────────────────────────────────────────────┘

┌──────────────┬──────────────┬──────────────┬──────────────┐
│ DIRECTOR (A) │ ATLAS-TECH(B)│   SWARM  (C) │ ANTIGRAV.(D) │
│ 14 activas   │ 20 activas   │ 23 activas   │ 7 activas    │
│ 🔴 1 crítica │ 🔴 1 crítica │ 🟢 0         │ 🟢 0         │
│ [Ver →]      │ [Ver →]      │ [Ver →]      │ [Ver →]      │
└──────────────┴──────────────┴──────────────┴──────────────┘

┌─────────────────────────┬────────────────────────────────┐
│ CABLES / INTEGRACIONES  │ HERMES REPORTER                │
│ 🟢 Meta CAPI (12 min)   │ 🤖 hermes-ops  · hace 2h      │
│ 🟢 Firecrawl (viernes)  │ 🔍 hermes-qa   · hace 8h      │
│ 🟢 payment_ledger       │ 💼 commercial  · hace 1h      │
│ 🟡 hotel_knowledge      │                                │
│ ⚪ Geniall (sin HC)     │ [Ver historial →]              │
│ ⚪ TBO Holidays         │                                │
└─────────────────────────┴────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ LIVE OPERATIONAL LOG                                       │
│ [Todos] [INFO] [WARN] [ERROR] [CRITICAL]  [origen ▼]     │
│ 15:42 🟢 hermes-ops   Evento completado                   │
│ 15:40 ⚠️  atlas-tech  Migración pendiente                 │
│ 15:38 🔴 hermes-qa    Test failed                         │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ SSOT HEALTH  (mantener igual que v4.1)                     │
└────────────────────────────────────────────────────────────┘
```

---

### CRITERIOS DE ACEPTACIÓN ACTUALIZADOS

- [ ] Owner cards leen de `get_warroom_task_summary()` — NO calculan en frontend
- [ ] Click owner card → drawer lateral con lista filtrable de tareas
- [ ] Semáforos usan `resolveCableStatus()` con integridad > freshness
- [ ] Meta CAPI mide último envío EXITOSO (no cualquier registro)
- [ ] Geniall muestra ⚪ gris (sin health check real)
- [ ] `loadAll()` usa `useRef` lock — sin solapamiento de requests
- [ ] Logs son reales de `logs_operativos` (no hardcodeados)
- [ ] Auto-refresh cada 60s sin parpadeo
- [ ] SSOT Health mantiene funcionamiento de v4.1 (no regresión)
- [ ] Hermes Reporter muestra actividad de últimas 24h por agente
- [ ] Botón "Ver historial →" en Hermes Reporter (vista filtrada de logs)

---

### DECISIONES DEL DIRECTOR (confirmadas)

| Cable | 🟢 | 🟡 | 🔴 |
|-------|-----|-----|-----|
| Meta CAPI | 0–6h exitoso | 6–24h | >24h |
| Firecrawl | dentro del ciclo semanal | retraso moderado | ciclo perdido |
| payment_ledger | freshness + integrity OK | warning | error |
| Geniall | — | — | ⚪ hasta HC real |

*ATLAS-TECH · WAR-ROOM-V50-SPEC v1.1 · Correcciones Computer aplicadas · 25 Jul 2026*
