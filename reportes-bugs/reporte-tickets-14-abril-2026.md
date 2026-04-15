# Reporte de Tickets - 14 de Abril 2026

**Sistema:** Cronos HGP B2B  
**Fecha del reporte:** 2026-04-15  
**Total de tickets procesados:** 12

---

## Resumen General

| ID | Tipo | Estado | Usuario | Rol | Título | Hora |
|----|------|--------|---------|-----|--------|------|
| #20 | REPORTE_ERROR | CERRADO | admin2 | ADMIN | Campos faltantes en formato FOD | 11:26 |
| #22 | REPORTE_ERROR | CERRADO | admin2 | ADMIN | Autocompletar datos desde Excel | 11:35 |
| #23 | REPORTE_ERROR | CERRADO | admin2 | ADMIN | Error al guardar línea OPEX | 11:50 |
| #25 | REPORTE_ERROR | CERRADO | vendor | VENDOR | Campos faltantes en Tarea Derivada | 12:00 |
| #27 | REPORTE_ERROR | CERRADO | vendor | VENDOR | Error al cerrar tarea derivada | 12:23 |
| #28 | REPORTE_ERROR | CERRADO | Control_de_Gestion | CONTROL_GESTION | Mostrar asignación en cabecera | 12:39 |
| #29 | REPORTE_ERROR | CERRADO | Control_de_Gestion | CONTROL_GESTION | Seeding automático OPEX/CAPEX | 12:44 |
| #31 | REPORTE_ERROR | CERRADO | Control_de_Gestion | CONTROL_GESTION | Duplicidad en carga de documentos | 12:56 |
| #36 | REPORTE_ERROR | REVISADO | comprador | COMPRADOR | Prueba workflow compra general | 14:35 |
| #40 | REPORTE_ERROR | CERRADO | admin2 | ADMIN | Reporte de logs de cambio de etapa vacío | 15:00 |
| #42 | REPORTE_ERROR | CERRADO | pm | PM | RFX no visibles en vista general | 15:06 |
| #45 | REPORTE_ERROR | CERRADO | pm | PM | Etapa Cesta Liberada no se marca verde | 15:21 |

---

## Detalle por Ticket

### Ticket #20
**Estado:** CERRADO | **Usuario:** admin2 (ADMIN) | **Hora:** 11:26:28

 `/compras/fod/3127`

**Descripción:**
Al formato de FOD le faltan los campos según HU 8:
- Cotizó: Si / No
- Evaluación Técnica: Cumple / No Cumple
- Valor Oferta Inicial
- Valor Oferta Final
- Número de contrato (Si es derivado de compra general)
- ID del Midas Aprobado
- Fecha de FOD

**Solución aplicada:**
Fix aplicado: se agregó visibilidad del campo Fecha de FOD (campo 21 HU-08) y se modificó la tabla de proveedores para mostrar estructura visible incluso cuando está vacía con mensaje guiado. Verificado: ID Midas Aprobado, Fecha FOD, Cotizó, Evaluación Técnica, Valor Oferta Inicial/Final y N° Contrato ahora son visibles y funcionales según HU-08.

**Evidencia:**

---

### Ticket #22
**Estado:** CERRADO | **Usuario:** admin2 (ADMIN) | **Hora:** 11:35:13

 `/compras/fod/3127`

**Descripción:**
Debe traer (autocompletarse) datos de Excel según HU 8.

**Solución aplicada:**
Fix: nuevo endpoint GET `/api/fod/tarea/{tareaId}/prefill` que lee los campos Excel almacenados en OPP_OPPORTUNITY y los mapea a FodDTO. El frontend ahora llama a este endpoint cuando no existe FOD previo y pre-rellena el formulario. Verificado: campos Objeto, Ambito, Cliente/Producto, Proveedor Condicionado, Naturaleza Juridica, ID Midas e idMidasAprobado se autocompleutan correctamente desde los datos del Excel.

**Evidencia:**

---

### Ticket #23
**Estado:** CERRADO | **Usuario:** admin2 (ADMIN) | **Hora:** 11:50:49

 `/presupuesto/3115`

**Descripción:**
Error al guardar la línea opex agregada, no funciona el botón de guardar.

