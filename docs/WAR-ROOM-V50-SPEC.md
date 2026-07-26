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


---

## ADDENDUM 2 — SEGUNDA REVISIÓN COMPUTER
**Fecha:** 25 Jul 2026 | **Estado:** SPEC-003-WARROOM-V5 — APPROVED WITH REQUIRED CORRECTIONS
**Orden de implementación validado por Computer (seguir este orden exacto)**

---

### ORDEN DE IMPLEMENTACIÓN (Computer — obligatorio)

```
1. Backend contract verification   ← RPC v3 ✅ ya actualizada
2. RPC get_warroom_task_summary()  ← ✅ v3 en Supabase
3. Real logs_operativos            ← Antigravity conecta
4. Cable health abstraction        ← resolveCableStatus()
5. Atomic refresh engine           ← useRef lock + swap atómico
6. Owner cards                     ← expandibles inline (lazy)
7. Live logs + filters             ← nivel + origen
8. SSOT health                     ← mantener v4.1
9. Hermes Reporter                 ← "Eventos registrados hoy"
10. QA acceptance tests            ← criterios de aceptación
```

---

### CORRECCIÓN A — Owner Cards: lazy-load, NO drawer

**Computer aprobó expansión inline (no drawer lateral):**

```
CERRADO:
┌──────────────────────────┐
│ ATLAS-TECH           (B) │
│ 20 tareas activas        │
│ 🔴 1 crítica             │
│ ⚠️ 15 altas              │
│ [Click para expandir ▼]  │
└──────────────────────────┘

EXPANDIDO (lazy-load al click):
┌──────────────────────────────────────────┐
│ ATLAS-TECH                           ▲  │
│ 20 tareas activas                        │
├──────────────────────────────────────────┤
│ 🔴 ATL-XXX  Migración ledger  CRÍTICA   │
│ ⚠️  ATL-YYY  Validar RPC       ALTA     │
│ ⚠️  ATL-ZZZ  Auditoría         ALTA     │
│                                          │
│ [Ver todas las tareas →]                 │
└──────────────────────────────────────────┘
```

**La RPC ya devuelve `top_tasks` (5 más importantes por owner).**
Para "Ver todas" → consulta lazy directa a Supabase:
```javascript
// Solo al hacer click "Ver todas"
const { data } = await supabase
  .from('atlas_tasks')
  .select('id, titulo, prioridad, estado, codigo')
  .eq('owner_type', ownerType)
  .in('estado', ['pendiente', 'en_progreso'])
  .order('prioridad')  // critica primero
  .limit(50);
```

---

### CORRECCIÓN B — Semáforos: health funcional ≠ freshness temporal

**Regla definitiva de Computer:**

```
HEALTH = estado funcional (¿el sistema falla?)
FRESHNESS = antigüedad del último dato (¿cuándo fue?)

Semáforo final = peor de ambos
```

**Tabla de comportamiento:**

| Último dato | Status | Resultado |
|-------------|--------|-----------|
| Hace 5 min  | success | 🟢 |
| Hace 5 min  | failed  | 🔴 |
| Hace 30h    | success | 🟡 |
| Hace 60h    | success | 🔴 |
| Nunca       | —       | 🔴 |

**Función resolveCableStatus() FINAL:**
```javascript
function resolveCableStatus({
  lastTimestamp,        // timestamp del último dato
  lastSuccessAt,        // timestamp del último ÉXITO
  expectedIntervalHours,
  warningAfterHours,
  criticalAfterHours,
  hasError = false,
  isConfigured = true,
  isHealthy = true,
  count = null,
  min = null
}) {
  if (!isConfigured) return 'gray';
  if (hasError || !isHealthy) return 'red';  // health primero
  
  // count-based (hotel_knowledge)
  if (count !== null && min !== null) {
    return count >= min ? 'green' : 'yellow';
  }
  
  const ts = lastSuccessAt || lastTimestamp;
  if (!ts) return 'red';
  
  const hours = (Date.now() - new Date(ts)) / 3600000;
  if (hours <= (warningAfterHours || expectedIntervalHours)) return 'green';
  if (hours <= (criticalAfterHours || expectedIntervalHours * 2)) return 'yellow';
  return 'red';
}
```

**Umbrales aprobados por Director:**

