# Requisitos

## 1. Contexto y alcance

### 1.1. Objetivo del sistema

Tras una reunión con la compañía Turbine, nos hablaron de que tenían el problema de que la gestión de las facturas de los clientes que contratan su servicio Brain actualmente es manual y que buscaban la forma de automatizarla para mejorar así su gestión administrativa, pero además, buscaban una solución sencilla y que pudiera ser gestionada sin muchas complicaciones por los empleados de la empresa.

El objetivo principal de nuestra aplicación es ofrecer una plataforma que permita gestionar la facturación a nuestros clientes y que automatice procesos que antes se hacían manualmente como el envío de las facturas, o la declaración de que se ha pagado o vencido el pago.

Hasta ahora, la empresa debía realizar manualmente todos los movimientos relacionados a la creación de una factura de un cliente, darle el formato establecido por la empresa, mandarlasela, enviarle notificaciones sobre el pago o un posible impago y guardarla en la base de datos, etc. Y el objetivo de nuestra aplicación sería poder automatizar todos esos procesos comunes a todos los clientes.

### 1.2. Alcance

Y se ha explicado que el objetivo principal de este sistema es automatizar y mejorar la gestión de las facturas emitidas por la empresa hacia los clientes.

En primer lugar se espera una integración de este sistema con la plataforma de pago de la empresa, Stripe, sin embargo, la plataforma nunca debe gestionar por su cuenta la realización de los pagos ni actuar como una pasarela de pago. Es un sistema encargado únicamente de la gestión de las propias facturas, pero no del pago de esta, y su integración con Stripe es necesaria para permitir la actualización del estado de una factura, detectar cuando alguna se haya pagado o si ha habido algún error en la cantidad recibida.

También es importante mencionar que esta aplicación se espera que sea totalmente interna de la empresa, pero que no esté relacionado con ningún módulo de la empresa ni con su producto principal, el Brain (salvo las integraciones imprescindibles, como la anterior), sino que funcione independiente de la funcionalidad de cualquier otro módulo de la empresa.

Hablando de la propia gestión de las facturas, de esta aplicación, no solo se espera automatización de los procesos recurrentes, sino también que la gestión sea lo más sencilla y rápida posible, manteniendo una interfaz simple al usuario que le permita realizar los procesos principales en unos pocos clicks.

Hablando ahora de procesos más específicos, la principal responsabilidad de este sistema es permitir la creación de factura asociadas a clientes, siendo estos clientes otras empresa que contratarán el servicio Brain, y que posteriormente la plataforma lleve un seguimiento automático de la fecha de vencimiento, del envío de la propia factura, del envío de notificaciones en las fechas establecidas y de los pagos realizados para saber si actualizar su estado a "pagada" o "impagada".

Pero además permite a los usuario, que son los empleados de Turbine que trabajen con esta plataforma, realizar ciertas modificaciones sobre esta y las facturas siempre y cuando se cumplan ciertas restricciones impuestas por la plataforma, empresa o requisitos legales.

Es decir, se espera principalmente que el sistema gestione automáticamente las facturas creadas por los usuario, pero que a su vez estos puedan realizar modificaciones sobre estas si lo ven pertinente.

Por otra parte también se espera que la plataforma se encargue automáticamente del almacenamiento de las facturas y los datos de los clientes en la base de datos de las empresa (Por lo que requerirá una integración con esta) que permita que los usuarios acceden y recuperen cualquier información anterior de la plataforma para posibles revisiones o notificaciones.

Lo anterior también se complementa con que la plataforma debe llevar un seguimiento a los movimientos relacionados por ella misma en diferentes niveles y que podrán ser vista por diferentes usuarios, siendo algunos de estos niveles, absolutamente todos los movimientos de la empresa, de las facturas que se declaran "pagadas" o "impagadas" y de los movimientos de un cliente o un usuario.

Y con ellos también se debe encargar de aportar opciones para filtrar la información a los usuario y mostrarla en un formato que mantenga la sencillez y rapidez detallada más arriba.

Con esto hemos visto el alcance de la aplicación en lo referente a la gestión de facturas, pero su otro pilar fundamental es la gestión de los usuario de la propia aplicación.

Esto se llevará a cabo con un sistema de roles que permitirá distinguir entre los roles Usuario y Administrador (Existiendo dentro de este el Super Administrador), cuya principal diferencia es que a los usuarios les serán asignados ciertos clientes por los administradores y solo podrán ver la información de esos clientes y realizar funciones sobre dichos clientes, pero además tendrán otras limitaciones en sus funcionalidades y los parámetros que puedan modificar, que también vendrán impuestas por los administradores.

Por otra parte los administradores son los que tendrán acceso a toda la información de la aplicación salvo una excepción y su función principal es controlar los registros generales de la plataforma y a sus usuarios, también pueden encargarse de clientes particulares pero la idea de este sistema es gestionar a los propios usuarios y dividirlos entre aquellos dedicados a las operaciones sobre un cliente y a la plataforma en general.

Y como se ha mencionado antes, existe una excepción a la que solo el súper administrador puede acceder, siendo estas una funciones más avanzadas o precondiciones que afectan a los procesos automatizados de esta y que solo el podrá modificar.

En resumen, la responsabilidad de esta plataforma es mejorar la gestión de las facturas de la empresa, mediante la automatización de procesos comunes a todas ellas, así como controlar y asignar roles a los usuarios que la usen para dividir las acciones que tengan que hacerse sobre la aplicación.

### 1.3. Actores y partes interesadas

A continuación se listaran a todos los actores y partes interesadas en el funcionamiento de la aplicación y se seguirá un formato donde se indica primero a actor en sí, una descripción sobre este y el interés que tenga por la aplicación, separado todo esto por un |

| Actor / stakeholder | Descripción | Intereses principales |
|---|---|---|
| Personal de la empresa | Empleados de Turbine destinados por la empresa a usar la aplicación para la gestión de facturas | Realizar la gestión de forma rápida, poder filtrar toda la información que se tenga sobre los clientes y que el sistema automatice los procesos más repetitivos como el envío de facturas y notificaciones. |
| Empleados con el rol usuario | Empleados de Turbine que usen la aplicación con el rol de usuario | Poder registrar clientes y gestionar a los que tengan asignados por los administradores de forma sencilla. |
| Empleados con el rol administrador | Empleados de Turbine que usen la aplicación con el rol de administrador | Poder gestionar todos los movimientos y clientes del sistema y tener notificaciones e historial de todo lo que ocurra en la aplicación. |
| Empleado con el rol super administrador | El empleado de la empresa con el rol de Super Administrador | Poder realizar todas las funciones que pueden hacer los administradores, es decir tener un control general sobre los movimientos del sistema, y poder gestionar las preconfiguraciones de la aplicación. |
| Clientes de la empresa | Empresa que solicitan los servicios de Turbine y se registraran en la aplicación | Esperan que el sistema les mande correctamente todas las facturas que generen por contratar el servicio de Turbine y que se registre correctamente cuando una factura se haya pagado. |
| Sistemas de pago | Los medios externos que gestionan los cobros de la empresa | Dar información que le permita al sistema determinar qué facturas se ha cobrado por completo o si alguna se ha cobrado parcialmente. |
| Departamento financiero | El equipo de la empresa dedicado a la gestión administrativa | Esperan que la aplicación refleje correctamente cualquier factura puntual que un empleado cree y actualice correctamente las facturas recurrentes para que no se escape ninguna factura y no se cree un problema por una factura duplicada. También seguimiento de los cobros pendientes y los impagos del trimestre y buen almacenamiento de todos los datos de la propia facturación que genere la empresa con el servicio que el sistema recopila. |
| Autoridades legales y equipo legal | Se refiere tanto al propio departamento legal de la empresa como en general a las propias autoridades exteriores de la empresa | Que la aplicación cumpla todas las leyes de protección de datos y privacidad de la normativa vigente del país donde se ubique la empresa. |
| Equipo de Marketing | El módulo de la empresa dedicado a la publicidad del servicio de la empresa | Que el sistema almacene (durante el máximo tiempo que la ley permite) información de los clientes que pueda usarse como incentivo en la publicidad del servicio. |
| Equipo técnico | Módulo de la empresa que pueda dedicarse al mantenimiento de la aplicación cuando esta esté terminada | Que el sistema esté bien documentado y sea estable y fácil de mantener. |
| Equipos de desarrollo externo al sistema | Módulos de la empresa dedicados al desarrollo de los servicios de la empresa externos al sistema | Que el sistema sea independiente y sus funciones o un fallo causado por estas no afecte a otros módulos de la empresa como el proyecto principal, el Brain. |
| Servicios de Correo | Los medios dedicados al envío de correos y notificaciones a los clientes | Una buena integración con el sistema que informe correctamente de los momentos donde haya que enviar un correo a un cliente y los periodos para enviar notificaciones de pago o impago. |


## 2. Requisitos de negocio

Los requisitos de negocio recogen lo que Turbine necesita como empresa para que la aplicación administrativa interna sea válida: reglas, políticas y restricciones que el sistema debe respetar con independencia de qué empleado lo use. Se redactan igual que los requisitos de usuario, pero aquí quien "espera" es la empresa.

Para los requisitos de negocio, Turbine como empresa es el único acto, por lo que la empresa exige su cumplimiento sea cual sea el rol que lo lleve a cabo.

Respecto a las entrevistas y reuniones donde se obtuvieron los requisitos:

- RI: Reunión Inicial: Denota la reunión original con el cliente donde nos presentó su producto y nos introdujo su problema con la gestión de facturas.

- RA: Reunión Ampliada: Denota la segunda reunión que se tuvo con los clientes donde se obtuvo más información sobre lo que se espera que el sistema haga.

### 1 Requisitos de negocio de gestión comercial (RNG)

#### 1.1 Clientes y relación comercial

Turbine trabaja con empresas que contratan un servicio y con las personas de contacto que trabajan en ellas. La empresa necesita que cada cliente esté registrado una sola vez, con información completa y con su historia intacta.

| ID | Requisito | Fuente |
|---|---|---|
| RNG-01 | Turbine necesita que de cada empresa cliente se guarde como mínimo el nombre comercial, la razón social, el NIF o identificador fiscal, el domicilio fiscal, el país, la moneda de trabajo y los datos de contacto | RA |
| RNG-02 | Turbine exige que una compañía aparezca una sola vez en el sistema aunque varias personas trabajen en ella, y que cuando una persona de contacto deje la compañía pueda marcarse como inactiva sin alterar el histórico de las facturas que se enviaron a su correo mientras trabajaba allí | RA |
| RNG-03 | Turbine necesita que el sistema detecte posibles duplicados por NIF y avise, tanto al registrar un cliente a mano como al importarlo | RA |
| RNG-04 | Turbine necesita registrar la relación comercial de cada cliente: cuándo empezó, qué servicio o plan tiene, cuál es la tarifa acordada, quién es su comercial o responsable interno y qué condiciones de pago tiene, pudiendo asociar además cualquier documento firmado fuera de la plataforma | RA |
| RNG-05 | Turbine exige que no se pueda dar de alta a un cliente si falta la información considerada más importante, para evitar problemas de comunicación en el futuro, y que la propia empresa pueda decidir qué datos son obligatorios y cuáles sensibles | RI |
| RNG-06 | Turbine necesita clasificar a sus clientes con etiquetas personalizadas (estratégico, beta, partner, riesgo, etc.) para poder agruparlos y valorar su riesgo de impago | RA |
| RNG-07 | Turbine necesita que las importaciones masivas de clientes o facturas permitan mapear columnas, ver errores y decidir si se corrige el fichero o se importan solo las filas válidas, y que cada importación quede registrada como una única operación con un resumen de registros creados, actualizados y rechazados y los motivos | RA |

#### 1.2 Facturación, cobro y casos especiales

La facturación es el núcleo del negocio. Turbine necesita que las facturas sean consistentes y trazables, que el cobro esté controlado y que los acuerdos particulares con clientes se reflejen sin romper el proceso general.

| ID | Requisito | Fuente |
|---|---|---|
| RNG-08 | Turbine distingue entre facturas recurrentes, que se generan automáticamente, y facturas puntuales, que se crean a mano, y necesita que el sistema permita tratar de forma distinta a los clientes con condiciones excepcionales | RI y RA |
| RNG-09 | Turbine exige que toda factura tenga un identificador único dentro de un sistema de series, controlado por el Super Administrador | RA |
| RNG-10 | Turbine exige que una factura emitida no pueda borrarse como si nunca hubiera existido, y que las correcciones o devoluciones se hagan con facturas rectificativas o notas de crédito vinculadas a la original, que requerirán aprobación si superan un umbral | RA |
| RNG-11 | Turbine exige que una factura solo pueda modificarse mientras esté vigente (no pagada), no haya terminado el trimestre en que se emitió y no se altere información sensible | RI |
| RNG-12 | Turbine necesita que antes de emitir una factura exista una vista previa con control de errores (datos faltantes o incorrectos) que permita bloquear la emisión de las que no cumplan los requisitos de la empresa, y que se genere siempre en el formato estándar de la empresa | RI y RA |
| RNG-13 | Turbine exige que la numeración, las operaciones monetarias críticas y la conciliación sean consistentes aunque dos personas actúen casi a la vez, detectando el conflicto o bloqueando la operación de una manera razonable | RA |
| RNG-14 | Turbine necesita un catálogo de productos ligero cuyos cambios no modifiquen nunca las facturas antiguas | RA |
| RNG-15 | Turbine necesita poder crear presupuestos previos a las facturas puntuales, registrar su aceptación o rechazo y convertir uno aceptado en factura reutilizando sus líneas y manteniendo el vínculo entre ambos | RA |
| RNG-16 | Turbine necesita que el plazo de pago sea configurable y que los vencimientos y recordatorios que caigan en día no laborable pasen al siguiente día laborable, según un calendario con fines de semana y festivos locales | RI y RA |
| RNG-17 | Turbine necesita que cada movimiento bancario (importe, fecha, moneda, concepto y referencia) se asocie a una factura: si la coincidencia es clara, el sistema la propone o la concilia automáticamente según la política definida, y si hay dudas debe decidir una persona | RA |
| RNG-18 | Turbine necesita un registro de impagos de todos los clientes con opciones de filtrado, y un sistema de incidencias de facturación (NIF incorrecto, pago no localizado, etc.) que se puedan asignar y resolver, con la opción de pausar los recordatorios mientras estén abiertas, de modo que la pausa esté vigilada y no quede activa indefinidamente | RI y RA. Véase URA-09. |
| RNG-19 | Turbine permite perdonar deudas, pero exige que toda condonación sea revisada y aprobada por un administrador | RA |
| RNG-20 | Turbine necesita poder acordar con un cliente que empiece a facturarse el mes siguiente sin cobrar prorrateo: hasta esa fecha no deben aparecer facturas ni recordatorios, pero el cliente debe verse como pendiente de activación | RA |
| RNG-21 | Turbine necesita poder acordar una pausa temporal con un cliente: durante la pausa no se generan mensualidades, pero el cliente sigue existiendo y puede conservar facturas antiguas pendientes. La pausa debe tener motivo, fecha de inicio y, si se conoce, fecha prevista de reactivación, y al llegar esa fecha el sistema avisará al responsable sin asumir nada importante si la política exige confirmación | RA |
| RNG-22 | Turbine exige que, cuando un cliente cause baja, se dejen de generar mensualidades desde la fecha de baja, pero que el cobro de las facturas anteriores siga activo si las hay. Si no tiene facturas pendientes, la recurrencia se cierra en la fecha efectiva | RI y RA |
| RNG-23 | Turbine necesita poder asociar documentos de soporte (presupuestos, pedidos, justificantes de cobro) a clientes y facturas, controlando formatos, tamaños y permisos de acceso | RA |

