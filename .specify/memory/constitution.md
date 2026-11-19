<!--
Sync Impact Report
==================
Version change: (plantilla sin versionar) → 1.0.0
Bump rationale: Ratificación inicial; todos los placeholders se reemplazan por contenido del
proyecto, incorporando las decisiones confirmadas por el usuario antes del primer commit.

Principios definidos:
- [PRINCIPLE_1_NAME] → I. Catálogo Cerrado de Canchas
- [PRINCIPLE_2_NAME] → II. Prevención de Colisiones (NO NEGOCIABLE)
- [PRINCIPLE_3_NAME] → III. Autenticación Obligatoria
- [PRINCIPLE_4_NAME] → IV. Errores Amigables y HTTP Semántico
- [PRINCIPLE_5_NAME] → V. Simplicidad y Cero Código Sombra

Secciones añadidas:
- Stack Tecnológico y Estructura (antes [SECTION_2_NAME])
- Estilo de Código y Flujo SDD (antes [SECTION_3_NAME])
- Governance (completada)

Secciones eliminadas: ninguna

Decisiones confirmadas por el usuario:
- Clases: libres en backend; funcional obligatorio en frontend.
- Bloques: fijos de 1 hora, alineados a HH:00, reloj de 24 h.
- Double-booking: transacción + restricción en PostgreSQL (atómico).

Templates dependientes: no modificados (leen la constitución en tiempo de ejecución).
TODOs diferidos: ninguno.
-->

# Constitución del Sistema de Reservas de Fútbol

## Core Principles

### I. Catálogo Cerrado de Canchas

El sistema gestiona reservas de tiempo para usuarios autenticados sobre un catálogo fijo.

- El sistema MUST manejar exactamente 5 canchas: **Cancha Azogues**, **Cancha Cuenca**,
  **Cancha Biblian**, **Cancha Cañar** y **Cancha Loja**.
- Las canchas MUST cargarse como datos semilla (seed) en `/db`; MUST NOT existir endpoints ni
  UI para crear, editar o eliminar canchas.
- Toda reserva que referencie una cancha fuera de este catálogo MUST rechazarse con HTTP 400.

**Rationale**: El dominio es cerrado; permitir altas dinámicas introduce complejidad no
solicitada y superficies de error.

### II. Prevención de Colisiones (NO NEGOCIABLE)

Es la regla crítica del sistema.

- Ninguna reserva MUST escribirse en la base de datos sin validar previamente que la cancha
  esté libre en el intervalo solicitado.
- La validación y la escritura MUST ser atómicas: ejecutarse dentro de una transacción y estar
  respaldadas por una restricción a nivel de PostgreSQL (p. ej. restricción de exclusión o
  índice único sobre cancha + intervalo) que impida solapamientos aun bajo concurrencia.
- Un intento de reservar un horario ocupado MUST responder HTTP 409 y la UI MUST mostrar un
  mensaje como "La cancha ya fue reservada en este horario".
- Los horarios MUST expresarse en formato de 24 horas (00:00–23:59).
- Las reservas MUST ocupar bloques fijos de 1 hora que inician en punto (HH:00); cada cancha
  dispone de 24 bloques por día (00:00–01:00 … 23:00–24:00). Se MUST rechazar con HTTP 400
  toda reserva cuyo inicio no esté alineado a la hora o cuya duración no sea de 1 hora.
- Toda funcionalidad que cree o modifique reservas MUST incluir pruebas que demuestren el
  rechazo de reservas solapadas.

**Rationale**: Un double-booking rompe la confianza del usuario; validar solo en aplicación
no protege contra peticiones simultáneas.

### III. Autenticación Obligatoria

- Todo flujo de reserva (crear, listar las propias, cancelar) MUST exigir un usuario con
  sesión activa.
- Las peticiones sin sesión válida MUST responder HTTP 401.
- Un usuario MUST poder operar únicamente sobre sus propias reservas.

**Rationale**: Las reservas pertenecen a una persona identificable; sin sesión no hay
responsable de la reserva.

