# Casos de uso: Proyecto TurbineH

(Notas superficiales)

Grupo (A) Elefantes

Apuntes de los casos de uso de la aplicación. Todavía no es el diagrama ni la especificación: es la lista de "cosas que alguien consigue hacer con la aplicación", para repartir las fichas y hacer el diagrama sobre la misma base.

Esta versión está rehecha sobre los requisitos funcionales del documento "Análisis de Requisitos" (CLI-01, FAC-03, API-01...), que ya incluyen la reunión ampliada con el cliente.

Cómo leer cada caso de uso:

- **CU-01, CU-02...** son los identificadores. Son los mismos que se usan en el diagrama y en las fichas.
- **Entre corchetes** van los requisitos funcionales que cubre, por ejemplo [FAC-03].
- **Entre paréntesis** van los párrafos de la transcripción de la primera reunión de donde sale, por ejemplo (P088), cuando aplica.

---

## Quién usa la aplicación (actores)

- **Usuario operativo (UO):** empleado de TurbineH que trabaja solo sobre los clientes que tiene asignados. El cliente lo llamó primero "lector" y después dijo que también puede crear y editar (USU-02, AMB-02).
- **Administrador (AD):** puede hacer todo lo del usuario operativo, pero sobre todos los clientes. Además gestiona usuarios, asignaciones y aprobaciones, y ve el dashboard financiero (USU-02).
- **Superadministrador (SA):** es un administrador con acceso a la configuración profunda del sistema (USU-02, CFG-01).
- **Stripe:** sistema externo donde se reflejan los cobros. Para el proyecto es una simulación (PAG-01).
- **Sistema externo de TurbineH:** otro sistema de la empresa que pregunta por API si un cliente tiene facturas pendientes o avisa de que un cliente pide la baja (API-01).
- **Reloj del sistema:** dispara lo que ocurre solo cuando llega la fecha (facturas del día 1, recordatorios, conciliación automática).

Los tres perfiles van en escalera: el superadministrador es un administrador, y el administrador puede todo lo del usuario operativo. En el diagrama se puede dibujar como herencia entre actores.

**El cliente de TurbineH NO usa la aplicación.** Solo recibe facturas y avisos por correo (y WhatsApp simulado). No se registra, no entra, no consulta nada (P141, P147, P148).

---

## Casos de uso

### Acceso y usuarios

- **CU-01 Iniciar sesión.** Actores: UO, AD, SA. Con email del dominio de TurbineH y contraseña; no hay autorregistro. Incluye recuperar la contraseña con un enlace que caduca. La sesión se cierra por inactividad y se protege frente a intentos fallidos repetidos. [USU-01] (P125, P138, P139)
- **CU-02 Gestionar usuarios internos.** Actores: SA, AD con permiso. Crear o invitar empleados, darles un perfil (SA, AD o UO) y desactivarlos conservando todo lo que hicieron. [USU-01, USU-02] (P118, P119)
- **CU-03 Asignar clientes a un usuario.** Actor: AD. Asignar y quitar clientes; el nuevo responsable hereda las tareas pendientes. Permite delegaciones temporales que caducan solas y reasignaciones masivas (por ejemplo, los 20 clientes de alguien que se va), mostrando a qué afecta antes de confirmar. [USU-03, CLI-03, PRO-03] (P147)

### Clientes

- **CU-04 Registrar un cliente.** Actores: AD, UO con permiso. La empresa (nombre comercial, razón social, identificador fiscal, domicilio, país, moneda) y una o varias personas de contacto con su función. Distingue los datos imprescindibles para facturar de los convenientes y muestra un resumen antes de confirmar. Si el identificador fiscal ya existe, avisa; si es un cliente que vuelve, permite reactivar su ficha. [CLI-01, CLI-02, CLI-04] (P136, P137)
- **CU-05 Gestionar la ficha de un cliente.** Actores: AD, UO (sus clientes). Consultar la ficha con su línea temporal, modificar datos, añadir o desactivar contactos y poner etiquetas ("riesgo de impago", "cliente estratégico"). [CLI-01, CLI-02, CLI-05] (P081)
- **CU-06 Importar datos desde CSV.** Actor: AD. Clientes: con vista previa, mapeo de columnas, errores y aviso de duplicados; la importación queda registrada con un resumen. Histórico de facturas: entra como "migradas", sin enviarse ni consumir numeración. [IMP-01, FAC-07] (P141)
- **CU-07 Dar de baja a un cliente.** Actores: AD, UO (sus clientes). La baja es efectiva: desde esa fecha no se generan más mensualidades, pero el recobro de las facturas anteriores sigue. La ficha no se borra (borrado lógico). [SUB-04, CLI-03, CLI-04, AMB-04] (P098, P112, P176-P179)

