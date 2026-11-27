# Diagramas UML de secuencia

Este documento unifica la documentación de los diagramas de secuencia de TurbineH para los casos de uso CU-01 a CU-31. Se incluyen los flujos principales y los escenarios alternativos y de excepción relevantes.

El CU-22 no dispone de un diagrama independiente porque su comportamiento es principalmente visual. Algunas funcionalidades compartidas, como CU-25 y CU-29, aparecen en más de un bloque con distinto nivel de detalle.

## Selección de escenarios

Los escenarios se han seleccionado según su relevancia para representar la interacción entre actores, interfaces, gestores, entidades y repositorios. Los flujos alternativos y de excepción se integran mediante fragmentos `alt`, `opt`, `loop` y `ref` cuando corresponde.

| Bloque | Casos de uso | Escenarios representados |
|---|---|---|
| Acceso y clientes | CU-01 a CU-07 | Inicio y cierre de sesión, empleados, asignaciones, clientes e importación |
| Contratos y aprobaciones | CU-08 a CU-11 | Suscripciones, revisiones anuales, excepciones comerciales y aprobaciones |
| Facturación | CU-12 a CU-17 | Borradores, emisión, recurrencias, consulta, rectificación y reintentos |
| Seguimiento y cobros | CU-18 a CU-21 | Avisos de impago, disputas y conciliación |
| Operaciones transversales | CU-23 a CU-31 | Incidencias, dashboard, informes, búsquedas, configuración, supervisión, periodos, protección de datos y API |

Los procesos automáticos se inician desde `Reloj`. Las operaciones que modifican el estado del sistema registran el movimiento o la auditoría correspondiente mediante `GestorMovimientos`.

## Descripción de CU-01 - Inicio de sesión

Actores: CE. Requisitos: USU-01.

1. El empleado introduce su correo y contraseña en `VentanaLoginCierre`.
2. `GestorAcceso` busca el usuario en `RepositorioUsuarios` y ejecuta `autenticar()`.
3. Si las credenciales son correctas, actualiza el último acceso y `GestorElementosVisuales` construye la interfaz según el rol.
4. Si son incorrectas, muestra un mensaje genérico y permite reintentar, recuperar la contraseña o cancelar.
5. En la recuperación, se busca la cuenta, se envía un correo y se guarda la contraseña nueva mediante `cambiarPassword()`.

## Descripción de CU-01.5 - Cierre de sesión

Actores: CE y Reloj. Requisitos: USU-01.

1. El empleado selecciona cerrar sesión y confirma la acción.
2. `GestorAcceso` ejecuta `cerrarSesion()` y devuelve al usuario a `VentanaLoginCierre`.
3. El `Reloj` puede iniciar el cierre por inactividad según el valor de `tiempoInactividad`.
4. El abandono de la ventana o la pérdida de conexión también termina la sesión.
5. Si existe una operación pendiente o una conexión inestable, el cierre se aplaza hasta que pueda completarse.

## Descripción de CU-02 - Crear o invitar empleados

Actores: AD y SA. Requisitos: USU-01 y USU-02.

1. El administrador introduce el correo y la contraseña del empleado.
2. `GestorAcceso` comprueba los permisos y valida los datos mediante `AutenticadorValidador`.
3. Si el correo es nuevo, solicita el rol, crea el usuario y lo guarda.
4. Si pertenece a una cuenta desactivada, ejecuta `reactivar()` y recupera su historial.
5. Los datos erróneos, un correo duplicado activo o una cancelación impiden el alta.
6. La operación completada registra `AltaBajaUsuario` y la auditoría.

## Descripción de CU-02.5 - Desactivar empleados

Actores: AD y SA. Requisitos: USU-01 y USU-02. Referencia: CU-03.

1. El administrador selecciona al usuario y confirma su desactivación.
2. `GestorAcceso` busca la cuenta y consulta el límite de clientes.
3. Si el usuario tiene clientes, solicita un responsable receptor y ejecuta la migración de cartera del CU-03.
4. Ejecuta `desactivar()`, guarda el usuario y registra el movimiento.
5. Si ningún usuario puede recibir la cartera, se rechaza la desactivación.

## Descripción de CU-03 - Asignar clientes

Actores: AD, SA y Reloj. Requisitos: USU-03, CLI-03 y PRO-03.

