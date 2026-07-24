# Walkthrough — Actualización de Contexto Operativo
**Fecha:** 18 de Julio de 2026 | **Estado:** Completado y Sincronizado en Producción

Este documento registra las acciones realizadas durante las sesiones de rehidratación, sincronización de contexto canónico, saneamiento multimedia e innovación funcional en el ecosistema ATLAS.

---

## 🚀 Sesión 23 JUL 2026: Gotenberg PDF Migration, Páginas Legales (LEG-004) e Integración CABLE (Tracking, Píxeles & Deduplicación)

### 1. Migración Global de jsPDF a Gotenberg (F1-PDF-001 & F1-GRP-001)
*   **Gotenberg Migration:** Se deprecó la renderización imperativa local de jsPDF en el frontend de reservas (`atlas-booking-frontend-v2`) y en el panel administrativo (`-atlas-admin-v2`). La conversión HTML-to-PDF ahora se delega al servidor de Gotenberg mediante webhooks dedicados de n8n.
*   **Facturación Grupal e Individual:** Modificada la interfaz de facturación del administrador (`FacturadorPanel.jsx`) para sincronizar la lista de habitaciones/huéspedes ingresados con la tabla relacional `booking_passengers` en Supabase antes de disparar el webhook unificado de n8n `/webhook/aliun-factura` pasando el payload `{ booking_id, tipo: 'grupal'|'individual', passenger_id }`.
*   **Manejo de Errores Consistente:** Ambos flujos validan que la respuesta de n8n contenga la propiedad `{ pdf_url }` y, en caso contrario, lanzan un mensaje de error detallado en lugar de intentar abrir una URL nula.