```javascript
const CABLE_CONFIG = {
  meta_capi: {
    warningAfterHours: 24,
    criticalAfterHours: 48,
    // usa lastSuccessAt (no sent_at genérico)
    // hasError = errors_24h > 0
  },
  firecrawl: {
    warningAfterHours: 168,   // 7 días (ciclo semanal)
    criticalAfterHours: 336,  // 14 días
  },
  payment_ledger: {
    // NO usar solo created_at
    // is_healthy viene de get_payment_ledger_breakdown()
    // integrity = 'OK' → green si freshness OK
  },
  hotel_knowledge: {
    count: activeEntries,
    min: 150
  },
  geniall: {
    isConfigured: false  // ⚪ hasta supplier_api_logs
  },
  tbo_holidays: {
    isConfigured: false  // ⚪ hasta onboarding
  }
}
```

---

### CORRECCIÓN C — Atomic refresh (sin parpadeo)

**NO hacer esto (causa parpadeo):**
```javascript
// ❌ MAL
const loadAll = async () => {
  setLogs([]);           // ← parpadeo
  setCables([]);         // ← parpadeo
  setTaskSummary(null);  // ← parpadeo
  // fetch...
};
```

**Hacer esto (swap atómico):**
```javascript
// ✅ CORRECTO
const refreshInFlight = useRef(false);

const loadAll = async () => {
  if (refreshInFlight.current) return;  // lock
  refreshInFlight.current = true;
  setIsRefreshing(true);  // solo el dot pulsante

  try {
    // Fetch en paralelo
    const [taskData, logsData, cablesData, agentsData, ssotData] = 
      await Promise.all([
        loadTaskSummary(),
        loadLogs(),
        loadCables(),
        loadAgentReports(),
        loadSsotHealth()
      ]);

    // Swap atómico — todo junto
    setTaskSummary(taskData);
    setLogs(logsData);
    setCables(cablesData);
    setAgentReports(agentsData);
    setSsotHealth(ssotData);
    setLastRefresh(new Date());

  } catch (err) {
    // En error: mantener estado anterior, solo mostrar badge
    console.error('War Room refresh failed:', err);
  } finally {
    refreshInFlight.current = false;
    setIsRefreshing(false);
  }
};
```

---

### CORRECCIÓN D — Hermes Reporter: "Eventos" no "Reportes"

```
❌ ANTES:          ✅ AHORA:
Reportes hoy: 42  Eventos registrados hoy: 42
```

**Un reporte puede generar múltiples eventos en logs_operativos.**
No mezclar conceptos. La tabla muestra `logs_operativos` → son eventos.

**Mini-card por agente:**
```
┌──────────────────────────────────────┐
│ 🤖 hermes-ops                       │
│ Último evento: hace 2h              │
│ Eventos hoy: 42                     │  ← NO "reportes"
│ Errores: 0 🟢                       │
│ [Ver historial →]                   │
└──────────────────────────────────────┘
```

**[Ver historial →] → drawer lateral (NO nueva ruta):**
```javascript
// Drawer: logs filtrados por agente
const { data } = await supabase
  .from('logs_operativos')
  .select('nivel, evento, mensaje, created_at')
  .eq('origen', agentName)
  .order('created_at', { ascending: false })
  .limit(50);
// Filtros en drawer: 24h / 7d / 30d
```

---

### CORRECCIÓN E — payment_ledger: usar get_payment_ledger_breakdown()

```javascript
// ❌ INCORRECTO:
const { data } = await supabase
  .from('payment_ledger')
  .select('created_at')
  .order('created_at', { ascending: false })
  .limit(1);

// ✅ CORRECTO:
const { data } = await supabase
  .rpc('get_payment_ledger_breakdown');

// Lógica:
const ledgerHealthy = data !== null &&
  data.length > 0 &&
  !data.some(p => p.application_status === 'error');
```

**Semáforo payment_ledger:**
```
🟢 = RPC responde + sin anomalías
🟡 = RPC responde + hay aplicaciones en estado anómalo
🔴 = RPC falla o errores críticos de integridad
```

---

### CRITERIOS DE ACEPTACIÓN v1.2 (finales)

