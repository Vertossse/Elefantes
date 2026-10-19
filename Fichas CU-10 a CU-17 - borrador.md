# Especificación de casos de uso: CU-10 a CU-17

Facturación y aprobaciones. Borrador.

Hasta que tengamos el cheatsheet, las fichas siguen los campos que pide la guía (apartado 6.5). Los códigos entre corchetes son los requisitos funcionales del documento de requisitos.

---

## CU-10 Solicitar una excepción comercial

**Objetivo:** proponer para un cliente un descuento temporal, un primer mes gratuito o un precio pactado, para que lo apruebe quien corresponda.

**Actores:** usuario operativo (sus clientes asignados) o administrador.

**Requisitos relacionados:** [APR-01]

**Precondiciones:**
- El usuario ha iniciado sesión.
- El cliente tiene una suscripción registrada y, si el usuario es operativo, lo tiene asignado.

**Flujo principal:**
1. El usuario abre la suscripción del cliente y elige "Solicitar excepción".
2. El sistema muestra los tipos de excepción: descuento temporal, primer mes gratuito o precio pactado.
3. El usuario elige el tipo, indica el valor (porcentaje o importe), el periodo en que se aplica y el motivo.
4. El sistema muestra un resumen con el efecto que tendría en las próximas facturas.
5. El usuario confirma la solicitud.
6. El sistema la registra como "pendiente de aprobación", decide quién debe aprobarla (un administrador, o el superadministrador si supera el umbral configurado) y le envía una notificación.

**Flujos alternativos:**
- 5a. El usuario cancela antes de confirmar: no se registra nada.
- 6a. Si el solicitante es administrador y la regla exige separación de funciones, la solicitud la tiene que aprobar otro administrador.

**Flujos de excepción:**
- 3a. Falta el motivo o el valor: el sistema marca el campo y no deja continuar, sin perder lo ya escrito.

**Postcondiciones:**
- La solicitud queda pendiente de aprobación y el aprobador está avisado.
- La tarifa del cliente no cambia hasta que se apruebe.

**Relaciones:** la solicitud se resuelve en CU-11.

---

## CU-11 Aprobar o rechazar solicitudes

**Objetivo:** decidir sobre las operaciones económicas que requieren aprobación antes de aplicarse.

**Actores:** administrador; superadministrador cuando se supera el umbral configurado.

**Requisitos relacionados:** [APR-01, APR-02, FAC-06, REC-02]

**Precondiciones:**
- El aprobador ha iniciado sesión como administrador o superadministrador.
- Hay al menos una solicitud pendiente asignada a él.

**Flujo principal:**
1. El aprobador abre la bandeja de aprobaciones.
2. El sistema muestra las solicitudes pendientes con su tipo, cliente, importe, solicitante y el motivo por el que necesitan aprobación.
3. El aprobador elige una solicitud.
4. El sistema muestra el detalle y su efecto (por ejemplo, precio anterior y nuevo).
5. El aprobador la aprueba y, si quiere, añade un comentario.
6. El sistema aplica la operación, registra quién la aprobó y cuándo, y avisa al solicitante.

**Flujos alternativos:**
- 3a. El aprobador asignado está ausente: un administrador reasigna la solicitud a otra persona o a su suplente, y la reasignación queda registrada.
- 5a. El aprobador la rechaza indicando el motivo: la operación no se aplica y el solicitante ve el rechazo con el comentario.

**Flujos de excepción:**
- 5b. El aprobador es quien hizo la solicitud y la regla exige separación de funciones: el sistema no le deja aprobarla.
- 5c. El importe supera el umbral del administrador: el sistema indica que debe aprobarla el superadministrador y se la deriva.

**Postcondiciones:**
- La solicitud queda aprobada o rechazada y registrada en la auditoría.
- El solicitante conoce la decisión.

**Relaciones:** resuelve las solicitudes que vienen de CU-10, CU-13, CU-15 y CU-19.

---

## CU-12 Crear una factura puntual

**Objetivo:** preparar una factura por un servicio adicional (instalación, consultoría, formación, etc.).

**Actores:** administrador, o usuario operativo sobre sus clientes asignados.

**Requisitos relacionados:** [FAC-01, FAC-03]

**Precondiciones:**
- El usuario ha iniciado sesión.
- El cliente está registrado y, si el usuario es operativo, lo tiene asignado.

**Flujo principal:**
1. El usuario elige el cliente y pulsa "Nueva factura puntual".
2. El sistema crea un borrador con los datos del cliente y los valores por defecto (moneda, impuesto, vencimiento).
3. El usuario añade una o varias líneas: elige un concepto del catálogo o escribe uno nuevo, y ajusta la descripción, cantidad, precio, descuento e impuesto.
4. El sistema calcula los subtotales, impuestos y total.
5. El usuario guarda el borrador.

