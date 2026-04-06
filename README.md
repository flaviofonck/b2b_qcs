# Entrega Fase 1 — Cronos HGP B2B (Versión Desconectada v3)

**Proyecto:** HGP B2B · On Premise · Stack Java 21 · Angular · MySQL 8  
**Referencia de planificación:** [Sprint 1 en `HU/03_Plan_Scrum_Sprints_Desconectado_v3.md`](../HU/03_Plan_Scrum_Sprints_Desconectado_v3.md)  
**Fecha de referencia documental:** 2026-04-04  

Este documento consolida la **documentación de la primera entrega (Fase 1 = Sprint 1)**: alcance acordado, enlaces a historias de usuario y plan Scrum, resumen de implementación en código, y una **guía de prueba muy detallada** para que el usuario final valide en la aplicación todo lo entregado, sin ambigüedades sobre rutas, roles y criterios de éxito.

---

## 1. Qué se entrega en Fase 1

### 1.1 Definición según plan Scrum

En el [plan de sprints](../HU/03_Plan_Scrum_Sprints_Desconectado_v3.md), **Sprint 1 (Semana 1)** tiene como foco:

- **Setup técnico:** repositorio, entorno de desarrollo, base de datos (Flyway), pipeline de CI básico, Docker Compose MySQL (según documentación del repo).
- **HU-03:** tarea **Gestión OBYC** — creación automática desde importación Excel **F1 Ganada + Liberada**, con modelo renombrado respecto a la antigua “Solicitud Grupo Artículo”, asignación a **Contabilidad**, e **hijos por proveedor** cuando el Excel aporta varios proveedores.
- **HU-05:** tarea **Kick Off Entrega** — creación automática desde el mismo Excel, asignación según **complejidad** (regla documentada en la HU), con propagación de campos desde la fila Excel.

**Criterios de Done del Sprint 1** (resumen del plan):

- Ambiente DEV operativo para el equipo.
- HU-03: la tarea aparece como **Gestión OBYC** y se crea de forma automática al procesar el Excel; comportamiento **idempotente** (no duplicar por oportunidad / tipo).
- HU-05: la tarea aparece como **Kick Off Entrega** y se crea automáticamente con el mismo disparador.
- Pruebas unitarias pasando en CI.

### 1.2 Historias de usuario enlazadas (lectura normativa)

| HU | Documento en repositorio |
|----|---------------------------|
| **HU-03** | [`HU/HU-03_Gestión_OBYC_Reclasificación.md`](../HU/HU-03_Gestión_OBYC_Reclasificación.md) |
| **HU-05** | [`HU/HU-05_Kick_Off_Entrega.md`](../HU/HU-05_Kick_Off_Entrega.md) |

Contexto global del producto (13 HUs, stack, módulos): [`.github/copilot-instructions.md`](../.github/copilot-instructions.md).

### 1.3 Documentación técnica de remediación / auditoría Sprint 1 (opcional)

Si necesita trazabilidad de brechas detectadas post-auditoría y tareas de cierre:

- [`docs/files/specs/sem1-sprint-audit/requirements.md`](files/specs/sem1-sprint-audit/requirements.md)
- [`docs/files/specs/sem1-sprint-audit/design.md`](files/specs/sem1-sprint-audit/design.md)
- [`docs/files/specs/sem1-sprint-audit/tasks.md`](files/specs/sem1-sprint-audit/tasks.md)

---

## 2. Qué quedó implementado (resumen técnico sin ambigüedad)

Esta sección alinea **lo que el usuario verá** con **lo que hace el backend** para que no queden dudas de implementación.

### 2.1 Disparador único: Excel F1 Ganada **y** Liberada

- La importación se realiza desde la pantalla **Workflow** (bloque de importación) o vía API `POST` multipart al endpoint documentado en el manual (`/api/excel/f1-ganada`, campo `file`).
- El lector [`F1GanadaExcelReader`](../src/main/java/com/hgp/b2b/excel/F1GanadaExcelReader.java) **solo procesa filas cuyo estado (columna **E**) cumple simultáneamente:**
  - contiene indicación de **F1** y **GANAD** (ganada); y  
  - contiene **LIBERAD** (liberada).  
