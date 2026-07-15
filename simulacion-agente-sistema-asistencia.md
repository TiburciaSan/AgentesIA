# Simulación del Agente Analista de Requerimientos (SCRUM)
## Caso: Sistema de Control de Asistencia de Personal (ProyectosIA)

> Entregables generados por el agente analista senior de requerimientos a partir del
> análisis del repositorio `ProyectosIA` (Node/Express + Prisma/PostgreSQL, integración
> biométrica ZKTeco).

**Roles del sistema:** ADMIN · OPERATIVO · JEFE (inmediato) · DIRECTOR (de área) · EMPLEADO

---

# 1. Catálogo de Módulos

| # | Módulo | Descripción |
|---|--------|-------------|
| M1 | Autenticación y Seguridad | Login, cambio de contraseña obligatorio, gestión de usuarios/roles |
| M2 | Empleados | Alta/edición de personal, tipos (DOC/DAC/PAAE), estatus y datos biométricos |
| M3 | Estructura Organizacional | Departamentos y áreas con jefe/director asignados |
| M4 | Horarios | Definición de horarios, detalle por día y vigencias por empleado |
| M5 | Días No Laborables | Festivos, vacaciones institucionales, puentes (global/área/empleado) |
| M6 | Dispositivos ZKTeco | Registro de biométricos, prueba de conexión y descarga de datos |
| M7 | Asistencia | Importación de checadas, cálculo de registros y edición manual |
| M8 | Incidencias | Motor que genera faltas/retardos y su recálculo |
| M9 | Justificaciones | Solicitud → V°B° Jefe → autorización Director → aplicación Operativo |
| M10 | Licencias (ISSSTE / CGT) | Registro de licencias médicas y sindicales que eliminan incidencias |
| M11 | Tiempo Extraordinario | Jornada extra habilitada por empleado y cálculo de horas |
| M12 | Días Económicos | Saldo anual por empleado y ajustes |
| M13 | Reportes | Generación y exportación de reportes de asistencia/incidencias |
| M14 | Parámetros del Sistema | Tolerancias, umbrales y configuración global |
| M15 | Auditoría | Bitácora inmutable de todas las acciones sensibles |

## Matriz de acciones por rol

Leyenda: ✔ permitido · 👁 solo lectura · — sin acceso

| Módulo | Acción | ADMIN | OPERATIVO | JEFE | DIRECTOR | EMPLEADO |
|--------|--------|:---:|:---:|:---:|:---:|:---:|
| M1 | Gestionar usuarios/roles | ✔ | — | — | — | — |
| M1 | Cambiar contraseña propia | ✔ | ✔ | ✔ | ✔ | ✔ |
| M2 | Crear/editar empleado | 👁 | ✔ | — | — | — |
| M2 | Cambiar estatus empleado | 👁 | ✔ | — | — | — |
| M2 | Consultar ficha propia | — | ✔ | 👁 | 👁 | 👁 |
| M3 | Gestionar depto/área | ✔ | 👁 | — | — | — |
| M4 | Crear horario / asignar | 👁 | ✔ | 👁 | 👁 | — |
| M5 | Registrar día no laborable | ✔ | ✔ | — | — | — |
| M6 | Registrar/probar dispositivo | ✔ | ✔ | — | — | — |
| M6 | Descargar datos del biométrico | 👁 | ✔ | — | — | — |
| M7 | Importar asistencia | 👁 | ✔ | — | — | — |
| M7 | Editar/eliminar registro | 👁 | ✔ | — | — | — |
| M7 | Consultar asistencia propia | — | ✔ | 👁 | 👁 | 👁 |
| M8 | Recalcular incidencias | 👁 | ✔ | — | — | — |
| M8 | Ver incidencias del área | — | ✔ | 👁 | 👁 | 👁 (propias) |
| M9 | Solicitar justificación | — | ✔ | ✔ | ✔ | ✔ |
| M9 | Dar V°B° (Jefe) | — | — | ✔ | — | — |
| M9 | Autorizar (Director) | — | — | — | ✔ | — |
| M9 | Aplicar justificación | 👁 | ✔ | — | — | — |
| M10 | Registrar/cancelar licencia | 👁 | ✔ | — | — | — |
| M11 | Habilitar tiempo extra | 👁 | ✔ | — | — | — |
| M12 | Ajustar saldo económico | ✔ | ✔ | — | — | 👁 |
| M13 | Generar/exportar reportes | ✔ | ✔ | 👁 (área) | 👁 (área) | 👁 (propio) |
| M14 | Cambiar parámetros | ✔ | — | — | — | — |
| M15 | Consultar auditoría | ✔ | — | — | — | — |

---

# 2. Historias de Usuario

### HU-001 — Login con cambio de contraseña forzado · Módulo M1
> Como **usuario** quiero iniciar sesión y ser obligado a cambiar mi contraseña temporal para asegurar mi cuenta.

```gherkin
Dado que soy un usuario con mustChangePwd = true
Cuando inicio sesión con credenciales válidas
Entonces el sistema me redirige a la pantalla de cambio de contraseña
Y no puedo acceder a otros módulos hasta cambiarla
```

### HU-002 — Alta de empleado · Módulo M2
> Como **Operativo** quiero registrar un empleado con su tipo, área e ID biométrico para incorporarlo al control de asistencia.

```gherkin
Dado que tengo rol OPERATIVO
Cuando registro un empleado con employeeNumber único y área válida
Entonces el empleado queda con estatus ACTIVO
Y queda disponible para asignarle horario y checadas
```

### HU-003 — Importar checadas del biométrico · Módulo M7
> Como **Operativo** quiero importar registros desde un ZKTeco (IP o archivo) para procesar la asistencia diaria.

