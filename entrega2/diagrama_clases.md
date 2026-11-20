# ADA TURBINEH – Guía para diagramas de clases y ER

## 1. Decisiones de diseño (resumen para el cliente)

| Decisión | Por qué |
| --- | --- |
| Capas: Interfaz, Gestores y Datos | Cada parte tiene una sola responsabilidad y se puede probar y cambiar por separado. |
| Un gestor por tema de negocio (11) | Agrupamos en casos de uso del mismo tema: acceso, clientes, suscripciones, aprobaciones, facturas, cobros, trabajo, informes, sistema, movimientos y elementos visuales. |
| Cinco superclases visuales con subtipos por rol (Interfaz y Bandeja tienen versión Usuario, Admin y Superadmin) | La seguridad no depende solo de tener clases separadas, sino de comprobar permisos en cada operación (un usuario operativo solo toca sus clientes). |
| Un único registro de auditoría append-only (RegistroAuditoria, subclase de Movimiento) | Se consulta con filtros por tabla, usuario y fecha. |
| Movimiento es una clase concreta con un enumerado de tipo | Solo RegistroAuditoria, EnvioComunicacion e ImportacionCSV son subclases, porque tienen campos propios; el resto de eventos comparten los mismos datos. |
| Una factura contiene una lista de conceptos | Una misma factura puede incluir la cuota y servicios puntuales. |
| Una factura emitida nunca se modifica | Se corrige con una rectificativa vinculada. |
| Las facturas recurrentes no se duplican | Una suscripción solo puede tener una factura por mes. |
| Los PDF no se guardan | En base de datos solo se guardan los datos; el PDF se genera al emitir o descargar y la descarga se comunica directamente con el sistema operativo. |
| Los filtros viajan como objeto hasta la base de datos | Evita cargar todos los datos en memoria (patrón Specification). |
| El Reloj lanza los procesos automáticos | Facturación del día 1, recordatorios, caducidad de delegaciones. Es un Singleton que emite señales periódicas de revisión a los gestores. |

## 2. Notación

- Visibilidad UML: `-` privado (atributos), `+` público (métodos).
- Estereotipos: <\<entity>> para dominio, <\<control>> para gestores, <\<boundary>> para pantallas, <\<singleton>> para Reloj y AjustesSistema, <\<static>> para la clase estática de validación.
- Composición (rombo relleno): la parte no existe sin el todo.
- Agregación (rombo vacío): el todo contiene partes que pueden existir por sí solas.
- Dependencia (flecha discontinua): una clase usa a otra.
- Herencia (flecha con triángulo vacío): Usuario y sus roles, Factura y sus tipos, Tarea y sus subclases, Movimiento y sus subclases, Notificación y sus subtipos.
- Los atributos cuyo tipo es otra clase del modelo expresan la relación entre ambas; esas clases mantienen su propia tabla en el ER.

## 3. Clases de dominio (entidades)

### 3.1 Paquete Usuarios y asignaciones

**Usuario (abstracta)** Empleado de TurbineH. Sin autorregistro. Su identidad y credenciales son comunes a los tres roles.

- Atributos: -idUsuario: Entero, -emailCorporativo: Texto (único, dominio @turbineh.com), -passwordHash: Texto, -nombreCompleto: Texto, -estadoCuenta: Enum{Activo, Desactivado}, -fechaAlta: FechaHora, -ultimoAcceso: FechaHora.
- Métodos: +autenticar(password): Booleano, +cambiarPassword(nueva), +desactivar(), +reactivar(), +puedeAccederA(cliente): Booleano.
- Subclases:
  - UsuarioOperativo (UO): -carteraClientes: Lista\<Cliente>, -puedeDarAltaClientes: Booleano. Solo accede a sus asignados.
  - Administrador (AD): -umbralAprobacion: Decimal. Acceso global; aprueba hasta su umbral.
  - SuperAdministrador (SAD): hereda de Administrador. Gestiona ajustes del sistema, series, tarifas y aprobaciones por encima del umbral.
- Las altas, bajas y desactivaciones de usuario generan un Movimiento de tipo AltaBajaUsuario.

**DelegacionTemporal (subclase de Tarea, ejecutor SISTEMA)** Cesión temporal de un cliente a otro usuario. La caducidad la ejecuta el sistema (Reloj).

- Atributos propios: -idDelegacion: Entero, -fechaInicio: FechaHora, -estado: Enum{Activa, Caducada, Revocada}, -cliente: Cliente, -usuarioOrigen: Usuario, -usuarioDelegado: Usuario. La fecha de caducidad es el atributo fechaFin heredado de Tarea.
- Métodos: +activar(), +caducar(), +revocar(), +estaVigente(fecha): Booleano.

**AsignacionCliente** Historial de quién ha llevado a cada cliente. Permite saber el responsable en cualquier fecha sin perder la trazabilidad. No es una Tarea porque se ejecuta al instante.

- Atributos: -idAsignacion: Entero, -fechaInicio: Fecha, -fechaFin: Fecha (nula si es la vigente), -motivo: Enum{Alta, Reasignacion, MigracionMasiva, Delegacion}.
- Relaciones: Cliente (1..N asignaciones) y Usuario (1..N asignaciones).
- Cada asignación o desasignación genera un Movimiento de tipo AsignacionDesasignacion.

