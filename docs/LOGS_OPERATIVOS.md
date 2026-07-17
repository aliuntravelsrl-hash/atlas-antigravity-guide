# 📡 LOGS_OPERATIVOS — Protocolo de Escritura del Swarm
**Creado:** 17 JUL 2026 | **Autor:** Computer (Perplexity) — cobertura por timeout Antigravity
**Fuente:** Supabase `oyihiyivdhfxpyiwnmqk` — datos reales de producción

Este documento es la referencia completa de la tabla `logs_operativos` — el feed de **Actividad Reciente** en Mission Control (`/mission`) y el canal de auditoría central del ecosistema ATLAS. Cubre el schema real, los valores válidos por campo, cómo escribe el swarm, y los patrones de escalado.

---

## 1. Propósito Operativo

`logs_operativos` es la **bitácora viva del sistema**. Cada agente, workflow de n8n, y acción del panel admin deja huella aquí. El Director lee este feed en Mission Control para saber qué está pasando sin que ningún agente tenga que reportarle directamente.

```
Agente / n8n / AdminPanel
        ↓  (INSERT)
  logs_operativos
        ↓  (SELECT — poll cada 30s)
  Mission Control /mission → "Actividad Reciente" (dual-feed)
        ↓  (si escalado = true)
  Director recibe alerta
```

**Regla doctrinal:** Si un agente hizo algo relevante y no lo escribió en `logs_operativos`, operativamente no ocurrió.

---

## 2. Schema Completo (producción — 17 JUL 2026)

| Columna | Tipo | Default | Nullable | Propósito |
|---------|------|---------|----------|-----------|
| `id` | uuid | `gen_random_uuid()` | NO | PK |
| `timestamp` | timestamptz | `now()` | NO | Momento del evento (se pone automáticamente) |
| `nivel` | text | `'info'` | NO | Severidad. Ver tabla de niveles. |
| `origen` | text | — | NO | Quién genera el log. Ver convención de nombres. |
| `evento` | text | — | NO | Código de evento en MAYUSCULAS_SNAKE. Ver catálogo. |
| `mensaje` | text | — | SÍ | Descripción legible en español. Narrativa humana del evento. |
| `booking_id` | uuid | — | SÍ | FK a `bookings.id` si el evento está relacionado a una reserva |
| `workflow_id` | text | — | SÍ | ID del workflow n8n que disparó el evento |
| `payload` | jsonb | `'{}'` | SÍ | Datos estructurados adicionales. Ver ejemplos. |
| `resuelto` | boolean | `false` | SÍ | El evento fue atendido/cerrado |
| `resuelto_at` | timestamptz | — | SÍ | Cuándo se marcó como resuelto |
| `escalado` | boolean | `false` | SÍ | Se escaló al Director |
| `escalado_at` | timestamptz | — | SÍ | Cuándo se escaló |
| `created_at` | timestamptz | `now()` | SÍ | Timestamp de inserción |
| `empleado_id` | uuid | — | SÍ | FK a la tabla de empleados/agentes si aplica |

---

## 3. Niveles de Severidad (`nivel`)

| Valor | Uso | Acción esperada del Director |
|-------|-----|------------------------------|
| `INFO` | Operación normal completada exitosamente | Ninguna — solo registro |
| `WARNING` | Anomalía detectada, no bloqueante | Revisar al próximo ciclo |
| `CRITICAL` | Fallo que bloquea operaciones / SEV-0 | Atención inmediata + `escalado = true` |

**Regla:** `CRITICAL` siempre va con `escalado = true`. `WARNING` va con `escalado = true` solo si supera un umbral definido (ej: agente sin heartbeat > 2x su intervalo).

---

## 4. Convención de Nombres para `origen`

El campo `origen` identifica quién escribe el log. **Usar siempre el mismo string por agente** — en producción hay inconsistencias históricas (`HERMES-OPS`, `Hermes Ops`, `hermes-ops`) porque no existía esta convención. A partir de ahora:

| Agente / Sistema | `origen` canónico |
|------------------|-------------------|
| Hermes Ops | `HERMES-OPS` |
| Hermes Commercial | `HERMES-COMMERCIAL` |
| Hermes Marketing | `HERMES-MARKETING` |
| Ariadne Data | `ARIADNE-DATA` |
| Hermes QA | `HERMES-QA` |
| ATLAS-TECH (orquestador) | `ATLAS-TECH` |
| Panel admin (AdminBookingsPanel) | `admin_bookings_panel` |
| Panel admin (Facturador) | `FACTURADOR-ADMIN` |
| Monitor de heartbeats (n8n) | `HEARTBEAT-MONITOR` |
| Monitor de rehidratación (n8n) | `REHIDRATACION-MONITOR` |
| Ariadne (tasa dólar) | `TASA-DOLAR` |
| Motor de reservas (n8n/web) | `BOOKINGS` |
| Director | `DIRECTOR` |