### Suscripción, tarifas y aprobaciones

- **CU-08 Gestionar la suscripción de un cliente.** Actores: AD, UO (sus clientes). Registrar la relación comercial: inicio, plan, tarifa, responsable, condiciones de pago y documentos firmados. Permite una fecha de inicio futura, calcula el prorrateo del primer mes antes de confirmar (o aplica la excepción "facturar desde el mes siguiente") y no deja activar la recurrencia si faltan datos fiscales. También pausar y reactivar la suscripción con motivo y fechas. [SUB-01, SUB-02, SUB-03, SUB-04, AMB-05] (P110, P212)
- **CU-09 Revisar el rango anual de un cliente.** Actor: AD. El sistema lista los clientes a revisar; se introduce la facturación verificada, el sistema propone el rango y una persona lo aprueba antes de cambiar el precio. Queda registrado quién lo aprobó, cuándo, y el precio anterior y el nuevo. [TAR-02] (P113, P114, P146)
- **CU-10 Solicitar una excepción comercial.** Actores: UO, AD. Proponer un descuento temporal, un primer mes gratis o un precio pactado. Queda pendiente de aprobación. [APR-01]
- **CU-11 Aprobar o rechazar solicitudes.** Actores: AD; SA si se supera el umbral. Bandeja con todo lo que espera aprobación: excepciones comerciales, importes modificados a mano, vencimientos atípicos, facturas puntuales muy altas, rectificaciones y deudas incobrables. Nadie aprueba lo suyo cuando se exige separación de funciones. [APR-01, APR-02, FAC-06, REC-02]

### Facturas

- **CU-12 Crear una factura puntual.** Actores: AD, UO (sus clientes). Se crea como borrador, con una o varias líneas (concepto, cantidad, precio, descuento, impuesto) usando el catálogo de conceptos. El borrador se corrige libremente y se puede eliminar mientras no se haya emitido. [FAC-01, FAC-03] (P037, P192)
- **CU-13 Emitir una factura.** Actores: AD, UO (sus clientes). Muestra una vista previa; bloquea si falta un dato fiscal imprescindible y avisa si el importe se sale de lo habitual. Al emitir se asigna el número de la serie, se genera el PDF corporativo y se envía al contacto de facturación. Si necesita aprobación, queda pendiente (CU-11). [FAC-04, FAC-05, PDF-01, ENV-01, ENV-02, APR-02] (P043, P111, P150, P218)
- **CU-14 Generar las facturas recurrentes (automático).** Actor: Reloj del sistema. El día 1 de cada mes, a partir de las suscripciones activas. Si se reintenta tras una caída no duplica nada, y avisa al responsable de cada caso particular (sin contacto de facturación, pausado, tarifa nueva, baja...). [GEN-01, SUB-01] (P162, P212)
- **CU-15 Rectificar una factura emitida.** Actor: AD, con aprobación según importe. Una factura emitida no se edita ni se borra: se corrige con una rectificativa o nota de crédito vinculada a la original. [FAC-06, FAC-02, AMB-01]
- **CU-16 Consultar una factura y descargar su PDF.** Actores: AD, UO (sus clientes). Ver sus datos y su estado (borrador, emitida, pagada, anulada, rectificada, incobrable) y descargar el PDF tal como se emitió. [FAC-02, PDF-01] (P147, P150)

### Envío, cobros e impagos

