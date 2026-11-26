# Arquitectura, despliegue y artefactos — Entrega 2

Proyecto TurbineH · Documentación diagramas de arquitectura · 2026-11-26

## 1. Alcance y vista común

Este documento reúne la documentación de artefactos de los cuatro bloques funcionales. Describe una arquitectura web en capas y un despliegue mínimo: un PC con navegador, un servidor de aplicación (`turbine-app`), PostgreSQL, un servidor de respaldo y los servicios externos de correo y WhatsApp. Los diagramas específicos de cada bloque se conservan junto a sus archivos `.puml` y exportaciones.

| Capa | Responsabilidad | Regla de dependencia |
|---|---|---|
| Frontera | Interfaces, ventanas, fichas, bandejas e historiales: presentan información y recogen la intención del usuario. | Solo hablan con controles; no acceden a entidades, filtros ni repositorios. |
| Control | Gestores, fábricas y coordinación de casos de uso. | Aplican reglas, validan, persisten y auditan; no contienen lógica visual. |
| Entidad | Datos, estados e invariantes del dominio. | No dependen de la interfaz ni de la persistencia. |
| Infraestructura | Repositorios, filtros, configuración, reloj, autenticación y servicios externos. | No toma decisiones de negocio. |

### Nodos y conexiones comunes

| Nodo | Artefactos o función |
|---|---|
| PC del usuario | Navegador e interfaz web. No almacena lógica ni datos de negocio. |
| Servidor de aplicación | `turbine-app`, clases de dominio, gestores, repositorios, filtros, `Reloj`, configuración y secretos. |
| Servidor de base de datos | PostgreSQL con el esquema `turbine`. |
| Servidor de respaldo | Réplica o copia de seguridad de PostgreSQL. |
| Servicios externos | Correo por SMTP y WhatsApp por HTTPS/API. |

| Entre | Protocolo | Propósito |
|---|---|---|
| PC del usuario ↔ servidor de aplicación | HTTPS | Operaciones de la interfaz web. |
| Servidor de aplicación ↔ PostgreSQL | SQL sobre TCP | Persistencia mediante repositorios. |
| PostgreSQL ↔ respaldo | Replicación o copia | Recuperación ante caída. |
| Servidor de aplicación ↔ correo | SMTP | Recuperación de contraseña y notificaciones. |
| Servidor de aplicación ↔ WhatsApp | HTTPS/API | Avisos y comunicaciones autorizadas. |

## 2. Bloque 1 — Acceso y usuarios (CU-01 a CU-07)

### Componentes y responsabilidades

| Componente | Responsabilidad |
|---|---|
| Interfaz web | Inicio y cierre de sesión, recuperación de contraseña, empleados, asignaciones, fichas de cliente, importación CSV y bajas. |
| `GestorAcceso`, `GestorClientes`, `GestorMovimientos` | Coordinan operaciones, permisos, reasignaciones, bajas e historial. |
| `AutenticadorValidador` | Comprueba credenciales, permisos, correo, NIF/CIF, datos fiscales y CSV. |
| `Reloj` | Gestiona inactividad, caducidades de delegaciones y fechas efectivas de baja. |
| Repositorios y filtros | Median entre controles y las tablas de usuarios, clientes, contactos, asignaciones, etiquetas e importaciones. |

**Decisiones.** El navegador no contiene reglas de negocio. `GestorAcceso` cubre sesión, empleados, asignaciones y delegaciones; `GestorClientes` cubre fichas, contactos, etiquetas, bajas e importaciones. Las entidades conservan trazabilidad mediante `AsignacionCliente`, `DelegacionTemporal`, `ImportacionCSV` y `Movimiento`.

**Trazabilidad.** CU-01 a CU-03.5 usan `GestorAcceso`; CU-04 a CU-07 usan `GestorClientes`; todos validan mediante `AutenticadorValidador` y persisten mediante los repositorios correspondientes.

## 3. Bloque 2 — Contratos y cobros (CU-08 a CU-11, CU-18 a CU-21)

### Componentes y responsabilidades

