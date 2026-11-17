# Diccionario de datos del ER físico

Motor: PostgreSQL. Base: guía de clases y ER (versión tras los cambios de la reunión). Cada tabla indica la clase de la que sale.

## Decisiones aplicadas

- Solo las clases `<<entity>>` son tablas. Los métodos no se pasan.
- Herencia de Usuario: una tabla `usuario` con la columna `rol` (UO, AD, SAD).
- Herencia de Factura: tabla `factura` con `tipo` y tabla `factura_rectificativa`. No existe `linea_factura`: los datos del concepto van en `factura`, con un solo concepto por factura (a confirmar; si hubiera varios se añade `factura_concepto`).
- Herencia de Tarea: tabla `tarea` con los datos generales y una tabla por subclase cuya clave primaria es también clave foránea de `tarea`.
- Herencia de Movimiento: tabla `movimiento` y tablas propias solo para `envio_comunicacion`, `registro_auditoria` e `importacion_csv`.
- Notificación: una tabla `notificacion` con la columna `tipo` (CORREO, WHATSAPP, USUARIO).
- Agregación N a M: `cliente_contacto` y `cliente_etiqueta`. Composición: la FK es obligatoria.
- `ajustes_sistema` es una tabla clave-valor e incluye series de facturación, festivos y filtros guardados.
- Los estados son `VARCHAR` con `CHECK`. La factura tiene dos ejes: `estado_documento` y `estado_cobro`.
- Los PDF y CSV no se guardan: se generan al emitir o descargar.
- Las claves son `BIGINT` propias. El NIF/CIF del cliente es único, pero no es la clave primaria.
- Las columnas `*_id` que apuntan a `usuario` son FK a `usuario.id`. Fechas de negocio como `DATE`, instantes como `TIMESTAMPTZ`.

## Usuarios y clientes

### usuario

Empleado de TurbineH (clase Usuario con sus tres roles). Una sola tabla con discriminador `rol`.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| email | VARCHAR(254) | no | único. Dominio @turbineh.com. |
| nombre_completo | VARCHAR(120) | no |  |
| password_hash | VARCHAR(255) | sí | Nulo hasta que el empleado fija su contraseña. |
| rol | VARCHAR(3) | no | CHECK IN ('UO','AD','SAD'). |
| estado_cuenta | VARCHAR(11) | no | CHECK IN ('ACTIVO','DESACTIVADO'). |
| umbral_aprobacion | NUMERIC(12,2) | sí | Solo AD y SAD. |
| puede_dar_alta_clientes | BOOLEAN | sí | Solo UO. |
| intentos_fallidos | SMALLINT | no | Por defecto 0. |
| bloqueado_hasta | TIMESTAMPTZ | sí |  |
| fecha_alta | TIMESTAMPTZ | no |  |
| ultimo_acceso | TIMESTAMPTZ | sí |  |

- CHECK: `umbral_aprobacion` solo con rol AD o SAD y `puede_dar_alta_clientes` solo con rol UO.
- Las altas, bajas y desactivaciones quedan en `movimiento` (ALTA_BAJA_USUARIO).

### asignacion_cliente

Historial de quién ha llevado a cada cliente (AsignacionCliente).

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| cliente_id | BIGINT | no | FK a `cliente` |
| usuario_id | BIGINT | no | FK a `usuario` |
| fecha_inicio | DATE | no |  |
| fecha_fin | DATE | sí | Nula si es la vigente. |
| motivo | VARCHAR(16) | no | CHECK IN ('ALTA','REASIGNACION','MIGRACION_MASIVA','DELEGACION'). |

- Índice único parcial: `UNIQUE (cliente_id) WHERE fecha_fin IS NULL AND motivo <> 'DELEGACION'` (un solo responsable vigente por cliente).
- No es una tarea: cada asignación o desasignación genera un `movimiento` (ASIGNACION_DESASIGNACION_CLIENTE).

### cliente