- **CU-17 Resolver un envío fallido.** Actores: AD, UO (responsable del cliente). Si un correo rebota, la factura no cuenta como notificada y se abre una incidencia; se corrige el contacto y se reenvía sin duplicar la factura. [ENV-02, ENV-03]
- **CU-18 Enviar recordatorios y avisos de impago (automático).** Actor: Reloj del sistema. Un recordatorio hacia el día 3, otro al vencimiento (5 días por defecto) y avisos cada 2-3 días hasta el pago o hasta que intervenga una persona. Los plazos son configurables y los avisos que caen en festivo pasan al siguiente día laborable. [REC-01] (P088, P090, P158)
- **CU-19 Gestionar una disputa.** Actores: AD, UO (sus clientes). Suspender los recordatorios de una factura con motivo y fecha de revisión; mientras tanto sigue apareciendo como no cobrada. Cancelar la deuda o declararla incobrable solo con motivo y aprobación (CU-11). [REC-02]
- **CU-20 Conciliar cobros automáticamente.** Actores: Reloj del sistema, Stripe. El sistema consulta los pagos y recibe avisos. Si la coincidencia con una factura es clara, la marca como pagada y avisa al responsable. Si el importe no coincide, no la marca como pagada y señala la diferencia. Un aviso duplicado no se cuenta dos veces. [PAG-01, PAG-02, NOT-01] (P106-P108, P196)
- **CU-21 Conciliar un cobro a mano.** Actor: AD. Asociar a una factura los movimientos dudosos o sin referencia, o deshacer una conciliación dejando traza de quién y por qué. [PAG-02, PAG-03] (P123)

### Seguimiento del día a día

- **CU-22 Consultar la pantalla de trabajo diario.** Actores: AD, UO. Vencimientos, pagos por conciliar, problemas de entrega, aprobaciones, revisiones anuales, tareas y notificaciones (bandeja global para el AD, propia para el UO), con acceso directo a cada elemento. [DAS-01, NOT-01]
- **CU-23 Gestionar incidencias de facturación.** Actores: AD, UO (sus clientes). Abrir una incidencia sobre un cliente o una factura, con responsable, prioridad, comentarios internos y adjuntos. Puede pausar los recordatorios. [NOT-02]
- **CU-24 Consultar el dashboard financiero.** Actor: AD. Facturado, cobrado, pendiente, vencido, tiempo medio de cobro, mayores deudas, recurrente esperado y previsión, con acceso al detalle de cada cifra. [DAS-02] (P104, P130, P131)
- **CU-25 Generar y exportar informes.** Actores: AD, UO (solo sus clientes). Filtrar por periodo, cliente, responsable, país, sector, estado, tipo y moneda, y exportar a CSV y PDF respetando los permisos. Permite informes programados y un paquete periódico para la gestoría. [INF-01] (P141)
- **CU-26 Buscar en el sistema.** Actores: AD, UO. Búsqueda global por nombre comercial, razón social, NIF, contacto, número de factura o correo, con filtros, vistas guardadas, favoritos y recientes. [BUS-01]

### Configuración y supervisión

- **CU-27 Configurar el sistema.** Actor: SA. Plazos de pago y de recordatorios, tabla de tarifas, series de numeración, impuestos, umbrales de aprobación, canales, plantillas de correo, calendario de festivos y reglas de alerta. Los cambios se validan, se auditan y no reescriben lo ya emitido; se pueden simular antes de activarlos. [CFG-01, TAR-01, FAC-04, REC-01, NOT-01] (P134, P144, P151)
- **CU-28 Supervisar el sistema.** Actor: SA. Consultar la auditoría de acciones clave y la pantalla de salud (última generación recurrente, cola de correos, errores de integración, último backup), y restaurar una copia de seguridad. [AUD-01, PRO-01, PRO-05] (P207, P208)
- **CU-29 Cerrar un periodo administrativo.** Actor: AD; reabrirlo, solo con permiso superior y motivo. [PRO-04, AMB-01] (P084, P085)
- **CU-30 Gestionar los datos personales de un contacto.** Actor: AD. Localizar qué datos personales hay de un contacto y anonimizarlos o eliminarlos cuando proceda, conservando los PDF emitidos y los datos contables obligatorios. [PRI-01] (P100, P185)

### Acceso desde fuera

- **CU-31 Atender consultas por API.** Actor: Sistema externo de TurbineH. Consultar si un cliente tiene facturas pendientes y avisar de que un cliente pide la baja; ese aviso genera una tarea para el responsable, no borra la recurrencia. [API-01] (P203, P204, P212)

---

## Cobertura de los requisitos funcionales

