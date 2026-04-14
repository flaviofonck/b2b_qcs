---
tags:
  - tipo/guia-pruebas
  - modulo/general
  - fase/fase2
  - sprint/sprint4
  - sprint/sprint5
  - rol/qa
  - rol/cliente
  - rol/pm
  - estado/completado
hu-relacionadas:
  - HU-16
  - HU-20
  - HU-21
  - HU-22
  - HU-24
related-docs:
  - "[[Guia_Pruebas_Fase1_Cliente]]"
  - "[[Plan_Implementacion_Fase2]]"
---

# Guía de Pruebas — Fase 2
## Sistema Cronos HGP B2B · Versión Desconectada v3
### Sprints 4 y 5 — 5 Historias de Usuario

**Fecha:** 13/04/2026  
**Preparado por:** Equipo de Desarrollo QCS  
**Versión documento:** 1.1  

---

> **Acceso al sistema**
> - **Frontend:** http://localhost:4201
> - **Usuario ADMIN/PM:** `admin` / `Admin2024`
> - **Usuario VENDOR:** `vendor.hgp` / `Admin2024`
> - **Backend API:** http://localhost:8081
> - **Prerequisito:** Backend corriendo (`iniciar-backend.bat`) y BD MySQL 8 activa (`docker compose up -d`).
>
> **Datos de prueba:** Se recomienda tener al menos una oportunidad de tipo **PUNTUAL** y una de tipo **GENERAL** con sus tareas de OBYC, Presupuesto y Kick Off ya cerradas (ver Guía de Pruebas Fase 1 para crearlas).

> **Estado de validación (actualización 13/04/2026)**
> - **Validación técnica interna:** ✅ Completa para HU-16, HU-20, HU-21, HU-22 y HU-24.
> - **Aprobación UAT cliente:** ⏳ Pendiente de ejecución y firma por PM Telefónica.
> - **Evidencia técnica:** `docs/files/specs/validacion_e2e_2026-04-13.md` y `docs/files/specs/auditoria_fase2_2026-04-13.md`.

---

## Sprint 4 — HU-16 + HU-20 + HU-21 + HU-22

---

### HU-16 — Repositorio Central de Documentos Adjuntos

**Descripción:** Cada oportunidad tiene una pestaña **"Documentos Adjuntos"** que centraliza todos los archivos subidos en las distintas tareas. El repositorio permite consultar, subir, descargar y eliminar documentos, y mantiene un log de auditoría exportable en CSV.

#### Caso 16-01 — Visualizar documentos adjuntos de una oportunidad

**Precondición:** Existe una oportunidad con al menos una tarea que tenga documentos adjuntos.

**Pasos:**
1. Navegar a **Repositorio** (`/repositorio/{oppId}`) o acceder desde la vista de detalle de la oportunidad.
2. Observar la tabla de documentos.

**Resultado esperado:**
- ✅ La tabla muestra las columnas: **Nombre del archivo · Fecha de creación · Usuario · Tarea de origen**.
- ✅ Los documentos adjuntados en Gestión OBYC, Presupuesto, Kick Off, Gestión Cesta o Tarea Derivada aparecen consolidados aquí.
- ✅ La tabla tiene paginación funcional.

---

#### Caso 16-02 — Subir un documento desde el panel de tareas

**Precondición:** Sesión activa con rol **PM**, **COMPRADOR** o **CONTROL_GESTION** (rol con permiso de escritura). Tiene una tarea abierta.

**Pasos:**
1. Abrir cualquier tarea (p.ej. Gestión OBYC de la oportunidad de prueba).
2. Localizar el panel **"Documentos Adjuntos"** dentro de la tarea.
3. Hacer clic en **"Adjuntar archivo"** y seleccionar un archivo (p.ej. `contrato-prueba.pdf`).
4. Confirmar la carga.

**Resultado esperado:**
- ✅ El archivo aparece en el panel de documentos de la tarea.
- ✅ Al navegar al Repositorio de la oportunidad (`/repositorio/{oppId}`), el archivo también aparece allí con la tarea de origen indicada.
- ✅ El archivo se puede descargar desde ambas vistas.

---

