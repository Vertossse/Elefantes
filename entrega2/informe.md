# Entrega 2 · Diseño del sistema

**Asignatura:** Análisis y Diseño de Aplicaciones · Grupo A · 2026–27  
**Equipo:** Elefantitos  
**Integrantes:**
- Tania Muñoz Iglesia (Scrum Master)
- Fernando de la Cruz Martos (Desarrollador)
- Lucas Ariel Fanelli Sánchez (Desarrollador)
- Lucas Grande Ramírez (Desarrollador)
- Julio Herrera Sánchez (Desarrollador)
- Alexandru Olsanschi (Desarrollador)
- Antonio Rengel Fernández (Desarrollador)
- Rubén Sánchez Rosa (Desarrollador)  
**Fecha:** 27/11/2026  
**Repositorio:** https://github.com/Vertossse/Elefantes/tree/main

PARA VER LAS IMÁGENES DE LOS DIAGRAMAS IR A LA CARPETA **EVIDENCIAS**, AQUÍ NO SE MUESTRAN EXPLÍCITAMENTE.

---

## 1. Introducción y alcance

El presente informe expone el diseño técnico y arquitectónico detallado del sistema de gestión de clientes, suscripciones, facturación y cobros desarrollado para la empresa TurbineH. El trabajo de diseño parte formalmente del análisis de requisitos y casos de uso establecido en la primera entrega del proyecto y cubre de manera integral la especificación de los 31 casos de uso identificados (denominados del CU-01 al CU-31).

El propósito central del sistema es automatizar de forma sólida, auditable y escalable todo el ciclo de vida operativo y financiero de TurbineH. La cobertura funcional del diseño se articula en torno a cuatro grandes bloques operativos interrelacionados:

1. **Gestión de usuarios y clientes:** Administración de los perfiles de empleados (Usuario Operativo, Administrador y Superadministrador), asignación e historial de clientes, gestión de contactos, canales de comunicación consentidos, etiquetas organizativas y mecanismos estrictos de baja lógica, anonimización y borrado en cumplimiento de la normativa de protección de datos (GDPR).
2. **Suscripciones, tarifas y contrataciones:** Gestión de planes y tramos tarifarios por rango, emisión de contratos mensuales con conservación de precio acordado, aplicación de excepciones comerciales (descuentos temporales, meses gratuitos, precios pactados), prorrateo inicial y automatización de revisiones anuales de tarifas.
3. **Ciclo de facturación e informes:** Emisión automatizada de facturas recurrentes y puntuales, gestión del ciclo de vida de facturas (borrador, emitida, pagada, anulada, rectificada e incobrable), emisión de facturas rectificativas por rectificación parcial o total, gestión de periodos administrativos trimestrales con cierre y reapertura controlada, y generación de informes parametrizados exportables a formatos PDF y CSV.
4. **Cobros, seguimiento y auditoría:** Conciliación de cobros recibidos a través de pasarelas de pago (Stripe) o transferencias bancarias, seguimiento de impagos con recordatorios automáticos gobernados por el reloj del sistema, gestión de disputas de deuda e incidencias de facturación, y mantenimiento de una bitácora de auditoría inmutable basada en eventos que registra cada operación efectuada en la plataforma.

---

## 2. Actualizaciones de la Entrega 1

A partir de las observaciones recibidas en la evaluación de la Entrega 1 y de las sesiones de revisión interna del equipo Elefantitos, se introdujeron modificaciones estructurales en las especificaciones de requisitos y casos de uso. Estas actualizaciones garantizan la consistencia semántica entre los modelos de dominio, el comportamiento dinámico y el esquema físico de persistencia.

El siguiente desglose detalla los elementos modificados, el motivo técnico de cada ajuste, su impacto directo en los modelos de la Entrega 2 y el estado de validación acordado:

