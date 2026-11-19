# Feature Specification: Reserva de Canchas de Fútbol

**Feature Branch**: `001-court-booking`

**Created**: 2026-11-19

**Status**: Draft

**Input**: User description: "Autenticación de usuarios (correo + contraseña), exploración de las 5
canchas con grilla de disponibilidad de 24 bloques de 1 hora por fecha, creación de reservas con
re-validación anti-colisión, y panel 'Mis Reservas' con
reservas futuras, historial y cancelación. Sin pagos, sin panel de administración, sin
notificaciones externas, sin reservas multi-hora, sin matchmaking."

> **Nota de alineación con la constitución**: la descripción original listaba las canchas
> Laureles, El Poblado, Belén, Robledo y Envigado. Por decisión del usuario se usa el catálogo
> fijado en el Principio I de la constitución: Azogues, Cuenca, Biblian, Cañar y Loja.
>
> **Decisiones del usuario (2026-11-19)**: (1) se elimina la regla "un usuario solo puede tener
> una reserva activa a la vez", que contradecía el Non-Goal de reservar 2 horas mediante 2
> reservas independientes; (2) el horizonte de reserva es de 7 días.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Registro e inicio de sesión (Priority: P1)

Una persona visitante crea una cuenta con su correo electrónico y una contraseña, y luego inicia
sesión para acceder a la disponibilidad completa y a las reservas. También puede cerrar sesión.

**Why this priority**: Sin una sesión activa no es posible ver la disponibilidad completa ni
reservar; es la puerta de entrada a todo el valor del sistema.

**Independent Test**: Registrar una cuenta nueva, cerrar sesión, iniciar sesión con las mismas
credenciales y comprobar que se accede al área privada; intentar acceder al área privada sin
sesión y comprobar que se redirige al inicio de sesión.

**Acceptance Scenarios**:

1. **Given** una persona sin cuenta, **When** se registra con un correo válido no usado y una
   contraseña que cumple la política, **Then** la cuenta se crea y queda con sesión activa.
2. **Given** un correo ya registrado, **When** alguien intenta registrarse con él, **Then** el
   registro se rechaza con el mensaje "Este correo ya está registrado".
3. **Given** un usuario registrado, **When** inicia sesión con credenciales correctas, **Then**
   accede al área privada.
4. **Given** un usuario registrado, **When** inicia sesión con una contraseña incorrecta,
   **Then** ve el mensaje "Correo o contraseña incorrectos" sin revelar cuál de los dos falló.
5. **Given** un usuario con sesión activa, **When** cierra sesión, **Then** pierde el acceso al
   área privada hasta volver a iniciar sesión.

---

### User Story 2 - Consultar disponibilidad y reservar un bloque (Priority: P1)

Un usuario con sesión activa elige una de las 5 canchas, selecciona una fecha en un calendario y
ve la grilla de 24 bloques de 1 hora (00:00–01:00 … 23:00–24:00), cada uno marcado como
"Disponible" o "Reservado". Elige un bloque disponible, confirma, y la reserva queda registrada.

**Why this priority**: Es el propósito central del sistema. Junto con la Historia 1 forma el MVP.

**Independent Test**: Con un usuario con sesión activa, abrir una cancha y una fecha futura,
reservar un bloque disponible y comprobar que ese bloque pasa a "Reservado" para cualquier
usuario que consulte la misma cancha y fecha.

**Acceptance Scenarios**:

1. **Given** un usuario con sesión activa, **When** abre una cancha y selecciona una fecha,
   **Then** ve exactamente 24 bloques de 1 hora, cada uno con estado "Disponible" o "Reservado".
2. **Given** un bloque futuro "Disponible", **When** el usuario lo selecciona y confirma,
   **Then** la reserva se crea y el bloque se muestra como "Reservado".
3. **Given** dos usuarios que ven el mismo bloque como "Disponible", **When** ambos confirman
   casi al mismo tiempo, **Then** solo uno obtiene la reserva y el otro ve el mensaje "La cancha
   ya fue reservada en este horario" y la grilla actualizada.
4. **Given** un usuario que ya reservó el bloque 14:00–15:00, **When** reserva también el
   bloque 15:00–16:00 de la misma cancha, **Then** obtiene dos reservas independientes de 1 hora.
5. **Given** un bloque cuya hora de inicio ya pasó (incluido el bloque en curso), **When** el
   usuario consulta la grilla, **Then** el bloque no se puede seleccionar; y si se intenta
   reservar igualmente, se rechaza con "No puedes reservar un horario que ya pasó".
6. **Given** una persona sin sesión activa, **When** intenta ver la grilla de disponibilidad o
   reservar, **Then** se le pide iniciar sesión.
