# Sentinel to Telegram Alert Bridge 🛡️📲

> **Orchestrating real-time incident notification workflows from Azure Sentinel to Telegram using Logic Apps automation.**

---

## 📋 Overview
This repository contains the deployment guide and Logic App code to enable a **Security Operations Center (SOC)** to receive instant alerts on mobile devices. By integrating the Telegram API with Microsoft Sentinel, we reduce the **MTTD (Mean Time to Detect)** and ensure critical incidents are never missed.

### 🔧 Tech Stack
* **SIEM:** Microsoft Sentinel
* **Automation:** Azure Logic Apps
* **Notification Endpoint:** Telegram Bot API
---

Primeros pasos, crear el bot de telegram
1. Crear el Bot:

Abre Telegram y busca el usuario @BotFather.

Envía el comando /newbot.

Asigna un nombre (ej. SentinelAlerts) y un usuario (ej. Sentinel_Corp_Bot).

BotFather te dará un Token de acceso (HTTP API Token). Guárdalo, es la llave de tu bot.
Nota: Cada actualización de telegran agrega o cambia la forma de extraer los diferentes tokens, esta version es del 2025

Boot father algunas veces se debe iniciar, por lo regular existe un boton de inicio, y semuestra un menu donde mediante botones puedes generar el token de acceso


2. Obtener el Chat ID (Identificador del chat):

Crea un grupo en Telegram donde quieras recibir las alertas y agrega a tu nuevo bot.

Envía un mensaje cualquiera al grupo (ej. "Hola").

Abre tu navegador y visita esta URL (reemplaza <TU_TOKEN> con el que te dio BotFather): https://api.telegram.org/bot<TU_TOKEN>/getUpdates

Busca en el texto resultante la sección "chat": { "id": -123456789 }.

Copia ese número (incluyendo el signo menos si es un grupo). Ese es tu Chat ID.


Paso 2: Crear la Logic App en Azure
Ve al portal de Azure y busca Logic Apps.

Haz clic en Crear (Agregar).

Configuración básica:

Tipo de plan: Selecciona Consumo (Consumption) para pagar solo por ejecución (es lo más barato y efectivo para esto).

Suscripción: Tu suscripción actual.

Grupo de recursos: Preferiblemente el mismo donde está tu Sentinel.

Nombre: Ej. Sentinel-To-Telegram.
Región: De preferencia la misma donde se encuentra tu grupo de recursos

y no se habilite Log Analytics, denido a que no se realizaran acciones relacionadas con este conector

Crea el recurso y ve a él cuando esté listo.

Nota: Si crees necesario asignale etiquetas

Paso 3: Configurar Permisos y Automatización en Sentinel
Para que Sentinel pueda "disparar" esta Logic App, necesita permisos.

Asignar Rol a Sentinel:

Ve a tu grupo de recursos donde está la Logic App.

Ve a Control de acceso (IAM) -> Agregar asignación de roles.

Busca el rol Microsoft Sentinel Automation Contributor (Colaborador de automatización de Microsoft Sentinel).

Dale a Siguiente y ve a la pestaña Miembros.

Haz clic en Seleccionar miembros.

⚠️ Aquí está el truco: En la barra de búsqueda, en lugar de "Sentinel", escribe: Azure Security Insights

Si aparece una aplicación con ese nombre (o "Security Insights"), selecciónala. Ese es Sentinel.

Termina de asignar el rol.

paso 4:

Ve a Microsoft Sentinel -> Automatización.

Pestaña Cuadernos de estrategias activos (Active playbooks).

Haz clic en Crear -> Cuaderno de estrategias con desencadenador de incidentes.

Elige tu suscripción y el grupo de recursos rg-demo-xdf-uin.

Ponle un nombre nuevo (ej: Telegram-Alert-V2).

En el apartado de conexiones, se selecciona Conectar con la identidad administrada, y por ultimo revisar y crear

una vez creada, se nos mostrara el playbook creado en la lista de automatizacion de sentinel, Esto te abrirá el diseñador de Logic Apps automáticamente con el disparador ya configurado correctamente.

paso 5.

Se mostrara un flujo de trabajo con el logo de sentinel, Debajo del disparador, haz clic en + Nuevo paso (New step).

Busca HTTP (el icono es un mundo verde).

Selecciona la acción llamada simplemente HTTP.

Configúralo exactamente así:

Método: POST

URI: https://api.telegram.org/bot<TU_NUEVO_TOKEN>/sendMessage (Recuerda: pega tu token nuevo sin espacios y sin la barra extra).

Encabezados (Headers):