#### Caso 16-03 — Eliminar un documento (rol escritura)

**Precondición:** Existe al menos un documento en el repositorio. Sesión activa con rol que tenga permisos de escritura.

**Pasos:**
1. En el panel de documentos de la tarea (o en `/repositorio/{oppId}`), localizar el archivo subido en el caso anterior.
2. Hacer clic en el icono de **eliminar** (papelera).
3. Confirmar la acción en el diálogo.

**Resultado esperado:**
- ✅ El archivo desaparece de la lista.
- ✅ En la tabla de auditoría, aparece un registro con: **Acción = "Elimina" · Documento = nombre del archivo · Usuario · Fecha y hora**.

---

#### Caso 16-04 — Restricción de escritura para rol COMPRADOR

**Precondición:** Sesión activa con rol **COMPRADOR**.

**Pasos:**
1. Navegar a `/repositorio/{oppId}`.
2. Intentar adjuntar un archivo o eliminar uno existente.

**Resultado esperado:**
- ✅ Los botones de **adjuntar** y **eliminar** no están disponibles (o muestran mensaje de permiso denegado).
- ✅ La descarga de archivos sigue funcionando (solo lectura).

---

#### Caso 16-05 — Exportar auditoría en CSV

**Precondición:** El repositorio tiene al menos 2 registros de auditoría (subidas o eliminaciones).

**Pasos:**
1. En la vista de Repositorio (`/repositorio/{oppId}`), localizar la sección de **auditoría**.
2. Hacer clic en **"Exportar CSV"**.

**Resultado esperado:**
- ✅ Se descarga un archivo `.csv` con las columnas: **Fecha y hora · Usuario · Acción · Documento · Oportunidad**.
- ✅ El número de filas en el CSV coincide con el número de eventos registrados en pantalla.
- ✅ El CSV se puede abrir en Excel sin errores de codificación (UTF-8).

---

### HU-20 — Workflow de Etapas

**Descripción:** Cada oportunidad tiene un **tablero de workflow** visual con las etapas del proceso de compra. Se muestran en colores según el estado: ⚪ Gris (pendiente) · 🟠 Naranja (asignada) · 🟢 Verde (completada). Además, el workflow aplica **alertas de color** por tiempo transcurrido respecto a la fecha compromiso del cliente.

#### Caso 20-01 — Visualizar tablero workflow de una oportunidad PUNTUAL

**Precondición:** Existe una oportunidad de tipo **PUNTUAL** con al menos la etapa F1 Ganado activa.

**Pasos:**
1. Navegar a **Workflow** (`/workflow/{oppId}`).

**Resultado esperado:**
- ✅ El tablero muestra las **9 etapas** en el orden: F1 Ganado → Gestión OBYC → Presupuesto → Kick Off Entrega → Crear Cesta → Cesta Liberada → Adjudicación → Formalización → Pago a Proveedores.
- ✅ La etapa **F1 Ganado** aparece en 🟢 **Verde** (completada al importar el Excel).
- ✅ Las etapas no iniciadas aparecen en ⚪ **Gris**.

---

#### Caso 20-02 — Visualizar tablero workflow de una oportunidad GENERAL

**Precondición:** Existe una oportunidad de tipo **GENERAL**.

**Pasos:**
1. Navegar a **Workflow** (`/workflow/{oppId}`) de la oportunidad GENERAL.

**Resultado esperado:**
- ✅ El tablero muestra las **6 etapas**: F1 Ganado → Gestión OBYC → Presupuesto → Kick Off Entrega → Tarea Derivada → Pago a Proveedores.
- ✅ **No aparecen** las etapas: Crear Cesta, Cesta Liberada, Adjudicación, Formalización.

---

#### Caso 20-03 — Verificar alertas de color por tiempo

**Precondición:** Existe una oportunidad con fecha compromiso en el pasado o muy próxima, con etapa **Gestión OBYC** aún no completada.

**Verificación directa:**
1. Revisar la etapa **Gestión OBYC** en el tablero.