### 2 Requisitos de negocio de control interno (RNC)

#### 2.1 Roles y aprobaciones

El sistema de roles es un pilar para la empresa. Turbine necesita controlar quién puede hacer qué y que las decisiones con impacto económico las tome la persona adecuada, sin perder la automatización.

| ID | Requisito | Fuente |
|---|---|---|
| RNC-01 | Turbine exige que el acceso se organice en tres roles jerárquicos (Usuario o lector, Administrador y Super Administrador), y que los cambios de rol, permisos y asignaciones se apliquen de forma dinámica y rápida | RI |
| RNC-02 | Turbine exige que el Usuario solo pueda ver y operar sobre los clientes que un administrador le haya asignado, con permisos que puedan limitarse función a función, mientras que el Administrador tiene acceso global y solo se ve limitado por las funciones reservadas al Super Administrador | RI |
| RNC-03 | Turbine exige que solo el Super Administrador pueda acceder a las funciones avanzadas: configuración global, tabla de tarifas, series de facturación, umbrales de aprobación y umbral de sensibilidad de los datos | RI |
| RNC-04 | Turbine exige que no exista autorregistro: solo el Super Administrador, o un administrador con permiso para ello, puede crear o invitar a nuevos usuarios | RA |
| RNC-05 | Turbine necesita que un administrador pueda delegar clientes a otra persona durante un intervalo sin perder la asignación principal, y que la delegación caduque al terminar, evitando así que se compartan contraseñas | RA |
| RNC-06 | Turbine exige que determinados cambios económicos (descuentos, rectificativas, perdón de deudas y cambios de tarifa) sean aprobados por un administrador, y que si superan un umbral configurado los apruebe el Super Administrador, sin que esto impida que el sistema siga siendo automático | RA |
| RNC-07 | Turbine necesita que los administradores puedan hacer un seguimiento particular de determinados clientes, recibiendo directamente sus notificaciones sin perder el acceso al resto | RI |
| RNC-08 | Turbine exige transparencia con sus empleados: cada uno debe poder consultar en la propia aplicación sus funciones, sus limitaciones, el motivo de estas y el historial de cambios en sus permisos y asignaciones | RI |

#### 2.2 Tarifas y configuración

Las reglas del negocio cambian con el tiempo y Turbine necesita poder ajustarlas sin comprometer lo ya emitido.

| ID | Requisito | Fuente |
|---|---|---|
| RNC-09 | Turbine aplica a cada cliente una tarifa según su rango de facturación anual verificada, y exige que los cambios en la tabla de tarifas, que solo hace el Super Administrador, se apliquen a nuevas revisiones sin reescribir nunca las facturas históricas | RA |
| RNC-10 | Turbine necesita que a principios de año el sistema liste las empresas cuyo rango debe revisarse, que una persona valide el dato de facturación antes de proponer el nuevo rango, y que el responsable apruebe la propuesta antes de aplicarse, con efecto solo a futuro | RA |
| RNC-11 | Turbine necesita que el Super Administrador pueda configurar los plazos por defecto, los dominios autorizados, los datos fiscales del emisor, los impuestos, las series y los umbrales de aprobación, y que cualquier cambio (por ejemplo, pasar el vencimiento de cinco a siete días) afecte solo a lo nuevo y se aplique al instante o al mes siguiente según se decida | RI y RA |
| RNC-12 | Turbine exige un entorno de prueba o simulación para previsualizar plantillas y reglas de recordatorio antes de activar automatizaciones que puedan resultar peligrosas | RA |
| RNC-13 | Turbine necesita fijar objetivos de facturación mensuales o anuales y ver su progreso en el panel de administración | RA |

### 3 Requisitos de negocio de comunicación e integración (RNI)

Turbine se comunica con sus clientes por el canal que estos autorizan, y necesita que otros sistemas de la empresa puedan consultarle o avisarle.

| ID | Requisito | Fuente |
|---|---|---|
| RNI-01 | Turbine necesita que los usuarios con permiso puedan editar las plantillas de comunicación (facturas, recordatorios, impagos, disputas y revisiones) en español e inglés, con variables dinámicas y previsualización | RA |
| RNI-02 | Turbine exige que los documentos y alertas se envíen por el canal preferido y autorizado del contacto (correo electrónico o WhatsApp) verificando su consentimiento explícito, y que el medio por defecto lo defina el Super Administrador siempre que sea compatible con los datos obligatorios del cliente | RA y RI |
| RNI-03 | Turbine exige que cada intento de envío, su resultado y sus reintentos queden registrados, y que se pueda ver el estado de los mensajes (en cola, enviado, fallido o reintentado) sin asumir nunca confirmaciones de lectura | RA |
| RNI-04 | Turbine exige que, si un envío rebota, la factura no se marque como entregada, se avise al responsable del cliente, se deje una incidencia de entrega y se permita corregir el contacto y reenviar | RA |
| RNI-05 | Turbine necesita que los empleados puedan dejar comentarios y notas privadas en facturas y fichas de cliente, con menciones a otros empleados que generen notificaciones internas | RA |
| RNI-06 | Turbine necesita que sistemas externos (como Brain) puedan consultar por API si un cliente tiene facturas pendientes, y que el sistema pueda recibir por API solicitudes de baja y generar el flujo interno de cierre sin eliminar el cobro de lo pendiente | RA |
| RNI-07 | Turbine exige una buena integración del sistema con los elementos fundamentales de la empresa, como la base de datos o la pasarela de pago, manteniendo a la vez su independencia de cualquier otro módulo, de forma que su actividad no afecte a otros ni se vea afectada por ellos | RI |

### 4 Requisitos de negocio de seguridad y continuidad (RNS)

Al manejar datos fiscales y económicos, Turbine necesita poder demostrar quién hizo qué, cumplir la normativa y no depender de un fallo técnico.

#### 4.1 Seguridad, cumplimiento y continuidad

| ID | Requisito | Fuente |
|---|---|---|
| RNS-01 | Turbine exige un registro de auditoría inalterable de las acciones críticas (altas, modificaciones de datos fiscales, emisiones, conciliaciones, reasignaciones y cambios de configuración), conservando el valor anterior y posterior de cada dato para poder corregir cambios hechos por error (véase RG-03) | RI y RA |
| RNS-02 | Turbine exige que el acceso se haga con usuario y contraseña, que las sesiones inactivas caduquen, que se pueda cerrar sesión de forma explícita y que se invaliden las sesiones de los usuarios desactivados | RI y RA |
| RNS-03 | Turbine necesita poder atender una solicitud legal de eliminación o anonimización de los datos de un contacto en las fichas activas, preservando intactos los documentos y el historial de las facturas ya emitidas | RA |
| RNS-04 | Turbine necesita que el Super Administrador reciba notificaciones de los movimientos destacados (cambios de permisos, altas y bajas, pagos, impagos e incidencias) | RI |
| RNS-05 | Turbine exige copias de seguridad automáticas de datos y documentos con una política definida, con posibilidad de restaurarlas y de probar la restauración antes de necesitarla | RA |
| RNS-06 | Turbine necesita poder cerrar períodos (meses o trimestres) para que no se modifiquen facturas ni datos económicos sin un permiso superior que quede auditado, y generar para cada periodo un paquete para la gestoría (CSV o ZIP) con las facturas emitidas, rectificativas, cobros y documentos | RA |
| RNS-07 | Turbine necesita poder consultar la salud del sistema: estado del generador recurrente, colas de correo, integraciones y últimas copias de seguridad | RA |

#### 4.2 Información e informes

| ID | Requisito | Fuente |
|---|---|---|
| RNS-08 | Turbine necesita una búsqueda global que localice información por nombre comercial, razón social, NIF, contactos, número de factura o correo, y que permita guardar filtros habituales (por ejemplo, "mis impagados de más de 15 días") | RA |
| RNS-09 | Turbine necesita poder ejecutar acciones masivas (reasignar responsables, cambiar el sector o exportar varios clientes) con confirmación previa y un resumen de resultados | RA |
| RNS-10 | Turbine exige que los informes puedan exportarse al menos a CSV y que exista un informe legible en PDF | RA |
| RNS-11 | Turbine necesita que la vista inicial refleje lo más importante del momento (facturas pendientes de pago, clientes con problemas e impagos recientes) y que las notificaciones lleguen de forma clara y puedan filtrarse | RA |


## 3. Requisitos de usuario

«El sistema» es la aplicación administrativa interna de Turbine:

En cada requisito se indicará el actor que espera que se cumpla el requisito y la fuente de donde se ha obtenido esta entrevista, pero debido al sistema de roles que será lo primero que se explicará en la lista de requisitos, y solo haber habido 2 entrevistas diferentes, estos serán los identificadores que usaremos:

Para los actores:

- CE: Cualquier empleado: Denota a cualquier usuario de Turbine interesado en usar la aplicación y se emplea para los requisitos que detallan acciones del sistema que cualquier usuario de cualquier rol espera que llevé a cabo. Opcionalmente en esos caso puede haber una subclasificación donde se indiquen particularidades y limitaciones que dentro de este requisito tenga cada rol.

- US: Usuario (En futuras conversaciones se empezó a referirse a este como "lector") Será el primero de los 3 roles que se detallarán más adelante y el nivel más bajo en cuanto a funcionalidades, suele depender de los otros 2.

- AD: Administrador: El nivel superior al usuario que no tendrá limitaciones en cuanto a funcionalidad salvo un caso particular.

- SAD: Super Administrador: Es uno de los administradores que además de lo que estos pueden hacer tendrá acceso a acciones aún más avanzadas.

Respecto a las entrevistas y reuniones donde se obtuvieron los requisitos:

- RI: Reunión Inicial: Denota la reunión original con el cliente donde nos presentó su producto y nos introdujo su problema con la gestión de facturas.

- RA: Reunión Ampliada: Denota la segunda reunión que se tuvo con los clientes donde se obtuvo más información sobre lo que se espera que el sistema haga.

A continuación comenzará el desglose de los requisitos de usuarios:

### 1 Requisitos asociados al sistema de roles:

Antes de comenzar es importante mencionar que la presencia de un sistema de roles no ha sido considerada como tal un requisito de usuario, pues el tener un sistema de roles no es una acción que espere un usuario que el sistema realice.

Sin embargo, debido a la importancia que el cliente le dio durante las reuniones, se espera que los empleados de la compañías esperan ciertas funciones a realizar por el sistema en función del rol que se les asigne. Por ello es que se propondrá un esquema de este sistema de roles, comenzando cada una con una descripción de dicho rol y a continuación una serie de requisitos de usuario asociados a este rol. Se indicará el actor con el identificador de ese rol aunque parezca redundante debido a acciones esperadas parecidas entre varios roles.

#### 1.1 Usuarios:

Referidos en posteriores reuniones como "lectores" este rol denota a empleados que tendrán unos permisos limitados sobre la cantidad de funciones que podrán realizar en el sistema, determinados por los administradores.

Esas limitaciones se refieren a que el usuario tendrá una serie de clientes asignados (por un administrador) y solo podrá ver o registrar facturas de dichos clientes, así como realizar ciertos cambios o gestionar notificaciones de dichos clientes (pudiendo limitarse esto también) o directamente una serie de funciones bloqueadas por completo cómo asociar un cliente a un usuario.

| ID | Actor | Requisito | Fuente |
|---|---|---|---|
| URU-01 | US | Poder acceder a las facturas no declaradas todavía como "pagadas" de los clientes que tenga asignados, es decir, a las facturas todavía vigentes | RI |
| URU-02 | US | Poder generar nuevas facturas de los clientes que tenga asignados cuando esta no pueda procesarse automáticamente (véase por ejemplo que se le asocie un cliente puntual, cuyas facturas se hacen a mano, o iniciar la recurrencia de uno recurrente) | RI |
| URU-03 | US | Poder registrar nuevos clientes siempre y cuando un administrador le haya dado permiso para ello | RI |
| URU-04 | US | Poder modificar las facturas de sus clientes asignados siempre y cuando se cumplan las restricciones de modificación que estas tienen (estar vigente, no haber terminado el trimestre donde se emitió la factura, etc.) y siempre y cuando no se modifique información sensible (Con un umbral de la sensibilidad decidido por el Super Administrador) | RI |
| URU-05 | US | Filtrado y conocimiento de la información relevante de las facturas y el historial de los clientes asignados | RI |
| URU-06 | US | Poder realizar chequeos a sus clientes o facturas de estos de forma rápida (con pocos clicks). |  |
| URU-07 | US | Que el sistema guarde la suficiente información que la empresa pueda necesitar para la comunicación o los chequeos y que bloquee los intentos de registros donde falte la información que se considere más importante (Para que no haya problemas de comunicación en el futuro) | RI |
| URU-08 | US | Espera que el sistema sea capaz de gestionar automáticamente los proceso comunes a todos sus clientes, como el envío de la facturas, y generación automática de las recurrentes, envío de notificaciones de pago/impago, conversión de las facturas al formato estándar de la empresa, envío de notificaciones al superadministrador, etc | RI |
| URU-09 | US | De la misma forma espera la posibilidad de descargar y recuperar información de las facturas de sus clientes (También sujeto a limitaciones impuestas de los administradores) | RI |
| URU-10 | US | Espera transparencia y claridad en la explicación de sus funciones y limitaciones, con una buena documentación de las funciones que sí tiene permitido realizar y aquellas limitaciones y el porqué se le restringen esas funciones, para facilitar conversaciones en la empresa en caso de ser necesaria (véase por ejemplo, te lo impide un administrador, es algo que solo puede hacer el superadministrador, etc.) | RI |
| URU-11 | US | El usuario espera estar asignado por defecto a un cliente y recibir notificaciones cuando algún administrador le dé o quite permisos para ciertas acciones | RI |
| URU-12 | US | El usuario espera que a partir del historial de movimientos globales de la plataforma poder conocer su historia de asignaciones o restricciones de permisos, es decir, poder conocer las cambios de permisos y asignaciones que ha tenido desde que se le registró como un usuario en la plataforma. Esta información también se espera si se cambia el rol del usuario por si fuera necesaria (véase que un cliente se queda sin nadie que se encargue de él y se le da a otro usuario). |  |
| URU-13 | US | El usuario espera un información y alineamiento entre los clientes que tiene asignados y los permisos que tiene, y las funciones que puede realizar, es decir espera poder conocer sobre qué clientes puede hacer modificaciones y cuales puede hacer (y por supuesto que eso se alinee con lo que la aplicación al final le deje). |  |
| URU-14 | US | El usuario espera también automatización, sobre el guardado, borrado y modificado de cualquier dato en la base de datos llevado a cabo por sus acciones permitidas. |  |

