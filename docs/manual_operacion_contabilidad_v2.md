# Manual de Operación y Mapeo de Versiones: Panel Contable e IA Engine
**Versión:** 1.0.0-FIN (2026-07-24)  
**Mapeo de Frente:** F1-FRONTEND (panel administrativo `/admin/bookings`) y F2-WORKFLOWS (n8n proxy `WF-ACCOUNTING-AI-v1`)

---

## 📌 1. Lógica del Módulo Contable (FIN-003)

El panel de contabilidad automatizado implementa un entorno de control financiero en tiempo real y procesamiento con IA de gestiones del Director:

### A. Sección de Métricas (Dashboard Meta $200k)
*   **RPC Supabase:** `get_accounting_dashboard()`
*   **Funcionamiento:** Realiza la consulta directa a las tablas canónicas de Supabase (`bookings` y `atlas_payments`), sumando en tiempo real lo facturado y cobrado, normalizando a USD mediante tasa estándar de 59.0 para medir el progreso exacto de caja hacia el objetivo de los $200,000 USD.
*   **Campos calculados:**
    1.  `total_facturado`: Suma de total de reservas en estado `'confirmed'`.
    2.  `cash_available`: Suma de pagos aprobados en `'approved'` de la tabla `atlas_payments`.
    3.  `pending_payments`: Diferencia de facturado y cobrado de reservas confirmadas.
    4.  `projected_income`: Suma de total de reservas en estado `'pending'`.

### B. Consola IA (IA Engine)
*   **Endpoint n8n:** `POST /webhook/accounting-process`
*   **Funcionamiento:** Envía el dictado de texto libre del Director al proxy de n8n con Gemini. Al retornar la extracción de entidades (cliente, hotel, monto, moneda y confianza), el panel de control muestra la tarjeta de confirmación contable.
*   **Confirmación:** Al validar el operador, se realiza el INSERT en `offline_operations` en Supabase y se avanza el stage del lead a `'deposito_recibido'` para iniciar el pipeline.

### C. Emisión de Documentos Grupales
*   **Factura Grupal:** Llama a `/webhook/aliun-factura` con `tipo: 'grupal'`.
*   **Vouchers Masivos (19):** Llama a `/webhook/vouchers-masivos` para gatillar el loop masivo en n8n para los 19 huéspedes.

---

## 🔬 2. Protocolo de Auditoría Contable y Verificación

1.  **Moneda y Descuadres:**
    Normalizar siempre a USD en el dashboard general para evitar sumar DOP y USD. La tasa de conversión está fijada a `59.0` en la RPC de Supabase.
2.  **Conciliación Cruzada:**
    Evaluar periódicamente discrepancias de pagos huérfanos sin reserva asociada o reservas confirmadas que no registren abonos en `atlas_payments`.

---
*ATLAS-FINANCE · Aliun Travel SRL · Julio 2026*
