**Especificación de los casos de uso**  
En este documento se realizará una especificación de cada caso de uso obtenido mediante el análisis de las 2 reuniones que tuvimos con el cliente. Todas ellas se realizarán con el mismo formato comenzando con un nombre, identificador, objetivo y actores y un desarrollo posterior que abarcara las pre y post condiciones y los diferentes flujos del dicho caso.

Antes de comenzar, para algunos de los apartados como el identificador o los actores se usarán una serie de identificadores que describen a estos y que pueden encontrarse explicados en la sección “Glosario y Definiciones” del documento de requisitos del proyecto y en una tabla que se encontrará al final de este documento.

De la misma forma al final de cada especificación se incluirá una serie de identificadores que denotan una serie de requisitos funcionales con los que dicho caso de uso esté relacionado, incluyéndose el significado de estos identificadores en la tabla al final del documento y en la sección “Glosario y Definiciones” del documento de requisitos.

A continuación se muestran las especificaciones de todos los casos de uso divididos en distintos bloques en función del tipo de funcionalidades que el actor espere realizar.

**1 Acceso y Usuarios:**  
**CU-01: Inicio de sesión:**  
**Objetivo:** Mostrar el comportamiento esperado del sistema cuando uno de los empleados de la empresa intenta iniciar sesión en la plataforma, independientemente del rol que tenga, detallando el flujo de acontecimientos esperados para un inicio de sesión correcto así como la actuación del sistema ante un inicio erróneo o la necesidad de recuperar la contraseña del empleado.

**Actores: CE** 

**Precondiciones:** 

- El usuario que desee iniciar sesión debe poseer un email del dominio de TurbineH así como una contraseña.   
- Además el sistema no permite un autorregistro, luego se espera que para realizar este caso de uso el usuario se encuentre en ese momento con sus sesión en la aplicación no iniciada.

**Flujo Principal:**

- El usuario selecciona la opción de iniciar sesión en la plataforma.  
- El usuario introduce su correo electrónico y su contraseña cuando la plataforma se lo pide.  
- Suponiendo que la información introducida es correcta, el sistema procesa la información, comprueba que, efectivamente, se corresponde a una cuenta de la aplicación y confirma al usuario que los datos son correctos.  
- El sistema le muestra al usuario la interfaz principal de la aplicación, ajustando esta a múltiples parámetros, como su rol en la aplicación (y en función de este los datos de sus clientes asignados) y los últimos movimientos y facturas pagadas o impagadas de los clientes a su disposición.

**Flujo Alternativo:**  
**1º Recuperación de la contraseña**, en caso de que el usuario no recuerde su contraseña, el sistema ofrece la alternativa de enviarle un correo a su email de la empresa con la opción de establecer un cambio de contraseña y poder acceder a la aplicación, el flujo de acciones en este caso sería el siguiente:

- El usuario selecciona la opción de iniciar sesión en la aplicación.  
- Con o sin fallos de inicio de sesión, al ver que no se acuerda de su contraseña, selecciona la opción de recuperar la contraseña.  
- El sistema le manda un correo a su email notificando al usuario de que alguien ha solicitado una recuperación (cambio) de la contraseña.  
- Al haber sido él, el usuario la acepta y establece una nueva contraseña y el sistema actualiza la información de su cuenta.  
- Aquí volvemos al principio, y el flujo podría repetirse, pero en caso de que el usuario ya pueda introducir su información correctamente, a partir de aqui, las acciones son iguales que el flujo principal.

**2º Cancelación**: Este es sencillo, el sistema debe proporcionar al usuario la opción de cancelar un inicio de sesión y volver a la pantalla donde solo puede elegir iniciar sesión, es decir:

- El usuario selecciona la opción de iniciar sesión en la aplicación.  
- En lugar de introducir nueva información, el usuario decide cancelar el inicio de sesión de alguna manera (una opción directa de cancelar, la opción de volver atrás en el buscador o incluso cerrar el propio navegador).  
- El sistema debe cancelar el inicio de sesión del usuario y devolver a la pantalla del inicio (siempre y cuando no se haya cerrado la aplicación).

**Flujo de Excepción:**   
Este flujo describe la reacción del sistema en caso de que un inicio de sesión no vaya de la forma correcta y las opciones que tendrá el usuario para recuperarlo.

- El usuario selecciona la opción iniciar sesión en la aplicación.  
- Introduce una información que no se relaciona con ninguna cuenta almacenada en la base de datos de la empresa.  
- Ante eso, el sistema muestra un mensaje de error genérico del tipo “Correo electrónico o contraseña incorrecta” pidiendo que el usuario introduzca una información correcta.  
- Ahora el usuario tiene la posibilidad de repetir el proceso, volviendo a introducir una información incorrecta (flujo de excepción), una información correcta (flujo principal) o pedirle al sistema recuperar su contraseña (flujo alternativo).

**Postcondiciones:**

- El usuario a partir de ahora tiene iniciada una sesión iniciada en la aplicación y partir de ese momento puede realizar sus operaciones asignadas hasta que se produzca un cierre de sesión  
- En caso de que se haya llevado a cabo el flujo alternativo, se habrán producido uno o más cambios en la información de la cuenta del usuario.

**Requisitos funcionales asociados: USU-01**

**CU-01.5: Cierre de sesión:**  
**Objetivo:** Describe el comportamiento del sistema cuando el usuario intenta cerrar su sesión en la plataforma, también independiente a su rol. Se mostrará la secuencia de acciones realizada por el sistema para un cierre manual de la sesión así como posibles condiciones que produzcan un cierre de la sesión incluso si el usuario no lo ha solicitado.

**Actores: CE**

**Precondiciones:** 

- El usuario que desee realizar el cierre de la sesión debe tener en ese momento una sesión iniciada con su cuenta en la plataforma.

**Flujo Principal:**

- Dentro de la propia interfaz de la plataforma el usuario seleccionará la opción de cerrar sesión que la plataforma le ofrecerá.  
- El sistema le mostrará al usuario una ventana con una pregunta de si el usuario está seguro de querer cerrar la sesión.  
- Suponiendo que el usuario seleccione que está seguro, el sistema deberá cerrar la sesión del usuario en la plataforma y cambiar la interfaz a la que este tiene acceso, a una donde solo tenga la elección de iniciar la sesión.

**Flujos alternativos:**  
**1º Cierre de sesión por inactividad:** Se produce cuando el sistema detecta que el usuario lleva una cierta cantidad de tiempo (modificable por el superadministrador) sin interactuar en la aplicación con la sesion iniciada y procede al cierre de sus sesión, con el siguiente flujo de acciones:

- El sistema detecta que el usuario lleva demasiado tiempo inactivo y con la sesión iniciada en la plataforma.  
- El sistema produce el cierre de la sesión del usuario en la plataforma.  
- El sistema envía al usuario a la página principal del inicio de sesión.

**2º Cierre por cerrar la ventana o por pérdida de la conexión:** Ya sea por cerrar la ventana del navegador o la propia aplicación que se haya usado para acceder a la plataforma. Este cierre viene debido a que el sistema no tiene autorregistros y, por tanto, cualquier nuevo intento de iniciar sesión requiere introducir los datos del usuario, y en este caso, al abandonar directamente la conexión con la plataforma un nuevo intento de entrar requiere volver a iniciar sesion y por ello un cierre previo. El flujo de actos es:

- El usuario abandona de una u otra forma su interacción o conexión con la plataforma.  
- La plataforma detecta esto y cierra la sesión del usuario en la plataforma.  
- Cuando el usuario vuelva a la plataforma, el sistema le mostrará la página de inicio de sesión.

**3º Cancelación:** El más simple, cuando el usuario decide cancelar la operación del cierre de sesión, y las acciones se reducen a:

- El usuario intenta cerrar sesión  
- En lugar de confirmar cancela el cierre de sesión  
- El sistema le devuelve a la plataforma en la que estaba.

**Flujo de excepción:**  
Realmente no hay flujos de excepción del todo definidos en este caso de uso, lo más destacado sería que si debido a una mala conexión o una operación importante que se ejecute en segundo plano y no se haya terminado no se le permita cerrar sesión al usuario hasta que se permita. Pero, salvo modificaciones de las preconfiguraciones, no se han identificado ninguno de estos casos y podría darse simplemente por la tardanza de las peticiones por mala conexión.

**Postcondiciones:** 

- El usuario ahora ha perdido la sesión iniciada en la plataforma que tenía, y para volver a realizar acciones sobre ella debe volver a iniciar sesión.

**Requisitos funcionales asociados: USU-01**

**CU-02: Crear o invitar empleados (usuarios en la aplicación):**  
**Objetivo:** Describe el poder que tienen los administradores y el superadministrador de registrar a nuevos empleados en la aplicación y asignarles un rol en la plataforma.

**Actores: AD y SA**

**Precondiciones:** 

- El usuario que quiera realizar estas acciones debe tener una sesión iniciada en la aplicación, además, debe pertenecer a los roles administrador o superadministrador.  
- Por último debe contar con la información del usuario que va a registrar, siendo como el primer caso, un email del dominio de TurbineH y una contraseña elegida por el usuario.

**Flujo principal:**

- El (super)administrador selecciona la opción de registrar a un usuario que la interfaz de la plataforma le proporcionará.  
- El sistema le pedirá al (super)administrador introducir la información del nuevo usuario que desea registrar.  
- En caso de que la información sea correcta (ver el flujo de excepción de este caso), el sistema le pedirá al (super)administrador asignarle un rol al nuevo usuario, aunque sin la opción del superadministrador, cuyo cambio se realiza de forma distinta, pero con la opción del administrador habilitada.  
- Una vez asignado, el sistema creará la cuenta del nuevo usuario/administrador y a partir de ese momento el empleado podrá iniciar y cerrar sesión en la plataforma (CU-01 y CU-02).  
- En caso de que el usuario fuera registrado y posteriormente desactivado (CU-04) el sistema debe permitir recuperar el historial de movimientos de dicho usuario (pero por defecto no los clientes asignados) y asignarlos a la cuenta registrada para que pueda volver a verlos el usuario.

**Flujos alternativos:**  
No se han concluido flujos alternativos significativos para este caso de uso, pues aunque otra interpretación podría haberlos obtenido a través de otra forma de registrar empleados, como registrar un cliente y al querer asociarlo a un usuario, crear uno en ese instante, el equipo ha decidido que en esos casos la asignación se limite a los usuarios ya existente, por tanto no existe otra manera de empezar la creación de un usuario.  
Eso nos dejaría con el flujo alternativo más simple de todos, **la cancelación del intento de creación**, cuyo flujo de acciones se resume en tratar de comenzar el proceso de creación y durante la introducción de su información o elección de su rol cancelar el proceso.

**Flujo de Excepción**  
**Información errónea o repetida:** Describe el comportamiento del sistema en el caso de que el (super)administrador introduzca una dirección de correo que no exista o no tenga el dominio de TurbineH, o si detecta que el correo introducido es correcto pero ya se había registrado previamente. En ambos casos el sistema cancelará la operación y volverá al inicio, cambiando solo el mensaje que el (super)administrador verá. El flujo es:

- El (super)administrador comienza la operación de registro de empleado.  
- Introduce un correo no existente/sin dominio de TurbineH o uno que ya haya sido registrado previamente (y una contraseña cualquiera).  
- El sistema detecta que la información es errónea o está repetida y le muestra por pantalla al (super)administrador un mensaje con información (“El correo introducido no existe o no tiene el dominio correcto” o “El correo introducido ya ha sido registrado previamente” respectivamente)  
- El sistema cancela la operación y vuelve a la pantalla principal.

**Post-condiciones:** 

- De ahora en adelante (y hasta una baja del empleado), el empleado podrá iniciar sesión en la plataforma y su correo ya no podrá volver a ser registrado.   
- El sistema guardará la información de la nueva cuenta en la base de datos.

**Requisitos funcionales asociados: USU-01, USU-02.**

**CU-02.5: Desactivar empleados:**  
**Objetivo:** La capacidad de un administrador o superadministrador de eliminar la cuenta de un empleado de tipo usuario, conservando todos los movimientos que haya realizado este en la aplicación y requiere un nuevo registro para recuperar esa cuenta.

**Actores: SA y AD.**

**Precondiciones:** 

- El (super)administrador debe tener una sesión iniciada en la plataforma y tener la información del usuario cuya cuenta desee desactivar.

**Flujo principal:**

- El (super)administrador selecciona el usuario cuya cuenta desee desactivar.  
- El (super)administrador selecciona la opción de desactivar dicha cuenta.  
- El sistema le pregunta si está seguro de querer realizar esa acción y de los efectos que esa tendrá.  
- Si selecciona que sí, el sistema desactivará la cuenta de dicho usuario y devolverá al (super)administrador a la página principal.  
- El sistema comenzará la reasignación de todos los clientes asociados a dicho usuario desactivado (CU-05 1º Flujo alternativo).  
- El sistema guardará en la base de datos todos los movimientos realizados por la cuenta desactivada y mantendrá la asignación de estos (y solo estos) con el correo desactivado en caso de volver a ser registrado. 

**Flujos alternativos:**  
Al igual que antes, por el funcionamiento que el equipo espera darle a la plataforma, no existe flujos alternativos significativos, solamente el **de cancelación**, que consta de que el (super)administrador accede al usuario que quiere desactivar, le da a la opción de hacerlo pero en lugar de confirmar cancela la operación y vuelve a la pantalla de usuarios.

**Flujos de excepción:**  
**1º Falta de usuarios:** Es poco probable que este se de, pero describe el caso en que el sistema solo tenga 1 usuario o la cantidad de clientes de la empresa sea tal que no permita otra asignación debido a limitaciones en las asignaciones (parámetros de las preconfiguraciones) y por ende se cancela la petición al requerir la aplicación de ese usuario en la plataforma. El flujo sería:

- El (super)administrador intenta realizar la desactivación del usuario  
- El sistema informa al (super)administrador que debido a una baja cantidad de usuario y/o exceso de clientes la acción no se puede llevar a cabo.  
- Le devuelve a la plataforma de usuarios.

Ese sería el único significativo aunque también se pueden mencionar los relacionados a fallos en la conexión o en la plataforma que dificulten que la acción se lleve a cabo, aunque estos están más relacionados al tiempo que tardan en llegar las peticiones que en acciones del sistema en sí.

**Postcondiciones:** 

- El usuario anteriormente registrado ahora tiene su cuenta desactivada y no puede volver a entrar en ella hasta que se le registre con el mismo correo,   
- Los clientes que tuviera este asignado han sido reasignados a otros usuarios y su historial se ha guardado en la base de datos manteniendo la asignación con el correo.

**Requisitos funcionales asociados: USU-01, USU-02**

**CU-03: Asignar clientes:**  
**Objetivo:** Este caso de uso habla de las acciones que realizará el sistema para que el (super)administrador asigne un cliente a un usuario, distinguiendo entre las asignaciones estándar, las asignaciones temporales, las migraciones masivas y los efectos de cualquier tipo de migración.

**Actores: SA y AD**

**Precondición:** 

- El usuario tiene que tener una sesión iniciada y ser del rol (super)administrador.   
- Además debe tener los datos de un empleado de tipo usuario en la aplicación que no tenga asignado al cliente que se espera asignar.

**Flujo Principal:**

- El (super)administrador debe entrar en el perfil del cliente que vaya a ser asignado.  
- Dentro de este debe seleccionar la opción de asignar/reasignar (según si ya estaba asignado o no) al cliente a un usuario.  
- El sistema le pedirá introducir los datos del usuario al que se le quiera asignar el cliente y confirmar la reasignación.  
- Una vez el (super)administrador introduzca los datos correctos el sistema cambiará la asignación del cliente a la nueva.  
- El sistema transferirá las responsabilidades que tenía la antigua asignación a la nueva asignación (En caso de haber una antigua asignación).

**Flujos alternativos:**  
**1º El flujo de cancelación** que ya se ha tratado en otros casos de uso y que a partir de ahora no se desarrollará al ser análogo en todos los casos de uso, consisten en comenzar la operación y en algún paso antes de introducir los datos correctos cancelarla de cualquier manera (volver atrás, cerrar la página, cerrar sesión, etc.)

**2º Migración temporal:** Describe las migraciones de clientes temporales, donde se heredan todas las responsabilidades y se hace el mismo cambio que el flujo principal, pero solo durante un tiempo limitado que será modificable por los administradores, y posteriormente se le devuelve la asignación al usuario original, lo cual nos da 2 flujos de acciones, ambos muy parecidos:

- En primer lugar el administrador selecciona la delegación temporal e introduce los datos correctos, y se desarrollan 2 flujos

**Flujo de delegación:**

- El sistema cambiará la asignación del cliente al nuevo usuario  
- El sistema delegara las responsabilidades al nuevo usuario.

Y una vez haya llegado la fecha de caducidad se produce el **flujo de devolución:**

- El sistema devolverá la asignación al usuario original  
- El sistema devolverá las responsabilidades al usuario original.

Donde el segundo se realizará automáticamente una vez llegue la fecha de caducidad de la delegación.

**3º Migración masiva:** Sucede cuando un (super)administrador desactiva a un usuario de la plataforma y se deben reasignar todos los clientes a un nuevo usuario. Pero para facilitar la reasignación y no correr el riesgo de tener que reasignar un montón de clientes uno a uno, el (super)administrador reasignará a todos los clientes a un mismo usuario, el flujo sería:

- El (super)administrador desactivará al usuario que tenga asignados los clientes.  
- El sistema le pedirá introducir los datos del usuario que heredará las asignaciones del cliente desactivado.  
- Una vez introducida el sistema se encargará de heredar automáticamente todas las asignaciones del cliente eliminando al seleccionado  
- También asignará las responsabilidades de los clientes asociados al empleado desactivado hacia el nuevo usuario.

**Flujos de excepción:**  
**1º Datos incorrectos:** Al igual que otras veces si el (super)administrador introduce unos datos incorrectos el sistema le avisará de que debe introducir los datos de un nuevo usuario para hacer las asignación y volverá a la pantalla anterior para que el administrador lo vuelva a intentar de nuevo. Este flujo de excepción se aplica igual a todos los tipos de asignación que se han visto.

**2º Exceso de clientes:** Es más raro pero en los parámetros de las preconfiguraciones puede existir un límite de los clientes que pueda tener asignado un usuario (por la carga de trabajo que supondría por ejemplo) y en caso de que una asignación viole el límite de asignaciones, el sistema debe impedir que se realice la asignación y avisar al usuario de que dicho usuario no puede tener tantos clientes asignados y le devuelve a loa pantalla anterior para cambiar al usuario.

**3º Usuario desactivado durante la caducidad:** En caso de que el sistema no detecte como activo al usuario original cuando se debería devolver la asignación, se optará por mantenerla en el nuevo cliente, eliminando la fecha de caducidad, de forma que si luego se reactiva el cliente original, la reasignación se tendrá que hacer manualmente.

**Postcondiciones:**

- El usuario seleccionado ahora tendrá asignado los movimientos y las responsabilidades del usuario seleccionado, ya sea de forma permanente o temporal  
- Y en caso de haber sido una reasignación, el usuario original habrá perdido el control sobre dicho cliente.   
- También el sistema refleja en la información del propio cliente el cambio de usuario asignado.

**Requisitos funcionales asociados: USU-03, CLI-03, PRO-03**

**CU-03.5: Quitar asignaciones de clientes:**  
**Objetivo:** Describe cuando un (super)administrador quiere eliminar la asignación de un cliente a un usuario y los efectos que conlleva.

**Actores: SA y AD**

**Precondiciones:** 

- El empleado debe tener una sesión iniciada en la plataforma con el rol de (super)administrador,   
- Además debe tener los datos de un nuevo usuario al que se va a reasignar el cliente.

**Flujo Principal:** 

- El administrador selecciona al cliente que vaya a eliminar su asignación.  
- El sistema le pide confirmación para realizar esa acción.  
- Antes de realizar la acción se le pide al (super)administrador la información del nuevo usuario al que se vaya a reasignar al cliente, pues por defecto se espera que cada cliente esté asignado a un usuario.  
- Si la información es correcta, el cliente eliminará del usuario original la información y los movimientos del cliente.  
- Con ello el usuario también pierde las responsabilidades sobre dicho cliente.  
- A partir de aquí ya se ejecutaría el flujo del caso CU-05 pero sin introducir la información, o sea heredarle al nuevo usuario la información del cliente.

Flujos alternativos:  
**1º El flujo de cancelación en el caso estándar.**

**2º Eliminación tras caducidad:** Describe cuando llega la fecha de caducidad de una delegación temporal y el sistema se encarga automáticamente de quitarle el cliente a el usuario y devolverlo al que lo tenía originalmente. Su traza sólo está compuesta por los pasos de eliminación de los datos y asignación al nuevo usuario, eliminando la introducción de los datos, al haberse encargado de eso el que pidió la delegación temporal.

**3º Migración Masiva:** Cuando un (super)administrador quiera desactivar a un usuario debe migrar a todos sus clientes asignados, por tanto el (super)administrador le indicará al sistema el usuario que recibirá las asignaciones y este se encargará de eliminar toda la información y movimientos del usuario que se va a desactivar.

**Flujos de excepción:**  
**1º Datos incorrectos para la reasignación**, el mismo flujo que el del caso de uso anterior, y por los mismos motivos.

**2º Devolución imposible:** Realmente es el mismo caso que el caso anterior y el sistema hará que la asignación al nuevo cliente pase a ser permanente y no se deshaga automáticamente ni aunque se reactive al usuario original.

**3º Único usuario o transferencia imposible:** Este es el más raro, pero en caso de que la transferencia del cliente no sea posible para ningún usuario, el sistema ni debe permitir que se quite la asociación de este a su usuario actual.

**Postcondiciones:** 

- La opuestas al anterior, el usuario original pierde el acceso a la información y movimientos del clientes desasignado,   
- A su vez pierde las responsabilidades sobre este, y el nuevo gana todas estas por supuesto.