#### 1.2 Administrador:

Es el nivel superior a los usuarios, los que no tienen permisos limitados ni dependen de asignaciones a clientes para poder realizar funciones sobre estos, pueden realizar cualquier función sobre cualquier cliente, y solo se ven limitados por las funciones más avanzadas que son llevadas a cabo por el Super Administrador u otras configuraciones que limitan algunas acciones discretas.

Así como el usuario posteriormente se refirió como el "lector", pese a que ya se ha dicho que si tiene algo de capacidad de modificación, es porque esta estaba limitada a sus clientes asignados y dentro de esto, mientras que el administrador tiene acceso global, y no se dedica solo a "leer y comprobar".

| ID | Actor | Requisito | Fuente |
|---|---|---|---|
| URA-01 | AD | El administrador espera poder dar de alta clientes en la plataforma sin restricciones de ningún tipo. |  |
| URA-02 | AD | El administrador espera tener acceso a todas las facturas vigentes de todos los clientes registrados en el sistema | RI |
| URA-03 | AD | El administrador espera poder generar facturas de todos los clientes que estén registrados o vayan a registrarse en la plataforma siempre y cuando está necesite crearse manualmente | RI |
| URA-04 | AD | El administrador espera poder modificar facturas vigentes de todos los clientes siempre y cuando se cumplan las múltiples restricciones de las modificaciones (La propia vigencia, o sea que no se haya pagado, haya pasado el trimestre o se modifiquen datos sensibles) | RI |
| URA-05 | AD | El administrador espera poder asignar o quitar la asignación de clientes a usuarios para habilitarlos a poder gestionar las facturas vigentes de estos | RI |
| URA-06 | AD | El administrador espera poder cambiar los permisos de los usuarios para limitar las funciones que estos pueden realizar sobre sus clientes (Poder modificar o no una factura de su cliente, qué valores puede modificar, etc.) y controlar los movimientos más importantes que estos realicen | RI |
| URA-07 | AD | El administrador espera poder acceder al registro de movimientos globales de la aplicación, aquel que registra todas las nuevas facturas, registro de clientes, facturas pagadas/vencidas, cambios en los usuarios, etc. | RI |
| URA-08 | AD | El administrador espera que el sistema informe no sólo sobre lo más esencial, como los clientes y empleados registrados, así como el valor de las preconfiguraciones, y de las capacidades y limitaciones que este rol tiene (como requisito no funcional se verá que se espera sencillez en este apartado) | RI |
| URA-09 | AD | El administrador espera que el sistema pueda mostrarle un registro de los impagos de todos los clientes, así como las correspondientes opciones de filtrado pertinentes | RI |
| URA-10 | AD | El administrador espera que el sistema sepa gestionar automáticamente todos los procesos comunes a todos los clientes, como la renovación automática de las facturas puntuales, crear la factura tanto en la aplicación como en el formato estándar de la empresa que se le envía al cliente, así como el envío de notificaciones de pago o impago | RI |
| URA-11 | AD | El administrador espera que el sistema le permita correctamente acceder al registro de todos los clientes en todos tipo de ámbitos (su registro particular, los movimientos relacionados a ellos en el de movimientos, sus impagos en el de los impagos, etc.) y que no sufra ningún tipo de restricción (salvo en las funciones del super Administrador) | RI |
| URA-12 | AD | Así como un acceso al historial global de movimientos, el administrador espera la existencia y acceso a un historial de movimientos propios que permita otra forma de ver a qué cliente está asociado cada usuario o si algún cliente se ha quedado sin usuario asignado al quitar la asignación a uno que estaba antes | RI |
| URA-13 | AD | El administrador espera la posibilidad (y facilidad) de poder descargar y recuperar la información de todos los clientes, ya sea una descarga de algún documento puntual como un pdf con una factura o acceder al registro de un cliente y ver datos como el teléfono y persona de contacto | RI |
| URA-14 | AD | El administrador espera poder dar de baja a los clientes sin restricciones y siempre cumpliendo las condiciones de la plataforma (no se da de baja hasta que no tenga facturas vigentes pero se le deja de mandar la factura recurrente) | RI |
| URA-15 | AD | El administrador también espera automatización en todo lo referente al guardado y borrado de los datos del cliente en la base de datos | RI |
| URA-16 | AD | El administrador espera que el sistema le permita asociarse a él o a otros administradores a ciertos clientes o que al menos estos puedan gestionar y estar al pendientes de las acciones en cuanto al envío de facturas de estos. Al no tener restricciones como el usuario la asignación del administrador parece más contradictoria, pero aunque este pueda seguir accediendo a todos los movimientos y todas las notificaciones enviadas por los clientes, el sistema debería permitir que los administradores realicen un seguimiento a clientes en particular. La diferencia sería que las notificaciones acerca de estos particulares le llegaría a él directamente, mientras que las otras no, pero las podría seguir viendo | RI |
| URA-17 | AD | De la misma forma, al igual que a un usuario, el administrador puede asignarse particularmente a varios clientes, el sistema debe permitirle, además de ver la información general, los clientes a los que les haya hecho seguimiento particular | RI |

#### 1.3 Super Administrador:

Es el último nivel, pero a diferencia de los anteriores no es un rol completamente nuevo, pues el super administrador realmente es un administrador que además de todo lo que estos pueden hacer, puede acceder a funciones más avanzadas que el resto de administradores no, como por ejemplo y más importante cambiar las preconfiguraciones que tiene la plataforma sobre varios elementos importantes como el número de días que tiene un cliente para pagar una factura o el medio por el que se envían las facturas y notificaciones.

Por ello es que sus requisitos de usuario realmente son los del administrador más algunos particulares que vamos a poner a continuación, pero de los de administrador vamos a recalcar algunos, por las leves diferencias que puedan tener para un super administrador:

| ID | Actor | Requisito | Fuente |
|---|---|---|---|
| URS-01 | SAD | El super administrador espera que el sistema le permita registrar clientes, crear nuevas facturas, modificar las facturas ya existentes (siguiendo siempre las restricciones) o dar de baja a los clientes (también siguiendo el procedimiento que se aplica al dar de baja a los clientes) | RI |
| URS-02 | SAD | El Super Administrador espera poder cambiar los clientes que tiene asignado cada usuario así como las limitaciones sobre las funciones que tiene asignadas cada usuario | RI |
| URS-03 | SAD | El super administrador espera que el sistema lleve y le permita acceder a un registro con todos los clientes, los empleados registrados y el registro particular de cada uno de estos, un historial de todos los movimientos de la aplicación, un historial de los impagos y los cobros y el historial de cada cliente y de notificaciones a administradores y usuarios. En general, que a diferencia de los otros 2, este pueda acceder a absolutamente toda la información en cuanto a los movimientos en la aplicación se refiere, de diferentes formas según se quiera ver algo más general o específico y con las opciones de filtrado correspondiente | RI |
| URS-04 | SAD | El super administrador espera que el sistema le muestra (y de forma sencilla) todas las funciones que puede realizar en la aplicación tanto para el mismo como en los registros particulares del resto de empleados, así como de los valores actuales de las preconfiguraciones que él y solo él podrá cambiar | RI |
| URS-05 | SAD | El super usuario espera que el sistema automatice todos los proceso comunes a todos los clientes, como las factura recurrentes o el envío de notificaciones | RI |
| URS-06 | SAD | El super administrador espera recibir notificaciones de cualquier movimiento destacado de la aplicación como lo serían cambiar los permisos de un usuario, dar de alta o baja a un cliente o empleado, un pago, un impago o un problema que haya ocurrido, la diferencia con un requisito anterior, es que el super administrador podrá acceder al historial de todos los movimientos que también ven los administradores pero al le llegarán notificaciones informando de esos movimientos | RI |
| URS-07 | SAD | El super administradores espera que el sistema le permite cambiar las configuraciones o preconfiguraciones de la aplicación, permitiendo cambiar algunos de los valores más importantes, como los datos que debe dar un cliente, los datos considerados "sensibles" y que pueden limitar a usuarios y hasta administradores o serán obligatorios para dar de alta a un cliente, el número de días que tiene un cliente para pagar su factura, la periodicidad de los envíos de notificaciones al cliente, todo lo relacionado a permisos de usuario y roles de cada empleados, modificación de facturas (sujeto a restricciones y cambios que se aplicarán al instante o al mes siguiente), el medio por donde se enviarán las facturas (se le impondrá que sea uno que con los datos obligatorios sea posible) e incluso modificar algunos datos más sensibles. |  |

### 2 Requisitos de usuarios generales:

Una vez visto los requisitos asociados a los roles, veremos ahora algunos que son compartidos por todos los empleados, recogiendo incluso lo que son compartidos por los 3 roles pero con diferencias. Para ellos usaremos el identificador de empleado CE.

| ID | Actor | Requisito | Fuente |
|---|---|---|---|
| RG-01 | CE | El empleado espera que la aplicación le permite loguearse mediante su usuario y su contraseña, sin necesidad de una información más interna o adicional | RI |
| RG-02 | CE | El empleado espera que el sistema guarde todos los datos necesarios para el registro, comunicación y seguimiento de cualquier cliente y miembro que use la aplicación. También opciones para ver a todos los clientes registrados (cumpliendo con las restricciones de su rol) para evitar duplicados en los registros | RA |
| RG-03 | CE | Seguridad y protección de los datos, complementado con algunos requisitos anteriores, el usuario espera un registro de los movimientos principales en los que se vea involucrado y especialmente de cualquier dato que pueda ser modificado antes y después de la modificación para poder arreglar sobreescrituras y cambios realizados por error | RI |
| RG-04 | CE | Cualquier empleado espera la implementación de sus funciones asignadas para su rol y la rápida ejecución de los cambios sobre estos que sea realizada por un ente externo a este empleado. Es decir, debido a que el sistema de roles es un pilar fundamental, la implementación y aplicación dinámica de las múltiples funciones y restricciones de cada rol es algo que cualquier empleado esperará del sistema al conocer su rol por información externa o de la propia aplicación | RI |
| RG-05 | CE | El empleado espera que el sistema permita distinguir entre las facturas generadas usualmente y aquellas que pueden cumplir alguna excepción y requieran un trato distinto, y la acciones necesarias para ello, como poner en pendiente de activar a alguien que se registre a finales de mes pero empiece a pagar a partir del siguiente | RA |
| RG-06 | CE | El empleado espera que el sistema implemente todas las opciones necesarias para dar la alta y baja a los clientes y para la generación, revisión y envío de las facturas en el formato decidido en la empresa | RI y RA |
| RG-07 | CE | Vista previa, el usuario espera que el sistema pueda generar una vista previa de las facturas que este pueda revisar para poder bloquear la emisión en alguna que no cumpla los requisitos de la empresa | RA |
| RG-08 | CE | Notificaciones, el empleado espera que las notificaciones les lleguen de forma clara y que el sistema permite filtrarlas y acceder a ellas fácilmente para controlar facturas incorrectas, no pagadas del todos, cambios que hay que aprobar o cambios de rol y funciones | RA |
| RG-09 | CE | Personalización ligera, el empleado espera ciertas opciones de personalización de la interfaz y la cuenta de su usuario, esto refiriéndose al posible mapeo de columnas, el filtrado, opciones varias como el idioma o hasta entornos de prueba individuales y simulados | RA |
| RG-10 | CE | Comunicación interna, no mediante un chatbot interno sino mediante elementos más sutiles como comentarios de los empleados y etiquetas que caractericen a los clientes en función de su riesgo de impagos y los datos que haya aportado y no | RA |

### 3 Requisitos de usuario no directamente funcionales

En esta sección se van a incluir una serie de requisitos de usuario no directamente relacionados a funcionalidades concretas del sistema, sino a restricciones y formas de comportamientos del sistema que el usuario espera al usarlo. Varios de estos requisitos están relacionados a algunos de los requisitos no funcionales, lo que se verá en la trazabilidad.

| ID | Actor | Requisito | Fuente |
|---|---|---|---|
| RNDF-01 | CE | El empleado espera que el sistema tenga una buena integración con elementos fundamentales de la empresa para el correcto funcionamiento de esta, como la base de datos o la pasarela de pago | RI |
| RNDF-02 | CE | Relacionado al anterior el usuario espera automatización en los procesos comunes a todos los clientes y para los cuales ha sido diseñada esta aplicación, como lo son el envío de facturas o los múltiples historiales | RI |
| RNDF-03 | CE | Portabilidad y simplicidad, el empleado espera no solo poder acceder al sistema desde cualquier dispositivo que suela usar para trabajar sino además que el sistema se mantenga en todo momento simple y rápido de usar especialmente para acciones más comunes y estándar, como generar una factura o dar de alta un cliente | RI |
| RNDF-04 | CE | Independecia, debido al carácter de esta aplicación, el usuario espera que el sistema funcione de forma independiente de cualquier otro módulo de la empresa y que su actividad no afecte a la de cualquier otro donde este se halle involucrado ni viceversa | RI |
| RNDF-05 | CE | Documentación, el usuario espera transparencia, claridad y documentación de todas las funciones que el sistema puede realizar, y esto se hará a partir de identificadores en el propio sistema, como con una descripción del sistema de roles o la simplicidad de la interfaz que no dé lugar a error en lo que produce cada acción | RI |
| RNDF-06 | CE | Vista inicial, el empleado espera que la vista inicial que observa al entrar en la aplicación refleje las facturas y movimientos más importantes de ese momento como las pendientes a pagar, no pagadas del todo, clientes con problemas o impagos recientes, para poder acceder rápidamente a ellos | RA |


