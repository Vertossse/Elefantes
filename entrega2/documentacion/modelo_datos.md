# Modelo físico de datos

## Alcance

Este documento explica el modelo Entidad-Relación (ER) físico del sistema de facturación de TurbineH: qué tablas tiene, cómo se relacionan y por qué se diseñó así. El modelo es para PostgreSQL y se obtuvo de los diagramas de clases finales siguiendo la guía de clases y ER acordada en la reunión del grupo. Tiene 42 tablas y cubre los cuatro bloques: usuarios y clientes, suscripciones y facturación, cobros y seguimiento (tareas, notificaciones y movimientos).

El diccionario de datos (documento «Diccionario de datos ER») detalla columna por columna el tipo, las restricciones y las notas de cada tabla. Este documento no lo repite.

En los diagramas, `PK` marca la clave primaria, `FK` la foránea y `UK` un valor único. El círculo negro (●) delante de una columna indica que no admite nulos. Las claves foráneas hacia `usuario` figuran como columna y solo se dibujan como línea las principales, para no saturar.

## Diagrama

El diagrama completo está en `entrega2/diagramas/modelo_datos.puml` (fuente PlantUML) y en el informe de la entrega.

## Cómo se pasó el diagrama de clases al ER

1. Solo las clases `<<entity>>` se convierten en tablas. Gestores, repositorios y otras clases de control no tienen tabla, y los métodos no se pasan. No toda clase implica una tabla: por ejemplo, los filtros guardados no son tablas propias.
2. Las asociaciones de uno a muchos se convierten en una clave foránea en el lado «muchos».
3. Las asociaciones de muchos a muchos se convierten en una tabla intermedia: `cliente_contacto`, `cliente_etiqueta` y `factura_concepto`. Un atributo en forma de mapa, como `canalesConsentidos`, se convierte en una tabla propia (`contacto_canal`).
4. La composición se convierte en una clave foránea obligatoria.
5. La herencia se resuelve con una tabla general con los datos comunes y una tabla por cada subclase con datos propios. La clave de la subclase es también clave foránea de la tabla general.
6. Las claves primarias son `BIGINT` propias. Los estados son `VARCHAR` con `CHECK`. Las fechas de negocio son `DATE` y los instantes `TIMESTAMPTZ`.
7. Los PDF y CSV no se guardan: se generan al emitir o descargar.

## Descripción de las tablas

### Usuarios y clientes

| Tabla | Clase de origen | Qué guarda |
|---|---|---|
| `usuario` | Usuario (UO, AD, SAD) | Empleados de TurbineH. Una sola tabla con la columna `rol`. |
| `asignacion_cliente` | (historial de responsables) | Quién ha llevado a cada cliente a lo largo del tiempo. |
| `cliente` | Cliente | Empresas que contratan el servicio. Estados ACTIVO, PAUSADO e INACTIVO; nunca se borra la fila. |
| `contacto` | Contacto | Personas de contacto con su rol, idioma y estado. |
| `contacto_canal` | Contacto (`canalesConsentidos`) | Canales (correo, WhatsApp) que el contacto ha consentido, con el correo o teléfono de cada uno. |
| `cliente_contacto` | (agregación) | Cliente y contacto, de muchos a muchos. |
| `etiqueta` | Etiqueta | Clasificación libre de clientes. |
| `cliente_etiqueta` | (agregación) | Cliente y etiqueta, de muchos a muchos. |

### Suscripción y tarifas

| Tabla | Clase de origen | Qué guarda |
|---|---|---|
| `tarifa_rango` | TarifaRango | Tramos de facturación con su importe mensual. Solo los edita el SAD. |
| `suscripcion` | Suscripcion | Contrato mensual de un cliente. Guarda el precio acordado aunque el tramo cambie y, si la tiene, la excepción comercial aplicada. Pertenece a un solo cliente. |
| `excepcion_comercial` | ExcepcionComercial | Descuentos y condiciones configuradas en el sistema (descuento temporal, primer mes gratis, precio pactado, sin prorrateo). |

### Facturación