**Requisitos funcionales asociados: USU-03, CLI-03, PRO-03**

**CU-04: Registrar un cliente:**  
**Objetivo:** Describe el comportamiento del sistema cuando un empleado con permisos registra una nueva empresa cliente en la plataforma junto con sus personas de contacto, diferenciando los datos obligatorios para la facturación de los opcionales, validando posibles duplicados y permitiendo reactivar fichas de antiguos clientes que regresan**.** 

**Actores: AD, SA y UO (con permiso)** 

**Precondiciones:**

* El usuario debe tener una sesión iniciada en la plataforma y contar con el rol de superadministrador o de usuario operativo con permisos habilitados para dar de alta clientes (realizándose cualquier acceso no autorizado según USU-04).  
* El empleado debe disponer al menos de los datos básicos de la empresa cliente y de sus contactos para iniciar el registro.

**Flujo Principal:**

* El usuario selecciona la opción de registrar un nuevo cliente dentro de la interfaz de la plataforma.  
* El sistema muestra el formulario de alta distinguiendo claramente entre los datos imprescindibles para poder facturar y aquellos datos que son únicamente convenientes u opcionales.  
* El usuario introduce los datos de la empresa (nombre comercial, razón social, identificador fiscal, domicilio, país y moneda) así como los datos de una o varias personas de contacto indicando la función de cada una.  
* Antes de finalizar, el sistema muestra por pantalla un resumen con toda la información introducida para que el usuario la revise antes de confirmar.  
* El usuario confirma la operación y el sistema guarda la ficha del nuevo cliente en la base de datos.

**Flujos Alternativos:**

**1º Flujo de cancelación:** Análogo al de los casos de uso anteriores; el usuario decide cancelar el proceso de alta en cualquier momento previo a la confirmación definitiva y el sistema descarta los datos sin guardar cambios. 2º Reactivación de un cliente que vuelve: Sucede cuando se intenta registrar un cliente cuyo identificador fiscal ya existía en la base de datos por haber contratado los servicios en el pasado (estando de baja o anonimizado según el CU-30). 

**El flujo sería:**

* El usuario introduce el identificador fiscal del cliente.  
* El sistema detecta que pertenece a un antiguo cliente y ofrece la opción de reactivar su ficha recuperando sus datos e historial previo.  
* El usuario acepta la reactivación, revisa o actualiza los datos que hayan podido cambiar en el resumen previo y confirma la operación.  
* El sistema reactiva la ficha del cliente y pospone indefinidamente cualquier proceso de borrado definitivo. 3º Registro con datos fiscales incompletos: Si el usuario registra al cliente introduciendo solo datos básicos o convenientes pero dejando pendientes algunos datos fiscales imprescindibles para facturar, el sistema permite guardar la ficha pero la marca como incompleta de cara a la emisión de facturas o activación de recurrencias (CU-08 y CU-13).

**Flujos de Excepción:**

**1º Identificador fiscal ya existente (duplicado activo):** Si el usuario introduce un identificador fiscal que ya pertenece a un cliente activo en el sistema, la plataforma avisa inmediatamente de la duplicidad e impide crear una ficha repetida. 2º Evitar duplicados por doble clic o concurrencia: Si el usuario pulsa repetidamente el botón de confirmar o dos usuarios intentan registrar el mismo cliente simultáneamente, el sistema bloquea las peticiones redundantes para evitar registros duplicados (PRO-02).

**Postcondiciones:**

* El nuevo cliente (o el cliente reactivado) queda registrado en la base de datos de la plataforma junto con sus contactos asociados.  
* Su ficha pasa a estar disponible para ser asignada, gestionada o vinculada a una suscripción.

**Requisitos funcionales asociados: CLI-01, CLI-02, CLI-04, USU-04, PRO-02**

**CU-05: Gestionar la ficha de un cliente:**

**Objetivo:** Describir las acciones que puede realizar un usuario para consultar la información detallada y la línea temporal de un cliente, así como modificar sus datos, gestionar sus personas de contacto y asignarle etiquetas de clasificación. 

**Actores: AD, SA y UO (sus clientes asignados)**

**Precondiciones:**

* El empleado debe tener una sesión iniciada en la plataforma.  
* El cliente que se desea consultar o editar debe estar registrado en el sistema y, en el caso de tratarse de un usuario operativo (UO), dicho cliente debe estar asignado a su cartera.

**Flujo Principal:**

* El usuario accede al perfil o ficha del cliente correspondiente desde la interfaz de la plataforma.  
* El sistema muestra los datos actuales del cliente junto con su línea temporal de actividad y movimientos.  
* El usuario selecciona la opción de editar la ficha y realiza las modificaciones necesarias: actualizar datos de la empresa, añadir nuevos contactos, desactivar contactos existentes o asignar/quitar etiquetas descriptivas (como «riesgo de impago» o «cliente estratégico»).  
* El usuario confirma los cambios realizados.  
* El sistema valida la información, actualiza la ficha del cliente en la base de datos y registra el cambio en su línea temporal.

**Flujos Alternativos:**

**1º Flujo de cancelación:** El usuario descarta las modificaciones antes de guardar y la ficha mantiene sus datos originales. 2º Solo consulta de la ficha y línea temporal: El usuario accede a la ficha únicamente para visualizar los datos de la empresa, sus contactos o revisar el historial en la línea temporal, saliendo de la pantalla sin realizar ninguna edición.

**Flujos de Excepción:**

**1º Edición simultánea o datos inválidos:** Si dos usuarios intentan editar la misma ficha a la vez o se produce un doble envío de guardado, el sistema previene la sobrescritura accidental o duplicidad de contactos (PRO-02) y avisa al usuario en caso de introducir algún formato de dato no válido. 

**2º Eliminación de datos fiscales obligatorios con suscripción activa:** Si el usuario intenta borrar un dato fiscal imprescindible en un cliente que tiene facturación activa, el sistema avisa de la restricción e impide dejar la ficha en un estado incoherente.

**Postcondiciones:**

* La información de la empresa, el estado de sus contactos y sus etiquetas quedan actualizados en la base de datos.  
* La línea temporal del cliente refleja las modificaciones realizadas.

**Requisitos funcionales asociados: CLI-01, CLI-02, CLI-05, USU-04, PRO-02**

**CU-06: Importar datos desde CSV:**

**Objetivo:** Mostrar el funcionamiento del sistema cuando un superadministrador realiza una carga masiva de fichas de clientes o de un histórico de facturas mediante un archivo CSV, incluyendo el mapeo de columnas, la validación previa de errores o duplicados y el registro del resultado de la importación. 

**Actores: AD y SA** 

**Precondiciones:**

* El usuario debe tener una sesión iniciada en la plataforma con el rol de administrador o superadministrador.  
* Debe disponer de un archivo en formato CSV con los datos de los clientes o del histórico de facturas que desea importar.

**Flujo Principal:**

* El superadministrador accede a la herramienta de importación de datos y selecciona el archivo CSV desde su equipo, indicando si se trata de una importación de clientes o de histórico de facturas.  
* El sistema procesa el archivo y muestra una pantalla con el mapeo de columnas y una vista previa de los datos que se van a cargar.  
* El (super)administrador ajusta el mapeo de columnas si fuera necesario y revisa los posibles avisos mostrados por el sistema.  
* Tras confirmar la operación, el sistema importa los registros válidos en la base de datos.  
* En el caso de tratarse de un histórico de facturas, el sistema las registra con el estado de «migradas», sin enviarlas por correo a los clientes ni consumir la numeración oficial de las series actuales.  
* Finalmente, el sistema deja constancia de la importación realizada y muestra por pantalla un resumen detallado del proceso.

**Flujos Alternativos:**

**1º Flujo de cancelación:** El superadministrador revisa la vista previa o los errores detectados y decide cancelar la importación antes de confirmar, sin que se modifique la base de datos. 

**2º Importación parcial con omisión de duplicados o filas erróneas:** Si durante la vista previa el sistema avisa de registros duplicados o filas con errores de formato, el superadministrador puede optar por continuar importando únicamente los registros correctos y omitir los conflictivos, quedando reflejado en el resumen final.

**Flujos de Excepción:**

**1º Archivo corrupto o formato incompatible:** Si el archivo subido no es un CSV válido, está vacío o su estructura impide realizar el mapeo mínimo de columnas obligatorias, el sistema muestra un mensaje de error detallando el fallo y cancela el proceso de carga.

**Postcondiciones:**

* Los clientes importados quedan dados de alta en la base de datos y las facturas históricas quedan almacenadas bajo el estado «migradas» sin haber generado envíos ni alterado la numeración activa.  
* Queda guardado en el sistema el registro de la importación junto con su resumen correspondiente.

**Requisitos funcionales asociados: IMP-01, FAC-07, USU-04, PRO-02**

**CU-07: Dar de baja a un cliente:**

**Objetivo:** Describir el procedimiento mediante el cual se tramita la baja efectiva de un cliente en la plataforma, asegurando que se detenga la generación de futuras mensualidades pero manteniendo activo el proceso de recobro de las facturas pendientes anteriores y aplicando un borrado lógico sobre su ficha. 

**Actores: AD, SA y UO (sus clientes asignados)** 

**Precondiciones:**

* El usuario debe tener su sesión iniciada en la plataforma y ser superadministrador o el usuario operativo responsable del cliente.  
* El cliente debe encontrarse actualmente en estado activo dentro del sistema.

**Flujo Principal:**

* El usuario accede a la ficha del cliente que solicita la baja y selecciona la opción de dar de baja al cliente.  
* El sistema solicita indicar la fecha efectiva de la baja, el motivo de la misma y pide confirmación al usuario informando de las consecuencias de la acción.  
* El usuario confirma la operación.  
* El sistema registra la baja efectiva a partir de la fecha indicada, desactivando la generación de nuevas facturas recurrentes o mensualidades para ese cliente.  
* El sistema aplica un borrado lógico sobre la ficha del cliente (no se elimina físicamente de la base de datos, conservando su historial y permitiendo su posterior tratamiento según el CU-30).  
* En caso de que el cliente tuviera facturas emitidas pendientes de pago o impagadas previas a la baja, el sistema mantiene activo el flujo de seguimiento y recobro de dichas deudas.

**Flujos Alternativos:**

**1º Flujo de cancelación:** El usuario decide no confirmar la baja en el cuadro de diálogo y el cliente permanece activo sin cambios. 

**2º Baja originada tras aviso por API:** Si un sistema externo de TurbineH notificó previamente la solicitud de baja (CU-31) generando una tarea pendiente para el responsable, el usuario accede desde dicha tarea para ejecutar este caso de uso y, al completarlo, la tarea queda marcada como resuelta.

**Flujos de Excepción:**

**1º Cliente ya dado de baja:** Si por un error de concurrencia o doble petición (PRO-02) se intenta dar de baja a un cliente que ya figura como inactivo, el sistema avisa al usuario e impide repetir la operación.

**Postcondiciones:**

