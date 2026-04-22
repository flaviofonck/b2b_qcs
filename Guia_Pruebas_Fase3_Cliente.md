---
tags:
  - tipo/guia-pruebas
  - tipo/guia-uat
  - modulo/general
  - modulo/admin
  - modulo/bug-reports
  - fase/fase3
  - sprint/sprint6
  - sprint/sprint7
  - sprint/sprint8
  - rol/qa
  - rol/cliente
  - rol/pm
  - rol/admin
  - estado/actualizado
hu-relacionadas:
  - GAP-ADM-01
  - GAP-ADM-02
  - GAP-ADM-03
  - GAP-ADM-04
  - GAP-ADM-05
related-docs:
  - "[[Guia_Pruebas_Fase2_Cliente]]"
  - "[[Plan_Implementacion_Fase3]]"
  - "[[release-notes-fase3]]"
  - "[[fase3-gaps-implementation]]"
  - "[[tasks]]"
---

# Guía de Pruebas UAT — Fase 3
## Sistema Cronos HGP B2B · Sprints 6, 7 y 8

**Fecha:** 20/04/2026  
**Preparado por:** Equipo de Desarrollo QCS  
**Versión documento:** 1.2  

---

> **Acceso al sistema**
> - **Frontend:** http://localhost:4201
> - **Usuario de prueba ADMIN:** admin / Admin2024
> - **Usuario de prueba VENDOR:** vendor.hgp / Admin2024
> - **Backend API:** http://localhost:8081
> - **Prerequisito:** Backend corriendo (`iniciar-backend.bat`) y BD MySQL 8 activa (`docker compose up -d`).

> **Actualización 20/04/2026 (v1.2)**
> - ✅ **Sprint 6 completado:** Administración de usuarios + Bug Reports (5/5 gaps cerrados)
> - ✅ **Sprint 7 completado:** Órdenes de Compra múltiples + Operaciones despliegue (7/7 tareas cerradas)
> - ✅ Agregados casos UAT-OC-05 a UAT-OC-07 para **Panel gestión OC** en Vista PM
> - ✅ Validación completa de migraciones V26 (tabla OC) y V27 (índices performance)
> - ✅ Tests: OrdenCompraServiceTest (11/11 pasan)

---

## Módulo 1 — Administración de Usuarios (Sprint 6 · ADM-01)

### Caso UAT-ADM-01: Alta de usuario

**Precondición:** Sesión activa con rol **ADMIN** o **SUPERADMIN**.

**Pasos:**
1. Navegar a **Administración → Usuarios** (`/admin/usuarios`).
2. Pulsar **Nuevo usuario**.
3. Completar el formulario:
   - **Usuario:** `test.pm01`
   - **Nombre:** `PM de prueba`
   - **Email:** `pm01@empresa.local`
   - **Rol:** `PM`
   - **Contraseña:** `Test1234!`
4. Pulsar **Guardar**.

**Resultado esperado:**
- ✅ El usuario aparece en la tabla con estado **Activo**.
- ✅ Los campos se validaron antes de guardar (username de 3-100 caracteres, email con formato correcto).
- ✅ No es posible guardar sin los campos obligatorios (mensaje de error inline).

---

### Caso UAT-ADM-02: Edición de usuario

**Precondición:** Existe el usuario `test.pm01` creado en UAT-ADM-01.

**Pasos:**
1. En la tabla de usuarios, pulsar **Editar** para `test.pm01`.
2. Cambiar el **Nombre** a `PM de prueba (modificado)`.
3. Pulsar **Guardar**.

**Resultado esperado:**
- ✅ El cambio se refleja en la tabla inmediatamente.
- ✅ El campo **Usuario** (username) **no se puede cambiar** (es solo lectura en edición).

---

### Caso UAT-ADM-03: Restablecer contraseña

**Precondición:** Existe el usuario `test.pm01`.

