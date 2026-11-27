# Diagrama UML de clases

## Alcance

Estructura estática de la plataforma de facturación y gestión de clientes de TurbineH, deducida de los 31 casos de uso (CU-01 a CU-31). El diagrama recoge tres capas:

- **Dominio** (`<<entity>>`): lo que el negocio maneja y recuerda: clientes, contactos, suscripciones, facturas, cobros, tareas, movimientos, notificaciones, informes y ajustes.
- **Control** (`<<control>>`): doce gestores, uno por tema de negocio. Reciben las peticiones de los casos de uso, comprueban permisos, aplican las reglas y coordinan al resto. Son `GestorAcceso`, `GestorEmpleados`, `GestorClientes`, `GestorSuscripciones`, `GestorAprobaciones`, `GestorFacturas`, `GestorCobros`, `GestorTareas`, `GestorNotificaciones`, `GestorInformes`, `GestorSistema` y `GestorMovimientos`.
- **Acceso a datos y servicios de apoyo**: repositorios (`<<boundary>>`), la interfaz `Filtro<T>`, el temporizador `Reloj` (`<<singleton>>`), `AutenticadorValidador` (`<<static>>`) y `ExportadorArchivos` (`<<singleton>>`).

Mapa de paquetes del diagrama:

| Paquete | Contenido | CU principales |
|---|---|---|
| `ClasesEmpleados` | `Usuario` (abstracta) y sus roles `Usuario_Operativo`, `Administrador`, `SuperAdministrador` | 01, 02, 02.5, 03 |
| `ClasesClientes` | `Cliente`, `Contacto`, `Etiqueta` y sus enumerados | 04, 05, 06, 07, 30 |
| `ClasesSuscripcion` | `Suscripcion`, `ExcepcionComercial` | 08, 10, 14 |
| `ClasesFacturas` | `Factura` (abstracta) con `Factura_Puntual`, `Factura_Recurrente`, `Factura_Rectificada`; `Concepto` | 12 a 16 |
| `ClasesPaqueteConciliacion` | `Conciliacion`, `MovimientoCobro` | 20, 21 |
| `ClasesTarea` | `Tarea` (abstracta) con `DelegacionTemporal`, `RevisionAnual` (ejecuta el sistema) y `DisputaDeuda`, `IncidenciaFacturacion`, `SolicitudAprobacion` (ejecuta un usuario); `TarifaRango`, `ComentarioIncidencia` | 03, 09, 10, 11, 19, 22, 23 |
| `ClasesMovimiento` | `Movimiento` (abstracta) con 14 subclases y el enumerado `TipoMovimiento` | 28 y trazabilidad de casi todos los CU |
| `ClasesNotificacion` | `Notificacion` (abstracta) con `CorreoElectronico`, `WhatsApp`, `NotificacionInterna` | 13, 17, 18, 20, 22 |
| `ClasesInforme` | `Informe`, `Archivo`, `CSV`, `PDF` | 24, 25 |
| `ClasesAjustesDelSistema` | `AjustesSistema`, `AjusteSistemaPuntual`, `Festivo`, `PeriodoAdministrativo`, `Reloj`, `AutenticadorValidador`, `Filtro<T>` y sus siete implementaciones | 26, 27, 29 y transversales |
| `ClasesGestor*` y `ClasesAdicionales` | Los gestores, los repositorios concretos y las interfaces `Repositorio<T>` y `RepositorioSobreescritura<T>` | todos |

Quedan fuera del diagrama: la capa de pantallas (bandejas, ventanas, fichas e historiales), los tipos de dato definitivos y las firmas finales de los métodos. Las operaciones que aparecen muestran responsabilidades, no una implementación cerrada.

## Decisiones de diseño

### Arquitectura y reparto de responsabilidades