| Elemento | Cambio realizado | Motivo técnico | Impacto en la Entrega 2 | Estado de validación |
| :--- | :--- | :--- | :--- | :--- |
| **CU-31 (API)** | Conectar en el diagrama al actor "Sistema externo de TurbineH". | La ficha descriptiva identificaba al sistema externo como actor, pero se había omitido su representación gráfica en el diagrama. | Integración en la arquitectura de componentes (API y actor consumidor) y en la secuencia de CU-31. | Acordado |
| **Reloj del sistema (CU-14, CU-18)** | Reclasificarlo como componente/disparador temporal interno en lugar de actor externo. | El reloj es un mecanismo de temporización interno del sistema y no un actor externo independiente. | Rediseño en el diagrama de arquitectura y en los diagramas de secuencia de CU-14 y CU-18. | Acordado |
| **Stripe (CU-20)** | Mantener a Stripe como actor externo para procesar cobros, separando la pasarela de la conciliación interna. | Evitar que el sistema asuma funciones de pasarela propia, cumpliendo el apartado 1.2 de los requisitos. | Definición de interfaces de comunicación externa y secuencias de conciliación en CU-20. | Acordado |
| **CU-14 «include» CU-13** | Conservar la relación de inclusión directa. | Cada factura recurrente se emite formalmente siguiendo las reglas de emisión general registradas en CU-13. | Reflejo continuo en la interacción de secuencias entre CU-13 y CU-14. | Acordado |
| **Matriz de Permisos (CU-02, CU-03, CU-04, CU-07, CU-21, CU-30)** | Definir una matriz explícita Actor × Operación y aplicarla uniformemente. | Garantizar autorizaciones homogéneas; el Usuario Operativo posee facultades de escritura específicas según su rol. | Especificación rigurosa de controles de acceso en gestores de control y secuencias. | Acordado |
| **Operaciones de CU-26, CU-27, CU-28** | Concretar objeto, criterio de búsqueda y resultados observables. | Los casos de uso presentaban definiciones genéricas u objetivos demasiado amplios. | Definición precisa de métodos en el Diagrama UML de Clases (`Filtro<T>`, `AjustesSistema`). | Acordado |
| **Trazado de diagramas 1 a 4** | Reducir cruces de líneas y trazos largos; agrupar funciones por ámbito funcional. | Mejorar significativamente la legibilidad y mantenibilidad de la documentación visual. | Reorganización gráfica de los esquemas UML de casos de uso y clases. | Acordado |
| **CU-04 «include» CU-08** | Ajustar la semántica de inclusión según la obligatoriedad de la suscripción al dar de alta a un cliente. | Distinguir el registro de datos fiscales del cliente frente a la activación inmediata de su contrato. | Separación de entidades y asociaciones en la clase Cliente, Suscripcion y el modelo ER. | Acordado |
| **CU-07 «extend» CU-08 y RNG-22** | Diferenciar baja comercial, pausa, fin de recurrencia y anonimización/borrado. | La baja comercial detiene las cuotas futuras pero mantiene vigentes las facturas emitidas pendientes de cobro. | Definición de estados en las entidades Suscripcion y Cliente; secuencias de CU-07 y CU-31. | Acordado |
| **CU-13 «extend» CU-12** | Modelar la emisión de borrador como paso previo obligatorio a la emisión definitiva de factura. | Cumplimiento del flujo de negocio que exige validación del borrador antes de fijar serie y número. | Estado «borrador» en la entidad Factura y secuencia correspondiente en CU-13. | Acordado |
| **CU-11 «extend»** | Explicitar condición, umbral económico, punto de extensión y permisos requeridos. | La aprobación condicional requiere criterios cuantitativos y roles autorizados explícitos. | Incorporación del estado de aprobación en clases y modelo ER, filtrado por umbral. | Acordado |
| **CU-17 «extend» emisión** | Tratar el envío fallido de facturas como flujo alternativo dentro del caso de uso. | El fallo en la entrega por canal externo no constituye un caso de uso independiente. | Integración del manejo de excepciones en el diagrama de secuencia de envío. | Acordado |
| **CU-19 «extend» CU-18** | Tratar la pausa de recordatorios por disputa como regla de negocio y condición de envío. | Prevenir el envío automático de avisos de morosidad mientras una factura esté en disputa activa. | Control de flujo en la secuencia de CU-18 y adición de la entidad DisputaDeuda. | Acordado |
| **CU-21 «extend» CU-20** | Representar la conciliación manual como flujo alternativo con intervención humana. | Justificar la intervención operativa cuando la conciliación automática no empareja importes. | Modelado de flujos en secuencias de CU-20 y CU-21 y estados de Conciliacion. | Acordado |
| **CU-08 (Cálculo de Prorrateo)** | Especificar la regla de cálculo de primer mes y la excepción de inicio en el mes siguiente. | Garantizar la trazabilidad del primer cobro según la fecha de alta. | Implementación del método de cálculo en Suscripcion y reflejo en el modelo físico. | Acordado |
| **RNC-10, CU-09, CU-11** | Asegurar la correspondencia exacta de tarifas e inmutabilidad de facturas pasadas. | La revisión de precios afecta a cuotas futuras sin alterar el histórico ya facturado. | Preservación histórica de importes en las tablas de facturas y conceptos del modelo ER. | Acordado |
| **CU-29 y CU-30** | Justificar el cierre de periodos (CU-29) y precisar retención y anonimización de datos (CU-30). | Proporcionar soporte contable normativo y cumplir los derechos de privacidad (GDPR). | Diseño de las tablas `periodo_administrativo`, `contacto_canal` y procesos de anonimización. | Acordado |
| **Leyenda RI/RA (RNG-10 a RNG-23)** | Declarar explícitamente qué funciones son reglas inferidas o ampliaciones sobre el enunciado. | Clarificar la trazabilidad entre el pliego inicial de TurbineH y las soluciones adoptadas. | Delimitación del alcance en el modelo de dominio y arquitectura. | Acordado |
| **Estados del Dominio** | Normalizar los estados de factura, baja, suscripción, disputa, conciliación y aprobación. | Separar el estado de los datos respecto al estado de los procesos operativos. | Implementación de enumerados y campos con restricciones CHECK en la base de datos. | Acordado |