- [ ] Owner cards muestran `top_tasks` del RPC — NO calculan en frontend
- [ ] Click owner card → expande inline (lazy-load, no drawer)
- [ ] "Ver todas las tareas" → consulta directa a `atlas_tasks`
- [ ] Semáforos usan `resolveCableStatus()` con health > freshness
- [ ] Meta CAPI evalúa `last_success_at` + `has_error` (no solo timestamp)
- [ ] payment_ledger usa `get_payment_ledger_breakdown()` no `created_at`
- [ ] Geniall muestra ⚪ gris + "Telemetría no disponible"
- [ ] TBO Holidays muestra ⚪ gris + "Pendiente onboarding"
- [ ] loadAll() usa useRef lock + swap atómico (sin nullear estado previo)
- [ ] Indicador "Actualizando" muestra dot pulsante — NO blank state
- [ ] Hermes Reporter dice "Eventos registrados hoy" (no "Reportes")
- [ ] Ver historial → drawer lateral en la misma página (no nueva ruta)
- [ ] Logs son reales de `logs_operativos` ORDER BY created_at DESC
- [ ] Filtros de log: nivel + origen (sin recargar página)
- [ ] Auto-refresh cada 60s sin parpadeo de contenido
- [ ] SSOT Health mantiene v4.1 (no regresión)

---

*ATLAS-TECH · WAR-ROOM-V50-SPEC v1.2 · Segunda revisión Computer aplicada · 25 Jul 2026*
*Veredicto final: APPROVED — ejecutable con correcciones aplicadas*


---

## ADDENDUM 3 — TERCERA REVISIÓN COMPUTER (FINAL)
**Estado:** ATLAS-SDD SPEC-WR-005 v1.3 — APPROVED FOR EXECUTION
**Convergencia de 3 revisiones: todos los puntos críticos resueltos**

---

### CORRECCIÓN F — Estado global: lógica explícita RED/YELLOW/GREEN

**NO hacer esto:**
```javascript
// ❌ MAL — un log INFO no es una alerta
const globalStatus = logs.length > 0 ? 'red' : 'green';
```

**Hacer esto:**
```javascript
// ✅ CORRECTO — lógica explícita
function resolveGlobalStatus({ owners, cables, logs }) {
  // RED si:
  const hasCritical =
    logs.some(l => l.nivel === 'CRITICAL' && !l.resuelto) ||
    Object.values(cables).some(c => c.status === 'red' && c.is_configured) ||
    owners.some(o => o.health === 'red' && o.critical > 0);

  if (hasCritical) return { status: 'red', label: 'REQUIERE ATENCIÓN' };

  // YELLOW si:
  const hasWarning =
    logs.some(l => l.nivel === 'ERROR' && !l.resuelto) ||
    Object.values(cables).some(c => c.status === 'yellow') ||
    owners.some(o => o.blocked > 0);

  if (hasWarning) return { status: 'yellow', label: 'ADVERTENCIAS ACTIVAS' };

  // GREEN:
  return { status: 'green', label: 'ECOSISTEMA OPERATIVO' };
}
```

**Header resultado:**
```
🟢 ECOSISTEMA OPERATIVO
Última actualización: hace 12s · 0 críticas · 2 warnings · 4 cables sanos

— o —

🔴 REQUIERE ATENCIÓN
2 alertas críticas · 1 cable caído · [Ver detalles ↓]
```

---

### CORRECCIÓN G — Orden visual FINAL (aprobado por Computer)

```
1. HEADER GLOBAL + STATUS
   ↓
2. ALERTAS CRÍTICAS (solo si existen — conditional render)
   🔴 N ALERTAS REQUIEREN ATENCIÓN
   [lista compacta de qué está mal]
   ↓
3. OWNER CARDS (A · B · C · D)
   ↓
4. CABLES / INTEGRACIONES
   ↓
5. SSOT HEALTH (v4.1 intacto)
   ↓
6. LIVE LOG (logs_operativos)
   ↓
7. HERMES REPORTER
```

**Razón:** El Director necesita primero "¿qué requiere mi atención?" antes de ver métricas.

---

### CORRECCIÓN H — Owner Card: badge "CUELLO ACTIVO"

Cuando un owner tiene tareas bloqueadas que frenan a otros:

```
┌─────────────────────────────┐
│ DIRECTOR (A)                │
│ 14 pendientes               │
│ 🔴 1 crítica                │
│ ⚠️ 10 altas                 │
│ ⏳ 3 bloqueadas             │
│                             │
│ 🔴 CUELLO ACTIVO            │  ← si blocked > 0
│ [Ver tareas →]              │
└─────────────────────────────┘
```

**La RPC ya devuelve `blocked` por owner.**
Mostrar el badge si `blocked > 0`:
```javascript
{owner.blocked > 0 && (
  <span className="badge-red">🔴 CUELLO ACTIVO</span>
)}
```

---

### TABLA DE CABLES FINAL (aprobada)

| Cable | Fuente | Tipo de salud |
|-------|--------|---------------|
| Meta CAPI | `crm_capi_logs` | Último éxito + tasa de error |
| Firecrawl Intel | `competitive_intel` | Freshness semanal |
| WF QA Briefing | `logs_operativos` | Heartbeat diario |
| Payment Ledger | `get_payment_ledger_breakdown()` | Integridad financiera |
| Hotel Knowledge | `hotel_knowledge` | Conteo ≥ 150 activas |
| Geniall | — | ⚪ Telemetría no disponible |
| TBO Holidays | — | ⚪ Pendiente onboarding (RNC) |
| n8n | `logs_operativos` origen='wf-*' | Workflows activos |

---

### CONVERGENCIA DE 3 REVISIONES — PUNTOS CLAVE

Todos los puntos críticos están resueltos. Resumen ejecutivo:

| Punto | Rev 1 | Rev 2 | Rev 3 | Estado |
|-------|-------|-------|-------|--------|
| health ≠ freshness | 🔴 | 🔴 | 🔴 | ✅ resuelto |
| Geniall ⚪ | 🔴 | 🔴 | 🔴 | ✅ resuelto |
| payment_ledger RPC | 🔴 | 🔴 | 🔴 | ✅ resuelto |
| loadAll() lock | 🟡 | 🟡 | 🟡 | ✅ resuelto |
| Owner cards clickeables | 🟡 | 🟡 | 🟡 | ✅ resuelto |
| Hermes Reporter semántica | 🟡 | 🟡 | 🟡 | ✅ resuelto |
| Global status explícito | — | — | 🔴 | ✅ añadido |
| Orden visual | — | — | 🟡 | ✅ añadido |
| Badge cuello activo | — | — | 🟡 | ✅ añadido |

---

### CRITERIOS DE ACEPTACIÓN v1.3 (FINALES)

**Estado global:**
- [ ] `resolveGlobalStatus()` usa lógica explícita RED/YELLOW/GREEN
- [ ] RED solo si: CRITICAL no resuelto | cable crítico caído | owner critical > 0
- [ ] Header muestra: estado + timestamp + resumen compacto

**Orden visual:**
- [ ] Sección "Alertas críticas" se muestra solo si existen (conditional render)
- [ ] Orden: Header → Alertas → Owners → Cables → SSOT → Logs → Hermes Reporter

**Owner Cards:**
- [ ] Badge "🔴 CUELLO ACTIVO" si `blocked > 0`
- [ ] top_tasks expandibles inline (lazy del RPC)
- [ ] "Ver todas" → consulta directa atlas_tasks

**Cables:**
- [ ] `resolveCableStatus()` con health > freshness
- [ ] Meta CAPI: last_success_at + errors_24h
- [ ] payment_ledger: `get_payment_ledger_breakdown()` integridad
- [ ] Geniall: ⚪ gris + "Telemetría no disponible"
- [ ] TBO Holidays: ⚪ gris + "Pendiente onboarding"

**Refresh:**
- [ ] `useRef` lock — sin solapamiento
- [ ] Swap atómico — sin nullear estado previo
- [ ] Indicador dot pulsante (no blank state)

**Logs:**
- [ ] Fuente real: `logs_operativos` ORDER BY created_at DESC LIMIT 50
- [ ] Filtros: nivel + origen (sin reload)

**Hermes Reporter:**
- [ ] "Eventos registrados hoy" (no "Reportes")
- [ ] Mini-cards: events_24h + last_event + errors
- [ ] [Ver historial →] → drawer lateral misma página

**SSOT Health:**
- [ ] Mantener v4.1 sin regresión

---