- **Un gestor por tema de negocio.** Cada gestor agrupa los CU de su tema (por ejemplo, `GestorFacturas` cubre CU-12 a CU-16 y `GestorCobros` CU-19 a CU-21). Así cada parte se puede probar y cambiar por separado. Las dependencias entre gestores (`..>`) existen porque una operación suele cruzar temas: emitir una factura (CU-13) necesita ajustes (serie y umbrales), notificaciones (envío), movimientos (auditoría) y aprobaciones (importes anómalos).
- **Los gestores no guardan datos, los repositorios sí.** Cada gestor de una entidad persistente delega en su repositorio (`RepositorioCliente`, `RepositorioFacturas`, `RepositorioTareas`, `RepositorioEmpleado`, `RepositorioMovimiento`, `RepositorioNotificacion`, `RepositorioInforme`).
- **Dos interfaces de repositorio.** `Repositorio<T>` solo permite `agregar` y buscar (`buscarPuntual`, `buscarPorFiltrado`). `RepositorioSobreescritura<T>` añade `actualizar` y `eliminar*`. Movimientos, notificaciones e informes implementan solo la primera porque son registros que no se modifican ni se borran (CU-28 exige un historial fiable). Clientes, facturas, tareas y empleados implementan la segunda.
- **`Filtro<T>` como objeto de consulta.** Las siete clases `FiltroClientes`, `FiltroFacturas`, `FiltroMovimientos`, `FiltroTareas`, `FiltroInformes`, `FiltroEmpleados` y `FiltroNotificacion` implementan `aplicarFiltro`. El gestor recibe el filtro y lo pasa al repositorio, de modo que no hace falta cargar todos los datos en memoria. Con ello se resuelven CU-16, CU-25, CU-26 y CU-28, que filtran por cliente, estado, fechas, tipo o responsable. Los filtros guardados (favoritos del CU-26) se almacenan en `AjustesSistema.filtros` y se gestionan desde `GestorSistema`.
- **`AutenticadorValidador` es una clase estática.** Concentra permisos (`comprobarPermisos(usuario, cliente)`, que da respuesta a USU-04) y validaciones (NIF, correo, contraseña, CSV, datos fiscales, importes en umbral, consultas por API). Se llama desde varios gestores sin crear objetos.
- **`Reloj` lanza los procesos automáticos.** Es un singleton que activa `enviarRecurrentes` (CU-14), `resolverTareas` (caducidad de delegaciones, CU-03), `resolverExportaciones` (informes programados, CU-25), `revisarClientesYContactosLimite` (borrado diferido, CU-30) y `revisarEmpleadosLimite` (CU-02.5). Lee los plazos y festivos de `AjustesSistema`. Los envíos de recordatorios (CU-18) y la conciliación automática (CU-20) también dependen de él (`Reloj ..> GestorCobros`, `GestorFacturas`).

### Jerarquías de herencia

- **`Usuario` → `Usuario_Operativo`, `Administrador`, `SuperAdministrador`.** Los tres roles comparten identidad y credenciales, pero difieren en permisos. El operativo solo accede a sus clientes asignados (`asignaciones`, `asignadoA`) y puede dar de alta clientes solo si tiene permiso (`puedeDarAltaClientes`, CU-04). El administrador tiene `umbralAprobacion`: aprueba hasta ese importe, y lo que lo supera le corresponde al superadministrador (CU-10, CU-11). El enumerado `Rol` se usa al crear cuentas (`GestorAcceso.crearUsuario`) y al filtrar empleados.
- **`Factura` → `Factura_Puntual`, `Factura_Recurrente`, `Factura_Rectificada`.** Comparten numeración, importes, estado (`EstadoFactura`) y ciclo de vida, y cada subclase añade lo propio: `presupuestoOrigen` (CU-12), `mesFacturado` y vínculo con `Suscripcion` (CU-14), y `ifdFacturaOriginal`, `motivoRectificacion` e `importeAjustado` (CU-15).
- **`Tarea` → cinco subclases.** `Tarea` es una clase abstracta (no una interfaz) porque aporta estado y fechas comunes (`estado`, `fechaCreacion`, `fechaAplazada`, `fechaFin`) y las operaciones `resolver` y `aplazar`. Es el elemento que se lista en la pantalla de trabajo diaria (CU-22). El enumerado `Ejecutores` distingue lo que resuelve el sistema (`DelegacionTemporal`, `RevisionAnual`) de lo que resuelve una persona (`DisputaDeuda`, `IncidenciaFacturacion`, `SolicitudAprobacion`). `AsignacionDesasigancionCliente` no es una tarea porque se ejecuta al instante.
- **`Movimiento` → 14 subclases.** Es el registro de todo lo que ocurre en el sistema (qué, quién y cuándo). Alimenta la línea temporal del cliente (CU-05), la auditoría y la supervisión (CU-28). `TipoMovimiento` identifica cada subclase y permite filtrar (`FiltroMovimientos.tipo`).
- **`Notificacion` → `CorreoElectronico`, `WhatsApp`, `NotificacionInterna`.** Cada canal añade sus datos (correo, teléfono) y `NotificacionInterna` añade `usuarioReceptor` y `leida` para la bandeja de los empleados. Se separa el contenido (`Notificacion`) del suceso de enviarlo (`EnvioComunicacion`, que es un `Movimiento`): el envío tiene estado, intentos y reintentos (CU-13, CU-17), mientras que la notificación es solo el mensaje.