**Resultado esperado (según tabla paramétrica de HU-20):**
- ✅ Si el % tiempo transcurrido es **< 3%** → etapa sin alert (color normal).
- ✅ Si el % tiempo transcurrido es **≥ 3% y < 5%** → borde/color 🟡 **Amarillo**.
- ✅ Si el % tiempo transcurrido es **≥ 5%** → borde/color 🔴 **Rojo**.
- ✅ Una etapa ya completada (verde) **no muestra alerta** de tiempo.

> **Nota:** Para forzar alertas en entorno de prueba, se puede modificar la `fechaCompromisoCliente` de la oportunidad a una fecha pasada mediante la API o la BD directamente.

---

#### Caso 20-04 — Descargar reporte de logs en CSV

**Precondición:** El workflow de la oportunidad tiene al menos 2 cambios de etapa registrados.

**Pasos:**
1. En la vista del workflow, localizar el botón **"Reporte de Logs"** o ir a la sección de reportes.
2. Seleccionar un rango de fechas que incluya las fechas de los cambios realizados.
3. Hacer clic en descargar.

**Resultado esperado:**
- ✅ Se descarga un archivo CSV con columnas: **Usuario · Fecha · Hora · Oportunidad · Proveedor · Orden de Compra · Etapa anterior · Etapa nueva**.
- ✅ El CSV refleja correctamente todos los cambios de etapa del rango seleccionado.

---

### HU-21 — Vista General Estado Oportunidades (Vista PM)

**Descripción:** El Project Manager tiene una vista consolidada **"Monitorea el estado de tus Oportunidades"** con contadores de resumen y tabla de todas las oportunidades asignadas, agrupadas por oportunidad y RFX/compra.

#### Caso 21-01 — Visualizar la vista PM

**Precondición:** Sesión activa con rol **PM**. Existen al menos 2 oportunidades asignadas al PM con distintas etapas.

**Pasos:**
1. Navegar a **Workflow** (`/workflow`) o la ruta de la vista PM.

**Resultado esperado:**
- ✅ Aparece el título **"Monitorea el estado de tus Oportunidades"**.
- ✅ En la parte superior se muestran **4 tarjetas de conteo**:
  - **Total de Oportunidades Asignadas**
  - **Total en Gestión Inicial** (etapas: OBYC, Presupuesto, Kick Off)
  - **Total en Ejecución** (etapas: Crear Cesta, Cesta Liberada, Adjudicación, Formalización / Tarea Derivada)
  - **Total en Gestión Pago** (etapa: Pago a Proveedores)
- ✅ Los conteos son **únicos por oportunidad** (no duplica si hay múltiples compras).

---

#### Caso 21-02 — Tabla agrupada por oportunidad y compra

**Pasos:**
1. En la vista PM, observar la tabla principal.

**Resultado esperado:**
- ✅ Cada fila representa una combinación **Oportunidad + RFX/Compra**.
- ✅ Si una oportunidad tiene 2 compras (2 RFX), aparecen **2 filas** bajo la misma oportunidad.
- ✅ La columna **Etapa actual** refleja el estado correcto de cada compra.
- ✅ Hay un enlace o botón para navegar al tablero de workflow de cada oportunidad.

---

#### Caso 21-03 — Importar Excel F1 desde vista PM

**Pasos:**
1. En la vista PM, buscar el botón **"Importar Excel F1"**.
2. Seleccionar el archivo de demo disponible en `scripts/sample-data/`.
3. Confirmar la importación.

**Resultado esperado:**
- ✅ El sistema procesa el Excel y muestra un resumen de registros importados.
- ✅ Las nuevas oportunidades aparecen en la tabla de la vista PM.
- ✅ Los contadores de la parte superior se actualizan.

---

### HU-22 — Cambios de Etapas del Workflow

**Descripción:** Las etapas del workflow cambian de color automáticamente al asignar o cerrar las tareas correspondientes. Las etapas manuales (Adjudicación, Formalización, Pago a Proveedores para compra puntual) se avanzan explícitamente por el usuario.

#### Caso 22-01 — Etapas automáticas: OBYC → Presupuesto → Kick Off

**Precondición:** Oportunidad PUNTUAL con tareas abiertas. Workflow en etapa inicial.