**Pasos:**
1. Abrir el detalle de `test.pm01`.
2. Pulsar **Restablecer contraseña**.
3. Ingresar nueva contraseña: `NuevoClave2024!`.
4. Confirmar.

**Resultado esperado:**
- ✅ El sistema confirma que la contraseña fue actualizada.
- ✅ El usuario puede iniciar sesión con la nueva contraseña.

---

### Caso UAT-ADM-04: Desactivar usuario

**Precondición:** Existe el usuario `test.pm01`.

**Pasos:**
1. En la tabla, pulsar **Desactivar** para `test.pm01`.
2. Confirmar la acción en el diálogo de confirmación.

**Resultado esperado:**
- ✅ El usuario aparece con estado **Inactivo** en la tabla.
- ✅ Si se intenta iniciar sesión con `test.pm01`, el sistema rechaza el acceso.

---

### Caso UAT-ADM-05: Regla "último ADMIN activo"

**Precondición:** Existe solo **un** usuario activo con rol `ADMIN`.

**Pasos:**
1. Intentar desactivar ese único ADMIN activo.

**Resultado esperado:**
- ✅ El sistema **rechaza** la operación con el mensaje: *"No es posible desactivar al último administrador activo"* (o similar).
- ✅ El usuario sigue activo en la tabla.

---

### Caso UAT-ADM-06: Vista de perfiles / roles

**Precondición:** Sesión activa con rol **ADMIN** o **SOPORTE**.

**Pasos:**
1. Navegar a **Administración → Perfiles** (`/admin/perfiles`).

**Resultado esperado:**
- ✅ Se muestran todos los roles del sistema: SUPERADMIN, ADMIN, SOPORTE, PM, CONTROL_GESTION, CONTABILIDAD, PLANIFICACION, COMPRADOR, VENDOR.
- ✅ Para cada rol se muestra la descripción de permisos.
- ✅ La lista es **de solo lectura** (no hay botones de edición).

---

### Caso UAT-ADM-07: Crear Bug Report (usuario estándar)

**Precondición:** Sesión activa con **cualquier rol** (ej. PM, VENDOR, COMPRADOR).

**Pasos:**
1. Desde cualquier página del sistema, localizar el botón **"Reportar Bug"** (en header o menú).
2. Completar el formulario:
   - **Tipo:** Seleccionar `REPORTE_ERROR`
   - **Descripción:** *"El filtro de fecha en la vista PM no funciona correctamente. Al seleccionar un rango de fechas, la tabla no se actualiza."*
   - **Captura de pantalla:** Adjuntar imagen (formato PNG/JPG, opcional)
3. Pulsar **Enviar reporte**.

**Resultado esperado:**
- ✅ El sistema confirma: *"Reporte enviado correctamente. ID: #123"*.
- ✅ El estado del reporte se inicializa como `NUEVO`.
- ✅ El usuario puede ver el reporte en **Mis Reportes** (si hay vista de usuario).

---

### Caso UAT-ADM-08: Visualizar mis reportes (usuario estándar)

**Precondición:** El usuario creó al menos 1 bug report en UAT-ADM-07.

**Pasos:**
1. Navegar a **Mis Reportes** (o endpoint `GET /api/bug-reports/mis`).

**Resultado esperado:**
- ✅ Se muestra la lista de reportes creados por el usuario actual.
- ✅ Cada reporte muestra: **ID · Tipo · Página · Descripción corta · Estado · Fecha**.
- ✅ Los reportes de **otros usuarios no aparecen** (filtrado por usuario).

---

### Caso UAT-ADM-09: Acceder a administración de Bug Reports

**Precondición:** Sesión activa con rol **ADMIN**, **SOPORTE** o **SUPERADMIN**.

**Pasos:**
1. Navegar a **Administración → Bug Reports** (`/admin/bug-reports`).