### Relaciones: composición, agregación, asociación y dependencia

- **Composición (la parte no existe sin el todo):** `Cliente` y `Suscripcion` (una suscripción es de un único cliente, CU-08); `IncidenciaFacturacion` y `ComentarioIncidencia` (CU-23); `AjustesSistema` y `AjusteSistemaPuntual` (CU-27).
- **Agregación (las partes existen por sí solas):**
  - `Cliente` con `Contacto`, relación N a M: una persona puede ser contacto de dos empresas, y CU-04 exige al menos un contacto de facturación.
  - `Cliente` con `Etiqueta`: las etiquetas son un catálogo compartido.
  - `Concepto` con `Factura`: el catálogo es independiente, y al añadir un concepto se copian sus datos para que un cambio posterior del catálogo no altere facturas ya creadas (postcondición del CU-12).
  - `TarifaRango` con `RevisionAnual`, `Conciliacion` con `MovimientoCobro`, y `AjustesSistema` con `Festivo` y con `Filtro`.
- **Asociación:** `Suscripcion` mantiene una asociación 0..1 con `ExcepcionComercial`. La excepción aprobada permanece vigente y se aplica a las facturas siguientes hasta que caduca (CU-10, CU-11, CU-14). Las clases `Movimiento` y `Tarea` apuntan a `Cliente`, `Factura` y `Usuario` para poder reconstruir quién hizo qué sobre qué.
- **Dependencia (`..>`):** se usa entre gestores y hacia clases de apoyo, porque son usos y no pertenencia.

### Reglas de negocio que el diagrama hace visibles

- **Facturas emitidas inalterables (CU-13, CU-15).** `Factura.esEditable()` solo es verdadero en borrador. Una emitida se corrige con una `Factura_Rectificada` vinculada, y la original pasa a `RECTIFICADA`. `PeriodoAdministrativo.permiteEmitir` y `cerrar`/`reabrir` (CU-29) bloquean o desbloquean modificar un trimestre.
- **Aprobaciones como tareas (CU-10, CU-11).** Una `SolicitudAprobacion` es una `Tarea` de usuario. Su `aprobar` y `rechazar` actúan sobre una operación pendiente (`OperacionesPosibles`) y, si procede, activan el efecto con `GestorAprobaciones.aplicarExcepcion`. La excepción solicitada y la ya vigente son clases distintas (`SolicitudAprobacion` frente a `ExcepcionComercial`).
- **Cambios de ajustes con validación (CU-27).** `AjustesSistema.proponerCambio` no aplica nada: `GestorAprobaciones.solicitarCambioAjustes` crea una solicitud y `GestorSistema.aplicarCambioAjuste` lo aplica solo si se aprueba. Cada ajuste pasa por los estados `States` (`PENDIENTE`, `VIGENTE`, `RECHAZADA`). Los cambios afectan solo a lo nuevo.
- **Recordatorios pausables (CU-18, CU-19, CU-23).** `Factura.desactivarNotificaciones` y `reactivarNotificaciones`, `DisputaDeuda.recordatoriosCongelados` e `IncidenciaFacturacion.pausarRecordatorios` expresan que una factura en disputa o con incidencia no recibe avisos.
- **Conciliación (CU-20, CU-21).** `MovimientoCobro` es el hecho recibido de la pasarela (`PasarelasPago`, `referenciaTransaccion` para detectar duplicados con `esDuplicado`). `Conciliacion` lo vincula con una factura, con tipo (`AUTOMATICA` o `MANUAL`), discrepancia de importe y posibilidad de `revertir`. No son la misma clase porque un cobro puede existir sin estar conciliado.
- **Auditoría (CU-28, y todo CU que cambia datos).** Cada gestor registra sus operaciones mediante `GestorMovimientos.registrarMovimiento` y `registrarAuditoria` (con valor anterior y nuevo en `RegistroAuditoria`).
- **Importes y fechas.** El diagrama usa `double` y `Date` por simplicidad. En la base de datos los importes serán decimales exactos.

## Trazabilidad

Los identificadores de requisito son los de la especificación de casos de uso.