1. El administrador selecciona un cliente y el usuario de destino.
2. `GestorAcceso` busca al usuario y comprueba el límite de clientes.
3. Si la asignación es válida, cierra la asignación vigente y crea la nueva.
4. En una migración masiva, `migrarCartera()` repite el proceso para todos los clientes del usuario de origen.
5. En una delegación temporal, `delegarTemporal()` crea una `DelegacionTemporal`.
6. Al caducar la delegación, el `Reloj` activa `devolverDelegacionesCaducadas()`.
7. Si el usuario original está desactivado, la asignación del delegado se convierte en permanente.
8. Cada cambio registra `AsignacionDesasignacion` y la auditoría.

## Descripción de CU-03.5 - Quitar asignaciones

Actores: AD y SA. Requisitos: USU-03, CLI-03 y PRO-03. Referencia: CU-03.

1. El administrador selecciona el cliente y confirma la eliminación de la asignación.
2. Introduce el nuevo responsable.
3. `quitarAsignacion()` valida el destino y cierra la asignación actual.
4. `asignarCliente()` completa la transferencia sin volver a solicitar los datos.
5. Si ningún usuario puede recibir al cliente, se mantiene la asignación existente.
6. La devolución de una delegación y la migración masiva reutilizan los flujos de CU-03.

## Descripción de CU-04 - Registrar un cliente

Actores: AD, SA y UO autorizado. Requisitos: CLI-01, CLI-02, CLI-04, USU-04 y PRO-02.

1. El usuario introduce los datos de la empresa y sus contactos.
2. `GestorClientes` comprueba los permisos y valida el NIF y los datos fiscales.
3. Si el NIF es nuevo, crea el `Cliente` y los `Contacto`.
4. Si el cliente estaba dado de baja o anonimizado, permite reactivarlo.
5. Los datos fiscales incompletos dejan la ficha marcada como no apta para facturación.
6. Opcionalmente se crea la suscripción durante el registro.
7. Los duplicados activos y las peticiones redundantes se rechazan.
8. El alta registra `AltaBajaCliente` y la auditoría.

## Descripción de CU-05 - Gestionar la ficha

Actores: AD, SA y UO autorizado. Requisitos: CLI-01, CLI-02, CLI-05, USU-04 y PRO-02.

1. `GestorClientes` comprueba el acceso y carga el cliente y su línea temporal.
2. El usuario puede consultar o editar la empresa, los contactos y las etiquetas.
3. `editarFicha()` valida y aplica los cambios.
4. Una edición simultánea, los datos inválidos o los contactos duplicados provocan el rechazo.
5. No se permite eliminar datos fiscales obligatorios mientras la facturación esté activa.
6. Los cambios se guardan y auditan.

## Descripción de CU-06 - Importar CSV

Actores: AD y SA. Requisitos: IMP-01, FAC-07, USU-04 y PRO-02.

1. El administrador selecciona un CSV de clientes o facturas históricas.
2. `GestorClientes` valida el archivo y presenta el mapeo de columnas y la vista previa.
3. Tras la confirmación, guarda los registros válidos.
4. Las facturas históricas se marcan como migradas, sin envío ni numeración activa.
5. Los registros erróneos o duplicados se omiten y aparecen en el resumen.
6. Un archivo corrupto o incompatible cancela la importación.
7. Se crea `ImportacionCSV` y se registra la auditoría.

## Descripción de CU-07 - Dar de baja a un cliente

Actores: AD, SA y UO autorizado. Requisitos: SUB-04, CLI-03, CLI-04, AMB-04, USU-04 y PRO-02.

1. El usuario selecciona la baja, indica su fecha efectiva y el motivo.
2. `GestorClientes` comprueba el permiso y que el cliente continúe activo.
3. `Cliente.darDeBaja()` realiza la baja lógica.
4. Se detiene la facturación recurrente futura, pero se conserva el seguimiento de facturas pendientes.
5. Si el flujo procede de una tarea creada por la API, `GestorTrabajo` la marca como resuelta.
6. Una petición duplicada no repite la operación.
7. La baja registra `AltaBajaCliente` y la auditoría.

## Descripción de CU-08 - Gestionar la suscripción

Actores: AD, SA y UO autorizado.

