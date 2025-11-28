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

# Integración de Alertas de Microsoft Sentinel con Telegram

Este proyecto documenta la implementación de un sistema de notificaciones automatizadas para un Centro de Operaciones de Seguridad (CSOC). Utilizando **Azure Logic Apps** y la API de **Telegram**, este flujo de trabajo permite recibir alertas de incidentes de seguridad críticos directamente en un grupo de mensajería en tiempo real.

## 📋 Prerrequisitos

* Una cuenta de **Telegram**.
* Suscripción activa de **Microsoft Azure**.
* Un Workspace de **Microsoft Sentinel** configurado.
* Permisos adecuados para crear recursos y asignar roles (Owner o User Access Administrator).

---

## 🚀 Guía de Implementación

### Paso 1: Configuración del Bot de Telegram

Antes de configurar Azure, necesitamos crear el "buzón" de entrega en Telegram.

1.  **Crear el Bot:**
    * Abre Telegram y busca el usuario **@BotFather**.
    * Envía el comando `/newbot`.
    * Asigna un nombre descriptivo (ej. `SentinelAlerts`) y un nombre de usuario único que termine en "bot" (ej. `Sentinel_Corp_Bot`).
    * 🔴 **Importante:** BotFather te entregará un **HTTP API Token**. Guárdalo en un lugar seguro; esta es la llave de tu bot.
    * *Nota:* Si BotFather muestra un botón de "Iniciar", úsalo para desplegar el menú.

2.  **Obtener el Chat ID:**
    * Crea un nuevo grupo en Telegram donde desees recibir las alertas y agrega al bot que acabas de crear.
    * Envía un mensaje de prueba al grupo (ej. "Hola").
    * Abre tu navegador y visita la siguiente URL (reemplaza `<TU_TOKEN>` con el token del paso anterior):
        ```
        [https://api.telegram.org/bot](https://api.telegram.org/bot)<TU_TOKEN>/getUpdates
        ```
    * Busca en el texto resultante la sección `"chat": { "id": -123456789 }`.
    * Copia el número completo (incluyendo el signo menos si es un grupo). Este es tu **Chat ID**.

---

### Paso 2: Crear la Logic App en Azure

1.  En el portal de Azure, busca **Logic Apps** y haz clic en **Crear**.
2.  Configura los detalles básicos:
    * **Tipo de plan:** Consumo (Consumption). *Ideal para pagar solo por ejecución.*
    * **Grupo de recursos:** Selecciona el mismo donde reside tu Sentinel (recomendado).
    * **Nombre:** Ej. `Sentinel-To-Telegram`.
    * **Región:** La misma región de tu grupo de recursos.
    * **Log Analytics:** No habilitar (no es necesario para este conector).
3.  Revisa y crea el recurso.
4.  *Opcional:* Asigna etiquetas (tags) si lo requieres para organizar tus recursos.

---

### Paso 3: Configurar Permisos (Identidad Administrada)

Para que Sentinel pueda ejecutar esta Logic App automáticamente, necesitamos otorgarle permisos explícitos.

1.  Ve al **Grupo de Recursos** donde creaste la Logic App.
2.  Entra a **Control de acceso (IAM)** > **Agregar asignación de roles**.
3.  Busca y selecciona el rol: **Microsoft Sentinel Automation Contributor** (Colaborador de automatización de Microsoft Sentinel).
4.  Haz clic en Siguiente y ve a la pestaña **Miembros**.
5.  Selecciona **Asignar acceso a:** "Usuario, grupo o entidad de servicio".
6.  En "Seleccionar miembros", busca exactamente: **Azure Security Insights**.
    > ⚠️ **Truco:** Si no aparece como "Azure Security Insights", busca "Security Insights". Esta es la identidad de servicio de Sentinel.
7.  Finaliza la asignación del rol.

---

### Paso 4: Diseño del Flujo de Trabajo (Playbook)

1.  Ve a tu recurso **Logic App** > **Diseñador de aplicaciones lógicas**.
2.  Inicia con una "Blank Logic App" (Aplicación lógica en blanco).

#### A. El Disparador (Trigger)
1.  Busca **Microsoft Sentinel**.
2.  Selecciona el disparador: **When Microsoft Sentinel incident creation rule was triggered** (Cuando se desencadena una regla de creación de incidentes).
    * *Asegúrate de elegir "Incident", no "Alert".*
3.  Si pide conectar, inicia sesión con tu cuenta de Azure.

#### B. La Acción (Enviar a Telegram)
1.  Debajo del disparador, haz clic en **+ Nuevo paso**.
2.  Busca **HTTP** y selecciona la acción genérica llamada **HTTP** (icono de mundo verde).
3.  Configura los parámetros:
    * **Método:** `POST`
    * **URI:** `https://api.telegram.org/bot<TU_NUEVO_TOKEN>/sendMessage`
        *(Pega tu token sin espacios).*
    * **Encabezados (Headers):**
        * Clave: `Content-Type`
        * Valor: `application/json`
    * **Cuerpo (Body):** Copia y pega el siguiente JSON.

```json
{
  "chat_id": "<TU_CHAT_ID>",
  "text": "🚨 **ALERTA DE SEGURIDAD SENTINEL** 🚨\n\nTitulo: @{triggerBody()?['object']?['properties']?['title']}\nSeveridad: @{triggerBody()?['object']?['properties']?['severity']}\nDescripcion: @{triggerBody()?['object']?['properties']?['description']}\n\nEnlace: @{triggerBody()?['object']?['properties']?['incidentUrl']}"
}
```

> **⚙️ Configuración de Contenido Dinámico:**
> 1. Reemplaza `<TU_CHAT_ID>` con tu ID numérico.
> 2. Si al pegar el código las expresiones `@{triggerBody...}` se ven como texto plano y no como "fichas" de colores:
>    * Borra el texto de la expresión (ej. `@{triggerBody()...}`).
>    * Haz clic en el espacio vacío.
>    * En la ventana emergente de **Contenido Dinámico** (rayo azul), busca y selecciona los campos correspondientes: *Incident Title*, *Incident Severity*, *Incident Description*, etc.

4.  Guarda la Logic App.

---

### Paso 5: Regla de Automatización en Sentinel

El último paso es decirle a Sentinel cuándo ejecutar este Playbook.

1.  Ve a **Microsoft Sentinel** > **Automatización**.
2.  Crea una nueva **Regla de automatización**.
3.  **Configuración:**
    * **Nombre:** Ej. `Auto-Alert-Telegram-High`.
    * **Desencadenador:** When incident is created.
    * **Condiciones:**
        * Haz clic en `+ Add` > `Condition (And)`.
        * Selecciona `Severity` > `Equals`.
        * Marca las casillas `High` y `Medium` (o según tu criterio).
    * **Acciones:**
        * Selecciona `Run Playbook`.
        * En el menú desplegable, busca y selecciona `Sentinel-To-Telegram`.
4.  Haz clic en **Aplicar**.

---

## ✅ Pruebas

Para verificar el funcionamiento:
1.  Dentro del diseñador de la Logic App, haz clic en **Run Trigger** > **Run**.
2.  Si la configuración es correcta, recibirás el mensaje formateado en tu grupo de Telegram inmediatamente.
3.  Alternativamente, genera un incidente de prueba en Sentinel que cumpla con las condiciones de severidad para ver la automatización completa.

---
*Documentación creada por Daniel - Ingeniero en Ciberseguridad.*
