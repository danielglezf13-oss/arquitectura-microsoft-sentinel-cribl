# Configuración de Cribl Stream para Microsoft Sentinel

Esta guía detalla el proceso para configurar Cribl.Cloud Stream para ingestar logs de Syslog (enviados por IP) y procesarlos para su envío a Microsoft Sentinel.

El proceso implica instalar el "Pack" de Sentinel desde el Dispensary y, fundamentalmente, re-configurar la fuente (Source) de Syslog para que se ejecute *dentro* del contexto de ese Pack.

> **Objetivo:** Ingestar Syslog de dispositivos y enviarlo a Sentinel.
> **Plataforma:** Cribl.Cloud

---

## 1. Configuración Inicial de la Cuenta

1.  Inicie sesión o cree una cuenta en `cribl.cloud`.
2.  Una vez en el portal, navegue a la sección **"Access"** (Acceso).
3.  Localice el apartado **"Ingress IPs"** (IPs de Ingreso).
4.  **Copie la dirección IP** que se muestra. Esta es la IP pública de su instancia de Cribl.Cloud a la cual deberá apuntar sus dispositivos (Firewalls, routers, etc.) para que envíen sus logs de Syslog.

## 2. Instalar el Pack de Microsoft Sentinel

Para que Cribl sepa cómo formatear y enviar datos a Sentinel, primero debemos instalar el Pack correspondiente.

1.  Desde el menú principal, navegue a **"Products"** > **"Stream"**.
2.  Seleccione **"Worker Groups"** y haga clic en el grupo por defecto (usualmente llamado `default`).
3.  En el menú de configuración del Worker Group, vaya a **"Processing"** > **"Packs"**.
4.  Haga clic en el botón **"Add Pack"** (Agregar Pack).
5.  Seleccione la opción **"Add from Dispensary"** (Agregar desde el Dispensary).
6.  En la barra de búsqueda, escriba **"sentinel"**.
7.  Localice el pack llamado **"Microsoft Sentinel Syslog"** y haga clic en él.
8.  Haga clic en **"Add Pack"** para instalarlo en su Worker Group.
9.  Cierre las ventanas emergentes. Ahora debería ver el pack "Microsoft Sentinel Syslog" en su lista de packs.

## 3. Configuración de la Fuente (Source) de Syslog

Este es el paso más crítico. Por defecto, Cribl tiene una fuente de Syslog "global" en el Worker Group. Para que los logs sean procesados por el Pack de Sentinel, debemos deshabilitar la fuente global y crear una réplica exacta *dentro* del Pack de Sentinel.

### A. Deshabilitar las Fuentes de Syslog Globales

> **⚠️ ¡Aviso Importante!**
> Por política de Cribl, no pueden existir dos fuentes escuchando en el mismo puerto (ej. `514`). Antes de crear las nuevas fuentes en el Pack, **debe deshabilitar** las fuentes de Syslog que vienen activas por defecto en el Worker Group.

1.  Navegue a la configuración del Worker Group `default`.
2.  Vaya a **"Data"** > **"Sources"** (Fuentes).
3.  Busque y seleccione **"Syslog"**.
4.  Verá las configuraciones activas (ej. `in_syslog_default`, `in_syslog_udp_default`, etc.).
5.  **Desactive el interruptor (toggle)** de cada una de estas fuentes para apagarlas.

### B. Replicar las Fuentes de Syslog dentro del Pack

Ahora, crearemos copias de esas fuentes, pero dentro del Pack de Sentinel.

1.  Regrese a **"Processing"** > **"Packs"**.
2.  Haga clic en el nombre del pack **"Microsoft Sentinel Syslog"** (el texto azul) para entrar en su configuración interna.
3.  Dentro del pack, vaya a **"Data"** > **"Sources"**.
4.  Busque y seleccione **"Syslog"**.
5.  Haga clic en **"Add Source"** (Agregar Fuente).
6.  **Replique la configuración** que acaba de desactivar. Generalmente, esto implica crear dos fuentes:
    * **Fuente TCP:**
        * **Input ID:** `in_syslog_tcp` (o un nombre descriptivo).
        * **Address:** `0.0.0.0`
        * **Port:** `514` (o el puerto que estuviera usando la fuente global).
        * Asegúrese de que el **Transport** esté en `TCP`.
    * **Fuente UDP:**
        * **Input ID:** `in_syslog_udp`
        * **Address:** `0.0.0.0`
        * **Port:** `514` (o el puerto que estuviera usando la fuente global).
        * Asegúrese de que el **Transport** esté en `UDP`.

7.  Guarde y **active (Enable)** ambas fuentes nuevas *dentro* del pack.

---

## 4. Conclusión

Al completar estos pasos, ha logrado que:

1.  Las fuentes de Syslog globales (que no procesaban para Sentinel) estén apagadas.
2.  Las nuevas fuentes de Syslog (TCP y UDP) estén activas *dentro* del Pack de Sentinel.

Ahora, todos los logs que lleguen a su IP de ingreso de Cribl en el puerto `514` (o el que haya configurado) serán capturados directamente por el Pack de "Microsoft Sentinel Syslog", asegurando que sean procesados y enrutados correctamente hacia Azure.