### 3.2 Paquete Clientes

**Cliente** Empresa que contrata el servicio.

- Atributos: -nifCif: Texto (identificador), -nombreComercial: Texto, -razonSocial: Texto, -domicilioFiscal: Texto, -pais: Texto, -moneda: Texto (por defecto EUR), -estado: Enum{Activo, BajaLogica, PendienteActivacion}, -fechaAlta: Fecha, -fechaBajaEfectiva: Fecha, -motivoBaja: Texto.
- Atributos de relación:
  - -contactos: Lista\<Contacto> → agregación (un contacto puede pertenecer a dos empresas).
  - -etiquetas: Lista\<Etiqueta> → agregación (las etiquetas existen independientemente del cliente).
  - -suscripcion: Suscripcion → composición.
  - -revisionAnual: RevisionAnual → asociación (RevisionAnual es subclase de Tarea).
  - -facturas: Lista\<Factura> → asociación.
- Métodos: +tieneDatosFiscalesCompletos(): Booleano, +darDeBaja(motivo, fecha), +reactivar(), +añadirEtiqueta(etiqueta), +quitarEtiqueta(etiqueta), +tieneContactoFacturacion(): Booleano.
- Reglas: el NIF/CIF es único. Si coincide con una baja, se reactiva la ficha. El alta y la baja generan un Movimiento de tipo AltaBajaCliente.

**Contacto** Persona de contacto de una empresa cliente.

- Atributos: -idContacto: Entero, -nombreCompleto: Texto, -email: Texto, -telefono: Texto, -rolContacto: Enum{Facturacion, Contratacion, Impagos}, -consentimientoEmail: Booleano, -consentimientoWhatsapp: Booleano, -estado: Enum{Activo, Inactivo, AnonimizadoRGPD}.
- Métodos: +desactivar(), +anonimizar() (sustituye datos personales por valores genéricos, CU-30), +admiteCanal(canal): Booleano.
- Relación con Cliente: agregación, N a M. Cada cliente tiene como mínimo un contacto de facturación activo.

**Etiqueta** Clasificación libre de clientes ("riesgo de impago", "estratégico").

- Atributos: -idEtiqueta: Entero, -nombre: Texto (único), -colorHex: Texto.
- Relación con Cliente: agregación, muchos a muchos.

### 3.3 Paquete Suscripciones y tarifas

**Suscripcion** Contrato mensual sin permanencia, genera las facturas recurrentes.

- Atributos: -idSuscripcion: Entero, -fechaInicio: Fecha, -planContratado: Texto, -tarifaAcordada: Decimal, -estado: Enum{Activa, Pausada, ProgramadaFutura, Cancelada}, -motivoPausa: Texto, -fechaInicioPausa: Fecha, -fechaFinPausaPrevista: Fecha, -excepcionSinProrrateo: Booleano, -plazoPagoDias: Entero, -rutaDocumentoContrato: Texto.
- Atributo de relación: -excepcionComercial: ExcepcionComercial → composición.
- Métodos: +calcularProrrateoPrimerMes(): Decimal, +pausar(motivo, inicio, fin), +reactivar(), +cancelar(), +debeFacturarse(mes): Booleano, +aplicarNuevaTarifa(tarifa, fechaEfecto).
- Relaciones: composición con Cliente (1:1); N:1 con TarifaRango; N:1 con el usuario comercial responsable.
- Fórmula del prorrateo: importe mensual × días restantes (incluido el día de alta) ÷ días naturales del mes.

**TarifaRango** Tabla maestra de tramos de facturación, editable solo por el SuperAdministrador.

- Atributos: -idTarifa: Entero, -rangoFacturacionMin: Decimal, -rangoFacturacionMax: Decimal, -importeMensualBase: Decimal, -fechaEfecto: Fecha, -validoHasta: Fecha.
- Métodos: +aplicaA(facturacionAnual): Booleano, +estaVigente(fecha): Booleano.

**RevisionAnual (subclase de Tarea, ejecutor SISTEMA)** Revisión del tramo de un cliente según su facturación.

- Atributos propios: -idRevision: Entero, -facturacionVerificada: Decimal, -estado: Enum{Pendiente, Aprobada, Rechazada, Mantenida}, -fechaRevision: FechaHora, -cliente: Cliente, -tarifaRango: TarifaRango (tarifa calculada), -tarifaAprobada: TarifaRango (opcional), -aprobador: Usuario (opcional).
- Métodos: +calcularTarifaPropuesta(tarifas): TarifaRango, +aprobar(usuario, tarifa), +rechazar(usuario), +mantener().

**SolicitudAprobacion (subclase de Tarea, ejecutor USUARIO)** Cualquier operación que requiera validación humana.