**Pasos:**
1. Abrir el tablero workflow de la oportunidad (`/workflow/{oppId}`).
2. En otra pestaña, ir a **Compras → Gestión OBYC**, abrir y cerrar la tarea OBYC.
3. Volver al tablero y recargar.

**Resultado esperado:**
- ✅ La etapa **Gestión OBYC** cambia a 🟢 **Verde** al cerrar la tarea.
- ✅ La etapa **Presupuesto** pasa a 🟠 **Naranja** (indicando que la tarea está asignada).
- ✅ Las etapas previas mantienen su color.

---

#### Caso 22-02 — Etapas automáticas: Crear Cesta + Cesta Liberada

**Precondición:** Oportunidad PUNTUAL con OBYC y Presupuesto ya cerradas (Gestión Cesta creada automáticamente).

**Pasos:**
1. Abrir el tablero workflow de la oportunidad.
2. Ir a **Compras → Gestión Cesta**, abrir la tarea y cerrarla (con Número de Cesta válido).
3. Volver al tablero y recargar.

**Resultado esperado:**
- ✅ La etapa **Crear Cesta** cambia a 🟢 **Verde**.
- ✅ La etapa **Cesta Liberada** también cambia a 🟢 **Verde** simultáneamente (ambas al cerrar Gestión Cesta).

---

#### Caso 22-03 — Avanzar etapa manual: Adjudicación

**Precondición:** El workflow está en etapa **Cesta Liberada** (verde). Sesión activa con rol **PM** o **ADMIN**.

**Pasos:**
1. En el tablero workflow, localizar la etapa **Adjudicación** (en gris).
2. Hacer clic en el botón **"Avanzar etapa"** o el control de la burbuja de Adjudicación.
3. Si el sistema lo solicita, ingresar un comentario opcional.
4. Confirmar la acción.

**Resultado esperado:**
- ✅ La etapa **Adjudicación** cambia a 🟠 **Naranja** (asignada).
- ✅ El cambio queda registrado en el log con: usuario · fecha · hora · etapa anterior (Cesta Liberada) · etapa nueva (Adjudicación).
- ✅ Volver a hacer clic/avanzar cambia **Adjudicación** a 🟢 **Verde** y activa **Formalización** en naranja.

---

#### Caso 22-04 — Avanzar etapa manual: Formalización y Pago

**Precondición:** Adjudicación en verde.

**Pasos:**
1. Avanzar la etapa **Formalización** (manual) → debe pasar a verde.
2. Avanzar la etapa **Pago a Proveedores** (manual) → debe pasar a naranja y luego a verde.

**Resultado esperado:**
- ✅ Formalización → 🟢 Verde.
- ✅ Pago a Proveedores → 🟠 Naranja al asignar, 🟢 Verde al completar.
- ✅ Todas las etapas previas mantienen el color verde.
- ✅ Cada cambio queda registrado en el log.

---

#### Caso 22-05 — Flujo workflow compra GENERAL (6 etapas)

**Precondición:** Oportunidad GENERAL con Presupuesto cerrado (Tarea Derivada creada).

**Pasos:**
1. Verificar el tablero de la oportunidad GENERAL.
2. Cerrar la **Tarea Derivada** (con Número de Derivada).
3. Avanzar manualmente la etapa **Pago a Proveedores**.

**Resultado esperado:**
- ✅ Secuencia de colores: F1 Ganado ✅ → OBYC ✅ → Presupuesto ✅ → Kick Off ✅ → Tarea Derivada 🟢 → Pago 🟠/🟢.
- ✅ Las etapas de compra puntual (Crear Cesta, Cesta Liberada, Adjudicación, Formalización) **no aparecen** en el tablero.

---

## Sprint 5 — HU-24

---

### HU-24 — Generación de Notificaciones

**Descripción:** El sistema genera notificaciones automáticas por **campana in-app** y **correo electrónico** al PM, Vendor y Comprador según 11 condiciones definidas (cambios de etapa, alertas de color).

> **Prerequisito de notificaciones:** Para verificar notificaciones por correo en entorno local es necesario tener configurado el SMTP en `application-local.yml`. La campana in-app (badge) funciona sin SMTP.

---

#### Caso 24-01 — Campana in-app: badge visible al recibir notificación