## 4. Requisitos del sistema

### 4.1. Requisitos funcionales

Los requisitos funcionales describen comportamientos, servicios u operaciones que
debe proporcionar el sistema. Siempre que sea posible, deben formularse de manera
clara, precisa y verificable.

«El sistema» es la aplicación administrativa interna de Turbine.

- SA = superadministrador

- AD = administrador

- UO = usuario operativo

#### 1. Clientes y contactos

| ID | Requisito | Relacionado con |
|---|---|---|
| CLI-01 | El sistema deberá permitir crear, modificar y consultar empresas cliente con nombre comercial, razón social, identificador fiscal, domicilio, país, moneda y contacto, sin asumir formato nacional del identificador ni estructura fija de dirección, distinguiendo los datos imprescindibles para facturar de los meramente convenientes y proponiendo valores por defecto con un resumen antes de confirmar el alta. |  |
| CLI-02 | El sistema deberá permitir varias personas de contacto por empresa con su rol (contratación, facturación, impagos), una misma persona en varias empresas y marcar contactos como inactivos sin perder el histórico de envíos, avisando si se intenta eliminar el único contacto de facturación de un cliente activo. |  |
| CLI-03 | El sistema deberá asignar un responsable interno a cada cliente y gestionar los estados del cliente que afectan a la facturación, con consecuencias explícitas en cada cambio. |  |
| CLI-04 | El sistema deberá aplicar borrado lógico a los clientes, permitir reactivar la ficha histórica de un cliente que vuelve y avisar de posibles duplicados por identificador fiscal sin fusionar fichas automáticamente. |  |
| CLI-05 | El sistema deberá mostrar una línea temporal de negocio por cliente y permitir etiquetas libres que no sustituyan a los estados. |  |

#### 2. Relación comercial (suscripción)

| ID | Requisito | Relacionado con |
|---|---|---|
| SUB-01 | El sistema deberá representar la relación comercial separada de las facturas (inicio, plan, tarifa, responsable, condiciones de pago, documentos firmados asociados), sin permanencias, y generar a partir de ella las facturas recurrentes. |  |
| SUB-02 | El sistema deberá gestionar fechas de inicio futuras, sin facturas ni recordatorios hasta esa fecha, y bloquear la activación de la recurrencia si faltan datos fiscales, indicando cuáles. |  |
| SUB-03 | El sistema deberá calcular y mostrar el prorrateo del alta antes de confirmar, con una única regla documentada (meses de longitud variable, alta el último día del mes), y permitir la excepción «facturar desde el mes siguiente sin prorrateo». |  |
| SUB-04 | El sistema deberá permitir pausas (con motivo, fecha de inicio y fecha prevista de reactivación) y bajas, sin generar mensualidades durante la pausa ni desde la fecha efectiva de baja y manteniendo el recobro de las facturas anteriores. |  |

#### 3. Tarifas y revisión anual

| ID | Requisito | Relacionado con |
|---|---|---|
| TAR-01 | El sistema deberá permitir al SA gestionar como configuración los rangos de facturación anual y sus precios, garantizando que los cambios no alteren facturas históricas y rigen desde su fecha de efecto. |  |
| TAR-02 | El sistema deberá gestionar la revisión anual de rango (listar clientes a revisar, aceptar el dato verificado con referencia y fecha, proponer el rango) y exigir validación humana antes de cambiar el precio, registrando quién aprueba, cuándo y precio anterior y nuevo, sin emitir facturas retroactivas automáticamente. |  |

#### 4. Descuentos y aprobaciones

| ID | Requisito | Relacionado con |
|---|---|---|
| APR-01 | El sistema deberá gestionar excepciones comerciales (descuento temporal, primer mes gratis, precio pactado) que un UO proponga, un AD apruebe y el SA apruebe si superan un umbral configurable. |  |
| APR-02 | El sistema deberá dejar pendiente de aprobación, explicando el motivo, los importes modificados a mano, descuentos extraordinarios, vencimientos atípicos, facturas puntuales muy altas y rectificaciones según importe, con bandeja de aprobaciones, sin autoaprobación cuando se exija separación de funciones y con reasignación o suplente registrados. |  |

#### 5. Facturas

| ID | Requisito | Relacionado con |
|---|---|---|
| FAC-01 | El sistema deberá gestionar facturas recurrentes y puntuales con varias líneas (concepto, cantidad, precio, descuento, impuesto), moneda de emisión conservada, catálogo ligero de conceptos e impuestos configurables, sin que los cambios de catálogo afecten a facturas existentes. |  |
| FAC-02 | El sistema deberá gestionar un ciclo de vida de la factura donde «pagada» solo signifique cobro detectado o validado y sea distinta de anulada, rectificada e incobrable. |  |
| FAC-03 | El sistema deberá permitir corregir una factura antes de emitirla, hacer trazable todo cambio relevante posterior sin alteraciones silenciosas y eliminar solo borradores nunca emitidos. |  |
| FAC-04 | El sistema deberá asignar al emitir un número único dentro de la serie (configurable por el SA), también con operaciones concurrentes, y usar como fecha legal la fecha de negocio y no la hora del servidor. |  |
| FAC-05 | El sistema deberá mostrar una vista previa antes de emitir, bloquear la emisión si falta un dato fiscal imprescindible indicando cuál y advertir, sin bloquear, si el importe se aparta del patrón del cliente según una regla configurable. |  |
| FAC-06 | El sistema deberá ofrecer un flujo de rectificación o nota de crédito vinculado a la factura original, con permiso administrativo y aprobación según importe, y calcular el saldo del cliente considerando las rectificaciones. |  |
| FAC-07 | El sistema deberá permitir importar histórico de facturas clasificadas como «migradas», sin envíos ni consumo de numeración. |  |

#### 6. Generación recurrente

| ID | Requisito | Relacionado con |
|---|---|---|
| GEN-01 | El sistema deberá generar automáticamente las facturas recurrentes el día 1 de cada mes de forma idempotente, recuperando tras una caída las no generadas sin duplicar, aislando los errores por cliente y comunicando cada caso particular (sin contacto de facturación, pausado, tarifa nueva, baja efectiva, condiciones a 30 días, descuento caducado) a quien pueda resolverlo. |  |

#### 7. PDF

| ID | Requisito | Relacionado con |
|---|---|---|
| PDF-01 | El sistema deberá generar el PDF desde los datos existentes con identidad corporativa, datos fiscales, numeración, líneas, impuestos, totales, condiciones de pago y contacto de soporte, y conservar el PDF emitido, o poder reproducirlo igual, tras cambios de logo o plantilla. |  |

#### 8. Envío y entrega

| ID | Requisito | Relacionado con |
|---|---|---|
| ENV-01 | El sistema deberá enviar automáticamente las facturas, por defecto por correo al contacto de facturación, con WhatsApp simulado como canal complementario y consentimiento registrado por canal, usando plantillas editables con variables en español e inglés según cliente o contacto. |  |
| ENV-02 | El sistema deberá registrar de cada envío destinatarios, canal, resultado y fecha, distinguiendo el original de los reintentos, permitir reintentar sin duplicar la factura y mostrar el estado de los mensajes (en cola, enviado, entregado, fallido, reintentado) sin inventar la lectura. |  |
| ENV-03 | El sistema deberá gestionar el rebote de un correo: no marcar la factura como notificada, avisar al responsable, abrir una incidencia y permitir corregir el contacto y reenviar. |  |

#### 9. Recordatorios, impagos y disputas

| ID | Requisito | Relacionado con |
|---|---|---|
| REC-01 | El sistema deberá enviar, tras la factura, un recordatorio hacia el día 3, otro al vencimiento (5 días por defecto) y avisos de impago cada 2-3 días hasta el pago o la intervención de una persona, con plazos configurables por el SA, prevaleciendo el vencimiento pactado con el cliente y trasladando al siguiente día laborable los avisos que caigan en festivo. |  |
| REC-02 | El sistema deberá permitir suspender los recordatorios de una factura (disputa) con motivo y fecha de revisión, manteniéndola visible como no cobrada, y cancelar o declarar incobrable una deuda sólo mediante acción controlada, con motivo y aprobación. |  |

#### 10. Pagos y conciliación

| ID | Requisito | Relacionado con |
|---|---|---|
| PAG-01 | El sistema deberá incluir una integración Stripe simulada que permita consultar pagos y recibir notificaciones, sin contabilizar dos veces un evento duplicado. |  |
| PAG-02 | El sistema deberá modelar la conciliación con independencia de la fuente (Stripe, Revolut, banco), con importe, fecha, moneda, concepto y referencia, conciliando automáticamente según la política cuando la coincidencia sea clara, pasando a conciliación manual los casos dudosos o sin referencia y, si el importe difiere del debido, sin marcar la factura como pagada y señalando la diferencia. |  |
| PAG-03 | El sistema deberá permitir deshacer una conciliación conservando la traza (quién o qué regla la hizo, quién la revirtió y por qué) y mostrar, para cada factura pagada, el movimiento que la justifica. |  |

#### 11. Notificaciones e incidencias

| ID | Requisito | Relacionado con |
|---|---|---|
| NOT-01 | El sistema deberá notificar al responsable del cliente los eventos relevantes (pago, vencimiento, rebote, disputa, revisión anual, aprobación rechazada), con bandeja global para el AD y propia para el UO, estados leída y atendida con prioridades, canal configurable por el SA y reglas de alerta configurables sobre un conjunto cerrado de condiciones. |  |
| NOT-02 | El sistema deberá permitir incidencias de facturación sobre un cliente (y opcionalmente una factura) con responsable, estado, prioridad, resolución y posibilidad de pausar recordatorios, además de comentarios internos con menciones y dos niveles de visibilidad y adjuntos con tipo y descripción, con acceso según los permisos del cliente o la factura. |  |

#### 12. Trabajo diario, dashboard, informes y búsqueda

| ID | Requisito | Relacionado con |
|---|---|---|
| DAS-01 | El sistema deberá ofrecer una pantalla de trabajo diario con vencimientos, pagos por conciliar, problemas de entrega, aprobaciones, revisiones anuales y tareas asignadas, con acceso directo al elemento. |  |
| DAS-02 | El sistema deberá ofrecer a los administradores un dashboard financiero (facturado, cobrado, pendiente, vencido, tiempo medio de cobro, mayores deudas, recurrente esperado, previsión simple) con acceso al detalle de cada cifra. |  |
| INF-01 | El sistema deberá permitir informes filtrables por periodo, cliente, responsable, país, sector, estado, tipo y moneda, exportables a CSV y PDF respetando los permisos del usuario, registrando fecha, filtros y usuario, auditando las exportaciones masivas y permitiendo informes programados y un paquete periódico para la gestoría. |  |
| BUS-01 | El sistema deberá ofrecer búsqueda global por nombre comercial, razón social, identificador fiscal, contacto, número de factura y correo, con filtros, vistas guardadas, favoritos y recientes, sin usarlos como mecanismo de autorización. |  |

#### 13. Importación

| ID | Requisito | Relacionado con |
|---|---|---|
| IMP-01 | El sistema deberá importar clientes desde CSV con vista previa, mapeo de columnas, errores y aviso de duplicados, registrando cada importación como operación con resumen de creados, actualizados y rechazados. |  |

#### 14. Usuarios, roles y accesos

| ID | Requisito | Relacionado con |
|---|---|---|
| USU-01 | El sistema deberá autenticar con correo del dominio autorizado y contraseña, sin autorregistro, con altas por el SA o un AD con permiso, recuperación de contraseña con enlace caducable, cierre de sesión por inactividad, protección frente a intentos fallidos repetidos y desactivación de usuarios conservando sus acciones e invalidando sus sesiones. |  |
| USU-02 | El sistema deberá prever tres perfiles (SA, AD, UO) con un modelo de autorización centralizado, donde el UO opera sobre sus clientes asignados sin poder asignárselos, cambiar tarifas globales, crear series ni ver el dashboard global. |  |
| USU-03 | El sistema deberá permitir al AD asignar y desasignar clientes, heredando el nuevo responsable las tareas pendientes y conservando el histórico su autor original, y permitir delegaciones temporales con caducidad automática. |  |
| USU-04 | El sistema deberá rechazar en el servidor el acceso a documentos y URL no autorizados. |  |

#### 15. Auditoría y configuración

| ID | Requisito | Relacionado con |
|---|---|---|
| AUD-01 | El sistema deberá registrar en una auditoría no editable las acciones clave (alta de cliente, datos fiscales, emisión, vencimientos, aprobaciones, reasignaciones, disputas, conciliaciones manuales, configuración), consultable y filtrable por el SA. |  |
| CFG-01 | El sistema deberá separar la configuración global (plazos, dominio, datos fiscales del emisor, logo, series, monedas, impuestos, umbrales, canales) de los datos operativos, sin fijar los datos del emisor en el código, validando y auditando los cambios sin reescribir documentos ya emitidos y permitiendo simular las automatizaciones antes de activarlas. |  |

#### 16. API e integraciones

| ID | Requisito | Relacionado con |
|---|---|---|
| API-01 | El sistema deberá exponer una API documentada, con autenticación separada, permisos, límites y registro, que incluya la consulta de obligaciones pendientes de un cliente y la notificación «cliente solicita baja» (que genera una tarea sin borrar la recurrencia), sin saltarse las reglas de la interfaz y con las integraciones encapsuladas. |  |

#### 17. Procesos automáticos y operación

| ID | Requisito | Relacionado con |
|---|---|---|
| PRO-01 | El sistema deberá registrar cada ejecución de los procesos automáticos, alertar si una tarea no se ejecuta, ejecutar en segundo plano las operaciones largas con estado persistente y ofrecer una pantalla de salud (última generación recurrente, cola de correos, errores de integración, último backup). |  |
| PRO-02 | El sistema deberá impedir duplicados por doble clic o recarga en emisión, envío y conciliación, y detectar conflictos de edición concurrente sobre una factura. |  |
| PRO-03 | El sistema deberá permitir acciones masivas mostrando cuántos elementos afectan, con confirmación si son sensibles y resultado de éxitos y errores, sin «marcar como pagadas» masivo. |  |
| PRO-04 | El sistema deberá permitir el cierre administrativo de periodos, con reapertura solo con permiso superior y motivo. |  |
| PRO-05 | El sistema deberá realizar copias de seguridad automáticas de datos y documentos y permitir su restauración. |  |