Empresa que contrata el servicio.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK. Clave propia; el NIF/CIF va como único. |
| nif_cif | VARCHAR(40) | no | único. Si coincide con una baja se reactiva la ficha. |
| nombre_comercial | VARCHAR(150) | no |  |
| razon_social | VARCHAR(200) | sí |  |
| domicilio_calle | VARCHAR(150) | sí | Domicilio fiscal en campos separados (propuesta de Sasha, a confirmar). |
| domicilio_numero | VARCHAR(20) | sí |  |
| domicilio_cp | VARCHAR(12) | sí |  |
| domicilio_ciudad | VARCHAR(80) | sí |  |
| pais | CHAR(2) | sí |  |
| moneda | CHAR(3) | no | Por defecto EUR. |
| estado | VARCHAR(20) | no | CHECK IN ('ACTIVO','BAJA_LOGICA','PENDIENTE_ACTIVACION'). |
| fecha_alta | DATE | no |  |
| fecha_baja_efectiva | DATE | sí |  |
| motivo_baja | TEXT | sí |  |
| usuario_responsable_id | BIGINT | sí | FK a `usuario`. Responsable actual (el historial está en `asignacion_cliente`). |

- Las altas y bajas generan un `movimiento` (ALTA_BAJA_CLIENTE).
- Borrado lógico: nunca se borra una fila (BAJA_LOGICA).

### contacto

Persona de contacto (Contacto). Puede estar en más de un cliente.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| nombre_completo | VARCHAR(120) | sí | Se sustituye al anonimizar (CU-30). |
| email | VARCHAR(254) | sí |  |
| telefono | VARCHAR(30) | sí |  |
| rol_contacto | VARCHAR(14) | no | CHECK IN ('FACTURACION','CONTRATACION','IMPAGOS'). |
| consentimiento_email | BOOLEAN | no |  |
| consentimiento_whatsapp | BOOLEAN | no |  |
| idioma | CHAR(2) | no | Para elegir la plantilla (ES/EN). |
| estado | VARCHAR(16) | no | CHECK IN ('ACTIVO','INACTIVO','ANONIMIZADO_RGPD'). |

### cliente_contacto

Agregación N a M entre cliente y contacto.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| cliente_id | BIGINT | no | PK, FK a `cliente` |
| contacto_id | BIGINT | no | PK, FK a `contacto` |

- Regla de negocio: cada cliente debe tener al menos un contacto de facturación activo (se valida en el gestor).

### etiqueta

Clasificación libre de clientes (Etiqueta).

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| nombre | VARCHAR(60) | no | único |
| color_hex | CHAR(7) | sí |  |

### cliente_etiqueta

Agregación N a M entre cliente y etiqueta.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| cliente_id | BIGINT | no | PK, FK a `cliente` |
| etiqueta_id | BIGINT | no | PK, FK a `etiqueta` |

## Suscripcion y tarifas

### tarifa_rango

Tramos de facturación (TarifaRango), solo los edita el SAD.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| rango_facturacion_min | NUMERIC(14,2) | no |  |
| rango_facturacion_max | NUMERIC(14,2) | sí | Nulo en el último tramo. |
| importe_mensual_base | NUMERIC(12,2) | no |  |
| fecha_efecto | DATE | no | Los cambios no alteran facturas históricas. |
| valido_hasta | DATE | sí |  |

### suscripcion

Contrato mensual sin permanencia (Suscripcion). Composición 1:1 con cliente.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| cliente_id | BIGINT | no | FK a `cliente`, único. Composición: obligatoria y única. |
| tarifa_rango_id | BIGINT | no | FK a `tarifa_rango` |
| usuario_comercial_id | BIGINT | sí | FK a `usuario`. Usuario comercial responsable. |
| fecha_inicio | DATE | no |  |
| plan_contratado | VARCHAR(80) | no |  |
| tarifa_acordada | NUMERIC(12,2) | no | Copia del precio acordado; no cambia si cambia el tramo. |
| estado | VARCHAR(18) | no | CHECK IN ('ACTIVA','PAUSADA','PROGRAMADA_FUTURA','CANCELADA'). Se guarda, no se calcula. |
| motivo_pausa | TEXT | sí |  |
| fecha_inicio_pausa | DATE | sí |  |
| fecha_fin_pausa_prevista | DATE | sí |  |
| excepcion_sin_prorrateo | BOOLEAN | no |  |
| plazo_pago_dias | SMALLINT | no | Por defecto 5. |
| ruta_documento_contrato | VARCHAR(255) | sí |  |