Escribe Content-Type en la izquierda.

Escribe application/json en la derecha.

Cuerpo (Body): Copia y pega este código:

{
  "chat_id": "<TU_CHAT_ID>",
  "text": "🚨 **ALERTA DE SEGURIDAD SENTINEL** 🚨\n\nTitulo: @{triggerBody()?['object']?['properties']?['title']}\nSeveridad: @{triggerBody()?['object']?['properties']?['severity']}\nDescripcion: @{triggerBody()?['object']?['properties']?['description']}\n\nEnlace: @{triggerBody()?['object']?['properties']?['incidentUrl']}"
}

Nota: Reemplaza <TU_CHAT_ID> por el número de tu chat. Si al pegar el código, las partes que dicen @{triggerBody...} se ven como texto simple (no como fichas de colores), bórralas y selecciónalas de la lista de Contenido Dinámico que aparece a la derecha (busca "Incident Title", "Incident Severity", etc.).

Dale a Guardar (arriba a la izquierda).

paso 6.

¡Excelente! Ese era el paso más difícil. Al asignar el rol a Azure Security Insights, ya le diste permiso a Sentinel para que "mande" a ejecutar tu Logic App.

Ahora vamos a configurar lo que hace la Logic App por dentro.

Siguientes pasos: Diseñar el Flujo
Ve a tu recurso Logic App que creaste.

En el menú de la izquierda, busca Diseñador de aplicaciones lógicas (Logic app designer).

Si te sale un video o bienvenida, ciérralo o dale a "Blank Logic App" (Aplicación lógica en blanco).

Paso A: El Disparador (Trigger)
En la barra de búsqueda escribe Sentinel.

Selecciona el icono azul de Microsoft Sentinel.

Busca y selecciona el disparador: When Microsoft Sentinel incident creation rule was triggered (Cuando se desencadena una regla de creación de incidentes).

Ojo: Asegúrate de elegir el que dice "Incident", no "Alert".

Si te pide conectar, dale a "Sign in" con tu cuenta.

Paso B: La Acción (Enviar a Telegram)
Debajo del disparador, haz clic en + Nuevo paso (New step).

Busca HTTP (el icono es un mundo verde).

Selecciona la acción llamada simplemente HTTP.

Configúralo exactamente así:

Método: POST

URI: https://api.telegram.org/bot<TU_NUEVO_TOKEN>/sendMessage (Recuerda: pega tu token nuevo sin espacios y sin la barra extra).

Encabezados (Headers):

Escribe Content-Type en la izquierda.

Escribe application/json en la derecha.

Cuerpo (Body): Copia y pega este código:

JSON

{
  "chat_id": "<TU_CHAT_ID>",
  "text": "🚨 **ALERTA DE SEGURIDAD SENTINEL** 🚨\n\nTitulo: @{triggerBody()?['object']?['properties']?['title']}\nSeveridad: @{triggerBody()?['object']?['properties']?['severity']}\nDescripcion: @{triggerBody()?['object']?['properties']?['description']}\n\nEnlace: @{triggerBody()?['object']?['properties']?['incidentUrl']}"
}
Nota: Reemplaza <TU_CHAT_ID> por el número de tu chat. Si al pegar el código, las partes que dicen @{triggerBody...} se ven como texto simple (no como fichas de colores), bórralas y selecciónalas de la lista de Contenido Dinámico que aparece a la derecha (busca "Incident Title", "Incident Severity", etc.).

Dale a Guardar (arriba a la izquierda).

Último Paso: Conectar en Sentinel
Ahora que la Logic App está lista y tiene permisos:

Ve a Microsoft Sentinel -> Automatización.

Crea una Regla de automatización.

en nombre, puedes poner el que sea de tu agrado, 

en trigger, selecciona: "when incident is created

conditions: da click en "+ add", selecciona "Condition (And)", en choose condition da click y selecciona severity, en el segundo recuadro da click y selecciona "equals",
y en eltercer valor selecciona las casillas de "High y Medium"

en Actions: daclick, selecciona run playbook seguido de esto semostrara una simbolo de carga debajo, y despues un espacio en blsnco, damos click y al desplegarde debe de
mostrarse el nombre del playbook ante screado, lo selecionamos y damos click en aplicar


Nota: Si necesitas probar tu logic app, es necesario dar estar dentro del editor de flujo y dar clic en el boton "run" seguido de "run" de nuevo y si esta bien configurada tu logic app en el grupo de telegram se mostraran los mensajes de prueba

y asi se realiza una integración delogic apps con sentinel y telegram