**Precondición:** Sesión activa con rol **PM**. La oportunidad avanza a etapa **Adjudicación** (por otro usuario o manualmente).

**Pasos:**
1. Con la sesión del PM activa, verificar la campana de notificaciones en la barra de navegación.

**Resultado esperado:**
- ✅ La campana muestra un **badge numérico** (ej. `1`) en rojo indicando notificaciones no leídas.
- ✅ Al hacer clic en la campana, se despliega el panel de notificaciones con el mensaje: *"La oportunidad número xxx-xxxxxxxxx pasó a etapa Adjudicación"*.

---

#### Caso 24-02 — Marcar notificación como leída

**Pasos:**
1. Abrir el panel de la campana (con al menos una notificación no leída).
2. Hacer clic en el botón **"Marcar como leída"** en una notificación.

**Resultado esperado:**
- ✅ La notificación cambia a estado leído (sin resalte o indicador visual distinto).
- ✅ El número del badge se reduce en 1.
- ✅ Si era la única notificación, el badge desaparece.

---

#### Caso 24-03 — Marcar todas las notificaciones como leídas

**Precondición:** El PM tiene 3 o más notificaciones no leídas.

**Pasos:**
1. Abrir el panel de la campana.
2. Hacer clic en **"Marcar todas como leídas"**.

**Resultado esperado:**
- ✅ Todas las notificaciones pasan a estado leído.
- ✅ El badge de la campana **desaparece** (contador en 0).

---

#### Caso 24-04 — Notificación al PM al pasar a etapa Crear Cesta

**Precondición:** Oportunidad PUNTUAL con OBYC y Presupuesto a punto de cerrarse.

**Pasos:**
1. Iniciar sesión como **Control de Gestión** y cerrar la tarea Presupuesto de la oportunidad PUNTUAL (si OBYC ya estaba cerrada, esto activa la Gestión Cesta).
2. Cambiar de sesión al usuario **PM** asignado a la oportunidad.
3. Revisar la campana de notificaciones.

**Resultado esperado:**
- ✅ El PM recibe notificación con mensaje: *"La oportunidad número xxx-xxxxxxxxx pasó a etapa Crear Cesta"*.
- ✅ (Si SMTP configurado) El PM también recibe un correo electrónico con el mismo mensaje.

---

#### Caso 24-05 — Notificación al Vendor al asignarse Gestión Cesta

**Pasos:**
1. Iniciar sesión como **VENDOR** (`vendor.hgp` / `Admin2024`).
2. Revisar la campana de notificaciones tras la creación automática de la Gestión Cesta (del caso anterior).

**Resultado esperado:**
- ✅ El Vendor recibe notificación: *"Se asignó la tarea Gestión cesta a la oportunidad número xxx-xxxxxxxxx"*.

---

#### Caso 24-06 — Notificación al Vendor: alerta Amarilla/Roja en OBYC o Presupuesto

**Precondición:** Existe una oportunidad con fecha compromiso vencida y etapa OBYC o Presupuesto aún abierta (para forzar alerta roja).

**Verificación:**
1. Iniciar sesión como **VENDOR**.
2. Revisar la campana de notificaciones.

**Resultado esperado:**
- ✅ Si la etapa entró en alerta **Amarilla**: notificación con *"Pronto se vencerá el tiempo establecido para completar la Etapa {Nombre de etapa} asociada a la oportunidad número xxx-xxxxxxxxx"*.
- ✅ Si la etapa entró en alerta **Roja**: notificación con *"Se ha vencido el tiempo establecido para completar la Etapa {Nombre de etapa} asociada a la oportunidad número xxx-xxxxxxxxx"*.

---

#### Caso 24-07 — Notificación al Comprador: Cesta Liberada en verde

**Precondición:** Oportunidad PUNTUAL con etapa Cesta Liberada al completarse la Gestión Cesta.

**Pasos:**
1. Tras cerrar la **Gestión Cesta** (Caso 22-02), iniciar sesión como **COMPRADOR**.
2. Revisar el correo electrónico del Comprador (solo correo, no campana).