7. **Given** el calendario, **When** el usuario intenta elegir una fecha a más de 7 días de hoy,
   **Then** la fecha no es seleccionable; y si se intenta reservar igualmente, se rechaza con
   "Solo puedes reservar hasta 7 días por adelantado".

---

### User Story 3 - Mis Reservas: consultar y cancelar (Priority: P2)

Un usuario con sesión activa abre su panel "Mis Reservas" y ve dos listas: reservas futuras e
historial de reservas pasadas. Cada ítem muestra nombre de la cancha, fecha y hora. Puede
cancelar una reserva futura, previa confirmación, y el bloque vuelve a quedar disponible.

**Why this priority**: Aporta control al usuario sobre sus reservas y permite liberar bloques
que no usará; pero el sistema ya entrega valor sin ella.

**Independent Test**: Con un usuario que tiene una reserva futura y al menos una pasada, abrir
el panel, verificar ambas listas, cancelar la futura y comprobar que desaparece de la lista y
que el bloque aparece "Disponible" en la grilla.

**Acceptance Scenarios**:

1. **Given** un usuario con reservas futuras y pasadas, **When** abre "Mis Reservas", **Then**
   ve las futuras ordenadas de la más próxima a la más lejana y el historial de la más reciente
   a la más antigua, cada ítem con cancha, fecha y hora.
2. **Given** una reserva futura propia, **When** el usuario elige cancelarla, **Then** el
   sistema pide confirmación antes de cancelar.
3. **Given** la confirmación de cancelación, **When** el usuario acepta, **Then** la reserva
   deja de estar activa, desaparece de la lista de futuras y el bloque vuelve a "Disponible".
4. **Given** una reserva del historial (pasada), **When** el usuario la ve, **Then** no tiene
   opción de cancelar.
5. **Given** un usuario sin reservas, **When** abre "Mis Reservas", **Then** ve un mensaje
   indicando que aún no tiene reservas.

---

### Edge Cases

- **Carrera entre usuarios**: dos confirmaciones simultáneas del mismo bloque y cancha → solo
  una tiene éxito; la otra recibe el mensaje de conflicto.
- **Grilla desactualizada**: el usuario dejó la grilla abierta y otro reservó el bloque → al
  confirmar se re-valida y se rechaza con el mensaje de conflicto.
- **Bloque en curso**: a las 14:20, el bloque 14:00–15:00 se considera transcurrido y no es
  reservable.
- **Cambio de día durante la sesión**: si el usuario confirma un bloque que pasó a ser pasado
  mientras tenía la grilla abierta, la reserva se rechaza.
- **Reserva que finaliza**: cuando termina el bloque de una reserva, esta pasa de "futuras" al
  historial.
- **Mismo usuario, mismo horario, otra cancha**: el sistema no lo impide; cada reserva se valida
  solo contra la cancha y el bloque que ocupa.
- **Cancelar un bloque ya iniciado**: si el usuario intenta cancelar una reserva cuyo bloque ya
  empezó, la cancelación se rechaza porque la reserva ya no es futura.
- **Acceso a reservas ajenas**: un usuario que intenta ver o cancelar una reserva de otro
  recibe un rechazo sin revelar datos de esa reserva.
- **Sesión expirada**: si la sesión expira durante el flujo de reserva, se pide iniciar sesión
  de nuevo y no se crea ninguna reserva.
- **Fecha lejana**: el usuario intenta reservar a más de 7 días de hoy → se rechaza (FR-013).

## Requirements *(mandatory)*

### Functional Requirements

**Autenticación**

- **FR-001**: El sistema MUST permitir registrarse con correo electrónico y contraseña.
- **FR-002**: El sistema MUST rechazar registros con un correo ya registrado o con formato
  inválido, y con contraseñas de menos de 8 caracteres.
- **FR-003**: El sistema MUST permitir iniciar y cerrar sesión con correo y contraseña.
- **FR-004**: El sistema MUST exigir una sesión activa para ver la grilla de disponibilidad,
  crear reservas y acceder a "Mis Reservas"; sin sesión MUST responder como no autenticado y
  la interfaz MUST dirigir al inicio de sesión.
- **FR-005**: Un usuario MUST poder ver y cancelar únicamente sus propias reservas.

**Canchas y disponibilidad**

- **FR-006**: El sistema MUST mostrar exactamente 5 canchas fijas: Cancha Azogues, Cancha
  Cuenca, Cancha Biblian, Cancha Cañar y Cancha Loja. No existe forma de crearlas, editarlas o
  eliminarlas desde la aplicación.
- **FR-007**: El usuario MUST poder seleccionar en un calendario una fecha entre hoy y 7 días
  después de hoy (inclusive) para una cancha elegida; las demás fechas no son seleccionables.
- **FR-008**: Para la cancha y fecha elegidas, el sistema MUST mostrar 24 bloques de 1 hora en
  formato de 24 horas (00:00–01:00 … 23:00–24:00), cada uno etiquetado "Disponible" o
  "Reservado".