* El cliente pasa a estado de baja (borrado lógico) y deja de generar facturación recurrente a partir de la fecha de efecto.  
* Las facturas pendientes anteriores continúan su ciclo normal de cobro y reclamación hasta ser saldadas.

**Requisitos funcionales asociados: SUB-04, CLI-03, CLI-04, AMB-04, USU-04, PRO-02**

### **Suscripción, tarifas y aprobaciones**

**CU-08: Gestionar la suscripción de un cliente:**

**Objetivo:** Describir las acciones necesarias para registrar y modificar la relación comercial recurrente con un cliente (plan, tarifa, condiciones, fecha de inicio y documentos), incluyendo el cálculo del prorrateo del primer mes y las opciones de pausar o reactivar una suscripción.

 **Actores: AD, SA y UO (sus clientes asignados)** 

**Precondiciones:**

* El empleado debe haber iniciado sesión en la plataforma con rol de superadministrador o usuario operativo asignado al cliente.  
* El cliente debe estar registrado en la plataforma.

**Flujo Principal:**

* El usuario accede al apartado de suscripción dentro de la ficha del cliente y selecciona crear o configurar su suscripción.  
* El usuario introduce los datos de la relación comercial: fecha de inicio (pudiendo ser actual o futura), plan contratado, tarifa aplicable, usuario responsable, condiciones de pago y adjunta los documentos firmados correspondientes.  
* Si la fecha de inicio no coincide con el día 1 del mes, el sistema calcula automáticamente el prorrateo correspondiente al primer mes y lo muestra por pantalla antes de confirmar.  
* El sistema comprueba que la ficha del cliente cuenta con todos los datos fiscales imprescindibles.  
* El usuario confirma la operación y el sistema activa la recurrencia de la suscripción guardando toda la información en la base de datos.

**Flujos Alternativos:**

**1º Flujo de cancelación:** El usuario cancela la configuración de la suscripción antes de confirmar.

**2º Excepción «facturar desde el mes siguiente»:** Al configurar una suscripción que comienza con el mes ya empezado, en lugar de aplicar el prorrateo del primer mes, el usuario selecciona aplicar la excepción de comenzar a facturar directamente desde el mes siguiente (quedando sujeta a aprobación si corresponde según el CU-10 y CU-11). 

**3º Pausar y reactivar la suscripción:** Cuando un cliente con suscripción activa requiere detener temporalmente el servicio o reanudarlo:

* El usuario accede a la suscripción y selecciona la opción de «Pausar» (o «Reactivar» si estaba pausada).  
* El sistema solicita obligatoriamente introducir el motivo y las fechas previstas de inicio y fin de la pausa (o la fecha de reactivación).  
* Tras confirmar, el sistema cambia el estado de la suscripción a pausada (deteniendo la emisión de facturas del día 1 según el CU-14 mientras dure la pausa) o la devuelve a estado activo según corresponda.

**Flujos de Excepción:**

**1º Falta de datos fiscales imprescindibles:** Si el usuario intenta activar la recurrencia de la suscripción pero al cliente le faltan datos fiscales obligatorios para poder emitir facturas (CU-04), el sistema bloquea la activación y muestra un aviso indicando qué datos deben completarse primero. 

**2º Datos o fechas incoherentes:** Si se introduce una fecha de fin anterior a la de inicio o falta el motivo al intentar pausar la suscripción, el sistema impide guardar los cambios y solicita corregir la información.

**Postcondiciones:**

* La suscripción del cliente queda registrada y actualizada en la base de datos (activa, programada con fecha futura o en pausa) junto con sus documentos firmados y condiciones.  
* El cliente queda incluido (o excluido, si se ha pausado) en el proceso automático de generación de facturas recurrentes del día 1 de cada mes (CU-14).

**Requisitos funcionales asociados: SUB-01, SUB-02, SUB-03, SUB-04, AMB-05, USU-04, PRO-02**

**CU-09: Revisar el rango anual de un cliente:**

**Objetivo:** Mostrar el proceso mediante el cual un superadministrador revisa anualmente la facturación verificada de los clientes para ajustar su rango tarifario, requiriendo aprobación antes de aplicar el nuevo precio y dejando trazabilidad completa del cambio. 

**Actores: AD y SA** 

**Precondiciones:**

* El usuario debe tener una sesión iniciada en la plataforma con rol de administrador o superadministrador.  
* Debe existir al menos un cliente con suscripción activa al que le corresponda su revisión anual de tarifa.

**Flujo Principal:**

* El superadministrador accede al apartado de revisiones anuales, donde el sistema lista automáticamente los clientes pendientes de revisar.  
* El superadministrador selecciona un cliente de la lista e introduce el dato de su facturación anual verificada.  
* En función de la cifra introducida y la tabla de tarifas vigente, el sistema calcula y propone automáticamente el nuevo rango y precio que le corresponde al cliente.  
* La propuesta queda sujeta a validación; una persona autorizada revisa y aprueba el cambio de rango antes de que se modifique el precio en firme (vinculado con el flujo de aprobaciones).  
* Una vez aprobado, el sistema actualiza la tarifa de la suscripción del cliente para las siguientes facturaciones y guarda un registro de auditoría con quién lo aprobó, en qué fecha y hora, el precio anterior y el precio nuevo.

**Flujos Alternativos:**

**1º Flujo de cancelación o rechazo del cambio:** Si el superadministrador cancela la operación o la persona encargada de validar rechaza la propuesta de cambio de rango, el sistema mantiene la tarifa anterior del cliente y registra la decisión. 

**2º Mantenimiento del mismo rango:** Si tras introducir la facturación verificada el sistema determina que el cliente se mantiene dentro del mismo tramo tarifario que ya tenía, se confirma la revisión anual dejando constancia de la comprobación sin alterar el precio de su suscripción.

**Flujos de Excepción:**

**1º Datos de facturación no válidos o falta de permisos de aprobación:** Si se introduce un valor numérico inválido para la facturación verificada, o si el mismo usuario intenta aprobar un cambio cuando las reglas exigen separación de funciones, el sistema bloquea la acción y muestra el mensaje de error correspondiente.

**Postcondiciones:**

* El cliente deja de aparecer como pendiente en la lista de revisiones anuales del periodo actual.  
* Su tarifa se actualiza al nuevo precio aprobado para las próximas facturas y queda almacenado en la base de datos el registro histórico con el autor de la aprobación, la fecha, el importe antiguo y el nuevo.

**Requisitos funcionales asociados: TAR-02, USU-04, PRO-02**

**CU-10: Solicitar una excepción comercial:**

**Objetivo:** Describir cómo un empleado propone para un cliente una condición económica fuera de lo habitual (un descuento temporal, un primer mes gratuito o un precio pactado) y la deja pendiente hasta que la apruebe quien corresponda.

**Actores:** UO (sus clientes asignados), AD y SA

**Precondiciones:**

- El usuario debe tener una sesión iniciada en la plataforma.  
- El cliente debe tener una suscripción registrada (CU-08) y, si el usuario es un UO, debe tenerlo asignado.

**Flujo Principal:**

1. El usuario accede a la suscripción del cliente y selecciona la opción de solicitar una excepción comercial.  
2. El sistema muestra los tipos de excepción disponibles: descuento temporal, primer mes gratuito o precio pactado.  
3. El usuario elige el tipo de excepción e indica su valor (porcentaje o importe), el periodo en que se aplicará y el motivo.  
4. El sistema muestra un resumen con el efecto que tendría la excepción sobre las próximas facturas del cliente.  
5. El usuario confirma la solicitud.  
6. El sistema registra la solicitud como pendiente de aprobación, determina quién debe aprobarla (un administrador, o el superadministrador si supera el umbral configurado) y le envía una notificación.

**Flujos Alternativos:**

**1º Flujo de cancelación:** El usuario cancela la solicitud antes de confirmarla y el sistema no registra nada.

**2º Solicitud de un administrador:** Si quien solicita la excepción es un administrador y la regla exige separación de funciones, el sistema asigna la aprobación a otro administrador distinto del solicitante.

**Flujos de Excepción:**

**1º Datos incompletos:** Si falta el motivo o el valor de la excepción, el sistema marca el campo que falta e impide confirmar, sin perder el resto de datos introducidos.

**Postcondiciones:**

- La solicitud queda registrada como pendiente de aprobación y el aprobador recibe la notificación.  
- La tarifa del cliente no cambia hasta que la solicitud sea aprobada en el CU-11.

**Requisitos funcionales asociados:** APR-01, USU-04, PRO-02

**CU-11: Aprobar o rechazar solicitudes:**

**Objetivo:** Describir cómo un administrador, o el superadministrador cuando se supera el umbral configurado, revisa desde la bandeja de aprobaciones las operaciones económicas que no pueden aplicarse sin validación humana y decide si se aprueban o se rechazan.

**Actores:** AD y SA

**Precondiciones:**

- El usuario debe tener una sesión iniciada en la plataforma con el rol de administrador o superadministrador.  
- Debe existir al menos una solicitud pendiente asignada al usuario. Las solicitudes pueden venir de una excepción comercial (CU-08 y CU-10), de un cambio de rango (CU-09), de una emisión de factura (CU-13), de una rectificación (CU-15) o de una deuda que se quiere cancelar o declarar incobrable (CU-19).

**Flujo Principal:**

1. El usuario accede a la bandeja de aprobaciones.  
2. El sistema muestra las solicitudes pendientes con su tipo, el cliente, el importe, quién la solicitó y el motivo por el que necesita aprobación.  
3. El usuario selecciona una solicitud.  
4. El sistema muestra el detalle de la solicitud y su efecto (por ejemplo, el precio anterior y el nuevo).  
5. El usuario aprueba la solicitud y, si lo desea, añade un comentario.  
6. El sistema aplica la operación aprobada, registra quién la aprobó y cuándo, y notifica al solicitante.

**Flujos Alternativos:**

**1º Rechazo de la solicitud:** En lugar de aprobarla, el usuario la rechaza indicando obligatoriamente el motivo. El sistema no aplica la operación y el solicitante ve el rechazo junto con el comentario.

**2º Reasignación por ausencia:** Si el aprobador asignado no está disponible, un administrador reasigna la solicitud a otra persona o a su suplente, y la reasignación queda registrada.

**Flujos de Excepción:**

**1º Autoaprobación no permitida:** Si el usuario intenta aprobar una solicitud que hizo él mismo y la regla exige separación de funciones, el sistema bloquea la aprobación.

**2º Importe por encima del umbral:** Si un administrador intenta aprobar una solicitud que supera su umbral, el sistema le indica que debe aprobarla el superadministrador y se la deriva.

**Postcondiciones:**

- La solicitud queda aprobada o rechazada y registrada en la auditoría.  
- El solicitante conoce la decisión tomada.

**Requisitos funcionales asociados:** APR-01, APR-02, FAC-06, REC-02, USU-04, PRO-02

## Facturas y envíos

**CU-12: Crear una factura puntual:**

**Objetivo:** Describir cómo un empleado prepara una factura puntual por un servicio adicional (instalación, consultoría, formación, etc.), que queda guardada como borrador hasta que se emita.

**Actores:** AD, SA y UO (sus clientes asignados)

