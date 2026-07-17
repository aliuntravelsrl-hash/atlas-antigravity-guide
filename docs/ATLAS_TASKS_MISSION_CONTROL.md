# 📋 ATLAS TASKS — Mesa de Tareas de Mission Control
**Creado:** 17 JUL 2026 | **Autor:** Computer (Perplexity) — cobertura por timeout Antigravity
**Fuente:** Supabase `oyihiyivdhfxpyiwnmqk` + código `MissionControlLive.jsx` (v2.8)

Este documento es la referencia completa de `atlas_tasks` — la tabla que alimenta la **Mesa de Tareas** en Mission Control (`/mission`). Cubre el schema real, las reglas de negocio, cómo opera el swarm con ella, y cómo interactuar con ella correctamente.

---

## 1. Propósito Operativo

`atlas_tasks` es la **hoja de ruta del swarm**. No es un backlog de Jira ni una lista de Notion. Es el único canal oficial por el que el Director encarga trabajo a los agentes y por el que ATLAS-TECH orquesta al swarm. Toda tarea que no exista aquí no existe operativamente.

```
Director / ATLAS-TECH
        ↓  (crea tarea con rpc_crear_tarea)
   atlas_tasks  ←→  Mission Control /mission  (UI en tiempo real)
        ↓  (agente lee, ejecuta, actualiza estado)
   logs_operativos  (feed de Actividad Reciente en Mission Control)
```

**Notion es borrador** — los apuntes de Notion no son fuente operativa. Supabase + Mission Control son la verdad.

---

## 2. Schema Completo (producción — 17 JUL 2026)

| Columna | Tipo | Default | Nullable | Propósito |
|---------|------|---------|----------|-----------|
| `id` | uuid | `gen_random_uuid()` | NO | PK |
| `codigo` | text | — | SÍ | Identificador legible. `ATL-001`, `OPS-002`. Auto-generado por RPC. |
| `titulo` | text | — | NO | Nombre corto de la tarea |
| `descripcion` | text | — | SÍ | Detalle del trabajo a realizar |
| `departamento` | text | `'TECH'` | NO | Área institucional |
| `asignado_a` | text | — | NO | Nombre del agente o rol receptor |
| `encargado_por` | text | `'director'` | SÍ | Quién encargó la tarea |
| `prioridad` | text | `'media'` | NO | `alta` / `media` / `baja` |
| `estado` | text | `'pendiente'` | NO | Ver tabla de estados |
| `bloqueante` | boolean | `false` | SÍ | ¿Bloquea a otras tareas? |
| `fecha_encargo` | timestamptz | `now()` | SÍ | Cuándo fue encargada |
| `fecha_inicio` | timestamptz | — | SÍ | Cuándo empezó la ejecución |
| `fecha_limite` | timestamptz | — | SÍ | Deadline |
| `fecha_completado` | timestamptz | — | SÍ | Cuándo se cerró |
| `archivo_afectado` | text | — | SÍ | Ruta del archivo que toca la tarea |
| `workflow_id` | text | — | SÍ | ID del workflow n8n asociado |
| `notas` | text | — | SÍ | Notas de ejecución del agente |
| `resultado` | text | — | SÍ | Resultado final documentado |
| `tiempo_estimado_min` | integer | — | SÍ | Estimado en minutos |
| `sprint` | text | `'backlog'` | SÍ | Sprint o fase (`backlog`, `sprint-1`, etc.) |
| `created_at` | timestamptz | `now()` | SÍ | Timestamp de creación |
| `updated_at` | timestamptz | `now()` | SÍ | Timestamp de última modificación |
| `frente` | text | — | SÍ | Frente de color. Ver tabla de valores. |
| `tipo` | text | `'proyecto'` | NO | `proyecto` / `operacional` |
| `asignado_tipo` | text | — | SÍ | Tipo de destinatario. Ver tabla de valores. |
| `bloqueado` | boolean | `false` | NO | ¿La tarea está bloqueada externamente? |
| `bloqueo_razon` | text | — | SÍ | Por qué está bloqueada (ej: "Esperando RNC") |
| `origen_incidente_id` | uuid | — | SÍ | FK a otra tarea que originó este incidente |
| `autorizado_por` | text | — | SÍ | Quién autorizó la ejecución (para el swarm) |

---

## 3. Valores y Enumeraciones

### Estados (`estado`)
| Valor | Significado | ¿Aparece en UI? |
|-------|-------------|-----------------|
| `pendiente` | Sin iniciar — en cola | ✅ Sí |
| `en_progreso` | En ejecución activa | ✅ Sí |
| `bloqueado` | Detenida por dependencia externa | ✅ Sí |
| `completado` | Ejecutada y cerrada con éxito | Historial |
| `resuelto` | Incidente resuelto | Historial |
| `archivado` | Purgada del flujo activo | No visible |

### Tipos (`tipo`)
| Valor | Código generado | Uso |
|-------|----------------|-----|
| `operacional` | `OPS-001`, `OPS-002`… | Incidente urgente, firefight, acción inmediata |
| `proyecto` | `ATL-001`, `ATL-002`… | Deuda técnica, roadmap, mejora planificada |