### excepcion_comercial

Descuento o condición vigente (ExcepcionComercial). Composición con suscripción.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| suscripcion_id | BIGINT | no | FK a `suscripcion`. Obligatoria (composición). No guarda la solicitud: la relación es por método. |
| tipo | VARCHAR(20) | no | CHECK IN ('DESCUENTO_TEMPORAL','PRIMER_MES_GRATIS','PRECIO_PACTADO','SIN_PRORRATEO'). |
| valor | NUMERIC(12,2) | sí |  |
| vigente_desde | DATE | no |  |
| vigente_hasta | DATE | sí |  |

- Al aplicarse genera un `movimiento` (APLICACION_EXCEPCION).

## Facturacion

### concepto

Catálogo de conceptos facturables (Concepto).

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| nombre | VARCHAR(120) | no |  |
| descripcion_por_defecto | VARCHAR(200) | sí |  |
| precio_base | NUMERIC(12,2) | no |  |
| impuesto_por_defecto | NUMERIC(5,2) | no | 21 por defecto. |
| activo | BOOLEAN | no |  |

### factura

Comprobante fiscal (Factura). Una tabla para los tres tipos; `LineaFactura` ya no existe.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| tipo | VARCHAR(14) | no | CHECK IN ('RECURRENTE','PUNTUAL','RECTIFICATIVA'). |
| cliente_id | BIGINT | no | FK a `cliente` |
| suscripcion_id | BIGINT | sí | FK a `suscripcion`. Obligatoria si es recurrente. |
| usuario_creador_id | BIGINT | no | FK a `usuario`. Quién la crea (puntual o recurrente). |
| concepto_id | BIGINT | no | FK a `concepto`. Un solo concepto por factura (a confirmar). |
| descripcion | VARCHAR(200) | no | Copia del concepto al añadirlo. |
| cantidad | NUMERIC(10,3) | no |  |
| precio_unitario | NUMERIC(12,2) | no | Copia del concepto al añadirlo. |
| porcentaje_descuento | NUMERIC(5,2) | no |  |
| porcentaje_impuesto | NUMERIC(5,2) | no | 21 por defecto. |
| importe_total_linea | NUMERIC(12,2) | no |  |
| numero_serie_completo | VARCHAR(30) | sí | único. Nulo en borrador. |
| serie | VARCHAR(10) | sí | La serie vive en `ajustes_sistema`. |
| numero_secuencial | INTEGER | sí |  |
| estado_documento | VARCHAR(12) | no | CHECK IN ('BORRADOR','EMITIDA','RECTIFICADA','ANULADA'). |
| estado_cobro | VARCHAR(10) | sí | CHECK IN ('PENDIENTE','PAGADA','INCOBRABLE'). Nulo en borrador. |
| fecha_emision | DATE | sí | Fecha de negocio. |
| fecha_vencimiento | DATE | sí |  |
| moneda | CHAR(3) | no |  |
| subtotal | NUMERIC(12,2) | no |  |
| impuestos | NUMERIC(12,2) | no |  |
| total | NUMERIC(12,2) | no |  |
| mes_facturado | CHAR(7) | sí | AAAA-MM. Obligatorio si es recurrente. |
| presupuesto_origen | VARCHAR(100) | sí | Solo puntuales. |
| es_migrada_historica | BOOLEAN | no |  |
| cliente_razon_social | VARCHAR(200) | sí | Datos fiscales congelados al emitir. |
| cliente_nif_cif | VARCHAR(40) | sí |  |
| cliente_domicilio | TEXT | sí |  |
| emisor_razon_social | VARCHAR(200) | sí |  |
| emisor_nif_cif | VARCHAR(40) | sí |  |
| emisor_domicilio | TEXT | sí |  |
| version | INTEGER | no | Control de edición concurrente (PRO-02). |
| creado_en | TIMESTAMPTZ | no |  |