| Componente | Responsabilidad |
|---|---|
| `GestorSuscripciones` | Suscripciones, tarifas, prorrateos y revisiones anuales. |
| `GestorAprobaciones` | Solicitudes pendientes, umbrales, separación de funciones y aprobación/rechazo. |
| `GestorCobros` | Recordatorios, avisos de impago, disputas y conciliaciones. |
| `GestorMovimientos` | Auditoría append-only. |
| `GestorTrabajo` | Tareas y bandeja de trabajo diario. |
| `Reloj` y `AjustesSistema` | Procesos automáticos, calendario, umbrales, plazos y margen de conciliación. |
| Repositorios | Persisten suscripciones, cobros y conciliaciones, tareas y movimientos por agregado. |

**Decisiones.** `Reloj` ejecuta CU-18 y CU-20 en el servidor aunque no haya usuarios conectados. `RepositorioTareas` reúne `RevisionAnual`, `SolicitudAprobacion` y `DisputaDeuda`; `RepositorioCobros` cubre `MovimientoCobro` y `Conciliacion`. Correo y WhatsApp se consumen como servicios externos.

**Trazabilidad.** CU-08 y CU-09 pasan por `GestorSuscripciones`; CU-10 y CU-11 por `GestorAprobaciones`; CU-18 a CU-21 por `GestorCobros`, con `GestorMovimientos` para auditoría.

## 4. Bloque 3 — Facturación y aprobaciones (CU-10 a CU-17)

### Componentes y responsabilidades

| Componente | Responsabilidad |
|---|---|
| `GestorFacturas` | Borradores, emisión, recurrentes, rectificación, consulta, PDF y reenvío. |
| `GestorExcepciones` | Solicitud y aplicación de excepciones comerciales. |
| `GestorAprobaciones` | Decide, registra y resuelve las operaciones que requieren aprobación. |
| Repositorios de facturas y excepciones | Acceso a facturas, líneas, series y excepciones. |
| Dependencias de otros bloques | Clientes, suscripciones, periodos, notificaciones, movimientos, tareas y preconfiguraciones. |

**Decisiones.** La interfaz solo habla con gestores y los repositorios son la única vía hacia la base de datos. Las excepciones se mantienen separadas de la facturación porque afectan también a los datos del cliente. El `Reloj` activa la facturación recurrente (CU-14). No se adoptan microservicios porque exceden el alcance del proyecto.

**Trazabilidad.** CU-10 y CU-11 reutilizan aprobaciones; CU-12 a CU-17 usan `GestorFacturas`, con dependencias de clientes, periodos, notificaciones y movimientos según cada flujo.

## 5. Bloque 4 — Seguimiento y sistema

### Componentes y responsabilidades

| Componente | Responsabilidad |
|---|---|
| `Reloj` | Singleton y disparador de procesos periódicos. |
| `GestorTareas`, `GestorMovimientos` y fábricas | Trabajo pendiente, movimientos, auditoría y creación controlada de objetos. |
| `Tarea`, `Movimiento`, `Notificacion` y subclases | Estado y comportamiento del seguimiento y las comunicaciones. |
| Repositorios de tareas y movimientos | Persistencia de tareas, incidencias, movimientos y notificaciones. |
| Bandejas, fichas e historiales | Presentación de trabajo y trazabilidad para el usuario. |

**Decisiones.** La lógica y los datos permanecen en el servidor. PDF y CSV se generan al descargar, no se persisten como ficheros de dominio. La configuración y secretos están separados del código. El detalle de balanceadores, cachés y réplicas adicionales queda fuera del alcance.

## 6. Reglas de diseño transversales

1. Una frontera solo se comunica con un control.
2. Las entidades no dependen de vistas, gestores ni repositorios.
3. Los gestores coordinan entidades, repositorios, filtros y servicios técnicos sin depender de una vista concreta.
4. Los repositorios encapsulan la persistencia y los filtros encapsulan los criterios de consulta.
5. `Reloj` y `AjustesSistema` tienen una única instancia lógica en el servidor.
6. La auditoría de operaciones que modifican estado se registra mediante movimientos.
7. Correo y WhatsApp son integraciones externas; no forman parte de los artefactos desplegados por TurbineH.