#### 18. Datos personales

| ID | Requisito | Relacionado con |
|---|---|---|
| PRI-01 | El sistema deberá permitir localizar los datos personales de un contacto y anonimizarlos o eliminarlos cuando proceda, conservando el PDF emitido y los datos contables obligatorios. |  |

#### Ampliaciones (fuera del alcance obligatorio)

El cliente presentó como opcionales o futuras estas funciones, que no se incluyen como requisitos: fusión de fichas de clientes, presupuestos, centros de coste o proyectos, varias sociedades emisoras, importe convertido para reporting, objetivos de facturación y cobro, rol externo para la gestoría y funciones de IA opcionales.

#### Ambigüedades y decisiones

- **AMB-01:** **Modificación de facturas.** En la reunión inicial el cliente dijo que las facturas son modificables hasta el cambio de trimestre; en la ampliada, que se corrigen antes de emitir y después son trazables o rectificables. Interpretación propuesta: el borrador se edita libremente; la factura emitida solo cambia mediante rectificación (FAC-03, FAC-06); el cierre del trimestre se trata como cierre administrativo de periodo (PRO-04). A confirmar con el cliente.

- **AMB-02: Rol «lector» / UO.** El cliente lo describió primero como solo lectura y luego como capaz de editar y crear sobre sus clientes asignados. Interpretación propuesta: el rol se define por capacidad y no por nombre, con edición limitada a los clientes asignados (USU-02).

- **AMB-03:** **Pagos parciales.** El cliente excluye el pago fraccionado como modalidad comercial, pero pide detectar importes distintos del debido. Interpretación propuesta: no existe pago fraccionado; el sistema solo detecta la diferencia, no marca la factura como pagada y la deja para revisión (PAG-02).

- **AMB-04:** **Baja con deuda.** En la reunión inicial el cliente dijo que no se puede dar de baja con facturas impagadas; en la ampliada, que la baja es efectiva pero el recobro continúa. Interpretación propuesta: prevalece la reunión ampliada (SUB-04).

- **AMB-05: Regla de prorrateo.** Interpretación propuesta: se factura por días naturales, contando el día del alta, sobre los días reales del mes (importe mensual × días restantes ÷ días del mes). Un alta el último día del mes factura un día (SUB-03). A confirmar con el cliente.

### 4.2. Requisitos no funcionales

En este apartado se recogen aquellos requisitos que no responden tanto a "qué" hace el programa sino a "cómo" lo ejecuta, así como la definición de sus estándares en cuanto a la calidad y el rendimiento además de sus restricciones.

#### 1 - Rendimiento y Mantenibilidad

Forma en la que se comporta la aplicación en respuesta a una cierta carga de trabajo,es decir, qué tan bien un sistema o red puede manejar el procesamiento de datos. Puede medirse en términos de tiempo de respuesta del sistema, utilización de recursos, etc.

| ID | Categoría | Requisito | Criterio verificable |
|---|---|---|---|
| RM-01 | Rendimiento y Mantenibilidad | El sistema debe funcionar con fluidez y rapidez sin comprometer cualquier otro aspecto del sistema, como puede ser, la seguridad de los datos. |  |
| RM-01.1 | Rendimiento y Mantenibilidad |  | Las tareas normales deben poder efectuarse en un minuto. Ej: Generación y consulta de facturas. |
| RM-01.2 | Rendimiento y Mantenibilidad |  | Las operaciones pesadas se ejecutan en segundo plano. |
| RM-02 | Rendimiento y Mantenibilidad | En relación con el tema de la auditoría y la revisión humana de modificaciones en el sistema, a pesar de esto, la automatización del sistema no debe ser afectada. |  |
| RM-02.1 | Rendimiento y Mantenibilidad | No todo necesita una revisión, habrá cosas que el sistema podrá permitir directamente. |  |
| RM-03 | Rendimiento y Mantenibilidad | Hablando de las autorizaciones que se dan entre los diferentes roles del sistema, el código que lo maneje debe seguir un modelo coherente. Es decir, el sistema debe estar diseñado de manera que su rendimiento no se vea comprometido. |  |
| RM-04 | Rendimiento y Mantenibilidad | Búsquedas eficientes según la carga del sistema. | Las búsquedas se resolverán por debajo de los 10 segundos para cargas grandes de trabajo. En circunstancias normales, serán prácticamente inmediatas. |
| RM-05 | Rendimiento y Mantenibilidad | El sistema no conservará para siempre lo generado. Al fin y al cabo, lo contrario puede llenar/saturar el servidor y se busca la eficiencia de los recursos. | ciertos generables sólo se mantendrán en el servidor un tiempo configurable, dependiendo de las circunstancias. |
| RM-06 | Rendimiento y Mantenibilidad | El sistema debe poder evolucionar sin problemas ante cualquier cambio. |  |

#### 2 - Portabilidad y Escalabilidad

Entendemos como portabilidad la propiedad que posee un software que le permite ser ejecutado en diferentes plataformas y/o sistemas operativos. También puede entenderse en términos de compatibilidad.

La escalabilidad, por otra parte, se refiere a la capacidad de un sistema de gestionar cambios en la carga de trabajo. La preocupación principal radica en qué puede sucederle al sistema si este comienza a recibir una carga de trabajo importante, puesto que su rendimiento puede verse afectado hasta tal punto de colapsar. Así, un sistema escalable es capaz de mantener dicho rendimiento a pesar de las altas cargas de trabajo.

| ID | Categoría | Requisito | Criterio verificable |
|---|---|---|---|
| PE-01 | Portabilidad y Escalabilidad | El sistema debe ser escalable, de forma que pueda soportar una alta carga de usuarios garantizando un estándar mínimo de operatividad | El sistema garantizará un mínimo de operatividad del 99% bajo altas cargas de trabajo. |
| PE-02 | Portabilidad y Escalabilidad | El sistema debe operar tanto en ordenadores como en dispositivos móviles. | La interfaz del sistema puede visualizarse y es completamente operativa en todo dispositivo. |
| PE-03 | Portabilidad y Escalabilidad | Compatibilidad con PDF e importaciones CSV (de forma que no aparezcan fragmentos o carácteres rotos). | Tras una importación, los caracteres se mantienen intactos. |
| PE-04 | Portabilidad y Escalabilidad | El sistema debe ser operable (estar disponible) desde cualquier lugar. | El acceso a la plataforma puede realizarse sin requerir instalaciones locales específicas de software y/o hardware. |
| PE-05 | Portabilidad y Escalabilidad | Compatibilidad con navegadores modernos habituales. | El sistema funciona correctamente en navegadores como Chrome, Firefox o Safari. |
| PE-06 | Portabilidad y Escalabilidad | El código y arquitectura debe de poder ser visionable en distintos equipos. | Los entornos de trabajo y despliegue son multiplataforma. |

#### 3 - Accesibilidad y adaptación

La accesibilidad puede considerarse uno de los puntos más importantes a la hora de desarrollar cualquier tipo de software. Al fin y al cabo, de él no se espera simplemente que funcione, sino que debe ser usable por cualquier usuario e incluso adaptarse a sus necesidades.

| ID | Categoría | Requisito | Criterio verificable |
|---|---|---|---|
| AA-01 | Accesibilidad y adaptación | La interfaz del sistema debe ser fácil, simple e intuitiva | Se minimizará lo posible el número de "clics" que necesitan efectuarse para llevar a cabo cualquier tarea. |
| AA-01.1 | Accesibilidad y adaptación | Debe evitar los textos largos. | Las descripciones e instrucciones en pantalla no superarán un máximo de líneas. |
| AA-01.2 | Accesibilidad y adaptación | Los textos deben ser claros y concisos. | El vocabulario de la interfaz será directo, evitando mensajes crípticos. |
| AA-02 | Accesibilidad y adaptación | La aplicación, principalmente, estará en español, pero el idioma será configurable. | Al cambiar el idioma en la configuración, los textos se actualizan. |
| AA-02.1 | Accesibilidad y adaptación | De la misma manera, también se configurarán los horarios según las necesidades de las empresas debido a festivos, etc. | El sistema permite la parametrización del calendario de festivos y franjas operativas desde el panel correspondiente. |
| AA-03 | Accesibilidad y adaptación | Los datos que se introduzcan y/o se guarden, como pueden ser direcciones, números de identificación, etc. se adapten según el lugar desde el que se opere o las condiciones del cliente (esto puede ser configurable). | Los formularios adaptan la validación y formato según el país o el tipo de cliente con el que se trabaja. |
| AA-04 | Accesibilidad y adaptación | El usuario debe requerir ayuda mínima para manejar el sistema. | El usuario es capaz de localizarse y manejarse en el sistema en cuestión de segundos o, como mucho, en unos pocos minutos. |
| AA-05 | Accesibilidad y adaptación | El sistema debe evitar menús profundos. | En ningún momento el sistema deberá navegar durante más de un número determinado de profundidad. |
| AA-06 | Accesibilidad y adaptación | Consistencia en la interfaz. | Todos los módulos y pantallas comparten la misma distribución visual. |
| AA-07 | Accesibilidad y adaptación | El sistema debe enviar confirmaciones solo cuando sean necesarias, sin saturar al usuario. | Las confirmaciones se encuentran restringidas a operaciones críticas. |

#### 4 - Integridad y trazabilidad

La integridad de los datos nos asegura que estos son precisos, completos y coherentes en cualquier momento de su ciclo de vida. Mantener la integridad de los datos implica protegerlos contra pérdidas, filtraciones o corrupciones.

La trazabilidad, por su parte, responde a la propiedad que nos permite "seguir" el camino "trazan" los datos por el sistema. Un sistema trazable nos permite tener un mayor control y vigilancia sobre este.

| ID | Categoría | Requisito | Criterio verificable |
|---|---|---|---|
| INT-01 | Integridad y trazabilidad | Debe asegurar en todo momento la precisión de los datos, evitando así cualquier discrepancia entre la plataforma y registros de cobro externos (Stripe). | Tras cada conciliación, el estado de pago de la factura coincide con el movimiento que lo justifica. |
| INT-01.1 | Integridad y trazabilidad | El sistema estará conectado a ciertas fuentes de datos que nos permitan asegurar de primera mano si los cobros han sido efectuados o no (cuenta bancaria de la empresa). | El sistema consulta el estado de cobro contra la fuente configurada. |
| INT-01.2 | Integridad y trazabilidad | Con motivo de asegurar la integridad y seguridad de los datos, el sistema deberá avisar de posibles revisiones así como recopilar o permitir introducir la facturación anual verificada. |  |
| INT-02 | Integridad y trazabilidad | Todo movimiento dentro del sistema es registrado y monitoreado (tanto por parte del cliente como de los propios administradores). Esto incluye: | Para las acciones clave se puede saber quién, cuándo y qué cambió. |
| INT-02.1 | Integridad y trazabilidad | Cualquier modificación de ficheros o facturas efectuadas | Toda modificación queda registrada con usuario, fecha y valor anterior y nuevo. |
| INT-02.2 | Integridad y trazabilidad | Historiales (búsqueda, interacción, etc.) |  |
| INT-02.3 | Integridad y trazabilidad | Cuando se envía una factura, queda constancia de a qué destinatarios se envió. | Por cada envío de factura se guarda destinatario, fecha, canal y resultado, y no cambia aunque después se modifiquen los datos. |
| INT-02.4 | Integridad y trazabilidad | Si un usuario deja Turbine, su "rastro" se mantiene en el sistema. | Tras desactivar a un empleado, sus acciones anteriores siguen en el registro atribuidas a su cuenta. |
| INT-02.5 | Integridad y trazabilidad | Registros de ejecución. | Cada ejecución de un proceso automático registra fecha, elementos procesados y fallos con su motivo correspondiente. |
| INT-03 | Integridad y trazabilidad | Se realizan copias de seguridad automáticas de los datos y documentos del sistema, siguiendo una política definida. | Existe una política documentada y la última copia correcta es visible en el sistema. |
| INT-03.1 | Integridad y trazabilidad | Las copias son restaurables. |  |
| INT-04 | Integridad y trazabilidad | Control y revisión humana de los datos |  |
| INT-04.1 | Integridad y trazabilidad | Determinados cambios económicos deben ser aprobados por un administrador. Si el descuento supera un umbral configurado, quizá tenga que aprobarlo el superadministrador. | Determinadas modificaciones de datos en el sistema necesitan aprobación y no se efectúan hasta que la recibe. Todo esto queda registrado en el sistema. |
| INT-05 | Integridad y trazabilidad | En cuanto a la seguridad de los datos |  |
| INT-05.1 | Integridad y trazabilidad | Se protegerán los datos de posibles sobreescrituras accidentales. |  |
| INT-05.1.1 | Integridad y trazabilidad | Modificaciones en facturas o tarifas actuales no deben afectar a facturas pasadas. | Tras cambiar una tarifa, plantilla o configuración, las facturas ya emitidas conservan su importe y su PDF original. |
| INT-05.2 | Integridad y trazabilidad | En el caso de que faltara un campo por rellenar en un formulario, los otros ya rellenados no deben perderse, sino que aparecerán marcados aquellos que deben ser rellenados. El sistema actuará de la misma forma si el usuario o trabajador se equivocara en un campo. | Al encontrar un campo erróneo o faltante el sistema conserva los demás datos introducidos y se indica el campo y motivo. |
| INT-06 | Integridad y trazabilidad | Tras una acción masiva en el sistema se mostrará todo a lo que afecta, pidiendo confirmación cuando sea destructiva y sensible y que sea exitosa. | El sistema muestra a qué afecta una acción masiva antes de confirmar y, al terminar, informa de los éxitos y errores sucedidos. |
| INT-07 | Integridad y trazabilidad | Con motivo de evitar discrepancias producidas por los cambios entre franjas horarias, se usará la fecha de negocio que corresponda para las fechas legales de factura, no solo la hora del servidor. |  |
| INT-08 | Integridad y trazabilidad | Las tareas automáticas mensuales deben ser reintentables. Esto, con el fin de minimizar daños o incluso superar los obstáculos que nos pueden ocasionar las caídas del sistema. | Si el sistema cae, al recuperarse detecta qué facturas no se generaron y las genera. |
| INT-08.1 | Integridad y trazabilidad | Continuando con las posibles caídas, evitar duplicados por reintentos o pérdidas de emisión. | Ejecutar dos veces una generación produce el mismo número de facturas (una sola). |
| INT-08.2 | Integridad y trazabilidad | Asimismo, el sistema debe detectar fallos y recuperarse. | Un cliente con datos incorrectos no impide procesar a otros. |
| INT-09 | Integridad y trazabilidad | Precisión decimal en los importes. De forma que el manejo de decimales en el código no hagan desaparecer céntimos y se expliquen ciertos redondeos. |  |
| INT-09.1 | Integridad y trazabilidad | Consecuentemente, todo debe cuadrar. |  |
| INT-10 | Integridad y trazabilidad | Debe quedar constancia de las consecuencias acarreadas por según qué acciones. |  |
| INT-11 | Integridad y trazabilidad | La automatización del sistema debe ser confiable, documentando cada evento. |  |