- `UNIQUE (serie, numero_secuencial)`.
- `UNIQUE (suscripcion_id, mes_facturado) WHERE tipo = 'RECURRENTE'`: el proceso del día 1 no duplica facturas.
- `CHECK (tipo <> 'RECURRENTE' OR (suscripcion_id IS NOT NULL AND mes_facturado IS NOT NULL))`.
- `CHECK (estado_documento <> 'BORRADOR' OR (numero_serie_completo IS NULL AND estado_cobro IS NULL))`.
- Una factura emitida no se modifica: se corrige con una rectificativa (RECTIFICADA) o se anula (ANULADA, p. ej. condonación). INCOBRABLE es un estado de cobro, no de documento.
- No se guarda el PDF: se genera al vuelo al emitir o descargar.

### factura_rectificativa

Datos propios de la rectificativa (FacturaRectificativa). Su clave es también FK de factura.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| factura_id | BIGINT | no | PK, FK a `factura` |
| factura_original_id | BIGINT | no | FK a `factura` |
| aprobado_por_id | BIGINT | no | FK a `usuario`. Administrador que la aprobó. |
| motivo_rectificacion | TEXT | no |  |
| importe_ajustado | NUMERIC(12,2) | no |  |

- Cada rectificativa genera un `movimiento` (RECTIFICACION_FACTURA).

### periodo_administrativo

Trimestre contable (PeriodoAdministrativo).

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| anio | SMALLINT | no |  |
| trimestre | SMALLINT | no | CHECK entre 1 y 4. UNIQUE con anio. |
| estado | VARCHAR(22) | no | CHECK IN ('ABIERTO','CERRADO','REABIERTO_EXCEPCIONAL'). |
| fecha_cierre | TIMESTAMPTZ | sí |  |
| autorizacion_superior | BOOLEAN | no |  |

- Quién cerró el periodo se consulta en `movimiento` (CIERRE_PERIODO); no hay FK a usuario.

## Cobros

### movimiento_cobro

Cobro recibido de una pasarela o del banco (MovimientoCobro). No confundir con `movimiento`.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| fuente_pasarela | VARCHAR(10) | no | CHECK IN ('STRIPE','REVOLUT','BANCO'). |
| referencia_transaccion | VARCHAR(100) | no | UNIQUE con fuente_pasarela: evita procesar avisos duplicados. |
| fecha_operacion | TIMESTAMPTZ | no |  |
| importe_recibido | NUMERIC(12,2) | no |  |
| moneda | CHAR(3) | no |  |
| concepto_banco | TEXT | sí |  |

- Registrar un cobro genera un `movimiento` (REGISTRO_COBRO).

### conciliacion_cobro

Vínculo entre un cobro y una factura (Conciliacion). Sin relación con usuario.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| movimiento_cobro_id | BIGINT | no | FK a `movimiento_cobro`, único. 1:1 con el cobro. |
| factura_id | BIGINT | no | FK a `factura` |
| tipo | VARCHAR(10) | no | CHECK IN ('AUTOMATICA','MANUAL'). |
| discrepancia_importe | NUMERIC(12,2) | no | 0 si cuadra. |
| fecha | TIMESTAMPTZ | no |  |
| estado | VARCHAR(10) | no | CHECK IN ('CONCILIADO','REVERTIDO'). Sin CON_DIFERENCIA: la diferencia va en discrepancia_importe. |

- Regla: no se concilia contra facturas anuladas ni ya pagadas (se valida en el gestor).

## Tareas

### tarea