**Solución aplicada:**
Fix: `@JoinColumn(name='task_id')` corregido a `'tarea_id'` en PresupuestoOpex.java y PresupuestoCapex.java. La migración Flyway V5 creaba las tablas con columna tarea_id pero las entidades JPA usaban task_id, bloqueando todas las operaciones OPEX/CAPEX. Se mejoró también el error handler del frontend para mostrar el mensaje real del backend.

**Evidencia:**

---

### Ticket #25
**Estado:** CERRADO | **Usuario:** vendor (VENDOR) | **Hora:** 12:00:09

 `/compras/tarea-derivada/3130`

**Descripción:**
Faltan los campos y completarlos desde origen:
- Tipo Presupuesto (desde Tarea Kick Off de Entrega)
- Número de contrato (desde Tarea Kick Off de Entrega)
- Complemento (desde tarea gestión OBYC)
- Valor cesta (COP/USD) (desde Tarea Kick Off de Entrega)
- RFX (desde Tarea Kick Off de Entrega)
- Comprador (desde Excel)
- ID materiales (desde Excel cuando aplique)
- Margen de Rentabilidad (desde Tarea Kick Off de Entrega)
- Tipo de Compra = General (desde Tarea Kick Off Entrega)

**Solución aplicada:**
Se añadieron los 9 campos propagados faltantes en la Tarea Derivada: tipoPresupuesto, numeroContrato, complementoGa, valorCestaCop/Usd, rfx, comprador, margenRentabilidad, idMateriales, tipoCompra. Se corrigió también la omisión del campo idMateriales del TareaDTO (backend).

**Evidencia:**


---

### Ticket #27
**Estado:** CERRADO | **Usuario:** vendor (VENDOR) | **Hora:** 12:23:06

 `/compras/tarea-derivada/3130`

**Descripción:**
Se presentó error al cerrar derivada de oportunidad OPP-DEMO-2026-00099, Dato ingresado: DER-2025-0000002

**Solución aplicada:**
Fix aplicado: el método `cerrarTarea()` ahora persiste el campo numeroDerivada vía PATCH `/campos-compras` antes de llamar a POST `/cerrar`, resolviendo el error 500 que ocurría cuando el usuario cerraba sin pulsar Guardar previamente.

**Evidencia:**


---

### Ticket #28
**Estado:** CERRADO | **Usuario:** Control_de_Gestion (CONTROL_GESTION) | **Hora:** 12:39:25

 `/presupuesto/3119`

**Descripción:**
La Tarea en asignada debe decir que esta asignada a Control de Gestión.

**Solución aplicada:**
Se añadió la línea 'Asignado a: <nombre>' en la cabecera de la Tarea Presupuesto. El campo assignedToNombre ya venía en el DTO pero no se mostraba en el template HTML.

**Evidencia:**

---

### Ticket #29
**Estado:** CERRADO | **Usuario:** Control_de_Gestion (CONTROL_GESTION) | **Hora:** 12:44:50

 `/presupuesto/2725`

**Descripción:**
Para la oportunidad OPP-DEMO-2026-00003, tanto el opex como el capex debería estar diligenciado según Excel, de acuerdo a HU 17.

**Solución aplicada:**
Se implementó el seeding automático de líneas OPEX/CAPEX en la Tarea Presupuesto al procesar el Excel F1 Ganada. El motor de reglas ahora crea los registros en TK_PRESUPUESTO_OPEX y TK_PRESUPUESTO_CAPEX con los valores de las columnas N y O del Excel (idempotente: no sobreescribe si ya hay líneas manuales).

**Evidencia:**

---

### Ticket #31
**Estado:** CERRADO | **Usuario:** Control_de_Gestion (CONTROL_GESTION) | **Hora:** 12:56:45

 `/repositorio/OPP-GEN-2026-VAL-01`

**Descripción:**
Esta cargando dos veces el documento, es decir, crea dos registros de cargue. OPP-GEN-2026-VAL-01

**Solución aplicada:**
Fix aplicado: se añadió flag `isUploading` en DocumentosAdjuntosComponent para deshabilitar el botón 'Subir' mientras la petición HTTP está en vuelo, previniendo doble clic y la creación de registros duplicados.

**Evidencia:**

