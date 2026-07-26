# WAR ROOM v5.0 — SPEC PARA ANTIGRAVITY
**Versión:** 1.5 FINAL | **Fecha:** 25 Jul 2026
**STATUS: ✅ APPROVED FOR EXECUTION — Computer (5 QA Gates)**
**Componente:** `WarRoomV41.jsx` → `WarRoomV50.jsx` | **Ruta:** `/warroom`

> Backend 100% listo. Antigravity solo construye el frontend.

---

## 1. EL PROBLEMA ACTUAL (War Room v4.1)

```
❌ Logs hardcodeados en useState([])
❌ Engines hardcodeados — no conectados a datos reales
❌ Sin visión de los 4 owners del ecosistema
❌ Hermes Ops/QA reportan a logs_operativos pero nunca llegan al panel
```

---

## 2. DISEÑO — ORDEN VISUAL APROBADO

```
┌────────────────────────────────────────────────────────────┐
│ WAR ROOM v5.0                    🟢 ECOSISTEMA OPERATIVO   │
│ Near real-time · Última actualización: hace 12s · [● ]     │
└────────────────────────────────────────────────────────────┘

[🔴 ALERTAS CRÍTICAS — solo si existen — conditional render]

┌──────────────┬──────────────┬──────────────┬──────────────┐
│ DIRECTOR (A) │ ATLAS-TECH(B)│   SWARM  (C) │ ANTIGRAV.(D) │
│ 14 activas   │ 20 activas   │ 23 activas   │ 7 activas    │
│ 🔴 1 crítica │ 🔴 1 crítica │ 🟢 0         │ 🟢 0         │
│ ⛓ 2 bloq.   │ ⛓ 3 bloq.   │              │              │
│ [▼ Expandir] │ [▼ Expandir] │ [▼ Expandir] │ [▼ Expandir] │
└──────────────┴──────────────┴──────────────┴──────────────┘

┌──────────────────────────┬─────────────────────────────────┐
│ CABLES / INTEGRACIONES   │ HERMES REPORTER                 │
│ 🟢 Meta CAPI             │ 🤖 hermes-ops · hace 2h         │
│    Último éxito: 12 min  │    Eventos hoy: 42              │
│    Errores 24h: 0        │    Errores: 0 🟢                │
│    Verificado: hace 12s  │                                 │
│ 🟢 Firecrawl             │ 🔍 hermes-qa · hace 8h          │
│ 🟢 payment_ledger        │    Eventos hoy: 3               │
│ 🟡 hotel_knowledge       │    Errores: 0 🟢                │
│ ⚪ Geniall               │                                 │
│ ⚪ TBO Holidays          │ [Ver historial →] drawer        │
└──────────────────────────┴─────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ LIVE OPERATIONAL LOG                                       │
│ [Todos] [INFO] [WARN] [ERROR] [CRITICAL] · [Origen ▼]    │
│ 15:42 🟢 hermes-ops   Evento completado                   │
│ 15:40 ⚠️  atlas-tech  Migración pendiente  [ACTIVO]        │
│ 15:38 🔴 hermes-qa    Test failed          [RESUELTO] ✓   │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ SSOT HEALTH (mantener igual que v4.1 — no tocar)          │
│ + añadir: crm_leads total · payment_ledger entries        │
└────────────────────────────────────────────────────────────┘
```

---

## 3. FUENTES DE DATOS (backend listo ✅)

### Owner Cards
```javascript
const { data } = await supabase.rpc('get_warroom_task_summary');
// Devuelve: owners[] con total_active, pending, in_progress,
//           blocked, critical, high, medium, health, top_tasks[]
```

### Logs (directo — sin RPC)
```javascript
const { data: allLogs } = await supabase
  .from('logs_operativos')
  .select('id, nivel, origen, evento, mensaje, created_at, resuelto')
  .order('created_at', { ascending: false })
  .limit(50);
// Filtrar LOCALMENTE con useMemo — NO refetch al filtrar
```

### Cables
```javascript
// Meta CAPI
const { data: capi } = await supabase
  .from('crm_capi_logs')
  .select('sent_at, status, created_at')
  .order('sent_at', { ascending: false })
  .limit(5);

// Firecrawl Intel
const { data: intel } = await supabase
  .from('competitive_intel')
  .select('scrapeado_at')
  .order('scrapeado_at', { ascending: false })
  .limit(1);

// payment_ledger (salud por integridad, no solo fecha)
const { data: ledger } = await supabase
  .rpc('get_payment_ledger_breakdown');

// hotel_knowledge
const { count: hkCount } = await supabase
  .from('hotel_knowledge')
  .select('id', { count: 'exact', head: true })
  .eq('activo', true);
```

### Hermes Reporter
```javascript
const { data: agentActivity } = await supabase
  .from('logs_operativos')
  .select('origen, nivel, created_at')
  .gte('created_at', new Date(Date.now() - 86400000).toISOString())
  .order('created_at', { ascending: false });
// Agrupar por origen en el frontend
```

---

## 4. LÓGICA DE SEMÁFOROS