Datos generales de toda tarea (Tarea, clase abstracta).

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| ejecutor | VARCHAR(7) | no | CHECK IN ('USUARIO','SISTEMA'). |
| estado | VARCHAR(10) | no | CHECK IN ('PENDIENTE','ATENDIDA','APLAZADA'). |
| fecha_creacion | TIMESTAMPTZ | no |  |
| fecha_aplazada_fin | DATE | sí | Fecha aplazada o de fin. |

- Sin tipo, usuario, cliente ni factura: el tipo lo da la tabla de la subclase y los demás datos viven en ellas.
- Cada subclase usa como clave primaria la clave de `tarea` (mismo patrón que `factura_rectificativa`).

### delegacion_temporal

Tarea de sistema (DelegacionTemporal). La caducidad la ejecuta el Reloj.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| tarea_id | BIGINT | no | PK, FK a `tarea` |
| cliente_id | BIGINT | no | FK a `cliente` |
| usuario_origen_id | BIGINT | no | FK a `usuario` |
| usuario_delegado_id | BIGINT | no | FK a `usuario` |
| fecha_inicio | TIMESTAMPTZ | no |  |
| estado | VARCHAR(10) | no | CHECK IN ('ACTIVA','CADUCADA','REVOCADA'). |

- La fecha de caducidad es `tarea.fecha_aplazada_fin`.

### revision_anual_tarifa

Tarea de sistema (RevisionAnual): revisión del tramo de un cliente.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| tarea_id | BIGINT | no | PK, FK a `tarea` |
| cliente_id | BIGINT | no | FK a `cliente` |
| tarifa_calculada_id | BIGINT | no | FK a `tarifa_rango` |
| tarifa_aprobada_id | BIGINT | sí | FK a `tarifa_rango` |
| aprobado_por_id | BIGINT | sí | FK a `usuario` |
| facturacion_verificada | NUMERIC(14,2) | sí |  |
| estado | VARCHAR(10) | no | CHECK IN ('PENDIENTE','APROBADA','RECHAZADA','MANTENIDA'). |
| fecha_revision | TIMESTAMPTZ | sí |  |

### solicitud_aprobacion

Tarea de usuario (SolicitudAprobacion): operación que requiere validación humana.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| tarea_id | BIGINT | no | PK, FK a `tarea` |
| cliente_id | BIGINT | no | FK a `cliente` |
| factura_id | BIGINT | sí | FK a `factura` |
| solicitante_id | BIGINT | no | FK a `usuario` |
| aprobador_id | BIGINT | sí | FK a `usuario` |
| tipo_operacion | VARCHAR(22) | no | CHECK IN ('DESCUENTO_TEMPORAL','PRIMER_MES_GRATIS','PRECIO_PACTADO','RECTIFICATIVA_ALTA','CONDONACION_DEUDA','FACTURA_PUNTUAL_ALTA'). Pendiente: valor DECLARACION_INCOBRABLE. |
| valor_solicitado | NUMERIC(12,2) | sí |  |
| motivo | TEXT | no |  |
| resolucion | VARCHAR(9) | sí | CHECK IN ('APROBADA','RECHAZADA'). Nula mientras está pendiente. |
| fecha_resolucion | TIMESTAMPTZ | sí |  |

- `CHECK (solicitante_id <> aprobador_id)`: el solicitante y el aprobador no pueden ser la misma persona.
- Crearla genera un `movimiento` (REGISTRO_TAREA).

### disputa_deuda

Tarea de usuario (DisputaDeuda): reclamación que congela los recordatorios.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| tarea_id | BIGINT | no | PK, FK a `tarea` |
| factura_id | BIGINT | no | FK a `factura` |
| cliente_id | BIGINT | no | FK a `cliente` |
| empleado_id | BIGINT | no | FK a `usuario` |
| motivo_reclamacion | TEXT | no |  |
| fecha_revision_prevista | DATE | no |  |
| recordatorios_congelados | BOOLEAN | no |  |