- Atributos propios: -idSolicitud: Entero, -tipoOperacion: Enum{DescuentoTemporal, PrimerMesGratis, PrecioPactado, RectificativaAlta, CondonacionDeuda, FacturaPuntualAlta}, -valorSolicitado: Decimal, -motivo: Texto, -resolucion: Enum{Aprobada, Rechazada}, -fechaResolucion: FechaHora, -cliente: Cliente, -factura: Factura (opcional), -solicitante: Usuario, -aprobador: Usuario (opcional). El estado pendiente/atendida lo hereda de Tarea.
- Métodos: +requiereAprobacion(umbral): Booleano, +aprobar(aprobador), +rechazar(aprobador, motivo).
- Regla: el solicitante y el aprobador no pueden ser la misma persona.
- Registrar una solicitud genera un Movimiento de tipo RegistroTarea.

**ExcepcionComercial** Resultado vigente de una solicitud aprobada: un descuento o condición que el sistema aplica al facturar.

- Atributos: -idExcepcion: Entero, -tipo: Enum{DescuentoTemporal, PrimerMesGratis, PrecioPactado, SinProrrateo}, -valor: Decimal, -vigenteDesde: Fecha, -vigenteHasta: Fecha.
- Métodos: +estaVigente(fecha): Booleano, +aplicar(factura) (modifica los importes de la factura recurrente), +solicitarAprobacion(cliente): SolicitudAprobacion, +recibirResolucion(solicitud).
- Relación con Suscripcion: composición.
- Relación con SolicitudAprobacion: por método. +solicitarAprobacion crea la solicitud a partir de los datos del cliente y de la excepción, y la operación queda en pausa hasta que la excepción se aprueba o se revierte (+recibirResolucion).
- Aplicar una excepción genera un Movimiento de tipo AplicacionExcepcion.

### 3.4 Paquete Facturación

**Factura (abstracta)** Comprobante fiscal.

- Atributos: -idFactura: Entero, -numeroSerieCompleto: Texto (único, nulo en borrador), -serie: Texto, -numeroSecuencial: Entero, -estado: Enum{Borrador, Emitida, Pagada, Rectificada, Incobrable, Anulada}, -fechaEmision: Fecha (fecha de negocio, no hora del servidor), -fechaVencimiento: Fecha, -moneda: Texto, -subtotal: Decimal, -impuestos: Decimal, -total: Decimal, -esMigradaHistorica: Booleano.
- Atributos de relación: -conceptos: Lista\<Concepto>, -usuarioCreador: Usuario (en facturas puntuales y recurrentes).
- Métodos: +recalcularTotales(), +esEditable(): Booleano (solo en Borrador), +emitir(numero, fecha), +marcarPagada(), +marcarRectificada(), +marcarIncobrable().
- Subclases:
  - FacturaRecurrente: -mesFacturado: Fecha. La pareja (suscripción, mes) es única, para que no se duplique si el proceso se repite. Relación N:1 con Suscripcion.
  - FacturaPuntual: -presupuestoOrigen: Texto (opcional). La crea un empleado, nace en Borrador.
  - FacturaRectificativa: -motivoRectificacion: Texto, -importeAjustado: Decimal. Relación N:1 con la factura original y con el administrador que la aprobó.
- Cada emisión genera un Movimiento de tipo GeneracionFactura y cada rectificativa uno de tipo RectificacionFactura.
- La gestión de los conceptos de una factura (añadir, quitar) la realiza GestorFacturas.

**Concepto** Catálogo de conceptos facturables. Una factura contiene una lista de conceptos.

- Atributos: -idConcepto: Entero, -nombre: Texto, -descripcionPorDefecto: Texto, -precioBase: Decimal, -impuestoPorDefecto: Decimal, -activo: Booleano.
- Al añadir un concepto a una factura se copian su nombre y su precio en ese momento; si luego cambia el catálogo, la factura no cambia.

**PeriodoAdministrativo** Trimestre contable abierto o cerrado.

- Atributos: -idPeriodo: Entero, -anio: Entero, -trimestre: Entero (1 a 4), -estado: Enum{Abierto, Cerrado, ReabiertoExcepcional}, -fechaCierre: FechaHora, -autorizacionSuperior: Booleano.
- Métodos: +cerrar(usuario), +reabrir(usuario, autorizacion), +contieneFecha(fecha): Booleano, +permiteEmitir(fecha): Booleano.
- Al cerrar un periodo se crea un Movimiento de tipo CierrePeriodo que guarda el usuario y el periodo que se cierra.
- Para mantener la inalterabilidad, en un periodo reabierto solo se permite emitir rectificativas, no editar la factura original.

### 3.5 Paquete Cobros y conciliación

**MovimientoCobro** Evento de cobro recibido de una pasarela o banco. No es la clase Movimiento (registro de acciones): registrar un cobro genera un Movimiento de tipo RegistroCobro que referencia a este MovimientoCobro.

- Atributos: -idMovimiento: Entero, -fuentePasarela: Enum{Stripe, Revolut, Banco}, -referenciaTransaccion: Texto (única; evita procesar avisos duplicados), -fechaOperacion: FechaHora, -importeRecibido: Decimal, -moneda: Texto, -conceptoBanco: Texto.
- Métodos: +esDuplicado(): Booleano, +coincideCon(factura): Booleano.

**Conciliacion** Vínculo entre un cobro y una factura.