---

## 3. Modelo físico de datos (Entidad–Relación)

El modelo físico de datos se diseñó para el gestor de bases de datos PostgreSQL a partir de la transformación sistemática de las clases de dominio (`<<entity>>`) consensuadas por el equipo Elefantitos. El esquema consta de **43 tablas** estructuradas para dar soporte completo a la persistencia transaccional, trazabilidad contable y auditoría del sistema.

### Reglas de transformación de Clases UML a Modelo ER Físico

La conversión del modelo analítico de clases al modelo físico de datos se llevó a cabo aplicando siete reglas de correspondencia:

1. **Transformación de entidades:** Únicamente las clases marcadas con el estereotipo `<<entity>>` generan tablas en la base de datos. Las clases de control (`Gestores`), los repositorios de acceso a datos y las clases auxiliares de presentación no generan tablas.
2. **Relaciones 1 a N:** Se transforman en una clave foránea (`FK`) colocada en la tabla del lado "muchos".
3. **Relaciones N a M y Colecciones:** Las asociaciones de muchos a muchos y los atributos multivaluados en forma de mapa generan tablas intermedias de unión. Ejemplos de ello son `cliente_contacto`, `cliente_etiqueta`, `factura_concepto` y `contacto_canal` (para canales de comunicación consentidos).
4. **Composición:** Se traduce en una clave foránea obligatoria (`NOT NULL`) y, en relaciones uno a uno de composición, en una restricción de unicidad (`UNIQUE`). Por ejemplo, `suscripcion.cliente_id`.
5. **Estrategias de Herencia:**
   - *Tabla raíz y tablas hijas (Table-per-Subclass):* Aplicada en jerarquías complejas con atributos específicos por subclase. La tabla base contiene los atributos comunes y cada subclase genera una tabla propia cuya clave primaria (`PK`) es simultáneamente clave foránea (`FK`) referente a la tabla base. Se utiliza en `tarea` (con `delegacion_temporal`, `revision_anual_tarifa`, `solicitud_aprobacion`, `disputa_deuda`, `incidencia_facturacion`) y en `movimiento` (con sus 14 tablas hijas).
   - *Tabla única con discriminador (Single-Table Inheritance):* Aplicada cuando las subclases comparten la inmensa mayoría de atributos. Se utiliza en `usuario` (con columna `rol` para Usuario Operativo, Administrador y Superadministrador) y en `notificacion` (con columna `tipo` y campos específicos opcionales).
