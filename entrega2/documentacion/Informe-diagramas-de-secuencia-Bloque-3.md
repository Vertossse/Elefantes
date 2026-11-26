# Diagramas UML de secuencia — Bloque 3 de TurbineH

Las secuencias representan las operaciones del bloque de facturación, informes y periodos administrativos. Las solicitudes del usuario llegan directamente a los gestores para centrar el diagrama en la lógica de negocio.

## 1. Crear y emitir una factura puntual

**Selección de escenarios.** Creación del borrador, emisión correcta y bloqueo por falta de permiso, datos o aprobación.

**Descripción de CU-12 y CU-13**

1. El usuario solicita un borrador, añade los conceptos y lo guarda.
2. Al confirmar la emisión, GestorFacturas comprueba los datos fiscales, el contacto de facturación y el periodo aplicable.
3. Si se cumplen las condiciones, el gestor asigna el número, guarda la factura emitida y registra el movimiento. Si falta aprobación, remite al CU-11. Si falla la entrega, remite al CU-17.

**Trazabilidad**

| Diagrama | Caso de uso | Participantes / responsabilidades |
| --- | --- | --- |
| `01-borrador-y-emision.puml` | CU-12,CU-13 | Usuario solicita, GestorFacturas coordina, AutenticadorValidador verifica permisos, FacturaPuntual calcula y registra la emisión, RepositorioFacturas guarda y GestorMovimientos registra el evento. |

**Consistencia.** FacturaPuntual hereda recalcularTotales() y emitir() de Factura. La numeración de AjustesSistema se resume dentro de la emisión.

## 2. Generar facturas recurrentes

**Selección de escenarios.** Generación automática, mensualidad ya emitida y suscripción que no corresponde facturar.

**Descripción de CU-14**

1. Reloj inicia la facturación mensual y GestorFacturas comprueba qué suscripciones deben facturarse.
2. El gestor busca una factura para esa suscripción y mes. Si ya está emitida, la conserva. Si queda pendiente, retoma el borrador o crea uno nuevo y continúa con la emisión del CU-13.

**Trazabilidad**

| Diagrama | Caso de uso | Participantes / responsabilidades |
| --- | --- | --- |
| `02-facturación-recurrente.puml` | CU-14 | Reloj inicia, GestorFacturas coordina, Suscripcion determina si procede facturar, RepositorioFacturas busca la mensualidad y FacturaRecurrente representa el resultado. |

**Consistencia.** Se respeta la regla del modelo de una factura por suscripción y mes. La emisión se reutiliza mediante una referencia al CU-13.

## 3. Consultar, descargar y rectificar una factura

**Selección de escenarios.** Búsqueda, descarga autorizada y rectificación con sus comprobaciones.

**Descripción de CU-15 y CU-16**

1. El usuario solicita una búsqueda y el gestor recupera las facturas de su ámbito autorizado.
2. Si solicita una descarga, el gestor verifica el acceso y genera el PDF a partir de los datos guardados.
3. Si un administrador confirma una corrección válida, el gestor crea una rectificativa vinculada. Una vez emitida, marca la original como rectificada y registra el movimiento.

**Trazabilidad**

| Diagrama | Caso de uso | Participantes / responsabilidades |
| --- | --- | --- |
| `03-consulta-y-rectificacion.puml` | CU-15, CU-16 | Usuario solicita, GestorFacturas valida, RepositorioFacturas consulta y guarda, FacturaRectificativa recoge el ajuste, Factura representa la original y GestorMovimientos registra la rectificación. |

**Consistencia.** La rectificativa conserva el vínculo con la factura original y sus datos emitidos. La descarga genera el PDF al vuelo, según el Markdown del modelo.

## 4. Resolver un envío fallido

**Selección de escenarios.** Reintento con entrega confirmada y reintento que vuelve a fallar.

**Descripción de CU-17**

1. Se registra el rebote y la incidencia queda abierta. Un usuario autorizado confirma o corrige el contacto y solicita el reenvío.
2. Se regenera el PDF con los datos de la misma factura. Si se entrega, se resuelve la incidencia. Si vuelve a fallar, sigue pendiente. Se registra el resultado.

**Trazabilidad**

| Diagrama | Caso de uso | Participantes / responsabilidades |
| --- | --- | --- |
| `04-entrega-y-reintento.puml` | CU-17 | Usuario solicita, EnvioComunicacion registra el intento, IncidenciaFacturacion refleja su resolución y GestorMovimientos aporta la auditoría. |

**Consistencia.** La coordinación de GestorTrabajo, que pertenece a otro bloque, se resume mediante referencias ref.

## 5. Generar y exportar un informe

**Selección de escenarios.** Generación con filtros, exportación a PDF o CSV y rechazo por datos insuficientes o falta de permisos.

**Descripción de CU-25**

1. El usuario indica el tipo de informe y los filtros. El gestor verifica los permisos, prepara el informe y guarda sus datos.
2. Si el usuario pide exportarlo, el gestor comprueba de nuevo el acceso, genera el archivo en el formato solicitado y registra la operación.

**Trazabilidad**

| Diagrama | Caso de uso | Participantes / responsabilidades |
| --- | --- | --- |
| `05-generacion-y-exportacion-de-informes.puml` | CU-25 | Usuario autorizado solicita, GestorInformes genera y exporta, AutenticadorValidador verifica el acceso, RepositorioInformes guarda y GestorMovimientos registra las operaciones. |

**Consistencia.** La secuencia usa generarInforme() y exportar() del modelo. La programación y el paquete para gestoría quedan fuera del escenario seleccionado.

## 6. Cerrar y reabrir un periodo administrativo

**Selección de escenarios.** Cierre autorizado, reapertura excepcional y rechazo cuando faltan requisitos o falla la copia previa.

**Descripción de CU-29**

1. El administrador solicita el cierre. Si tiene autorización y el periodo está abierto, GestorSistema lo cierra, registra el movimiento y remite al flujo que genera el informe de cierre.
2. Para reabrirlo, el gestor verifica la autorización superior, el estado y el límite de antigüedad. La reapertura se completa cuando está disponible la copia previa.

**Trazabilidad**

| Diagrama | Caso de uso | Participantes / responsabilidades |
| --- | --- | --- |
| `06-cierre-y-reapertura-de-periodo.puml` | CU-29 | Administrador solicita, GestorSistema coordina, AutenticadorValidador verifica permisos, PeriodoAdministrativo cambia de estado, AjustesSistema aporta el límite de antigüedad y GestorMovimientos registra el cambio. |

**Consistencia.** El cierre y la reapertura se realizan sobre PeriodoAdministrativo. Una vez reabierto, las correcciones de facturas emitidas se tramitan mediante rectificativas, conforme al modelo.