| Requisito / CU | Clases y operaciones | Motivo |
|---|---|---|
| CU-01 · USU-01 | `GestorAcceso.iniciarSesion`, `recuperarContrasenya`; `Usuario.autenticarPassword`, `cambiarPassword`, `estado` (`EstadoCuenta`), `ultimoAcceso`; `AutenticadorValidador.verificarCredenciales`, `validarCorreo`, `validarPassword` | Acceso solo con correo corporativo y sin autorregistro. La recuperación de contraseña se envía por correo (`CorreoElectronico`). |
| CU-01.5 · USU-01 | `GestorAcceso.cerrarSesion`, `cerrarPorInactividad`; tiempo de inactividad como `AjusteSistemaPuntual` | El cierre manual y el automático por inactividad, cuyo plazo configura el superadministrador. |
| CU-02 · USU-01, USU-02 | `GestorEmpleados.registrarEmpleado`, `GestorAcceso.crearUsuario(correo, password, rol)`; `Usuario` y roles; `Rol`; `AltaBajaEmpleado`; `RepositorioEmpleado.agregar`; `AutenticadorValidador.validarCorreo` | Alta de empleados con rol. Se rechazan correos fuera de dominio o repetidos, y el alta queda como `Movimiento`. |
| CU-02.5 · USU-01, USU-02 | `GestorEmpleados.desactivar`, `activar`, `revisarEmpleadosLimite`; `GestorAcceso.desactivarUsuario`; `Usuario.desactivar`, `reactivar`; `AltaBajaEmpleado.esBaja`; `Reloj.revisarEmpleadosLimite` | Desactivar sin perder el historial (`Movimiento.encargado` se conserva) y reasignar los clientes antes de dejar la cuenta inactiva. |
| CU-03 · USU-03, CLI-03, PRO-03 | `GestorClientes.asignar`; `Cliente.responsable`; `Usuario_Operativo.asignaciones`, `asignadoA`; `AsignacionDesasigancionCliente` (`responsableOriginal`, `responsableNuevo`); `DelegacionTemporal.activar`, `caducar`, `estaVigente`; `Reloj.resolverTareas`; `AjustesSistema.limiteClientesPorUsuario` | Asignación estándar, temporal (la caducidad la ejecuta el sistema, de ahí que sea una `Tarea` de ejecutor `SISTEMA`) y masiva. Se respeta el límite de clientes por usuario. |
| CU-03.5 · USU-03, CLI-03, PRO-03 | `GestorClientes.desasignar`; `AsignacionDesasigancionCliente`; `DelegacionTemporal.caducar`, `revocar`; `GestorEmpleados.obtenerAsignados` | Quitar una asignación exige un nuevo responsable. La devolución tras caducidad es el mismo mecanismo. |
| CU-04 · CLI-01, CLI-02, CLI-04, USU-04, PRO-02 | `GestorClientes.registrarCliente`, `registrarContacto`, `reactivarCliente`; `Cliente.tieneDatosFiscalesCompletos`, `tieneContactoFacturacion`; `Contacto.rolContacto` (`RolesContacto`); `Usuario_Operativo.puedeDarAltaClientes`; `AutenticadorValidador.validarNIF`, `validarDatosFiscales`, `comprobarPermisos`; `AltaBajaCliente`; `RepositorioCliente.agregar` | Alta con diferencia entre datos imprescindibles y opcionales, detección de NIF duplicado, reactivación de fichas antiguas y marca de ficha incompleta. |
| CU-05 · CLI-01, CLI-02, CLI-05, USU-04, PRO-02 | `GestorClientes.recuperarClientePuntual`, `actualizarCliente`, `anyadirContacto`, `desactivarContacto`, `etiquetar`; `Cliente.anyadirEtiqueta`, `quitarEtiqueta`; `Contacto.activar`, `desactivar`; `Etiqueta`; `GestorMovimientos.recuperarMovimientos` | Consultar y editar la ficha con su línea temporal (los `Movimiento` del cliente) y gestionar contactos y etiquetas. |
| CU-06 · IMP-01, FAC-07, USU-04, PRO-02 | `GestorClientes.importarCSV`; `AutenticadorValidador.validarCSV`; `ImportacionCSV` (`filasTotales`, `filasCorrectas`, `EstadoCSV`); `CSV` (hereda de `Archivo`); `Factura.esMigradaHistorica` | La carga masiva se valida antes de importar y deja constancia como `Movimiento`. Las facturas históricas se marcan como migradas. |
| CU-07 · SUB-04, CLI-03, CLI-04, AMB-04, USU-04, PRO-02 | `GestorClientes.darDeBaja`; `Cliente.darDeBaja`, `fechaBajaEfectiva`, `motivo`; `EstadoCliente`; `Suscripcion.cancelar`; `AltaBajaCliente.esBaja`; `GestorFacturas.listarImpagos` | Baja lógica a una fecha efectiva. Se detiene la facturación recurrente pero se mantiene el recobro de las facturas pendientes. |
| CU-08 · SUB-01 a SUB-04, AMB-05, USU-04, PRO-02 | `GestorSuscripciones.crearSuscripcion`, `actualizarSuscripcion`, `pausar`, `reactivar`, `calcularProrrateo`; `Suscripcion.calcularProrrateoPrimerMes`, `pausar`, `reactivar`, `excepcionSinProrrateo`, `plazoPagosDias`, `EstadoSub` | Alta y modificación del contrato recurrente, prorrateo del primer mes y pausas. |
| CU-09 · TAR-02, USU-04, PRO-02 | `RevisionAnual` (`calcularTarifaPropuesta`, `aprobar`, `rechazar`, `mantener`, `EstadoRevision`); `TarifaRango.aplicaA`, `estaVigente`; `Suscripcion.aplicarNuevaTarifa`; `GestorMovimientos.registrarAuditoria` | La revisión anual es una `Tarea` del sistema. La tarifa nueva se propone y, aprobada, se aplica a las facturas siguientes dejando auditoría. |
| CU-10 · APR-01, USU-04, PRO-02 | `GestorAprobaciones.solicitar`; `SolicitudAprobacion` (`tipoOperacion`, `valorSolicitado`, `motivo`, `requiereAprobacion`); `ExcepcionComercial` (`tipoExcepcion`, `descuento`, `vigenteDesde`, `vigenteHasta`); `AjustesSistema.umbralAprobacion`; `NotificacionInterna` | Solicitar un descuento, un primer mes gratis o un precio pactado. Queda pendiente y se notifica a quien debe aprobarlo. |
| CU-11 · APR-01, APR-02, FAC-06, REC-02, USU-04, PRO-02 | `GestorAprobaciones.listarPendiente`, `aprobar`, `rechazar`, `aplicarExcepcion`; `SolicitudAprobacion.aprobar`, `rechazar`, `resolucion` (`Resolucion`), `aprobador`; `AutenticadorValidador.validarImporteEnUmbral`; `GestorNotificaciones.comunicarResultado`; `ExcepcionComercial.aplicar`; `AplicarExcepcionComercial` | Un administrador resuelve hasta su umbral y el superadministrador por encima. Se notifica al solicitante y se registra la decisión. |
| CU-12 · FAC-01, FAC-03, USU-04, PRO-02 | `GestorFacturas.crearBorrador`, `anyadirConcepto`, `eliminarConcepto`, `eliminarBorrador`; `Factura_Puntual.presupuestoOrigen`; `Factura.recalcularTotales`, `esEditable`; `Concepto`; `EstadoFactura.BORRADOR`; `RepositorioFacturas` | Preparar una factura puntual que queda en borrador, sin número definitivo y editable. |
| CU-13 · FAC-04, FAC-05, PDF-01, ENV-01, ENV-02, APR-02, USU-04, PRO-02 | `GestorFacturas.validarEmision`, `emitir`, `generarPDF`; `Factura.emitir`, `numeroSerieCompleto`, `serie`, `numeroSecuencia`; `AjustesSistema.siguienteNumeroSerie`, `margenPatronImporte`, `umbralDeFacturaPuntualAlta`; `PeriodoAdministrativo.permiteEmitir`; `EnvioComunicacion` (`marcarEnviado`, `marcarRebotado`, `reenviar`); `Contacto.admiteCanal`; `GeneracionFactura` | Convertir un borrador en definitivo: validar, numerar, generar el PDF (sin guardarlo) y enviarlo. Se registra el envío y, si hay anomalía de importe, se pide aprobación. |
| CU-14 · GEN-01, SUB-01, PRO-01 | `Reloj.enviarRecurrentes`; `GestorFacturas.generarRecurrentes`; `Factura_Recurrente.mesFacturado`; `Suscripcion.debeFacturarse`, `aplicarNuevaTarifa`; `ExcepcionComercial.estaVigente`, `aplicar` | El día 1 se genera una factura por suscripción activa, aplicando tarifa y descuentos vigentes. Se emite siguiendo CU-13. |
| CU-15 · FAC-06, FAC-02, AMB-01, USU-04, PRO-02 | `GestorFacturas.rectificarFactura`; `Factura_Rectificada` (`ifdFacturaOriginal`, `motivoRectificacion`, `importeAjustado`); `Factura.marcarRectificada`; `RectificacionFactura`; `SolicitudAprobacion` (`OperacionesPosibles.RECTIFICATIVAALTA`) | La factura original no se modifica: se corrige con una rectificativa vinculada, que pide aprobación si supera el umbral. |
| CU-16 · FAC-02, PDF-01, USU-04 | `GestorFacturas.obtenerFacturaPuntual`, `obtenerFacturas`, `generarPDF`; `FiltroFacturas`; `Usuario.puedeAccederACliente`; `AutenticadorValidador.comprobarPermisos`; `PDF` | Buscar y consultar una factura y descargar su PDF. Un usuario operativo solo ve las de sus clientes. |
| CU-17 · ENV-02, ENV-03, USU-04, PRO-02 | `IncidenciaFacturacion` (`TiposIncidencia.ENVIOREBOTADO`); `GestorNotificaciones.gestionarRebote`, `reintentarEnvio`; `EnvioComunicacion` (`EstadoMensaje`, `esReintentado`, `numeroIntento`); `GestorTareas.resolverIncidencia` | Reenviar la misma factura con el mismo número y distinguir el envío original de los reintentos. |
| CU-18 · REC-01 | `Reloj` → `GestorFacturas.listarImpagos`, `enviarRecordatorio`, `pausarRecordatorios`, `reactivarRecordatorios`; `Factura.recibirNotificaciones`; `TipoMensaje` (`RECORDATORIO`, `IMPAGO`); `AjustesSistema.diasHastaRecordatorio`, `diasAvisosEnImpagos`, `esLaborable`, `festivos` | Proceso diario que envía recordatorios y avisos de impago, respeta festivos y se detiene con disputa, pausa o pago. |
| CU-19 · REC-02 | `GestorCobros.abrirDisputa`, `cerrarDisputa`; `DisputaDeuda` (`motivoReclamacion`, `fechaRevisionPrevista`, `recordatoriosCongelados`, `cerrar`); `Factura.marcarIncobrable`; `SolicitudAprobacion` (`CONDONACIONDEUDA`) | Registrar la reclamación del cliente y congelar los avisos. Cancelar o declarar incobrable una deuda requiere aprobación. |
| CU-20 · PAG-01, PAG-02, NOT-01 | `GestorCobros.conciliarAutomaticamente`; `MovimientoCobro` (`fuentePasarela`, `esDuplicado`, `coincideCon`); `Conciliacion` (`tipo`, `discrepanciaImporte`, `tieneDiscrepancia`); `Factura.marcarPagada`; `RegistroCobro`; `GestorNotificaciones` | Emparejar el cobro recibido con la factura, ignorar avisos duplicados, marcar pagada y avisar al responsable. |
| CU-21 · PAG-02, PAG-03 | `GestorCobros.conciliarManual`, `revertirConciliacion`; `Conciliacion.revertir`, `motivoRevertido`, `EstadoConciliacion`; `TipoConciliacion.MANUAL`; `IncidenciaFacturacion` (`PAGONOLOCALIZADO`) | El cobro dudoso pasa a un responsable, que lo concilia a mano o lo deshace dejando traza. |
| CU-22 · DAS-01, NOT-01 | `GestorTareas.obtenerTareas`, `resolverTarea`, `aplazarTarea`; `FiltroTareas`; `Tarea.resolver`, `aplazar` y sus cinco subclases; `GestorAprobaciones.listarPendiente`; `GestorFacturas.listarImpagos`; `NotificacionInterna.leida`, `marcarLeida`; `GestorNotificaciones.obtenerNotificacionesRecibidas` | Una sola bandeja con todo lo pendiente (vencimientos, cobros, entregas, aprobaciones, revisiones, tareas y notificaciones), según el rol. Por eso `Tarea` es polimórfica. |
| CU-23 · NOT-02 | `GestorTareas.abrirIncidencia`, `comentarIncidencia`, `resolverIncidencia`; `IncidenciaFacturacion` (`tipoIncidencia`, `prioridad`, `estadoIncidencia`, `pausarRecordatorios`, `asignar`, `cambiarEstado`); `ComentarioIncidencia.nivelVisibilidad` | Abrir y documentar una incidencia sobre un cliente o factura, con responsable, prioridad, comentarios y pausa de recordatorios. |
| CU-24 · DAS-02 | `GestorInformes.construirDashboard`; `GestorCobros.obtenerDatosFinancieros` | Cifras agregadas de facturado, cobrado y pendiente para el administrador. |
| CU-25 · INF-01 | `GestorInformes.generarInforme`, `generarInformeCambioPeriodo`, `exportarInforme`, `exportacionMasiva`, `programar`, `buscarInformes`; `Informe` (`tipo`, `parametros`, `formato`); `FormatosValidos`; `ExportadorArchivos.exportar`; `GeneracionInforme`; `FiltroInformes`; `Reloj.resolverExportaciones` | Generar, filtrar, exportar (CSV o PDF) y programar informes. Las exportaciones masivas se auditan. |
| CU-26 · BUS-01 | `Filtro<T>` y `FiltroClientes`, `FiltroFacturas`, `FiltroInformes`; `GestorClientes.recuperarClientes`, `GestorFacturas.obtenerFacturas`; `GestorSistema.obtenerFiltros`, `gestionarFiltros`; `Tipo_Filtro` | Búsqueda global y filtros guardados. Un mismo contrato de filtro sirve para todas las entidades. |
| CU-27 · CFG-01, TAR-01, FAC-04, REC-01, NOT-01 | `GestorSistema.obtenerAjustes`, `proponerCambioAjustes`, `aplicarCambioAjuste`, `guardarAjustesActualizados`, `gestionarFestivos`, `gestionarExcepciones`; `AjustesSistema` y `AjusteSistemaPuntual` (`States`); `GestorAprobaciones.solicitarCambioAjustes` | Los parámetros estructurales los cambia solo el superadministrador, con validación previa, y solo afectan a lo nuevo. |
| CU-28 · AUD-01, PRO-01, PRO-05 | `GestorMovimientos.recuperarMovimientos`, `registrarMovimiento`, `registrarAuditoria`; `RegistroAuditoria` (`TipoAccion`, `valorAnterior`, `valorNuevo`); `FiltroMovimientos`; `GestorAprobaciones.solicitarAuditoria`; `GestorSistema.restaurarCopia`; `GestorNotificaciones.recuperarNotificaciones` | Supervisar acciones, errores y correos enviados, y restaurar copias. La auditoría es una bitácora que no se modifica. |
| CU-29 · PRO-04, AMB-01 | `GestorSistema.cerrarPeriodo`, `reabrirPeriodo`, `restaurarCopia`; `PeriodoAdministrativo` (`cerrar`, `reabrir`, `contieneFecha`, `permiteEmitir`, `EstadoPeriodo`, `autorizacionSuperior`); `CierrePeriodoAdministrativo`; `GestorInformes.generarInformeCambioPeriodo`; `AjustesSistema.antiguedadMaximaReaperturasPeriodos` | Cerrar un trimestre bloquea sus facturas. Reabrirlo exige autorización y respeta una antigüedad máxima. Cada cierre queda como `Movimiento`. |
| CU-30 · PRI-01 | `GestorClientes.anonimizarCliente`, `anonimizarContacto`, `borrarClientesYContactoLimite`; `Contacto.anonimizar` (`EstadoContacto.ANONIMIZADORGPD`); `Cliente.idTareaEliminar`; `Reloj.revisarClientesYContactosLimite` | Anonimizar primero y borrar al cabo de un tiempo. Si el cliente vuelve antes, se recupera su ficha (`reactivarCliente`). |
| CU-31 · API-01, USU-04, PRO-02 | `GestorSistema.atenderConsultaAPI`; `AutenticadorValidador.validarConsultaAPI`; `GestorFacturas.listarImpagos`; `IncidenciaFacturacion` (`AVISOBAJAAPI`); `GestorTareas.abrirIncidencia` | La consulta de deuda no modifica datos. Un aviso de baja crea una tarea para el responsable en lugar de dar la baja automáticamente. |
| Transversal · USU-04, PRO-02 | `AutenticadorValidador.comprobarPermisos`; `Usuario.puedeAccederACliente`; `GestorMovimientos.registrarMovimiento`; `Factura_Recurrente` frente a `Suscripcion` | Permiso por cliente en cada operación, un registro de cada acción y protección frente a duplicados (una factura recurrente por suscripción y mes). |