6. **Tipos de datos y claves:** Las claves primarias utilizan el tipo `BIGINT` autonumérico. Los importes monetarios se representan mediante `NUMERIC` para evitar errores de redondeo en coma flotante. Las fechas sin hora emplean `DATE` y las marcas temporales precisas utilizan `TIMESTAMPTZ`. Los estados se almacenan mediante `VARCHAR` validados por restricciones `CHECK`.
7. **Documentos dinámicos:** Los archivos derivados (PDF de facturas, extractos CSV) no se almacenan como objetos binarios (`BLOB`) en la base de datos; se generan al vuelo bajo demanda a partir de los datos transaccionales almacenados.

### Estructura de las Tablas por Ámbitos Funcionales

El esquema relacional agrupa sus 43 tablas en siete áreas funcionales:

#### 1. Usuarios y Clientes
- `usuario`: Empleados de TurbineH. Incluye los campos de autenticación, rol (`UO`, `AD`, `SAD`), umbral de aprobación y permisos de alta.
- `cliente`: Empresas contratantes. Contiene el NIF/CIF (único), razón social, domicilio fiscal y estado (`ACTIVO`, `PAUSADO`, `INACTIVO`). Las filas nunca se borran físicamente.
- `contacto`: Personas físicas de contacto asociadas a un cliente, con idioma y estado.
- `contacto_canal`: Canales consentidos (correo electrónico, WhatsApp) vinculados a cada contacto.
- `cliente_contacto`: Tabla intermedia que resuelve la agregación N a M entre clientes y contactos.
- `etiqueta` y `cliente_etiqueta`: Clasificación taxonómica libre de clientes.
- `asignacion_cliente`: Historial de gestores asignados a cada cliente a lo largo del tiempo.

#### 2. Suscripciones y Tarifas
- `tarifa_rango`: Tramos tarifarios oficiales de la plataforma con su importe mensual base.
- `suscripcion`: Contrato comercial de un cliente. Registra el precio acordado inmutable ante cambios de tarifa general, el periodo de facturación y la referencia a la excepción comercial activa.
- `excepcion_comercial`: Catálogo de condiciones especiales (descuento temporal, primer mes gratis, tarifa fija pactada, exención de prorrateo).

#### 3. Facturación
- `factura`: Factura emitida o en borrador (puntual, recurrente o rectificativa). Contiene la serie, número secuencial, fecha de emisión, estado (`BORRADOR`, `EMITIDA`, `PAGADA`, `ANULADA`, `RECTIFICADA`, `INCOBRABLE`) y periodo administrativo.
- `concepto`: Catálogo general de conceptos facturables.
- `factura_concepto`: Tabla intermedia de líneas de detalle. Guarda una copia denormalizada de la descripción, precio unitario e impuesto vigentes en el momento de la emisión para preservar el histórico inalterable (RNG-14).
- `factura_rectificativa`: Extensión de la factura original que registra el motivo, tipo de rectificación y el importe rectificado.
- `periodo_administrativo`: Registro de los trimestres contables y su estado de cierre.

#### 4. Cobros y Conciliación
- `movimiento_cobro`: Transacciones de pago registradas desde pasarelas externas (Stripe) o transferencias bancarias.
- `conciliacion_cobro`: Emparejamiento entre un cobro y una factura, con estado (`CONCILIADO`, `REVERTIDO`) y motivo explícito en caso de reversión.

#### 5. Tareas y Gestión de Incidencias
- `tarea`: Tabla base para la gestión de tareas del sistema e incidencias humanas.
- `delegacion_temporal`, `revision_anual_tarifa`, `solicitud_aprobacion`, `disputa_deuda`, `incidencia_facturacion`: Tablas especializadas derivadas de `tarea`.
- `comentario_incidencia`: Hilo de seguimiento de comentarios asociados a una incidencia de facturación.

#### 6. Notificaciones, Movimientos y Auditoría
- `notificacion`: Registro unificado de notificaciones por correo, WhatsApp o bandeja interna.
- `envio_comunicacion`: Trazabilidad de intentos de envío externo y gestión de rebotes.
- `movimiento`: Tabla base de auditoría append-only (solo lectura/inserción).
- 14 tablas hijas de movimiento: `registro_auditoria`, `importacion_csv`, `alta_baja_cliente`, `alta_baja_usuario`, `asignacion_desasignacion_cliente`, `asignacion_desasignacion_contacto`, `generacion_factura`, `rectificacion_factura`, `generacion_informe`, `aplicacion_excepcion`, `registro_cobro`, `registro_tarea`, `cierre_periodo`.

