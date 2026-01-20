# Bot Usaditos

**Versión:** 2.0 (Simple - Sin Typebot)  
**Plataforma:** n8n + Chatwoot

## Descripción General

Este repositorio contiene el flujo de trabajo para un bot automatizado diseñado para calificar leads de autos usados. Su función principal es dar la bienvenida a los usuarios, capturar su nombre y entregarles un catálogo de vehículos. Además, cuenta con un sistema de seguimiento automático para reactivar clientes potenciales que no responden.

## Arquitectura

El sistema utiliza **n8n** para la lógica del flujo y **Chatwoot** para la gestión de la mensajería.
Funciona mediante una máquina de estados que decide la respuesta adecuada según el estado del usuario:

- **Webhook Chatwoot:** Recibe los mensajes entrantes.
- **Máquina de Estados:** Gestiona los estados `NEW`, `WAITING_NAME`, `COMPLETED`, y `BLOCKED`.
- **Sistema de Recordatorios:** Un Cron job verifica cada minuto los usuarios en espera para enviar recordatorios.

## Lógica de Conversación

### Estados del Usuario

| Estado         | Condición                       | Comportamiento                                                             |
| :------------- | :------------------------------ | :------------------------------------------------------------------------- |
| `NEW`          | Usuario escribe por primera vez | Envía bienvenida y solicita el nombre. Pasa a `WAITING_NAME`.              |
| `WAITING_NAME` | Bot espera el nombre            | Si responde: Guarda nombre, envía catálogo y pasa a `COMPLETED`.           |
| `COMPLETED`    | Flujo terminado                 | **Ventana de 7 días:** No responde automáticamente. Tras 7 días, reinicia. |
| `BLOCKED`      | Comando STOP ejecutado          | Ignora mensajes por 30 días.                                               |

### Secuencia de Recordatorios

Si el usuario permanece en `WAITING_NAME`, el bot envía seguimiento:

1.  **Recordatorio 1 (2 minutos):** Intento de reactivación.
2.  **Recordatorio 2 (5 minutos):** Envía el catálogo y marca como `COMPLETED`.

## Comandos de Control (Admin)

Estos comandos se pueden ejecutar enviando un mensaje directo al bot:

- `stop`: Pausa el bot para el usuario por 30 días.
- `restart`: Reinicia la sesión del usuario desde cero.
- `/sesion`: Muestra el estado actual y datos de la sesión.
- `/stats`: Muestra estadísticas globales.
- `/help`: Muestra la lista de comandos disponibles.

## Configuración e Instalación

1.  **Importar Flujo:** Importa el archivo `Bot Usaditos - Simple.json` en tu instancia de n8n.
2.  **Configurar Webhook:** Conecta el webhook de Chatwoot al nodo correspondiente en n8n.
3.  **Ajustar Tiempos (Producción):**

    Por defecto, el bot está en **MODO PRUEBA** (recordatorios a los 2 y 5 minutos). Para producción, edita el nodo **"Verificar Sesiones"**:

    ```javascript
    // Modo Producción
    const REMINDER_1_TIME = 60 * 60 * 1000; // 1 hora
    const REMINDER_2_TIME = 24 * 60 * 60 * 1000; // 24 horas
    ```

## Mantenimiento

- Usa `restart` para resetear usuarios durante pruebas.
- Revisa `Executions` en n8n para logs detallados.