**Resultado esperado:**
- ✅ Se muestra la vista de administración con:
  - **Filtros:** Estado (pills: TODOS, NUEVO, REVISADO, CERRADO), Tipo (dropdown), Rango de fechas, Búsqueda libre
  - **Lista:** Todos los reportes del sistema (sin filtro por usuario)
  - **Panel detalle:** Al seleccionar un reporte, se muestra el detalle completo a la derecha

---

### Caso UAT-ADM-10: Filtrar Bug Reports por estado y tipo

**Precondición:** Existen al menos 3 reportes con distintos estados (NUEVO, REVISADO, CERRADO).

**Pasos:**
1. En `/admin/bug-reports`, pulsar el filtro **NUEVO**.
2. Verificar que solo aparecen reportes en estado NUEVO.
3. Cambiar filtro a **Tipo: SOLICITUD_CAMBIO**.
4. Verificar que solo aparecen reportes de ese tipo.
5. Aplicar filtro de rango de fechas (últimos 7 días).

**Resultado esperado:**
- ✅ Cada filtro funciona independientemente.
- ✅ Los filtros son acumulativos (estado + tipo + fechas).
- ✅ El contador de resultados se actualiza dinámicamente.

---

### Caso UAT-ADM-11: Gestionar Bug Report (cambiar estado y responder)

**Precondición:** Sesión activa con rol **SOPORTE** o **SUPERADMIN**. Existe un reporte en estado `NUEVO`.

**Pasos:**
1. En `/admin/bug-reports`, seleccionar el reporte #123 (del caso UAT-ADM-07).
2. En el panel de gestión (derecha), completar:
   - **Estado:** Cambiar a `REVISADO`
   - **Respuesta al usuario:** *"Hemos identificado el problema. Será corregido en el próximo sprint."*
   - **Notas internas:** *"Bug confirmado. Relacionado con el componente DateRangeFilter. Ver TASK-XXX."*
3. Pulsar **Guardar cambios**.

**Resultado esperado:**
- ✅ El reporte cambia a estado `REVISADO`.
- ✅ El campo **Fecha de actualización** se actualiza.
- ✅ Si el usuario creador del reporte inicia sesión, puede ver la **respuesta al usuario** (las notas internas **no se muestran** al usuario).
- ✅ Solo usuarios con rol **SUPERADMIN** o **SOPORTE** pueden editar (otros roles ven solo lectura).

---

### Caso UAT-ADM-12: Exportar Bug Reports en ZIP

**Precondición:** Existen al menos 2 reportes con capturas de pantalla adjuntas.

**Pasos:**
1. En `/admin/bug-reports`, aplicar filtros opcionales (ej. estado NUEVO).
2. Pulsar **Exportar ZIP**.

**Resultado esperado:**
- ✅ Se descarga un archivo `bug-reports-YYYYMMDD-HHMMSS.zip`.
- ✅ El ZIP contiene:
  - **indice.csv:** Tabla con columnas `ID,Usuario,Tipo,Estado,Fecha,Página,Descripción`
  - **reporte-{id}.txt:** Archivos de texto con el detalle de cada reporte (metadata + descripción + respuesta usuario + notas internas)
  - **captura-{id}.png:** Imágenes de las capturas de pantalla (si existen)
- ✅ Los archivos .png son válidos y se pueden abrir en cualquier visor de imágenes.
- ✅ Si un reporte no tiene captura, no se incluye el archivo .png correspondiente.

---

### Caso UAT-ADM-13: Copiar Prompt para IA (análisis automático)

**Precondición:** Existe un reporte en `/admin/bug-reports`. Sesión activa con rol **SOPORTE** o **SUPERADMIN**.

**Pasos:**
1. Seleccionar un reporte en el panel de administración.
2. Pulsar **Copiar Prompt para IA** (o similar).