- La fecha de apertura es `tarea.fecha_creacion`.

### incidencia_facturacion

Tarea de usuario (IncidenciaFacturacion).

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| tarea_id | BIGINT | no | PK, FK a `tarea` |
| cliente_id | BIGINT | no | FK a `cliente` |
| factura_id | BIGINT | sí | FK a `factura`. Opcional. |
| usuario_id | BIGINT | no | FK a `usuario`. Responsable. |
| tipo | VARCHAR(18) | no | CHECK IN ('ENVIO_REBOTADO','NIF_INCORRECTO','PAGO_NO_LOCALIZADO','AVISO_BAJA_API'). |
| prioridad | VARCHAR(5) | no | CHECK IN ('ALTA','MEDIA','BAJA'). |
| pausar_recordatorios | BOOLEAN | no |  |

### comentario_incidencia

Comentario de una incidencia (ComentarioIncidencia). Composición.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| incidencia_id | BIGINT | no | FK a `incidencia_facturacion`. Obligatoria (composición). |
| nivel_visibilidad | VARCHAR(14) | no | CHECK IN ('INTERNO_GESTOR','GLOBAL_ADMIN'). A confirmar si se conserva. |
| mensaje | TEXT | no |  |
| fecha_hora | TIMESTAMPTZ | no |  |

- Sin autor: es el usuario de la incidencia.

## Movimientos y notificaciones

### notificacion

Jerarquía de comunicaciones (Notificacion con Correo, WhatsApp y NotificacionUsuario).

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| tipo | VARCHAR(8) | no | CHECK IN ('CORREO','WHATSAPP','USUARIO'). |
| asunto | VARCHAR(200) | no |  |
| texto | TEXT | no |  |
| fecha | TIMESTAMPTZ | no |  |
| usuario_destinatario_id | BIGINT | sí | FK a `usuario`. Obligatorio si tipo = USUARIO. |
| leida | BOOLEAN | sí | Solo para tipo = USUARIO. |

- `CHECK ((tipo = 'USUARIO') = (usuario_destinatario_id IS NOT NULL))` y `leida` solo con tipo USUARIO.
- La bandeja de entrada del empleado muestra solo las de tipo USUARIO.

### movimiento

Registro de todo lo que ocurre en el sistema (Movimiento, clase abstracta; antes EjecucionProceso).

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| tipo | VARCHAR(34) | no | CHECK IN (los 13 tipos de abajo). |
| fecha_hora | TIMESTAMPTZ | no |  |
| usuario_id | BIGINT | sí | FK a `usuario`. Nulo si lo origina el sistema. |
| entidad_afectada | VARCHAR(40) | sí |  |
| id_entidad_afectada | BIGINT | sí |  |
| detalle | TEXT | sí |  |

- Tipos: ENVIO_COMUNICACION, REGISTRO_AUDITORIA, IMPORTACION_CSV, CIERRE_PERIODO, ALTA_BAJA_CLIENTE, ALTA_BAJA_USUARIO, ASIGNACION_DESASIGNACION_CLIENTE, GENERACION_FACTURA, RECTIFICACION_FACTURA, GENERACION_INFORME, APLICACION_EXCEPCION, REGISTRO_COBRO, REGISTRO_TAREA.
- Solo tienen tabla propia las subclases con atributos específicos; las demás viven solo aquí, distinguidas por `tipo`.
- Solo INSERT y SELECT para la aplicación: no se modifica ni se borra (AUD-01).
- Pendiente: dónde se guarda el resumen de los procesos automáticos (un tipo EJECUCION_PROCESO como decimocuarta subclase).

### envio_comunicacion

