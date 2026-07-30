# MANUAL DE OPERACIÓN Y MAPEO DE VERSIONES — WAR ROOM v5.0
**Código de Componente:** `WarRoomV50.jsx`
**Ruta en Admin:** `/warroom`
**Ecosistema:** ATLAS
**Última actualización:** 30 Jul 2026 (Fase 12 - Incorporación de Readiness Gates y Dimensiones Constitucionales)

---

## 📌 1. HISTORIAL DE VERSIONES Y CAMBIOS (MAPEO DE VERSIONES)

| Versión | Fecha | Autor | Cambios Clave | Estado |
|---|---|---|---|---|
| **v4.1** | Legacy | Swarm | Logs hardcodeados, sin conexión real de base de datos ni semáforos de cables. | Deprecado ❌ |
| **v5.0.0** | 25 Jul 2026 | Antigravity | Primera implementación real de la especificación con Promise.allSettled y RPCs de Supabase. | Reemplazado 🔄 |
| **v5.0.1** | 26 Jul 2026 (AM) | Antigravity | Reparación de bucles infinitos e inyección de mecanismos de resiliencia (`withTimeout` de 4s y `safetyTimeout` de 5s). | Reemplazado 🔄 |
| **v5.0.2** | 26 Jul 2026 (PM) | Antigravity | Integración de filtros locales en memoria con `timeFilter` ('24h'/'7d'/'all'), ampliación de la ventana de incidentes de Hermes a 7 días y reescritura absoluta `/index.html` en `.htaccess` contra 403/404. | Reemplazado 🔄 |
| **v5.0.3** | 30 Jul 2026 | Antigravity | Refactorización de `ConstitutionalReadiness.jsx` para adaptarla a la nueva respuesta JSON de `mc_constitutional_readiness()`, implementando puertas (gates), barras de dimensiones y motivos de bloqueo. | Reemplazado 🔄 |
| **v5.0.4** | 30 Jul 2026 | Antigravity | Integración en el Plano Operativo del nuevo componente autogestionado `DependencyIntelligence.jsx` que consume `mc_dependency_intelligence()` con refresco en vivo cada 30 segundos. | **ACTIVO (En producción) ✅** |

---

## ⚙️ 2. ARQUITECTURA DE DATOS Y CONEXIONES

La War Room v5.0 interactúa de manera reactiva y aislada con la base de datos de Supabase para evitar caídas globales ante tiempos de respuesta prolongados.

### A. Consultas y RPCs en Paralelo (Tolerantes a Fallos)
Toda llamada de red se encapsula de forma individual en promesas individuales resueltas con `Promise.allSettled()`, limitando el tiempo máximo de espera a **4.000 ms** mediante la función envoltura `withTimeout`.

1. **backlog (Backlog Activo):** `supabase.rpc('get_warroom_task_summary')`
   * *Corrección Crítica:* Retorna un objeto unificado `{ cables, owners, agent_activity, total_pending }`. El frontend extrae el array de dueños de tareas utilizando la propiedad `.owners`.
2. **logs (Consola de Incidentes):** `supabase.from('logs_operativos').select(...)`
   * Trae las últimas 50 filas de la base de datos ordenadas descendientemente.
3. **capi (Meta CAPI):** `supabase.from('crm_capi_logs').select(...)`
4. **intel (Firecrawl):** `supabase.from('competitive_intel').select(...)`
5. **ledger (Integridad Ledger):** `supabase.rpc('get_payment_ledger_breakdown')`
6. **hotel_knowledge (Registros HK):** `supabase.from('hotel_knowledge').select(...)`
7. **SSOT individuales (Hoteles/Habitaciones/Tarifas/Temporadas/Leads/Ledger):** Consultas independientes por conteo (`count: 'exact', head: true`).
8. **mc_constitutional_readiness():** Retorna el estatus de las puertas constitucionales (gates: AGF, EVO, TPP, KBP, OVR, CRP), dimensiones de madurez (pipeline, execution, knowledge, governance, evidence) y el listado de razones de bloqueo.
9. **mc_execution_readiness():** Entrega el nivel de integridad KBP por agente del Swarm (ej. `hermes-commercial`), ordenando para mostrar el último evento de rehidratación.
10. **mc_pipeline_health():** Proporciona métricas en tiempo real de la cola de tareas del pipeline (pending, ready, executing, completed_today, failed, sla_breached).
11. **mc_swarm_health():** Proporciona la telemetría en vivo del Swarm (ejecutando, pendientes, completadas hoy y estatus de última actividad para los 6 agentes).
12. **mc_dependency_intelligence():** Retorna las estadísticas del backlog de dependencias, cuellos de botella activos (prioridad crítica/alta) y tareas recién resueltas y desbloqueadas en vivo. Consumido dinámicamente y con refresco automático de 30s por `DependencyIntelligence.jsx`.

---

## 🔴 3. PROTOCOLO DE SOPORTE CONTRA WHITE SCREENS Y 404

### A. Diagnóstico y Fix de 404 / 403 (Apache & Hostinger)
Si al recargar o entrar directamente a `/warroom` el servidor de Apache retorna un error 404/403:
1. **MIME Type Mismatch:** Asegurar que el archivo `.htaccess` esté presente en la raíz de `/public_html` del host remoto.
2. **Directiva de Reescritura:** Verificar que `.htaccess` use la ruta absoluta para el index:
   ```apache
   RewriteRule ^ /index.html [L]
   ```
   *(La ruta relativa `index.html` sin barra diagonal causa fallos en Hostinger al intentar buscar el archivo relativo al subdirectorio `/warroom/`)*.

### B. Diagnóstico y Fix de Pantallas en Blanco (Runtime Crash)
El componente cuenta con un **`MissionErrorBoundary`** inyectado en `AppShell.jsx` que intercepta cualquier excepción del árbol de React. 
* Si se reporta `TypeError: e.some is not a function` o similar: indica que la RPC `get_warroom_task_summary` devolvió el objeto consolidado en lugar del array, y se debe verificar que la asignación al estado `owners` se haga a través de `.owners`.

---

## 🛠️ 4. INSTRUCCIONES DE SOPORTE OPERACIONAL (TOGGLE Y FILTROS)

### A. Toggle de Filtro Temporal
El panel de **Live Operational Log** dispone de un selector local interactivo:
* `Tiempo: TODOS`: Muestra las 50 filas de logs sin restricciones de fecha.
* `Últimas 24h`: Filtra en memoria (con `useMemo`) solo las filas con `created_at` dentro de las últimas 24 horas.
* `Últimos 7 días`: Filtra en memoria las filas dentro del rango de una semana.

### B. Intervalo de Sincronización y Seguro de Vida
* **Sincronización:** Se ejecuta de forma automática cada **60 segundos** (`60000ms`).
* **Seguro de Vida (safetyTimeout):** Al montarse el componente, se dispara un temporizador físico de **5 segundos** (`5000ms`) que apaga forzosamente el loader principal (`loading: false`) impidiendo que retardos infinitos en las consultas de red congelen la interfaz de usuario.
