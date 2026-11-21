# Cambios respecto a la Entrega 1

| Elemento | Cambio | Motivo | Impacto en Entrega 2 | Validación / pendiente |
|---|---|---|---|---|
| CU-31 (API) | Conectar en el diagrama al actor «Sistema externo de TurbineH». | La ficha ya lo identifica como actor; la omisión era solo del diagrama. | Arquitectura (API y actor consumidor) y secuencia de CU-31. | Acordado. |
| Reloj del sistema (CU-14, CU-18) | Dejar de representarlo como actor externo; mostrar el disparador temporal en el flujo. | Es un componente interno, no un actor externo independiente. | Arquitectura (temporizador interno) y secuencias de CU-14 y CU-18. | Acordado. |
| Stripe (CU-20) | Mantenerlo como actor externo de CU-20; conservar la separación entre consultar/conciliar el pago y procesarlo. | El sistema no debe convertirse en pasarela propia (apartado 1.2 de requisitos). | Interfaces, arquitectura y secuencia de CU-20. | Acordado. |
| CU-14 «include» CU-13 | Mantener la relación. | La ficha indica que cada factura se emite siguiendo CU-13. | Secuencias de CU-13 y CU-14. | Acordado. |
| Permisos (CU-02, CU-03, CU-04, CU-07, CU-21, CU-30) | Crear una matriz actor × operación y aplicarla de forma uniforme. | Fichas y diagrama deben atribuir las mismas autorizaciones; el Usuario operativo no equivale a un lector sin capacidad de escritura. | Autorizaciones en arquitectura y secuencias. | Acordado. |
| CU-27, CU-28, CU-26 | Concretar objeto, criterio y resultados observables. | Eran objetivos demasiado amplios o genéricos. | Clases y operaciones del diagrama de clases. | Acordado. |
| Diagramas 1 a 4 | Reducir cruces y trazos largos; separar funciones por actor o ámbito. | Legibilidad. | Ninguno directo. | Acordado. |
| CU-04 «include» CU-08 | Revisar la relación según si registrar un cliente obliga siempre a gestionar su suscripción. | Confirmar la semántica del include. | Clases (Cliente–Suscripción) y ER. | Acordado. |
| CU-07 «extend» CU-08 y RNG-22 | Distinguir baja comercial, pausa, fin de recurrencia y borrado/anonimización; documentar fecha efectiva, cobros pendientes y excepciones. | La baja detiene las mensualidades futuras pero mantiene las facturas pendientes. | Estados de baja y suscripción en ER y clases; secuencias de CU-07 y CU-31. | Acordado. |
| CU-13 «extend» CU-12 | Revisar si «emitir borrador» es extensión o continuación/separación de objetivos. | La ficha de CU-13 exige un borrador previo. | Estado «borrador» de la factura; secuencia de CU-13. | Acordado. |
| CU-11 «extend» (emisión y rectificación) | Indicar condición, umbral, punto de extensión y permisos. | La aprobación condicional debe quedar explícita. | Estado «aprobación» en ER y clases; permisos. | Acordado. |
| CU-17 «extend» emisión | Tratar el envío fallido como flujo alternativo o caso independiente. | El fallo solo sería extensión si forma parte de esa instancia. | Secuencia de envío fallido. | Acordado. |
| CU-19 «extend» CU-18 | Tratar la pausa por disputa como condición de envío o regla de negocio. | Evitar recordatorios inapropiados durante una disputa. | Secuencia de CU-18; estado de disputa. | Acordado. |
| CU-21 «extend» CU-20 | Representarlo como flujo alternativo con intervención humana. | La ficha lo justifica sin implicar siempre una extensión UML. | Secuencias de CU-20 y CU-21; estado de conciliación. | Acordado. |
| CU-08 (prorrateo) | Indicar qué regla fija el prorrateo y cuál la excepción de empezar a facturar el mes siguiente; verificar el primer importe. | Aclarar la trazabilidad. | Cálculo del importe inicial; clases y ER. | Acordado. |
| RNC-10, CU-09, CU-11 | Asegurar la correspondencia exacta y conservar tarifas e importes de las facturas ya emitidas. | La revisión anual tiene efecto futuro con validación humana. | Histórico de tarifas e importes en ER. | Acordado. |
| CU-29, CU-30 | Justificar CU-29 como función adicional y precisar conservación, acceso y operación permitida en CU-30. | Alcance y privacidad. | Datos personales en ER; permisos. | Acordado. |
| RNG-10…RNG-23, RNC-13 | Explicar la leyenda RI/RA y declarar qué funciones son inferencias o ampliaciones; priorizar lo imprescindible. | No presentarlas como solicitudes literales de Turbine. | Alcance del modelo de datos y de clases. | Acordado. |
| Estados del dominio | Revisar estados de factura, baja, suscripción, disputa, conciliación y aprobación; no mezclar procesos con datos de estado. | Orientación para el modelo de dominio. | ER y diagrama de clases. | Acordado. |
