# Documentación Técnica: Bot Usaditos

**Versión:** 2.0 (Simple - Sin Typebot)
**Archivo:** `Bot Usaditos - Simple.json`
**Plataforma:** n8n + Chatwoot

---

## 1. Descripción General

Bot automatizado para calificar leads de autos usados. Su función es dar la bienvenida, capturar el nombre del cliente y entregar el catálogo de vehículos. Si el cliente no responde, el bot realiza un seguimiento automático para intentar reactivarlo.

---

## 2. Arquitectura del Flujo

### A. Recepción de Mensajes (Webhook)

1. **Webhook Chatwoot:** Recibe el mensaje entrante.
2. **Máquina de Estados:** Decide qué responder según el estado del usuario (`NEW`, `WAITING_NAME`, `COMPLETED`, `BLOCKED`).
3. **Respuesta:** Envia el mensaje correspondiente a Chatwoot.

### B. Sistema de Recordatorios (Cron)

1. **Trigger:** Se ejecuta **cada 1 minuto**.
   > ⚠️ **Importante:** La frecuencia de 1 minuto es crítica para que los tiempos de 2m y 5m funcionen con precisión.
2. **Verificación:** Revisa usuarios en estado `WAITING_NAME`.
3. **Acción:** Envía recordatorios si se cumplen los tiempos de espera.

---

## 3. Lógica de Conversación

### Estados del Usuario

| Estado         | Condición                       | Comportamiento                                                                                          |
| -------------- | ------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `NEW`          | Usuario escribe por primera vez | Bot envía bienvenida y pide nombre. Pasa a `WAITING_NAME`.                                              |
| `WAITING_NAME` | Bot espera el nombre            | Si usuario responde: Guarda nombre, envía catálogo y pasa a `COMPLETED`.                                |
| `COMPLETED`    | Flujo terminado                 | **Ventana de 7 días:** El bot NO responde automáticamente. Si pasaron >7 días, reinicia con bienvenida. |
| `BLOCKED`      | Usuario ejecutó comando STOP    | Bot ignora mensajes por 30 días.                                                                        |

### Secuencia de Recordatorios (Si no responde)

El bot enviará hasta 2 mensajes de seguimiento si el usuario se queda en `WAITING_NAME`:

1. **Recordatorio 1 (2 minutos):** _"¿Sigues ahí? Tenemos autos usados disponibles..."_
2. **Recordatorio 2 (5 minutos):** _"Comprendo... Te dejo nuestro catálogo..."_
   - → **Acción:** Envía catálogo y marca como `COMPLETED` (inicia ventana de 7 días).

---

## 4. Comandos de Control (Admin)

Disponibles enviando el mensaje directo al bot:

| Comando   | Acción                                                                          |
| --------- | ------------------------------------------------------------------------------- |
| `stop`    | **Pausa el bot** para ese usuario por 30 días (ideal para intervención humana). |
| `restart` | **Reinicia la sesión** del usuario desde cero (borra estado y memoria).         |
| `/sesion` | Muestra el estado actual, tiempos y flags de la sesión del usuario.             |
| `/stats`  | Muestra estadísticas globales (total sesiones, en espera, completados).         |
| `/help`   | Muestra esta lista de comandos.                                                 |

---

## 5. Configuración para Producción

Actualmente configurado en **MODO PRUEBA**.

Para pasar a producción, editar el nodo **"Verificar Sesiones"**:

```javascript
// CAMBIAR ESTAS VARIABLES:

// Modo Prueba
const REMINDER_1_TIME = 2 * 60 * 1000; // 2 minutos
const REMINDER_2_TIME = 5 * 60 * 1000; // 5 minutos

// Modo Producción (Recomendado)
// const REMINDER_1_TIME = 60 * 60 * 1000;      // 1 hora
// const REMINDER_2_TIME = 24 * 60 * 60 * 1000; // 24 horas
```

---

## 6. Mantenimiento

- **Resetear usuario:** Usa el comando `restart`.
- **Verificar fallos:** Usa el comando `/sesion` para ver si el bot detecta al usuario.
- **Historial:** Revisa las "Executions" en n8n para ver logs detallados.
