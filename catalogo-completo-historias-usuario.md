# Catálogo Completo de Historias de Usuario — Sistema de Control de Asistencia

> Generado por el agente analista senior de requerimientos (SCRUM) a partir del análisis
> del código real de `ProyectosIA`: rutas (`server/routes/*.js`), middleware de roles
> (`requireRoles` / `requireSelfOrRoles`) y esquema Prisma. Los permisos reflejan lo que
> **efectivamente** protege cada endpoint.

**Roles:** ADMIN · OPERATIVO · JEFE · DIRECTOR · EMPLEADO
Convención de estimación: puntos Fibonacci (1,2,3,5,8,13). Prioridad: MoSCoW.

---

## Épica E1 — Autenticación y Seguridad (M1)

### HU-001 — Login
> Como **usuario** quiero autenticarme con usuario y contraseña para acceder al sistema.
```gherkin
Dado un usuario activo con credenciales válidas
Cuando envío usuario y contraseña a POST /auth/login
Entonces recibo un token de sesión y mis roles
Y si las credenciales son inválidas se registra LOGIN_FAILED en auditoría
```

### HU-002 — Cambio de contraseña forzado
> Como **usuario** con contraseña temporal quiero ser obligado a cambiarla para proteger mi cuenta.
```gherkin
Dado que tengo mustChangePwd = true
Cuando inicio sesión
Entonces no puedo operar hasta cambiar la contraseña vía POST /auth/change-password
Y se registra PASSWORD_CHANGE en auditoría
```

### HU-003 — Consultar sesión actual (/me)
> Como **usuario** autenticado quiero consultar mi perfil y roles para que la UI se adapte a mis permisos.
```gherkin
Dado que tengo sesión activa
Cuando llamo GET /auth/me
Entonces obtengo mi usuario, empleado vinculado y roles
```

### HU-004 — Logout
> Como **usuario** quiero cerrar sesión para terminar mi acceso de forma segura.
```gherkin
Dado que tengo sesión activa
Cuando llamo POST /auth/logout
Entonces mi sesión se invalida y se registra LOGOUT
```

### HU-005 — Gestión de usuarios y roles
> Como **Administrador** quiero crear usuarios y asignarles roles para controlar el acceso.
```gherkin
Dado que tengo rol ADMIN
Cuando creo un usuario y le asigno uno o más roles
Entonces el usuario puede autenticarse con los permisos de esos roles
```

---

## Épica E2 — Empleados y Tipos (M2)

### HU-006 — Listar empleados
> Como **ADMIN/OPERATIVO/JEFE/DIRECTOR** quiero listar empleados con filtros para ubicar personal.
```gherkin
Dado que tengo un rol de gestión o supervisión
Cuando llamo GET /employees con filtros (área, estatus, tipo)
Entonces obtengo la lista paginada de empleados
```

### HU-007 — Consultar ficha de empleado (self o gestión)
> Como **Empleado** quiero consultar mi propia ficha, y como supervisor la de mi personal.
```gherkin
Dado que soy el propio empleado o tengo rol ADMIN/OPERATIVO/JEFE/DIRECTOR
Cuando llamo GET /employees/:id
Entonces obtengo los datos del empleado
Y un EMPLEADO que consulta un id ajeno recibe 403
```

### HU-008 — Alta de empleado
> Como **ADMIN/OPERATIVO** quiero registrar un empleado con tipo, área e ID biométrico.
```gherkin
Dado que tengo rol ADMIN u OPERATIVO
Cuando creo un empleado con employeeNumber único
Entonces queda con estatus ACTIVO y se registra CREATE en auditoría
```

### HU-009 — Editar empleado
> Como **ADMIN/OPERATIVO** quiero editar los datos de un empleado para mantenerlos actualizados.
```gherkin
Dado un empleado existente
Cuando actualizo sus datos vía PUT /employees/:id
Entonces los cambios se guardan y se registra UPDATE
```

### HU-010 — Cambiar estatus de empleado
> Como **ADMIN/OPERATIVO** quiero cambiar el estatus (ACTIVO/NO_ACTIVO/BAJA/SUSPENDIDO).
```gherkin
Dado un empleado existente
Cuando llamo PATCH /employees/:id/status
Entonces el estatus se actualiza y se registra CHANGE_EMPLOYEE_STATUS
```

### HU-011 — Eliminar empleado
> Como **Administrador** quiero eliminar un empleado registrado por error.
```gherkin
Dado que tengo rol ADMIN
Cuando llamo DELETE /employees/:id
Entonces el empleado se elimina y se registra DELETE
Y OPERATIVO no puede ejecutar esta acción (403)
```

