# Guía de Pruebas — Fase 1
## Sistema Cronos HGP B2B · Versión Desconectada v3
### Sprints 1, 2 y 3 — 8 Historias de Usuario

**Fecha:** 06/04/2026  
**Preparado por:** Equipo de Desarrollo QCS  
**Versión documento:** 1.0  

---

> **Acceso al sistema**  
> - **Frontend:** http://localhost:4201  
> - **Usuario de prueba:** admin / Admin2024 *(o el que corresponda al entorno)*  
> - **Backend API:** http://localhost:8081  
> - **Prerequisito:** El backend debe estar corriendo (`iniciar-backend.bat`) y la BD MySQL 8 activa (`docker compose up -d`).

---

## Sprint 1 — Setup técnico + HU-03 + HU-05

### HU-03 — Gestión OBYC (Tarea padre automática)

**Descripción:** Al cargar el archivo Excel F1 Ganada + Liberada, el sistema debe crear automáticamente la **Tarea Gestión OBYC** (una tarea padre) y tantas **tareas hijas** como proveedores tenga la oportunidad. La bandeja de entrada es del rol **Contabilidad**.

#### Pasos de prueba

1. Ingresar al sistema con usuario de rol **PM**, **ADMIN** o **CONTROL_GESTION**.
2. Ir a la sección **"Importar Excel F1 Ganada"** (menú lateral o módulo Compras).
3. Seleccionar un archivo Excel F1 Ganada que contenga al menos **una oportunidad** con estado "Ganada + Liberada" y que tenga proveedores en la columna correspondiente.
4. Hacer clic en **"Procesar"** o **"Importar"**.

#### Resultado esperado

- ✅ El sistema muestra un resumen con el número de filas procesadas y tareas creadas.
- ✅ En la bandeja **Contabilidad**, aparece la tarea **"Gestión OBYC"** para la oportunidad procesada.
- ✅ Si la oportunidad tiene varios proveedores (separados por `;` o `|`), aparecen **tareas hijas** (una por cada proveedor), vinculadas a la tarea padre.
- ✅ Si se vuelve a importar el mismo Excel, **no se duplican** las tareas (idempotencia).

#### Verificación adicional (segundo clic)
1. Importar **el mismo archivo Excel** por segunda vez.
2. Verificar que el número de tareas en Contabilidad **no aumentó** (idempotencia confirmada).

---

### HU-05 — Kick Off Entrega (Tarea automática)

**Descripción:** Al procesar el Excel F1 Ganada, también se crea la **Tarea Kick Off Entrega**. Su asignación varía según la complejidad de la oportunidad.

#### Pasos de prueba

1. Procesar el mismo Excel F1 Ganada (o uno nuevo).
2. Observar la tarea **"Kick Off Entrega"** creada para la oportunidad.

#### Resultado esperado

- ✅ Aparece la tarea **"Kick Off Entrega"** en el sistema.
- ✅ Si la complejidad de la oportunidad es **BAJA** → asignada al rol **PM**.
- ✅ Si la complejidad es **MEDIA, ALTA o Proyectos Especiales** → asignada al rol **PLANIFICACIÓN**.
- ✅ El campo **Proveedor** en la tarea corresponde al proveedor indicado en el Excel (columna AF).
- ✅ Re-importar el Excel **no crea una segunda** tarea Kick Off para la misma oportunidad.

---

## Sprint 2 — HU-01 + HU-04 + HU-09

### HU-01 — Campos de Compra en Gestión Cesta

**Descripción:** Las tareas de **Gestión Cesta** deben mostrar y permitir editar los campos especiales de compra: **Ámbito de Compra**, **Tipo de Compra**, **PEP**, **Número de Derivada/Cesta** y **Valor Total de Compra**.

#### Pasos de prueba

1. Ir al módulo **Gestión Cesta** (o buscar la tarea correspondiente via bandeja).
2. Abrir el detalle de una tarea de tipo **Gestión Cesta**.

#### Resultado esperado

- ✅ El formulario muestra los campos: **Ámbito de Compra**, **Tipo de Compra** (PUNTUAL/GENERAL), **PEP**, **Número de Derivada/Cesta**, **Valor Total de Compra**.
- ✅ Los campos vienen **pre-cargados** con los valores propagados desde la oportunidad y el Excel.
- ✅ Se pueden editar y guardar (botón "Actualizar" o similar).
- ✅ Los campos actualizados se reflejan en el DTO de la tarea (verificable vía API o recargando la página).

---