1. `GestorClientes` recupera al cliente y `GestorSuscripciones` consulta su suscripción vigente.
2. El usuario introduce el inicio, plan, tarifa, responsable, plazo y contrato.
3. Si procede, `Suscripcion` calcula el prorrateo del primer mes.
4. `AutenticadorValidador` comprueba los permisos, los datos fiscales y la vigencia de la tarifa.
5. Se crea y guarda la suscripción y se registra la auditoría.
6. La suscripción también puede pausarse y reactivarse.

## Descripción de CU-09 - Revisar anualmente el rango

Actores: AD, SA y aprobador.

1. Se listan las revisiones pendientes y se selecciona un cliente.
2. El usuario introduce la facturación anual verificada.
3. `GestorRevisionesAnuales` crea la revisión y obtiene las tarifas vigentes.
4. Si el tramo no cambia, guarda la revisión sin modificar el precio.
5. Si cambia, la revisión queda pendiente de aprobación.
6. El aprobador debe superar las comprobaciones de permisos y separación de funciones.
7. Una aprobación aplica la nueva tarifa; un rechazo conserva la anterior.

## Descripción de CU-10 - Solicitar una excepción comercial

Actores: UO, AD y SA.

1. Se consulta la suscripción y se muestran las excepciones disponibles.
2. El usuario introduce el tipo, valor, periodo y motivo.
3. `GestorSuscripciones` previsualiza el efecto económico.
4. Al confirmar, valida los permisos y la coherencia de los datos.
5. Crea `ExcepcionComercial` y solicita la aprobación correspondiente.
6. `GestorAprobaciones` determina al aprobador, guarda la solicitud, notifica y audita.
7. La cancelación previa no crea ningún registro.

## Descripción de CU-11 - Aprobar o rechazar solicitudes

Actores: AD y SA con rol de aprobador.

1. `GestorAprobaciones` lista las solicitudes y revisiones pendientes.
2. El aprobador abre una tarea y consulta su efecto económico.
3. Se validan el permiso, la separación de funciones y el umbral.
4. Según el tipo, se aplica la excepción comercial o la revisión anual.
5. Un rechazo exige un motivo.
6. La tarea puede reasignarse por ausencia.
7. El resultado se guarda, se notifica y se audita.

## Descripción de CU-12 y CU-13 - Crear y emitir una factura puntual

Actores: Usuario autorizado.

1. El usuario solicita un borrador, añade los conceptos y lo guarda.
2. Al confirmar la emisión, `GestorFacturas` comprueba los datos fiscales, el contacto y el periodo.
3. Si las condiciones son válidas, asigna el número y emite la factura.
4. Si falta una aprobación, remite al CU-11.
5. Si falla la entrega, remite al CU-17.
6. La factura emitida se guarda y se registra el movimiento.

## Descripción de CU-14 - Generar facturas recurrentes

Disparador: Reloj.

1. El `Reloj` inicia la facturación mensual.
2. `GestorFacturas` consulta las suscripciones que deben facturarse.
3. Para cada suscripción busca una factura del mes.
4. Si ya está emitida, no crea otra.
5. Si existe un borrador pendiente, lo retoma; de lo contrario, crea uno nuevo.
6. La emisión se completa mediante el flujo de CU-13.

## Descripción de CU-15 y CU-16 - Consultar, descargar y rectificar facturas

Actores: Usuario autorizado y administrador.

1. El usuario solicita una búsqueda y `GestorFacturas` recupera las facturas autorizadas.
2. Para una descarga, verifica el acceso y genera el PDF con los datos guardados.
3. Para una rectificación, comprueba los permisos y la validez de la corrección.
4. Crea una `FacturaRectificativa` vinculada a la original.
5. Tras emitirla, marca la factura original como rectificada.
6. La rectificación se guarda y se audita.

## Descripción de CU-17 - Resolver un envío fallido

Actores: Usuario autorizado.

1. Se registra el rebote y se abre una incidencia.
2. El usuario confirma o corrige el contacto y solicita el reenvío.
3. Se regenera el PDF de la factura y se intenta entregar.
4. Si la entrega se confirma, la incidencia se resuelve.
5. Si vuelve a fallar, permanece pendiente.
6. El resultado del intento se registra.

## Descripción de CU-18 - Generar avisos de impago