### HU-012 — Carga masiva de empleados
> Como **Administrador** quiero cargar empleados en lote para la puesta en marcha.
```gherkin
Dado un archivo/lote válido de empleados
Cuando llamo POST /employees/bulk
Entonces se crean los empleados válidos y se reportan los rechazados
```

### HU-013 — Consultar tipos de empleado
> Como **usuario** de gestión quiero consultar los tipos (DOC/DAC/PAAE) para clasificar personal.
```gherkin
Dado que estoy autenticado
Cuando llamo GET /employees/employee-types
Entonces obtengo los tipos con sus días económicos por defecto
```

---

## Épica E3 — Estructura Organizacional (M3)

### HU-014 — Gestionar departamentos
> Como **Administrador** quiero crear, editar y eliminar departamentos.
```gherkin
Dado que tengo rol ADMIN
Cuando uso POST/PUT/DELETE /org/departments
Entonces el catálogo de departamentos se actualiza
Y otros roles solo pueden consultarlos (GET)
```

### HU-015 — Gestionar áreas
> Como **Administrador** quiero crear, editar y eliminar áreas dentro de un departamento.
```gherkin
Dado que tengo rol ADMIN
Cuando uso POST/PUT/DELETE /org/areas
Entonces el catálogo de áreas se actualiza
```

### HU-016 — Asignar Jefe a un área
> Como **Administrador** quiero asignar el jefe inmediato de un área para el flujo de aprobaciones.
```gherkin
Dado un área existente y un usuario
Cuando llamo PATCH /org/areas/:id/jefe
Entonces ese usuario queda como jefe del área
```

### HU-017 — Asignar Director a un área
> Como **Administrador** quiero asignar el director de un área para la autorización final.
```gherkin
Dado un área existente y un usuario
Cuando llamo PATCH /org/areas/:id/director
Entonces ese usuario queda como director del área
```

---

## Épica E4 — Horarios (M4)

### HU-018 — Crear horario
> Como **ADMIN/OPERATIVO** quiero definir horarios con tolerancia para asignarlos a empleados.
```gherkin
Dado que tengo rol ADMIN u OPERATIVO
Cuando creo un horario vía POST /schedules
Entonces queda disponible con su tolerancia en minutos
```

### HU-019 — Definir detalle por día
> Como **ADMIN/OPERATIVO** quiero configurar entrada/salida por día de la semana y marcar días de descanso.
```gherkin
Dado un horario existente
Cuando defino ScheduleDetail por dayOfWeek (0-6)
Entonces cada día tiene entryTime/exitTime o isRestDay = true
```

### HU-020 — Asignar horario a empleado con vigencia
> Como **ADMIN/OPERATIVO** quiero asignar un horario a un empleado con fecha de vigencia.
```gherkin
Dado un empleado y un horario
Cuando llamo POST /schedules/assign con effectiveFrom
Entonces el horario queda vigente (effectiveTo null) y se registra ASSIGN_SCHEDULE
```

### HU-021 — Editar/eliminar horario
> Como **ADMIN/OPERATIVO** quiero editar horarios; como **ADMIN** eliminarlos.
```gherkin
Dado un horario existente
Cuando lo edito (PUT, ADMIN/OPERATIVO) o lo elimino (DELETE, solo ADMIN)
Entonces el cambio se aplica según el permiso del rol
```

### HU-022 — Consultar historial de horarios de un empleado
> Como supervisor o el propio empleado quiero ver el historial de horarios asignados.
```gherkin
Dado un empleado
Cuando llamo GET /employees/:id/schedules
Entonces obtengo sus asignaciones con vigencias
```

---

## Épica E5 — Días No Laborables (M5)

### HU-023 — Registrar día no laborable
> Como **ADMIN/OPERATIVO** quiero registrar festivos, puentes o periodos especiales por alcance (global/área/empleado).
```gherkin
Dado que tengo rol ADMIN u OPERATIVO
Cuando creo un día no laborable con scope y rango de fechas
Entonces se registra ADD_NON_WORKING_DAY y esos días no generan incidencia
```

### HU-024 — Carga masiva de días no laborables
> Como **ADMIN/OPERATIVO** quiero cargar el calendario oficial en lote.
```gherkin
Dado un conjunto de fechas oficiales
Cuando llamo POST /non-working-days/bulk
Entonces se crean todos los registros del periodo
```