#### 7. Configuración y Sistema
- `informe`: Registro de informes solicitados y generados por los usuarios.
- `ajustes_sistema`: Tabla de configuración global clave-valor (series de facturación, calendario de festivos, filtros guardados).

---

## 4. Diagrama UML de clases

El Diagrama UML de Clases constituye el modelo de referencia del sistema. Se estructura formalmente en tres capas arquitectónicas bien definidas:

1. **Capa de Dominio (`<<entity>>`):** Modela la información persistente y las reglas del negocio. Incluye las clases principales `Cliente`, `Contacto`, `Suscripcion`, `Factura`, `Movimiento`, `Tarea`, `Notificacion`, `Informe` y `AjustesSistema`.
2. **Capa de Control (`<<control>>`):** Formada por **12 Gestores** especializados encargados de coordinar la lógica de negocio, validar autorizaciones y orquestar las transacciones:
   - `GestorAcceso`: Autenticación, control de sesiones y recuperación de credenciales.
   - `GestorEmpleados`: Alta, baja y modificación de perfiles de empleados.
   - `GestorClientes`: Gestión integral de fichas de clientes, contactos y asignaciones.
   - `GestorSuscripciones`: Alta, modificación, prorrateo, pausas y bajas de suscripciones.
   - `GestorAprobaciones`: Evaluación y flujo de autorización de solicitudes por superadministradores.
   - `GestorFacturas`: Emisión de borradores, facturas ordinarias, recurrentes y rectificativas.
   - `GestorCobros`: Registro de cobros y proceso de conciliación con facturas.
   - `GestorTareas`: Asignación, delegación y resolución de tareas e incidencias.
   - `GestorNotificaciones`: Composición y despacho de avisos por correo o WhatsApp.
   - `GestorInformes`: Construcción, filtrado y exportación masiva de informes financieros.
   - `GestorSistema`: Cierre de periodos contables, gestión de ajustes globales y auditoría.
   - `GestorMovimientos`: Registro inalterable de eventos y bitácora de operaciones.
3. **Capa de Acceso a Datos y Apoyo (`<<boundary>>`, `<<singleton>>`, `<<static>>`):** Contiene los repositorios de persistencia (`RepositorioClientes`, `RepositorioFacturas`, etc.), la interfaz genérica `Filtro<T>`, el temporizador central `Reloj` (`<<singleton>>`), el componente `AutenticadorValidador` (`<<static>>`) y el servicio `ExportadorArchivos` (`<<singleton>>`).

### Organización en Paquetes y Patrones de Diseño

Las clases se agrupan operacionalmente en siete paquetes funcionales:

- **ClasesEmpleados:** `Usuario` (abstracta), `Usuario_Operativo`, `Administrador`, `SuperAdministrador`.
- **ClasesClientes:** `Cliente`, `Contacto`, `Etiqueta`, `Canal`, `EstadoContacto`.
- **ClasesSuscripcion:** `Suscripcion`, `ExcepcionComercial`, `TarifaRango`.
- **ClasesFacturas:** `Factura` (abstracta), `Factura_Puntual`, `Factura_Recurrente`, `Factura_Rectificada`, `Concepto`.
- **ClasesPaqueteConciliacion:** `Conciliacion`, `MovimientoCobro`.
- **ClasesTarea:** `Tarea` (abstracta), `DelegacionTemporal`, `RevisionAnual`, `DisputaDeuda`, `IncidenciaFacturacion`, `SolicitudAprobacion`.
- **ClasesMovimiento:** `Movimiento` (abstracta) con sus 14 subclases de registro y el enumerado `TipoMovimiento`.

En el diseño se aplicaron patrones estructurales y de comportamiento reconocidos:
- **Singleton:** Garantiza una única instancia para el `Reloj` del sistema, el catálogo de `AjustesSistema` y el `ExportadorArchivos`.
- **Strategy / Filter Pattern:** Implementado mediante la interfaz parametrizada `Filtro<T>` y sus realizaciones (`FiltroClientes`, `FiltroFacturas`, `FiltroInformes`), lo que permite aplicar criterios de búsqueda dinámicos sobre cualquier entidad.
- **State Pattern:** Controla las transiciones explícitas de estado en las entidades `Factura`, `Suscripcion`, `PeriodoAdministrativo` y `Tarea`.