**Flujos alternativos:**
- 5a. El usuario quiere emitirla en ese momento: continúa en CU-13.
- 1a. El usuario abre un borrador que ya existía para seguir editándolo.
- 1b. El usuario elimina un borrador que nunca se emitió: el sistema pide confirmación, lo elimina y deja constancia.

**Flujos de excepción:**
- 5b. El usuario cierra la pantalla sin guardar: el sistema conserva un borrador automático para recuperarlo.

**Postcondiciones:**
- Queda una factura en estado borrador, todavía sin número definitivo.
- Cambiar el catálogo más adelante no modifica esta factura.

**Relaciones:** el borrador se emite con CU-13.

---

## CU-13 Emitir una factura

**Objetivo:** convertir un borrador en una factura definitiva y enviarla al cliente.

**Actores:** administrador, o usuario operativo sobre sus clientes asignados.

**Requisitos relacionados:** [FAC-04, FAC-05, PDF-01, ENV-01, ENV-02, APR-02]

**Precondiciones:**
- Existe una factura en borrador (creada en CU-12 o generada en CU-14).
- El cliente tiene un contacto de facturación.

**Flujo principal:**
1. El usuario abre el borrador y pulsa "Emitir".
2. El sistema comprueba que están todos los datos fiscales imprescindibles y que el importe no se aparta del patrón habitual del cliente.
3. El sistema muestra la vista previa del PDF.
4. El usuario confirma la emisión.
5. El sistema asigna el siguiente número de la serie, fija la fecha de emisión y genera y guarda el PDF con la imagen corporativa.
6. El sistema envía la factura por correo al contacto de facturación y registra el envío (destinatario, canal, fecha y resultado).
7. El sistema marca la factura como emitida y enviada.

**Flujos alternativos:**
- 2a. El importe se sale del patrón del cliente: el sistema avisa, pero el usuario puede seguir.
- 4a. La factura necesita aprobación (importe modificado a mano, vencimiento fuera de lo habitual o factura puntual muy alta): queda pendiente con el motivo explicado y se emite cuando se aprueba en CU-11.
- 6a. El contacto acepta WhatsApp: también se envía por ese canal (simulado).

**Flujos de excepción:**
- 2b. Falta un dato fiscal imprescindible: el sistema bloquea la emisión, indica qué dato falta y vuelve al borrador.
- 4b. El usuario pulsa dos veces o dos personas emiten a la vez: el sistema indica que ya se está procesando y no duplica la factura ni el número.
- 6b. El envío falla: la factura queda emitida pero sin enviar, y se puede reintentar sin generar otro número. Si el correo rebota, se pasa a CU-17.

**Postcondiciones:**
- La factura tiene un número único dentro de su serie y su PDF queda guardado.
- El envío queda registrado.
- A partir de aquí la factura no se puede editar: solo se corrige con CU-15.

**Relaciones:** CU-14 la incluye para cada factura recurrente. Si hace falta aprobación, interviene CU-11. Si el correo rebota, sigue en CU-17.

---

## CU-14 Generar las facturas recurrentes

**Objetivo:** emitir automáticamente cada mes la factura de todos los clientes con suscripción activa.

**Actores:** reloj del sistema.

**Requisitos relacionados:** [GEN-01, SUB-01]

**Precondiciones:**
- Es día 1 de mes, o el proceso de ese mes no terminó y se está reintentando.
- Hay suscripciones activas.

**Flujo principal:**
1. El reloj del sistema inicia el proceso el día 1.
2. El sistema obtiene las suscripciones que se tienen que facturar ese mes.
3. Para cada una, genera la factura con los conceptos y la tarifa vigentes, aplicando los descuentos aprobados que sigan en vigor.
4. El sistema emite cada factura (CU-13).
5. El sistema registra la ejecución: clientes procesados, facturas creadas y fallos.

**Flujos alternativos:**
- 3a. La suscripción está pausada, tiene la baja efectiva o empieza en una fecha futura: no se factura y queda anotado.
- 3b. El cliente tiene una tarifa nueva aprobada: se usa desde su fecha de efecto.
- 3c. El cliente tiene condiciones de pago pactadas (por ejemplo, a 30 días): el vencimiento de la factura respeta esas condiciones.
- 3d. Un descuento caducó: deja de aplicarse y se avisa al responsable del cliente.