- Ejemplos válidos de texto en columna E: `F1 Ganada Liberada`, `F1_GANADA_LIBERADA`, `F1 GANADA + LIBERADA` (la lógica normaliza espacios y mayúsculas).

### 2.2 Formato del Excel (primera hoja, fila 1 = encabezados)

El mapeo columnas → negocio está documentado en el propio lector. Referencia rápida:

| Columna | Letra | Uso relevante Fase 1 |
|---------|------|------------------------|
| Número oportunidad | A | Clave de oportunidad |
| Estado | E | **Trigger** F1 Ganada + Liberada |
| Complejidad | F | **HU-05** asignación Kick Off |
| Tipo compra | G | Oportunidad (PUNTUAL / GENERAL) |
| … | … | Resto de campos según DTO |
| Proveedor(s) | AF | **HU-03:** puede listar **varios proveedores** separados por `;` **o** `|` **o** `,`; el motor crea **un hijo OBYC por cada proveedor** (además del padre) |

Texto orientativo para generar datos de prueba: [`scripts/sample-data/LEEME_F1_Ganada_Demo.txt`](../scripts/sample-data/LEEME_F1_Ganada_Demo.txt) y script Python [`scripts/generate_f1_ganada_demo_excel.py`](../scripts/generate_f1_ganada_demo_excel.py).

### 2.3 Motor de reglas: tipos de tarea (`validation_id`)

En [`TaskRulesEngine`](../src/main/java/com/hgp/b2b/rules/TaskRulesEngine.java) consta:

| validation_id | Tarea | Fase 1 |
|---------------|--------|--------|
| **6** | Gestión OBYC **padre** | HU-03 — asignación a rol **CONTABILIDAD** |
| **9** | Gestión OBYC **hijo** (por proveedor) | HU-03 — mismo rol **CONTABILIDAD**, vinculado al padre |
| **7** | Kick Off Entrega | HU-05 — asignación según complejidad (ver §2.4) |

**Idempotencia:** no se duplican tareas del mismo tipo para la misma oportunidad (y los hijos OBYC por proveedor no se duplican si ya existe ese proveedor normalizado).

### 2.4 HU-05 — Regla de asignación Kick Off (código actual)

En implementación, `rolParaKickOff` hace:

- Si **Complejidad** (columna F) es **`BAJA`** (ignorando mayúsculas) → asignación a rol **PM**.
- Cualquier otra complejidad → rol **CONTROL_GESTION** (en el código se documenta como proxy de **Planificación** frente a la tabla de la HU-05; valide con negocio si el catálogo de roles debe mostrar otro nombre en pantalla).

Si **no hay ningún usuario activo** con el rol destino, la tarea puede quedar **sin asignado** y el sistema registra advertencia en log — en pruebas, debe existir al menos un usuario por rol necesario.

### 2.5 HU-03 — Padre + hijos por proveedor

- Siempre se intenta crear la tarea **padre** Gestión OBYC (vid 6).
- A partir de la celda de **proveedor(es)** (AF), se segmenta la lista; por cada proveedor distinto se crea un **hijo** (vid 9) si aún no existía para esa oportunidad y proveedor.
- Si la celda de proveedor viene vacía, puede existir solo el **padre** sin hijos (según datos de la fila).

### 2.6 Nota importante: misma importación crea también Presupuesto (alcance posterior)

El mismo método `procesarF1GanadaRow` crea, cuando corresponde, la tarea **Presupuesto** (**HU-17**, Sprint 3 en el plan de producto). **No forma parte del alcance funcional de “solo Sprint 1” en el plan Scrum**, pero **sí puede aparecer en la aplicación** tras importar, porque comparte el disparador Excel. Para una demo estrictamente “Sprint 1”, puede ignorarse la verificación de Presupuesto o documentarse como dependencia técnica compartida.

### 2.7 Workflow

Tras procesar filas, el motor refresca etapas de workflow para la oportunidad (`workflowService.inicializarSiAusente` / `refrescarEtapasDesdeTareas`). La **visualización completa** del tablero por etapas es objeto de **fases posteriores** (HU-20, HU-21, HU-22); en Fase 1 lo verificable es que existan las tareas y rutas de compras.