---

## 5. Diagramas UML de secuencia

Los diagramas de secuencia especifican el comportamiento dinámico del sistema y la interacción entre actores, componentes de presentación, gestores de control, entidades de dominio y repositorios de persistencia a lo largo de los 31 casos de uso.

### Selección de Escenarios por Bloques Funcionales

Para dar una cobertura metodológica rigurosa sin redundancia gráfica, se modelaron los escenarios más representativos utilizando fragmentos combinados de UML (`alt`, `opt`, `loop` y `ref`):

| Bloque funcional | Casos de uso cubiertos | Escenarios representados y dinámica del flujo |
| :--- | :--- | :--- |
| **Acceso y Clientes** | CU-01 a CU-07, CU-30 | Autenticación, alta de clientes (CU-04), asignación y delegación temporal de responsables (CU-03), y anonimización de datos personales por GDPR (CU-30). |
| **Contratos y Aprobaciones** | CU-08 a CU-11 | Alta de suscripción con cálculo de prorrateo (SEC-08 / CU-08), solicitudes de excepción comercial (CU-10) y revisión anual de tarifas (CU-09). |
| **Facturación** | CU-12 a CU-17 | Emisión de borradores y facturas definitivas (CU-12, CU-13), generación recurrente automatizada (CU-14), rectificación de facturas (CU-16) y tratamiento de errores de envío (CU-17). |
| **Seguimiento y Cobros** | CU-18 a CU-21 | Emisión automática de recordatorios de impago regulada por el reloj (SEC-18 / CU-18), congelación por disputa de deuda (CU-19) y conciliación automática/manual de cobros (CU-20, CU-21). |
| **Operaciones Transversales** | CU-23 a CU-31 | Apertura de incidencias (CU-23), generación de dashboard directivo (CU-24), exportación de informes (CU-25), consultas API externas (CU-31) y cierre de periodos (CU-29). |

---

## 6. Arquitectura y componentes

La arquitectura del sistema sigue una estructura por capas desacopladas que independiza la presentación visual, la lógica de negocio orientada a servicios y la capa de persistencia relacional.

### Estructura de Capas y Componentes

1. **Capa de Presentación:** Interfaz de usuario web utilizada por los tres perfiles de empleados (`Usuario Operativo`, `Administrador` y `Superadministrador`). Captura los eventos del usuario, valida formatos de entrada y canaliza las peticiones hacia la capa de control.
2. **Capa de Lógica de Negocio (Gestores de Control):** Representa el núcleo ejecutor de la plataforma. Coordina las operaciones complejas entre entidades y servicios. Por ejemplo, en el módulo de facturación y aprobaciones, destacan el `Gestor de Facturas`, el `Gestor de Excepciones` y el `Gestor de Aprobaciones`.
3. **Capa de Persistencia (Repositorios y Base de Datos):** Abstrae las consultas SQL mediante repositorios especializados (`Repositorio de Facturas`, `Repositorio de Excepciones`, etc.) conectados directamente a la base de datos PostgreSQL.
4. **Componentes Transversales de Apoyo:**
   - **Reloj del Sistema:** Temporizador interno de alta precisión que dispara eventos periódicos (generación de facturas recurrentes el día 1 de cada mes, comprobación de vencimientos de delegaciones y despacho de avisos de impago).
   - **Sistema Externo de TurbineH / API:** Componente de integración que permite a sistemas externos consultar el estado de morosidad mediante CU-31.
   - **Pasarela Externa de Pagos (Stripe):** Servicio externo de procesamiento de cobros con tarjetas y transferencias.

---

## 7. Informe Scrum

El desarrollo del diseño del sistema para la Entrega 2 se ejecutó bajo la metodología ágil Scrum durante un periodo de cuatro semanas (del 28 de octubre al 27 de noviembre de 2026), dividido en dos Sprints de trabajo.

### Identificación del Equipo y Roles