### HU-025 — Editar/eliminar día no laborable
> Como **ADMIN/OPERATIVO** quiero corregir o quitar días no laborables mal capturados.
```gherkin
Dado un día no laborable existente
Cuando lo edito (PUT) o elimino (DELETE)
Entonces se registra DELETE_NON_WORKING_DAY cuando corresponde
```

### HU-026 — Consultar calendario anual
> Como **usuario** quiero ver el calendario de días no laborables del año.
```gherkin
Dado un año
Cuando llamo GET /non-working-days/calendar/:year
Entonces obtengo los días no laborables aplicables
```

---

## Épica E6 — Dispositivos ZKTeco (M6)

### HU-027 — Registrar dispositivo
> Como **ADMIN/OPERATIVO** quiero registrar un biométrico con IP y puerto para importar checadas.
```gherkin
Dado que tengo rol ADMIN u OPERATIVO
Cuando creo un dispositivo vía POST /devices
Entonces queda activo y disponible para pruebas e importación
```

### HU-028 — Probar conexión
> Como **ADMIN/OPERATIVO** quiero probar la conexión al dispositivo antes de importar.
```gherkin
Dado un dispositivo registrado
Cuando llamo POST /devices/:id/test-connection
Entonces obtengo el resultado de conectividad
```

### HU-029 — Descargar datos del dispositivo
> Como **ADMIN/OPERATIVO** quiero descargar las checadas almacenadas en el biométrico.
```gherkin
Dado un dispositivo alcanzable
Cuando llamo POST /devices/:id/download
Entonces se descargan las marcaciones y se registra DOWNLOAD_DEVICE_DATA
```

### HU-030 — Editar/eliminar dispositivo
> Como **ADMIN/OPERATIVO** quiero editar dispositivos; como **ADMIN** eliminarlos.
```gherkin
Dado un dispositivo existente
Cuando lo edito (PUT, ADMIN/OPERATIVO) o lo elimino (DELETE, solo ADMIN)
Entonces el cambio se aplica según el permiso del rol
```

---

## Épica E7 — Asistencia (M7)

### HU-031 — Importar checadas desde archivo
> Como **ADMIN/OPERATIVO** quiero importar un archivo .dat/.csv del software ZKTeco.
```gherkin
Dado un archivo exportado del biométrico
Cuando lo subo vía POST /attendance/import/file
Entonces obtengo una vista previa antes de confirmar
```

### HU-032 — Confirmar importación
> Como **ADMIN/OPERATIVO** quiero confirmar la importación tras revisar la vista previa.
```gherkin
Dado un lote en vista previa
Cuando llamo POST /attendance/import/confirm
Entonces se crean AttendanceRaw sin duplicados y se registra IMPORT_ATTENDANCE
```

### HU-033 — Importar directo desde dispositivo
> Como **ADMIN/OPERATIVO** quiero importar checadas conectando al dispositivo por IP.
```gherkin
Dado un dispositivo activo
Cuando llamo POST /attendance/import/device
Entonces las checadas se importan al sistema
```

### HU-034 — Procesar checadas en registros
> Como **ADMIN/OPERATIVO** quiero convertir las marcaciones crudas en registros de entrada/salida.
```gherkin
Dado AttendanceRaw sin procesar
Cuando llamo POST /attendance/process
Entonces se generan AttendanceRecord (entryTime/exitTime) por empleado y día
```

### HU-035 — Editar registro de asistencia
> Como **ADMIN/OPERATIVO** quiero corregir manualmente un registro erróneo.
```gherkin
Dado un registro existente
Cuando lo edito vía PUT /attendance/raw/:id
Entonces se guarda el cambio y se registra EDIT_ATTENDANCE_RECORD
```

### HU-036 — Eliminar registro de asistencia
> Como **ADMIN/OPERATIVO** quiero eliminar una marcación inválida.
```gherkin
Dado un registro existente
Cuando llamo DELETE /attendance/raw/:id
Entonces se elimina y se registra DELETE_ATTENDANCE_RECORD
```

### HU-037 — Consultar registros de asistencia
> Como **usuario** autenticado quiero consultar registros de asistencia (propios o de gestión).
```gherkin
Dado que estoy autenticado
Cuando llamo GET /attendance/records
Entonces obtengo los registros según mi alcance de rol
```

---

## Épica E8 — Incidencias (M8)