**Resultado esperado:**
- ✅ Se copia al portapapeles un texto con formato:
  ```
  # Bug Report #{id} — Sistema Cronos HGP B2B
  
  **Módulo detectado:** {módulo basado en URL}
  **Tipo:** {tipo}
  **Estado:** {estado}
  **Reportado por:** {usuario} ({rol})
  **Fecha:** {fecha}
  **Página:** {paginaTitulo} ({paginaUrl})
  
  ## Descripción del problema
  {descripcion}
  
  {Si hay captura: "## Captura de pantalla\n[Ver captura adjunta en el ZIP exportado]"}
  
  {Si hay notas internas: "## Contexto interno\n{notasInternas}"}
  
  ## Análisis solicitado
  Analiza este reporte y proporciona:
  1. Posible causa técnica
  2. Prioridad sugerida (Alta/Media/Baja)
  3. Pasos de reproducción estimados
  4. Componente/archivo afectado probable
  ```
- ✅ El prompt es útil para pegar en ChatGPT o GitHub Copilot y obtener análisis técnico.

---

## Módulo 2 — Órdenes de Compra por Oportunidad (Sprint 7 · EVL-01)

### Caso UAT-OC-01: Crear Orden de Compra en una oportunidad

**Precondición:** Existe una oportunidad cargada (p. ej. con número `OPP-DEMO-001`). Sesión activa con rol **PM** o **ADMIN**.

**Pasos:**
1. Navegar al **Tablero workflow** de la oportunidad `OPP-DEMO-001` (`/workflow/OPP-DEMO-001`).
2. Localizar la sección **Órdenes de Compra** o usar la API directamente:
   ```
   POST /api/oportunidades/OPP-DEMO-001/ordenes-compra
   Body: { "codigoOc": "OC-2026-001", "estado": "PENDIENTE" }
   ```
3. Verificar la respuesta.

**Resultado esperado:**
- ✅ La OC `OC-2026-001` queda registrada con estado `PENDIENTE`.
- ✅ El campo `codigoOc` fue saneado (trim aplicado).
- ✅ Al intentar crear una segunda OC con el mismo código para la misma oportunidad, el sistema devuelve un error `409` (o mensaje de duplicado).

---

### Caso UAT-OC-02: Listar Órdenes de Compra

**Precondición:** La oportunidad `OPP-DEMO-001` tiene al menos una OC registrada.

**Pasos:**
1. Llamar al endpoint:
   ```
   GET /api/oportunidades/OPP-DEMO-001/ordenes-compra
   ```

**Resultado esperado:**
- ✅ La respuesta incluye la OC `OC-2026-001` con sus campos `id`, `codigoOc`, `estado`, `fechaCreacion`.
- ✅ Se listan **en orden cronológico** (más antigua primero).

---

### Caso UAT-OC-03: Vista PM muestra OC por oportunidad

**Precondición:** La oportunidad `OPP-DEMO-001` tiene al menos una OC **no cerrada**.

**Pasos:**
1. Navegar a **Workflow** (home, `/workflow`).
2. Localizar la oportunidad `OPP-DEMO-001` en la tabla "Monitorea el estado de tus Oportunidades".

**Resultado esperado:**
- ✅ La fila de la oportunidad muestra la **Orden de Compra** asociada en la columna correspondiente.
- ✅ Si la oportunidad tiene **varias OCs no cerradas**, aparece una fila por cada OC.
- ✅ Las OCs en estado `CERRADA` **no generan fila** en la vista PM.

---

### Caso UAT-OC-04: Idempotencia código OC

**Pasos:**
1. Intentar crear dos veces la OC con código `  OC-2026-002  ` (con espacios) para la misma oportunidad.
2. Primera creación → esperar éxito.
3. Segunda creación con el mismo código (con o sin espacios) → esperar rechazo.

**Resultado esperado:**
- ✅ Primera llamada: OC creada con `codigoOc = "OC-2026-002"` (sin espacios).
- ✅ Segunda llamada: error informativo de duplicado.

---

### Caso UAT-OC-05: Panel gestión OC inline en Vista PM

**Precondición:** Sesión activa con rol **PM**. Existe al menos una oportunidad con 1+ OCs registradas.

