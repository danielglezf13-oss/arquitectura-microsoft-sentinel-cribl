# Configuración del Destino (Destination) de Cribl a Sentinel

Esta guía detalla el paso final: configurar el "Destino" (Destination) dentro del Pack de Sentinel en Cribl para enviar los logs procesados a su Log Analytics Workspace usando la DCR y la App Registration.

---

## ✅ 1. Verificación Previa: Confirmar Ingesta de Datos

Antes de configurar el *destino*, debe confirmar que los logs están *llegando* correctamente a su Pack de Syslog.

1.  Asegúrese de que su dispositivo (ej. un Fortinet) esté configurado para enviar logs de Syslog a la IP de ingreso (Ingress IP) de su instancia de Cribl.Cloud (obtenida en la guía anterior).
2.  Dentro de su Pack de Sentinel, vaya a **Data** > **Sources** > **Syslog**.
3.  Haga clic en una de sus fuentes activas (ej. `in_syslog_udp`).
4.  Vaya a la pestaña **Live Data**.
5.  Debería ver eventos de log llegando en tiempo real. Si no ve datos aquí, resuelva la ingesta antes de continuar.

---

## ⚙️ 2. Crear el Destino de Microsoft Sentinel

1.  Dentro de su Pack de **Microsoft Sentinel Syslog**, navegue a **Data** > **Destinations**.
2.  Busque y seleccione **Microsoft Sentinel**.
3.  Haga clic en el botón **"Add"** (o "New Destination") para crear una nueva configuración.

## ✏️ 3. Configuración General (General Settings)

Complete los detalles básicos del destino:

* **Output ID:** Asigne un nombre único. (ej. `out_sentinel_syslog`). **Nota:** Los espacios deben ser reemplazados por guiones bajos (`_`).
* **Description:** (Opcional) Añada una descripción clara (ej. "Envío de Syslog a LAW principal").
* **Endpoint configuration:** Seleccione la opción **"ID"**.

Al seleccionar "ID", aparecerán 3 campos nuevos. Debe obtener esta información desde el Portal de Azure:

| Campo en Cribl | Valor de Azure | Dónde Encontrarlo |
| :--- | :--- | :--- |
| **Data Collection Endpoint** | `URI de ingesta de registros` | Vaya a su **Data Collection Endpoint (DCE)** -> `Información general (Overview)` -> Copie el **URI de ingesta de registros**. |
| **Data Collection Rule ID** | `ID inmutable` | Vaya a su **Data Collection Rule (DCR)** -> `Información general (Overview)` -> Copie el **ID inmutable**. |
| **Stream Name** | `Nombre de la Tabla` | Escriba el nombre del "Stream" (flujo) definido en su DCR. Para logs de Syslog, este es comúnmente **`Custom-Syslog`**. |

---

## 🔐 4. Configuración de Autenticación (Authentication)

Esta es la sección más importante. Conecta Cribl con la App Registration que creó en Azure.

| Campo en Cribl | Valor de Azure | Dónde Encontrarlo |
| :--- | :--- | :--- |
| **Login URL** | `Punto de conexión de token de OAuth 2.0 (v2)` | Vaya a `Registros de aplicaciones` -> Seleccione su App -> `Puntos de conexión (Endpoints)` -> Copie el **Punto de conexión de token de OAuth 2.0 (v2)**. |
| **OAuth Secret (Value)** | `Valor del Secreto` | Pegue el **Valor** del Secreto de Cliente que generó y guardó. (Este es el secreto en sí, no el "ID del Secreto"). |
| **Client ID** | `ID de aplicación (cliente)` | Vaya a `Registros de aplicaciones` -> Seleccione su App -> `Información general (Overview)` -> Copie el **ID de aplicación (cliente)**. |

---

## 🔬 5. Guardar, Probar y Desplegar

### A. Guardar y Probar la Conexión

1.  Haga clic en **"Save"** en la parte inferior.
2.  Cribl intentará autenticarse. **Nota:** La autenticación y el emparejamiento inicial pueden tardar varios minutos (a veces hasta 30). Es normal ver errores de "rojo" a "verde" durante este tiempo.
3.  Una vez que el estado parezca estable (verde), vaya a la pestaña **"Test"** dentro de la configuración del Destino.
4.  Haga clic en **"Run Test"**. Un mensaje de `"Successfull"` (Exitoso) indica que la implementación fue un éxito.

> **Solución de Problemas (Troubleshooting):**
> Si la prueba falla o si usa "Select Sample" y los datos no llegan a Sentinel, es posible que falte el conector de datos base en Sentinel.
> 1. En Azure, vaya a **Microsoft Sentinel** -> **Configuración** -> **Conectores de datos**.
> 2. Haga clic en **"Content Hub"**.
> 3. Busque **"Syslog"** e instale la solución de **"Syslog"** (la que tiene el logo de Sentinel).
> 4. Espere unos minutos y vuelva a ejecutar la prueba en Cribl.

### B. Activar el Enrutamiento (Routes)

Ahora que el destino funciona, debe decirle a Cribl que envíe datos a él.

1.  Salga de la configuración del Destino y vuelva a la configuración principal del Pack de Sentinel.
2.  Vaya a **Processing** > **Routes**.
3.  Verá una lista de rutas (probablemente solo una, la `default`). Haga clic en ella para expandirla.
4.  Busque el campo **"Destination"** (o "Output").
5.  Haga clic en el menú desplegable y seleccione el destino que acaba de crear (ej. `out_sentinel_syslog`).
6.  Haga clic en **"Save"**.

### C. Confirmar y Desplegar (Commit & Deploy)

> **¡Paso Crítico!**
> Después de CUALQUIER cambio en la configuración de Cribl (fuentes, rutas, destinos), debe desplegar los cambios.
>
> 1.  Haga clic en el botón **"Commit & Deploy"** en la esquina superior derecha de la interfaz de Cribl.
> 2.  Escriba un mensaje de confirmación (ej. "Ruta de Sentinel activada") y confirme el despliegue.

---

## 🏁 6. Verificación Final

### En Cribl
1.  Vaya a su Destino (`Data` > `Destinations` > `out_sentinel_syslog`).
2.  Navegue a la pestaña **"Charts"**.
3.  Después de unos minutos, debería empezar a ver gráficos que muestran la cantidad de datos (Eventos/Bytes) que **salen** (Data Out) hacia Azure.

### En Microsoft Sentinel
1.  Vaya a su **Log Analytics Workspace** (o a **Microsoft Sentinel** -> **Registros (Logs)**).
2.  Abra la ventana de consultas.
3.  En el editor de consultas KQL, escriba el nombre de la tabla de destino:
    ```kql
    Syslog
    ```
4.  Haga clic en **"Ejecutar"**.
5.  Debería ver los logs de sus dispositivos aparecer en los resultados de la consulta.

¡Felicidades! Ha completado la solución de envío de logs desde sus dispositivos a Microsoft Sentinel a través de Cribl.