### IV. Errores Amigables y HTTP Semántico

- La UI MUST NOT exponer errores crudos, mensajes de excepción ni *stack traces* al usuario
  final; todo error técnico MUST traducirse a un mensaje comprensible en español.
- El backend MUST retornar códigos HTTP semánticos: 400 (petición inválida),
  401 (no autenticado), 409 (conflicto de reserva), y 5xx solo para fallas del servidor.
- Las respuestas de error del backend MUST NOT incluir *stack traces* ni detalles internos
  de la base de datos.

**Rationale**: Mensajes claros reducen fricción; códigos semánticos permiten que el frontend
traduzca errores de forma determinista.

### V. Simplicidad y Cero Código Sombra

- Se MUST construir estrictamente lo documentado en `spec.md`. Se MUST NOT añadir
  características "por si acaso" (pasarelas de pago, perfiles complejos, notificaciones,
  paneles de administración, etc.) sin que estén especificadas.
- Se MUST aplicar Clean Architecture de forma ligera (dominio / casos de uso / infraestructura
  / presentación) sin capas, abstracciones o paquetes adicionales que no aporten valor
  inmediato.
- Toda complejidad adicional MUST justificarse explícitamente en `plan.md`.

**Rationale**: El valor del SDD depende de que el código refleje la especificación y nada más.

## Stack Tecnológico y Estructura

- **Lenguaje**: TypeScript en todo el stack (frontend, backend, scripts de BD).
- **Frontend**: React con componentes funcionales y Hooks; estilos con Tailwind CSS.
- **Backend**: NestJS.
- **Base de datos**: PostgreSQL accedido mediante TypeORM, ejecutado en contenedores Docker
  (p. ej. `docker compose`) para desarrollo.
- **Estructura de repositorio** (plana): `/frontend`, `/backend`, `/db`. Se MUST NOT crear
  directorios raíz adicionales sin justificación en `plan.md`.
- Sustituir cualquier tecnología del stack requiere enmendar esta constitución.

## Estilo de Código y Flujo SDD

- **Frontend (funcional obligatorio)**: React MUST usar exclusivamente componentes
  funcionales y Hooks; se MUST NOT usar componentes de clase ni clases para lógica de UI.
- **Backend (clases permitidas)**: En NestJS se MAY usar clases libremente (módulos,
  controladores, servicios, entidades TypeORM, DTOs, casos de uso). La lógica de dominio
  crítica (p. ej. detección de solapamiento de bloques) SHOULD aislarse en funciones puras
  testeables.
- **Nomenclatura**: `camelCase` para funciones y variables; `PascalCase` para interfaces,
  tipos, componentes React y clases requeridas por el framework.
- **Fuente de la verdad**: Si una instrucción del usuario contradice esta constitución, o si
  se detecta una falla lógica en la especificación, el agente de IA MUST detenerse, advertir
  del problema y solicitar la actualización de `spec.md` antes de modificar código fuente.
- **Puertas de calidad**: Cada `plan.md` MUST incluir un "Constitution Check" que verifique
  los principios I–V antes de pasar a tareas e implementación.

## Governance

- Esta constitución prevalece sobre cualquier otra práctica, guía o instrucción puntual.
- **Enmiendas**: Se realizan mediante `/speckit-constitution`, documentando el cambio en el
  Sync Impact Report y revisando su impacto en specs, planes y tareas existentes.
- **Versionado** (SemVer):
  - MAJOR: eliminación o redefinición incompatible de principios o del stack.
  - MINOR: nuevo principio/sección o ampliación material de una guía.
  - PATCH: aclaraciones, redacción o correcciones sin cambio semántico.
- **Cumplimiento**: Toda revisión de código y todo `plan.md` MUST verificar el cumplimiento
  de los principios; cualquier desviación MUST quedar justificada por escrito o rechazarse.

**Version**: 1.0.0 | **Ratified**: 2026-11-19 | **Last Amended**: 2026-11-19
