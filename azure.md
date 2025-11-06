# Integración de Cribl con Microsoft Sentinel

Este documento describe la configuración necesaria para integrar **Cribl** con **Microsoft Sentinel**.  
El objetivo es que los dispositivos que pueden enviar logs a través de una IP de ingesta los redirijan a **Cribl**, el cual actuará como puente hacia Sentinel.

---

## 1. Creación del Grupo de Recursos

> **Requisito previo:** Debes contar con una cuenta activa en Azure.

1. Inicia sesión en el [Portal de Azure](https://portal.azure.com).
2. En el menú lateral izquierdo, selecciona **Grupos de recursos**.
3. Haz clic en **Crear** en la parte superior.

   ![Crear grupo de recursos](https://github.com/user-attachments/assets/3f8c9ab7-47cd-468f-b3c5-3e03a808c3a2)

   ![Formulario grupo de recursos](https://github.com/user-attachments/assets/61298376-4d87-4dcb-ab1b-c3d01386b618)

4. Completa los campos:
   - **Suscripción:** Selecciona la suscripción de Azure.
   - **Nombre del grupo de recursos:** Define un nombre único y descriptivo.
   - **Región:** Selecciona la ubicación geográfica (solo afecta metadatos).

5. Haz clic en **Revisar y crear** y luego en **Crear**.

---

## 2. Creación de Log Analytics Workspace (LAW)

1. En el portal de Azure, busca **Log Analytics workspaces** en la barra superior.
2. Selecciona la opción y haz clic en **Crear**.
3. En la pestaña **Básico**, completa:
   - **Suscripción:** Selecciona la suscripción activa.
   - **Grupo de recursos:** Usa el grupo creado previamente.
   - **Nombre del área de trabajo:** Ejemplo: `law-monitoring-prod`.
   - **Región:** Selecciona la misma región de tus recursos principales.
4. Haz clic en **Revisar y crear** y luego en **Crear**.
5. Una vez desplegado, valida que el recurso aparezca en tu grupo de recursos.  
   Desde aquí podrás conectarlo a **Azure Monitor, VMs, Defender for Cloud, Sentinel**, entre otros.

---

## 3. Configuración de Puntos de Conexión de Recopilación de Datos (DCE)

1. En el portal de Azure, busca **Data Collection Endpoints**.
2. Haz clic en **Crear**.
3. En la pestaña **Básico**, completa:
   - **Suscripción:** Selecciona la suscripción de Azure.
   - **Grupo de recursos:** Usa el grupo creado previamente.
   - **Nombre:** Ejemplo: `dce-monitoring-prod`.
   - **Región:** Selecciona la misma región que tu LAW.
4. (Opcional) Configura etiquetas si deseas clasificar el recurso.
5. Haz clic en **Revisar y crear** y luego en **Crear**.
6. Una vez creado, valida que el DCE aparezca en tu grupo de recursos.  
   Desde aquí podrás asociarlo a una **Regla de recopilación de datos (DCR)**, que define qué datos se recopilan y a dónde se envían (por ejemplo, a tu LAW).

---

## ✅ Conclusión

Con estos pasos ya tienes:
- Un **Grupo de Recursos** para organizar.
- Un **Log Analytics Workspace (LAW)** para almacenar logs.
- Un **Data Collection Endpoint (DCE)** para recibir y enrutar datos.

El siguiente paso será crear y asociar una **Regla de recopilación de datos (DCR)** que conecte Cribl con Sentinel a través del LAW.