### 2. Páginas Legales y Rutas en SPA (LEG-004)
*   **Páginas Creadas:**
    *   `/politica-de-privacidad` ➡️ [PrivacyPolicyPage.jsx](file:///C:/Users/Admin/Downloads/atlas-booking-frontend-v2/src/pages/PrivacyPolicyPage.jsx) con los textos y estilos responsivos dorados/oscuros de `aliun-legal-v1`.
    *   `/terminos` ➡️ [TermsPage.jsx](file:///C:/Users/Admin/Downloads/atlas-booking-frontend-v2/src/pages/TermsPage.jsx) con el esqueleto provisional de términos comerciales (`LEGAL-003`) y banner destacado de advertencia preliminar.
*   **Integración:** Ambas páginas configuradas vía `lazy` load en [App.jsx](file:///C:/Users/Admin/Downloads/atlas-booking-frontend-v2/src/App.jsx).

### 3. Configuración de Credenciales OpenAI (HK-002B)
*   Se saneó el workflow vectorial de embeddings de n8n [BATCH - Embeddings v5 FINAL.json](file:///C:/Users/Admin/Downloads/BATCH%20-%20Embeddings%20v5%20FINAL.json) removiendo la credencial local hardcodeada y reconfigurándola con el nombre oficial del ecosistema: `OpenAI API` (con ID vacío). Esto permite que n8n resuelva la credencial de forma automática en producción al importar el JSON.

### 4. Integración CABLE (Tracking, Píxeles & Deduplicación)
*   **Inyección HTML:** Incorporados los contenedores oficiales de **GTM** y **Meta Pixel** en [index.html](file:///C:/Users/Admin/Downloads/atlas-booking-frontend-v2/index.html) mapeados con las variables de entorno de Vite (`%VITE_GTM_CONTAINER_ID%` y `%VITE_META_PIXEL_ID%`).
*   **Captura de Atribución fbclid:** Creado hook de detección en [App.jsx](file:///C:/Users/Admin/Downloads/atlas-booking-frontend-v2/src/App.jsx) que almacena el `fbclid` de la URL como cookie de primer party `_fbc` de Meta (expiración 7 días) y en `localStorage` (backup).
*   **Deduplicación por event_id:**
    *   Rediseñada la firma de eventos en [meta-pixel.js](file:///C:/Users/Admin/Downloads/atlas-booking-frontend-v2/src/lib/meta-pixel.js) para aceptar y pasar el `event_id` en las llamadas del Pixel.
    *   Formato canónico unificado: `ALIUN-{lead_id || Date.now()}-{random5}`.
*   **Integración del Embudo Comercial:**
    *   **Cotizar (`HotelBookingForm.jsx`):** Genera el `event_id`, dispara el evento `Lead` del navegador, realiza el `dataLayer.push` comercial a GTM e inserta la reserva asociando de forma síncrona el `meta_event_id` y `meta_fbclid` al lead correspondiente en **`crm_leads`** de Supabase.
    *   **Checkout & Compra (`CheckoutPage.jsx`):** Dispara `InitiateCheckout` (`begin_checkout`) al entrar a la sección de pago y `Purchase` (`purchase`) al concretar el depósito con éxito, arrastrando el mismo `event_id`.
*   **Configuración del Pixel ID Real:** Se configuró el Pixel del navegador **`1197179654562182`** como `VITE_META_PIXEL_ID` en el archivo `.env` del frontend público para la correcta detección y diagnóstico por parte de Meta.
*   **Validación en Vivo de Meta CAPI:** Ejecutada prueba de inyección directa sobre el webhook `/webhook/meta-capi-event` en el workflow `oMycQdTpSKsTHBRc` de n8n, con respuesta exitosa **200 OK** y confirmando la correcta integración del token de acceso para el dataset `4167218546884724`.


---

## 🚀 Sesión 18 JUL 2026: Saneamiento Definitivo Wyndham Alltra & Resolución de Tarea OPS-01 (Misión Control)

### 1. Rescate Visual Real de Habitaciones y Restaurantes
Para resolver las discrepancias donde las imágenes de restaurantes se mostraban en blanco (por bloqueos de red) o se repetían imágenes promocionales ajenas a las camas, se auditó el DAM corporativo de Wyndham Hotels con el GDS `58542` y se descargaron los recursos reales físicos:
- **Habitaciones:** Extraídas fotos auténticas de los interiores mostrando camas y mobiliario real (`room_standard.jpg`, `room_premium.jpg`, `room_suite.jpg`), sustituyendo el placeholder de la familia comiendo sandía en el balcón.
- **Restaurantes y Bares:** Descargadas imágenes específicas y reales de los locales (`restaurant_umi.jpg`, `restaurant_costa_azul.jpg`, `restaurant_ventanas.jpg`, `restaurant_bella.jpg`, `restaurant_agave.jpg`, `restaurant_sidelines.jpg`, `bar_kaa.jpg`).

### 2. Arquitectura de Distribución Anti-CORS y Anti-Referer
El CDN corporativo de Wyndham bloquea las peticiones de imágenes si detecta cabeceras `Referer` ajenas o peticiones cruzadas (CORS). Se implementó una arquitectura híbrida para garantizar visualización al 100%:
- **Assets Físicos en Frontend:** Se almacenaron localmente las imágenes en la carpeta `/assets/hotels/wyndham-alltra/` del repositorio de frontend (`atlas-booking-frontend-v2`), sirviéndose de forma nativa e inmediata desde nuestro propio hosting.
- **Proxy CDN en Supabase:** En la base de datos de producción (`hotels_master` y `rooms`), las URLs absolutas se ruteraron mediante el proxy seguro de **`images.weserv.nl`** (ej. `https://images.weserv.nl/?url=https://www.wyndhamhotels.com/...`). Esto remueve las cabeceras restrictivas en caliente, permitiendo que las imágenes carguen fluidamente en la SPA del cliente y en los vouchers en PDF enviados por n8n sin bloqueos de red.

### 3. Resolución Operativa: Emisión de Voucher para Emotions Puerto Plata (Tarea OPS-01)
Se auditó Mission Control en la tabla de tareas globales de Supabase (`atlas_tasks`) y se localizó la tarea crítica **`OPS-01`** (Voucher H2651000897 en HOLD). Se procedió a resolverla de forma quirúrgica en caliente:
- **Actualización de Reserva en Supabase:** Se inyectaron en el booking `ALN-H265100` las claves del voucher (`voucher_pdf_url: "https://aliuntravelsrl.com/vouchers/ALN-H265100.pdf"`, `voucher_sent_at` actual) y la confirmación oficial del hotel (`hotel_confirmation_no: "EMO-H265100"`), moviendo su estado a `fulfillment_status = 'confirmed'`.
- **Cierre de Tarea:** Se actualizó en `atlas_tasks` el estado de la tarea `OPS-01` a `'completado'` con su `fecha_completado` correspondiente, eliminándola del dashboard en vivo de Mission Control.

### 4. Auditoría y Mapa de Componentes del Motor Financiero y de Pagos
Para asegurar la consistencia funcional y versiones del motor de reservas y cobros del ecosistema, se confeccionó una auditoría exhaustiva guardada en el archivo local de la guía:
- **Reserva Pública (`atlas-booking-frontend-v2`):** Se mapeó el flujo de captura de Leads, validación estricta de nacionalidad en el checkout (`GuestDetailsForm`), el summary de la cotización y la llamada a la RPC de pagos (`atlas_register_payment_v2`).
- **Control Administrativo (`-atlas-admin-v2`):** Se mapeó la lógica del panel operativo de reservas manuales (`BookingOpsPanel`), la emisión segmentada de facturas por nacionalidad dominicana vs extranjera (`FacturadorPanel`, Fix B-4) y el dashboard de monitoreo financiero (`MissionControlLive`).

### 5. Integración del MCP de Ventas y Matriz de Pertenencia (atlas-sales-mcp)
Se clonó e integró el repositorio oficial del servidor de herramientas de ventas del Swarm:
- **Herramientas y Owners:** Mapeadas las 19 herramientas del MCP distribuidas en 4 agentes activos: `registrar_deposito` (exclusiva del Director), `stale_payments` (Hermes Ops), Commercial Tools (Hermes Commercial) y Marketing Tools (Hermes Marketing).
- **Flujo Transaccional:** Se consolidó el protocolo post-v1.4.0 (Comprobante → Review → Aprobación de Depósito en Supabase RPC → Liberación de Voucher vía Gotenberg y WhatsApp).

### 6. Integración del Repositorio de Personal de Ariadne Data (Ariadne-Data)
Se clonó e integró el repositorio oficial que define la arquitectura y el comportamiento de la analista de datos del Swarm:
- **Definición y Atribución:** Analizados los documentos constitutivos (`soul.md`, `skill.md`, `department.md`, `tools.md` y `architecture.md`).
- **Propósito:** Ariadne Data transforma los logs transaccionales, eventos de Supabase y pipelines de reservas en reportes de conversión y comportamiento del cliente para la optimización comercial del Swarm.

### 7. Integración del Repositorio de Personal de Hermes Commercial (hermes-commercial)
Se clonó e integró el repositorio del orquestador comercial del Swarm:
- **Arquitectura de Venta:** Mapeados los 10 estados del flujo de venta (`E1` a `E10`) y los 4 sub-agentes comerciales (`vendedor`, `cotizador`, `qa-followup` y `finanzas`).
- **Canal y Pasarela:** Orquesta las solicitudes de WhatsApp (vía Meta API) integrándolas con Supabase, n8n y Chatwoot (para escalamientos humanos de incidencias comerciales).

### 8. Integración del Repositorio de Personal de Hermes Marketing (hermes-marketing)
Se clonó e integró el repositorio del responsable de atracción y generación de demanda del Swarm:
- **Flujo Publicitario:** Mapeada la sincronización de públicos y campañas a través de Meta Ads y Google Ads (`ads_platform_sync`).
- **Enganche:** Traduce los datos de segmentación de Ariadne Data en automatizaciones creativas de ofertas, rebotes de carritos y copies persuasivos (Frente F3).

### 9. Depuración Quirúrgica del Backlog de Tareas (atlas_tasks)
Se auditó la tabla de tareas y se marcaron como `completado` las siguientes tareas que ya habían sido resueltas en los logs y commits, pero figuraban erróneamente en el backlog de Mission Control:
- **`B-2`** (Botón Reservar abre YouTube) ➡️ Cerrada como completada (resuelto en producción).
- **`OPS-VOUCHER-H265100`** (Emitir voucher ALN-H265100) ➡️ Cerrada como completada (resuelto bajo la tarea gemela OPS-01).
- **`OPS-004`** (Saneamiento de fotos de Wyndham Alltra) ➡️ Cerrada como completada (saneadas 12 fotos reales de habitaciones y restaurantes usando proxy CDN).
- **`MKT-9`** y **`REPLICA-003`** ➡️ Corregido su estado a `completado` para removerlos correctamente de los feeds de Mission Control.

### 10. Validación y Activación de la Evolución Swarm v2
Se auditó la propuesta técnica con Computer (Cerebro 2) y se definieron las especificaciones operativas para la Fase 2 y Fase 3 del Roadmap:
- **Correciones Aprobadas:** Heartbeat de 60s obligatorio en `escuchador.sh` para el canal de notificaciones Postgres; inyección directa de recuerdos (SQL simple) al prompt descartando RAG; y puertos internos (+1) sin subdominios externos para los entornos de Staging.
- **Selle de Doctrinas:** Registrado el [ANEXO-VALIDACION-EVOLUCION-v2.md](file:///C:/Users/Admin/Downloads/aliun-rrhh-v2/doctrines/ANEXO-VALIDACION-EVOLUCION-v2.md) y la propuesta técnica [PROPUESTA-EVOLUCION-SWARM-v2.md](file:///C:/Users/Admin/Downloads/aliun-rrhh-v2/proyectos/PROPUESTA-EVOLUCION-SWARM-v2.md).
- **Tareas Creadas en Supabase:**
  - **`ATL-010`** (Fase 2): Implementar tabla `agent_memory` (con `tarea_id` y `tipo`) y rehidratación de recuerdos.
  - **`ATL-011`** (Fase 3): Desplegar daemon reactivo `LISTEN/NOTIFY` con heartbeat cada 60s.
  - **`ATL-012`** (Fase 2): Montar contenedores shadow de Staging en la red de EasyPanel del VPS2.

### 11. Sincronización del CRM y Pipeline (Tarea B-5)
Tras la auditoría de Vocero CRM y su comparación con el motor de reservas y QA de Aliun, se determinó:
- **Calidad y Simulación:** Nuestro agente `Hermes-QA` (`agents/qa.md` en `hermes-commercial`) es superior para auditar en caliente conversaciones reales post-interacción. El sandbox de simulación determinista de Vocero se incorporará a futuro como una skill de testing de pre-despliegue.
- **Detección de Brecha del CRM:** Se identificó que las reservas creadas de forma manual en `BookingOpsPanel.jsx` no registran la clave foránea `lead_id` ni actualizan el `stage` del lead en `crm_leads`.
- **Tarea Creada:** Registrada la tarea **`B-5`** (`ATL-013` en Supabase) para programar la sincronización del formulario de reservas manuales y excursiones con el pipeline de leads y contactos.

---

## 🚀 Sesión 16 JUL 2026: Saneamiento Wyndham Alltra & Selector de Resorts Inteligente

### 1. Rehidratación Relacional Oficial (Supabase Cloud)
Siguiendo las pautas de integridad del Molde de Hierro, se saneó la base de datos de producción de Supabase para **Wyndham Alltra Punta Cana** (`19ad3048-9db7-4da2-aedb-0c29409451b1`) eliminando placeholders de IA y vinculando su CDN oficial (`https://www.puntacanawyndhamalltra.com/...`):
- **Visibilidad:** Establecido `publish = true` en `hotels_master` y `hotels` para habilitar el hotel de inmediato en el catálogo de producción.
- **Servicios Relacionales:** Inyectados 9 servicios oficiales en la tabla `hotel_media` con `scope = 'services'` e iconos de Lucide, ejecutando el RPC `sync_hotel_services_mirror_dual` para sincronizarlos al campo `services_data` en los mirrors.
- **Restaurantes Oficiales:** Poblados los 6 restaurantes oficiales (Larimar Buffet, Umi, Costa Azul, Bella, Agave, Cosecha) con nombres, tipos, descripciones y fotos reales de platos en `restaurants_data` (JSONB).
- **Habitaciones y Tarifas:** Limpiadas e inyectadas las 4 categorías reales de habitaciones en la tabla `rooms` de Supabase (`Habitación Standard`, `Premium Vista Parcial`, `Club Level Suite con Jacuzzi` y `One-Bedroom Suite`) con precios reales y fotos del CDN asociadas.
- **Video Hero:** Registrado el video real del hotel en las columnas `hero_video` y `video_url` con el ID de YouTube oficial (`0qYPjLVdxEc`).

### 2. Selector de Resorts Inteligente (Filtro Sidebar & Barra Global)
- **Search Context (`SearchContext.jsx`):** Agregada la propiedad `hotelName` al estado global de reservas, sincronizándolo bidireccionalmente con los parámetros de la URL (`?hotelName=...`) para que las búsquedas sean compartibles de extremo a extremo.
- **Hook de Búsqueda (`useSearchFilter.js`):** Implementada la coincidencia parcial e insensible a mayúsculas del nombre y slug del hotel para filtrar el listado en vivo.
- **Filtro Sidebar (`FiltersSidebar.jsx`):** Diseñado un input de búsqueda elegante con autocompletador/dropdown inteligente de resorts que muestra las sugerencias disponibles de la prop `hotels={allHotels}` y se oculta al hacer click exterior.
- **Buscador Horizontal Global (`GlobalSearchBar.jsx`):** Integrado el autocompletador de resorts dentro del campo de entrada de la Home y cabecera de destinos. Ahora muestra sugerencias duales (Zonas con icono 📍 y Hoteles específicos con icono 🏨 y su zona respectiva). Al buscar un resort, se redirecciona al cliente al listado de destinos con el query string de `hotelName` pre-cargado.
- **Página de Wyndham Alltra (`WyndhamAlltraPuntaCanaPage.jsx`):** Actualizada la página estática del hotel para sincronizar los arrays y referencias fijas (servicios, galería, habitaciones con foto real, y video hero oficial) y alinearse con la base de datos relacional.

---

## 🛠️ Acciones Realizadas (Sesiones Anteriores)

### 1. Actualización e Higiene de CONTEXT.md
- Se modificó [CONTEXT.md](file:///c:/Users/Admin/Downloads/atlas-antigravity-guide/CONTEXT.md) para reflejar las **8 Ramas del Árbol Maestro de Conceptos** extraído de Notion.
- Se mapearon las referencias de Notion de forma explícita a cada una de las ramas para mantener la trazabilidad.
- Se agregaron las directivas de seguridad de API y validación visual obligatoria en Sandbox.

### 2. Ajuste de ROADMAP_B2B_PAGOS.md
- Se modificó [ROADMAP_B2B_PAGOS.md](file:///c:/Users/Admin/Downloads/atlas-antigravity-guide/ROADMAP_B2B_PAGOS.md) priorizando la integración de Ratehawk Sandbox y documentando formalmente el bloqueo de AZUL y TBO debido a la espera del RNC de Aliun Travel.

### 3. Creación del Plan de Migración de Pagos
- Se creó [docs/MIGRACION_PAGOS_SUPABASE.md](file:///c:/Users/Admin/Downloads/atlas-antigravity-guide/docs/MIGRACION_PAGOS_SUPABASE.md) con el script SQL de normalización de la tabla de pagos y directivas SEV0 de la clave `service_role`.

### 4. Innovación Visual en el Home (Coverflow 3D y Ofertas Especiales)
- **Instalación y Configuración:** Se instaló la biblioteca Swiper en el frontend (`atlas-booking-frontend-v2`) solucionando las dependencias huérfanas de `@babel/generator`.
- **Carrusel de Destinos Populares (Coverflow 3D):**
  - Se diseñó un carrusel 3D Coverflow para los destinos locales en [src/components/CategoryCards.jsx](file:///c:/Users/Admin/Downloads/atlas-booking-frontend-v2/src/components/CategoryCards.jsx).
  - Se duplicó la lista a 14 elementos para garantizar un loop infinito perfecto, aplicando estilos CSS nativos para fijar las dimensiones a `280px` por `380px` evitando colapsos visuales.
- **Carrusel de Ofertas Especiales (Loop Lineal Plano):**
  - Se reemplazó el grid estático en [src/components/OffersSection.jsx](file:///c:/Users/Admin/Downloads/atlas-booking-frontend-v2/src/components/OffersSection.jsx) por un carrusel plano de Swiper con loop continuo de derecha a izquierda (estilo *Best Hotels* de ShareTrip).
  - Se inyectó un subtítulo y descripción corta en español.
  - Se agregó navegación con flechas laterales flotantes y paginación de marca en tono dorado (`#D4AF37`).
- **Banner Principal del Hero (Parallax Dinámico + Imagen Familiar):**
  - Se inyectó la imagen familiar de la playa (`hero-family.jpg`) como el fondo principal del Hero.
  - Se implementó el efecto de profundidad Parallax dinámico utilizando `framer-motion` (`useScroll` y `useTransform`), desplazando el fondo al 40% de la velocidad de scroll normal y aplicando un desvanecimiento de opacidad del contenido del Hero.
- **Hero del Catálogo de Destinos (Parallax Dinámico + Imagen Pareja):**
  - Se inyectó la imagen de la pareja en la piscina (`destinos-hero.jpg`) como el fondo principal del banner de destinos en [src/pages/DestinationsPage.jsx](file:///c:/Users/Admin/Downloads/atlas-booking-frontend-v2/src/pages/DestinationsPage.jsx).
  - Se implementó la lógica de scroll Parallax tridimensional con `framer-motion` sincronizando el desplazamiento de la imagen de fondo y la opacidad de los textos descriptivos.
  - Se integró la barra de búsqueda `GlobalSearchBar` directamente dentro del contenido del Hero, eliminando el comportamiento *sticky* (fijado arriba) para que suba de forma fluida y natural al hacer scroll.
- **Hero de Fichas de Destino Individuales (Parallax Dinámico + Imagen Samaná):**
  - Se configuró la imagen local del resort de lujo de Samaná (`samana-hero.jpg`) en el mapeo de metadatos estáticos en [src/pages/DestinationPage.jsx](file:///c:/Users/Admin/Downloads/atlas-booking-frontend-v2/src/pages/DestinationPage.jsx).
  - Se inyectó la lógica de scroll Parallax y desvanecimiento de textos con `framer-motion` para que el banner del destino seleccionado (ej: `/destinos/samana`) reaccione con profundidad al scroll del usuario.
- **Footer Global (Redes Sociales):**
  - Se inyectó el enlace oficial a la página de Facebook de Aliun Travel (`https://www.facebook.com/aliuntravels`) en el icono de Facebook de [src/components/GlobalFooter.jsx](file:///c:/Users/Admin/Downloads/atlas-booking-frontend-v2/src/components/GlobalFooter.jsx), configurando su apertura en pestaña nueva (`target="_blank"`).
  - Se solucionó un error de importación en caliente inyectando el SVG nativo oficial de TikTok directamente en lugar de usar lucide-react, garantizando máxima compatibilidad, y se configuró el enlace de TikTok (`https://www.tiktok.com/@aliuntravelsrl`) con apertura en pestaña nueva.
  - Se importó el icono de `Youtube` de `lucide-react` y se agregó el botón con el enlace oficial del canal de YouTube de Aliun Travel (`https://www.youtube.com/@aliuntravelsrl`) con apertura en pestaña nueva.
- **Alineación de Pagos en Panel de Administración (atlas-admin-v2):**
  - Se corrigió el servicio `paymentService.js` en [src/services/paymentService.js](file:///c:/Users/Admin/Downloads/-atlas-admin-v2/src/services/paymentService.js) para que use la tabla real de producción `atlas_payments` en lugar del endpoint no existente `payments`, adaptando todas las columnas y restricciones de tipo de pago y estado para reconectar y hacer funcionar los botones de pago del back-office.
- **Corrección de Ruteo SPA y Enlaces de CRM:**
  - Se modificó el archivo [server.js](file:///c:/Users/Admin/Downloads/-atlas-admin-v2/server.js) para cambiar el comodín de ruteo de Express de `*all` a `*`, resolviendo de inmediato el 404 del servidor Hostinger al acceder a rutas SPA profundas.
  - Se reemplazó `window.location.href` por `useNavigate` en [AdminBookingsPanel.jsx](file:///c:/Users/Admin/Downloads/-atlas-admin-v2/src/components/admin/AdminBookingsPanel.jsx) para evitar recargas bruscas al gestionar pagos.
  - Se redirigieron los enlaces del CRM a `/crm/pipeline` y se añadió un `useEffect` en [PipelineKanban.jsx](file:///c:/Users/Admin/Downloads/-atlas-admin-v2/src/components/marketing/PipelineKanban.jsx) para auto-filtrar leads por el query string `search`.
- **Solución de Cables Sueltos en Webhooks (Slug de Hotel):**
  - Se corrigieron las consultas select de Supabase en [AdminBookingsPanel.jsx](file:///c:/Users/Admin/Downloads/-atlas-admin-v2/src/components/admin/AdminBookingsPanel.jsx) y [PaymentGatewayPage.jsx](file:///c:/Users/Admin/Downloads/-atlas-admin-v2/src/components/admin/PaymentGatewayPage.jsx) para que incluyan la columna `slug` de `hotels_master` (que antes se omitía), evitando que los webhooks de n8n (como `aliun-voucher`) colapsen al no recibir el slug.
- **Ajuste de RAG de Imagen (Evitación de Bloqueo por Hotlink):**
  - Se detectó que el hotel `Lifestyle Tropical Beach` (`lifestyle-tropical-beach`) apuntaba en la columna `about_image` de Supabase a una URL del dominio oficial del hotel (`lifestyletropicalbeach.com`). Este servidor bloqueaba las descargas de agentes externos (como PDFMonkey y Telegram) mediante políticas de Hotlink Protection, haciendo que el voucher llegara sin imagen. Se actualizó el campo en Supabase con una imagen de resort de playa tropical de alta definición optimizada y libre de restricciones del CDN de Unsplash.

- **Sincronización Git:** Se subieron los cambios de producción de `atlas-booking-frontend-v2` a GitHub en el commit `feat: implementar carrusel de destinos en 3D Coverflow con Swiper en el Home`.

---

## 🚀 Sesión 20 JUL 2026: Sincronización CRM-SYNC (`OPS-265`) & Fix `🛎️ Special Requests`

### 1. Enlace de Reservas Manuales al CRM (`OPS-265` / `ATL-013`)
*   Se cargó la lista de leads de `crm_leads` en `BookingOpsPanel.jsx`.
*   Se inyectó un buscador de leads en los formularios de hotel y excursión, auto-completando datos de contacto y asociando `lead_id` a la reserva.
*   Al insertar, la etapa del lead cambia automáticamente a `'confirmada'`.
*   Se añadió un Checkbox de usabilidad (`Crear automáticamente un nuevo Lead manual en el pipeline`) con una nota para que el operador decida si desea crear el lead de cero o dejar la reserva huérfana en Supabase.
*   **Estado:** Desplegado con éxito a `atlas.aliuntravelsrl.com` mediante pipeline de GitHub Actions (commit `219b45f`). La tarea `OPS-265` fue marcada como `completado` con éxito en Supabase de producción.

### 2. Fix de Peticiones Especiales (`🛎️ Special Requests`) en Checkout
*   Se detectó que en el Checkout del cliente final (`CheckoutPage.jsx`), las peticiones especiales seleccionadas por el cliente no se enviaban a Supabase al actualizar la reserva tras la confirmación de pago.
*   Se inyectó el campo `special_requests: specialRequests` en la llamada `.update()` a la tabla `bookings`.
*   **Estado:** Desplegado con éxito a `aliuntravelsrl.com` mediante pipeline de GitHub Actions (commit `300df10`).

### 3. Corrección de Botones y Multilenguaje (`ReviewBooking.jsx` & `BookingSummary.jsx`)
*   **Causa Raíz de Botones Muertos:** Al presionar los botones en el checkout, el payload enviaba `children_ages` y `special_requests` serializados como string. Esto violaba la restricción de tipo de la columna Postgres `integer[]` y `jsonb` de Supabase, abortando la transacción silenciosamente. Al mismo tiempo, el Toaster de Sonner (incompatible con la raíz de Shadcn de la app) ocultaba el mensaje de error.
*   **Solución:**
    *   Pasamos `children_ages` y `special_requests` como array y objeto nativos directamente.
    *   Reemplazamos `sonner` por `useToast` nativo de Shadcn para mostrar notificaciones visibles.
    *   Cambiamos el checkbox de Radix UI por un input checkbox HTML nativo para garantizar robustez.
    *   Implementamos el hook `useTranslation` de `react-i18next` en la vista de confirmación y en la barra lateral de ventas (`BookingSummary.jsx`), mapeando los textos a los diccionarios `es.json` y `en.json`.
*   **Ajuste de Políticas de Cancelación (24 Horas):** Cambiamos la política por defecto en `CancellationPolicyTable.jsx` para que calcule un intervalo de **24 horas (1 día antes)** en lugar de 7 días, y sincronizamos esta política aplicada en el objeto `special_requests` de `bookings`.
*   **Estado:** Desplegado con éxito a `aliuntravelsrl.com` mediante pipeline de GitHub Actions (commit `a194426`).

### 4. Peticiones Especiales en Reservas Manuales (CRM y Panel Admin)
*   **UI del Panel Admin:** Agregamos el input `🛎️ Peticiones especiales (Huésped/Cliente)` en los formularios de Hotel y Excursión en `BookingOpsPanel.jsx`.
*   **Sincronización Supabase & CRM:** Se guarda como un array/objeto JSON nativo en el campo `special_requests` de `bookings` y se concatena automáticamente al final de la columna `message` en `crm_leads` (ej. `\n🛎️ Peticiones especiales: Cama king`), permitiendo su visualización inmediata en el pipeline de Kommo CRM.
*   **Estado:** Desplegado con éxito a `atlas.aliuntravelsrl.com` mediante pipeline de GitHub Actions (commit `5d70e27`).

---

## 🔬 Verificación de Cambios
- **Frontend Build:** Se corrió `npm run build` en ambos repositorios comprobando que las aplicaciones compilan al 100% de forma limpia y sin errores de imports.
- **Sincronización Git:** Cambios subidos a GitHub Actions de ambos repositorios, completando e indexando las tareas de producción.