## Consistencia con otros modelos

El diagrama de clases es el modelo de referencia. El ER físico lo materializa, pero **no toda clase implica una tabla ni toda tabla procede de una clase**.

### Reglas de correspondencia

- **Atributo `idX` → clave primaria `id BIGINT`.** Los importes (`double`) se guardan como `NUMERIC`, las fechas (`Date`) como `DATE` o `TIMESTAMPTZ` y los enumerados como columnas `VARCHAR`, sin tabla propia.
- **Herencia con tabla raíz y tablas hijas.** `Tarea` → `tarea` con `delegacion_temporal`, `revision_anual_tarifa`, `solicitud_aprobacion`, `disputa_deuda` e `incidencia_facturacion`, cuya clave es la de `tarea` (`tarea_id`). `Movimiento` → `movimiento` con una tabla hija por subclase (`movimiento_id`). `Factura` → `factura` con columna `tipo` y `factura_rectificativa`.
- **Herencia con tabla única.** `Usuario` y sus roles → `usuario` con la columna `rol` (el umbral y el permiso de alta son columnas opcionales). `Notificacion` y sus canales → `notificacion` con `tipo` y columnas opcionales (`correo_*`, `telefono_*`, `usuario_receptor_id`, `leida`).
- **Composición → clave foránea obligatoria**; si es 1 a 1 también única (`suscripcion.cliente_id`, `comentario_incidencia.incidencia_id`). **Agregación N a M → tabla intermedia.**