Disparador: Reloj.

1. `GestorCobros` obtiene los plazos configurados y las facturas pendientes.
2. Omite las facturas en disputa o con seguimiento pausado.
3. Determina el aviso y ajusta su envío al siguiente día laborable cuando procede.
4. Obtiene el contacto de impagos y comprueba el canal consentido.
5. `GestorNotificaciones` evita duplicados y crea la notificación y el envío.
6. La comunicación se guarda y se audita.

## Descripción de CU-19 - Gestionar una disputa

Actores: AD y UO autorizado.

1. El usuario selecciona una factura e introduce el motivo y la fecha de revisión.
2. `GestorCobros` valida el permiso y el motivo.
3. Crea `DisputaDeuda`, congela los recordatorios y guarda la disputa.
4. La deuda permanece pendiente durante la disputa.
5. La cancelación o declaración de deuda incobrable genera una solicitud de aprobación.
6. La apertura y los cambios se auditan.

## Descripción de CU-20 - Conciliar cobros automáticamente

Disparador: Reloj.

1. `GestorCobros` consulta el margen de conciliación y comprueba que el movimiento no esté duplicado.
2. Guarda el movimiento y busca facturas pendientes coincidentes.
3. Sin coincidencia clara, notifica al responsable y deriva al CU-21.
4. Con una diferencia fuera de margen, mantiene la factura pendiente y notifica la incidencia.
5. Con coincidencia válida, crea una conciliación automática y marca la factura como pagada.
6. El resultado se notifica y audita.

## Descripción de CU-21 - Conciliar un cobro manualmente

Actores: AD y UO autorizado.

1. `GestorCobros` lista los movimientos pendientes y sus facturas candidatas.
2. El usuario selecciona una factura.
3. `AutenticadorValidador` comprueba el permiso y la cartera.
4. `Factura.puedeConciliarse()` verifica que siga disponible.
5. Se crea una conciliación manual y la factura se marca como pagada.
6. Una reversión restaura el estado anterior y deja constancia del motivo.
7. La operación se audita.

## CU-22 - Caso sin diagrama independiente

El CU-22 se ha omitido como diagrama autónomo por su carácter principalmente visual. Sus efectos se representan mediante referencias desde los casos de uso que generan o atienden tareas y avisos.

## Descripción de CU-23 - Gestionar incidencias de facturación

Actores: CE. Requisitos: RG-08, REC-02 y NOT-02.

1. El usuario solicita la creación de una incidencia.
2. `GestorTareas` pide a `AutenticadorValidador` que compruebe los permisos y los datos.
3. Si son válidos, solicita a `GestorFacturas` los cambios necesarios.
4. Guarda la incidencia y registra el movimiento.
5. La cancelación o una validación incorrecta impiden la creación.

## Descripción de CU-24 - Consultar el dashboard financiero

Actores: SA. Requisitos: URS-03 y DAS-02.

1. El superadministrador solicita el dashboard financiero.
2. `GestorInformes` obtiene los datos financieros mediante `GestorCobros`.
3. Construye el dashboard y lo devuelve al superadministrador.

## Descripción de CU-25 - Generar, consultar y exportar informes

Actores: CE. Requisitos: INF-01, PRO-01 y RG-04.

1. El usuario consulta informes y aplica filtros.
2. Para generar uno nuevo, introduce el tipo y los criterios.
3. `GestorInformes` comprueba los permisos y obtiene los datos de clientes y facturas.
4. Si los datos son válidos, crea y guarda el informe.
5. La exportación puede realizarse inmediatamente en PDF o CSV, o programarse.
6. El `Reloj` activa las exportaciones cuando llega su fecha.
7. Una exportación masiva requiere aprobación y auditoría.
8. Los datos insuficientes, la falta de permisos o una aprobación rechazada cancelan la operación.

## Descripción de CU-26 - Buscar en el sistema

Actores: CE. Requisitos: USU-02, BUS-01, RG-02 y RG-04.

1. El usuario define filtros para clientes, facturas o informes.
2. El gestor correspondiente envía los filtros al repositorio.
3. El repositorio devuelve los elementos permitidos por el ámbito del usuario.
4. Si no hay resultados, la interfaz muestra un mensaje informativo.

## Descripción de CU-27 - Configurar el sistema