#### 5 - Independencia

Responde a la capacidad del sistema para operar de manera uniforme independientemente del sistema operativo o del entorno de hardware subyacente. En el contexto de las empresas, esta característica resulta de vital importancia puesto que facilita la integración y gestión de diferentes tecnologías sin depender de un sistema operativo, recursos, etc.

| ID | Categoría | Requisito | Criterio verificable |
|---|---|---|---|
| IND-01 | Independencia | El sistema es capaz de funcionar individualmente, sin depender de otros módulos de la empresa, aplicaciones, etc. | La aplicación opera con normalidad con Brain desconectado. |
| IND-01.1 | Independencia | Esto también afecta a los proveedores, es decir. El sistema no debería verse obligado a reescribir el modelo de factura en el caso de que se cambie de proveedor. |  |

#### 6 - Seguridad

La misión de la seguridad en el desarrollo software radica en reducir las vulnerabilidades del sistema durante todo su ciclo de vida. Con ello podemos evitar ataques o "fugas" que comprometan la integridad de sus datos.

| ID | Categoría | Requisito | Criterio verificable |
|---|---|---|---|
| SEG-01 | Seguridad | Cada usuario (refiriéndonos tanto a clientes como a administradores) tendrá un tiempo límite de sesión si permanece inactivo. Es decir, si permanece inactivo durante demasiado tiempo, su sesión puede caducar y necesitará autenticarse de nuevo. | Tras un periodo de inactividad, la sesión caduca y el sistema exige volver a iniciar sesión. |
| SEG-02 | Seguridad | Cada usuario solo podrá descargar lo que tenga permiso para ver. | Un usuario sin permiso sobre algún cliente, facturas, etc. es rechazado por el servidor aunque conozca la URL correspondiente. |
| SEG-03 | Seguridad | Facturas, datos fiscales y contactos no deberían quedar en URLs públicas adivinables. |  |
| SEG-04 | Seguridad | Los enlaces generados serán temporales, es decir, caducarán. |  |
| SEG-05 | Seguridad | Las comunicaciones deben ir cifradas. |  |
| SEG-06 | Seguridad | Las contraseñas no se guardan en texto plano. | Las contraseñas no aparecen en claro en la base de datos ni en los registros. |
| SEG-07 | Seguridad | La exportación debe respetar los permisos del usuario, es decir, cada usuario solo podrá exportar aquello a lo que tenga acceso y pueda ver. | Un usuario al exportar sus clientes solo es capaz de hacerlo con los clientes que le han sido asignados. |
| SEG-08 | Seguridad | Las delegaciones temporales de cargos, clientes ,etc. deben caducar, para así evitar la compartición de contraseñas. | Cuando llega la fecha límite, la delegación caduca, el usuario correspondiente pierde el acceso. |
| SEG-09 | Seguridad | Si algún informe generado contiene detalles sensibles de clientes, sólo debe enviarse a usuarios autorizados. | Un informe programado con detalles de clientes no se envía a un destinatario sin contar con el permiso pertinente. |

#### 7- Legislación

Este apartado se centra en las restricciones legales con las que el sistema puede encontrarse.

| ID | Categoría | Requisito | Criterio verificable |
|---|---|---|---|
| LEG-01 | Legislación | Los datos de usuario se mantendrán en el sistema el tiempo que permita la ley (Conservación). | Al darse de baja un cliente, sus datos no se eliminan hasta cumplirse el plazo correspondiente. |
| LEG-02 | Legislación | Se mantendrá un histórico y estado de cada cliente (hablando de el borrado de estos) sujeto a lo que permita la ley (Minimización). | Al modificar un cliente, incluso en su borrado, el sistema mantiene ciertos datos como sus facturas emitidas. |
| LEG-03 | Legislación | Las copias de seguridad también se ven sujetas a restricciones legales, ya sea por tiempo, el tipo de datos que puede almacenar, etc. |  |


## 5. Reglas de negocio

En este apartado se tratarán aquellas reglas/cuestiones del dominio/organización que condicionan el comportamiento del sistema.

Los siguientes puntos no representan por sí mismos una funcionalidad, simplemente son eso, puntos que la propia empresa nos pide que tengamos en cuenta.

Como aclaración, realmente la línea de división entre lo que entendemos como "requisito" (de cualquier tipo, aunque afecta más a los funcionales y no funcionales) y "regla de negocio" en ocasiones puede ser ambigua. Por lo mismo, en este apartado es posible que aparezcan ideas, puntos o temas ya mencionados anteriormente, pero con una ligera diferencia: aquí se tratarán como los deseos del cliente y en otros puntos, más bien, como el propio comportamiento del sistema y cómo este debe asegurarlos.

### 5.1 Facturación

| ID | Regla |
|---|---|
| RB-01 | El plazo de pago estándar es de cinco días. |
| RB-01.1 | Si un cliente tiene pactadas otras condiciones, el vencimiento derivado de estas prevalece sobre el general. EJ: Si un importe es muy grande, se puede pagar a una cantidad determinada de días. |
| RB-02 | Cuando se emite una factura, esta no puede alterarse en silencio, sino que todo cambio queda trazado y, cuando legalmente no proceda modificarla o el periodo esté cerrado, se corrige con una rectificativa. |
| RB-03 | Una factura emitida no se elimina. |
| RB-04 | No existen las facturas simplificadas, cada factura debe ser igual a la otra, al menos, en formato. |
| RB-05 | Existen distintos tipos de facturas |
| RB-05.1 | Las facturas recurrentes, por lo general, son mensuales, emitiéndose el día uno de cada mes. Una factura con otra fecha es puntual, no sigue el estándar. |
| RB-05.1.1 | En caso de que un cliente se dé de alta en días intermedios del mes, se genera una factura proporcional en relación a los días restantes del mes, volviendo a la normalidad a partir del mes siguiente. |
| RB-05.1.2 | También puede darse el caso de que un cliente no empiece a facturar hasta el mes siguiente o, en general, en fechas futuras. Hasta esa fecha, no se emiten facturas ni recordatorios de cobro. |
| RB-05.2 | Las facturas puntuales no tienen rango de precios |
| RB-06 | Si el vencimiento de una factura (o notificación) cae en un día no laborable (fines de semanas o festivos), las comunicaciones se moverán al siguiente día laborable según la configuración del calendario. |

### 5.2 Alcance y límites del sistema

| ID | Regla |
|---|---|
| RB-07 | La aplicación es independiente de uno de los productos de la empresa, Brain, y de su interfaz. Se trata de una aplicación de uso interno y gestión de la organización. |
| RB-07.1 | Así mismo, para su uso no se permiten correos personales ni de terceros, sólo pertenecientes a Turbine. |
| RB-07.2 | Tampoco implementa servicios fiscales como Verifactu ni deduce la legislación de cada país, puesto que de eso se encarga la propia gestoría de Turbine. |
| RB-08 | El cliente no espera una pasarela de pago, puesto que ese tema lo gestiona la pasarela de la propia empresa, es decir, fuera del sistema solicitado. El deber del sistema es íntegramente el de la gestión de la facturación de la empresa. |
| RB-09 | El sistema no tiene chat propio ni general. Sí es posible que tenga algún tipo de conexión con terceros (ej: Whatsapp, e-mail). |
| RB-09.1 | Por defecto, el envío es por correo. |
| RB-09.2 | En la práctica, los servicios externos pueden simularse. |
| RB-10 | Las bajas se gestionan fuera de la aplicación, el sistema simplemente las registra e informa sobre posibles deudas. |
| RB-11 | Puede proporcionarse un catálogo, algo ligero. |
| RB-12 | El sistema estará preparado para una posible extensión, puesto que, aunque actualmente funcione en una sola entidad, es posible que en un futuro opere con varias. |

### 5.3 Clientes y ciclo de vida

| ID | Regla |
|---|---|
| RB-13 | El servicio funciona mes a mes, sin permanencia. |
| RB-14 | Se puede registrar a un cliente con datos básicos. |
| RB-15 | Se prohíbe la emisión o activación de la recurrencia en clientes que no tengan todos los datos necesarios |
| RB-16 | No se puede dejar a un cliente activo sin contacto de facturación. En caso de que algo similar pueda suceder, el sistema avisa. |
| RB-17 | En lo referente a "bajas", "pausas o estados similares de los clientes" |
| RB-17.1 | El cliente puede darse de baja, en cuanto la notifique, dejan de generarse mensualidades nuevas. |
| RB-17.2 | El cliente puede hacer una pausa. Durante este periodo no se generan mensualidades y el cliente sigue presente en el sistema. Además, y diferenciándose de las bajas, un cliente en pausa puede tener facturas antiguas pendientes. |
| RB-18 | Los datos del cliente se conservan el máximo tiempo que permita la ley, como su histórico y estado. |
| RB-18.1 | Estos datos se mantienen si, por ejemplo, un cliente se da de baja. |
| RB-19 | La anonimización o eliminación de un contacto no invalida las facturas históricas: aspectos como el PDF emitido se conservan según la ley y solo la ficha operativa se anonimiza cuando proceda. |

### 5.4 Cobros e impagos

| ID | Regla |
|---|---|
| RB-20 | Una factura solo se considera pagada cuando se detecta o valida un cobro por su importe. |
| RB-21 | En relación con las notificaciones pertinentes, si un cliente solicita una baja pero tiene algún impago, estas deben seguir enviándose. |
| RB-21.1 | Un usuario normal no puede apagar el flujo de recordatorios de uno de sus clientes por su cuenta a excepción de que tenga los permisos para ello. |
| RB-22 | La deuda no bloquea la baja y la baja no hace desaparecer el impago. |
| RB-23 | Como tal, no hay pagos fraccionados como modalidad comercial, un pago parcial se registra como tal. |
| RB-24 | Los impagos ni cancelan ni sancionan al cliente, pero sí se mantiene el flujo de notificaciones. |
| RB-25 | Se puede llegar a perdonar una deuda, pero, en caso de que eso ocurra, debe ser una acción muy controlada, con motivo y aprobación. |
| RB-25.1 | Una deuda perdonada no significa que esté pagada, puesto que eso significa que se ha detectado o validado un cobro. |

### 5.5 Tarifas y suscripciones

| ID | Regla |
|---|---|
| RB-26 | La tarifa depende del rango de facturación anual de la empresa/cliente con el que se trabaje. |
| RB-26.1 | Estas son revisadas una vez al año para poder actualizarlas si fuera necesario. |
| RB-26.1.1 | Si algún precio se actualiza, rige desde su fecha de efecto, sin reescribir meses anteriores. |
| RB-26.2 | El precio se mantiene fijo durante el año, salvo en casos excepcionales pactados y aprobados entre el cliente y la empresa. |

### 5.6 Control, datos y legalidad

| ID | Regla |
|---|---|
| RB-27 | Los cambios en el sistema, aunque se busca que todo sea automatizable, deben ser monitorizables y revisables por trabajadores humanos (no todos, pero algunas modificaciones requerirían validación antes de efectuarse). |
| RB-27.1 | Como aclaración, esto afecta tanto a las facturas, cualquier modificación de los datos del sistema y, también en la jerarquía de roles, como puede ser la asignación de clientes o de los propios roles. |
| RB-28 | A la hora de realización de pruebas sobre el sistema, los datos con las que se operen estas no pueden ser de clientes reales. En el caso de que deban usarse datos reales, se asegurará su privacidad y anonimato. |
| RB-29 | Se asegurará la diferenciación y separación entre datos de facturación y datos dedicados al márketing. Con esto pretende evitarse que se use automáticamente datos de facturación para campañas publicitarias y/o comerciales. |
| RB-30 | Un periodo cerrado administrativamente no admite cambios económicos ordinarios sin un procedimiento excepcional y auditado. |
| RB-31 | El sistema puede contar con un seguimiento del rendimiento de la aplicación mediante unas métricas de uso, pero estas métricas existen exclusivamente para eso, no se hace ningún tipo de seguimiento de los empleados. |


## 6. Suposiciones y decisiones de análisis

En esta sección se documentarán las decisiones tomadas por el equipo cuando la
información proporcionada por el cliente no sea suficiente para determinar unívocamente
un requisito.

| ID   | Suposición o decisión                                                                                                 | Justificación                                                                                                                  | Requisitos afectados |
|------|-----------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------|----------------------|
| A-01 | El "Superadministrador" se considera un tercer rol independiente.                                                     | Aunque su diferencia con el administrador normal es pequeña, separarlo facilita su gestión e identificación en el sistema.     | [Añadir FR]          |
| A-02 | Los clientes finales no interactúan con la plataforma; las altas y bajas las realizan exclusivamente los empleados.   | El sistema está pensado para uso interno y solo permite registrar facturas de empresas reales gestionadas por los trabajadores.| [Añadir FR]          |
| A-03 | Las facturas emitidas no se pueden modificar ni borrar directamente; deben ser rectificadas.                          | Eliminar facturas dejaría un hueco debido a la obligación de mantener una numeración correlativa (AMB-01, FAC-03).             | [Añadir FR]          |
| A-04 | La baja de un cliente que tenga deuda pendiente se hace efectiva en el servicio, pero el proceso de recobro continúa. | Permite cancelar la prestación del servicio sin detener la gestión de los impagos activos (AMB-04).                            | [Añadir FR]          |
| A-05 | El sistema solo informa de los pagos al usuario y al administrador, no confirma directamente al cliente.              | La confirmación oficial del pago al cliente la emite la plataforma de pagos externa al recibir los fondos.                     | [Añadir FR]          |
| A-06 | No existe un proceso de registro público para usuarios; solo mecanismo de login.                                      | El cliente especificó que él mismo introduce los usuarios en el sistema, asumiendo un alta inicial interna.                    | [Añadir FR]          |
| A-07 | La integración de una API se contempla explícitamente como un caso de uso (API-01).                                   | Cubre la necesidad de negocio de evolucionar e integrarse con otros sistemas en el futuro.                                     | API-01, [Añadir FR]  |