**Regla:** Si es urgente y bloquea operaciones → `operacional`. Si es mejora o deuda → `proyecto`.

### Tipos de destinatario (`asignado_tipo`)
| Valor | A quién va | Ejemplos de `asignado_a` |
|-------|-----------|--------------------------|
| `swarm` | Agentes Hermes internos | `hermes-ops`, `hermes-commercial`, `hermes-marketing`, `ariadne-data`, `hermes-qa` |
| `antigravity` | Antigravity (agente externo) | `antigravity`, `director` |
| `computer` | Computer/Perplexity | `computer` |
| `null` | Sin clasificar (legacy) | `claude`, `atlas-tech`, `director` |

### Frentes (`frente`) — con colores
| Valor | Color | Área |
|-------|-------|------|
| `F1-FRONTEND` | `#3B82F6` (azul) | UI, componentes React, web pública |
| `F2-BACKEND-CORE` | `#8B5CF6` (violeta) | Supabase, RPCs, APIs, n8n |
| `F3-ATRACCION` | `#F59E0B` (ámbar) | Marketing, ofertas, motor comercial |
| `F4-RRHH-IA` | `#10B981` (verde) | Agentes, doctrina, SOUL.md, RRHH-IA |
| `F5-SEGURIDAD` | `#EF4444` (rojo) | Seguridad, RLS, keys, auditoría |

---

## 4. RLS — Políticas de Acceso

| Política | Operación | Quién | Condición |
|----------|-----------|-------|-----------|
| `service_role_atlas_tasks` | ALL | service_role | `true` (bypass total) |
| `atlas_tasks_select_anon` | SELECT | anon | `true` (lectura pública) |
| `atlas_tasks_select_authenticated` | SELECT | authenticated | `true` |
| `atlas_tasks_insert_anon` | INSERT | anon | sin restricción |
| `atlas_tasks_update_anon` | UPDATE | anon | `true` |

**Nota:** La política de SELECT para `anon` fue agregada el 06 JUL 2026 — antes de eso la UI mostraba 0 tareas aunque existían 30+ en la tabla.

---

## 5. RPC: `rpc_crear_tarea` — La forma correcta de crear tareas

**Nunca hacer INSERT directo**. Siempre usar el RPC — genera el código `ATL-xxx` / `OPS-xxx` automáticamente con secuencia correcta.

### Firma completa
```sql
SELECT rpc_crear_tarea(
  p_titulo          TEXT,                      -- REQUERIDO
  p_descripcion     TEXT     DEFAULT NULL,
  p_tipo            TEXT     DEFAULT 'proyecto',   -- 'proyecto' | 'operacional'
  p_asignado_tipo   TEXT     DEFAULT 'computer',   -- 'swarm' | 'antigravity' | 'computer'
  p_asignado_a      TEXT     DEFAULT NULL,         -- nombre del agente
  p_prioridad       TEXT     DEFAULT 'media',      -- 'alta' | 'media' | 'baja'
  p_frente          TEXT     DEFAULT NULL,         -- 'F1-FRONTEND' | 'F2-BACKEND-CORE' | etc.
  p_sprint          TEXT     DEFAULT NULL,         -- 'backlog' | 'sprint-1' | etc.
  p_bloqueado       BOOLEAN  DEFAULT false,
  p_bloqueo_razon   TEXT     DEFAULT NULL,
  p_origen_incidente_id UUID DEFAULT NULL          -- FK a otra tarea que originó este incidente
);
```

### Lógica interna del RPC
1. Determina el prefijo: `operacional` → `OPS`, `proyecto` → `ATL`.
2. Busca el MAX numérico de los códigos existentes con ese prefijo y suma 1.
3. Genera el código: `ATL-001`, `ATL-002`... / `OPS-001`, `OPS-002`...
4. Inserta en `atlas_tasks` con `estado = 'pendiente'` y `fecha_encargo = NOW()`.
5. Retorna el registro completo como JSON.

### Ejemplos de uso

**Crear tarea de proyecto para hermes-ops:**
```sql
SELECT rpc_crear_tarea(
  'Implementar RLS en tabla nueva xyz',
  'Agregar política SELECT para anon en la tabla xyz tras detectar que retorna 0 rows en UI',
  'proyecto',
  'swarm',
  'hermes-ops',
  'alta',
  'F2-BACKEND-CORE',
  'sprint-1'
);
```

**Crear incidente operacional para Antigravity:**
```sql
SELECT rpc_crear_tarea(
  'White screen en /admin41',
  'El dashboard muestra pantalla blanca tras el último commit. MIME mismatch en assets.',
  'operacional',
  'antigravity',
  'director',
  'alta',
  'F1-FRONTEND'
);
```