### Clases con tabla

| Concepto del diagrama | Tabla del ER | Observación |
|---|---|---|
| `Usuario`, `Usuario_Operativo`, `Administrador`, `SuperAdministrador` | `usuario` | Tabla única. `Rol` es la columna `rol`. |
| `Cliente` | `cliente` | `responsable` → `usuario_responsable_id`. |
| `Contacto` | `contacto`, `contacto_canal` | `canalesConsentidos` (mapa canal → valor) se materializa en `contacto_canal`. |
| `Etiqueta` | `etiqueta` | `Cliente`–`Etiqueta` en `cliente_etiqueta`. |
| `Suscripcion`, `ExcepcionComercial` | `suscripcion`, `excepcion_comercial` | `suscripcion.cliente_id` es único y `excepcion_comercial_id` es opcional. |
| `TarifaRango` | `tarifa_rango` | |
| `Factura` y subclases | `factura`, `factura_rectificativa` | `mesFacturado` y `presupuestoOrigen` son columnas opcionales de `factura`. `UNIQUE(suscripcion_id, mes_facturado)` evita facturas recurrentes duplicadas. |
| `Concepto` | `concepto`, `factura_concepto` | La lista `conceptos` se materializa en `factura_concepto`, que guarda por línea descripción, cantidad, precio, descuento e impuesto. |
| `PeriodoAdministrativo` | `periodo_administrativo` | |
| `MovimientoCobro`, `Conciliacion` | `movimiento_cobro`, `conciliacion_cobro` | `movimiento_cobro_id` es único: un cobro tiene como máximo una conciliación. |
| `Tarea` y sus 5 subclases, `ComentarioIncidencia` | `tarea`, tablas hijas, `comentario_incidencia` | |
| `Movimiento` y subclases | `movimiento` y 13 tablas hijas | `RegistroAuditoria` → `registro_auditoria`, `ImportacionCSV` → `importacion_csv`, etc. El ER indica que la aplicación solo hace `INSERT` y `SELECT`, de acuerdo con `Repositorio<T>`. |
| `Notificacion` y canales | `notificacion` | Tabla única. |
| `Informe` | `informe` | `FormatosValidos` es la columna `formato`. |
| `AjustesSistema`, `AjusteSistemaPuntual`, `Festivo`, `Filtro` guardados | `ajustes_sistema` | Tabla clave-valor: cada `AjusteSistemaPuntual` es una fila (`clave`, `valor`, `tipo`, `fecha_efecto`, `estado`). `AjustesSistema` es el conjunto de filas y los festivos o filtros guardados son filas con su clave. |

### Clases sin tabla propia

- **Control y apoyo:** los 12 gestores, los repositorios, `Reloj`, `AutenticadorValidador` y `ExportadorArchivos` (comportamiento, no datos).
- **`Filtro<T>` y sus siete implementaciones:** son objetos de consulta. Solo los filtros guardados se persisten, como filas de `ajustes_sistema`.
- **`Archivo`, `CSV`, `PDF`:** los ficheros no se guardan. El PDF se genera al vuelo y de un CSV solo queda la traza (`importacion_csv`).
- **`AsignacionDesasignacionContacto`:** no tiene tabla (el ER tiene 13 tipos de movimiento y esta sería la decimocuarta).

### Tablas sin clase en el diagrama

- **Tablas de enlace sin clase propia:** `cliente_contacto`, `cliente_etiqueta`, `contacto_canal` y `factura_concepto`. Surgen de relaciones N a M, de un mapa y de una lista con atributos por línea.
- **`asignacion_cliente`** (historial de quién ha llevado cada cliente, CU-03): el diagrama la cubre con `Cliente.responsable`, `Usuario_Operativo.asignaciones` y los movimientos `AsignacionDesasigancionCliente`, pero no tiene clase propia. Conviene decidir si se añade al diagrama o se queda solo en el ER.