| Nombre y apellidos | Rol asignado | Principales áreas de responsabilidad |
| :--- | :--- | :--- |
| **Tania Muñoz Iglesia** | Scrum Master | Facilitación de ceremonias, informe Scrum, actas y diagramación de clases/secuencias. |
| **Fernando de la Cruz Martos** | Desarrollador | Diseño del diagrama de clases, secuencias y revisión de requisitos. |
| **Lucas Ariel Fanelli Sánchez** | Desarrollador | Responsable principal del diseño del Modelo Físico ER y diccionario de datos. |
| **Lucas Grande Ramírez** | Desarrollador | Elaboración de diagramas de arquitectura, secuencias y patrones. |
| **Julio Herrera Sánchez** | Desarrollador | Aplicación de patrones de diseño, arquitectura y secuencias. |
| **Alexandru Olsanschi** | Desarrollador | Elaboración de diagramas de clases, secuencias y documentación. |
| **Antonio Rengel Fernández** | Desarrollador | Desarrollo de la arquitectura de componentes y patrones. |
| **Rubén Sánchez Rosa** | Desarrollador | Coordinación de clases, secuencias y verificación cruzada entre modelos. |

### Planificación y Ejecución de Sprints

#### Sprint 1 (28/10/2026 – 10/11/2026)
- **Objetivo del Sprint:** Elaboración del borrador del Diagrama de Clases UML y diseño preliminar del Modelo Entidad-Relación (ER).
- **Reuniones celebradas:** Presencial en laboratorio (28/10/2026, 1:45h), presencial y virtual (04/11/2026 y 08/11/2026, 4:15h total).
- **Impedimentos detectados:** Dificultad para abstraer inicialmente las clases a partir de los casos de uso sin saturar el diagrama con gestores de UI; incertidumbre sobre cómo reflejar los patrones de diseño en PlantUML.
- **Soluciones aplicadas:** Separación del modelado en dos fases (primer diagrama puramente de entidades de dominio y segundo esquema para gestores y patrones); decisión de no saturar PlantUML con elementos visuales de interfaz.

#### Sprint 2 (11/11/2026 – 27/11/2026)
- **Objetivo del Sprint:** Finalización del esquema físico ER, elaboración de diagramas de secuencia por bloques, diagrama de arquitectura e integración global de la documentación.
- **Reuniones celebradas:** Presencial en ETSII (18/11/2026, 1:00h) y presencial en laboratorio (24/11/2026, 1:45h).
- **Verificación cruzada realizada:**
  - *Diagrama de Clases → Diagrama ER:* Conforme (verificado el 11/11/2026).
  - *Diagrama de Clases → Diagramas de Secuencia:* Conforme (verificado el 18/11/2026).
  - *Casos de Uso → Secuencias / Clases:* Conforme (verificado el 08/11/2026).
- **Tablero de gestión y repositorio auditable:** Alojado en GitHub: `https://github.com/Vertossse/Elefantes/tree/main`.

---

## 8. Conclusiones y referencias

El trabajo realizado en esta Entrega 2 consolida un diseño técnico coherente, robusto y escalable para la plataforma de facturación de TurbineH. Se ha logrado una trazabilidad bidireccional perfecta entre los requisitos iniciales, las modificaciones aprobadas sobre la Entrega 1, la arquitectura lógica en tres capas y el esquema relacional físico en PostgreSQL.

### Síntesis de las Decisiones de Diseño Más Relevantes

1. **Arquitectura por capas desacopladas:** La incorporación de 12 gestores de control especializados independiza la interfaz de usuario de la capa de persistencia, garantizando la mantenibilidad y facilitando la ejecución de pruebas unitarias y de integración.
2. **Modelo de persistencia optimizado:** El esquema relacional de 43 tablas equilibra la normalización contable con el rendimiento mediante el uso combinado de herencia por tabla única (en `usuario` y `notificacion`) y herencia por tabla especializada (en `tarea` y `movimiento`).
3. **Inmutabilidad de datos financieros y auditoría:** La preservación histórica del valor de los conceptos al emitir cada factura y el diseño *append-only* de la bitácora de auditoría (`movimiento`) garantizan la integridad legal ante inspecciones contables.
4. **Cumplimiento estricto de GDPR:** Se diseñó un ciclo de vida de datos personales que prioriza la anonimización activa (`ANONIMIZADORGPD`) y el borrado diferido sin romper la integridad referencial de facturas y cobros pasados.