**Nota:** El campo es text libre — no hay constraint en BD. La consistencia depende del protocolo, no del schema.

---

## 5. Catálogo de Eventos Reales (`evento`)

Eventos observados en producción. Usar estos strings exactos cuando aplique — no inventar variantes:

### Swarm — Hermes Ops
| `evento` | `nivel` | Descripción |
|----------|---------|-------------|
| `HIGIENE_EJECUTADA` | INFO | Limpieza periódica de heartbeats vencidos y logs antiguos |
| `OPS_001_002_CIERRE` | INFO | Cierre de tareas operacionales OPS-001 y OPS-002 |
| `OPS_003_DIAGNOSTICO` | INFO | Diagnóstico de acceso Docker socket |
| `SKILL_MODIFICADO_SIN_AUTORIZACION` | WARNING | Agente modificó un SKILL.md sin aprobación F4-RRHH-IA |
| `VERIFICACION_POST_LIMPIEZA` | INFO | Auditoría del swarm tras incidente |

### Swarm — ATLAS-TECH (orquestador)
| `evento` | `nivel` | Descripción |
|----------|---------|-------------|
| `GAP_DOCKER_SOCKET` | WARNING | hermes-agent no puede acceder al Docker daemon |
| `OPENROUTER_KEY_INVALID` | CRITICAL | API key de OpenRouter devuelve 401 — swarm inoperativo |
| `N8N_TUNNEL_DOWN` | CRITICAL | Túnel Cloudflare → n8n caído, webhooks afectados |
| `SEV0_SERVICE_ROLE_KEY_EXPUESTA` | CRITICAL | Service role key hardcoded en bundle JS público |

### Monitor de heartbeats (n8n)
| `evento` | `nivel` | Descripción |
|----------|---------|-------------|
| `HEARTBEAT_VENCIDO` | WARNING | Agente sin heartbeat más allá de 2x su intervalo normal |

### Monitor de rehidratación (n8n)
| `evento` | `nivel` | Descripción |
|----------|---------|-------------|
| `AGENTE_ACTIVO_SIN_REHIDRATAR` | WARNING | Agente con heartbeat reciente pero sin confirmar lectura de doctrina en >24h |

### Motor de reservas / Panel admin
| `evento` | `nivel` | Descripción |
|----------|---------|-------------|
| `BOOKING_CANCELLED` | INFO | Reserva cancelada |
| `PAGO_PAID` | INFO | Pago confirmado en una reserva |
| `RESERVA_CLIENTE_MODIFICADA` | INFO | Admin modificó datos del cliente de una reserva |
| `FACTURA_INDIVIDUAL_GENERADA` | INFO | Factura/voucher generado y enviado |

### Ariadne Data
| `evento` | `nivel` | Descripción |
|----------|---------|-------------|
| `TASA_ACTUALIZADA` | INFO | Tasa de cambio DOP/USD actualizada desde Banco Popular |
| `ACCION_FUERA_DE_SCOPE` | WARNING | Agente intentó acción fuera de su scope autorizado |

---

## 6. RLS — Políticas de Acceso

| Política | Operación | Quién | Noción |
|----------|-----------|-------|--------|
| `service_role_logs_operativos` | ALL | service_role | Acceso total (backend, n8n) |
| `anon_insert_logs_operativos` | INSERT | anon | El panel admin y agentes con anon key pueden escribir |

**Importante:** No hay política SELECT para `anon` — la lectura del feed en Mission Control usa `supabaseAdmin` (service_role o anon key con fallback). Si ves 0 logs en la UI, verificar que el cliente usado para SELECT tiene los permisos correctos.

---

## 7. Cómo Debe Escribir el Swarm — Protocolo

### Regla general
Cada agente debe insertar un log al **inicio** y al **cierre** de toda tarea de `atlas_tasks` que ejecute. Nunca dejar una tarea `completado` en `atlas_tasks` sin su correspondiente log de cierre en `logs_operativos`.

### Plantilla SQL para el swarm
```sql
INSERT INTO logs_operativos (nivel, origen, evento, mensaje, payload, booking_id, workflow_id, escalado)
VALUES (
  'INFO',                          -- nivel: 'INFO' | 'WARNING' | 'CRITICAL'
  'HERMES-OPS',                    -- origen: usar el canónico de la tabla sección 4
  'NOMBRE_EVENTO_EN_MAYUSCULAS',   -- evento: SNAKE_CASE, descriptivo
  'Descripción legible en español de lo que ocurrió, con datos concretos.',
  '{"tarea_codigo": "ATL-005", "detalle": "valor"}',  -- payload JSONB
  null,                            -- booking_id si aplica
  null,                            -- workflow_id si aplica
  false                            -- escalado: true solo si CRITICAL o umbral WARNING
);
```