Actores: SA. Requisitos: CFG-01 y URS-07.

1. El superadministrador consulta los ajustes.
2. Propone un cambio mediante `GestorSistema`.
3. `GestorAprobaciones` crea la solicitud y `GestorNotificaciones` comunica su estado.
4. Si se aprueba, el cambio se aplica de forma inmediata o programada.
5. Los ajustes actualizados se guardan y el movimiento se registra.
6. Si se rechaza, la configuración permanece intacta.

## Descripción de CU-28 - Supervisar el sistema

Actores: SA. Requisitos: URS-03 y AUD-01.

1. El superadministrador solicita movimientos, tareas, aprobaciones y auditorías.
2. Los gestores correspondientes recuperan los datos mediante filtros.
3. El sistema construye la tabla de salud.
4. El superadministrador puede consultar las notificaciones relacionadas.
5. La reapertura de un periodo se deriva al flujo de CU-29.

## Descripción de CU-29 - Cerrar o reabrir un periodo administrativo

Actores: SA. Requisitos: PRO-04.

1. Tras obtener la aprobación necesaria, el administrador solicita el cierre.
2. `GestorSistema` comprueba el permiso y que el periodo esté abierto.
3. `PeriodoAdministrativo.cerrar()` cambia el estado.
4. `GestorInformes` genera el informe de cierre y `GestorMovimientos` registra la acción.
5. Para reabrirlo, se comprueban la autorización, el estado, la antigüedad y la copia previa.
6. Si se cumplen las condiciones, `PeriodoAdministrativo.reabrir()` restaura el estado operativo.
7. La falta de requisitos o de la copia previa impide la operación.

## Descripción de CU-30 - Anonimizar datos personales

Actores: AD y SA. Requisitos: URA-11, URS-03 y CLI-02.

1. El administrador busca al cliente y sus contactos mediante filtros.
2. Solicita la anonimización de uno o varios contactos o del cliente completo.
3. `GestorClientes` sustituye los datos personales sin eliminarlos físicamente.
4. Programa la fecha límite de conservación.
5. Guarda la actualización y registra el movimiento.

## Descripción de CU-30.5 - Borrar o reactivar clientes y contactos

Actores: AD, SA, Empresa y Reloj. Requisitos: API-01 y CLI-04.

1. En la eliminación manual, la Empresa emite una petición por API.
2. Tras la aprobación oficial, el administrador solicita el borrado definitivo.
3. `GestorClientes` elimina los datos mediante `RepositorioClientes`.
4. En la eliminación automática, el `Reloj` detecta el vencimiento del plazo legal.
5. Si el cliente vuelve antes de su eliminación, se recupera la ficha, se ejecuta `reactivarCliente()` y se cancela la fecha de borrado.

## Descripción de CU-31 - Atender consultas por API

Actores: Empresa. Requisitos: API-01, CLI-01, FAC-01 y USU-02.

1. La Empresa envía una consulta a la API.
2. `AutenticadorValidador` comprueba la autenticación, el formato y los permisos.
3. `GestorSistema` identifica el recurso y la operación solicitada.
4. Delega en los gestores de clientes, empleados, facturas o notificaciones.
5. Los gestores consultan o modifican sus repositorios y devuelven el resultado.
6. Si la consulta es inválida, no se ejecuta y se devuelve el error correspondiente.

## Trazabilidad