| Tabla | Clase de origen | Qué guarda |
|---|---|---|
| `concepto` | Concepto | Catálogo de conceptos facturables. |
| `factura` | Factura (Puntual, Recurrente, Rectificada) | Facturas de los tres tipos, con un solo campo de estado y el periodo administrativo. |
| `factura_concepto` | (Concepto – Factura) | Conceptos de cada factura, con una copia de su descripción y precio. |
| `factura_rectificativa` | Factura_Rectificada | Motivo, importe ajustado y factura original a la que corrige. |
| `periodo_administrativo` | PeriodoAdministrativo | Trimestres contables. |

### Cobros

| Tabla | Clase de origen | Qué guarda |
|---|---|---|
| `movimiento_cobro` | MovimientoCobro | Cobros recibidos de una pasarela o del banco. No es la tabla `movimiento`. |
| `conciliacion_cobro` | Conciliacion | Vínculo entre un cobro y una factura. Estado CONCILIADO o REVERTIDO, con el motivo si se revierte. |

### Tareas

| Tabla | Clase de origen | Qué guarda |
|---|---|---|
| `tarea` | Tarea (abstracta) | Datos generales: estado, ejecutor (usuario o sistema) y fechas. |
| `delegacion_temporal` | DelegacionTemporal | Tarea de sistema. La caducidad la ejecuta el Reloj. |
| `revision_anual_tarifa` | RevisionAnual | Tarea de sistema: revisión del tramo de un cliente. |
| `solicitud_aprobacion` | SolicitudAprobacion | Tarea de usuario: operación que valida un aprobador. |
| `disputa_deuda` | DisputaDeuda | Tarea de usuario: reclamación que congela los recordatorios. |
| `incidencia_facturacion` | IncidenciaFacturacion | Tarea de usuario: incidencia con tipo, prioridad y estado. |
| `comentario_incidencia` | ComentarioIncidencia | Comentarios de una incidencia (composición). |

### Notificaciones y movimientos

| Tabla | Clase de origen | Qué guarda |
|---|---|---|
| `notificacion` | Notificacion (CorreoElectronico, WhatsApp, NotificacionInterna) | Todas las notificaciones, con la columna `tipo` y los datos propios de cada una. |
| `movimiento` | Movimiento (abstracta) | Registro de todo lo que ocurre en el sistema. Solo se añaden filas. |
| `envio_comunicacion` | EnvioComunicacion | Envíos externos a un contacto, con su estado e intentos. |
| `registro_auditoria` | RegistroAuditoria | Bitácora de cambios con valor anterior y nuevo. |
| `importacion_csv` | ImportacionCSV | Traza de cada carga masiva. |
| `alta_baja_cliente` | AltaBajaCliente | Altas y bajas de clientes. |
| `alta_baja_usuario` | AltaBajaEmpleado | Altas y bajas de empleados. |
| `asignacion_desasignacion_cliente` | AsignacionDesasigancionCliente | Cambios de responsable de un cliente. |
| `generacion_factura` | GeneracionFactura | Facturas generadas. |
| `rectificacion_factura` | RectificacionFactura | Facturas rectificadas y su fecha. |
| `generacion_informe` | GeneracionInforme | Informes generados. |
| `aplicacion_excepcion` | AplicarExcepcionComercial | Excepciones comerciales aplicadas a una factura. |
| `registro_cobro` | RegistroCobro | Cobros registrados y su conciliación. |
| `registro_tarea` | RegistroTarea | Tareas registradas. |
| `cierre_periodo` | CierrePeriodoAdministrativo | Cierres de periodo. |

### Sistema

| Tabla | Clase de origen | Qué guarda |
|---|---|---|
| `informe` | Informe | Informes generados con su usuario, cliente, factura y periodo. No se guarda el archivo. |
| `ajustes_sistema` | AjustesSistema (Singleton) | Parámetros globales clave-valor: series, festivos y filtros guardados. |

## Justificación del modelo

