# Manual de Operación: Inyección Dinámica de Analytics e Inmunidad SSG
**Versión:** 1.0.0-Release (2026-07-29)  
**Mapeo de Frente:** F1-FRONTEND (público) y SEO-ANALYTICS-001

---

## 📌 1. Descripción del Problema Técnico
En versiones previas, los scripts de **Google Tag Manager (GTM)** y **Meta Pixel (Facebook)** estaban insertados de forma estática en el archivo `index.html`. 

Al ejecutar el proceso de compilación estática (SSG/Pre-rendering con `react-snap` que levanta Puppeteer en local a través de `http://localhost:3189`):
1. El navegador headless de Puppeteer cargaba e inicializaba estos píxeles de analíticas.
2. Los scripts de tracking inyectaban elementos dinámicos en el DOM y disparaban pageviews de prueba.
3. Puppeteer capturaba el DOM serializado (con los scripts inyectados y modificados en caliente) y los guardaba como los archivos estáticos definitivos en `dist/**/*.html`.
4. Esto provocaba **dos fallos críticos**:
    *   **Contaminación de HTML:** Los archivos index estáticos desplegados en producción contenían código y scripts basura hardcodeados.
    *   **Errores JS en Consola:** Excepciones en tiempo de ejecución como `Uncaught TypeError: a.__fbeventsModules[e] is not a function` y desajustes de hidratación de React.

---

## 💻 2. Arquitectura de Solución Implementada

Se implementó una arquitectura de inyección en caliente modularizada y sensible al entorno.

### A. Limpieza de Plantilla Base
Se eliminaron por completo las etiquetas `<script>` y `<noscript>` estáticas de GTM y Meta Pixel de:
*   [index.html](file:///C:/Users/Admin/Downloads/atlas-booking-frontend-v2/index.html)

### B. Módulo de Inyección Dinámica
Se creó el archivo [analytics.js](file:///C:/Users/Admin/Downloads/atlas-booking-frontend-v2/src/analytics.js), que expone la función `initAnalytics()`:
```javascript
export function initAnalytics() {
  // 1. Evitar inyección fuera de producción
  if (!import.meta.env.PROD) {
    console.log('📊 [Analytics] Modo de desarrollo activo. Omitiendo scripts de seguimiento.');
    return;
  }

  // 2. Filtro de exclusión de entornos locales y pre-renderizado SSG
  const host = window.location.hostname || '';
  const ua = window.navigator.userAgent || '';
  if (
    host === 'localhost' || 
    host === '127.0.0.1' || 
    host.startsWith('192.168.') || 
    host.includes('local') || 
    ua.includes('ReactSnap')
  ) {
    console.log('📊 [Analytics] Omitiendo inicialización en localhost/prerender para evitar falsos hits o errores SSG.');
    return;
  }

  // 3. Inyección en caliente de GTM y Meta Pixel
  // ... Código de carga asíncrona mediante document.createElement('script') ...
}
```

### C. Inicialización en Punto de Entrada
Se importa e invoca en [main.jsx](file:///C:/Users/Admin/Downloads/atlas-booking-frontend-v2/src/main.jsx) inmediatamente después del arranque de la app:
```javascript
import { initAnalytics } from './analytics';

initAnalytics();
```

---

## 📋 3. Protocolo de Auditoría y Verificación de Analíticas

Para asegurar que los scripts de tracking no vuelvan a inyectarse en los HTML estáticos durante futuras compilaciones:
1.  **Limpiar directorio dist:** Eliminar cualquier compilación previa.
2.  **Correr Compilación Secuencial Directa:** En Windows PowerShell, evitar el operador de cadena del package.json que salta Vite. Ejecutar directamente:
    ```bash
    npx vite build
    node scripts/prerender.js
    ```
3.  **Auditar HTML generado:**
    *   Inspeccionar el archivo `dist/index.html` resultante.
    *   Verificar que **NO** existan ocurrencias de `connect.facebook.net/en_US/fbevents.js` ni de `googletagmanager.com/gtm.js`.
    *   Verificar que no existan llamadas directas a `fbq('init')` o `gtag('config')` inyectadas en duro en el código estático.