---

## 3. Infraestructura y entorno para probar (SETUP)

### 3.1 Base de datos

- MySQL 8, base `hgp_b2b`, puerto típico `3306`.
- En desarrollo puede usarse `docker-compose` en la raíz del backend (según [copilot-instructions](../.github/copilot-instructions.md)).
- Import inicial: script `scripts/db-import.ps1` (Windows) según instrucciones del proyecto.

### 3.2 Backend y frontend (puertos en scripts del repo)

Los scripts batch del repositorio usan por defecto:

- **Backend:** `http://localhost:8081` (véase [`iniciar-backend.bat`](../iniciar-backend.bat)).
- **Frontend:** `http://localhost:4201` (véase [`iniciar-frontend.bat`](../iniciar-frontend.bat)).

El frontend apunta al API en [`hgp-b2b-frontend/src/environments/environment.ts`](../hgp-b2b-frontend/src/environments/environment.ts): `apiUrl: 'http://localhost:8081/api'`.

Si su documentación genérica menciona `:8080` o `:4200`, para esta entrega **siga los puertos anteriores** salvo que haya cambiado `environment` o configuración local.

### 3.3 CI

- Workflow Maven: [`.github/workflows/maven.yml`](../.github/workflows/maven.yml) (ejecución de tests en JDK 21).

---

## 4. Guía de prueba para el usuario final (paso a paso)

### 4.1 Preparación

1. Arranque MySQL y cargue datos base (usuarios por rol **CONTABILIDAD**, **PM**, **CONTROL_GESTION**, y un usuario con permiso de **importación Excel**: típicamente **PM**, **CONTROL_GESTION**, **SOPORTE** o **ADMIN**, según mensaje en pantalla).
2. Arranque el backend y espere a que la API responda.
3. Arranque el frontend y abra la URL correcta (`http://localhost:4201` con los scripts actuales).
4. Inicie sesión con un usuario **autorizado para importar** (si la importación falla con “No tiene permiso…”, use otro usuario con rol indicado en el mensaje).

### 4.2 Prueba A — Importación correcta y aparición de tareas HU-03 / HU-05

1. Obtenga o genere un archivo **`.xlsx`** compatible (ver §2.2 y `LEEME_F1_Ganada_Demo.txt`).
2. Confirme que al menos una fila de datos tenga en columna **E** un texto que dispare el trigger (p. ej. `F1 Ganada Liberada`).
3. Menú lateral → **Workflow** (`/workflow`).
4. Localice el bloque **Importar F1 Ganada + Liberada** (o equivalente).
5. Pulse **Elegir archivo Excel**, seleccione el `.xlsx` y confirme la carga.
6. Espere el mensaje de **éxito** o el detalle de error (si hay error, corrija el archivo o permisos y repita).

**Qué debe comprobar:**

- En **Compras** (`/compras`), en el listado de tareas abiertas, aparecen filas cuyo **tipo** corresponde a **Gestión OBYC** (padre y, si aplica, hijos por proveedor) y **Kick Off Entrega**.
- Anote el **ID de tarea** y el **número de oportunidad** para las pruebas B y C.

### 4.3 Prueba B — HU-03 Gestión OBYC (navegación y bandeja Contabilidad)

1. Cierre sesión e inicie sesión con un usuario de rol **Contabilidad** (según datos semilla de su BD).
2. Vaya a **Compras** y abra una tarea **Gestión OBYC** con **Abrir tarea** (o la acción equivalente).
3. La ruta esperada de detalle es del estilo: `/compras/gestion-obyc/{id}`.

**Criterios de éxito HU-03 en esta fase:**

- La tarea existe **sin creación manual** previa (solo por importación).
- Si el Excel tenía **varios proveedores** en columna AF, existen **varias** tareas hijas OBYC (además del padre), diferenciables por proveedor en listado o detalle según la UI.
- La tarea está asignada al flujo de **Contabilidad** cuando hay usuario disponible para ese rol.