*ATLAS-TECH · WAR-ROOM-V50-SPEC v1.3 FINAL · 3 revisiones Computer aplicadas · 25 Jul 2026*
*Veredicto: APPROVED FOR EXECUTION — entregar a Antigravity*
*Componente: WarRoomV41.jsx → WarRoomV50.jsx | Ruta: /warroom*


---

## ADDENDUM 4 — CUARTA REVISIÓN COMPUTER (PUNTOS NUEVOS)
**Estado:** ATLAS-SDD SPEC-WARROOM-005 — APPROVED WITH REQUIRED CORRECTIONS
**Nota:** Las 6 correcciones críticas ya están en v1.3. Solo 3 puntos nuevos.

---

### NUEVO 1 — Promise.allSettled() en lugar de Promise.all()

```javascript
// ❌ Promise.all() — si un request falla, TODA la pantalla falla
const results = await Promise.all([...]);

// ✅ Promise.allSettled() — cada sección maneja su propio error
const snapshot = await Promise.allSettled([
  loadTaskSummary(),
  loadLogs(),
  loadCables(),
  loadAgentReports(),
  loadSsotHealth()
]);

// Aplicar lo que llegó exitosamente
snapshot.forEach((result, index) => {
  if (result.status === 'fulfilled') {
    // actualizar la sección correspondiente
  } else {
    // mantener el estado anterior de esa sección
    console.warn('Sección falló:', index, result.reason);
  }
});
```

**Regla:** Una consulta caída no debe romper el War Room completo.

---

### NUEVO 2 — Filtros de logs: locales (no refetch)

```javascript
// ❌ MAL — refetch cada vez que el usuario filtra
const handleFilterChange = async (nivel) => {
  const { data } = await supabase
    .from('logs_operativos')
    .select('*')
    .eq('nivel', nivel)
    .limit(50);
  setLogs(data);
};

// ✅ CORRECTO — traer 50 registros, filtrar localmente
const [allLogs, setAllLogs] = useState([]);
const [logFilter, setLogFilter] = useState({ nivel: 'all', origen: 'all' });

const filteredLogs = useMemo(() => {
  return allLogs.filter(log => {
    if (logFilter.nivel !== 'all' && log.nivel !== logFilter.nivel) return false;
    if (logFilter.origen !== 'all' && log.origen !== logFilter.origen) return false;
    return true;
  });
}, [allLogs, logFilter]);
```

---

### NUEVO 3 — Cables ⚪ NO afectan el estado global

```javascript
// Regla: ⚪ pendiente de configurar ≠ ❌ fallo operacional

function calculateEcosystemHealth({ cables, criticalLogs, failedRpc }) {
  // ⚪ cables NUNCA cuentan como red
  const redCables = Object.values(cables).filter(c =>
    c.status === 'red' && c.is_configured === true  // solo los configurados
  );

  if (failedRpc) return 'critical';
  if (criticalLogs.filter(l => !l.resuelto).length > 0) return 'critical';
  if (redCables.length > 0) return 'degraded';
  return 'healthy';
}

// Ejemplo correcto:
// Geniall ⚪ → NO cuenta para el estado global
// TBO ⚪ → NO cuenta para el estado global
// Meta CAPI 🔴 → SÍ cuenta (is_configured: true)
```

---

### SOBRE EL TRIGGER payment_ledger (CORRECCIÓN YA APLICADA)

Computer señala que `AFTER INSERT` no puede modificar `NEW.payment_id`.

**Esto ya está correcto en producción:**
```
trg_prepare_ledger_insert  → BEFORE INSERT  ✅
  → hereda payment_id de la aplicación original
  → valida reversal_of_ledger_id
  → prohibe reversión de reversión

trg_sync_ledger_to_crm     → AFTER INSERT   ✅
  → solo llama atlas_sync_lead_finance_state()
  → NO modifica ningún campo
```

SPEC-003 v1.0.6 ya implementó esta separación correcta. El cable `payment_ledger` en el War Room puede conectarse con confianza.

---

### CLASIFICACIÓN FINAL — P0 / P1

**P0 — Antes de merge (obligatorio):**
- [x] No usar `bookings.created_at` como health de Geniall
- [x] Estado global con lógica explícita (`calculateEcosystemHealth`)
- [x] ⚪ cables no generan alerta global
- [x] `Promise.allSettled()` para refresh resiliente
- [x] Snapshot atómico — sin blank state durante refresh
- [x] Filtros de logs: locales (no refetch)