- `cliente (1)` — `suscripcion (1)`: cada cliente tiene una sola suscripción y no existe sin él (composición, FK obligatoria y única).
- `suscripcion (0..N)` — `excepcion_comercial (0..1)`: una suscripción puede tener una excepción, y una misma excepción configurada puede usarse en varias suscripciones. La suscripción guarda la referencia.
- `tarifa_rango (1)` — `revision_anual_tarifa (0..N)`: el tramo propuesto en cada revisión anual. La suscripción no guarda el tramo: guarda el precio acordado.
- `cliente (1)` — `asignacion_cliente (0..N)`: un cliente acumula asignaciones a lo largo del tiempo. `usuario (1)` — `asignacion_cliente (0..N)`: un empleado ha llevado varios clientes.
- `cliente (N)` — `contacto (N)` mediante `cliente_contacto`; `cliente (N)` — `etiqueta (N)` mediante `cliente_etiqueta`. `contacto (1)` — `contacto_canal (1..N)`: cada contacto tiene al menos un canal consentido.
- `cliente (1)` — `factura (0..N)`: un cliente puede no tener facturas todavía. Una factura recurrente además apunta a su suscripción.
- `factura (N)` — `concepto (N)` mediante `factura_concepto`: una factura tiene uno o más conceptos, y un concepto aparece en muchas facturas.
- `periodo_administrativo (1)` — `factura (0..N)`: cada factura emitida pertenece a un periodo.
- `factura (1)` — `factura_rectificativa (0..1)`: la rectificativa extiende a la factura y apunta a la original.
- `movimiento_cobro (1)` — `conciliacion_cobro (0..1)` — `factura (N)`: un cobro se asocia a una factura y se puede deshacer conservando la traza.
- `tarea (1)` — subclases (0..1): cada tarea tiene como mucho una fila en la tabla de su tipo, con la misma clave.
- `incidencia_facturacion (1)` — `comentario_incidencia (0..N)`: composición.
- `movimiento (1)` — cada una de las 13 tablas de subclase (0..1): cada movimiento tiene una sola fila en la tabla de su tipo.
- `notificacion (1)` — `envio_comunicacion (0..N)`: un envío externo guarda qué notificación envió. `usuario (1)` — `notificacion (0..N)`: responsable de cada una.
- `usuario (1)` — `movimiento (0..N)`: quién realizó cada operación.

Decisiones de diseño:

- **Usuario.** Una tabla con `rol` en lugar de tres: los tres roles comparten casi todos los datos.
- **Estado de la factura.** Un solo campo con los seis estados de las clases (borrador, emitida, pagada, anulada, rectificada, incobrable). «Pagada» solo significa cobro detectado o validado (FAC-02).
- **Conceptos.** Una factura tiene varios conceptos (FAC-01). Como el catálogo puede cambiar sin alterar facturas antiguas (RNG-14), cada línea guarda una copia de la descripción, el precio y el impuesto.
- **Notificación.** Una tabla con `tipo`, porque correo, WhatsApp y notificación interna comparten casi todas las columnas. Las columnas específicas son nulas para los demás tipos.
- **Movimiento.** Base de la auditoría no editable (AUD-01): solo se insertan filas. Las 13 subclases tienen atributos propios, así que cada una tiene su tabla.
- **Excepción comercial.** Es una configuración del sistema que la suscripción referencia. Ya no es una composición de la suscripción.
- **Cliente.** Clave propia y NIF/CIF único, para que las referencias no se rompan si se reactiva una ficha.
- **Ajustes y filtros.** Tabla clave-valor para añadir parámetros sin cambiar el esquema. Los filtros guardados van como filas de esta tabla.
- **Borrado.** No se borran clientes, facturas ni movimientos: se marcan con un estado.

## Reglas y restricciones

La notación gráfica no valida estas reglas, por eso se describen aquí:

- Claves: `cliente.nif_cif`, `usuario.email` y `etiqueta.nombre` son únicos.
- `usuario`: `umbral_aprobacion` solo se rellena con rol AD o SAD y `puede_dar_alta_clientes` solo con rol UO (`CHECK`).
- `asignacion_cliente`: un solo responsable vigente por cliente (índice único parcial con `fecha_fin` nula, excluyendo las delegaciones).
- `suscripcion.cliente_id` es único. Los comentarios de incidencia tienen FK obligatoria hacia su incidencia (composición).
- `factura`: número único dentro de la serie (`UNIQUE (serie, numero_secuencial)`) y una sola factura recurrente por suscripción y mes (FAC-04, GEN-01). El estado lleva `CHECK` con sus seis valores y una factura en borrador no tiene número.
- `factura_concepto`: toda factura emitida tiene al menos un concepto (se valida en el gestor).
- `notificacion`: cada columna específica solo se rellena para su tipo (`CHECK`).
- `conciliacion_cobro`: el motivo de reversión es obligatorio si el estado es REVERTIDO. Se puede revertir, pero no se borra la fila.
- `factura_rectificativa`, las subclases de `tarea` y las de `movimiento`: su clave primaria es también FK de la tabla general.
- `movimiento` y sus subtablas son de solo inserción: la aplicación solo hace `INSERT` y `SELECT` (AUD-01).
- Reglas que se validan en el gestor, no en la base de datos: cada cliente activo tiene al menos un contacto de facturación activo (RB-16), cada contacto tiene al menos un canal consentido y una factura emitida no se modifica, se corrige con una rectificativa (RB-02, FAC-06).

## Trazabilidad

| Requisitos | Tablas | Observación |
|---|---|---|
| CLI-01, CLI-02, CLI-03, CLI-04, CLI-05 | `cliente`, `contacto`, `contacto_canal`, `cliente_contacto`, `etiqueta`, `cliente_etiqueta`, `asignacion_cliente` | Empresas, contactos, responsable, baja lógica y etiquetas. |
| USU-01, USU-02, USU-03 | `usuario`, `asignacion_cliente` | Tres perfiles y asignación de clientes. |
| SUB-01, SUB-02, SUB-03, SUB-04 | `suscripcion` | Relación comercial separada de las facturas, con pausas y bajas. |
| TAR-01, TAR-02 | `tarifa_rango`, `revision_anual_tarifa` | Rangos de tarifa y su revisión anual. |
| APR-01, APR-02 | `excepcion_comercial`, `solicitud_aprobacion` | Excepciones comerciales y aprobaciones. |
| FAC-01, FAC-02, FAC-03, FAC-04, FAC-06 | `factura`, `factura_concepto`, `factura_rectificativa`, `concepto`, `periodo_administrativo` | Varios conceptos, ciclo de vida, numeración y rectificación. |
| GEN-01 | `factura`, `suscripcion` | Facturas recurrentes sin duplicados. |
| PAG-02, PAG-03 | `movimiento_cobro`, `conciliacion_cobro` | Conciliación independiente de la fuente y reversible. |
| REC-02, NOT-01, NOT-02 | `disputa_deuda`, `notificacion`, `incidencia_facturacion`, `comentario_incidencia`, `tarea` | Disputas, notificaciones e incidencias. |
| ENV-01, ENV-02, ENV-03 | `notificacion`, `envio_comunicacion` | Envío por correo o WhatsApp, registro de cada envío y rebotes. |
| AUD-01 | `movimiento` y sus 13 subtablas, `registro_auditoria` | Auditoría no editable. |
| IMP-01 | `importacion_csv` | Traza de cada importación. |
| INF-01 | `informe`, `generacion_informe` | Informes generados. |
| CFG-01 | `ajustes_sistema` | Configuración global. |
| PRO-04 | `periodo_administrativo`, `cierre_periodo` | Cierre administrativo de periodos. |

## Decisiones abiertas

1. `asignacion_cliente` (historial de responsables) no está en las clases finales. Se mantiene en el ER: el historial se podría recuperar también de los movimientos de asignación, pero así se consulta directamente. Conviene añadirlo a las clases como una lista.
2. `factura_rectificativa.factura_original_id` no está en la clase Factura_Rectificada, pero FAC-06 exige vincular la rectificativa con la original. Se conserva y conviene añadirlo a las clases.
3. Las columnas de copia de `factura_concepto` (precio, descripción, impuesto) no están en las clases; hacen falta para no alterar facturas antiguas (RNG-14).
4. Los filtros guardados (`FiltroClientes`, `FiltroFacturas`...) están marcados como `<<entity>>` en las clases. En el ER van como filas de `ajustes_sistema` (tipo FILTRO), no como tablas.
5. Dónde se guarda el resumen de los procesos automáticos (decimocuarta subclase de movimiento).