**Precondiciones:**

- El usuario debe tener una sesión iniciada en la plataforma.  
- El cliente debe estar registrado y, si el usuario es un UO, debe tenerlo asignado.

**Flujo Principal:**

1. El usuario selecciona el cliente y elige la opción de crear una nueva factura puntual.  
2. El sistema crea un borrador con los datos del cliente y los valores por defecto (moneda, impuesto y vencimiento).  
3. El usuario añade una o varias líneas: elige un concepto del catálogo o escribe uno nuevo, y ajusta la descripción, la cantidad, el precio, el descuento y el impuesto.  
4. El sistema calcula los subtotales, los impuestos y el total de la factura.  
5. El usuario guarda el borrador.

**Flujos Alternativos:**

**1º Flujo de cancelación:** El usuario cancela la creación antes de guardar y el sistema descarta el borrador.

**2º Emitir en el momento:** Al terminar, el usuario decide emitir la factura directamente. Se continúa con el CU-13, que extiende a este caso de uso.

**3º Editar un borrador existente:** El usuario abre un borrador guardado anteriormente, lo modifica y vuelve a guardarlo.

**4º Eliminar un borrador:** El usuario elimina un borrador que nunca se emitió. El sistema pide confirmación, lo elimina y deja constancia de la eliminación.

**Flujos de Excepción:**

**1º Cierre accidental:** Si el usuario cierra la pantalla sin guardar, el sistema conserva un borrador automático que puede recuperar después.

**Postcondiciones:**

- Queda guardada una factura en estado de borrador, todavía sin número definitivo.  
- Los cambios posteriores en el catálogo de conceptos no modifican esta factura.

**Requisitos funcionales asociados:** FAC-01, FAC-03, USU-04, PRO-02

**CU-13: Emitir una factura:**

**Objetivo:** Describir cómo un borrador se convierte en una factura definitiva: el sistema valida los datos, asigna el número de la serie, genera el PDF y lo envía al cliente. Este caso de uso es incluido por la generación de facturas recurrentes (CU-14).

**Actores:** AD, SA y UO (sus clientes asignados)

**Precondiciones:**

- Debe existir una factura en borrador, creada en el CU-12 o generada en el CU-14.  
- El cliente debe tener al menos un contacto de facturación.

**Flujo Principal:**

1. El usuario abre el borrador y selecciona la opción de emitir.  
2. El sistema comprueba que la factura tiene todos los datos fiscales imprescindibles y que su importe no se aparta del patrón habitual del cliente.  
3. El sistema muestra una vista previa del PDF.  
4. El usuario confirma la emisión.  
5. El sistema asigna el siguiente número de la serie, fija la fecha de emisión y genera y guarda el PDF con la imagen corporativa.  
6. El sistema envía la factura por correo al contacto de facturación y registra el envío (destinatario, canal, fecha y resultado).  
7. El sistema marca la factura como emitida y enviada.

**Flujos Alternativos:**

**1º Flujo de cancelación:** Durante la vista previa, el usuario decide no emitir. La factura sigue como borrador sin cambios.

**2º Importe fuera del patrón:** Si el importe se aparta del patrón del cliente, el sistema muestra una advertencia, pero el usuario puede continuar con la emisión.

**3º Requiere aprobación:** Si la factura tiene un importe modificado a mano, un vencimiento fuera de lo habitual o es una factura puntual de importe muy alto, queda pendiente de aprobación con el motivo explicado. Se emite cuando se aprueba en el CU-11, que extiende a este caso de uso.

**4º Envío por WhatsApp:** Si el contacto ha dado su consentimiento para ese canal, la factura también se envía por WhatsApp (simulado).

**Flujos de Excepción:**

**1º Falta un dato fiscal:** El sistema bloquea la emisión, indica qué dato falta y devuelve al usuario al borrador.

**2º Doble clic o emisión simultánea:** Si el usuario pulsa varias veces o dos personas emiten la misma factura a la vez, el sistema indica que ya se está procesando y no duplica ni la factura ni el número.

**3º Fallo en el envío:** La factura queda emitida pero sin enviar, y el envío puede reintentarse sin generar otro número. Si el correo rebota, se continúa con el CU-17, que extiende a este caso de uso.

**Postcondiciones:**

- La factura tiene un número único dentro de su serie y su PDF queda guardado.  
- El envío queda registrado.  
- A partir de este momento la factura no puede editarse: solo puede corregirse mediante el CU-15.

**Requisitos funcionales asociados:** FAC-04, FAC-05, PDF-01, ENV-01, ENV-02, APR-02, USU-04, PRO-02

**CU-14: Generar las facturas recurrentes (automático):**

**Objetivo:** Describir el proceso automático por el que, el día 1 de cada mes, el sistema genera y emite la factura de todos los clientes con una suscripción activa, sin intervención de ningún empleado.

**Actores:** El propio reloj del sistema.

**Precondiciones:**

- Es día 1 de mes, o el proceso de ese mes no terminó y se está reintentando.  
- Existen suscripciones activas.

**Flujo Principal:**

1. El reloj del sistema inicia el proceso el día 1 del mes.  
2. El sistema obtiene las suscripciones que deben facturarse ese mes.  
3. Para cada suscripción, el sistema genera la factura con los conceptos y la tarifa vigentes, aplicando los descuentos aprobados que sigan en vigor.  
4. El sistema emite cada factura siguiendo el CU-13, que este caso de uso incluye.  
5. El sistema registra la ejecución del proceso: clientes procesados, facturas creadas y fallos.

**Flujos Alternativos:**

**1º Suscripción sin facturar ese mes:** Si la suscripción está pausada, tiene la baja efectiva o empieza en una fecha futura, el sistema no genera su factura y lo deja anotado.

**2º Tarifa nueva:** Si el cliente tiene una tarifa nueva aprobada (CU-09), el sistema la aplica desde su fecha de efecto.

**3º Condiciones de pago pactadas:** Si el cliente tiene un plazo de pago pactado (por ejemplo, a 30 días), el vencimiento de la factura respeta ese plazo en lugar del general.

**4º Descuento caducado:** Si un descuento dejó de estar vigente, el sistema deja de aplicarlo y avisa al responsable del cliente.

**Flujos de Excepción:**

**1º Datos insuficientes:** Si al cliente le falta el contacto de facturación o algún dato fiscal imprescindible, el sistema no emite su factura, avisa a su responsable y continúa con el resto de clientes.

**2º Caída del sistema:** Si el sistema estaba caído el día 1, al volver detecta qué facturas no se generaron y las genera sin duplicar las que ya existían.

**Postcondiciones:**

- Cada suscripción activa tiene emitida y enviada su factura del mes.  
- Cada caso que no pudo facturarse ha llegado a su responsable.  
- La ejecución del proceso queda registrada.

**Requisitos funcionales asociados:** GEN-01, SUB-01, PRO-01

**CU-15: Rectificar una factura emitida:**

**Objetivo:** Describir cómo se corrige una factura ya emitida sin modificarla, mediante una factura rectificativa o nota de crédito vinculada a la original.

**Actores:** AD y SA

**Precondiciones:**

- El usuario debe tener una sesión iniciada en la plataforma con el rol de administrador o superadministrador.  
- La factura que se quiere corregir debe estar emitida.

**Flujo Principal:**

1. El usuario abre la factura emitida y selecciona la opción de rectificar.  
2. El sistema crea una factura rectificativa vinculada a la original, tomando sus datos como punto de partida.  
3. El usuario indica qué líneas o importes se corrigen y el motivo de la rectificación.  
4. El sistema muestra una vista previa y cómo queda el saldo del cliente.  
5. El usuario confirma la rectificación.  
6. El sistema asigna a la rectificativa un número de la serie que corresponda, genera el PDF y la envía al contacto de facturación igual que cualquier factura emitida.  
7. El sistema marca la factura original como rectificada y recalcula el saldo del cliente.

**Flujos Alternativos:**

**1º Flujo de cancelación:** El usuario cancela la rectificación antes de confirmarla y la factura original queda sin cambios.

**2º Importe por encima del umbral:** Si el importe rectificado supera el umbral de aprobación, la rectificativa queda pendiente hasta que se apruebe en el CU-11, que extiende a este caso de uso.

**Flujos de Excepción:**

**1º Falta el motivo:** Si no se indica el motivo de la rectificación, el sistema no permite continuar.

**2º Usuario sin permiso:** Si un UO intenta rectificar una factura, el sistema rechaza la acción.

**Postcondiciones:**

- Existe una factura rectificativa emitida y vinculada a la original.  
- La factura original no cambia, pero aparece como rectificada.  
- El saldo del cliente refleja la corrección y la acción queda registrada en la auditoría.

**Requisitos funcionales asociados:** FAC-06, FAC-02, AMB-01, USU-04, PRO-02

**CU-16: Consultar una factura y descargar su PDF:**

**Objetivo:** Describir cómo un empleado localiza una factura, consulta sus datos y su estado, y descarga el PDF tal como se emitió.

**Actores:** AD, SA y UO (sus clientes asignados)

**Precondiciones:**

- El usuario debe tener una sesión iniciada en la plataforma.

**Flujo Principal:**

1. El usuario busca o filtra las facturas por cliente, estado, tipo o fecha.  
2. El sistema muestra el listado con el número, el cliente, el importe, el vencimiento y el estado de cada factura. Un UO solo ve las facturas de sus clientes asignados.  
3. El usuario selecciona una factura.  
4. El sistema muestra sus datos principales, su estado (borrador, emitida, pagada, anulada, rectificada o incobrable) y su historial de envíos, cobros y rectificaciones.  
5. El usuario selecciona la opción de descargar el PDF.  
6. El sistema entrega el PDF original de la emisión.

**Flujos Alternativos:**

**1º Solo consulta:** El usuario revisa los datos de la factura y sale sin descargar nada.

**2º Factura en borrador:** Si la factura todavía no se ha emitido, el sistema muestra una vista previa en lugar del PDF definitivo.

**Flujos de Excepción:**

**1º Acceso no autorizado:** Si un UO intenta abrir una factura de un cliente que no tiene asignado (por ejemplo, copiando un enlace), el sistema rechaza el acceso.

**Postcondiciones:**

- La factura no se modifica.

**Requisitos funcionales asociados:** FAC-02, PDF-01, USU-04

**CU-17: Resolver un envío fallido:**

**Objetivo:** Describir cómo el responsable de un cliente consigue entregar una factura cuyo envío falló o cuyo correo rebotó, sin crear una factura nueva. Este caso de uso extiende al CU-13.

**Actores:** AD, SA y UO (responsable del cliente)

**Precondiciones:**

- El envío de una factura falló o el correo rebotó.  
- El sistema abrió una incidencia de entrega y notificó al responsable del cliente.

**Flujo Principal:**

1. El responsable abre la incidencia de entrega desde la notificación o desde la pantalla de trabajo diario.  
2. El sistema muestra la factura, el destinatario, el motivo del fallo y los intentos anteriores.  
3. El responsable corrige el correo del contacto o elige otro contacto de facturación.  
4. El responsable selecciona la opción de reenviar.  
5. El sistema reenvía la misma factura, con el mismo número, y registra el envío como reintento.  
6. El envío se realiza correctamente y el sistema marca la factura como notificada y cierra la incidencia.