### resolveCableStatus() — función principal
```javascript
function resolveCableStatus({
  lastSuccessAt,        // timestamp del último ÉXITO (preferir sobre lastTimestamp)
  lastTimestamp,        // timestamp del último dato
  warningAfterHours,
  criticalAfterHours,
  hasError = false,
  isConfigured = true,
  isHealthy = true,
  count = null,
  min = null
}) {
  if (!isConfigured) return 'gray';        // ⚪ pendiente de configurar
  if (hasError || !isHealthy) return 'red'; // INTEGRIDAD > FRESHNESS

  if (count !== null && min !== null) {
    return count >= min ? 'green' : 'yellow';
  }

  const ts = lastSuccessAt || lastTimestamp;
  if (!ts) return 'red';

  const hours = (Date.now() - new Date(ts)) / 3600000;
  if (hours <= warningAfterHours) return 'green';
  if (hours <= criticalAfterHours) return 'yellow';
  return 'red';
}
```

### Configuración por cable
```javascript
const CABLE_CONFIG = {
  meta_capi: {
    impact: 'operational',     // ← afecta estado global
    warningAfterHours: 6,
    criticalAfterHours: 24,
    // hasError = capi.some(c => c.status === 'error' && recent)
    // lastSuccessAt = capi.find(c => c.status === 'sent')?.sent_at
  },
  firecrawl: {
    impact: 'operational',
    warningAfterHours: 168,    // 7 días (ciclo semanal)
    criticalAfterHours: 336,
  },
  payment_ledger: {
    impact: 'operational',
    // isHealthy = get_payment_ledger_breakdown() responde sin anomalías
  },
  hotel_knowledge: {
    impact: 'operational',
    count: hkCount,
    min: 150
  },
  geniall: {
    impact: 'external',
    isConfigured: false,       // ⚪ hasta supplier_api_logs real
  },
  tbo_holidays: {
    impact: 'external',
    isConfigured: false,       // ⚪ pendiente onboarding RNC
  },
  google_capi: {
    impact: 'planned',
    isConfigured: false,       // ⚪ pendiente activar
  }
};
```

### calculateEcosystemHealth() — estado global
```javascript
function calculateEcosystemHealth({ cables, criticalLogs, owners }) {
  // SOLO cables 'operational' afectan el estado global
  const redOperational = Object.entries(cables).filter(([id, cable]) =>
    cable.status === 'red' && CABLE_CONFIG[id]?.impact === 'operational'
  ).length;

  // Solo CRITICAL no resueltos
  const activeCritical = criticalLogs.filter(l =>
    l.nivel === 'CRITICAL' && !l.resuelto
  ).length;

  const hasBlockedCritical = owners.some(o =>
    o.health === 'red' && o.critical > 0
  );

  if (redOperational > 0 || activeCritical > 0) return 'red';
  if (hasBlockedCritical) return 'yellow';
  return 'green';
}
```

---

## 5. OWNER CARDS — comportamiento

```javascript
// La tarjeta muestra top_tasks del RPC (ya incluidos)
// Click → expande inline con lazy load
const [expandedOwner, setExpandedOwner] = useState(null);

// Al expandir, cargar tareas completas:
const loadOwnerTasks = async (ownerType) => {
  const { data } = await supabase
    .from('atlas_tasks')
    .select('id, codigo, titulo, prioridad, estado, depende_de')
    .eq('owner_type', ownerType)
    .in('estado', ['pendiente', 'en_progreso'])
    .order('prioridad')  // critica primero
    .limit(50);
  return data;
};
```

**UI tarjeta expandida:**
```
┌─────────────────────────────────────┐
│ DIRECTOR (A)          ▲ cerrar      │
│ 14 activas                          │
├─────────────────────────────────────┤
│ 🔴 MKT-002  CRÍTICA   VPS3 Hostinger│
│ ⚠️  LEGAL-01 ALTA     RNC pendiente  │
│ ⛓  CABLE-AT ALTA     bloqueada     │
│                                     │
│ [Ver todas →] supabase query        │
└─────────────────────────────────────┘
```

**Badge cuello activo:**
```jsx
{owner.blocked > 0 && (
  <span className="text-xs text-red-400 border border-red-800 px-2 py-0.5 rounded">
    ⛓ CUELLO ACTIVO
  </span>
)}
```

---

## 6. HERMES REPORTER — cadencia por agente

```javascript
const AGENT_CADENCE = {
  'hermes-ops':        { expectedHours: 4,   label: 'Centro Nervioso'    },
  'hermes-qa':         { expectedHours: 24,  label: 'Briefing 8AM'       },
  'hermes-commercial': { expectedHours: 12,  label: 'Orquestador Comerc.'},
  'hermes-marketing':  { expectedHours: 24,  label: 'Marketing'          },
  'ariadne-data':      { expectedHours: 48,  label: 'Analytics'          },
  'intel':             { expectedHours: 168, label: 'Intel semanal'      },
  'atlas-tech':        { expectedHours: 24,  label: 'Backend'            }
};

// hermes-qa sin reporte en 8h → sigue VERDE (cadencia 24h)
// hermes-ops sin reporte en 5h → AMARILLO (cadencia 4h)
```