**Pasos:**
1. Navegar a **Workflow → Estado de Oportunidades** (`/workflow/estado-oportunidades`).
2. Localizar la oportunidad `OPP-DEMO-001` en la tabla.
3. Pulsar el botón **"Gestionar OC"** (o ícono similar junto al nombre de la oportunidad).

**Resultado esperado:**
- ✅ Se expande un panel inline **debajo de la fila** de la oportunidad (no modal).
- ✅ El panel muestra:
  - **Tabla de OCs existentes** con columnas: Código OC, Descripción, Estado, Fecha Creación, Acciones
  - **Formulario "Nueva OC"** con campos: Código OC (input), Descripción (textarea opcional)
  - Botón **"Crear OC"**
- ✅ El panel es colapsable (se cierra al pulsar de nuevo "Gestionar OC" o al seleccionar otra oportunidad).

---

### Caso UAT-OC-06: Crear OC desde Vista PM (panel inline)

**Precondición:** Panel "Gestionar OC" abierto para la oportunidad `OPP-DEMO-001`.

**Pasos:**
1. En el formulario "Nueva OC" del panel inline:
   - **Código OC:** `OC-2026-PM-001`
   - **Descripción:** `Orden creada desde Vista PM`
2. Pulsar **Crear OC**.

**Resultado esperado:**
- ✅ La nueva OC aparece **inmediatamente en la tabla del panel** sin recargar la página.
- ✅ El formulario se resetea tras la creación.
- ✅ Se muestra mensaje de éxito: *"Orden de compra creada correctamente"* (snackbar o similar).
- ✅ Si se cierra y reabre el panel, la OC sigue visible en la lista.

---

### Caso UAT-OC-07: Validar restricciones en creación de OC desde panel

**Precondición:** Panel "Gestionar OC" abierto para una oportunidad.

**Pasos:**
1. Intentar crear OC **sin código** (dejar campo vacío).
2. Intentar crear OC con código **duplicado** (que ya existe en esa oportunidad).
3. Intentar crear OC con código que contiene **caracteres especiales** (ej. `OC#@123`).

**Resultado esperado:**
- ✅ **Campo vacío:** El botón "Crear OC" está deshabilitado o se muestra error de validación.
- ✅ **Código duplicado:** Error 409 con mensaje: *"Ya existe una OC con ese código para esta oportunidad"*.
- ✅ **Caracteres especiales:** Si hay validación de formato, se rechaza. Si no, se almacena tal cual (verificar con equipo).

---

## Módulo 3 — Flujo E2E completo (Regresión Fase 2 + Fase 3)

### Caso UAT-E2E-01: Flujo puntual completo con OC

**Pasos:**
1. Importar Excel F1 Ganada (archivo de demo disponible en `scripts/sample-data/`).
2. Verificar que se crearon automáticamente las tareas: **Gestión OBYC**, **Kick Off Entrega**, **Presupuesto**.
3. Cerrar la tarea **Gestión OBYC** (usuario Contabilidad).
4. Cerrar la tarea **Presupuesto** (usuario Control de Gestión).
5. Verificar que la etapa **Crear Cesta** se activa en el workflow.
6. Crear una OC para la oportunidad importada (`POST /api/oportunidades/{oppId}/ordenes-compra`).
7. Cerrar la tarea **Gestión Cesta** indicando el número de cesta.
8. Verificar el estado del workflow: etapa **Cesta Liberada** en verde.
9. Verificar que el PM recibió notificación (campana y/o correo).

**Resultado esperado:**
- ✅ Cada etapa del workflow avanza al cerrar la tarea correspondiente (colores: GRIS → NARANJA → VERDE).
- ✅ La OC aparece en la vista PM asociada a la oportunidad.
- ✅ Las notificaciones se generaron para los roles correspondientes.
- ✅ En el repositorio de documentos (`/repositorio`) figuran los adjuntos subidos durante las tareas.

