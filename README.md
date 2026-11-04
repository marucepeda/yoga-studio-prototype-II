# Yoga Studio — Class Booking Management

Proyecto II de Fundamentos de Product Engineering con IA · Código Facilito.

MVP funcional para la gestión de clases y reservas de un pequeño estudio de yoga.

## El problema

La gestión manual de paquetes mediante tarjetas físicas y las reservas informales pueden generar errores en los saldos, sobrecupos y dificultades para administrar cancelaciones y asistencia.

Este proyecto explora una solución digital para centralizar esas operaciones.

## Funcionalidades implementadas

- Gestión de reservas con validación de créditos disponibles.
- Control de capacidad configurable por clase.
- Lista de espera con orden FIFO.
- Cancelaciones y promoción automática de alumnas en espera.
- Registro de asistencia y consumo de créditos.
- Historial auditable de movimientos de créditos.
- Roles simulados para administración, instructoras y alumnas.
- Datos ficticios de demostración y reinicio del entorno.
- Políticas configurables de cancelación y ausencia.

## Tecnología y alcance

Aplicación web construida con HTML, CSS y JavaScript.

- Persistencia local mediante `localStorage`.
- Datos de demostración precargados.
- Sin backend ni base de datos remota.
- Sin autenticación real ni sincronización entre dispositivos.
- No apta todavía para gestionar información real de alumnas.

La demo permite explorar y probar las reglas de negocio en un único navegador.

## Estructura del proyecto

- `index.html`: aplicación web y lógica del MVP.
- `AGENTS.md`: contexto e instrucciones persistentes para el agente.
- `specs/class-booking/spec.md`: especificación funcional y criterios de aceptación.
- `.agents/skills/yoga-brand-design/SKILL.md`: lineamientos de identidad visual.

## Cómo probarlo

1. Clonar o descargar el repositorio.
2. Abrir `index.html` en un navegador.
3. Explorar los roles y las funcionalidades con los datos de demostración.
4. Usar la opción de reinicio para recuperar el estado inicial.

## Metodología

El desarrollo sigue un enfoque de Specification-Driven Development (SDD): contexto persistente, especificación funcional, criterios de aceptación e implementación asistida por IA.

## Proyecto académico

Desarrollado como parte de **Fundamentos de Product Engineering con IA**, de Código Facilito.