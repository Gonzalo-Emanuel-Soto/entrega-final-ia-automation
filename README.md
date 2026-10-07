Entrega final – AI Automation

1. Resumen de información:
Repositorio Github: https://github.com/Gonzalo-Emanuel-Soto/entrega-final-ia-automation
Base de datos Airtable:
https://airtable.com/appbn42w9qKUO7mks/tblZrNu4tQKceB3Tu/viw2S2HhUVlImo5AC
Orquestador: N8N
Resumen proyecto: Ecosistema que captura tickets, los clasifica con IA en categorías
(Pendiente / Procesado por IA / Aprobado por Humano) y valida con una aprobación
humana antes de responder al cliente final.

# 🏥 Consultorio Médico — Sistema de Turnos Automatizado con IA

Sistema de automatización desarrollado en **n8n** para gestionar de punta a punta la solicitud de turnos de un consultorio médico.

El proyecto integra **Telegram, formularios de n8n, Google Calendar, Airtable, Inteligencia Artificial y Gmail**, incorporando además un proceso de **Human-in-the-Loop (HITL)** para que la aprobación final del turno sea realizada por una persona.

---

## 📌 Descripción del proyecto

El objetivo del proyecto es automatizar el proceso de reserva de turnos médicos, desde el primer contacto del paciente hasta la confirmación definitiva.

El paciente comienza la interacción mediante un bot de **Telegram**, selecciona una especialidad y recibe un formulario para completar sus datos.

A partir de allí, **n8n** se encarga de:

- Validar y normalizar los datos ingresados.
- Verificar que la fecha solicitada sea válida.
- Consultar la disponibilidad en Google Calendar.
- Registrar pacientes y turnos en Airtable.
- Analizar la solicitud mediante Inteligencia Artificial.
- Enviar la solicitud al médico para aprobación.
- Crear automáticamente el evento en Google Calendar.
- Confirmar el turno al paciente por Telegram y Gmail.
- Registrar errores y situaciones inválidas.

---

## 🎯 Objetivos

El sistema busca:

- Automatizar la gestión de turnos.
- Reducir tareas administrativas manuales.
- Evitar errores en la carga de información.
- Prevenir superposición de horarios.
- Centralizar la información de pacientes y turnos.
- Incorporar IA como herramienta de apoyo.
- Mantener una aprobación humana antes de confirmar el turno.
- Registrar errores para facilitar el monitoreo del sistema.

---

## 🛠️ Tecnologías utilizadas

| Tecnología | Función |
|---|---|
| **n8n** | Orquestación y automatización del workflow |
| **Telegram Bot** | Interacción inicial con el paciente |
| **n8n Form** | Formulario para solicitar el turno |
| **Airtable** | Base de datos de pacientes, turnos y errores |
| **Google Calendar** | Consulta de disponibilidad y creación de turnos |
| **IA / API** | Análisis administrativo de la solicitud |
| **Gmail** | Aprobación humana y confirmaciones por email |
| **JavaScript** | Validación, transformación y normalización de datos |

---

# 🔄 Arquitectura del sistema

El workflow está dividido en diferentes etapas:

```text
Paciente
   │
   ▼
Telegram Bot
   │
   ▼
Selección de especialidad
   │
   ▼
Formulario de turno
   │
   ▼
Validación y normalización
   │
   ├── Datos inválidos ──► Airtable Errores ──► Telegram
   │
   ▼
Google Calendar
Consulta disponibilidad
   │
   ├── Horario ocupado ──► Telegram
   │
   ▼
Airtable
Paciente + Turno pendiente
   │
   ▼
Análisis con IA
   │
   ▼
Gmail
Aprobación humana (HITL)
   │
   ├── RECHAZADO ──► Airtable ──► Telegram
   │
   └── APROBADO
          │
          ▼
     Google Calendar
          │
          ▼
       Airtable
          │
          ├──► Telegram
          │
          └──► Gmail
```

---

# ⚙️ Funcionamiento paso a paso

## 1️⃣ Inicio desde Telegram

El proceso comienza cuando el paciente interactúa con el bot de Telegram.

El workflow recibe mensajes y acciones realizadas mediante botones.

Antes de procesarlos se utiliza un filtro **Anti-Bucle**, encargado de evitar que mensajes generados por bots vuelvan a iniciar el flujo.

---

## 2️⃣ Selección de especialidad

