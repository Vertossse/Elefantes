# Modelo físico de datos

## Alcance

Este documento explica el modelo Entidad-Relación (ER) físico del sistema de facturación de TurbineH: qué tablas tiene, cómo se relacionan y por qué se diseñó así. El modelo es para PostgreSQL y se obtuvo del diagrama de clases siguiendo la guía de clases y ER acordada en la reunión del grupo. Tiene 30 tablas y cubre los cuatro bloques: usuarios y clientes, suscripciones y facturación, cobros y seguimiento (tareas, notificaciones y movimientos).

El diccionario de datos (`modelo_datos.md`) detalla columna por columna el tipo, las restricciones y las notas de cada tabla. Este documento no lo repite.

![Diagrama Entidad-Relación de TurbineH](ER_Turbine.png#land)

El diagrama también está en vector en `doc/er.pdf` del repositorio. En él, `PK` marca la clave primaria, `FK` la foránea y `UK` un valor único. El círculo negro (●) delante de una columna indica que no admite nulos. La leyenda está en la esquina inferior derecha.

## Cómo se pasó el diagrama de clases al ER

1. Solo las clases `<<entity>>` se convierten en tablas. Gestores, repositorios y otras clases de control no tienen tabla, y los métodos no se pasan. No toda clase implica una tabla.
2. Las asociaciones de uno a muchos se convierten en una clave foránea en el lado "muchos".
3. Las asociaciones de muchos a muchos (agregación) se convierten en una tabla intermedia: `cliente_contacto` y `cliente_etiqueta`.
4. La composición se convierte en una clave foránea obligatoria.
5. La herencia se resuelve con una tabla general con los datos comunes y una tabla por cada subclase con datos propios. La clave de la subclase es también clave foránea de la tabla general.
6. Las claves primarias son `BIGINT` propias. Los estados son `VARCHAR` con `CHECK`. Las fechas de negocio son `DATE` y los instantes `TIMESTAMPTZ`.
7. Los PDF y CSV no se guardan: se generan al emitir o descargar.

## Descripción de las tablas

### Usuarios y clientes

| Tabla | Clase de origen | Qué guarda |
|---|---|---|
| `usuario` | Usuario (UO, AD, SAD) | Empleados de TurbineH. Una sola tabla con la columna `rol`. |
| `asignacion_cliente` | AsignacionCliente | Historial de quién ha llevado a cada cliente. |
| `cliente` | Cliente | Empresas que contratan el servicio. Baja lógica, nunca se borra la fila. |
| `contacto` | Contacto | Personas de contacto, con consentimiento de correo y WhatsApp e idioma. |
| `cliente_contacto` | (agregación) | Cliente y contacto, de muchos a muchos. |
| `etiqueta` | Etiqueta | Clasificación libre de clientes. |
| `cliente_etiqueta` | (agregación) | Cliente y etiqueta, de muchos a muchos. |

### Suscripción y tarifas

| Tabla | Clase de origen | Qué guarda |
|---|---|---|
| `tarifa_rango` | TarifaRango | Tramos de facturación con su importe mensual. Solo los edita el SAD. |
| `suscripcion` | Suscripcion | Contrato mensual de un cliente. Guarda el precio acordado aunque el tramo cambie. |
| `excepcion_comercial` | ExcepcionComercial | Descuentos y condiciones vigentes de una suscripción. |

### Facturación

| Tabla | Clase de origen | Qué guarda |
|---|---|---|
| `concepto` | Concepto | Catálogo de conceptos facturables. |
| `factura` | Factura | Facturas recurrentes, puntuales y rectificativas, con sus dos estados. |
| `factura_rectificativa` | FacturaRectificativa | Factura original a la que corrige y quién la aprobó. |
| `periodo_administrativo` | PeriodoAdministrativo | Trimestres contables. |

### Cobros

| Tabla | Clase de origen | Qué guarda |
|---|---|---|
| `movimiento_cobro` | MovimientoCobro | Cobros recibidos de una pasarela o del banco. No es la tabla `movimiento`. |
| `conciliacion_cobro` | Conciliacion | Vínculo entre un cobro y una factura. Estado CONCILIADO o REVERTIDO. |

### Tareas

| Tabla | Clase de origen | Qué guarda |
|---|---|---|
| `tarea` | Tarea (abstracta) | Datos generales: estado, ejecutor (usuario o sistema) y fechas. |
| `delegacion_temporal` | DelegacionTemporal | Tarea de sistema. La caducidad la ejecuta el Reloj. |
| `revision_anual_tarifa` | RevisionAnual | Tarea de sistema: revisión del tramo de un cliente. |
| `solicitud_aprobacion` | SolicitudAprobacion | Tarea de usuario: operación que valida un aprobador. |
| `disputa_deuda` | DisputaDeuda | Tarea de usuario: reclamación que congela los recordatorios. |
| `incidencia_facturacion` | IncidenciaFacturacion | Tarea de usuario: incidencia con tipo y prioridad. |
| `comentario_incidencia` | ComentarioIncidencia | Comentarios de una incidencia (composición). |

### Movimientos, notificaciones y sistema

| Tabla | Clase de origen | Qué guarda |
|---|---|---|
| `notificacion` | Notificacion (Correo, WhatsApp, NotificacionUsuario) | Todas las notificaciones, con la columna `tipo`. |
| `movimiento` | Movimiento (abstracta) | Registro de todo lo que ocurre en el sistema. |
| `envio_comunicacion` | EnvioComunicacion | Envíos externos a un cliente. |
| `registro_auditoria` | RegistroAuditoria | Bitácora de cambios con valor anterior y nuevo. |
| `importacion_csv` | ImportacionCSV | Traza de cada carga masiva. |
| `informe` | Informe | Informes generados. No se guarda el archivo. |
| `ajustes_sistema` | AjustesSistema (Singleton) | Parámetros globales clave-valor: series, festivos y filtros guardados. |

## Justificación del modelo

- `cliente (1)` — `suscripcion (1)`: cada cliente tiene una sola suscripción y no existe sin él (composición, FK obligatoria y única).
- `cliente (1)` — `asignacion_cliente (0..N)`: un cliente acumula asignaciones a lo largo del tiempo. `usuario (1)` — `asignacion_cliente (0..N)`: un empleado ha llevado varios clientes.
- `cliente (N)` — `contacto (N)` mediante `cliente_contacto`: un contacto puede estar en varios clientes y un cliente tiene varios contactos.
- `cliente (N)` — `etiqueta (N)` mediante `cliente_etiqueta`.
- `tarifa_rango (1)` — `suscripcion (0..N)`: un tramo lo usan muchas suscripciones.
- `suscripcion (1)` — `excepcion_comercial (0..N)`: composición.
- `cliente (1)` — `factura (0..N)`: un cliente puede no tener facturas todavía. Una factura recurrente además apunta a su suscripción.
- `factura (1)` — `factura_rectificativa (0..1)`: la rectificativa extiende a la factura y apunta a la original.
- `movimiento_cobro (N)` — `factura (N)` mediante `conciliacion_cobro`: un cobro se asocia a una factura y se puede deshacer conservando la traza.
- `tarea (1)` — subclases (0..1): cada tarea tiene como mucho una fila en la tabla de su tipo, con la misma clave.
- `incidencia_facturacion (1)` — `comentario_incidencia (0..N)`: composición.
- `movimiento (1)` — `envio_comunicacion`, `registro_auditoria`, `importacion_csv` (0..1): solo esos tres tipos tienen datos propios. El resto usa solo `movimiento`.
- `usuario (1)` — `movimiento (0..N)`: quién realizó cada operación. `usuario (1)` — `notificacion (0..N)`: destinatario de los avisos internos.

Decisiones de diseño:

- **Usuario.** Una tabla con `rol` en lugar de tres: los tres roles comparten casi todos los datos.
- **Estados de la factura.** Dos ejes separados, `estado_documento` (borrador, emitida, rectificada, anulada) y `estado_cobro` (pendiente, pagada, incobrable), porque una factura puede estar emitida y pendiente de cobro. «Pagada» solo significa cobro detectado o validado (FAC-02).
- **Notificación.** Una tabla con `tipo`, porque correo, WhatsApp y notificación de usuario comparten casi todas las columnas.
- **Movimiento.** Base de la auditoría no editable (AUD-01): solo se insertan filas.
- **Cliente.** Clave propia y NIF/CIF único, para que las referencias no se rompan si se reactiva una ficha.
- **Ajustes.** Tabla clave-valor para añadir parámetros sin cambiar el esquema.
- **Borrado.** No se borran clientes, facturas ni movimientos: se marcan con un estado.

## Reglas y restricciones

La notación gráfica no valida estas reglas, por eso se describen aquí:

- Claves: `cliente.nif_cif`, `usuario.email` y `etiqueta.nombre` son únicos.
- `usuario`: `umbral_aprobacion` solo se rellena con rol AD o SAD y `puede_dar_alta_clientes` solo con rol UO (`CHECK`).
- `asignacion_cliente`: un solo responsable vigente por cliente (índice único parcial con `fecha_fin` nula, excluyendo las delegaciones).
- `suscripcion.cliente_id` es único. Las excepciones y los comentarios tienen FK obligatoria hacia su padre (composición).
- `factura`: número único dentro de la serie (`UNIQUE (serie, numero_secuencial)`) y una sola factura recurrente por suscripción y mes (FAC-04, GEN-01). Los estados llevan `CHECK` con sus valores permitidos.
- `factura_rectificativa` y las subclases de `tarea` y `movimiento`: su clave primaria es también FK de la tabla general.
- `movimiento` y sus subtablas son de solo inserción: la aplicación solo hace `INSERT` y `SELECT` (AUD-01).
- `conciliacion_cobro`: se puede revertir, pero no se borra la fila.
- Reglas que se validan en el gestor, no en la base de datos: cada cliente activo tiene al menos un contacto de facturación activo (RB-16) y una factura emitida no se modifica, se corrige con una rectificativa (RB-02, FAC-06).

## Trazabilidad

| Requisitos | Tablas | Observación |
|---|---|---|
| CLI-01, CLI-02, CLI-03, CLI-04, CLI-05 | `cliente`, `contacto`, `cliente_contacto`, `etiqueta`, `cliente_etiqueta`, `asignacion_cliente` | Empresas, contactos, responsable, baja lógica y etiquetas. |
| USU-01, USU-02, USU-03 | `usuario`, `asignacion_cliente` | Tres perfiles y asignación de clientes. |
| SUB-01, SUB-02, SUB-03, SUB-04 | `suscripcion` | Relación comercial separada de las facturas, con pausas y bajas. |
| TAR-01, TAR-02 | `tarifa_rango`, `revision_anual_tarifa` | Rangos de tarifa y su revisión anual. |
| APR-01, APR-02 | `excepcion_comercial`, `solicitud_aprobacion` | Excepciones comerciales y aprobaciones. |
| FAC-01, FAC-02, FAC-03, FAC-04, FAC-06 | `factura`, `factura_rectificativa`, `concepto`, `periodo_administrativo` | Ciclo de vida, numeración y rectificación. |
| GEN-01 | `factura`, `suscripcion` | Facturas recurrentes sin duplicados. |
| PAG-02, PAG-03 | `movimiento_cobro`, `conciliacion_cobro` | Conciliación independiente de la fuente y reversible. |
| REC-02, NOT-01, NOT-02 | `disputa_deuda`, `notificacion`, `incidencia_facturacion`, `comentario_incidencia`, `tarea` | Disputas, notificaciones e incidencias. |
| ENV-02 | `envio_comunicacion` | Registro de cada envío. |
| AUD-01 | `movimiento`, `registro_auditoria` | Auditoría no editable. |
| IMP-01 | `importacion_csv` | Traza de cada importación. |
| INF-01 | `informe` | Informes generados. |
| CFG-01 | `ajustes_sistema` | Configuración global. |
| PRO-04 | `periodo_administrativo` | Cierre administrativo de periodos. |

## Decisiones abiertas

1. **Conceptos por factura.** El modelo guarda un solo concepto por factura, pero FAC-01 pide facturas con varias líneas. Si el grupo mantiene varias líneas, hay que añadir una tabla `factura_concepto`. Es el punto más importante por confirmar.
2. Domicilio fiscal del cliente: campos separados (como está) o texto único (como dice la guía).
3. Estado de la factura: dos ejes (como está) o un único campo como en la guía.
4. Estado del cliente: se guarda (como dice la guía) o se deriva de la suscripción.
5. Valor `DECLARACION_INCOBRABLE` en `solicitud_aprobacion.tipo_operacion` (cerrado por la guía).
6. Dónde se guarda el resumen de los procesos automáticos (decimocuarta subclase de movimiento).
7. Nivel de visibilidad de los comentarios de incidencia.
8. Las tablas propias de las subclases de Tarea y Movimiento (propuesta de la guía, por validar).