---

### Caso UAT-E2E-02: Flujo general completo (6 etapas)

**Pasos:**
1. Importar Excel F1 Ganada con una oportunidad de tipo **GENERAL**.
2. Verificar que se crearon: **Gestión OBYC**, **Kick Off Entrega**, **Presupuesto**.
3. Cerrar **OBYC** y **Presupuesto**.
4. Verificar que se activó la **Tarea Derivada**.
5. Cerrar **Tarea Derivada** con número de derivada.
6. Verificar que el workflow de 6 etapas avanzó hasta **Pago a Proveedores**.

**Resultado esperado:**
- ✅ El workflow muestra la secuencia: F1 Ganado → Gestión OBYC → Presupuesto → Kick Off Entrega → Tarea Derivada → Pago a Proveedores.
- ✅ No se crean tareas de **Gestión Cesta** para compra general.

---

## Módulo 4 — Alertas de tiempo y notificaciones (HU-24 · Regresión)

### Caso UAT-NOT-01: Alerta rojo/amarillo en etapa OBYC

**Precondición:** Existe una oportunidad cuya fecha de compromiso está próxima o vencida, con etapa **Gestión OBYC** abierta.

**Verificación:**
- ✅ La etapa muestra color **AMARILLO** (≥ 80% del tiempo trascurrido) o **ROJO** (vencida).
- ✅ El Vendor tiene una notificación de alerta en la campana.

---

### Caso UAT-NOT-02: Notificación por cambio de etapa

**Precondición:** La etapa actual es **Presupuesto** y se cierra la tarea Presupuesto.

**Verificación:**
- ✅ La etapa avanza a **Kick Off Entrega** (o siguiente según tipo de compra).
- ✅ El PM recibe notificación en la campana indicando el cambio de etapa.

---

## Módulo 5 — Validación de Despliegue On-Premise (Sprint 7 · OPS-01)

> Estos casos se ejecutan en el **entorno real del cliente** (Windows Server o Linux on-premise), no en desarrollo local.

### Caso UAT-OPS-01: Arranque del backend en producción

**Precondición:** El servidor on-premise tiene JDK 21, MySQL 8 y las variables de entorno configuradas (`DB_USER`, `DB_PASSWORD`, `MAIL_HOST`, `MAIL_PORT`, `MAIL_USER`, `MAIL_PASSWORD`, `JWT_SECRET`, `server.port=8081`).

**Pasos:**
1. En el servidor, ejecutar `iniciar-backend.bat` (o el equivalente Linux con `./mvnw spring-boot:run`).
2. Esperar a que el log muestre `Started HgpB2bApplication`.

**Resultado esperado:**
- ✅ El backend arranca sin errores en el log.
- ✅ Las migraciones Flyway (V1–V27) se aplican correctamente (log: `Successfully applied N migrations`).
- ✅ El endpoint de health responde: `GET http://{servidor}:8081/actuator/health` → `{"status":"UP"}`.

---

### Caso UAT-OPS-02: Arranque del frontend en producción

**Precondición:** Node.js instalado. El `proxy.config.json` del frontend apunta a la URL del backend en producción.

**Pasos:**
1. Desde `hgp-b2b-frontend/`, ejecutar `npm start` (o servir el build con `npm run build`).
2. Abrir el navegador en `http://{servidor}:4201`.

**Resultado esperado:**
- ✅ La pantalla de login del sistema Cronos HGP B2B es visible.
- ✅ Las llamadas al API no generan errores de CORS en la consola del navegador.

---

### Caso UAT-OPS-03: Seed de usuarios iniciales de producción

**Precondición:** La BD está vacía (primera puesta en marcha) o se acaba de reimportar con la migración V25.

**Pasos:**
1. Verificar que el usuario `admin` / `Admin2024` puede iniciar sesión.
2. Verificar que existe al menos un usuario con rol `SUPERADMIN` en la tabla `USR_USER`.