### HU-038 — Recalcular incidencias
> Como **ADMIN/OPERATIVO** quiero recalcular incidencias según horario vigente y días no laborables.
```gherkin
Dado empleados con checadas procesadas y horario vigente
Cuando llamo POST /incidents/recalculate
Entonces se generan/actualizan incidencias tipificadas y se registra RECALCULATE_INCIDENTS
Y los días no laborables no generan incidencia
```

### HU-039 — Listar incidencias con filtros
> Como **usuario** quiero listar incidencias por empleado, fecha, tipo o estatus.
```gherkin
Dado que estoy autenticado
Cuando llamo GET /incidents con filtros
Entonces obtengo las incidencias según mi alcance
```

### HU-040 — Resumen de incidencias del día
> Como **ADMIN/OPERATIVO/JEFE/DIRECTOR** quiero un tablero con el resumen del día.
```gherkin
Dado el día de hoy
Cuando llamo GET /incidents/summary/today
Entonces obtengo conteos por tipo de incidencia
```

### HU-041 — Consultar incidencias de un empleado
> Como supervisor o el propio empleado quiero ver el detalle de incidencias de un empleado.
```gherkin
Dado un empleado
Cuando llamo GET /employees/:id/incidents
Entonces obtengo sus incidencias (self o roles de gestión/supervisión)
```

---

## Épica E9 — Justificaciones (M9)

### HU-042 — Consultar catálogo de justificaciones
> Como **usuario** quiero ver los motivos de justificación disponibles.
```gherkin
Dado que estoy autenticado
Cuando llamo GET /justifications/catalog
Entonces obtengo los motivos activos con sus reglas (folio, nota, día económico)
```

### HU-043 — Administrar catálogo de justificaciones
> Como **Administrador** quiero crear y editar motivos de justificación.
```gherkin
Dado que tengo rol ADMIN
Cuando uso POST/PUT /justifications/catalog
Entonces el catálogo se actualiza
```

### HU-044 — Solicitar justificación
> Como **Empleado** quiero solicitar la justificación de una incidencia adjuntando evidencia.
```gherkin
Dado que tengo una incidencia PENDIENTE
Cuando envío POST /justifications/request con motivo y evidencia
Entonces la solicitud queda en PENDIENTE_JEFE y se registra REQUEST_JUSTIFICATION
```

### HU-045 — Visto bueno del Jefe
> Como **Jefe** quiero dar V°B° a las solicitudes de mi área.
```gherkin
Dado una solicitud en PENDIENTE_JEFE
Cuando llamo PATCH /justifications/requests/:id/approve
Entonces pasa a PENDIENTE_DIRECTOR y se registra APPROVE_JUSTIFICATION
```

### HU-046 — Autorización del Director
> Como **Director** quiero autorizar las solicitudes con V°B° del jefe.
```gherkin
Dado una solicitud en PENDIENTE_DIRECTOR
Cuando llamo PATCH /justifications/requests/:id/authorize
Entonces pasa a PENDIENTE_APLICAR y se registra AUTHORIZE_JUSTIFICATION
```

### HU-047 — Aplicar justificación
> Como **ADMIN/OPERATIVO** quiero aplicar la justificación autorizada sobre la incidencia.
```gherkin
Dado una solicitud en PENDIENTE_APLICAR
Cuando llamo PATCH /justifications/requests/:id/apply
Entonces la incidencia pasa a JUSTIFICADA y se registra APPLY_JUSTIFICATION
```

### HU-048 — Rechazar solicitud
> Como **JEFE/DIRECTOR/OPERATIVO/ADMIN** quiero rechazar una solicitud improcedente.
```gherkin
Dado una solicitud pendiente
Cuando llamo PATCH /justifications/requests/:id/reject
Entonces pasa a RECHAZADA_JEFE o RECHAZADA_DIRECTOR y se registra REJECT_JUSTIFICATION
```

### HU-049 — Contador de solicitudes pendientes
> Como supervisor quiero ver cuántas solicitudes tengo por atender.
```gherkin
Dado que soy jefe o director de un área
Cuando llamo GET /justifications/requests/pending-count
Entonces obtengo el número de solicitudes pendientes de mi acción
```

### HU-050 — Aplicación directa de justificación (sin flujo)
> Como **ADMIN/OPERATIVO** quiero aplicar directamente una justificación sobre una incidencia.
```gherkin
Dado una incidencia PENDIENTE
Cuando llamo POST /justifications/apply
Entonces se crea la justificación y la incidencia pasa a JUSTIFICADA
```

---

## Épica E10 — Licencias ISSSTE / CGT (M10)