- Atributos: -idConciliacion: Entero, -tipo: Enum{Automatica, Manual}, -discrepanciaImporte: Decimal, -fecha: FechaHora, -estado: Enum{Conciliado, Revertido}, -cobro: MovimientoCobro, -factura: Factura.
- Métodos: +tieneDiscrepancia(): Booleano, +revertir(motivo).
- Reglas: no se concilia contra facturas anuladas ni ya pagadas. El usuario responsable se obtiene de la factura asociada.

**DisputaDeuda (subclase de Tarea, ejecutor USUARIO)** Reclamación del cliente por el importe de una factura recibida. Congela los recordatorios de esa factura.

- Atributos propios: -idDisputa: Entero, -motivoReclamacion: Texto, -fechaRevisionPrevista: Fecha, -recordatoriosCongelados: Booleano, -factura: Factura, -cliente: Cliente, -empleado: Usuario. La fecha de apertura es la fechaCreacion heredada y el estado lo hereda de Tarea.
- Métodos: +cerrar(usuario), +congelarRecordatorios().
- Registrarla genera un Movimiento de tipo RegistroTarea.

### 3.6 Paquete Seguimiento y sistema

**Tarea (abstracta)** Pendiente de la pantalla de trabajo diario.

- Atributos: -idTarea: Entero, -estado: Enum{Pendiente, Atendida, ...}, -fechaCreacion: FechaHora, -fechaAplazada / fechaFin: Fecha, -ejecutor: Enum{USUARIO, SISTEMA} (quién ejecuta o atiende la tarea).
- Métodos: +atender(usuario), +aplazar(fecha).
- Subclases:

| Ejecutor | Subclases |
| --- | --- |
| SISTEMA | DelegacionTemporal, RevisionAnual |
| USUARIO | DisputaDeuda, IncidenciaFacturacion, SolicitudAprobacion |

Cada subclase guarda sus propios cliente, factura y usuario.

**IncidenciaFacturacion (subclase de Tarea, ejecutor USUARIO)** Problemas de entrega (factura no entregada al cliente), pago no localizado, NIF incorrecto o aviso de baja por API.

- Atributos propios: -idIncidencia: Entero, -tipo: Enum{EnvioRebotado, NifIncorrecto, PagoNoLocalizado, AvisoBajaApi}, -prioridad: Enum{Alta, Media, Baja}, -pausarRecordatorios: Booleano, -cliente: Cliente, -factura: Factura (opcional, puede ser nula), -usuario: Usuario, -comentarios: Lista\<ComentarioIncidencia>. El estado y la fecha de apertura se heredan de Tarea.
- Métodos: +asignar(usuario), +cambiarEstado(estado), +resolver(usuario).
- Registrarla genera un Movimiento de tipo RegistroTarea.

**ComentarioIncidencia** Comentario asociado a una incidencia.

- Atributos: -idComentario: Entero, -nivelVisibilidad: Enum{InternoGestor, GlobalAdmin}, -mensaje: Texto, -fechaHora: FechaHora.
- Relación: composición con IncidenciaFacturacion (atributo -comentarios). El autor es el usuario de la incidencia.

**Notificación (interfaz o clase abstracta)** Jerarquía de las comunicaciones del sistema.

- Atributos: -idNotificacion: Entero, -asunto: Texto, -texto: Texto, -fecha: FechaHora, -usuarioDestinatario: Usuario (cuando aplica).
- Subtipos: Correo, WhatsApp y NotificacionUsuario (avisos internos de la bandeja de entrada de los empleados; tiene -leida: Booleano y +marcarLeida()).

**Movimiento** Registro de todo lo que ocurre en el sistema. Es una clase concreta; el tipo de evento lo indica un enumerado.

- Atributos: -idMovimiento: Entero, -fechaHora: FechaHora, -usuario: Usuario (nulo en movimientos del sistema), -tipoMovimiento: Enum{AltaBajaCliente, AltaBajaUsuario, AsignacionDesasignacion, GeneracionFactura, RectificacionFactura, GeneracionInforme, AplicacionExcepcion, RegistroCobro, RegistroTarea, CierrePeriodo}, -detalle: Texto, -entidadAfectada: Texto, -idEntidadAfectada: Texto.
- Constructores: +Movimiento(tipo, usuario, detalle), +Movimiento(tipo, usuario, detalle, entidadAfectada, idEntidadAfectada).
- Tipos de movimiento:

| Tipo | Se genera cuando | Entidad afectada |
| --- | --- | --- |
| AltaBajaCliente | se da de alta o de baja un cliente | Cliente |
| AltaBajaUsuario | se da de alta o de baja un empleado | Usuario |
| AsignacionDesasignacion | se asigna o desasigna un cliente a un usuario | AsignacionCliente |
| GeneracionFactura | se emite una factura | Factura |
| RectificacionFactura | se emite una rectificativa | FacturaRectificativa |
| GeneracionInforme | se genera un informe | Informe |
| AplicacionExcepcion | se aplica una excepción comercial | ExcepcionComercial |
| RegistroCobro | se registra un cobro | MovimientoCobro |
| RegistroTarea | se registra una DisputaDeuda, una IncidenciaFacturacion o una SolicitudAprobacion | Tarea |
| CierrePeriodo | se cierra un periodo administrativo (guarda usuario y periodo) | PeriodoAdministrativo |