```gherkin
Dado que existe un dispositivo registrado y activo
Cuando importo checadas por IP o archivo .dat/.csv
Entonces se crean registros AttendanceRaw sin duplicados (zkUserId+timestamp+punch)
Y puedo ver una vista previa antes de confirmar
```

### HU-004 — Generación automática de incidencias · Módulo M8
> Como **Operativo** quiero que el sistema calcule faltas y retardos según el horario vigente para no hacerlo manualmente.

```gherkin
Dado un empleado con horario vigente y checadas procesadas
Cuando ejecuto el recálculo de incidencias
Entonces se generan incidencias tipificadas (RETARDO_MENOR, FALTA_COMPLETA, etc.)
Y los días no laborables NO generan incidencia
```

### HU-005 — Flujo de justificación (Empleado→Jefe→Director→Operativo) · Módulo M9
> Como **Empleado** quiero solicitar la justificación de una incidencia para que sea revisada por mi jefe y autorizada por el director.

```gherkin
Dado que tengo una incidencia en estado PENDIENTE
Cuando solicito su justificación con un motivo del catálogo
Entonces la solicitud queda en estado PENDIENTE_JEFE
Cuando el Jefe da V°B° pasa a PENDIENTE_DIRECTOR
Cuando el Director autoriza pasa a PENDIENTE_APLICAR
Cuando el Operativo la aplica la incidencia pasa a JUSTIFICADA
```

### HU-006 — Licencia que elimina incidencias · Módulo M10
> Como **Operativo** quiero registrar una licencia ISSSTE/CGT para que las incidencias del período se eliminen automáticamente.

```gherkin
Dado un empleado con incidencias en un rango de fechas
Cuando registro una licencia ACTIVA que cubre ese rango
Entonces las incidencias pasan a ELIMINADA_POR_LICENCIA
```

### HU-007 — Consulta de asistencia propia · Módulo M7
> Como **Empleado** quiero consultar mis registros e incidencias para conocer mi situación.

```gherkin
Dado que tengo rol EMPLEADO
Cuando ingreso a "Mi asistencia"
Entonces solo veo mis propios registros e incidencias
Y no puedo ver los de otros empleados
```

### HU-008 — Auditoría de acciones · Módulo M15
> Como **Administrador** quiero una bitácora de todas las acciones sensibles para garantizar trazabilidad.

```gherkin
Dado que ocurre una acción auditable (LOGIN, APPLY_JUSTIFICATION, etc.)
Cuando se ejecuta
Entonces se registra en la bitácora con usuario, acción, fecha y detalle
Y ningún rol puede modificar ni borrar esos registros
```

> Catálogo total estimado: ~45–55 historias. Arriba se presenta la muestra troncal por módulo.

---

# 3. Backlog Priorizado

**Criterio de priorización:** valor de negocio × dependencia técnica. Primero el núcleo sin
el cual nada funciona (seguridad → empleados → asistencia → incidencias), luego los flujos de
aprobación y por último reportes/configuración.

| ID | Historia | Épica | Prioridad | Valor (1-5) | Est. (pts) | Dependencias |
|----|----------|-------|:---:|:---:|:---:|----|
| **HU-001** | Login + cambio pwd forzado | E1 Seguridad | **Must** | 5 | 5 | — |
| **HU-008** | Auditoría de acciones | E1 Seguridad | **Must** | 4 | 8 | HU-001 |
| **HU-002** | Alta/edición empleado | E2 Personal | **Must** | 5 | 5 | HU-001 |
| — | Estructura org (depto/área) | E2 Personal | **Must** | 4 | 5 | HU-001 |
| — | Horarios + vigencias | E2 Personal | **Must** | 4 | 8 | HU-002 |
| **HU-003** | Importar checadas ZKTeco | E3 Asistencia | **Must** | 5 | 13 | HU-002 |
| — | Registro/prueba dispositivos | E3 Asistencia | **Must** | 4 | 8 | HU-001 |
| **HU-004** | Motor de incidencias | E3 Asistencia | **Must** | 5 | 13 | HU-003, Horarios |
| — | Días no laborables | E3 Asistencia | **Should** | 3 | 5 | HU-004 |
| **HU-005** | Flujo de justificaciones | E4 Aprobaciones | **Must** | 5 | 13 | HU-004 |
| **HU-006** | Licencias ISSSTE/CGT | E4 Aprobaciones | **Should** | 4 | 8 | HU-004 |
| **HU-007** | Consulta asistencia propia | E5 Autoservicio | **Should** | 3 | 3 | HU-004 |
| — | Tiempo extraordinario | E6 Nómina | **Could** | 3 | 8 | HU-003 |
| — | Días económicos (saldo) | E6 Nómina | **Could** | 3 | 5 | HU-006 |
| **HU-013** | Reportes + exportación | E7 Reportes | **Should** | 4 | 8 | HU-004 |
| — | Parámetros del sistema | E8 Config | **Should** | 3 | 3 | HU-001 |

---

## Supuestos
- `[SUPUESTO]` El Empleado puede autoservicio solo si tiene `User` vinculado (`employeeId`); no todos los empleados tienen cuenta.
- `[SUPUESTO]` "Descargar datos del biométrico" y "editar registro" son exclusivos de OPERATIVO/ADMIN según middleware de roles (a confirmar contra `roles.middleware.js`).
- `[SUPUESTO]` ADMIN no gestiona empleados/asistencia directamente (solo seguridad, parámetros y auditoría), según README.

## Preguntas abiertas
- ¿El DIRECTOR/JEFE pueden generar reportes de toda su área o solo consultarlos?
- ¿Las licencias CGT/ISSSTE las registra solo OPERATIVO o también ADMIN?
- ¿Existe notificación (correo) en el flujo de justificaciones o es solo cambio de estado en pantalla?