**Flujos Alternativos:**

**1º Flujo de cancelación:** El responsable sale de la incidencia sin reenviar y esta queda abierta.

**2º Fallo temporal:** Si no hay nada que corregir (por ejemplo, el servidor de correo del cliente estaba caído), el responsable reenvía sin cambiar el contacto.

**Flujos de Excepción:**

**1º Nuevo fallo:** Si el reenvío vuelve a fallar, la incidencia sigue abierta y el nuevo intento queda registrado.

**Postcondiciones:**

- La factura queda notificada, o la incidencia sigue abierta con todos los intentos registrados.  
- El historial distingue el envío original de los reintentos y no se crea ninguna factura nueva.

**Requisitos funcionales asociados:** ENV-02, ENV-03, USU-04, PRO-02

**Envío, cobros e impagos.**  
**CU-18: Enviar recordatorios y avisos de impago (automático):**  
**Objetivo:** Marca las pautas a seguir a la hora de mantener un flujo de cierto tipo de notificaciones.

**Actores:** El propio reloj del sistema.  
**Precondiciones:** 

- Para recibir recordatorios, el cliente debe tener un pago pendiente en su cuenta.  
- Para recibir avisos de impago, el cliente debe no haber pagado alguna factura tras su fecha de vencimiento.

**Flujo Principal:**  
Se trata de un proceso automatizado, por lo que no se requerirá intervención de ningún usuario. Además, tendrá que ejecutarse, al menos, una vez al día, por lo que actuará en segundo plano. Por lo general: 

- El sistema ve un cliente y accede a sus facturas.  
- Para cada factura pendiente, el sistema compara las fechas de los pagos pendientes con la fecha actual.  
- Si han pasado tres días desde la emisión de la factura o es el día del vencimiento del pago (5 días, configurable), envía un recordatorio.  
- Si el pago ya ha vencido y han pasado 2-3 días (configurable por el SA) desde el vencimiento o desde el último aviso, envía un aviso de impago.  
- En todo caso, se envía con los datos de contacto de soporte.  
- Se registra el envío como enviado correctamente junto con la fecha y el tipo de notificación.

**Flujos alternativos:**  
No se contemplarán como flujos alternativos los “caminos” provocados por modificaciones en los plazos, ya que no modifica el flujo principal de forma sustancial. 

**1º Seguimiento pausado o en disputa:** Durante el periodo de tiempo en el que estos estados tengan efecto, las notificaciones y avisos cesan.

**2º Pago o intervención:** Si la factura ya ha sido pagada o el problema se ha solucionado de otra manera, tanto avisos como notificaciones cesan. 

**3º Festivos:** En caso de que un recordatorio o aviso caiga en un día no laboral, se pasará al siguiente día laboral según el calendario.

**Flujos de excepción:**  
No se contempla ningún flujo de excepción para este caso. Los errores de entrega y ejecución se tratan en otros casos de uso y en los requisitos.

**Postcondiciones:**

- Los recordatorios han debido ser enviados a la persona correcta sin problemas.  
- Queda constancia del envío en el sistema

**Requisitos funcionales asociados: REC-01**

**CU-19: Gestionar una disputa:**  
**Objetivo:** En relación con el caso anterior. Describe el protocolo a seguir en caso de que algún pago o factura se encuentre en disputa, permitiendo pausar los avisos de impago de forma justificada.

**Actores:** AD, UO (sus clientes)  
**Precondiciones:** 

- El usuario a gestionar la disputa debe haber accedido al sistema con el rol de administrador.  
- El cliente debe haber puesto una reclamación sobre alguno de los pagos.

**Flujo Principal:**

- El administrador accede a la ficha del cliente y selecciona la factura reclamada.  
- El sistema muestra la opción de registrar la disputa con el motivo de la reclamación y fecha de la revisión.  
- El administrador rellena los campos con la información correspondiente y confirma la acción.  
- El sistema procede a la suspensión de los recordatorios y/o avisos de impago activados referentes a la factura reclamada.  
- Una vez todo hecho, el sistema registra la operación en el historial del sistema.

**Flujos Alternativos:**  
**1º Deuda cancelada o declarada incobrable:** En caso de que se determine que la factura no va a ser cobrada, se procederá a su cancelación o declaración como incobrable. En este caso, deberá presentarse, de nuevo, un motivo y aprobación pertinentes.

**Flujos de Excepción:**  
**1º Cancelación de la suspensión:** Este proceso puede cancelarse o gestionarse como excepción en el caso de que falten bien o los motivos por los cuales se ha procedido a la revisión o no se tiene la aprobación requerida para efectuarse. En ambos casos, el flujo principal simplemente se detiene y se vuelve al estado original mostrando un mensaje acorde al error. 

**Postcondiciones:**

- La deuda debe seguir apareciendo como no cobrada.  
- El flujo de recordatorios y avisos de impagos deben quedar desactivados.  
- Las modificaciones efectuadas deben recogerse en el sistema con los datos de auditoría correspondientes.

**Requisitos funcionales asociados: REC-02**

**CU-20: Conciliar cobros automáticamente:**  
**Objetivo:** Similar al apartado CU-18. Se trata de conciliar un protocolo factible a la hora de gestionar los cobros automáticos. De nuevo, se trata de algo, como su propio nombre indica, automatizado y que no requiere de actuación externa (salvo en caso de fallo o necesidad de mantenimiento del sistema).

**Actores:** Reloj del sistema, Stripe  
**Precondiciones:** 

- Deben haber facturas pendientes de pago.  
- Se debe haber recibido una notificación y/o aviso de la plataforma/pasarela de pago correspondiente (Stripe, Revolut, etc.) marcando la entrada de un cobro. 

**Flujo Principal:**

- El sistema recibe el aviso de cobro.  
- El sistema compara los datos del importe recibido con las facturas pendientes registradas.  
- En caso de que la coincidencia sea clara, modela la conciliación con independencia de la fuente (Stripe) con importe, fecha, moneda, concepto y referencia.  
- Se actualiza el estado de la factura a pagada.  
- Notifica al responsable pertinente sobre el pago efectuado. 

**Flujos Alternativos:**  
**1º Importe no coincidente:** En caso de que al comparar los importes resulte que son distintos tanto más como de menos, el sistema, al final, no la marcará como pagada y señalará la diferencia encontrada.

**2º Ambigüedades o casos dudosos:** Si el sistema no sabe exactamente qué hacer con un cobro:

- No lo marca como pagado  
- Avisa al responsable, delegando en él las responsabilidades pertinentes (CU-21).

**Flujos de Excepción:**  
**1º Avisos duplicados:** Si el sistema recibe un aviso duplicado, ignora el segundo y continúa con el flujo normal de ejecución. 

**Postcondiciones:**

- El cobro debe aparecer reflejado en el sistema aportando a la factura el estado de “pagada”.  
- El responsable debe recibir la notificación del cobro.

**Requisitos funcionales asociados: PAG-01, PAG-02, NOT-01**

**CU-21: Conciliar un cobro a mano:**  
**Objetivo:** Trata la gestión de un cobro que no ha podido ser automatizado y, por lo tanto, debe ser revisado y/o gestionado por usuarios humanos. 

**Actores:** AD  
**Precondiciones:** 

- El sistema debe haber delegado una factura en manos del responsable a cargo (precondición derivada de CU-20).

**Flujo Principal:**

- El administrador recibe un aviso por parte del sistema de caso dudoso o sin referencia.  
- Analiza el movimiento y lo empareja con la factura correspondiente.  
- Una vez gestionado, el administrador confirma la acción realizada y la conciliación del cobro.  
- Devuelve la factura y por tanto las responsabilidades al sistema y este actualiza el estado de la factura a pagada. 

**Flujos Alternativos:**  
**1º Conciliación deshecha:** En caso de que el pago no pueda ser conciliado por algún motivo, se marcará como tal y se conservará la traza de sus actores y motivos del efecto. Además, se cancelará dicha conciliación. 

**Flujos de Excepción:**  
No se contemplan flujos de excepción para este proceso al tratarse de algo gestionado por los administradores. El sistema deberá prohibir la conciliación de cobros a facturas canceladas o que ya han recibido su cobro correspondiente. 

**Postcondiciones:**

- Debe quedar constancia y auditoría de los movimientos efectuados sobre la conciliación o no conciliación del cobro.  
- En todo caso, el administrador debe haber delegado de nuevo las responsabilidades del cobro.   
- El estado de la factura debe reflejar correctamente cualquier modificación realizada sobre esta (pagada, impagada, cancelada, etc.)

**Requisitos funcionales asociados: PAG-02, PAG-03**

**Seguimiento del día a día**  
**CU-22: Consultar la pantalla de trabajo diario:**  
**Objetivo:** Describe la funcionalidad del sistema que permite, según el rol del usuario en el sistema, un resumen de lo necesario, revisable, etc. para ese día.

**Actores:** AD, UO  
**Precondiciones:** 

- El usuario debe haber iniciado sesión correctamente en el sistema.

**Flujo Principal:**

- El usuario accede al sistema de forma normal, recibiendo la pantalla de inicio del sistema.  
- Accede a su bandeja, que será una bandeja normal en caso de ser un administrador y una bandeja propia en caso de ser un usuario operativo.  
- En todo caso, el sistema recopila los elementos pendientes: Vencimientos, pagos por conciliar, problemas de entrega, aprobaciones, revisiones anuales, tareas y notificaciones.  
- El usuario podrá seleccionar cualquier elemento de la lista para acceder directamente a los datos de este.

**Flujos Alternativos:**  
No se contemplan flujos alternativos, se trata de una funcionalidad sencilla del sistema que solo debe actuar de una manera. Sí se puede, tal vez, profundizar en la naturaleza de ambas bandejas:

- Bandeja global (AD): Una visión, como su propio nombre indica, global del estado de los clientes a su cargo.  
- Bandeja propia (UO): Visión que se centra simplemente en lo que al usuario operativo se le permite ver. 

**Flujos de Excepción:**  
Tampoco se contemplan flujos de excepción para este caso de uso. Como mucho, lo que puede suceder es que falte algún dato, cosa que no debe impedir el flujo normal y será un problema a gestionar en otras partes del sistema.

**Postcondiciones:**

- El sistema debe de haber mostrado en pantalla los datos pertinentes, recogiendo en cada caso los vencimientos, pagos por conciliar, problemas de entrega, aprobaciones, revisiones anuales, tareas y notificaciones pertinentes.  
- Cada elemento mostrado debe tener un acceso directo. 

**Requisitos funcionales asociados:** **DAS-01, NOT-01**

**CU-23: Gestionar incidencias de facturación:**  
**Objetivo:** La capacidad de un administrador de abrir una incidencia sobre una factura o cliente con su correcta documentación posterior gestión. 

**Actores:** AD, UO (sus clientes)  
**Precondiciones:** 