El bot presenta un menú con las especialidades disponibles:

- 🩺 Clínica Médica
- 👶 Pediatría
- 🦷 Odontología

Cuando el usuario selecciona una opción, n8n obtiene:

- Especialidad seleccionada.
- Identificador del chat de Telegram.

Luego genera y envía el enlace correspondiente al formulario de solicitud de turno.

---

## 3️⃣ Formulario de solicitud

El paciente completa un formulario con la información necesaria para reservar el turno.

Entre los datos solicitados se encuentran:

- Nombre completo.
- DNI.
- Correo electrónico.
- Teléfono.
- Fecha de nacimiento.
- Domicilio.
- Especialidad.
- Fecha del turno.
- Horario.
- Motivo de consulta.

También se conserva internamente el `chat_id` de Telegram para poder continuar la comunicación con el paciente.

---

## 4️⃣ Validación y normalización

Una vez enviado el formulario, n8n valida automáticamente la información.

Entre las validaciones implementadas se encuentran:

- Nombre y apellido.
- DNI válido.
- Formato de correo electrónico.
- Número de teléfono.
- Especialidad permitida.
- Fecha de nacimiento.
- Edad válida.
- Fecha futura.
- Día laborable.
- Identificador de Telegram.

También se normalizan diferentes campos para mantener información consistente en la base de datos.

---

## ❌ Datos inválidos

Si alguna validación falla, el flujo no continúa con la reserva.

El sistema:

1. Registra el error en Airtable.
2. Guarda información sobre el motivo del error.
3. Envía un mensaje al paciente mediante Telegram.

Ejemplo:

```text
⚠️ No pudimos registrar tu turno:
El consultorio atiende de lunes a viernes.

Volvé a abrir el formulario desde el bot e intentalo de nuevo.
```

---

## 5️⃣ Consulta de disponibilidad

Cuando los datos son válidos, n8n consulta **Google Calendar** para comprobar si el horario solicitado está disponible.

El sistema evalúa si existe algún evento que se superponga con el turno solicitado.

### Horario ocupado

Si existe un conflicto, el paciente recibe una notificación por Telegram indicando que debe seleccionar otro horario.

### Horario disponible

Si el horario está libre, el workflow continúa con el registro del paciente y del turno.

---

## 6️⃣ Registro en Airtable

El proyecto utiliza Airtable como base de datos principal.

### 👤 Pacientes

El paciente se registra o actualiza mediante una operación **Upsert**, utilizando el DNI para identificar registros existentes.

Esto evita generar duplicados innecesarios.

### 📅 Turnos

Luego se crea el turno con información como:

- Paciente.
- Especialidad.
- Fecha.
- Horario.
- Motivo.
- Estado.
- Chat ID.
- Información generada por IA.
- Referencia del evento de Google Calendar.

Inicialmente el turno queda pendiente de aprobación.

---

# 🤖 Análisis mediante Inteligencia Artificial

Antes de enviar la solicitud al médico, el sistema utiliza IA para analizar la información ingresada.

La IA funciona como **asistente administrativo**, generando información estructurada para facilitar la revisión.

Entre los datos generados se encuentran:

```json
{
  "resumen": "Resumen administrativo de la solicitud",
  "prioridad": "Media",
  "alerta": "Sin alertas administrativas"
}
```

La prioridad puede clasificarse como:

- 🔴 Alta
- 🟡 Media
- 🟢 Baja

> **Importante:** la IA no realiza diagnósticos ni toma la decisión final sobre el turno.

La información generada se almacena posteriormente en Airtable.

---

# 👨‍⚕️ Human-in-the-Loop (HITL)

Una característica importante del proyecto es la implementación de **Human-in-the-Loop**.

Aunque gran parte del proceso está automatizado, la decisión final continúa en manos del médico.

El médico recibe un correo mediante Gmail con información como:

- Paciente.
- DNI.
- Datos de contacto.
- Especialidad.
- Fecha y horario.
- Motivo de consulta.
- Resumen generado por IA.
- Prioridad.
- Alertas.

El correo incluye dos acciones:

```text
❌ Rechazar

✅ Aprobar turno
```

El workflow queda a la espera de la decisión humana.

---

# ✅ Turno aprobado

Si el médico aprueba la solicitud:

1. n8n crea el evento en **Google Calendar**.
2. Airtable actualiza el estado del turno.
3. Se registra la información del evento.
4. Telegram envía una confirmación al paciente.
5. Gmail envía un correo de confirmación.