### HU-051 — Registrar licencia ISSSTE
> Como **ADMIN/OPERATIVO** quiero registrar una licencia médica/maternidad con su documento.
```gherkin
Dado un empleado
Cuando llamo POST /licenses/issste con rango de fechas y documento
Entonces la licencia queda ACTIVA y se registra REGISTER_LICENSE
```

### HU-052 — Editar/cancelar licencia ISSSTE
> Como **ADMIN/OPERATIVO** quiero corregir o cancelar una licencia ISSSTE.
```gherkin
Dado una licencia existente
Cuando la edito (PUT) o la cancelo (PATCH /cancel)
Entonces se registra MODIFY_LICENSE o CANCEL_LICENSE
```

### HU-053 — Registrar/cancelar licencia CGT
> Como **ADMIN/OPERATIVO** quiero registrar y cancelar licencias sindicales (CGT).
```gherkin
Dado un tipo de licencia CGT aplicable al empleado
Cuando llamo POST /licenses/cgt o PATCH /licenses/cgt/:id/cancel
Entonces la licencia se registra o cancela según corresponda
```

### HU-054 — Eliminar incidencias por licencia
> Como **Operativo** quiero que las incidencias cubiertas por una licencia activa se eliminen automáticamente.
```gherkin
Dado un empleado con incidencias en un rango cubierto por licencia ACTIVA
Cuando la licencia queda vigente
Entonces las incidencias pasan a ELIMINADA_POR_LICENCIA
```

### HU-055 — Ciclo de vida de licencias
> Como **Administrador** quiero que las licencias cambien de estado automáticamente (PENDIENTE→ACTIVA→VENCIDA).
```gherkin
Dado licencias con fechas de inicio/fin
Cuando llamo POST /licenses/check-lifecycle
Entonces cada licencia se actualiza al estado que corresponde por fecha
```

---

## Épica E11 — Tiempo Extraordinario y Días Económicos (M11, M12)

### HU-056 — Consultar tiempo extraordinario
> Como **ADMIN/OPERATIVO/JEFE/DIRECTOR** quiero consultar las horas extra registradas.
```gherkin
Dado que tengo rol de gestión o supervisión
Cuando llamo GET /overtime
Entonces obtengo los registros de jornada extraordinaria
```

### HU-057 — Consultar saldo de horas extra por empleado
> Como usuario autorizado quiero ver el balance de horas extra de un empleado.
```gherkin
Dado un empleado con jornada extra habilitada
Cuando llamo GET /overtime/:employeeId/balance
Entonces obtengo sus horas acumuladas
```

### HU-058 — Consultar saldo de días económicos
> Como supervisor o el propio empleado quiero ver el saldo anual de días económicos.
```gherkin
Dado un empleado y un año
Cuando llamo GET /employees/:id/economic-balance
Entonces obtengo totalDays, usedDays y disponible
```

### HU-059 — Ajustar saldo de días económicos
> Como **Administrador** quiero ajustar manualmente el saldo de días económicos.
```gherkin
Dado un empleado con saldo del año
Cuando ajusto su balance
Entonces se registra ADJUST_ECONOMIC_BALANCE
```

---

## Épica E12 — Reportes (M13)

### HU-060 — Kardex de empleado
> Como usuario autorizado quiero un kardex de asistencia por empleado.
```gherkin
Dado un empleado y un periodo
Cuando llamo GET /reports/kardex
Entonces obtengo su historial de asistencia e incidencias
```

### HU-061 — Reporte de incidencias
> Como **ADMIN/OPERATIVO/JEFE/DIRECTOR** quiero un reporte de incidencias por periodo/área.
```gherkin
Dado un periodo y filtros
Cuando llamo GET /reports/incidents
Entonces obtengo el reporte agregado de incidencias
```

### HU-062 — Reporte de justificaciones
> Como **ADMIN/OPERATIVO** quiero un reporte de justificaciones aplicadas.
```gherkin
Dado un periodo
Cuando llamo GET /reports/justifications
Entonces obtengo las justificaciones con su motivo y estatus
```

### HU-063 — Reporte de tiempo extraordinario
> Como **ADMIN/OPERATIVO/JEFE** quiero un reporte de horas extra.
```gherkin
Dado un periodo
Cuando llamo GET /reports/overtime
Entonces obtengo las horas extra por empleado
```

### HU-064 — Reporte de días económicos y licencias
> Como **ADMIN/OPERATIVO** quiero reportes de saldos económicos y licencias.
```gherkin
Dado un periodo/año
Cuando llamo GET /reports/economic-days o /reports/licenses
Entonces obtengo los reportes correspondientes
```

