# Sistema Cronos HGP B2B
## Versión Desconectada v3 · Telefónica · QCS

Sistema de gestión de compras B2B para el seguimiento de oportunidades, tareas de aprovisionamiento, workflow de etapas y notificaciones. Desarrollado por el equipo QCS como herramienta on-premise para el área de Compras de Telefónica.

---

## Stack tecnológico

| Capa | Tecnología |
|------|-----------|
| Backend | Java 21 · Spring Boot 3.x |
| Frontend | Angular 17+ |
| Base de datos | MySQL 8 |
| Seguridad | JWT · Roles (PM, COMPRADOR, CONTROL_GESTION, VENDOR, SOPORTE, ADMIN) |

---

## Módulos implementados (13 HUs · 5 Sprints)

| Sprint | HU | Módulo |
|--------|----|--------|
| S1–S2 | HU-03, HU-05 | Tareas automáticas: Gestión OBYC y Kick Off Entrega |
| S2 | HU-01, HU-04, HU-09 | Campos de compra, PxQ tabla contable, Gestión Cesta |
| S3 | HU-08, HU-10, HU-17 | Formulario FOD, Tarea Derivada, Tarea Presupuesto |
| S4 | HU-16, HU-20, HU-21, HU-22 | Repositorio documental, Workflow, Vista PM |
| S5 | HU-24 | Motor de notificaciones (campana + correo SMTP) |

---

## Guías de pruebas UAT

### 📋 [Guía de Pruebas — Fase 1](Guia_Pruebas_Fase1_Cliente)
**Sprints 1, 2 y 3 · 8 Historias de Usuario**  
Cubre las HUs: HU-01, HU-03, HU-04, HU-05, HU-08, HU-09, HU-10, HU-17  
Módulos: Gestión OBYC · Presupuesto · Kick Off Entrega · Campos de Compra · Gestión Cesta · Tarea Derivada · Formulario FOD  
Fecha: 06/04/2026 · Versión 1.0

---

### 📋 [Guía de Pruebas — Fase 2](Guia_Pruebas_Fase2_Cliente)
**Sprints 4 y 5 · 5 Historias de Usuario**  
Cubre las HUs: HU-16, HU-20, HU-21, HU-22, HU-24  
Módulos: Repositorio Documental · Workflow de Etapas · Vista PM · Notificaciones  
Fecha: 13/04/2026 · Versión 1.1

---

## Acceso al sistema (entorno local)

- **Frontend:** http://localhost:4201  
- **Backend API:** http://localhost:8081  
- **Usuario admin:** `admin` / `Admin2024`  
- **Usuario vendor:** `vendor.hgp` / `Admin2024`

> El backend requiere MySQL 8 activo (`docker compose up -d`) y arrancar con `iniciar-backend.bat`.

---

*Documentación técnica — Cronos HGP B2B · QCS · 2026*