> **Nota:** La pantalla completa de **PxQ manual y tabla contable** (relleno de complemento GA, cuenta contable, cierre) corresponde a **HU-04 (Sprint 2)**. En Fase 1 la prueba objetivo es **existencia, creación automática, asignación e idempotencia**, no el cierre contable avanzado.

### 4.4 Prueba C — HU-05 Kick Off Entrega (asignación por complejidad)

1. Prepare **dos filas** en Excel (u oportunidades distintas) con el mismo trigger en columna E:
   - Fila 1: **Complejidad** = `BAJA` → según código, Kick Off debe asignarse a un usuario **PM** (si existe usuario activo PM).
   - Fila 2: **Complejidad** = `MEDIA` (o ALTA / ESPECIAL) → Kick Off debe asignarse a **CONTROL_GESTION** (Planificación en negocio).
2. Importe el archivo.
3. Inicie sesión como **PM** y verifique que ve la tarea Kick Off de la oportunidad de complejidad **BAJA** (si la asignación está hecha a rol PM).
4. Inicie sesión como usuario **CONTROL_GESTION** y verifique la tarea de la oportunidad de complejidad no baja.

**Ruta de detalle:** `/compras/kick-off-entrega/{id}`.

**Criterios de éxito HU-05:**

- Tarea creada automáticamente; nombre/tipo coherente con **Kick Off Entrega**.
- Campos de cabecera que provengan del Excel (proveedor, contrato, valores, etc.) visibles cuando el DTO los trajo — si un campo no venía en Excel, puede mostrarse vacío según HU-05.

### 4.5 Prueba D — Idempotencia (no duplicar al reimportar)

1. Vuelva a importar **el mismo archivo** sin cambiar números de oportunidad.
2. **No** deben crearse tareas duplicadas del mismo tipo para la misma oportunidad (ni hijos OBYC duplicados para el mismo proveedor).

### 4.6 Prueba E — API directa (opcional, equipo técnico)

`POST /api/excel/f1-ganada` con `multipart/form-data`, campo `file` = Excel. Debe producir el mismo efecto que la UI. Útil para automatización o diagnóstico.

---

## 5. Matriz rápida: HU Fase 1 ↔ dónde probarlo

| Entregable | HU | Pantalla / ruta principal | Rol típico |
|------------|-----|-----------------------------|------------|
| Gestión OBYC automática + hijos por proveedor | HU-03 | Workflow (import) → Compras → `/compras/gestion-obyc/{id}` | Contabilidad |
| Kick Off Entrega automático + asignación por complejidad | HU-05 | Workflow (import) → Compras → `/compras/kick-off-entrega/{id}` | PM o Control de Gestión / Planificación |
| Excel F1 Ganada + Liberada | HU-03 + HU-05 | `/workflow` — importación | PM, Control de Gestión, Soporte o Admin |

---

## 6. Qué **no** se exige validar como “solo Fase 1”

Para no mezclar alcances:

- **HU-04** (PxQ manual, tabla contable completa, cierre OBYC avanzado) — Sprint 2.
- **HU-09, HU-01** (Gestión Cesta y campos extendidos) — Sprint 2.
- **HU-08, HU-10, HU-17** (FOD, Derivada, Presupuesto funcional completo) — Sprint 3 en el plan (aunque el trigger de Presupuesto pueda ya crear la tarea al importar).
- **HU-16, HU-20, HU-21, HU-22, HU-24** — Sprints 4–5.

El [manual de usuario general](manual-usuario.md) cubre el flujo completo v3; use este documento **solo** para acotar la **primera entrega**.

---

## 7. Documentación relacionada

| Documento | Uso |
|-----------|-----|
| [Manual de usuario (v3 completo)](manual-usuario.md) | Navegación y módulos finales |
| [Despliegue on premise](despliegue-on-premise.md) | Instalación en entorno productivo |
| [`.github/copilot-instructions.md`](../.github/copilot-instructions.md) | Stack, rutas backend, roadmap |

---

*Documento generado para la entrega de Fase 1 (Sprint 1). Ajuste fechas y notas de waiver si su comité de proyecto acordó excepciones formales sobre alcance.*