- Subclases con campos propios, que no usan el enumerado (su tipo lo define la clase):
  - RegistroAuditoria: bitácora append-only, no se modifica ni se borra. Incluye el atributo usuario que realiza la acción y métodos getter para consultar sus datos. Atributos: -tablaAfectada: Texto, -idRegistroAfectado: Texto, -tipoAccion: Enum{Alta, Modificacion, Emision, Conciliacion, Reasignacion, CierrePeriodo}, -valorAnterior: JSON, -valorNuevo: JSON. Solo se crea mediante GestorMovimientos.registrarAuditoria().
  - EnvioComunicacion: registra el envío externo a clientes. Atributos: -notificacion: Notificación (Correo o WhatsApp), -factura: Factura, -contacto: Contacto, -tipoMensaje: Enum{Factura, Recordatorio, AvisoImpago}, -estado: Enum{EnCola, Enviado, FallidoRebotado, Reintentado}, -esReintento: Booleano, -numeroIntento: Entero. El canal lo da el tipo de notificación. Métodos: +marcarEnviado(), +marcarRebotado(), +reintentar().
  - ImportacionCSV: traza cada carga masiva de clientes. Atributos: -nombreArchivo: Texto, -filasTotales: Entero, -filasCorrectas: Entero, -filasConError: Entero, -estado: Texto.

**AjustesSistema (singleton)** Parámetros globales del sistema que solo edita el SuperAdministrador y que requieren validación. Los cambios solo afectan a lo nuevo, nunca a lo ya emitido. Su gestor es GestorSistema.

- Atributos: -clave: Texto, -valor: Texto, -tipo: Texto, -fechaEfecto: Fecha, -estado: Enum{Propuesto, Vigente, Rechazado}.
- Contenido: series de facturación (prefijo, año, último número, activa), lista de festivos (fechas en las que se pospone una notificación si cae en día inhábil) y lista de filtros guardados.
- Parámetros: IVA por defecto, plazo de vencimiento (5 días), días hasta recordatorio (3), días entre avisos de impago (2-3), tiempo de inactividad, límite de clientes por usuario, umbral de aprobación, umbral de factura puntual alta, margen del patrón de importe, antigüedad máxima para reabrir un periodo.
- Métodos: +obtener(clave): Texto, +proponerCambio(clave, valor, usuario), +aplicarCambio(clave), +esLaborable(fecha): Booleano, +siguienteNumero(serie): Entero (se ejecuta de forma atómica, para que dos emisiones simultáneas no repitan número).

**Informe** Informe generado y descargable.

- Atributos: -idInforme: Entero, -tipo: Texto, -parametros: JSON, -fechaGeneracion: FechaHora, -formato: Enum{PDF, CSV}.
- Relación: usuario generador; Cliente (opcional). Genera un Movimiento de tipo GeneracionInforme.
- El archivo no se guarda: el PDF o CSV se genera al vuelo al descargar.

## 4. Clases de control e interfaz (no van al ER)

### 4.1 Repositorios y filtros

- Repositorio\<T> <\<interface>>: +guardar(t), +buscarPorId(id), +buscar(filtro: Filtro\<T>), +contar(filtro), +eliminar(t) (solo permitido para borradores).
  - Está implementada por 8 repositorios: RepositorioClientes, RepositorioFacturas, RepositorioSuscripciones, RepositorioUsuarios, RepositorioCobros, RepositorioTareas, RepositorioInformes y RepositorioMovimientos.
  - Métodos propios: RepositorioClientes.buscarPorNif(nif), RepositorioFacturas.buscarRecurrente(suscripcion, mes) (comprueba que no existe ya la factura del mes).
- Filtro\<T> <\<interface>> (patrón Specification): +criterios(): Mapa, +orden(): Texto. Los filtros predefinidos están en código; su lista se gestiona y recupera de base de datos mediante AjustesSistema y GestorSistema.
  - Implementaciones: FiltroFacturas (cliente, estado, fechas, importe), FiltroClientes (nombre, estado, etiqueta, responsable), FiltroMovimientos (tipo, usuario, fechas; cobros, conciliaciones, etc.), FiltroTareas.
  - Los gestores reciben el filtro y se lo pasan al repositorio, que lo traduce a la consulta.

### 4.2 Gestores (un gestor por tema de negocio)

Todos son <\<control>>. Antes de cada operación llaman a AutenticadorValidador.comprobarPermiso() y después a GestorMovimientos.registrarAuditoria().