[Captura Ticket #31 - Duplicidad documentos](31.png)

---

### Ticket #36
**Estado:** REVISADO | **Usuario:** comprador (COMPRADOR) | **Hora:** 14:35:33

 `/workflow/COL-004068960`

**Descripción:**
Prueba exitosa del workflow para compra general de 6 etapas. COL-004068960.

**Notas:**
¿Esto es un soporte?

**Evidencia:**

---

### Ticket #40
**Estado:** CERRADO | **Usuario:** admin2 (ADMIN) | **Hora:** 15:00:55

 `/workflow/OPP-DEMO-2026-00098`

**Descripción:**
Se cerró tarea presupuesto de oportunidad OPP-DEMO-2026-00098 pero al descargar el reporte de logs de cambio de etapa no se muestra la información. La funcionalidad esta ok ya que descarga el Excel, pero el Excel viene vacío.

**Solución aplicada:**
Fix aplicado: el método `refrescarEtapasDesdeTareas()` en WorkflowService no registraba las transiciones automáticas de etapa en WF_ETAPA_LOG. Solo avanzarEtapaManual() escribía en esa tabla. Se añadió lógica para detectar cambios de estado en cada etapa y persistir un WorkflowEtapaLog por cada transición automática, usando el mapa prevEstadoPorEtapa ya existente. Ahora al cerrar una tarea (presupuesto, OBYC, etc.) el log de cambio de etapa se registra correctamente y el Excel del reporte muestra los datos.

**Evidencia:**

---

### Ticket #42
**Estado:** CERRADO | **Usuario:** pm (PM) | **Hora:** 15:06:08

 `/workflow`

**Descripción:**
En la vista general, para la oportunidad OPP-DEMO-2026-00002, no aparecen los dos RFX, pero sí aparecen al darle a Gestionar tarea.

**Solución aplicada:**
Fix aplicado: `expandirPorOrdenes()` en WorkflowService ahora expande por RFX de líneas PxQ de la tarea OBYC cuando la oportunidad no tiene OCs registradas. Para OPP-DEMO-2026-00002 el servicio detecta 2 RFX distintos (RFX-DEMO-2026-0003 y RFX-DEMO-2026-0004) en TK_PXQ_LINE y genera 2 filas en la vista general del PM, coincidiendo con lo visible al gestionar la tarea.

**Evidencia:**

---

### Ticket #45
**Estado:** CERRADO | **Usuario:** pm (PM) | **Hora:** 15:21:23

 `/workflow/OPP-FULL-2026-VAL-01`

**Descripción:**
Para oportunidad OPP-FULL-2026-VAL-01, se cierra tarea de cesta y se coloca en verde etapa de crear cesta pero no se coloca en verde etapa de cesta liberada.

**Solución aplicada:**
Fix aplicado: `mapearNumeroEtapaAValidationId()` no mapeaba la etapa 6 (Cesta Liberada) para compra puntual — el case 6 faltaba en el switch de TipoCompra.PUNTUAL, por lo que la etapa siempre quedaba en null y se forzaba a PENDIENTE/GRIS. Se añadió case 6 -> 5 para que Cesta Liberada consulte la misma tarea Gestión Cesta (validationId=5) que Crear Cesta, quedando ambas etapas en VERDE al cerrarse dicha tarea.

**Evidencia:**

---

## Estadísticas del Día

### Por Estado
- **CERRADO:** 11 tickets
- **REVISADO:** 1 ticket

### Por Rol
- **ADMIN:** 4 tickets (#20, #22, #23, #40)
- **CONTROL_GESTION:** 3 tickets (#28, #29, #31)
- **VENDOR:** 2 tickets (#25, #27)
- **PM:** 2 tickets (#42, #45)
- **COMPRADOR:** 1 ticket (#36)

### Por Tipo de Issue
- Campos faltantes/missing: 3 tickets (#20, #25, #28)
- Funcionalidad workflow: 4 tickets (#27, #36, #40, #45)
- Integración datos: 3 tickets (#22, #29, #42)
- Error técnico/backend: 2 tickets (#23, #31)

---

## Archivos Relacionados

### Reportes (.txt)
- reporte-20.txt, reporte-22.txt, reporte-23.txt, reporte-25.txt
- reporte-27.txt, reporte-28.txt, reporte-29.txt, reporte-31.txt
- reporte-36.txt, reporte-40.txt, reporte-42.txt, reporte-45.txt

### Capturas de Pantalla (.png)
- captura-20.png, captura-22.png, captura-23.png, captura-25.png
- captura-27.png, captura-28.png, captura-29.png, captura-31.png
- captura-36.png, captura-40.png, captura-42.png, captura-45.png

---

*Reporte generado automáticamente el 2026-04-15*
