# Agregar Event Hub a la Integración

> **Contexto:** Algunos complementos de Azure, como la recolección de logs de XDR, no permiten una integración directa con Cribl. En estos casos, se utiliza un **Event Hub** como intermediario. Event Hub recopila los logs que no se pueden enviar directamente y, posteriormente, los reenvía a Microsoft Sentinel (a través de un Log Analytics Workspace - LAW).

Crear un Event Hub en Azure es un proceso de dos partes: primero creas el **Namespace** (un contenedor para tus flujos) y luego creas el **Event Hub** (el flujo específico) dentro de ese namespace.

Aquí están los pasos detallados desde el portal de Azure.

---

## 📦 1. Crear el "Event Hubs Namespace" (Contenedor)

Este es el primer recurso que debes crear. Actúa como un contenedor de administración para uno o más Event Hubs.

1.  Inicia sesión en el **Portal de Azure**.
2.  En la barra de búsqueda superior, escribe `"Event Hubs"` y selecciona el servicio "Event Hubs" de la lista.
3.  En la página de Event Hubs, haz clic en el botón `+ Crear`.
4.  Rellena la pestaña **"Aspectos básicos"**:
    * **Suscripción:** Elige la suscripción donde quieres que se facture.
    * **Grupo de recursos:** Selecciona uno existente o haz clic en "Crear nuevo" para crear uno (ej. `rg-logging`).
    * **Nombre del namespace:** Dale un nombre **globalmente único** (ej. `mi-namespace-logs-prod`). Verás una marca de verificación verde si está disponible.
    * **Ubicación:** Elige la región de Azure donde quieres desplegarlo (ej. `East US 2`).
    * **Nivel de precios:** Se recomienda **Estándar (Standard)** para la ingesta de logs y la integración con servicios como Sentinel o Cribl. El nivel Básico es muy limitado.
    * **Unidades de procesamiento (Opcional):** Déjalo en `1` por ahora. Puedes aumentarlo más tarde si necesitas más capacidad.
5.  Haz clic en `Revisar + crear`.
6.  Azure validará la configuración. Una vez validado, haz clic en `Crear`.
7.  Espera a que termine la implementación (puede tardar unos minutos).

---

## 🌊 2. Crear el "Event Hub" (El Flujo de Datos)

Ahora que tienes el contenedor (Namespace), puedes crear el flujo de datos específico dentro de él.

1.  Una vez implementado el Namespace, haz clic en **"Ir al recurso"** (o búscalo por su nombre).
2.  En el menú de la izquierda de tu Namespace, busca la sección "Entidades" y haz clic en `Event Hubs`.
3.  En la parte superior, haz clic en `+ Event Hub`.
4.  Rellena los datos:
    * **Nombre:** Este es el nombre de tu flujo o "topic" específico (ej. `checkpoint-logs`, `fortinet-logs`, `xdr-events`).
    * **Recuento de particiones:** Para la mayoría de los casos de ingesta de logs, `2` o `4` es un buen comienzo.
    * **Retención de mensajes:** Por defecto es `1` día. Puedes aumentarlo hasta `7` días en el nivel Estándar.
    * **Capture (Opcional):** Actívalo si quieres que Azure guarde automáticamente una copia de todos los logs en un Azure Storage Account. Es muy útil para archivado a largo plazo.
5.  Haz clic en `Crear`.

¡Listo! Tu Event Hub ya está creado y listo para recibir datos.

---

## 🔑 3. (Paso Clave) Obtener las Claves de Conexión

Ahora necesitas los permisos para que una aplicación (como Cribl, un script de Python o Logstash) pueda *enviar* datos a este hub.

1.  Regresa a la página principal de tu **Namespace** (el contenedor, no el hub individual).
2.  En el menú de la izquierda, en "Configuración", haz clic en `Directivas de acceso compartido` (Shared access policies).

Aquí tienes dos opciones:

> **Opción A (Rápida, menos segura):**
> Puedes usar la directiva por defecto llamada `RootManageSharedAccessKey`. Esta tiene permisos para *todo* (enviar, escuchar, administrar). **No es recomendada para entornos de producción.**

> **Opción B (Recomendada y más segura):**
> Crea una nueva directiva solo con los permisos necesarios.
> 1.  Haz clic en `+ Agregar`.
> 2.  **Nombre de la directiva:** `policy-send-only` (o el nombre de tu app, ej. `cribl-sender`).
> 3.  Marca **solamente** la casilla `Enviar` (Send).
> 4.  Haz clic en `Crear`.

### Obtener la cadena de conexión

1.  Haz clic en el **nombre de la directiva** que has decidido usar (ya sea `RootManageSharedAccessKey` o la nueva que creaste).
2.  Verás varias claves. La que casi siempre necesitarás es la **"Cadena de conexión: clave principal"** (Connection string–primary key).
3.  Haz clic en el icono de copiar junto a esa cadena.

Esa cadena de conexión es la que usarás en tu aplicación externa para autenticarte y enviar eventos a tu nuevo Event Hub.
