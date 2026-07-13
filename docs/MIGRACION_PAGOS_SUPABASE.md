# Guía de Migración de Base de Datos — Pagos y Transacciones
**ID de Tareas:** FIN-001 | FIN-002 SEV0 | FIN-003

## Contexto Operativo

Se detectó una discrepancia de diseño crítico heredada en el esquema de Supabase:
La tabla de pagos legacy (`atlas_payments_flat`) permitía registros huérfanos sin ninguna relación de clave foránea con el registro de reservas (`bookings.id`). Para garantizar la integridad contable y la trazabilidad financiera del modelo Merchant of Record (MoR) de Aliun Travel, toda transacción financiera debe tener un vínculo FK estricto y no nulable hacia una reserva.

---

## 🛠️ 1. Script SQL de Normalización (Supabase)

Ejecuta las siguientes instrucciones DDL en el editor de SQL de Supabase (`oyihiyivdhfxpyiwnmqk`):

```sql
-- 1. Crear la nueva tabla atlas_payments con restricción de clave foránea estricta
CREATE TABLE IF NOT EXISTS public.atlas_payments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    booking_id UUID NOT NULL REFERENCES public.bookings(id) ON DELETE RESTRICT,
    amount DECIMAL(12, 2) NOT NULL CHECK (amount > 0),
    currency VARCHAR(3) NOT NULL DEFAULT 'USD',
    gateway VARCHAR(50) NOT NULL, -- 'azul', 'transferencia', 'ratehawk_destination', etc.
    transaction_reference VARCHAR(255) UNIQUE NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'pending', -- 'pending', 'completed', 'failed', 'refunded'
    created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc'::text, now()) NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc'::text, now()) NOT NULL
);

-- 2. Habilitar Row Level Security (RLS)
ALTER TABLE public.atlas_payments ENABLE ROW LEVEL SECURITY;

-- 3. Crear vista contable de transacciones (FIN-001)
CREATE OR REPLACE VIEW public.transactions AS
SELECT 
    id AS transaction_id,
    booking_id,
    amount,
    currency,
    gateway,
    transaction_reference,
    status,
    created_at AS transaction_date
FROM public.atlas_payments;

-- 4. Otorgar permisos de lectura en la vista para auditoría
ALTER VIEW public.transactions OWNER TO postgres;
GRANT SELECT ON public.transactions TO authenticated;
GRANT SELECT ON public.transactions TO service_role;
```

---

## 🚨 2. Seguridad Crítica de API Keys (FIN-002 SEV0)

> [!CAUTION]
> **NUNCA utilices ni hardcodees la clave `service_role` de Supabase en el código de cliente (Frontend / React).**
> La clave `service_role` evade todas las políticas RLS y permite borrados y lecturas completas de la base de datos.
>
> **Directiva de Implementación:**
> - El frontend de reservas (`atlas-booking-frontend-v2`) debe usar únicamente la clave pública de Supabase (`anon` key).
> - Toda inserción, actualización o validación de pagos en `atlas_payments` que requiera bypass de RLS o comunicación con pasarelas externas (AZUL) debe realizarse de forma segura del lado del servidor utilizando workflows de **n8n** o Edge Functions independientes.
> - Las variables de entorno de n8n deben configurarse de manera aislada utilizando las bóvedas del EasyPanel en el VPS 1.

---

## 📈 3. Plan de Transición y Depuración

1. **Migración de Datos Existentes:**
   - Antes de eliminar la tabla `atlas_payments_flat`, exportar los registros históricos.
   - Si existen pagos huérfanos sin `booking_id`, el Director o el equipo de Operaciones debe asignarles la reserva correspondiente en producción o vincularlos a un booking genérico de control antes de inyectarlos en la nueva tabla `atlas_payments`.
2. **Purga Física:**
   - Una vez validados todos los registros migrados, se debe ejecutar:
     ```sql
     DROP TABLE IF EXISTS public.atlas_payments_flat CASCADE;
     ```
