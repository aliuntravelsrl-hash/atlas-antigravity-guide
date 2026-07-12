# 🧠 CONTEXT — Antigravity (ATLAS Ecosistema)
**Actualizado:** 12 JUL 2026

Guía de rehidratación rápida de contexto operativo y memoria de estado para Antigravity al iniciar cualquier sesión de pair programming en el ecosistema ATLAS.

---

## 🤖 1. Identidad y Rol Operativo

- **Nombre:** Antigravity — Agente Técnico Externo Soberano de ATLAS.
- **Foco:** Terminal, Docker, repositorios, deploys y cableados de infraestructura.
- **Director:** Aldo Hilario | Aliun Travel SRL | República Dominicana.
- **Idioma de trabajo:** Español obligatorio.
- **Límite de rol:** Antigravity NO habla con clientes finales. Es ingeniería pura.

---

## 🖥️ 2. Infraestructura

| Componente | Detalle |
|---|---|
| **Supabase** | `oyihiyivdhfxpyiwnmqk` — SSOT absoluto |
| **n8n** | `https://n8n-n8n.xaruuo.easypanel.host` |
| **VPS 1** | `72.61.12.170` — EasyPanel, n8n, frontends (expira 2027-11-21) |
| **VPS 2** | `2.24.198.231` — Docker Swarm con 5 agentes Hermes (expira 2027-04-14) |
| **Chatwoot** | `https://n8n-chatwoot.xaruuo.easypanel.host` |
| **Token Chatwoot** | `T5YDT4onGs3wve7wHf3fZXpr` |
| **MCP Server** | `https://n8n-atlas-sales-mcp.xaruuo.easypanel.host` (v1.5.0, 20 tools) |
| **OpenClaw UI** | `https://musculo-aliun.tail02a864.ts.net/` (solo vía Tailscale) |
| **Tailscale** | desktop=`100.102.102.116` · musculo=`100.100.201.7` · note20=`100.110.83.1` |

---

## 📂 3. Repositorios bajo responsabilidad de Antigravity

| Repo | Propósito |
|---|---|
| `atlas-booking-frontend-v2` | Web pública de reservas (`aliuntravelsrl.com`) |
| `atlas-admin-v2` | Panel de administración (`atlas.aliuntravelsrl.com`) |
| `atlas-sales-mcp` | Servidor MCP de herramientas de ventas |
| `atlas-cableados` | Documentación de infraestructura y redes |
| `atlas-marketing-v2` | Motor de ofertas de marketing (en construcción) |
| `aliun-rrhh-v2` | Doctrina y SOUL.md de agentes Hermes |
| `musculo-vps` | Scripts y compose de VPS2 |
| `atlas-antigravity-guide` | **Este repo** — contexto y guía de Antigravity |

---

## 🐳 4. Swarm VPS2 — 5 Agentes Hermes

| Slug | Puerto | Estado |
|---|---|---|
| `hermes-ops` | 32769 | ✅ Producción activa |
| `hermes-commercial` | 32770 | ✅ Activo |
| `hermes-marketing` | 32771 | ✅ Activo |
| `ariadne-data` | 32772 | ✅ Activo |
| `hermes-qa` | 32773 | ✅ Activo |

**Path de stacks en VPS2:** `/opt/data/atlas-hermes-v2/stacks/{slug}/`

Archivos relevantes por stack: `SOUL.md`, `REHIDRATACION.md`, `ALCANCE.md`

---

## 📈 5. Estado de avance — 12 JUL 2026

### ✅ Completado esta sesión (Fase 1: Puerto Plata, Juan Dolio y Boca Chica)

**Estabilización y Seguridad del Frontend (F5-SEC)**
- Actualización de seguridad del core a **Vite 6** (`vite@6.4.3`) y **React Router DOM v6** (`react-router-dom@6.30.4`) para mitigar vulnerabilidades.
- Configuración de la Notion API Key actual de julio 2026 en los archivos `.env` locales.
- Consolidación de Chatwoot como canal de comunicación oficial del agente (Kommo CRM degradado).