| Diagrama | Caso de uso | Participantes / responsabilidades |
|---|---|---|
| `CU-01.puml` | CU-01 | `VentanaLoginCierre`, `GestorAcceso`, `RepositorioUsuarios`, `Usuario`, `GestorElementosVisuales`, `Interfaz`, `Correo` |
| `CU-01.5.puml` | CU-01.5 | `Interfaz`, `VentanaConfirmacion`, `GestorAcceso`, `VentanaLoginCierre`, `Reloj`, `AjustesSistema` |
| `CU-02.puml` | CU-02 | Interfaces administrativas, `GestorAcceso`, `AutenticadorValidador`, `RepositorioUsuarios`, `Usuario`, `GestorMovimientos` |
| `CU-02.5.puml` | CU-02.5 | Interfaces administrativas, `VentanaConfirmacion`, `GestorAcceso`, `RepositorioUsuarios`, `AjustesSistema`, `Usuario`, `GestorMovimientos` |
| `CU-03.puml` | CU-03 | `FichaCliente`, `GestorAcceso`, `RepositorioUsuarios`, `AjustesSistema`, `AsignacionCliente`, `RepositorioClientes`, `GestorMovimientos` |
| `CU-03-temporal.puml` | CU-03 | `FichaCliente`, `GestorAcceso`, `DelegacionTemporal`, `AsignacionCliente`, `RepositorioUsuarios`, `Reloj`, `GestorMovimientos` |
| `CU-03.5.puml` | CU-03.5 | `FichaCliente`, `VentanaConfirmacion`, `GestorAcceso`, `RepositorioUsuarios`, `AsignacionCliente`, `GestorMovimientos` |
| `CU-04.puml` | CU-04 | `FichaCliente`, `GestorClientes`, `AutenticadorValidador`, `RepositorioClientes`, `Cliente`, `Contacto`, `GestorMovimientos`, `GestorSuscripciones` |
| `CU-05.puml` | CU-05 | `FichaCliente`, `GestorClientes`, `AutenticadorValidador`, `RepositorioClientes`, `Cliente`, `Contacto`, `Etiqueta`, `GestorMovimientos` |
| `CU-06.puml` | CU-06 | `VentanaImportacion`, `GestorClientes`, `AutenticadorValidador`, repositorios, `ImportacionCSV`, `GestorMovimientos` |
| `CU-07.puml` | CU-07 | `FichaCliente`, `VentanaConfirmacion`, `GestorClientes`, `RepositorioClientes`, `Cliente`, `GestorMovimientos`, `GestorTrabajo` |
| `CU-08.puml` | CU-08 | `FichaCliente`, `VentanaSuscripcion`, gestores de clientes y suscripciones, repositorios, `Cliente`, `Suscripcion`, `TarifaRango` |
| `CU-09.puml` | CU-09 | Ventanas de revisión y aprobación, gestores de revisiones, tarifas y suscripciones, `RevisionAnual`, `TarifaRango` |
| `CU-10.puml` | CU-10 | `VentanaSuscripcion`, `GestorSuscripciones`, `GestorAprobaciones`, `ExcepcionComercial`, repositorios y auditoría |
| `CU-11.puml` | CU-11 | `BandejaAprobaciones`, `VentanaAprobacion`, `GestorAprobaciones`, solicitudes, revisiones, usuarios y auditoría |
| `01-borrador-y-emision.puml` | CU-12 y CU-13 | Usuario, `GestorFacturas`, `AutenticadorValidador`, `FacturaPuntual`, `RepositorioFacturas`, `GestorMovimientos` |
| `02-facturacion-recurrente.puml` | CU-14 | `Reloj`, `GestorFacturas`, `Suscripcion`, `RepositorioFacturas`, `FacturaRecurrente` |
| `03-consulta-y-rectificacion.puml` | CU-15 y CU-16 | Usuario, `GestorFacturas`, `RepositorioFacturas`, `Factura`, `FacturaRectificativa`, `GestorMovimientos` |
| `04-entrega-y-reintento.puml` | CU-17 | Usuario, `EnvioComunicacion`, `IncidenciaFacturacion`, `GestorMovimientos` |
| `CU-18.puml` | CU-18 | `Reloj`, gestores de cobros, sistema y notificaciones, repositorios, `Cliente`, `Contacto`, `Notificacion`, `EnvioComunicacion` |
| `CU-19.puml` | CU-19 | `FichaCliente`, `VentanaDisputa`, gestores de cobros, facturas, clientes y aprobaciones, `DisputaDeuda`, repositorios |
| `CU-20.puml` | CU-20 | `Reloj`, gestores de cobros, sistema y notificaciones, `Conciliacion`, `Factura`, repositorios |
| `CU-21.puml` | CU-21 | `VentanaConciliacion`, `GestorCobros`, `AutenticadorValidador`, `Conciliacion`, `Factura`, repositorios y auditoría |
| `CU-23.puml` | CU-23 | Usuario, gestores y repositorios de tareas, facturas y movimientos, `AutenticadorValidador` |
| `CU-24.puml` | CU-24 | Superadministrador, `GestorInformes`, `GestorCobros` |
| `CU-25.puml` / `05-generacion-y-exportacion-de-informes.puml` | CU-25 | Usuario, `GestorInformes`, `AutenticadorValidador`, repositorios, `ExportadorArchivos`, `Reloj`, aprobaciones y auditoría |
| `CU-26.puml` | CU-26 | Usuario, gestores y repositorios de clientes, facturas e informes |
| `CU-27.puml` | CU-27 | Superadministrador, `GestorSistema`, `GestorAprobaciones`, `GestorNotificaciones`, tareas, `Reloj`, movimientos |
| `CU-28.puml` | CU-28 | Superadministrador, gestores y repositorios de tareas, notificaciones y movimientos, `GestorAprobaciones` |
| `CU-29.puml` / `06-cierre-y-reapertura-de-periodo.puml` | CU-29 | Administrador, `GestorSistema`, `PeriodoAdministrativo`, `AjustesSistema`, `GestorInformes`, `GestorMovimientos` |
| `CU-30.puml` | CU-30 | Administrador, `GestorClientes`, `RepositorioClientes`, `GestorMovimientos` |
| `CU-30.5.puml` | CU-30.5 | Administrador, Empresa, `GestorAprobaciones`, `GestorNotificaciones`, `GestorClientes`, `RepositorioClientes`, `Reloj`, `GestorSistema` |
| `CU-31.puml` | CU-31 | Empresa, `GestorSistema`, `AutenticadorValidador`, gestores y repositorios de empleados, clientes, facturas y notificaciones |