**Flujos de excepción:**
- 3e. Al cliente le falta el contacto de facturación o un dato fiscal: no se emite su factura, se avisa a su responsable y el proceso sigue con los demás.
- 1a. El sistema estaba caído el día 1: al volver detecta qué facturas no se generaron y las genera sin duplicar las que ya existían.

**Postcondiciones:**
- Cada suscripción activa tiene su factura del mes emitida y enviada.
- Cada caso que no se pudo facturar le llegó a su responsable.
- La ejecución queda registrada.

**Relaciones:** incluye CU-13 para cada factura.

---

## CU-15 Rectificar una factura emitida

**Objetivo:** corregir una factura ya emitida sin modificarla, mediante una factura rectificativa vinculada.

**Actores:** administrador.

**Requisitos relacionados:** [FAC-06, FAC-02, AMB-01]

**Precondiciones:**
- El administrador ha iniciado sesión.
- La factura está emitida.

**Flujo principal:**
1. El administrador abre la factura y elige "Rectificar".
2. El sistema crea una rectificativa vinculada a la original, con sus datos como punto de partida.
3. El administrador indica qué líneas o importes se corrigen y el motivo.
4. El sistema muestra la vista previa y cómo queda el saldo del cliente.
5. El administrador confirma.
6. El sistema asigna un número de la serie de rectificativas, genera el PDF, lo envía al cliente, marca la factura original como rectificada y recalcula el saldo.

**Flujos alternativos:**
- 5a. El importe rectificado supera el umbral de aprobación: la rectificativa queda pendiente hasta que se apruebe en CU-11.

**Flujos de excepción:**
- 3a. No se indica el motivo: el sistema no deja continuar.
- 1a. Un usuario operativo intenta rectificar: el sistema no se lo permite.

**Postcondiciones:**
- Existe una rectificativa emitida y vinculada a la original.
- La factura original no cambia, pero aparece como rectificada.
- El saldo del cliente refleja la corrección y la acción queda en la auditoría.

**Relaciones:** puede necesitar aprobación en CU-11.

---

## CU-16 Consultar una factura y descargar su PDF

**Objetivo:** ver el estado de una factura y obtener el documento tal como se emitió.

**Actores:** administrador, o usuario operativo sobre sus clientes asignados.

**Requisitos relacionados:** [FAC-02, PDF-01]

**Precondiciones:**
- El usuario ha iniciado sesión.

**Flujo principal:**
1. El usuario busca o filtra facturas por cliente, estado, tipo o fecha.
2. El sistema muestra el listado con número, cliente, importe, vencimiento y estado. El usuario operativo solo ve las de sus clientes.
3. El usuario elige una factura.
4. El sistema muestra sus datos principales, su estado y su historial (envíos, cobros, rectificaciones).
5. El usuario pulsa "Descargar PDF".
6. El sistema entrega el PDF original de la emisión.

**Flujos alternativos:**
- 5a. La factura todavía es un borrador: el sistema muestra una vista previa en lugar del PDF definitivo.

**Flujos de excepción:**
- 3a. El usuario intenta abrir una factura de un cliente que no tiene asignado (por ejemplo, copiando el enlace): el sistema rechaza el acceso.

**Postcondiciones:**
- La factura no cambia.

---

## CU-17 Resolver un envío fallido

**Objetivo:** conseguir que una factura que no llegó al cliente se entregue, sin duplicarla.

**Actores:** administrador, o usuario operativo responsable del cliente.

**Requisitos relacionados:** [ENV-02, ENV-03]

**Precondiciones:**
- El envío de una factura falló o el correo rebotó.
- El sistema abrió una incidencia de entrega y avisó al responsable del cliente.

**Flujo principal:**
1. El responsable abre la incidencia desde la notificación o desde la pantalla de trabajo diario.
2. El sistema muestra la factura, el destinatario, el motivo del fallo y los intentos anteriores.
3. El responsable corrige el correo del contacto o elige otro contacto de facturación.
4. El responsable pulsa "Reenviar".
5. El sistema reenvía la misma factura, con el mismo número, y registra el envío como reintento.
6. El envío sale bien: el sistema marca la factura como notificada y cierra la incidencia.

**Flujos alternativos:**
- 3a. El fallo fue temporal y no hay nada que corregir: el responsable reenvía sin cambios.

**Flujos de excepción:**
- 6a. El reenvío vuelve a fallar: la incidencia sigue abierta y el nuevo intento queda registrado.

**Postcondiciones:**
- La factura queda notificada, o la incidencia sigue abierta con todos los intentos registrados.
- El historial distingue el envío original de los reintentos y no se crea ninguna factura nueva.