**P1 — Dentro de v5.0:**
- Owner Cards expandibles inline
- Drawer historial reporters
- Badge "🔴 CUELLO ACTIVO"

---

*ATLAS-TECH · WAR-ROOM-V50-SPEC v1.4 FINAL · 4 revisiones Computer · 25 Jul 2026*
*STATUS: APPROVED WITH REQUIRED CORRECTIONS — EXECUTOR: ANTIGRAVITY*


---

## ADDENDUM 5 — QUINTA REVISIÓN COMPUTER (PUNTOS FINALES)
**5 revisiones convergentes — SPEC EJECUTABLE**

---

### FINAL 1 — Cadencia por agente (hermes reporter)

```javascript
// Cada agente tiene su propio umbral de salud
const AGENT_CADENCE = {
  'hermes-ops':        { expectedIntervalHours: 4,  label: 'Centro Nervioso' },
  'hermes-qa':         { expectedIntervalHours: 24, label: 'Briefing diario 8AM' },
  'hermes-commercial': { expectedIntervalHours: 12, label: 'Orquestador Comercial' },
  'hermes-marketing':  { expectedIntervalHours: 24, label: 'Marketing' },
  'ariadne-data':      { expectedIntervalHours: 48, label: 'Analytics' },
  'intel':             { expectedIntervalHours: 168,label: 'Intel semanal' }
};

// Ejemplo: hermes-qa sin reporte en 8h NO es amarillo — su ciclo es 24h
```

**Mini-card por agente:**
```
🔍 HERMES QA
Último evento: hace 8h (briefing 8AM)
Eventos 24h: 3
Errores activos: 0
Estado: 🟢 ACTIVO    ← verde porque su cadencia es 24h
```

---

### FINAL 2 — Cable impact type: solo 'operational' afecta estado global

```javascript
const CABLE_DEFINITIONS = [
  // operational — afecta salud global si está rojo
  { id: 'meta_capi',     impact: 'operational', is_configured: true  },
  { id: 'firecrawl',     impact: 'operational', is_configured: true  },
  { id: 'payment_ledger',impact: 'operational', is_configured: true  },
  { id: 'hotel_knowledge',impact:'operational', is_configured: true  },
  { id: 'n8n',           impact: 'operational', is_configured: true  },

  // planned — NO afecta salud global (pendiente de activar)
  { id: 'google_capi',   impact: 'planned',     is_configured: false },

  // external — NO afecta salud global (onboarding pendiente)
  { id: 'tbo_holidays',  impact: 'external',    is_configured: false },
  { id: 'geniall',       impact: 'external',    is_configured: false },
];

// calculateEcosystemHealth solo cuenta cables operational
const criticalFailures = cables.filter(c =>
  c.status === 'red' && c.impact === 'operational'
).length;
```

---

### FINAL 3 — Logs: resuelto=true NO afecta estado actual

```javascript
// Solo logs activos (no resueltos) cuentan para el estado global
const activeErrors = logs.filter(l =>
  l.nivel === 'CRITICAL' && !l.resuelto
);

// En la UI — mostrar badge según estado de resolución:
// [CRÍTICO] [ACTIVO]    → fondo rojo intenso
// [CRÍTICO] [RESUELTO]  → fondo gris/apagado
// [ERROR]   [ACTIVO]    → fondo amber
// [ERROR]   [RESUELTO]  → fondo gris
```

---

### FINAL 4 — last_checked en cable cards

```
🟢 Meta CAPI
Último evento exitoso: hace 5 min
Verificado: hace 12s    ← timestamp del último refresh del War Room
```

```javascript
// Añadir al estado de cables
const [cablesCheckedAt, setCablesCheckedAt] = useState(null);

// Actualizar al cargar cables
setCablesCheckedAt(new Date());

// Render
<span>Verificado: {formatRelative(cablesCheckedAt)}</span>
```

---

### FINAL 5 — Geniall: success rate basado en transacciones