**Desde JavaScript (Supabase client):**
```javascript
const { data, error } = await supabase.rpc('rpc_crear_tarea', {
  p_titulo: 'Fix RLS en atlas_payments',
  p_tipo: 'operacional',
  p_asignado_tipo: 'swarm',
  p_asignado_a: 'hermes-ops',
  p_prioridad: 'alta',
  p_frente: 'F5-SEGURIDAD'
});
```

---

## 6. Mission Control — UI de la Mesa de Tareas

**Ruta:** `https://atlas.aliuntravelsrl.com/mission`
**Componente:** `src/components/MissionControlLive.jsx` (v2.8)
**Poll rate:** SLOW = 300s (5 min) para la Mesa de Tareas

### Funciones disponibles en la UI

| Función | Descripción |
|---------|-------------|
| **Filtros** | Por estado (`pendiente`, `en_progreso`, `bloqueado`), frente (F1-F5), tipo (`proyecto` / `operacional`) |
| **+ Nueva Tarea** | Modal para crear tarea directamente desde el dashboard — llama a `rpc_crear_tarea` |
| **Badges de frente** | Chip de color por `frente` (F1=azul, F2=violeta, F3=ámbar, F4=verde, F5=rojo) |
| **Chip `bloqueante`** | Indicador visual rojo si `bloqueante = true` |
| **Chip `bloqueado`** | Indicador visual naranja si `bloqueado = true` con tooltip de `bloqueo_razon` |

### Actividad Reciente (dual-feed)
La sección de Actividad Reciente de Mission Control muestra **dos feeds paralelos**:
1. **Operarios** — acciones humanas/agente desde `logs_operativos` (filtro: `origen = 'admin_bookings_panel'` u otros operacionales)
2. **Sistema** — eventos automáticos desde `logs_operativos` (filtro: `origen = 'sistema'` o automáticos)

**Poll rate Actividad Reciente:** FAST = 30s

---

## 7. Estado Real de la Tabla (17 JUL 2026)

### Tareas pendientes activas — 72 registros total en la tabla

| Estado | Total |
|--------|-------|
| `pendiente` | ~72 tareas activas |
| `completado` | ~38 |
| `archivado` | ~10 |
| `resuelto` | 3 |

### Pendientes por owner/tipo (post-remap 07 JUL 2026)
| `asignado_a` | `asignado_tipo` | Tareas pendientes |
|---|---|---|
| Hermes Ops | swarm | 13 |
| ATLAS-TECH | — | 16 (F2+F3 pendientes) |
| Hermes Commercial | swarm | 8 |
| Hermes Marketing | swarm | 4 |
| Director | antigravity | 5 |
| Antigravity | — | 4 |
| Ariadne Data | swarm | 2 |
| Hermes QA | swarm | 1 |

### Tareas reasignadas en sesión 07 JUL (legacy → swarm real)
Las siguientes tareas que tenían `asignado_a = 'computer'` o `asignado_a = 'claude'` fueron remapeadas al owner real del swarm:
- `ATL-001` (Auth) → `hermes-ops`
- `ATL-003` (n8n owners) → `hermes-ops`
- `OPS-002` (service_role key) → `hermes-ops`
- `OPS-003` (forense git) → `hermes-ops`
- `planning_agent` → renombrado a `legacy/planning_agent`

---

## 8. Reglas de Negocio — Qué NO hacer

1. **NUNCA hacer INSERT directo** en `atlas_tasks`. Siempre usar `rpc_crear_tarea` para garantizar la secuencia de códigos correcta.
2. **NUNCA modificar tareas `completado` o `archivado`** — conservan el owner histórico para trazabilidad.
3. **NUNCA crear tareas sin `frente`** para trabajo que va al swarm — el frente es lo que categoriza el trabajo en Mission Control.
4. **El código `ATL-xxx` / `OPS-xxx` es inmutable** una vez creado — no renombrar ni reasignar el número.
5. **`bloqueado = true` requiere `bloqueo_razon`** — documentar siempre el motivo (ej: "Esperando RNC", "Pendiente approval Director").

---

## 9. Migraciones Aplicadas — Historial

| Fecha | Migración | Cambio |
|-------|-----------|--------|
| 06 JUL 2026 | `atlas_tasks_v2_migration` | +6 columnas: `frente`, `tipo`, `asignado_tipo`, `bloqueado`, `bloqueo_razon`, `origen_incidente_id`, `autorizado_por`. RPC `rpc_crear_tarea`. |
| 06 JUL 2026 | `atlas_tasks_select_anon` | Política SELECT para `anon` y `authenticated`. Sin esto la UI mostraba 0 tareas. |
| 06 JUL 2026 | `rls_audit_fix_all_unprotected_tables` | 7 tablas saneadas incluyendo `booking_qa_tasks` (tenía RLS OFF). |
| 07 JUL 2026 | `atlas_tasks_remap_owners_swarm_real_0707` | Remap masivo: owners legacy (`computer`, `claude`, `planning_agent`) → owners reales del swarm. |

---

*Mantenido por: Director Aldo Hilario | Aliun Travel SRL | República Dominicana*
*Creado por: Computer (Perplexity) — 17 JUL 2026 — Cobertura de Antigravity (time-out Google)*