## Consistencia

Los diagramas siguen la arquitectura por capas de TurbineH:

1. Los actores interactúan con participantes de tipo boundary, como ventanas, fichas, interfaces o bandejas.
2. Las interfaces delegan la lógica en gestores de tipo control.
3. Los gestores coordinan las entidades y los repositorios.
4. Las entidades encapsulan las reglas y los cambios de estado.
5. Los repositorios representan la persistencia.
6. `GestorMovimientos` registra los movimientos y auditorías de las operaciones relevantes.
7. `AutenticadorValidador` centraliza las comprobaciones de permisos y las validaciones.
8. `Reloj` inicia los procesos automáticos.

Las operaciones de búsqueda se representan mediante métodos generales como `buscar()`, `buscarPorId()`, `buscarPorNif()` o `buscarFiltrado()`. Los criterios concretos se consideran implementaciones de `Filtro<T>` y no siempre aparecen como participantes independientes.

La creación de entidades se expresa mediante mensajes como "crear Usuario", "crear Suscripcion" o "crear Conciliacion" cuando el modelo de clases no define constructores explícitos. De forma equivalente, los mensajes dirigidos a las interfaces describen información mostrada o recogida, no necesariamente métodos del dominio.

La persistencia se limita a mensajes hacia los repositorios. Se omiten la base de datos, las consultas concretas y los detalles transaccionales por pertenecer al nivel de implementación.

Las referencias `ref` reutilizan comportamientos definidos en otros casos de uso, como la emisión de facturas, las aprobaciones, la migración de clientes, la devolución de delegaciones, los reintentos de envío y la generación de informes.

Existen las siguientes diferencias o decisiones de unificación:

- Las referencias antiguas a CU-05 para reasignar clientes se interpretan como referencias a CU-03.
- El cierre de sesión mencionado antiguamente como CU-02 se identifica como CU-01.5.
- La desactivación de empleados mencionada como CU-04 se identifica como CU-02.5.
- `GestorNotificaciones` se utiliza para las comunicaciones de CU-18 y CU-20, mientras que `GestorTrabajo` o `GestorTareas` se reserva para tareas e incidencias.
- CU-25 y CU-29 conservan los diagramas de ambas documentaciones porque representan alcances complementarios: uno general y otro centrado en escenarios concretos.
- CU-30 se divide en CU-30 y CU-30.5 para separar la anonimización del borrado definitivo y la reactivación.
- CU-22 queda representado mediante referencias desde los casos que producen o consumen sus elementos visuales.
- En la importación histórica, `GestorClientes` coordina el guardado en `RepositorioFacturas`, aunque la gestión ordinaria de facturas corresponda a `GestorFacturas`.
- La Empresa se modela como actor externo para las operaciones realizadas mediante API y para las aprobaciones corporativas.