**Resultado esperado:**
- ✅ Login con `admin` / `Admin2024` exitoso.
- ✅ La pantalla de Administración (`/admin/usuarios`) muestra el usuario admin con rol ADMIN o SUPERADMIN.

---

### Caso UAT-OPS-04: Backup y restauración MySQL

**Pasos:**
1. Ejecutar el backup documentado en `docs/despliegue-on-premise.md`:
   ```
   mysqldump -u root -p hgp_b2b > backup_$(date +%Y%m%d).sql
   ```
2. Restaurar en una BD de prueba:
   ```
   mysql -u root -p hgp_b2b_restore < backup_YYYYMMDD.sql
   ```
3. Arrancar el backend apuntando a `hgp_b2b_restore` y verificar login.

**Resultado esperado:**
- ✅ El backup se genera sin errores.
- ✅ La restauración aplica correctamente sin errores de FK o charset.
- ✅ El sistema funciona normalmente con la BD restaurada.

---

### Caso UAT-OPS-05: Verificación migraciones V26 y V27 (Sprint 7)

**Precondición:** Base de datos limpia o con migraciones hasta V25.

**Pasos:**
1. Arrancar el backend (las migraciones Flyway se aplican automáticamente).
2. Revisar el log del backend buscando:
   ```
   Migrating schema `hgp_b2b` to version "26 - opp orden compra"
   Migrating schema `hgp_b2b` to version "27 - indices performance"
   ```
3. Conectar a MySQL y verificar:
   ```sql
   -- Verificar tabla OPP_ORDEN_COMPRA existe
   SHOW TABLES LIKE 'OPP_ORDEN_COMPRA';
   
   -- Verificar índices creados
   SHOW INDEX FROM TK_TASK WHERE Key_name = 'IDX_TK_TASK_OPP_ID';
   SHOW INDEX FROM WF_WORKFLOW_ETAPA WHERE Key_name = 'IDX_WF_ETAPA_WF_ID';
   SHOW INDEX FROM NT_NOTIFICATION WHERE Key_name = 'IDX_NT_NOTIF_USR_LEIDA';
   ```

**Resultado esperado:**
- ✅ V26 aplicada: tabla `OPP_ORDEN_COMPRA` existe con columnas `id`, `oportunidad_id`, `codigo_oc`, `descripcion`, `estado`, `fecha_creacion`, `fecha_cierre`.
- ✅ V27 aplicada: índices `IDX_TK_TASK_OPP_ID`, `IDX_WF_ETAPA_WF_ID`, `IDX_NT_NOTIF_USR_LEIDA` existen.
- ✅ Constraint única `uq_oc_por_oportunidad` presente en `OPP_ORDEN_COMPRA`.
- ✅ No hay errores en el log de Flyway.

---

### Caso UAT-OPS-06: Validación script seed-produccion.ps1

**Precondición:** Servidor Windows con PowerShell. Base de datos MySQL accesible.

**Pasos:**
1. Ejecutar desde la raíz del proyecto:
   ```powershell
   .\scripts\seed-produccion.ps1 -Password "Admin2024" -DBPassword "admin"
   ```
2. Verificar la salida del script (debe mostrar confirmación de usuarios creados).
3. Conectar a la BD y consultar:
   ```sql
   SELECT username, email, rol, activo 
   FROM USR_USUARIO 
   WHERE username IN ('soporte.admin', 'admin.principal', 'admin.backup', 'pm.zona1', 'comprador.senior');
   ```
4. Intentar ejecutar el script **de nuevo** (debe ser idempotente).

**Resultado esperado:**
- ✅ Primera ejecución: crea 8 usuarios (1 SUPERADMIN, 2 ADMIN, 3 PM, 2 COMPRADOR).
- ✅ Segunda ejecución: **no falla** (actualiza los registros existentes sin duplicar).
- ✅ Todos los usuarios tienen `activo = 1` y `debe_restablecer_password = 1`.
- ✅ Las contraseñas están cifradas con BCrypt (no plaintext en BD).
- ✅ Login funcional con cualquiera de los usuarios creados usando contraseña temporal.