**Resultado esperado:**
- ✅ (Si SMTP configurado) El Comprador recibe un correo con: *"Se ha liberado la cesta de la oportunidad número xxx-xxxxxxxxx"*.
- ✅ La campana del Comprador **no muestra** esta notificación (solo correo, según definición de HU-24).

---

#### Caso 24-08 — Verificar endpoint REST de notificaciones

**Pasos:**
1. Con sesión activa (usuario PM), llamar directamente a la API:
   ```
   GET http://localhost:8081/api/notificaciones
   ```
2. Llamar al endpoint del badge:
   ```
   GET http://localhost:8081/api/notificaciones/badge
   ```

**Resultado esperado:**
- ✅ `GET /api/notificaciones` devuelve la lista de notificaciones del usuario autenticado (JSON array).
- ✅ `GET /api/notificaciones/badge` devuelve el número de notificaciones no leídas (número entero).
- ✅ Llamar a `PATCH /api/notificaciones/{id}/leida` marca la notificación como leída y el badge se reduce.

---

## Checklist de Aceptación — Fase 2

| # | Caso | HU | Módulo | Validación técnica (QCS) | Aprobación cliente (UAT) | Observaciones |
|---|------|----|--------|---------------------------|---------------------------|---------------|
| 16-01 | Visualizar documentos del repositorio | HU-16 | Repositorio | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 16-02 | Subir documento desde tarea | HU-16 | Repositorio | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 16-03 | Eliminar documento + auditoría | HU-16 | Repositorio | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 16-04 | COMPRADOR solo lectura | HU-16 | Repositorio | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 16-05 | Exportar auditoría CSV | HU-16 | Repositorio | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 20-01 | Tablero workflow puntual (9 etapas) | HU-20 | Workflow | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 20-02 | Tablero workflow general (6 etapas) | HU-20 | Workflow | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 20-03 | Alertas de color por tiempo | HU-20 | Workflow | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 20-04 | Reporte logs CSV | HU-20 | Workflow | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 21-01 | Vista PM — contadores | HU-21 | Vista PM | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 21-02 | Vista PM — tabla agrupada | HU-21 | Vista PM | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 21-03 | Vista PM — importar Excel | HU-21 | Vista PM | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 22-01 | Etapas automáticas OBYC/Presupuesto | HU-22 | Workflow | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 22-02 | Etapas automáticas Cesta Liberada | HU-22 | Workflow | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 22-03 | Etapa manual Adjudicación | HU-22 | Workflow | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 22-04 | Etapas manuales Formalización + Pago | HU-22 | Workflow | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 22-05 | Workflow completo compra GENERAL | HU-22 | Workflow | ✅ | ☐ | Validado en pruebas internas + auditoría técnica Fase 2 |
| 24-01 | Campana in-app: badge visible | HU-24 | Notif. | ✅ | ☐ | Evidencia API/campana en validación E2E 13/04 |
| 24-02 | Marcar notificación como leída | HU-24 | Notif. | ✅ | ☐ | Evidencia API/campana en validación E2E 13/04 |
| 24-03 | Marcar todas como leídas | HU-24 | Notif. | ✅ | ☐ | Evidencia API/campana en validación E2E 13/04 |
| 24-04 | Notificación PM → Crear Cesta | HU-24 | Notif. | ✅ | ☐ | Evidencia API/campana en validación E2E 13/04 |
| 24-05 | Notificación Vendor → asig. Cesta | HU-24 | Notif. | ✅ | ☐ | Evidencia API/campana en validación E2E 13/04 |
| 24-06 | Notificación Vendor → alerta color | HU-24 | Notif. | ✅ | ☐ | Evidencia API/campana en validación E2E 13/04 |
| 24-07 | Notificación Comprador → Cesta Lib. | HU-24 | Notif. | ✅ | ☐ | Evidencia API/campana en validación E2E 13/04 |
| 24-08 | API notificaciones funcional | HU-24 | Notif. | ✅ | ☐ | Evidencia API/campana en validación E2E 13/04 |

---

**Firma de aceptación:**

| Rol | Nombre | Fecha | Firma |
|-----|--------|-------|-------|
| PM Telefónica | | | |
| Responsable QCS | | | |

---

*Guía de Pruebas Fase 2 — Cronos HGP B2B — 13/04/2026*