### HU-04 — Gestión OBYC: PxQ manual y tabla contable

**Descripción:** El formulario de **Gestión OBYC** debe permitir gestionar las líneas PxQ (Precio por Cantidad) de forma manual, con dos columnas editables por Contabilidad: **Complemento GA** y **Cuenta Contable**. Al ingresar la cuenta contable, el sistema auto-completa el **Área Funcional**.

#### Pasos de prueba

1. Abrir una tarea **Gestión OBYC** (en la bandeja de Contabilidad o buscándola por oportunidad).
2. Ir a la pestaña/sección **"PxQ"** o **"Contabilidad"**.
3. Editar una línea PxQ:
   - Ingresar un valor en **"Complemento GA"**.
   - Ingresar un valor válido en **"Cuenta Contable"** (debe estar en la paramétrica del sistema).
4. Guardar los cambios.

#### Resultado esperado

- ✅ Las columnas **Complemento GA** y **Cuenta Contable** son editables.
- ✅ Al guardar la **Cuenta Contable** válida, el campo **Área Funcional** se auto-completa con el valor correspondiente de la paramétrica.
- ✅ Si la cuenta contable no existe en la paramétrica, el sistema muestra un **mensaje de error** indicando que la cuenta no es válida.
- ✅ Los cambios persisten al recargar la página.

#### Prueba de cierre
Completar los datos PxQ y verificar que al intentar cerrar la tarea OBYC, el sistema lo permite solo si se cumplen las precondicoines necesarias (ver HU-09 más abajo para el cierre encadenado).

---

### HU-09 — Gestión de Cesta Automática

**Descripción:** La tarea **Gestión de Cesta** se crea **automáticamente** al cerrar las dos tareas previas: **Gestión OBYC** y **Presupuesto**, pero **solo para compras tipo PUNTUAL**.

> **Prerequisito de datos:** Tener una oportunidad de tipo **PUNTUAL** con sus tres tareas iniciales creadas (OBYC, Kick Off, Presupuesto). Se pueden usar las oportunidades `OPP-CESTA-2026-00101` a `OPP-CESTA-2026-00115` del Excel de demo o cualquier otra PUNTUAL importada.

#### Paso 1 — Cerrar la tarea Gestión OBYC (rol: Contabilidad)