- **FR-009**: Los bloques cuya hora de inicio ya pasó MUST mostrarse como no seleccionables.

**Reservas**

- **FR-010**: El usuario MUST poder seleccionar un bloque disponible y confirmar su reserva.
- **FR-011**: Al confirmar, el sistema MUST re-validar que el bloque siga libre y registrar la
  reserva de forma que dos reservas nunca puedan ocupar la misma cancha en el mismo bloque, aun
  si se confirman simultáneamente. Si el bloque ya está ocupado, MUST rechazarse como conflicto
  con el mensaje "La cancha ya fue reservada en este horario".
- **FR-012**: Cada reserva MUST corresponder a exactamente un bloque de 1 hora que comienza en
  punto (HH:00); el sistema MUST rechazar cualquier otra duración o alineación.
- **FR-013**: El sistema MUST rechazar reservas cuyo bloque haya iniciado o terminado, y
  reservas con fecha posterior a 7 días después de hoy.
- **FR-014**: Un usuario MAY tener varias reservas a la vez, incluidas reservas consecutivas en
  la misma cancha; cada reserva es independiente y se crea con una confirmación propia.

**Mis Reservas**

- **FR-015**: El sistema MUST ofrecer un panel "Mis Reservas" con dos listas: reservas futuras
  (no canceladas cuyo bloque no ha iniciado) e historial (reservas cuyo bloque ya terminó).
- **FR-016**: Cada ítem MUST mostrar nombre de la cancha, fecha y hora del bloque.
- **FR-017**: El usuario MUST poder cancelar una reserva futura propia tras confirmar la acción;
  la cancelación MUST liberar el bloque inmediatamente.
- **FR-018**: Las reservas canceladas MUST NOT aparecer en ninguna de las dos listas.

**Errores**

- **FR-019**: Todo error MUST mostrarse al usuario como un mensaje comprensible en español, sin
  detalles técnicos.

### Key Entities

- **Usuario**: persona registrada. Atributos: correo (único), contraseña (almacenada de forma
  segura, nunca visible), fecha de registro.
- **Cancha**: una de las 5 canchas fijas. Atributos: nombre. Inmutable.
- **Reserva**: ocupación de una cancha por un usuario en un bloque. Atributos: usuario, cancha,
  fecha, hora de inicio (HH:00; fin = inicio + 1 h), estado (activa / cancelada), fecha de
  creación. Regla: no puede existir más de una reserva no cancelada para la misma cancha, fecha
  y hora de inicio.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Un usuario nuevo puede registrarse y completar su primera reserva en menos de
  2 minutos.
- **SC-002**: Un usuario con sesión activa ve la grilla de disponibilidad de una cancha y fecha
  en menos de 2 segundos.
- **SC-003**: 0 reservas duplicadas (misma cancha, fecha y hora) en pruebas de 50 intentos
  simultáneos sobre el mismo bloque: exactamente 1 éxito y 49 rechazos con mensaje de conflicto.
- **SC-004**: 100% de los intentos de reservar en el pasado, en bloques no alineados o a más de
  7 días son rechazados con un mensaje comprensible.
- **SC-005**: 100% de los intentos de ver o cancelar reservas ajenas son rechazados.
- **SC-006**: Ningún mensaje mostrado al usuario contiene trazas, códigos internos o mensajes
  técnicos.
- **SC-007**: Al menos el 90% de los usuarios de prueba completan una cancelación desde
  "Mis Reservas" en el primer intento.

## Assumptions

- Todos los horarios se interpretan en la zona horaria local del club (una sola zona horaria).
- El bloque en curso cuenta como transcurrido: no se puede reservar ni cancelar.
- Una persona sin sesión puede ver la lista de las 5 canchas, pero no la grilla de
  disponibilidad ni las acciones de reserva ("disponibilidad completa" solo con sesión).
- Política de contraseña mínima: 8 caracteres. No se incluye recuperación de contraseña ni
  verificación de correo en esta iteración (implicarían correos, excluidos por Non-Goals).
- La sesión permanece activa hasta que el usuario la cierra o expira por inactividad según un
  plazo estándar.
- Se puede cancelar una reserva en cualquier momento antes de que comience su bloque.
- Las reservas canceladas se conservan internamente pero no se muestran al usuario.
- El pago se realiza presencialmente en el club; el sistema no registra pagos.

### Non-Goals (fuera de alcance en esta iteración)

- Pasarela de pagos integrada.
- Panel de administración para gestionar canchas.
- Notificaciones externas (correo, SMS, WhatsApp).
- Reservas de más de 1 hora continua en una sola acción (2 horas = 2 reservas independientes).
- Matchmaking, búsqueda de compañeros o torneos.