```javascript
// ❌ NO: timestamp mensual
// ✅ SÍ: éxito de las últimas N reservas

// Cuando existan bookings con provider_name = 'geniall':
const { data: geniallBookings } = await supabase
  .from('bookings')
  .select('status, created_at')
  .eq('provider_name', 'geniall')
  .order('created_at', { ascending: false })
  .limit(10);

const successRate = geniallBookings ?
  geniallBookings.filter(b => b.status === 'confirmed').length / geniallBookings.length : null;

// Semáforo:
// successRate === null → ⚪ sin actividad reciente
// successRate >= 0.9   → 🟢 OK
// successRate >= 0.7   → 🟡 alertas
// successRate < 0.7    → 🔴 fallos

// HOY: ⚪ hasta tener bookings con provider_name='geniall' (ya hay campo en BD)
```

---

### RESUMEN EJECUTIVO — 5 REVISIONES (para Antigravity)

**Todo lo aprobado en una sola tabla:**

| Decisión | Versión | Estado |
|----------|---------|--------|
| health ≠ freshness en semáforos | Rev1 | ✅ |
| Geniall ⚪ sin bookings.created_at | Rev1 | ✅ |
| payment_ledger → RPC integridad | Rev1 | ✅ |
| loadAll() useRef lock | Rev1 | ✅ |
| Owner cards expandibles inline | Rev2 | ✅ |
| Swap atómico sin parpadeo | Rev2 | ✅ |
| "Eventos" no "Reportes" | Rev2 | ✅ |
| Estado global calculateEcosystemHealth() | Rev3 | ✅ |
| Orden visual Header→Alertas→... | Rev3 | ✅ |
| Badge "CUELLO ACTIVO" si blocked>0 | Rev3 | ✅ |
| Promise.allSettled() resiliente | Rev4 | ✅ |
| Filtros logs locales sin refetch | Rev4 | ✅ |
| ⚪ grises no afectan estado global | Rev4 | ✅ |
| Cadencia por agente (hermes-ops:4h etc) | Rev5 | ✅ |
| Cable impact: operational/planned/external | Rev5 | ✅ |
| resuelto=true no contamina estado | Rev5 | ✅ |
| last_checked en cable cards | Rev5 | ✅ |
| Geniall: success rate transaccional | Rev5 | ✅ |

---

*ATLAS-TECH · WAR-ROOM-V50-SPEC v1.5 FINAL EJECUTABLE · 5 revisiones Computer · 25 Jul 2026*
*STATUS: APPROVED FOR EXECUTION — EXECUTOR: ANTIGRAVITY*
*"Antigravity, ejecutar War Room v5.0" — Computer, Rev5*


---

## ✅ APROBACIÓN FINAL — COMPUTER
**Fecha:** 25 Jul 2026 | **STATUS: GO — EJECUTAR WAR ROOM v5.0**

> "Antigravity puede ejecutar la SPEC v1.5 FINAL sin nuevas revisiones de arquitectura."
> — Computer, aprobación final

### CRITERIO DE ACEPTACIÓN OBLIGATORIO (Computer — último añadido)

```
Si una fuente individual falla, el War Room debe:
  ✅ Mostrar esa sección como UNKNOWN / DATA UNAVAILABLE
  ✅ Conservar la última información válida del resto del ecosistema
  ❌ NUNCA declarar todo el sistema como rojo por un fallo parcial
```

```javascript
// Implementación con Promise.allSettled() (ya en spec):
snapshot.forEach((result, index) => {
  if (result.status === 'rejected') {
    // mostrar sección individual como "DATA UNAVAILABLE"
    // NO propagar el error al estado global
    setSectionError(index, 'DATA UNAVAILABLE');
  } else {
    setSectionData(index, result.value);
  }
});
```

### 4 INVARIANTES QA (Computer — obligatorias en QA antes de merge)

```
1. Promise.allSettled()
   → una consulta fallida no puede tumbar todo el War Room

2. Swap atómico
   → nunca datos parcialmente nuevos mezclados con datos antiguos

3. loadAll() con lock/ref
   → nunca dos refresh simultáneos

4. calculateEcosystemHealth() — jamás cuenta:
   → cables planned ⚪
   → cables external ⚪
   → cables grises ⚪
   → logs resueltos
   Solo cuenta: CRITICAL activo + cable operational rojo + blocked > 0
```

---

*ATLAS-SDD SPEC-WARROOM-005 v1.5 — ✅ APPROVED FOR EXECUTION*
*5 revisiones de Quality Gate completadas — Antigravity ejecutar*