**Rescate Visual y Enriquecimiento de Hoteles (Health Score 7/7 — 🟢 READY TO SELL)**
Se completó la depuración de placeholders genéricos, inyección de fotos reales, normalización de datos (habitaciones, temporadas, tarifas 2026, políticas y restaurantes) en Supabase para los siguientes complejos:
1.  **Lopesan Caoba Lagoon Resort, Spa & Casino** (`20f915c7-8e2e-41ab-9ace-fec759b64f51`)
2.  **Be Live Collection Marien** (`332657c0-a75b-4580-8a77-55f058ff01ec`)
3.  **Sunscape Coco Punta Cana** (`8bbc304d-fb21-4ee2-a78c-bceb90583c4f`)
4.  **Sunscape Dominicus La Romana** (`2080f7c9-2ffb-4ff0-9781-d8341b866765`)
5.  **Bahia Principe Explorer Legend** / *Fantasia Punta Cana* (`2490ce7a-02fe-495c-a9cb-3f0565a4bd2a`)
6.  **Bahia Principe Luxury Esmeralda** (`db00338f-9404-4cbf-a2fb-98479ab47bed`)
7.  **Bahia Principe Grand Aquamarine** (`d54473c3-9964-458c-a26c-72a0c1407e10`)
8.  **Bahia Principe Luxury Ambar** (`c84e6f0d-90e2-4eec-a19a-3bbb9c0ea9e0`)
9.  **Bahia Principe Grand Punta Cana** (`2db11109-511d-47c2-9ed3-1c31f6392b70`)
10. **Bahia Principe Grand La Romana** (`d0383dcd-47c9-48d8-bed8-c8012c3f9a1a`)
11. **Barceló Bávaro Palace** (`2d49fd1f-0de0-4c61-bdd9-f7bdb5542ab5`)
12. **Occidental Caribe** (`20f915c7-8e2e-41ab-9ace-fec759b64f51`)
13. **Occidental Punta Cana**
14. **Barceló Bávaro Beach (Adults Only)**
15. **Coral Costa Caribe Beach Resort** (`6bccee79-9f3a-4f68-9873-a9d3f33722b8`)
16. **Emotions by Hodelpa Juan Dolio** (`6a8ca8ea-7193-4adf-81ec-a1a423d961fb`)
17. **Hodelpa Garden Suites** (`92c1d70a-1333-4810-af87-9541de83480a`)
18. **Santo Domingo Bay** (`33e19288-e397-4581-a7fe-3f251c6610ca`)
19. **HM Whala! Boca Chica** (`b54c6215-8fbd-44f5-af21-32c77b633225`)
20. **VH Gran Ventana, Viva Heavens & Viva Wyndham Tangerine** (Depuración de placeholders y restauración de galería/gastro).
21. **BlueBay Villas Doradas & Casa Marina Beach & Reef** (Consolidación de habitaciones y gastronomía).
22. **Iberostar Costa Dorada** (Mapeo de habitaciones comerciales y tarifario 2026 activo).
23. **Cofresi Palm Beach & Spa Resort** (Refinamiento de 6 restaurantes y 2 bares).
24. **Lifestyle Tropical Beach Resort & Spa** (Habitaciones y tarifas 2026 inyectadas, 5 restaurantes, galería masiva de 23 slides reales).
25. **Presidential Suites By Lifestyle Puerto Plata** (Creado desde cero con 4 habitaciones reales, tarifas 2026, alineación de geolocalización/zona y 10 restaurantes hermanos unificados).
26. **Playa Bachata Spa Resort** (Habitaciones, tarifas de niños y 5 restaurantes enlazados de forma segura con proxy HTTPS `images.weserv.nl` contra mixed content).

---

## 📋 6. Pendientes críticos — próxima sesión

### P1 — Rescate Visual Fase 2: Samaná, La Romana, Miches y Punta Cana
Iniciar el saneamiento de las galerías e inventario del resto de zonas de República Dominicana en Supabase, comenzando por el hotel **Viva V Samana** (`v-samana-viva`).

### P2 — Resolución de Conflicto en `atlas_payments`
Corregir discrepancias entre la estructura plana obsoleta y la estructura normalizada relacional de producción con vinculación FK a `bookings.id`.

### P3 — Fase Inicial de Adaptadores B2B (Ratehawk Sandbox)
Comenzar la Fase 0/1 del roadmap de B2B: validación de credenciales sandbox de Ratehawk y creación del adaptador de búsqueda en n8n.

### P4 — Continuación de Rescate Visual de Hoteles (Supabase)
Mantener la migración de fotos de hoteles maestros con imágenes rotas hacia el bucket `hotel-media` y actualizar `gallery_urls` en Supabase.

---

## ⚙️ 7. Reglas innegociables

| Regla | Detalle |
|---|---|
| **Supabase** | Verificar registros existentes ANTES de cualquier inyección |
| **Docker** | NUNCA eliminar volúmenes persistentes |
| **Infraestructura** | Todo cambio de red o Docker → documentar en `atlas-cableados` |
| **Tasa de cambio** | Siempre dinámica desde tabla `exchange_rates` en Supabase |
| **Identidad de agentes** | No resolver contradicciones de identidad sin confirmación del Director |
| **Legacy** | Ignorar repos y docs anteriores a este repo. Este es el SSOT de contexto |

---

## 🔗 8. Referencias rápidas

| Recurso | Valor |
|---|---|
| Supabase project | `oyihiyivdhfxpyiwnmqk` |
| Notion War Room root | `346293f4-6b24-811a-9c56-ddf2ce0fd7a7` |
| WF Notificación reserva (Telegram) | chatId `683265740` |
| WF Excursion doc webhook | `aliun-excursion-doc` |
| WF IDs activos | ATLAS-SALES=`0ps4wRmBFXcAy0u2` · COTIZACION=`Da46ZVQGRpdgaI02` · VOUCHER=`XwCARt2qk6U-dvz_RgYy7` |