Ejemplo de mensaje:

```text
✅ ¡Turno confirmado!

📌 Clínica Médica
📅 2026-10-30 a las 14:00

Te esperamos.
```

De esta manera, el evento en Calendar se crea únicamente después de recibir la aprobación humana.

---

# ❌ Turno rechazado

Si el médico rechaza la solicitud:

1. Airtable actualiza el estado del turno.
2. El turno queda registrado como rechazado.
3. Telegram informa al paciente.
4. El paciente puede iniciar nuevamente el proceso.

---

# 🚨 Manejo de errores

El proyecto incluye un sistema de captura de errores globales.

```text
Error Trigger
      │
      ▼
Preparar Error Global
      │
      ▼
Airtable — Log de Error
```

Cuando ocurre un error inesperado en una ejecución, n8n captura información del fallo y la almacena en Airtable.

Esto permite mantener trazabilidad y facilita el diagnóstico de problemas.

---

# 🗃️ Estructura de datos

Airtable centraliza la información utilizada por el sistema.

Las principales tablas son:

### 👤 Pacientes

Contiene información personal y de contacto del paciente junto con su historial de turnos.

### 📅 Turnos

Almacena las solicitudes realizadas, especialidad, fecha, horario, estado y datos relacionados con la automatización.

### 🚨 Errores

Registra errores de validación y errores producidos durante las ejecuciones del workflow.

---

# 🛡️ Validaciones implementadas

El workflow contempla diferentes escenarios antes de confirmar una reserva:

```text
Formulario recibido
        │
        ▼
¿Datos válidos?
   │          │
   NO         SÍ
   │          │
   ▼          ▼
Error      ¿Horario libre?
              │
          ┌───┴───┐
          │       │
         NO       SÍ
          │       │
          ▼       ▼
      Informar   IA
                  │
                  ▼
                HITL
             ┌────┴────┐
             │         │
          Rechazar   Aprobar
```

Esto evita que una solicitud incorrecta llegue directamente a la agenda del consultorio.

---

# 📊 Estados del turno

Durante el proceso, una solicitud puede atravesar diferentes estados.

| Estado | Descripción |
|---|---|
| **Pendiente** | Solicitud registrada y esperando procesamiento/aprobación |
| **Aprobado** | Turno autorizado por el médico |
| **Rechazado** | Solicitud rechazada durante la revisión |

---

# 🧪 Casos probados

Durante el desarrollo se verificaron distintos escenarios:

- ✅ Solicitud válida.
- ✅ Selección de diferentes especialidades.
- ✅ Registro del paciente.
- ✅ Registro del turno.
- ✅ Consulta de disponibilidad.
- ✅ Aprobación mediante HITL.
- ✅ Creación automática del evento.
- ✅ Confirmación mediante Telegram.
- ✅ Confirmación mediante Gmail.
- ❌ Fecha correspondiente a un día no laborable.
- ❌ Datos inválidos.
- ❌ Horario ocupado.
- ❌ Rechazo del médico.
- 🚨 Registro de errores del workflow.

---

# 🔐 Consideraciones de seguridad

Las credenciales utilizadas por las integraciones deben almacenarse utilizando el sistema de credenciales de n8n.

No deben publicarse en el repositorio:

```text
API Keys
Tokens de Telegram
Credenciales de Airtable
Credenciales de Google
Credenciales de Gmail
Secrets
```

Los archivos exportados del workflow también deben revisarse antes de publicarlos para evitar exponer información sensible.

---

# 📂 Repositorio

El repositorio contiene la implementación y documentación correspondiente al proyecto de automatización.

**Orquestador:** n8n

**Workflow principal:**

```text
Consultorio Médico - Turnos con IA
(Telegram + Airtable + IA + HITL)
```

---

# 🚀 Resultado final

El sistema permite automatizar el proceso:

**Paciente → Telegram → Formulario → Validación → Disponibilidad → Airtable → IA → Aprobación humana → Calendar → Confirmación**

El proyecto demuestra cómo combinar **automatización, Inteligencia Artificial y supervisión humana** para resolver un proceso administrativo real manteniendo trazabilidad y control sobre las decisiones finales.

---

## 👨‍💻 Autor

**Gonzalo Emanuel Soto**

Proyecto desarrollado como entrega final de **AI Automation**.