---

## Checklist de Aceptación UAT

| # | Caso | Módulo | ¿Aprobado? | Observaciones |
|---|------|--------|-----------|---------------|
| ADM-01 | Alta de usuario | Admin | ☐ | |
| ADM-02 | Edición de usuario | Admin | ☐ | |
| ADM-03 | Restablecer contraseña | Admin | ☐ | |
| ADM-04 | Desactivar usuario | Admin | ☐ | |
| ADM-05 | Regla último ADMIN | Admin | ☐ | |
| ADM-06 | Vista perfiles | Admin | ☐ | |
| ADM-07 | Crear bug report (usuario) | Bug Reports | ☐ | |
| ADM-08 | Ver mis reportes | Bug Reports | ☐ | |
| ADM-09 | Acceder admin bug reports | Bug Reports | ☐ | |
| ADM-10 | Filtrar bug reports | Bug Reports | ☐ | |
| ADM-11 | Gestionar bug report | Bug Reports | ☐ | |
| ADM-12 | Exportar ZIP | Bug Reports | ☐ | |
| ADM-13 | Copiar prompt IA | Bug Reports | ☐ | |
| OC-01 | Crear OC | Evolutivo | ☐ | |
| OC-02 | Listar OCs | Evolutivo | ☐ | |
| OC-03 | Vista PM con OC | Evolutivo | ☐ | |
| OC-04 | Idempotencia OC | Evolutivo | ☐ | |
| OC-05 | Panel gestión OC inline | Evolutivo | ☐ | |
| OC-06 | Crear OC desde panel PM | Evolutivo | ☐ | |
| OC-07 | Validar restricciones OC | Evolutivo | ☐ | |
| E2E-01 | Flujo puntual + OC | Regresión | ☐ | |
| E2E-02 | Flujo general | Regresión | ☐ | |
| NOT-01 | Alerta color etapa | Notif. | ☐ | |
| NOT-02 | Notif. cambio etapa | Notif. | ☐ | |
| OPS-01 | Arranque backend producción | Ops | ☐ | |
| OPS-02 | Arranque frontend producción | Ops | ☐ | |
| OPS-03 | Seed usuarios iniciales | Ops | ☐ | |
| OPS-04 | Backup y restauración MySQL | Ops | ☐ | |
| OPS-05 | Verificar migraciones V26/V27 | Ops | ☐ | |
| OPS-06 | Validar seed-produccion.ps1 | Ops | ☐ | |

---

**Firma de aceptación:**

| Rol | Nombre | Fecha | Firma |
|-----|--------|-------|-------|
| PM Telefónica | | | |
| Responsable QCS | | | |

---

*Guía UAT Fase 3 — Cronos HGP B2B — Versión 1.2 — 20/04/2026*

---

## Resumen de Implementación Sprint 7

**Estado:** ✅ Completado (20/04/2026)

**Componentes entregados:**
- **EVL-01:** Modelo Órdenes de Compra múltiple (V26 + backend + frontend panel inline)
- **OPS-01:** Documentación despliegue extendida (HTTPS, backup, seed usuarios)
- **Tests:** 11/11 pasan en `OrdenCompraServiceTest` (0 fallos, 0 errores)
- **Migraciones:** V26 (tabla `OPP_ORDEN_COMPRA`) + V27 (índices performance)
- **Scripts:** `seed-produccion.ps1` idempotente y funcional

**Casos UAT añadidos:**
- UAT-OC-05 a UAT-OC-07: Panel gestión OC en Vista PM
- UAT-OPS-05 a UAT-OPS-06: Validación migraciones y script seed

**Total casos UAT Fase 3:** 30 (13 Admin + 7 OC + 2 E2E + 2 Notif. + 6 Ops)