| Gestor | Métodos principales | CU |
| --- | --- | --- |
| GestorAcceso | iniciarSesion(email, pass), cerrarSesion(), recuperarPassword(email), cerrarPorInactividad(), crearUsuario(datos, rol), desactivarUsuario(id, nuevoResponsable), asignarCliente(cliente, usuario), quitarAsignacion(cliente), delegarTemporal(cliente, usuario, fechaFin), devolverDelegacionesCaducadas(), migrarCartera(origen, destino) | 01 a 03.5 |
| GestorClientes | registrarCliente(datos, contactos), reactivarCliente(nif), editarFicha(cliente, cambios), añadirContacto(), desactivarContacto(), etiquetar(), darDeBaja(cliente, motivo), anonimizarContacto(contacto), importarCSV(rutaArchivo: Texto) | 04 a 07, 30 |
| GestorSuscripciones | crearSuscripcion(cliente, datos), pausar(), reactivar(), calcularProrrateo(), listarRevisionesPendientes(), proponerRevision(cliente, facturacion), aprobarRevision() | 08, 09 |
| GestorAprobaciones | solicitar(tipo, cliente, valor, motivo), aprobar(solicitud), rechazar(solicitud), listarPendientes(usuario), aplicarExcepcion(solicitud) | 10, 11 |
| GestorFacturas | crearBorrador(cliente), añadirConcepto(factura, concepto), quitarConcepto(factura, concepto), guardarBorrador(), eliminarBorrador(), validarEmision(factura), emitir(factura), rectificar(factura, conceptos, motivo), descargarPDF(factura): void, generarRecurrentes(mes), buscar(filtro) | 12 a 16 |
| GestorCobros | registrarMovimiento(datos), conciliarAutomaticamente(movimiento), conciliarManual(movimiento, factura), revertirConciliacion(), abrirDisputa(), cerrarDisputa(), enviarRecordatorios(fecha), listarImpagos() | 18 a 21 |
| GestorTrabajo | crearTarea(), atenderTarea(), aplazarTarea(), abrirIncidencia(), comentarIncidencia(), resolverIncidencia(), enviarNotificacion(), reintentarEnvio(envio), gestionarRebote(envio) | 17, 22, 23 |
| GestorInformes | construirDashboard(usuario), generarInforme(tipo, filtro), exportar(informe, formato): void, buscar(texto) | 24 a 26 |
| GestorSistema | obtenerAjustes(), proponerCambioAjustes(), gestionarFiltros(), cerrarPeriodo(), reabrirPeriodo(), consultarSalud(), restaurarCopia(), atenderConsultaAPI(peticion) | 27 a 29, 31 |
| GestorMovimientos | registrarMovimiento(tipo, usuario, detalle), registrarMovimiento(tipo, usuario, detalle, entidad, id), registrarAuditoria(tabla, id, accion, usuario, antes, despues), consultarMovimientos(filtro) | transversal |
| GestorElementosVisuales | construirInterfaz(usuario), construirBandeja(usuario), abrirVentana(tipo), construirFicha(entidad), construirHistorial(tipo, filtro) | transversal |

### 4.3 Servicios de apoyo y patrones

- Reloj <\<singleton>>: +iniciar(), +programar(proceso, frecuencia), +tic(). Emite señales periódicas de revisión a los gestores: facturación del día 1 (GestorFacturas.generarRecurrentes), recordatorios diarios (GestorCobros.enviarRecordatorios), caducidad de delegaciones (GestorAcceso.devolverDelegacionesCaducadas) y recuperación de procesos incompletos tras una caída.
- AutenticadorValidador <\<static>>: clase estática (solo métodos de clase, sin instanciar objetos) que verifica las reglas internas. Métodos: +verificarCredenciales(email, pass), +comprobarPermiso(usuario, operacion, cliente): Booleano (prevención de accesos indebidos a objetos), +validarCorreo() (dominio), +validarPassword() (requisitos), +validarNIF(), +validarCSV(), +validarImporteEnUmbral(), +validarDatosFiscales().
- ControladorAPI: +consultarDeuda(idCliente), +avisarBaja(idCliente). Gestiona la sincronización directa y las bajas con la base de datos externa de TurbineH.
- FabricaInterfaces (patrón Factory): +crearInterfaz(usuario) devuelve la pantalla según el rol; la usa GestorElementosVisuales.
- Documentos PDF y archivos CSV: el PDF de una factura o de un informe se genera al vuelo y no se almacena. +descargarPDF(factura) y +exportar(informe, formato) no devuelven valor: generan el documento y solicitan al sistema operativo la descarga. La importación recibe la ruta del archivo CSV (+importarCSV(rutaArchivo: Texto)). El envío de una factura por correo o WhatsApp genera el PDF en el momento del envío (+enviarNotificacion).
- Elementos que no forman parte del modelo: PasarelaPago, ServicioMensajeria, GestorDescargas, clases PDF y CSV, LineaFactura, ServicioAuditoria (es el método GestorMovimientos.registrarAuditoria), Festivo (lista de fechas en AjustesSistema) y SerieFacturacion (integrada en AjustesSistema).

### 4.4 Interfaz (boundary)

Todas son <\<boundary>>. El GestorElementosVisuales controla 5 superclases o interfaces principales, y cada pantalla habla con su gestor de negocio.