## 7. Dudas, ambigüedades y cuestiones pendientes

Se incluirán aquí aspectos de la información proporcionada por el cliente que sean
ambiguos, incompletos, contradictorios o que requieran aclaración.

Notas sobre las fuentes: las referencias (P0xx) corresponden a los bloques de la transcripción de la
primera reunión (sesión del 29 de septiembre de 2026). La transcripción de la reunión ampliada no tiene
bloques numerados, por lo que se cita como "Reunión ampliada, pág. N".

| ID | Cuestión | Origen | Impacto | Estado / resolución |
|----|----------|--------|---------|---------------------|
| Q-01 | ¿El superadministrador es un tercer rol o un administrador con un permiso extra? En las notas de requisitos "al principio no se tendrá en cuenta", pero hay funciones que solo puede realizar él (configuración profunda). | Entrevista (P144, P149, P151, P208) / notas de requisitos; Reunión ampliada, pág. 7 | Roles y permisos de usuario; configuración del sistema | Resuelta: se considera un tercer rol independiente (ver A-01). En la primera reunión el cliente dijo que debía existir alguien con poder de configuración más profunda y que lo dejaba para pensar durante la asignatura (P149, P151). La reunión ampliada lo confirma: configuración profunda para el superadministrador, operación para los administradores y trabajo sobre lo asignado para los usuarios operativos. |
| Q-02 | Baja de cliente: el cliente dice en un momento que él mismo cambia el estado del cliente en la aplicación (P123) y en otro que la baja se hace en otro sitio y solo nos consultan (P202, P203). ¿Se hacen las dos cosas? | Entrevista (P098, P123, P202, P203); Reunión ampliada, pág. 4 | Alta y baja de clientes; fin de la facturación recurrente | Resuelta en parte: en la reunión ampliada el cliente aclara que la baja del producto ocurre fuera, y que esta aplicación debe recibir o registrar la solicitud y consultar si hay deuda. La baja efectiva en facturación se registra en la aplicación (ver A-04). Falta decidir quién la registra cuando no llega por API (ver Q-11). |
| Q-03 | A la pregunta "¿la aplicación quiere tener la posibilidad de crear un cliente o no?" el cliente respondió "No" (P209), lo que choca con el resto de la información. Probablemente entendió otra pregunta. | Entrevista (P141, P209) | Registro de clientes | Resuelta por interpretación del equipo: se entiende que no se contemplan simulaciones, solo el registro de facturas de empresas reales. En P141 el cliente confirma que solo su empresa registra clientes y que un cliente no puede registrarse a sí mismo (ver A-02), y la reunión ampliada (págs. 1-2) describe el alta de clientes como función normal de la aplicación. Conviene confirmarlo con el cliente. |
| Q-04 | Facturas ya emitidas: ¿hasta cuándo pueden modificarse? En la primera reunión el cliente dijo que debería poder modificarlas siempre, pero que se va por trimestres: una factura de marzo ya no se puede tocar el 1 de abril (P083-P085). Dentro del grupo había dos lecturas: modificables hasta fin de trimestre o hasta el pago, o solo rectificables. | Notas de requisitos / Entrevista (P083, P084, P085); Reunión ampliada, págs. 3 y 12 | Modificación y rectificación de facturas; cierre de periodos | Resuelta: en la reunión ampliada el cliente aclara que antes de emitir la factura se puede corregir; después de emitirla, todo cambio relevante debe ser trazable y se hace mediante rectificativa. El cierre de trimestre pasa a ser el cierre administrativo de periodo (CU-29), que impide cambios ordinarios. Se mantiene A-03. Hay que retirar del grupo la lectura "modificable hasta fin de trimestre o pago" y la frase "Modificar factura mientras esté permitido". |
| Q-05 | Borrar facturas: el cliente habló de "CRUD" (P149), pero con numeración correlativa borrar una factura deja un hueco. ¿Se borran o se anulan? Nota: en P149 el CRUD se refiere a lo que puede hacer el usuario operativo sobre lo que tiene asignado, no a las facturas en concreto. | Entrevista (P149); Reunión ampliada, pág. 3 | Gestión de facturas | Resuelta: no se borran las facturas emitidas (ver A-03). Solo pueden eliminarse borradores nunca emitidos, dejando traza si procede (confirmado por el cliente en la reunión ampliada). La numeración correlativa también aparece en el material mostrado (P218). |
| Q-06 | Confirmación de pago al cliente: las notas de requisitos la incluyen, pero el cliente solo pidió avisar al empleado de TurbineH (P196). | Notas de requisitos / Entrevista (P195, P196, P199); Reunión ampliada, pág. 6 | Conciliación de cobros y notificaciones | Resuelta: el sistema solo informa internamente al responsable del cliente y al administrador. En P196 el cliente dice que no hay que crear un PDF de confirmación, solo notificar al usuario de Turbine que corresponda. La confirmación al cliente la emite la plataforma de pagos (ver A-05). |
| Q-07 | Recordatorios de impago: el cliente dijo "al menos cada dos o tres días" (P158). Hay que elegir un valor; en requisitos se ha puesto 2. | Entrevista (P088, P158); Reunión ampliada, pág. 5 | Recordatorios de impago | Resuelta: el cliente repite "dos o tres días" y pide que los tiempos sean configurables por el superadministrador. Se toma 2 días como valor por defecto. El vencimiento pactado de un cliente (p. ej. 30 días) prevalece sobre el general. |
| Q-08 | Periodos del panel: primero el cliente dijo semanal, mensual y anual (P104); después semanal, mensual y trimestral (P130). | Entrevista (P104, P130); Reunión ampliada, pág. 6 | Dashboard financiero | Resuelta: en la reunión ampliada el cliente enumera semana, mes, trimestre y año. Se ofrecen los cuatro periodos. |
| Q-09 | Canal de avisos: el cliente dijo "e-mail o WhatsApp la mejor opción posible" (P101). ¿Email, WhatsApp o ambos? | Entrevista (P101); Reunión ampliada, págs. 4-5 | Envío de facturas y avisos | Resuelta: el correo electrónico al contacto de facturación es el canal por defecto. WhatsApp es un canal complementario simulado, y solo si hay consentimiento del contacto. |
| Q-10 | "Darse de alta": las notas de requisitos dicen que el usuario se da de alta con su email, pero el cliente indicó que solo hay login y que los usuarios los introduce él (P118). | Notas de requisitos / Entrevista (P118, P125, P138, P141); Reunión ampliada, pág. 6 | Acceso y gestión de usuarios | Resuelta: no hay autorregistro. El login es con email de dominio de Turbine y contraseña (P138, P139), y los usuarios los introduce el cliente (P118). El superadministrador o un administrador con permiso crea o invita a los usuarios (ver A-06). |
| Q-11 | API: el grupo decidió dejarla fuera porque "las bajas se hacen desde la app". Pero ya en la primera reunión el cliente dijo que la aplicación debía tener una API para decir si hay facturas pendientes (P203, P204), que debe quedar documentada (P128) y que dejar la puerta abierta a una API externa es buena práctica (P212). En la reunión ampliada pide documentar una API (consulta de clientes, deuda, facturas) y dos endpoints concretos: consulta de obligaciones pendientes y aviso de "cliente solicita baja" que genera una tarea. Permite implementar solo un subconjunto representativo, con el contrato documentado. | Entrevista (P127, P128, P129, P203, P204, P212); Reunión ampliada, págs. 4 y 10 | Integración externa por API; baja de clientes | **Pendiente:** la decisión actual contradice al cliente, que lo pidió en las dos reuniones. Propuesta: incluir en alcance la API con un subconjunto mínimo (los dos endpoints de baja) y documentar el contrato completo, y dejar fuera el resto. Confirmar con el equipo y, si es posible, con el cliente. Si se acepta, hay que corregir A-07 y mantener CU-31. |
| Q-12 | Rol del usuario operativo: el cliente lo llamó primero "lector" y dijo que solo ve lo que se le asigna (P119); luego, que no puede crear ni editar (P147); y después, que debería poder hacer CRUD sobre lo asignado (P149). | Entrevista (P119, P147, P149); Reunión ampliada, pág. 7 | Permisos de usuario | Resuelta: el cliente dice que importa la capacidad, no el nombre. Puede ver y modificar la información operativa de sus clientes asignados y crear facturas si su permiso lo permite. No puede cambiar tarifas globales, crear series ni ver el dashboard financiero global. |
| Q-13 | Citas de la reunión ampliada: la transcripción no tiene bloques numerados, por lo que no se puede citar como la primera reunión. | Equipo | Trazabilidad de la fuente de los requisitos | Resuelta: se cita por página ("Reunión ampliada, pág. N"). |
| Q-14 | Prorrateo del primer mes: en la primera reunión el cliente dijo que si se da de alta el día 22 se le factura la parte proporcional del mes (P212). En la reunión ampliada pide una regla de negocio coherente y documentada (no asumir meses de 30 días) y que se sepa qué hacer si el alta es el último día del mes. No da la regla. | Entrevista (P212); Reunión ampliada, pág. 3 | Suscripciones y facturación recurrente | Pendiente: el equipo debe proponer la regla (p. ej. días reales del mes) y registrarla en la sección 5 y en la sección 6. |
| Q-15 | Alcance de las ampliaciones: el cliente menciona presupuestos, centros de coste, varias sociedades emisoras, objetivos de facturación, rol externo de gestoría, funciones de IA y fusión de fichas, pero dice que son ampliaciones. ¿Entran o no? | Reunión ampliada, págs. 10-14 | Alcance (sección 1.2); casos de uso | Pendiente: se propone dejarlas fuera del alcance inicial y documentarlas como tales, como pide el cliente. Los datos del emisor deben ser configuración, para no cerrar la puerta a varias sociedades. |
| Q-16 | Umbrales de aprobación, plazos y otros valores: el cliente pide que sean configurables y no da cifras. | Reunión ampliada, págs. 2-3 | Aprobaciones; configuración del sistema | Resuelta: se establecen como configuración sin valor fijo en los requisitos. Se propondrán valores por defecto razonables en el diseño. |
| Q-17 | Baja con deuda pendiente: en la primera reunión el cliente da versiones distintas. Primero dice que si hay un impago hay que notificar que no puede darse de baja hasta que pague (P098), y que "te daremos de baja cuando hagas frente a la siguiente factura" (P154), y que una factura puntual impagada impide la baja (P168, P169). Luego dice que si se notifica la baja ya no se pasan más facturas, pero se sigue recordando el impago anterior hasta que se pague (P176-P179). | Entrevista (P098, P154, P168, P169, P176-P179); Reunión ampliada, pág. 4 | Baja de clientes; recobro de deuda | Resuelta por interpretación del equipo: la baja es efectiva en la fecha indicada (no se generan más facturas recurrentes) y el recobro de las facturas anteriores continúa (ver A-04). La reunión ampliada lo apoya: el cliente no quiere que una deuda bloquee eternamente la baja comercial ni que la baja haga desaparecer el impago. |


## 8. Trazabilidad

En esta sección se mostrarán las relaciones entre diferentes requisitos del proyecto, mostrando cuando se relacionan requisitos de distintos tipos y con indicaciones cuando la asociación se de con varios requisitos de un mismo subbloque de un tipo o hasta con el subbloque entero (Eso indicándose con el nombre del subbloque o con la notación (Identificador del primer requisito) - (Identificador del último requisito) cuando sea un subconjunto de requisitos consecutivos de un mismo subbloque con la parte numérica ascendente).

Se mostrarán en una tabla donde las columnas será el tipo de requisito que se indique y cada fila será una relación ternaria entre dichos requisitos.