- El usuario que pretende abrir la incidencia debe haber iniciado sesión en el sistema con el rol de administrador.  
- El administrador, por lo tanto, debe tener acceso a los datos objetivos de la incidencia.

**Flujo Principal:**

- El administrador accede al cliente sobre el que quiere abrir la incidencia.  
- El sistema ofrecerá una pantalla para rellenar ciertos campos necesarios para efectuar la incidencia, como el responsable, su estado, prioridad y, además, la posibilidad de pausar los recordatorios para el cliente, si lo considera necesario, mientras se resuelve la incidencia.  
- También le aparecerá un apartado de comentarios internos con menciones y dos niveles de visibilidad y adjuntos con tipo y descripción.  
- El administrador registra y envía la incidencia al sistema.

**Flujos Alternativos:**  
No se contempla ningún flujo alternativo, la funcionalidad es la que es y no hay casos especiales que deben tratarse a parte.

**Flujos de Excepción:**  
**1º Error durante el rellenado de datos:** El administrador debe, al menos, dejar constancia sobre qué factura o cliente se está abriendo la incidencia y dejar un responsable. En caso de que estos campos no estuvieran rellenados, el sistema debería avisar al administrador y pedirlos expresamente.

**Postcondiciones:**

- La incidencia debe verse correctamente registrada en el sistema  
- Si el administrador ha decidido pausar los recordatorios, estos deberían dejar de aparecer durante el periodo de tiempo correspondiente.

**Requisitos funcionales asociados: NOT-02**

**CU-24: Consultar el dashboard financiero:**   
**Objetivo:** Describe la funcionalidad del sistema que permite visionar los datos financieros a un administrador con, además, acceso al detalle de cada cifra.

**Actores:** AD  
**Precondiciones:**

- El usuario debe haber iniciado sesión en el sistema con el rol de administrador.

**Flujo Principal:**

- El administrador accede al sistema con normalidad.  
- Entra en el apartado del dashboard financiero.  
- El sistema le muestra en pantalla una visualización de los datos financieros: facturado, cobrado, pendiente, vencido, tiempo medio de cobro, mayores deudas, recurrente esperado y previsión simple. Esto, en formato semanal, mensual y trimestral.  
- El administrador podrá seleccionar cualquiera de las cifras para acceder a una versión más detallada de sus datos.

**Flujos Alternativos:**  
No se contemplan flujos alternativos. Al igual que sucede con otros apartados/casos de uso como el CU-22, se tratan de funcionalidades muy sencillas y que realmente no ofrecen mucho más que una visualización. 

**Flujos de Excepción:**  
Tampoco se contemplan más flujos de excepción revisables por administradores o clientes debido a la sencillez y la propia funcionalidad que ofrece. Como mucho, puede darse un error de responsividad, lo que sería gestionable, más bien, por un equipo técnico que pueda revisar el código o incluso se pueda deber a un error de conexión, o un error en los datos mostrados que, como al final esto es algo íntegro del sistema en el que administrador solo actúa como “cliente” de la funcionalidad, sería más bien cuestión de revisar la estructura del sistema y a su acceso a según qué datos. 

**Postcondiciones:**

- El sistema debe mostrar en pantalla con éxito, precisión y fiabilidad los datos pertinentes.

**Requisitos funcionales asociados: DAS-02**

**CU-25: Generar y exportar informes:**  
**Objetivo:** Tratar la generación y exportación de informes así como el filtrado de los mismos según ciertas características. También se añaden un par de funcionalidades más como la opción de programación de informes y paquetes periódicos para la gestoría.

**Actores:** AD, UO (solo sus clientes)  
**Precondiciones:** 

- El usuario debe acceder al sistema con el rol de administrador, puesto que las funciones de generación y exportación de informes responden a las necesidades financieras de la propia empresa, por lo que el cliente queda excluido. 

**Flujo Principal:**

- El administrador entra al sistema y accede al apartado de informes.  
- El sistema le ofrece varias funcionalidades las cuales el administrador será libre de utilizar las veces que necesite con sus respectivas pantallas:   
  - Generar informes.  
  - Exportar informes a CSV y PDF respetando los permisos del usuario, registrando fecha, filtros y usuario.  
  - Filtrar informes por periodo, cliente, responsable, país, sector, estado, tipo y moneda.  
  - Programar informes.  
  - Paquete peridódico para la gestoría.

**Flujos Alternativos:**  
**1º Exportaciones masivas:** En caso de que se efectúe una exportación masiva debe auditarse.

**Flujos de Excepción:**  
**1º Error de datos:** Si falta algún dato crucial para el informe, su filtrado etc., el sistema avisará y devolverá al usuario a la pantalla correspondiente señalando las acciones a tomar.

**2º Error en la exportación:** Si durante la exportación el informe se corrompe debido a fallos en la transferencia de datos, incompatibilidades de software o interrupciones en el proceso el sistema aborta el proceso, devuelve un mensaje acorde al error y vuelve a la pantalla de exportación. 

**Postcondiciones:**

-  Al generarse un informe:  
  - Debe ser filtrable  
  - Debe aparecer correctamente en el sistema.  
- Al exportarse un informe:   
  - Debe aparecer formateado correctamente, sin caracteres “corruptos” debido a un mal formateo o similares.  
- Al filtrarse por alguna característica particular:  
  - Deben aparecer los informes correspondientes en cada caso.  
- Al programarse un informe:   
  - Debe quedar constancia de su futuro comportamiento y, finalmente, al emitirse en el sistema

**Requisitos funcionales asociados: INF-01**

**CU-26: Buscar en el sistema**  
**Objetivo:** Tratar la funcionalidad del sistema que permite hacer una búsqueda rápida y global de facturas, informes, clientes, etc. dependiendo del rol del usuario.

**Actores:** AD, UO  
**Precondiciones:** 

- El usuario debe haber accedido correctamente al sistema con su rol correspondiente.

**Flujo Principal:**

- Tras entrar en el sistema, el usuario se dirige a la barra de búsqueda.  
- Introduce un nombre comercial, razón social, NIF, contacto, número de factura o correo.  
- El sistema, además, le ofrece un filtrado de los datos y un apartado de favoritos y recientes para agilizar el trabajo.  
- Con todo esto, el sistema mostrará al usuario los datos correspondientes según lo estipulado.

**Flujos Alternativos:**  
Vuelve a suceder como en otros apartados. Se trata de una funcionalidad simple del sistema para la que no se contemplan flujos alternativos ni similares. 

**Flujos de Excepción:**  
**1º Búsqueda de datos inexistentes:** Si esto sucede, simplemente aparecerá un mensaje notificando la incidencia.

No se contemplan otros flujos de excepción. Al fin y al cabo, se trata de una funcionalidad sencilla del sistema y este debe asegurar la precisión y fiabilidad de los datos devueltos.

**Postcondiciones:**

- Tras la búsqueda y/o filtrado deberán aparecer los ítems que le corresponden, sin duplicados. 

**Requisitos funcionales asociados: BUS-01**

**Configuración y Supervisión**  
**CU-27: Configurar el sistema:**   
**Objetivo:** Describir las acciones que puede hacer el superadministrador para alterar los parámetros estructurales del funcionamiento de la aplicación en las preconfiguraciones, como los periodos de pago y notificaciones, y los efectos de estos cambios.

**Actores: SA**  
**Precondiciones:** 

- El empleado debe tener su sesión iniciada en la plataforma y ser del rol del superadministrador.

**Flujo Principal:**

- El superadministrador accede a las preconfiguraciones, acceso que su interfaz le proporciona.  
- Realiza las modificaciones que tenga intención de realizar  
- El sistema espera la validación de los cambios, a partir de una auditoría con otros administradores o incluso los altos cargos de la empresa.  
- Una vez validados el sistema aplicará esos cambios a todas las facturas y registros de ahora en adelante, sin modificar nada de lo previamente emitido.  
- El sistema guardará los cambios para la próxima vez que se vean las preconfiguraciones.

**Flujos alternativos:**  
1º Flujo de cancelación ya sea por decisión del superadministrador o por no aprobarse los cambios en la empresa

**Flujo de Excepción:**  
Realmente no se contempla ninguno ya que el sistema se encargará de no modificar nada de los ya emitido y que sea sólo lo totalmente nuevo lo que se vea afectado, además aquí no se introduce datos de ningún tipo o se le muestra la opción al resto de roles, así que realmente ninguna acción podría provocar un fallo en el proceso.

**Postcondiciones:**

- El sistema aplicará todos los cambios en su funcionamiento que suponga la modificación de las preconfiguraciones.

**Requisitos funcionales asociados: CFG-01, TAR-01, FAC-04, REC-01, NOT-01**

**CU-28: Supervisar el sistema:**  
**Objetivo:** Describe las acciones de supervisión que realiza el superadministrador relacionadas con los últimos movimientos y errores producidos en el sistema y la posibilidad de restaurar copias de seguridad.

**Actores: SA**

**Precondiciones:** 

- El usuario debe tener una sesión iniciada en la plataforma y debe tener asociado el rol de superadministrador. 

**Flujo Principal:**

- El superadministrador accede a la auditoría de las acciones clave a través de la opción que su interfaz le debe dar.  
- Ya puede consultar la aprobación de la empresa ante la solicitud de cambios en la plataforma.  
- En función de la aprobación ante los cambios podrá realizar los cambios en las preconfiguraciones o estos serán cancelados.  
- Adicionalmente visualiza la tabla de salud de la plataforma, pudiendo observar el historial de movimientos recientes, últimos correos enviados o errores generados.

**Flujo Alternativo:**  
**1º Flujo de cancelación:** En verdad se aplicaría el caso de uso anterior, pero este flujo se refiere más a una inicialización de este y se refiere a cuando la auditoría acaba en la decisión de no aplicar los cambios en las preconfiguraciones, y por ende estos cambios se cancelan y las preconfiguraciones se quedan como estaban.

**2º Restaurar copias de seguridad:**  
Este flujo se refiere a cuando el superadministrador revisa la tabla de salud y tras revisar los últimos correos, movimientos o facturas o incluso alguno de los errores producidos, decide restaurar una copia de seguridad sobre alguno de estos apartados para recuperar cierta información o arreglar cierto error, el flujo sería:

- El superadministrador accede a la tabla de salud con la opción que su interfaz le debe permitir.  
- Revisa la tabla de salud y decide que un elemento de esta requiere que se restaure una copia de seguridad donde el estado de la tabla era diferente.  
- Selecciona la copia que desea restaurar y confirma la operación (o cancela y termina el flujo).  
- El sistema recupera la copia de seguridad y la restaura para arreglar los últimos errores cometidos.

**Flujos de excepción:**  
No se prevén flujos de excepción concretos más allá de todo lo que sean fallos puntuales del sistema, ya que si el super administrador accede a la ventana de salud o resultados de la auditoría, lo único que puede pasar es que la recuperación de la copia de seguridad acierte o falle (pero eso sería un fallo técnico del sistema) o que las acciones se lleven a cabo o no respectivamente.

Postcondiciones:

- En caso de aprobarse los cambios, estos serán realizados por el superadministrador  
- En caso de tomar el flujo alternativo, la información de la plataforma se actualizará según lo restaurado en la copia de seguridad.