### HU-065 — Resumen anual
> Como **ADMIN/OPERATIVO** quiero un resumen anual consolidado por empleado.
```gherkin
Dado un año
Cuando llamo GET /reports/annual-summary
Entonces obtengo el consolidado de asistencia, incidencias y justificaciones
```

---

## Épica E13 — Parámetros del Sistema (M14)

### HU-066 — Consultar parámetros
> Como **usuario** quiero consultar los parámetros del sistema que afectan la operación.
```gherkin
Dado que estoy autenticado
Cuando llamo GET /params
Entonces obtengo los parámetros y sus valores
```

### HU-067 — Actualizar parámetros
> Como **Administrador** quiero modificar tolerancias y umbrales del sistema.
```gherkin
Dado que tengo rol ADMIN
Cuando actualizo un parámetro (PUT /params/:key o /params/bulk)
Entonces el valor se guarda y se registra CHANGE_SYSTEM_PARAM
```

---

## Épica E14 — Auditoría (M15)

### HU-068 — Consultar bitácora
> Como **Administrador** quiero consultar la bitácora de acciones con filtros.
```gherkin
Dado que tengo rol ADMIN
Cuando llamo GET /audit con filtros (usuario, acción, tabla, fecha)
Entonces obtengo los registros de auditoría
Y ningún otro rol puede acceder (403)
```

### HU-069 — Exportar bitácora
> Como **Administrador** quiero exportar la bitácora para evidencia/cumplimiento.
```gherkin
Dado un rango de fechas
Cuando llamo GET /audit/export
Entonces obtengo el archivo exportable de auditoría
```

### HU-070 — Consultar detalle de un registro de auditoría
> Como **Administrador** quiero ver el detalle completo de un evento auditado.
```gherkin
Dado un id de auditoría
Cuando llamo GET /audit/:id
Entonces obtengo el detalle (usuario, acción, antes/después, fecha)
```

---

# Backlog Priorizado Consolidado

Criterio: valor de negocio × dependencia técnica. El núcleo (seguridad → personal →
asistencia → incidencias) es **Must**; flujos de aprobación y licencias **Must/Should**;
reportes, extras y configuración **Should/Could**.

| Épica | Historias | Prioridad dominante | Puntos (aprox.) |
|-------|-----------|:---:|:---:|
| E1 Seguridad | HU-001…005 | Must | 26 |
| E2 Empleados | HU-006…013 | Must | 34 |
| E3 Estructura Org | HU-014…017 | Must | 18 |
| E4 Horarios | HU-018…022 | Must | 29 |
| E5 Días No Laborables | HU-023…026 | Should | 16 |
| E6 Dispositivos | HU-027…030 | Must | 26 |
| E7 Asistencia | HU-031…037 | Must | 52 |
| E8 Incidencias | HU-038…041 | Must | 31 |
| E9 Justificaciones | HU-042…050 | Must | 55 |
| E10 Licencias | HU-051…055 | Should | 34 |
| E11 Extra / Económicos | HU-056…059 | Could | 21 |
| E12 Reportes | HU-060…065 | Should | 39 |
| E13 Parámetros | HU-066…067 | Should | 8 |
| E14 Auditoría | HU-068…070 | Must | 21 |

**Total:** 70 historias · ~410 puntos.

---

## Notas de verificación
- Permisos extraídos directamente de los `requireRoles(...)` / `requireSelfOrRoles(...)` de cada endpoint en `server/routes/*.js` (verificación por lectura de código, no ejecución).
- Correcciones respecto a la simulación previa:
  - Reportes de **overtime** SÍ incluyen a JEFE (`requireRoles('ADMIN','OPERATIVO','JEFE')`).
  - Estructura organizacional (departamentos/áreas y asignación de jefe/director) es **exclusiva de ADMIN**.
  - Eliminar empleado / dispositivo / horario es **exclusivo de ADMIN** (OPERATIVO no puede).
  - Catálogo de justificaciones lo administra **solo ADMIN**.
  - Rechazo de solicitudes lo pueden hacer JEFE, DIRECTOR, OPERATIVO y ADMIN.

## Preguntas abiertas
- Alcance por área: el código usa roles pero conviene confirmar si JEFE/DIRECTOR quedan además acotados a SU área en las consultas (filtro de datos, no solo de rol).
- ¿Se requiere notificación (correo/push) en las transiciones del flujo de justificaciones?
- ¿El EMPLEADO sin `User` vinculado queda fuera del autoservicio por diseño?