| Superclase visual | Subtipos / clases hijas | Detalles de diseño |
| --- | --- | --- |
| 1. Interfaz | InterfazUsuario, InterfazAdmin, InterfazSuperadmin, Dashboard | Vistas principales del sistema según el rol (Dashboard solo AD y SAD). |
| 2. BandejaEntrada | BandejaUsuario, BandejaAdmin, BandejaSuperadmin | Muestra únicamente los objetos de tipo NotificacionUsuario. |
| 3. Ventana | Login y Cierre, Confirmación, Vista previa (borrador y PDF), Suscripción, Factura, Aprobación, Ajustes del sistema, Pantalla de salud, Periodo, Importación | Vistas completas o modales de interacción e introducción de datos. |
| 4. Ficha | FichaCliente, FichaFactura, FichaSuscripcion, FichaInforme (con cliente opcional y usuario), FichaMovimiento, FichaAsignacion | Vistas resumidas (tipo fila/tarjeta) que componen los elementos de cada historial. |
| 5. Historial | HistorialClientes, HistorialFacturas (emitidas), HistorialMovimientos (con filtros para cobros, usuarios, conciliaciones, etc.), HistorialAsignaciones (por usuario), HistorialInformes | Se construyen visualmente agrupando listas de su Ficha correspondiente. |

Relación con los gestores: Login y Cierre habla con GestorAcceso; Ventana Suscripción con GestorSuscripciones; Ventana Factura y Vista previa con GestorFacturas; Ventana Aprobación con GestorAprobaciones; Dashboard e Historial de informes con GestorInformes; Ajustes del sistema, Pantalla de salud y Periodo con GestorSistema; Importación y FichaCliente con GestorClientes; las tareas y notificaciones de la bandeja con GestorTrabajo; HistorialMovimientos con GestorMovimientos.

## 5. Relaciones entre clases

| Origen | Relación | Destino | Cardinalidad | Tipo UML |
| --- | --- | --- | --- | --- |
| Usuario | es especializado en | UsuarioOperativo, Administrador |  | Herencia |
| Administrador | es especializado en | SuperAdministrador |  | Herencia |
| Usuario | es responsable de | Cliente | 1 a 0..N | Asociación |
| Cliente | tiene historial de | AsignacionCliente | 1 a 0..N | Composición |
| Usuario | tiene | AsignacionCliente | 1 a 0..N | Asociación |
| Cliente | se delega mediante | DelegacionTemporal | 1 a 0..N | Asociación |
| Cliente | agrega | Contacto | N a M (mín. un contacto de facturación activo) | Agregación |
| Cliente | se clasifica con | Etiqueta | 0..N a 0..N | Agregación |
| Cliente | formaliza | Suscripcion | 1 a 1 | Composición |
| TarifaRango | determina el precio de | Suscripcion | 1 a 0..N | Asociación |
| Cliente | es revisado en | RevisionAnual | 1 a 0..N | Asociación |
| RevisionAnual | usa como tarifa calculada y aprobada | TarifaRango | N a 1 (la aprobada, opcional) | Asociación |
| Suscripcion | contiene | ExcepcionComercial | 1 a 0..N | Composición |
| ExcepcionComercial | crea y espera la resolución de (por método) | SolicitudAprobacion |  | Dependencia |
| Usuario | solicita / aprueba | SolicitudAprobacion | 1 a 0..N (dos roles) | Asociación |
| Tarea | es especializada en | DelegacionTemporal, RevisionAnual (Sistema); DisputaDeuda, IncidenciaFacturacion, SolicitudAprobacion (Usuario) |  | Herencia |
| Movimiento | es especializado en | RegistroAuditoria, EnvioComunicacion, ImportacionCSV |  | Herencia |
| Notificación | es especializada en | Correo, WhatsApp, NotificacionUsuario |  | Herencia |
| Factura | es especializada en | Recurrente, Puntual, Rectificativa |  | Herencia |
| Cliente | recibe | Factura | 1 a 0..N | Asociación |
| Usuario | crea | Factura | 1 a 0..N | Asociación |
| Suscripcion | genera (día 1) | FacturaRecurrente | 1 a 0..N | Asociación |
| Factura | contiene (-conceptos) | Concepto | N a M | Asociación |
| FacturaRectificativa | corrige a | Factura | N a 1 | Asociación |
| PeriodoAdministrativo | al cerrarse genera | Movimiento (CierrePeriodo) | 1 a 0..N | Dependencia |
| Conciliacion | incluye (-cobro, -factura) | MovimientoCobro, Factura | N a 1 con el cobro (único) y N a 1 con la factura | Asociación |
| Factura | puede tener | DisputaDeuda | 1 a 0..N | Asociación |
| DisputaDeuda | se relaciona con | Factura, Cliente, Usuario (empleado) | N a 1 cada uno | Asociación |
| IncidenciaFacturacion | se relaciona con | Cliente, Factura (opcional), Usuario | N a 1 cada uno | Asociación |
| IncidenciaFacturacion | se documenta en | ComentarioIncidencia | 1 a 0..N | Composición |
| EnvioComunicacion | incluye | Notificación (Correo o WhatsApp), Factura, Contacto | N a 1 cada uno | Asociación |
| Usuario | recibe | Notificación (como destinatario) | 1 a 0..N | Asociación |
| Usuario | realiza | Movimiento | 1 a 0..N | Asociación |
| Gestores | usan | Repositorio, Filtro, AutenticadorValidador, GestorMovimientos |  | Dependencia |
| Reloj | activa | GestorFacturas, GestorCobros, GestorAcceso |  | Dependencia |
| GestorElementosVisuales | controla | las 5 superclases visuales |  | Asociación |

## 6. Cómo pasar las clases al diagrama ER