**Mini-card:**
```
🤖 HERMES OPS
Último evento: hace 2h
Eventos hoy: 42        ← "Eventos" NO "Reportes"
Errores: 0 🟢
[Ver historial →]      ← drawer lateral, NO nueva ruta
```

---

## 7. AUTO-REFRESH — sin parpadeo

```javascript
const refreshInFlight = useRef(false);
const [isRefreshing, setIsRefreshing] = useState(false);

const loadAll = async () => {
  if (refreshInFlight.current) return;  // lock
  refreshInFlight.current = true;
  setIsRefreshing(true);

  try {
    // Fetch en paralelo — si uno falla, los demás siguen
    const [taskData, logsData, cablesData, agentsData, ssotData] =
      await Promise.allSettled([
        loadTaskSummary(),
        loadLogs(),
        loadCables(),
        loadAgentReports(),
        loadSsotHealth()
      ]);

    // Swap atómico — todos juntos, sin nullear estado previo
    if (taskData.status === 'fulfilled') setTaskSummary(taskData.value);
    else setSectionError('tasks', 'DATA UNAVAILABLE');

    if (logsData.status === 'fulfilled') setAllLogs(logsData.value);
    if (cablesData.status === 'fulfilled') setCables(cablesData.value);
    if (agentsData.status === 'fulfilled') setAgentReports(agentsData.value);
    if (ssotData.status === 'fulfilled') setSsotHealth(ssotData.value);

    setLastRefresh(new Date());
    setCablesCheckedAt(new Date());  // last_checked en cada cable

  } finally {
    refreshInFlight.current = false;
    setIsRefreshing(false);
  }
};

// Solo en primera carga muestra loading completo
const isInitialLoad = !lastRefresh;

useEffect(() => {
  loadAll();
  const interval = setInterval(loadAll, 60000);
  return () => clearInterval(interval);
}, []);
```

---

## 8. LOGS — render y filtros locales

```javascript
// Filtros LOCALES — sin refetch
const filteredLogs = useMemo(() => {
  return allLogs.filter(log => {
    if (logFilter.nivel !== 'all' && log.nivel !== logFilter.nivel) return false;
    if (logFilter.origen !== 'all' && log.origen !== logFilter.origen) return false;
    return true;
  });
}, [allLogs, logFilter]);

// Badge resuelto
const logColor = {
  CRITICAL: log.resuelto
    ? 'border-slate-700 opacity-50'          // ← resuelto: apagado
    : 'bg-red-950/30 border-red-800',        // ← activo: rojo
  ERROR: log.resuelto
    ? 'border-slate-700 opacity-50'
    : 'bg-amber-950/30 border-amber-800',
  WARN:  'border-amber-500/30',
  INFO:  'border-slate-700',
  SUCCESS: 'border-emerald-500/30'
};
```

---

## 9. DISEÑO — paleta y tokens

```
Paleta base (no cambiar):  slate-950 / slate-900 / slate-800
Owner badges:
  director     = blue-500
  atlas-tech   = violet-500
  swarm        = emerald-500
  antigravity  = amber-500
Semáforo:
  🟢 text-emerald-400
  🟡 text-amber-400
  🔴 text-red-400
  ⚪ text-slate-500
Refresh dot:   animate-pulse text-emerald-400
Header status:
  🟢 ECOSISTEMA OPERATIVO
  🟡 ADVERTENCIAS ACTIVAS
  🔴 REQUIERE ATENCIÓN
```

---

## 10. CRITERIOS DE ACEPTACIÓN QA (obligatorios antes de merge)

```
□ Promise.allSettled() — un fallo no tumba todo el War Room
□ Fuente fallida → "DATA UNAVAILABLE" sin afectar otras secciones
□ Swap atómico — sin nullear estado previo durante refresh
□ loadAll() usa useRef lock — sin refresh simultáneos
□ calculateEcosystemHealth() solo cuenta cables "operational"
□ cables planned/external → ⚪ y NO afectan estado global
□ logs resuelto=true → apagados visualmente, no cuentan en estado
□ Owner cards expandibles inline con lazy load al click
□ Badge "⛓ CUELLO ACTIVO" si blocked > 0
□ Hermes Reporter usa "Eventos hoy" (no "Reportes")
□ cadencia por agente: hermes-qa en 8h sigue verde (cadencia 24h)
□ Filtros de logs son LOCALES (no refetch)
□ last_checked en cada cable card
□ [Ver historial →] → drawer lateral, NO nueva ruta
□ Header: "Near real-time" (no "tiempo real")
□ SSOT Health de v4.1 mantiene funcionando (no regresión)
□ Auto-refresh cada 60s — dot pulsante, sin parpadeo de contenido
```

---

## 11. SSOT HEALTH — qué añadir

Al panel existente de v4.1, añadir solo:
```javascript
// En la sección SSOT Health:
{ label: 'Leads CRM',      value: crm_leads_count }
{ label: 'Asientos Ledger', value: payment_ledger_count }
```

---

*ATLAS-TECH + Computer (5 QA Gates) · War Room v5.0 SPEC FINAL · 25 Jul 2026*
*STATUS: ✅ APPROVED FOR EXECUTION*
