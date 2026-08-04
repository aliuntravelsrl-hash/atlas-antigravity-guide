# Execution Workspace — Índice General

> Espacio de trabajo exclusivo para el registro de ejecución de programas activos del COS.
> **No es un documento de orientación.** Contiene información operativa mutable.

---

## Regla de uso

- `MAPA_GPS` y `MASTER-INDEX` son documentos de orientación atemporal — no se escriben aquí.
- Todo avance, decisión, observación e hito de un programa se registra en su carpeta bajo `programs/`.
- Un programa = una carpeta. Misma estructura interna en todos.

## Estructura

```
execution/
├── README.md          ← este archivo — índice general
├── portfolio.md       ← estado consolidado de todos los programas
└── programs/
    ├── KPS/           ← Kernel Projection System (activo)
    ├── CRM/           ← (reservado)
    ├── Hospitality/   ← (reservado)
    ├── Marketing/     ← (reservado)
    └── Intelligence/  ← (reservado)
```

## Plantilla por programa

Cada programa contiene:

| Archivo | Contenido |
|---------|-----------|
| `README.md` | Ficha del programa — objeto, alcance, firmas |
| `EXECUTION_LOG.md` | Bitácora diaria de avances |
| `DECISIONS.md` | Decisiones de implementación con contexto y evidencia |
| `OBSERVATIONS.md` | Observaciones técnicas locales al programa |
| `MILESTONES.md` | Hitos de Sprint — estado, fecha, evidencia |
| `evidence/` | Capturas, JSONs, logs, artefactos de verificación |

---
*Creado: 2026-08-03 · Autorizado por UI-OVR-002 · Director: ALIUN TRAVELSRL*