1. Ir al menú **Compras** (http://localhost:4201/compras).
2. En el selector **"Tipo de Tarea"**, elegir **"Gestión OBYC"**.
3. Localizar la tarea de la oportunidad PUNTUAL deseada y hacer clic en **"Abrir tarea"**.
   - La URL será `/compras/gestion-obyc/{id}`.
4. *(Opcional)* Editar una línea PxQ: columna **Complemento GA** y/o **Cuenta Contable**.
5. Hacer clic en el botón **"Cerrar Tarea OBYC"** (parte inferior del formulario).
6. Confirmar la acción en el diálogo de confirmación.

#### Paso 2 — Cerrar la tarea Presupuesto (rol: Control de Gestión)

1. Volver a **Presupuesto** (http://localhost:4201/presupuesto) **o** a **Compras** y filtrar por **"Presupuesto"**.
2. Localizar la tarea Presupuesto de la **misma oportunidad** y hacer clic en **"Abrir tarea"**.
   - La URL será `/presupuesto/{id}`.
3. *(Opcional)* Añadir al menos una línea OPEX o CAPEX con el botón **"Añadir OPEX"** / **"Añadir CAPEX"** y guardarla.
4. Hacer clic en el botón **"Cerrar tarea Presupuesto"** (parte inferior).
5. En este momento el backend crea automáticamente la **Tarea Gestión Cesta**.

#### Resultado esperado

- ✅ Al cerrar la tarea Presupuesto, el sistema **crea automáticamente** la tarea **"Gestión Cesta"**.
- ✅ La tarea Gestión Cesta aparece en **Compras** (filtro "Gestión Cesta") asignada al rol **VENDOR**.
- ✅ Los campos de la Gestión Cesta vienen **pre-cargados** con los datos de la oportunidad, el Kick Off y el OBYC (~20 campos: PEP, Proveedor, Valor Cesta COP/USD, RFX, Complemento GA, Cuenta Contable, Área Funcional, etc.).
- ✅ Si OBYC ya estaba cerrada y se cierra Presupuesto (o viceversa), la Gestión Cesta se crea al cerrarse la **segunda** de las dos.
- ✅ Si la oportunidad es de tipo **GENERAL**, la tarea Gestión Cesta **NO se crea** (este flujo es exclusivo de PUNTUAL).

#### Paso 3 — Verificar y cerrar Gestión Cesta (rol: Vendor)

1. Ir a **Compras** y filtrar por **"Gestión Cesta"**.
2. Abrir la tarea recién creada: clic en **"Abrir tarea"** (URL `/compras/gestion-cesta/{id}`).
3. Verificar que los campos vienen pre-cargados (Ámbito de Compra, Tipo de Compra, PEP, Valor Total de Compra).
4. Intentar cerrar la tarea **sin ingresar** el campo **"Número de Cesta"** → el sistema debe bloquear el cierre con un mensaje de error.
5. Ingresar el **Número de Cesta** (ej. `CESTA-TEST-001`) y hacer clic en **"Cerrar Tarea"**.

#### Resultado esperado

- ✅ Sin Número de Cesta → error **"El campo 'Número de cesta' es obligatorio para cerrar..."**
- ✅ Con Número de Cesta → cierre exitoso, estado cambia a **CERRADA**.

---

## Sprint 3 — HU-08 + HU-10 + HU-17

### HU-08 — Formulario FOD

**Descripción:** Cada **Tarea Kick Off Entrega** tiene asociado un **Formulario FOD** (Formulario de Orden de Despacho) con 21 campos. El FOD es un documento formal que debe completarse y puede imprimirse como PDF.

#### Pasos de prueba

1. Abrir una **Tarea Kick Off Entrega** (creada desde el Excel en Sprint 1).
2. Ir a la sección/pestaña **"FOD"** o hacer clic en el botón **"Ver FOD"** / **"Completar FOD"**.
3. Completar los campos principales:
   - **Objeto** del contrato.
   - **Ámbito** de la compra.
   - **Cliente / Producto**.
   - **Fecha de Inicio**, **Fecha de Entrega**, **Fecha de Terminación**.
   - **Proveedor Condicionado** (Sí/No).
   - **Tipo de Presupuesto** (OPEX/CAPEX).
   - **ID Midas Aprobado**.
   - **Proveedor Escogido** y **Justificación**.
   - Tabla de proveedores (proveedor, cotizó, evaluación técnica, valores oferta, número contrato).
4. Guardar el FOD.

#### Resultado esperado

- ✅ El FOD se guarda correctamente con todos los campos completados.
- ✅ Al abrir nuevamente la tarea, el FOD aparece **pre-cargado** con los datos guardados.
- ✅ El flag **"FOD Completado"** en la tarea queda en **true** (visible en el detalle de la tarea).

#### Prueba de validación de fechas

1. Ingresar una **Fecha de Entrega anterior a la Fecha de Inicio**.
2. El sistema debe mostrar error: **"Fecha de Inicio debe ser anterior a Fecha de Entrega"**.
3. Ingresar una **Fecha de Terminación anterior a la Fecha de Entrega**.
4. El sistema debe mostrar error: **"Fecha de Entrega debe ser anterior a Fecha de Terminación"**.

#### Resultado esperado

- ✅ Validaciones de fecha en client-side y server-side activas.
- ✅ No se puede guardar con fechas inválidas.

#### Prueba de impresión / exportar PDF

1. Con el FOD guardado, hacer clic en el botón **"Imprimir FOD"** o **"Exportar PDF"**.
2. El navegador debe abrir el **diálogo de impresión** del sistema.

#### Resultado esperado

- ✅ Se abre el diálogo de impresión del navegador con el contenido del FOD.
- ✅ Se puede guardar como PDF desde el diálogo del navegador.

#### Prueba de idempotencia
1. Intentar crear un segundo FOD para la misma tarea Kick Off.
2. El sistema debe mostrar un error: **"Ya existe un FOD para la tarea..."**

---

### HU-10 — Tarea Derivada (Compra General)

**Descripción:** Para las oportunidades de tipo **GENERAL**, al cerrar la **Tarea Presupuesto**, el sistema crea automáticamente la **Tarea Derivada** con los 11 campos propagados de las tareas anteriores.

> **Prerequisito de datos:** Tener una oportunidad de tipo **GENERAL** con sus tareas iniciales creadas. En el Excel de demo las oportunidades de índice **impar** (`OPP-DEMO-2026-00001`, `00003`, `00005`…) son de tipo GENERAL.

#### Pasos de prueba

1. Ir a **Presupuesto** (http://localhost:4201/presupuesto) o a **Compras** → filtro **"Presupuesto"**.
2. Localizar la tarea Presupuesto de una oportunidad **GENERAL** y hacer clic en **"Abrir tarea"** (URL `/presupuesto/{id}`).
3. *(Opcional)* Añadir líneas OPEX/CAPEX y guardarlas.
4. Hacer clic en **"Cerrar tarea Presupuesto"**.
5. El backend crea automáticamente la **Tarea Derivada**.

#### Resultado esperado

- ✅ Al cerrar el Presupuesto, el sistema crea automáticamente la **Tarea Derivada**.
- ✅ La tarea Derivada queda **asignada al rol VENDOR**.
- ✅ Los 11 campos pre-cargados correctamente:
  - Tipo Presupuesto (del Kick Off)
  - Número de Contrato (del Kick Off)
  - Proveedor (del Kick Off)
  - PEP (del Presupuesto)
  - Complemento GA (del OBYC)
  - Valor Cesta COP/USD (del Kick Off)
  - RFX (del Kick Off)
  - Tipo de Compra = GENERAL
  - Margen de Rentabilidad (del Kick Off)
  - ID Materiales (del Kick Off)
  - Comprador (del Excel)
- ✅ Si la oportunidad es **PUNTUAL**, la Tarea Derivada **NO se crea** (flujo exclusivo de GENERAL).

#### Prueba de cierre de Tarea Derivada

1. Ir a **Compras** y filtrar por **"Tarea Derivada"**.
2. Abrir la **Tarea Derivada** recién creada (URL `/compras/tarea-derivada/{id}`).
3. Verificar que los 11 campos están pre-cargados (Tipo Presupuesto, Proveedor, PEP, Complemento GA, Valor Cesta COP/USD, etc.).
4. Intentar cerrarla **sin ingresar** el **"Número de Derivada"** → el sistema debe bloquear el cierre.
5. Ingresar el Número de Derivada (ej. `DER-TEST-001`) y cerrar.

#### Resultado esperado

- ✅ Sin Número de Derivada → error **"El campo 'Número de derivada' es obligatorio para cerrar la Tarea Derivada"**.
- ✅ Con Número de Derivada → cierre exitoso.

---

### HU-17 — Tarea Presupuesto (Automática desde Excel)

**Descripción:** Al procesar el Excel F1 Ganada + Liberada, el sistema también crea automáticamente la **Tarea Presupuesto** con pestañas **OPEX** y **CAPEX**, asignada al rol **Control de Gestión**.

#### Pasos de prueba

1. Procesar el Excel F1 Ganada (mismo proceso que HU-03/05).
2. Buscar la tarea **"Presupuesto"** creada para la oportunidad (en la bandeja de Control de Gestión o buscando por oportunidad).

#### Resultado esperado

- ✅ Aparece la tarea **"Presupuesto"** asignada al rol **Control de Gestión**.
- ✅ El tipo de presupuesto (**OPEX** / **CAPEX**) se infiere automáticamente de los valores del Excel:
  - Solo OPEX > 0 → OPEX
  - Solo CAPEX > 0 → CAPEX
  - Ambos > 0 → el que tenga mayor valor

#### Prueba de líneas OPEX

1. Abrir la tarea Presupuesto.
2. Ir a la pestaña **"OPEX"**.
3. Agregar una línea con: PEP, Descripción, Presupuesto COP/Mes, Duración Meses.
4. Los campos **Total COP** y **Total USD** deben calcularse correctamente.
5. Guardar la línea.

#### Resultado esperado

- ✅ La línea OPEX se guarda y aparece en la tabla OPEX.
- ✅ Se pueden editar líneas existentes.

#### Prueba de líneas CAPEX

1. Ir a la pestaña **"CAPEX"**.
2. Agregar una línea con: Código PEP 1, Nombre PEP 2, Primera Solicitud COP/USD, Total COP/USD.
3. Guardar la línea.

#### Resultado esperado

- ✅ La línea CAPEX se guarda y aparece en la tabla CAPEX.

#### Prueba de cierre con trigger en cadena

1. Cerrar la tarea Presupuesto (sin campo requerido especial para esta tarea).
2. Si la oportunidad es PUNTUAL: verificar que se creó **Gestión Cesta** (ver HU-09).
3. Si la oportunidad es GENERAL: verificar que se creó **Tarea Derivada** (ver HU-10).

#### Resultado esperado

- ✅ Al cerrar Presupuesto, el workflow se actualiza en la UI.
- ✅ Se dispara la creación de la tarea encadenada correcta según el tipo de compra.

---

## Flujo End-to-End Completo (Compra Puntual)

Para validar toda la Fase 1 de forma integral:

```
1. [Admin/PM] Workflow → "Importar Excel" → seleccionar F1_Ganada_Demo_115.xlsx → "Procesar"
         ↓
2. [Sistema] Crea automáticamente para cada oportunidad PUNTUAL:
   - Tarea Gestión OBYC (padre) + hijos por proveedor   → bandeja Contabilidad
   - Tarea Kick Off Entrega                             → bandeja PM o Planificación
   - Tarea Presupuesto                                  → bandeja Control de Gestión
         ↓
3. [Contabilidad] Compras → filtro "Gestión OBYC" → "Abrir tarea"
   URL: /compras/gestion-obyc/{id}
   Acción: editar PxQ (Complemento GA + Cuenta Contable) → "Cerrar Tarea OBYC"
         ↓
4. [PM/Planificación] Compras → filtro "Kick Off Entrega" → "Abrir tarea"
   URL: /compras/kick-off-entrega/{id}
   Acción: completar FOD (21 campos + validaciones fecha) → guardar
         ↓
5. [Control de Gestión] Presupuesto → "Abrir tarea"
   URL: /presupuesto/{id}
   Acción: añadir líneas OPEX/CAPEX → "Cerrar tarea Presupuesto"
         ↓
6. [Sistema] Crea automáticamente: Tarea Gestión Cesta → asignada a VENDOR
         ↓
7. [Vendor] Compras → filtro "Gestión Cesta" → "Abrir tarea"
   URL: /compras/gestion-cesta/{id}
   Acción: verificar campos pre-cargados → ingresar Número de Cesta → "Cerrar Tarea"
         ↓
✅ Fase 1 completada para esta oportunidad
```

## Flujo End-to-End Completo (Compra General)

```
1. [Admin/PM] Workflow → "Importar Excel" → seleccionar F1_Ganada_Demo_115.xlsx → "Procesar"
         ↓
2. [Sistema] Crea: OBYC, Kick Off, Presupuesto  (oportunidades GENERAL = índices impares)
         ↓
3. [Control de Gestión] Presupuesto → "Abrir tarea"
   URL: /presupuesto/{id}
   Acción: añadir líneas → "Cerrar tarea Presupuesto"
         ↓
4. [Sistema] Crea automáticamente: Tarea Derivada → asignada a VENDOR
         ↓
5. [Vendor] Compras → filtro "Tarea Derivada" → "Abrir tarea"
   URL: /compras/tarea-derivada/{id}
   Acción: verificar 11 campos pre-cargados → ingresar Número de Derivada → "Cerrar Tarea"
         ↓
✅ Flujo General completado
```

---

## Checklist de Aceptación por HU

| HU | Prueba | ¿Pasa? |
|----|--------|--------|
| HU-03 | Tarea OBYC creada al importar Excel | ☐ |
| HU-03 | Tareas hijas por proveedor creadas | ☐ |
| HU-03 | Re-importar no duplica | ☐ |
| HU-05 | Tarea Kick Off creada | ☐ |
| HU-05 | Asignación correcta por complejidad | ☐ |
| HU-01 | Campos ambitoCompra/tipoCompra/pep/valorTotal visibles en Cesta | ☐ |
| HU-04 | Editar Complemento GA + Cuenta Contable en PxQ | ☐ |
| HU-04 | Área Funcional auto-completada al ingresar cuenta contable válida | ☐ |
| HU-09 | Gestión Cesta creada al cerrar OBYC + Presupuesto (PUNTUAL) | ☐ |
| HU-09 | Cerrar Cesta sin Número de Cesta → ERROR | ☐ |
| HU-09 | Cerrar Cesta con Número de Cesta → OK | ☐ |
| HU-08 | FOD con 21 campos visible en Kick Off | ☐ |
| HU-08 | Validación fechas (inicio < entrega < terminación) | ☐ |
| HU-08 | Botón imprimir abre diálogo de navegador | ☐ |
| HU-10 | Tarea Derivada creada al cerrar Presupuesto (GENERAL) | ☐ |
| HU-10 | Cerrar Derivada sin Número de Derivada → ERROR | ☐ |
| HU-17 | Tarea Presupuesto creada desde Excel → Control de Gestión | ☐ |
| HU-17 | Líneas OPEX/CAPEX editables en pestañas | ☐ |
| HU-17 | Cerrar Presupuesto dispara creación de Cesta o Derivada | ☐ |

---

*Documento v1.0 — Proyecto HGP B2B — Fase 1 — 06/04/2026*