### 6.1 Reglas de conversión

1. Solo las clases <\<entity>> se convierten en tablas.
2. Los métodos no se pasan.
3. Herencia de Usuario: una tabla USUARIO con la columna rol (UO, AD, SAD). Los atributos específicos (umbral, permiso de alta) son columnas opcionales.
4. Herencia de Factura: una tabla FACTURA con la columna tipo (Recurrente, Puntual, Rectificativa) y una tabla FACTURA\_RECTIFICATIVA (su clave es también clave foránea de FACTURA). Para las recurrentes, id\_suscripcion y mes\_facturado son columnas opcionales.
5. Lista de conceptos de una factura: tabla intermedia FACTURA\_CONCEPTO, que guarda el nombre y el precio del concepto en el momento de añadirlo.
6. Herencia de Tarea: una tabla TAREA con las columnas generales (id\_tarea, ejecutor, estado, fecha\_creacion, fecha\_aplazada\_fin) y una tabla por cada subclase cuya clave primaria es también clave foránea de TAREA.
7. Movimiento: una tabla MOVIMIENTO (id\_movimiento, tipo, fecha\_hora, id\_usuario, entidad\_afectada, id\_entidad\_afectada) y tablas propias para las subclases con atributos específicos: ENVIO\_COMUNICACION, REGISTRO\_AUDITORIA e IMPORTACION\_CSV, cuya clave primaria es también clave foránea de MOVIMIENTO.
8. Notificación: una tabla NOTIFICACION con la columna tipo (Correo, WhatsApp, Usuario), usuario\_destinatario (opcional) y leida (solo para el tipo Usuario).
9. Composición: la clave foránea es obligatoria. Agregación con relación N a M (Cliente–Contacto): tabla intermedia CLIENTE\_CONTACTO. Muchos a muchos Cliente–Etiqueta: tabla intermedia CLIENTE\_ETIQUETA.
10. Singleton AjustesSistema: tabla clave-valor (AJUSTES\_SISTEMA) que incluye series de facturación, lista de festivos y filtros guardados.
11. Asociaciones con dos roles hacia Usuario (solicitante y aprobador): dos claves foráneas distintas a la misma tabla, con nombres claros (id\_solicitado\_por, id\_aprobado\_por).

### 6.2 Tablas resultantes

| Tabla | Clave primaria | Claves foráneas principales |
| --- | --- | --- |
| USUARIO | id\_usuario |  |
| DELEGACION\_TEMPORAL | id\_tarea (FK a TAREA) | cliente, usuario\_origen, usuario\_delegado |
| ASIGNACION\_CLIENTE | id\_asignacion | cliente, usuario |
| CLIENTE | nif\_cif | usuario\_responsable |
| CONTACTO\_CLIENTE | id\_contacto |  |
| CLIENTE\_CONTACTO | (cliente, contacto) | cliente, contacto |
| ETIQUETA\_CLIENTE | id\_etiqueta |  |
| CLIENTE\_ETIQUETA | (cliente, etiqueta) | cliente, etiqueta |
| SUSCRIPCION | id\_suscripcion | cliente (única), tarifa, usuario\_comercial |
| TARIFA\_RANGO | id\_tarifa |  |
| REVISION\_ANUAL\_TARIFA | id\_tarea (FK a TAREA) | cliente, tarifa\_calculada, tarifa\_aprobada, aprobado\_por |
| SOLICITUD\_APROBACION | id\_tarea (FK a TAREA) | cliente, factura, solicitado\_por, aprobado\_por |
| EXCEPCION\_COMERCIAL | id\_excepcion | suscripcion |
| FACTURA | id\_factura | cliente, suscripcion (opcional), usuario\_creador |
| FACTURA\_RECTIFICATIVA | id\_factura\_rectificativa | factura\_original, aprobado\_por |
| CONCEPTO | id\_concepto |  |
| FACTURA\_CONCEPTO | (factura, concepto) | factura, concepto |
| PERIODO\_ADMINISTRATIVO | id\_periodo |  |
| MOVIMIENTO\_COBRO | id\_movimiento |  |
| CONCILIACION\_COBRO | id\_conciliacion | movimiento (única), factura |
| DISPUTA\_DEUDA | id\_tarea (FK a TAREA) | factura, cliente, empleado |
| INCIDENCIA\_FACTURACION | id\_tarea (FK a TAREA) | cliente, factura (opcional), usuario |
| COMENTARIO\_INCIDENCIA | id\_comentario | incidencia (obligatoria, composición) |
| TAREA | id\_tarea |  |
| MOVIMIENTO | id\_movimiento | usuario (opcional) |
| ENVIO\_COMUNICACION | id\_movimiento (FK a MOVIMIENTO) | notificacion, factura, contacto |
| REGISTRO\_AUDITORIA | id\_movimiento (FK a MOVIMIENTO) | usuario |
| IMPORTACION\_CSV | id\_movimiento (FK a MOVIMIENTO) | usuario |
| NOTIFICACION | id\_notificacion | usuario\_destinatario (opcional) |
| INFORME | id\_informe | usuario, cliente (opcional) |
| AJUSTES\_SISTEMA | clave |  |