| Requisito de negocio | Requisito de usuario | Requisito funcional / no funcional |
|---|---|---|
| RNG-01 | URU-03, URA-01, URS-01, RG-02, RG-06 | CLI-01, IMP-01, AA-03 |
| RNG-02 | URU-03, URU-08, RG-06 | CLI-02 |
| RNG-03 | URU-03, URU-08 | CLI-03, PRI-01 |
| RNG-04 | URU-05 | CLI-05 |
| RNG-05 | URU-07 | CLI-01 |
| RNG-06 | URU-06 | CLI-03 |
| RNG-07 | URU-07, URU-14 | CLI-05, INT-01, INT-02, PE-07 |
| RNG-08 | URU-02 | FAC-01, FAC-02 |
| RNG-09 | URU-01, URS-01 | FAC-04 |
| RNG-10 | URU-04, URU-09 | FAC-03, FAC-06, RM-05 |
| RNG-11 | URU-04, URA-04, URS-01 | FAC-03, INT-03, INT-04 e Int-07 |
| RNG-12 | URU-07, URU-08, URS-06 | FAC-05, FAC-06, INT-04, INT-11 |
| RNG-13 | URU-08 | PRO-02, PRO-03, RM-01, AA-07, INT-01, INT-08 |
| RNG-14 | URU-09, URU-14, URA-15 | TAR-01, FAC-03, PE-03, INT-05 |
| RNG-15 | URU-06, URA-10, URS-01, URS-06 | PDF-01, FAC-01, FAC-06, APR-01, INT-01, INT-04, RM-01 |
| RNG-16 | URS-07, URS-03, URA-10, URU-08 | ENV-01, REC-01, REC-02, NOT-01, NOT-02, AA-07, |
| RNG-17 | URU-05, URU-08, URA-10, URS-05, RNDF-01, RNDF-02 | PAG-02, AUD-01, APR-01, APR-02, FAC-05, FAC-06, INT-04, |
| RNG-18 | URA-09, URU-08, URA-10, URS-03, URS-07, RG-02 | SUB-04, CLI-05, ENV-01, REC-01, REC-02, NOT-01, NOT-02, INT-01, INT-02, SEG-03 |
| RNG-19 | URA-09, URS-06, URS-07, RG-08 | NOT-02, AUD-01, PRI-01, REC-02, INT-10, INT-04, RM-04 |
| RNG-20 | URA-03, URU-02, URS-01 | SUB-03, APR-01, FAC-01, CLI-01, INT-07, INT-11 |
| RNG-21 | URA-03, URS-01, URS-03, RG-05, RG-06 | CLI-05, SUB-04, APR-01, REC-02, DAS-01, CFG-01, PRI-01, RM-04, INT-02, INF-01, BUS-01 |
| RNG-22 | URA-14, URU-08, URA-09, URA-10, URS-03, URA-11, RG-04, RG-06, RNDF-02 | AUD-01, DAS-01, INF-01, NOT-01, FAC-01, SUB-04, CLI-04, INT-05, INT-01 |
| RNG-23 | URU-05, URU-07, URA-07, URA-11, URS-03, RG-06 | CLI-03, SUB-01, FAC-01, DAS-02, INF-01, BUS-01, IMP-01, AA-01. |
| RNC-01 | URU-10, URU-11, URA-07, URA-11, URA-12, URS-04, URS-07, RG-02, RG-04, RNDF-02, RNDF-05 | DAS-02, USU-01 - USU-04, PRO-01, AMB-02, RM-03, SEG-01, SEG-02 |
| RNC-02 | URU-01 - URU-06, URU-09, URU-13, URA-01 - URA-07, URA-11 - URA-14, URS-01, URS-02, URS-05 - URS-07 | USU-01 - USU-04, "15 Auditoría y Configuración", CLI-03, RM-03, AA-06, INT-02 |
| RNC-03 | "1.3 Super Administrador", RG-04, RG-09, RG-10 | USU-01, USU-03, "15 Auditoría y Configuración", PRO-04, DAS-02, PAG-01, REC-01, APR-01, INT-04, SEG-03 - SEG-06, RM-05, TAR-01 |
| RNC-04 | URA-05, URA-06, URA-08, URA-12, URS-03, URS-07, RG-04, RNDF-05 | CLI-03, NOT-01, USU-01 - USU-03, AMB-02, RM-03, INT-02, SEG-02, SEG-07, SEG-09. |
| RNC-05 | URU-12, URU-13, URA-05, URS-02, URS-04, | CLI-03, NOT-01, BUS-01, USU-03, PRO-01, INT-01, INT-02, INT-07, SEG-08, |
| RNC-06 | URA-04, URS-01, URS-06, RG-05, | "4 Descuentos y Aprobaciones", TAR-01, SUB-04, FAC-06, "10 Pagos y Conciliación", DAS-02, AUD-01 |
| RNC-07 | URA-16, URA-17, URS-06, RG-08 | USU-02, USU-03, BUS-01, NOT-01, ENV-03, CLI-03, RM-03, AA-04, LEG-02 |
| RNC-08 | URU-05, URU-10, URU-12, URU-13, URA-08, URA-11, URS-04, RG-4 | USU-02, AA-06, RM-03 |
| RNC-09 | URU-02, URU-08, URA-03, URA-10, URA-13, URS-01, URS-06, RG-02, RG-06, | "3 Tarifas y Revisión anual", "4 Descuentos y Aprobaciones", FAC-06, GEN-01, PDF-01, NOT-01, INF-01, AUD-01, PE-03, INT-03 |
| RNC-10 | URS-01 | CLI-01, SUB-01, "3 Tarifas t Revisión anual", APR-02, GEN-01, IMP-01, AUD-01, INT-04, |
| RNC-11 | URS-07, RNDF-01 | PRO-04, "15 Auditoría y Configuración", DAS-02, "10 Pagos y Conciliación", ENV-01, APR-02, RM-02, INT-06, INT-10, LEG-03 |
| RNC-12 | RG-06, RG-07 | CLI-05, SUB-01, FAC-03 - FAC-05, FAC-07, SEG-03 - SEG-06, INT-08, INT-04, PE-06 |
| RNC-13 | URA-07, URS-03, RNDF-05 | DAS-02, DAS-01, CFG-01, PRO-04, PAG-01, TAR-01, INT-01, INT-05, INT-07 |
| RNI-01 | RG-09 | ENV-01, FAC-03, FAC-06, AA-02 |
| RNI-02 | URU-08, RNDF-01, URS-03, URA-10 | ENV-01, NOT-01 |
| RNI-03 | URU-08 | ENV-02, ENV-03, PRO-02, INT-08.1, INT-02.3 |
| RNI-04 | RG-08 | ENV-02, ENV-03, NOT-01, NOT-02, FAC-06 |
| RNI-05 | RG-10 | NOT-01, NOT-02, CLI-05 |
| RNI-06 | RNDF-01, RNDF-04 | API-01 |
| RNI-07 | RNDF-01 | IND-01, API-01, INT-01, PAG-01 |
| RNS-01 | RG-03 | TAR-02, APR-02, INF-01, IMP-01, AUD-01, INT-02 + subapartados, PAG-03 |
| RNS-02 | RG-01 | SEG-01, USU-01 |
| RNS-03 | URS-06, URS-07 | PRI-01, CLI-04, LEG-02, LEG-01 |
| RNS-04 | URS-03, URS-06, RG-08 | NOT-01 |
| RNS-05 | URA-13, URU-14, RG-03 | LEG-03, PRO-05, INT-03 |
| RNS-06 | URU-04, URA-04 | PRO-04, INF-01, AUD-01, INT-05.1.1 |
| RNS-07 | URA-08, URA-11, URS-03 | PRO-01, PRO-05, GEN-01 |
| RNS-08 | URU-05, URA-09, URS-03, RG-08, RG-09 | BUS-01, RM-04 |
| RNS-09 | RG-03, URS-03, URA-10, URU-08 | INF-01, PRO-03, INT-06 |
| RNS-10 | URA-13 | INF-01, PDF-01, PE-03 |
| RNS-11 | URU-11, URA-10, URA-09, URS-06, RG-08 | NOT-01, NOT-02, DAS-01, DAS-02 |


## 9. Glosario

### Términos de la sección Requisitos de Negocio

| Término | Definición |
|---|---|
| Turbine | Único actor de esta sección. Se refiere a la propia empresa, que exige el cumplimiento de las reglas, políticas y restricciones del sistema con independencia del empleado o rol que lo lleve a cabo. |
| RNG | Requisitos de negocio de gestión comercial. Requisitos centrados en el cliente y en el sistema de facturación y de cobro. |
| RNC | Requisitos de negocio de control interno. Requisitos centrados en los roles dentro de la empresa, configuraciones que gestionan dichos roles y tarifas. |
| RNI | Requisitos de negocio de comunicación e integración. Requisitos centrados en los mensajes con los clientes y conexión con otros sistemas dentro de la empresa. |
| RNS | Requisitos de negocio de seguridad y continuidad. Requisitos centrados en las copias de seguridad y cumplimiento legal en la empresa. |
| RSL | Requisitos de seguridad lógica / respaldo (usado en RSL-05). Se refiere a los requisitos enfocados en la realización de copias de seguridad automáticas de datos y documentos y la posibilidad de probar y ejecutar su restauración. |

### Términos de la sección Requisitos de Usuario

| Término | Definición |
|---|---|
| CE | Cualquier empleado. Denota a cualquier empleado que use la plataforma |
| US | Usuario. Se refiere a los empleados que usen la aplicación y tengan el rol de usuario. |
| AD | Administrador: Se refiere a los empleados que usen la aplicación bajo el rol de Administrador. |
| SAD | Super Administrador: Se refiere a los empleados que usen la aplicación bajo el rol de Super Administrador. |
| URU | User-Requirements-User: Se refiere a los requisitos de usuario enfocados en funcionalidades referentes a los empleados en el rol de usuario. |
| URA | User.Requirements-Administrador: Se refiere a los requisitos de usuario enfocados en funcionalidades referentes a los empleados que ocupan el rol de Administrador. |
| URS | User-Requirements-Super: Se refiere a los requisitos de usuario enfocados en funcionalidades referentes a los empleados que ocupan el rol de Super Administrador. |
| RG | Registros generales, se refieren a aquellos requisitos de usuario enfocados en funciones del sistema que son esperadas por cualquier empleado independientemente de su rol. |
| RNDF | Requisitos-No-Directamente-Funcionales: Se refiere a requisitos que al igual que los anteriores son esperados por todos los empleados que usen la aplicación que no están centrados en funcionalidades específicas sino en el comportamiento de la aplicación. |

### Términos de la sección Requisitos Funcionales

| Término | Definición |
|---|---|
| SA | Super Administrador: Rol por encima de los administradores con acceso a todas las funciones de estos y a la modificación de las preconfiguraciones. |
| AD | Administrador: Rol por encima de los usuarios con acceso a todas las funcionalidades, movimientos y clientes de la aplicación y que tan solo no pueden modificar las preconfiguraciones. |
| UO | Usuario Operativo: Rol de menor nivel referido a los empleados que solo pueden actuar de forma limitada sobre los clientes que un administrador le han asignado. |
| CLI | Clientes y contactos: Se refiere a los requisitos dedicados a la creación, gestión, borrado y almacenamiento de datos de los clientes y los contactos proporcionados por dichos clientes. |
| SUB | Subscripción: Referido a los requisitos dedicados a la relación comercial establecida con los clientes, y la opción de suscripción al servicio que estos hayan elegido y no a la creación y gestión de facturas en sí. |
| TAR | Tarifas y Revisión anual: Denota a los requisitos centrados en la gestión de los periodos de revisión de los precios y las tarifas que se cobraran a las empresas con pago recurrente |
| APR | Aprobaciones y descuentos: Indica los requisitos referidos a la posibilidad de hacer excepciones comerciales a ciertos clientes, con la necesaria aprobación de dicha excepción por un administrador. |
| FAC | Facturas: Todos los requisitos dedicados a la creación y gestión de facturas, sus posibles modificaciones, identificadores que las separan del resto, facturas rectificadas y enviadas. |
| GEN | Generación Recurrente: Se refiere a los requisitos que explican el funcionamiento de la generación recurrente para los clientes que hayan optado por ese modelo de suscripción, explicando funcionamiento y plazo de emisión. |
| PDF | Referido a los requisitos relacionados a la creación de las facturas en un formato PDF, los datos que se deben incluir y el formato esperado que debe tener. |
| ENV | Envío y entrega: Denotan los requisitos dedicados a la gestión de envio de las facturas y notificaciones, los datos que el sistema debe guardar sobre estos y el comportamiento del sistema ante excepciones como un correo fallido o que no le puede llegar a su destinatario. |
| REC | Recordatorios, impagos y disputas: Referidos a los recordatorios al cliente de realizar el pago durante el periodo de pago, el comportamiento cuando se produce un impago y el periodo de envío de notificaciones y los cambios que se hagan sobre este. |
| PAG | Pagos y conciliación: Denota los requisitos acerca de la integración con el sistema de pago de la empresa, la independencia de la conciliación con respecto a la fuente y la traza de conciliaciones y facturas. |
| NOT | Notificaciones e incidencias: Requisitos sobre las notificaciones que reciben los usuarios y administradores sobre los movimientos en los clientes y posibles incidencias en la facturación. |
| DAS | Trabajo diario, Dashboard, búsqueda e informes: Denota los requisitos acerca de las múltiples opciones de filtrado que debe ofrecer el sistema y la construcción del dashboard financiero que se presenta a los administradores. |
| IMP | Importación: Referido a los requisitos que especifican las opciones de importar clientes y las modificaciones que deben realizarse sobre la interfaz de la aplicación tras una operación de importación. |
| USU | Usuario, roles y acceso: Incluye los requisitos que detallan el inicio de sesión de los usuario, las distinciones en roles y la explicación de las funcionalidades de cada rol. |
| AUD | Auditoría: Incluye los requisitos referidos al establecimientos de preconfiguración o parámetros globales que la aplicación y los usuarios deben respetar y solo el SA puede reemplazar. |
| CFG | Configuraciones: Separación de la preconfiguración global y los datos operativos y comportamiento del sistema antes el intento de modificación de alguno de estos. |
| API | API e Integraciones: Requisitos sobre la API y la documentación necesaria respetando la sencillez del sistema. |
| PRO | Procesos, automáticos y operación: Denota los requisitos referidos a la ejecución y registros de los procesos automáticos del sistema, restricciones, escalabilidad, modificaciones y copias de seguridad. |
| PRI | Datos Personales: Incluye los requisitos acerca del tratamiento de datos personales de representantes de los clientes, así como las opciones de filtrado, borrado y modificación. |
| AMB | Ambigüedades y Decisiones: Se refiere a aquellos requisitos que no vienen definidos de una conclusión directa y general de todo el equipo a partir de la información obtenida del cliente en las reuniones, sino de aquella que el equipo no supo determinar del todo y obtuvo una conclusión de ellas sin la máxima seguridad de que sea lo que el cliente originalmente pidió. |

### Términos de la sección Requisitos No Funcionales

| Término | Definición |
|---|---|
| RM | Rendimiento y Mantenibilidad. Se refiere a esos requisitos relativos principalmente a la eficacia del sistema, así como su gestión de los recursos y la facilidad con la que este pueda actualizarse y/o corregirse. |
| PE | Portabilidad y Escalabilidad. Para aquellos requisitos centrados en la cualidad del sistema que le permite ser compatible con distintos dispositivos y sistemas operativos y la que asegura que soporte altas cargas de trabajo. |
| AA | Accesibilidad y Adaptación. Requisitos más cercanos al usuario y a su experiencia con el sistema. Búsqueda de un sistema amigable y personalizable para el usuario. |
| INT | Integridad y Trazabilidad. En relación a los datos y la necesidad de mantener su precisión, fiabilidad y consistencia, así como de hacerse un seguimiento sobre ellos y sobre otros aspectos del sistema. |
| IND | Independencia. Referente a los requisitos que destacan la capacidad del sistema de operar por sí solo, sin depender de otros programas o un hardware concreto. |
| SEG | Seguridad. Para los requisitos enfocados en las posibles vulnerabilidades del sistema, ya sea frente ataques o filtraciones accidentales y cómo enfrentarlas. |
| LEG | Legislación. Requisitos legales que debe cumplir el sistema. |

### Otros términos

| Término | Definición |
|---|---|
| RB | Referente a las reglas de negocio, que no representan por sí mismas una funcionalidad pero condicionarán el funcionamiento del sistema. |
| RI | Reunión Inicial. Denota la primera reunión original con el cliente donde nos presentó su producto y nos introdujo su problema con la gestión manual de las facturas. |
| RA | Reunión Ampliada. Denota la segunda reunión que se tuvo con los clientes donde se obtuvo más información sobre el funcionamiento que se espera del sistema. |