Subclase EnvioComunicacion: envío externo a un cliente.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| movimiento_id | BIGINT | no | PK, FK a `movimiento` |
| notificacion_id | BIGINT | no | FK a `notificacion`. Correo o WhatsApp; el canal lo da su tipo. |
| factura_id | BIGINT | sí | FK a `factura` |
| contacto_id | BIGINT | no | FK a `contacto` |
| tipo_mensaje | VARCHAR(12) | no | CHECK IN ('FACTURA','RECORDATORIO','AVISO_IMPAGO'). |
| estado | VARCHAR(16) | no | CHECK IN ('EN_COLA','ENVIADO','FALLIDO_REBOTADO','REINTENTADO'). |
| es_reintento | BOOLEAN | no |  |
| numero_intento | SMALLINT | no |  |

### registro_auditoria

Subclase RegistroAuditoria: bitácora append-only.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| movimiento_id | BIGINT | no | PK, FK a `movimiento` |
| tabla_afectada | VARCHAR(40) | no |  |
| id_registro_afectado | VARCHAR(40) | no |  |
| tipo_accion | VARCHAR(14) | no | CHECK IN ('ALTA','MODIFICACION','EMISION','CONCILIACION','REASIGNACION','CIERRE_PERIODO'). |
| valor_anterior | JSONB | sí |  |
| valor_nuevo | JSONB | sí |  |

- El inicio y cierre de sesión no se registran aquí: van a un log de aplicación fuera del ER.

### importacion_csv

Subclase ImportacionCSV: traza de cada carga masiva.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| movimiento_id | BIGINT | no | PK, FK a `movimiento` |
| nombre_archivo | VARCHAR(255) | no |  |
| filas_totales | INTEGER | no |  |
| filas_correctas | INTEGER | no |  |
| filas_con_error | INTEGER | no |  |
| estado | VARCHAR(10) | no | CHECK IN ('EXITOSA','PARCIAL','FALLIDA'). |

## Sistema

### informe

Informe generado (Informe). No se guarda el archivo: se genera al descargar.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| usuario_id | BIGINT | no | FK a `usuario`. Usuario generador. |
| cliente_id | BIGINT | sí | FK a `cliente`. Opcional. |
| tipo | VARCHAR(40) | no |  |
| parametros | JSONB | sí |  |
| fecha_generacion | TIMESTAMPTZ | no |  |
| formato | VARCHAR(3) | no | CHECK IN ('PDF','CSV'). |

- Genera un `movimiento` (GENERACION_INFORME).

### ajustes_sistema

Parámetros globales (AjustesSistema, singleton): tabla clave-valor.

| Columna | Tipo | Nulo | Restricciones y notas |
|---|---|---|---|
| id | BIGINT | no | PK |
| clave | VARCHAR(60) | no |  |
| valor | TEXT | no |  |
| tipo | VARCHAR(20) | no | Tipo de dato o grupo: PARAMETRO, SERIE, FESTIVO o FILTRO. |
| fecha_efecto | DATE | no | Los cambios solo afectan a lo nuevo. |
| estado | VARCHAR(10) | no | CHECK IN ('PROPUESTO','VIGENTE','RECHAZADO'). |

- `UNIQUE (clave) WHERE estado = 'VIGENTE'`.
- Integra las series de facturación (prefijo, año, último número, activa), la lista de festivos y los filtros guardados.
- El siguiente número de serie se obtiene de forma atómica (`SELECT ... FOR UPDATE`) para que dos emisiones no repitan número.

## Puntos pendientes de confirmar

1. Un solo concepto por factura o una lista (en ese caso, tabla `factura_concepto`).
2. Domicilio fiscal del cliente: campos separados (como está) o texto único (como dice la guía).
3. Estado de la factura: dos ejes (como está) o un único campo como en la guía.
4. Estado del cliente: se guarda (como dice la guía) o se deriva de la suscripción.
5. Valor `DECLARACION_INCOBRABLE` en `solicitud_aprobacion.tipo_operacion` (cerrado por la guía).
6. Dónde se guarda el resumen de los procesos automáticos (decimocuarta subclase de movimiento).
7. Nivel de visibilidad de los comentarios de incidencia.
8. Las tablas propias de las subclases de Tarea y Movimiento (propuesta de la guía, por validar).