Cada funcional del documento de requisitos y los casos de uso que lo cubren. Los 50 quedan cubiertos.

| Funcional | Casos de uso |
|---|---|
| CLI-01, CLI-02 | CU-04, CU-05 |
| CLI-03 | CU-03, CU-07 |
| CLI-04 | CU-04, CU-07 |
| CLI-05 | CU-05 |
| SUB-01 | CU-08, CU-14 |
| SUB-02, SUB-03 | CU-08 |
| SUB-04 | CU-07, CU-08 |
| TAR-01 | CU-27 |
| TAR-02 | CU-09 |
| APR-01 | CU-10, CU-11 |
| APR-02 | CU-11, CU-13 |
| FAC-01, FAC-03 | CU-12 |
| FAC-02 | CU-15, CU-16 |
| FAC-04 | CU-13, CU-27 |
| FAC-05 | CU-13 |
| FAC-06 | CU-11, CU-15 |
| FAC-07 | CU-06 |
| GEN-01 | CU-14 |
| PDF-01 | CU-13, CU-16 |
| ENV-01 | CU-13 |
| ENV-02 | CU-13, CU-17 |
| ENV-03 | CU-17 |
| REC-01 | CU-18, CU-27 |
| REC-02 | CU-11, CU-19 |
| PAG-01 | CU-20 |
| PAG-02 | CU-20, CU-21 |
| PAG-03 | CU-21 |
| NOT-01 | CU-20, CU-22, CU-27 |
| NOT-02 | CU-23 |
| DAS-01 | CU-22 |
| DAS-02 | CU-24 |
| INF-01 | CU-25 |
| BUS-01 | CU-26 |
| IMP-01 | CU-06 |
| USU-01 | CU-01, CU-02 |
| USU-02 | CU-02 |
| USU-03 | CU-03 |
| AUD-01 | CU-28 |
| CFG-01 | CU-27 |
| API-01 | CU-31 |
| PRO-01, PRO-05 | CU-28 |
| PRO-03 | CU-03 |
| PRO-04 | CU-29 |
| PRI-01 | CU-30 |
| USU-04, PRO-02 | Todos (ver abajo) |

**USU-04 y PRO-02 no tienen un caso de uso propio** porque son reglas que el sistema cumple en todos: rechazar accesos no autorizados y evitar duplicados por doble clic o edición a la vez. En las fichas aparecen como precondiciones o flujos de excepción.

---

## Cosas que NO son casos de uso

Las pidió el cliente, pero no son "algo que alguien hace con la aplicación", son cualidades. Están en los requisitos no funcionales (RM, PE, AA, INT, IND, SEG, LEG):

- Que todo se haga en pocos clics y en menos de un minuto.
- Que funcione en móvil, tablet y ordenador.
- Que aguante miles de clientes.
- Seguridad, copias de seguridad automáticas y conservación de datos.

Y cosas que quedan fuera del alcance:

- Pasarela de pago: la aplicación no cobra, solo comprueba (P074, P075).
- Chat dentro de la aplicación (P101).
- Tocar o integrar Brain (P092, P093).
- Verifactu (P146).
- Pagos fraccionados (P175, AMB-03).
- Las ampliaciones que el cliente dio como opcionales: fusión de fichas, presupuestos, centros de coste, varias sociedades emisoras, rol externo para la gestoría y funciones de IA.

---

## Decisiones

Alineadas con el documento de requisitos:

- **Modificar facturas:** se sigue AMB-01. El borrador se edita libremente; una factura emitida solo cambia con una rectificativa (CU-15); el cierre de trimestre es el cierre de periodo (CU-29).
- **Borrar facturas:** solo se eliminan borradores nunca emitidos (FAC-03). Una factura emitida no se borra.
- **Baja con deuda:** se sigue AMB-04. La baja es efectiva y el recobro de lo pendiente continúa (CU-07).
- **API:** entra como caso de uso (CU-31), porque el cliente la pidió en la reunión ampliada y está en API-01.
- **Superadministrador:** es un tercer perfil (USU-02).
- **Confirmación de pago al cliente:** no la envía nuestra aplicación; avisamos al responsable del cliente (CU-20).

Pendiente:

- **Fuente de la reunión ampliada:** para poder citar de qué párrafo sale cada caso de uso nuevo, igual que con la primera reunión.