**Requisitos funcionales asociados: AUD-01, PRO-01, PRO-05**

**CU-29: Cerrar un periodo administrativo:**  
**Objetivo:** Describe la capacidad de un (super)administrador de finalizar un periodo administrativo (trimestre) y la posibilidad de reabrirlos, siempre que se cuente con el permiso de los superior de las empresa. Además describe la actuación del sistema cuando un periodo administrativo finaliza.

**Actores: AD y SA**

**Precondiciones:** 

- El usuario debe iniciar sesión en la plataforma con un rol de administrador o superadministrador.  
- El periodo administrativo debe seguir abierto en caso de que se quiera cerrar o cerrado en caso de que se quiera abrir.

**Flujo principal:** 

- El (super)administrador accede a las configuraciones pertinentes a la gestión del periodo administrativo actual que maneje la plataforma.  
- El (super)administrador selecciona la opción de terminar el periodo administrativo.  
- Suponiendo que se tengan los permisos adecuados, el sistema da por terminado el periodo administrativo, emite el informe correspondiente a su participación en el trimestre y guarda los historiales de movimientos acabados, notificaciones, etc. clasificados ahora como de un periodo anterior.  
- Además ahora impedirá la modificación de cualquier factura emitida durante el recién acabado período.

**Flujos alternativos:**   
**1º Flujo de cancelación.** Tanto para cerrar un periodo como para reabrirlo. y ya sea debido a la falta de permisos por parte de los superiores o que por ejemplo fuera un error del empleado y rectifique.

**2º Reabrir un periodo:**   
Se supondrá que se cuenta con el permiso de los superiores para reabrir un periodo administrativo, en caso contrario se vuelve al primer flujo alternativo.  
Este flujo detalla el caso en el que un (super)administrador reabre un periodo administrativo contando con el permiso de la empresa ya sea para modificar algo que ya no se podía y es necesario o buscar cierta información. El flujo sería: 

- El (super)administrador accede a la información del periodo administrativo acabado en las configuraciones pertinentes que el sistema si le muestra por su rol.  
- En estas, le da a la opción de reabrir el periodo y confirma la operación.  
- El sistema se encargará de volver a establecer dicho periodo como principal, recuperará de la base de datos el historial de movimientos que ahora pasaría a ser principal y el historial de facturas emitidas en este.  
- Volvería a habilitar la opción de modificar las facturas emitidas en este periodo.  
- No obstante, también realizaría una copia de seguridad del estado anterior de la aplicación para poder recuperar la información anterior en caso de que se modifique una factura o haya algún error.

**Flujos de excepción:**   
No se prevén flujos de excepción como tal, más allá de errores técnicos, al ser estas unas acciones muy limitadas por el permiso de los superiores y que se han incluido en los flujos alternativos. La única posibilidad sería:  
**1º Reabrir flujos demasiado lejanos:**   
Lo que entenderá el sistema por “lejano” dependerá de las preconfiguraciones cerca del guardado de los datos en la base de datos incluso cuando los clientes se den de baja, pero aún así al haber un tiempo límite donde los datos se guardarán, existe la posibilidad de querer recuperar algo de un periodo cuya información empiece a ser eliminada por haber sobrepasado el umbral de tiempo que se guarda en la base de datos.  
Y al querer reabrir ese proceso, eso podría dar lugar a errores, por lo que el sistema no permitirá reabrir períodos cuya fecha de inicio diste de la fecha actual más de una cantidad de tiempo modificable por el superadministrador.

**Postcondiciones:** 

- Las facturas y movimientos del periodo recién abierto pasan a estar declaradas entre las principales, las que son modificables y primero te muestra el sistema.  
- El sistema hará una copia de seguridad de los datos que se modifiquen y guardará la fecha de reapertura, y usará tanto esta como la del posterior cierre para determinar el tiempo en el que se guardan los datos.

**Requisitos funcionales asociados: PRO-04, AMB-01**

**CU-30: Gestionar los datos personales de un contacto:**   
**Objetivo:** Describir las capacidades de localización, anonimato y eliminación de los datos de los clientes por parte de los (super)administradores y la reacción del sistema ante estas acciones. Este caso no cubre el registro de nuevos clientes ni la modificación o incluso de datos adicionales, se centra más bien en el proceso de anonimizarlos cuando el cliente se da de baja, el periodo donde se deben mantener en la base de datos y la posibilidad de eliminarlos.

**Actores: AD y SA:**

**Precondiciones:** 

- El usuario debe tener una sesión iniciada en la aplicación como administrador o superadministrador.  
- El sistema debe contar con al menos un cliente que se haya registrado en la plataforma y que ya no esté usando los servicios de la empresa.

**Flujo Principal:** 

- El (super)administrador selecciona el registro del cliente en la plataforma gracias a las opciones de filtrado que esta le da.  
- Ahora el cliente tendrá acceso a todos los datos aportados por este cliente y, suponiendo que ya no este usando los servicios de la empresa, anonimizarlos para que los usuarios que tuvieran asociados ya no puedan verlos pero que se sigan guardando en la base de datos  
- El sistema se encargará de bloquear el acceso a los datos para los clientes, de no mostrar dichos datos como los de un cliente activo de la empresa pero no eliminará su información todavía.  
- Si ha pasado el tiempo suficiente sin que el cliente vuelva a registrarse, se eliminarán por completo los datos de la base de datos, pero SIN borrar los PDF emitidos durante su uso del servicio, como sus facturas, ni los datos contables necesarios para el cierre trimestral asociado a él.

**Flujos alternativos:**   
**1º Flujo de cancelación**

**2º Eliminación manual:**  
Describe el caso donde debido a causas de fuerza mayor, y con permiso de la empresa y el superadministrador, se efectúa una eliminación completa de los datos de la base de datos manualmente y sin que haya tenido que pasar el periodo de tiempo establecido. El flujo sería:

- El (super)administrador accede a la información del cliente que sigue presente en la base de datos.  
- El (super)administrador, suponiendo que tenga el permiso de los superiores, selecciona eliminar los datos del cliente asociado y confirma la operación.  
- El sistema comienza el borrado completo de los datos del cliente, salvo facturas emitidas y datos contables.

**3º Recuperación de los datos y retraso indefinido del proceso de cancelación:**  
Sucede cuando tras anonimizar los datos, y antes de que se eliminen el cliente vuelva a solicitar el servicio de la empresa, y en vez de guardar sus datos desde cero otra vez se recuperará la información anterior que tenía, favoreciendo la continuidad y el trabajo de los usuarios que lo gestionen. El flujo es:

- Cuando un cliente que cumpla las condiciones anteriores vuelva a contratar los servicios de la empresa, el (súper)administrador, lo registra pero no como un cliente totalmente nuevo, sino que le pedirá al sistema, a partir del identificador que tenía, recuperar los datos que tenía de la base de datos, con posibilidad de modificarlos antes de registrarlo (Esto se relaciona con los casos de la sección **Clientes**)  
- El sistema registrará al cliente a partir de esos datos guardados y recuperará su historial de movimientos.

**Flujos de excepción:**   
**1º Cliente activo o eliminado/erróneo:**   
Sucede cuando el sistema detecta que el cliente cuyos datos se quieren borrar manualmente es uno que sigue activo usando el servicio de la empresa, o que uno que se quiere recuperar ya se encuentra eliminado o los datos introducidos son erróneos.  
La reacción es la misma para ambos casos, cuando el sistema detecte esa anomalía en la acción debe impedir el proceso y avisar al (super)administrador según el caso (“No se puede eliminar a un cliente que sigue activo” o “No se encuentran los datos del cliente en la base de datos” respectivamente por ejemplo).

**Postcondiciones:**

- Los datos del cliente ya no le aparecen a ningún usuario al haber sido anonimizados y sus movimientos ya no aparecen en la interfaz principal.  
- En caso de que pase el tiempo establecido sin que el cliente vuelva, sus datos se eliminan de la base de datos.  
- En el caso de que vuelva, se recuperan, se vuelve a poner los datos visibles y se pospone la eliminación de forma indefinida.

**Requisitos funcionales asociados: PRI-01**

**CU-31: Atender consultas por API:**

**Objetivo:** Describir la respuesta automatizada del sistema cuando otro sistema externo de TurbineH se comunica a través de la API para consultar el estado de deuda de un cliente o para notificar que un cliente ha solicitado darse de baja. 

**Actores: Sistema externo de TurbineH** 

**Precondiciones:**

* El sistema externo de TurbineH debe estar debidamente autenticado y autorizado para realizar peticiones a la API de la plataforma (USU-04).  
* El cliente sobre el que se realiza la consulta o el aviso debe existir en la base de datos de la plataforma.

**Flujo Principal (Consulta de facturas pendientes):**

* El sistema externo de TurbineH envía una petición a la API indicando el identificador del cliente para consultar si mantiene facturas pendientes de pago.  
* El sistema valida las credenciales de la petición y localiza la ficha del cliente en la base de datos.  
* El sistema comprueba el estado de las facturas asociadas a dicho cliente.  
* El sistema devuelve una respuesta a través de la API indicando si el cliente tiene o no facturas pendientes (y el detalle correspondiente según el contrato de la interfaz).

**Flujos Alternativos:**

**1º Aviso de solicitud de baja de un cliente:** Describe el caso en el que el sistema externo utiliza la API no para consultar deuda, sino para notificar que un cliente ha pedido la baja desde otra plataforma de la empresa. 

**El flujo de acciones es el siguiente:**

* El sistema externo envía una petición a la API notificando que un cliente determinado solicita darse de baja.  
* El sistema recibe el aviso, valida la petición y localiza al usuario responsable asignado a dicho cliente.  
* En lugar de borrar o desactivar automáticamente la recurrencia del cliente, el sistema genera una tarea pendiente y una notificación para su responsable (visible en su pantalla de trabajo diario, CU-22) para que revise el caso y tramite la baja de forma controlada (CU-07).  
* El sistema responde al sistema externo confirmando la correcta recepción del aviso.

**Flujos de Excepción:**

**1º Petición no autorizada o cliente inexistente:** Si la petición a la API carece de credenciales válidas (USU-04), tiene un formato incorrecto o hace referencia a un identificador de cliente que no figura en la base de datos, el sistema rechaza la operación, devuelve el código de error correspondiente al sistema externo y registra la incidencia en el panel de salud/integración (CU-28). 

**2º Avisos duplicados:** Si el sistema externo envía por error dos veces seguidas el mismo aviso de solicitud de baja para el mismo cliente (PRO-02), el sistema detecta que ya existe una tarea de baja abierta para ese cliente e ignora el duplicado sin generar tareas repetidas.

**Postcondiciones:**

* En caso de consulta, el sistema externo recibe la información actualizada sobre las facturas pendientes del cliente sin que se modifiquen datos internos.  
* En caso de aviso de baja, la recurrencia del cliente sigue intacta por seguridad, pero queda registrada una tarea asignada a su responsable para gestionar la baja.

**Requisitos funcionales asociados: API-01, USU-04, PRO-02**

