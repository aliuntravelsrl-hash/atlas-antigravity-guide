# Auditoría e Integridad Relacional: CRM y Reserva (booking_id)

Este documento detalla la auditoría de la relación entre el **CRM (`crm_deals`, `crm_leads`)** y las **Reservas (`bookings`)** en Supabase, analizando el cumplimiento de la regla de asociación obligatoria mediante claves foráneas duras.

---

## 🔍 Hallazgos de la Auditoría

### 1. Columnas Estructurales Existentes (DB Schema)
* **Tabla `bookings`:** Contiene la columna `lead_id` (UUID) que apunta al lead original en `crm_leads` (cumple con la estructura relacional).
* **Tabla `crm_deals`:** Contiene la columna `booking_id` (UUID) que apunta a la reserva correspondiente en `bookings` (cumple con la estructura relacional).

### 2. Diagnóstico de Datos Reales (Producción)
Al consultar los registros de negociaciones activas en caliente en Supabase, detectamos una desvinculación crítica:
* **Total de Tratos (`crm_deals`):** 3
* **Tratos con `booking_id` vinculado:** 0 (100% son `null`)
* **Tratos con `booking_id` nulo:** 3

> [!WARNING]
> **Riesgo Operativo:** Debido a que el pipeline de automatización (o n8n) no asocia el UUID de la reserva al confirmarse el trato, el frontend se ve obligado a realizar un cruce aproximado por texto (`ilike('lead_guest_name', '%Nombre%')`). Esto puede provocar que no se encuentren los abonos o se asignen a un cliente con nombre homónimo.

---

## 🛠️ Plan de Solución Relacional

Proponemos una solución en tres niveles para consolidar la consistencia del motor de ingresos:

### Nivel 1: Saneamiento de Datos (Supabase)
Ejecutar un script SQL de actualización en la base de datos para asociar los tratos actuales con sus reservas correspondientes en base a la referencia o nombre.
```sql
-- Ejemplo de asociación manual/automática para registros existentes:
UPDATE crm_deals d
SET booking_id = b.id
FROM bookings b
WHERE d.booking_id IS NULL 
  AND b.lead_guest_name ILIKE '%' || (SELECT split_part(l.full_name, ' ', 1) FROM crm_leads l WHERE l.id = d.lead_id) || '%';
```

### Nivel 2: Optimización del Frontend (`CrmDashboard.jsx`)
Modificar la función `generarReciboPDF` en [CrmDashboard.jsx](file:///c:/Users/Admin/Downloads/-atlas-admin-v2/src/components/marketing/CrmDashboard.jsx) para que priorice el enlace relacional duro, manteniendo la búsqueda por nombre solo como fallback secundario.

```javascript
  // 1. Buscar booking del cliente
  let booking = null;
  if (deal.booking_id) {
    // Relación dura relacional (100% segura y óptima)
    const { data } = await supabase
      .from('bookings')
      .select('*, hotels_master(name)')
      .eq('id', deal.booking_id)
      .maybeSingle();
    booking = data;
  } else {
    // Fallback legacy por aproximación de nombre si booking_id es null
    const { data: bookings } = await supabase
      .from('bookings')
      .select('*, hotels_master(name)')
      .ilike('lead_guest_name', `%${(deal.crm_leads?.full_name || '').split(' ')[0]}%`)
      .order('created_at', { ascending: false })
      .limit(3);
    booking = bookings?.[0];
  }
```

### Nivel 3: Validación del Workflow de n8n
* **Directriz de Automatización:** Se debe actualizar la automatización de confirmación de ventas en n8n para que al insertar/confirmar una reserva en la tabla `bookings`, capture su `id` (UUID generado) y realice un `UPDATE` en la tabla `crm_deals` asociando ese `booking_id` al trato correspondiente.