### Desde JavaScript (supabase client — anon key)
```javascript
await supabase.from('logs_operativos').insert({
  nivel: 'INFO',
  origen: 'HERMES-OPS',
  evento: 'TAREA_COMPLETADA',
  mensaje: `ATL-007 completada: RLS auditado en 7 tablas — todas saneadas.`,
  payload: { tarea_codigo: 'ATL-007', tablas_auditadas: 7 },
  escalado: false
});
```

### Desde n8n (HTTP Request node → Supabase REST API)
```
POST https://oyihiyivdhfxpyiwnmqk.supabase.co/rest/v1/logs_operativos
Headers:
  apikey: <ANON_KEY>
  Authorization: Bearer <ANON_KEY>
  Content-Type: application/json
  Prefer: return=minimal

Body:
{
  "nivel": "WARNING",
  "origen": "HEARTBEAT-MONITOR",
  "evento": "HEARTBEAT_VENCIDO",
  "mensaje": "Hermes Marketing sin heartbeat hace 485 minutos (limite: 480 min = 2x su intervalo de 240 min)",
  "escalado": false
}
```

---

## 8. Cuándo Usar `escalado = true`

| Condición | `escalado` | `nivel` |
|-----------|-----------|---------|
| Operación normal completada | `false` | INFO |
| Anomalía detectada, agente puede auto-resolver | `false` | WARNING |
| Anomalía que supera umbral (ej: heartbeat > 2x intervalo) | `true` | WARNING |
| Fallo que bloquea operaciones del swarm | `true` | CRITICAL |
| SEV-0 de seguridad (key expuesta, RLS roto) | `true` | CRITICAL |
| Agente actuó fuera de scope sin autorización | `true` | WARNING |

Cuando `escalado = true`, el Director verá el log resaltado en Mission Control. Marcar `resuelto = true` y `resuelto_at = now()` cuando el Director o el agente responsable cierre el incidente.

---

## 9. Payload JSONB — Qué Incluir

El `payload` debe contener datos estructurados que complementen el `mensaje` legible. No repetir en payload lo que ya está en mensaje — agregar datos que permitan filtrar o procesar programáticamente.

### Ejemplos por tipo de agente

**Hermes Ops — cierre de tarea:**
```json
{
  "tarea_codigo": "ATL-007",
  "tarea_titulo": "Auditar RLS en tablas expuestas",
  "tablas_auditadas": ["booking_qa_tasks", "logs_operativos", "crm_leads"],
  "resultado": "7 tablas saneadas, 0 críticas pendientes"
}
```

**Heartbeat Monitor — agente sin pulso:**
```json
{
  "agente": "hermes-marketing",
  "minutos_sin_heartbeat": 485,
  "limite_minutos": 480,
  "intervalo_normal_min": 240
}
```

**BOOKINGS — cambio de estado:**
```json
{
  "booking_ref": "ALN-CG-005",
  "estado_anterior": "pending",
  "estado_nuevo": "paid",
  "monto_usd": 1250.00
}
```

**ATLAS-TECH — incidente CRITICAL:**
```json
{
  "severidad": "SEV-1",
  "servicio_afectado": "openrouter",
  "error_code": 401,
  "modelos_afectados": ["glm-5.1", "nemotron-ultra-550b:free"],
  "ips_afectadas": ["45.235.231.128", "45.235.231.130"],
  "sla_horas": 2
}
```

---

## 10. Higiene de la Tabla — Política de Retención

Hermes Ops ejecuta limpieza periódica (`HIGIENE_EJECUTADA`) según esta política:
- Logs `INFO` con `resuelto = true` de más de **30 días** → se eliminan.
- Heartbeats vencidos de más de **7 días** → se eliminan.
- Logs `WARNING` y `CRITICAL` **no se eliminan automáticamente** — requieren revisión manual del Director antes de archivar.

El cron de higiene corre en el workflow de n8n de Hermes Ops. Si ves el log `HIGIENE_EJECUTADA` con `"Total: 0 registros"` — la tabla está limpia en ese momento.

---

## 11. Dual-Feed en Mission Control — Cómo se Clasifica

Mission Control (`MissionControlLive.jsx` v2.8) muestra dos columnas de Actividad Reciente leyendo la misma tabla con filtros distintos:

| Feed | Filtro | Qué muestra |
|------|--------|-------------|
| **Operarios** | `origen IN ('admin_bookings_panel', 'FACTURADOR-ADMIN', 'DIRECTOR')` | Acciones humanas directas sobre reservas, clientes, facturas |
| **Sistema** | Todos los demás `origen` | Eventos automáticos del swarm, monitores, n8n workflows |

**Poll rate:** FAST = 30 segundos. Los logs aparecen en Mission Control dentro de los próximos 30s tras ser insertados.

---

*Mantenido por: Director Aldo Hilario | Aliun Travel SRL | República Dominicana*
*Creado por: Computer (Perplexity) — 17 JUL 2026 — Cobertura de Antigravity (time-out Google)*
