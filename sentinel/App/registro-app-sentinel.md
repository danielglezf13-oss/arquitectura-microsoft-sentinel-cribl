# Registro de Aplicación en Azure

Esta guía detalla el proceso para registrar una nueva aplicación en Azure, asignarle los permisos de API necesarios para leer datos de Log Analytics y generar un secreto de cliente para la autenticación.

## 1. Pasos para el Registro de Aplicación

1.  Inicia sesión en el **Portal de Azure** en `portal.azure.com`.
2.  En la barra de búsqueda superior, escribe y selecciona **"Registros de aplicaciones"** (App registrations).
3.  En la pantalla "Registros de aplicaciones", haz clic en el botón **"+ Nuevo registro"** (New registration).
4.  Completa el formulario de registro:
    * **Nombre (Name):** Asigna un nombre descriptivo para tu aplicación (ej. "MiScriptDeBackup", "WebAppDeInventario").
    * **Tipos de cuenta admitidos (Supported account types):** Selecciona la opción que se ajuste a tus necesidades.
        * *Opción común:* **"Solo cuentas en este directorio organizativo"** (Single tenant), si la aplicación solo será usada dentro de tu organización.
    * **URI de redirección (Opcional):** Si estás creando un script o un servicio de backend, puedes dejar esto en blanco por ahora.
5.  Haz clic en el botón **"Registrar"** en la parte inferior.

---

## 2. Asignación de Permisos de API

Una vez creada la aplicación, debemos asignarle permisos para que pueda leer datos.

1.  Busca tu aplicación recién creada en la lista de **"Registros de aplicaciones"** y haz clic en ella.
2.  En el menú de la izquierda, bajo la sección **"Administrar"**, selecciona **"Permisos de API"**.
3.  Haz clic en el botón **"+ Agregar un permiso"**.
4.  En la nueva ventana, selecciona la pestaña **"API usadas en mi organización"**.
5.  En el cuadro de búsqueda, escribe `Log Analytics API` y selecciónala de la lista.
6.  Selecciona el tipo de permiso **"Permisos de la aplicación"**.
7.  Se desplegará una lista de permisos. Marca la casilla del permiso `Data.Read`.
8.  Haz clic en el botón **"Agregar permisos"**.
9.  **Importante:** Debido a que es un permiso de aplicación, un administrador debe conceder el consentimiento. Haz clic en el botón **"Conceder consentimiento de administrador para [Tu Directorio]"** y acepta la solicitud.
10. Verifica que el permiso `Microsoft Graph` con `User.Read` (permiso delegado) también esté presente (normalmente se añade por defecto al crear la app).

---

## 3. Creación de un Secreto de Cliente

Tu aplicación necesita una "contraseña" para autenticarse. A esto se le llama Secreto de Cliente.

1.  En el menú de la izquierda de tu aplicación, bajo la sección **"Administrar"**, selecciona **"Certificados y secretos"** (Certificates & secrets).
2.  Haz clic en la pestaña **"Secretos de cliente"**.
3.  Haz clic en **"+ Nuevo secreto de cliente"** (New client secret).
4.  Escribe una **Descripción** (ej. "Clave para script de backup") y elige una **Fecha de expiración** (se recomienda usar un periodo corto, como 6 meses, por seguridad).
5.  Haz clic en **"Agregar"**.

> **¡ATENCIÓN! Copia el Secreto Inmediatamente**
>
> Después de agregar el secreto, este se mostrará en la columna **"Valor" (Value)**. Cópialo y guárdalo en un lugar seguro (como un gestor de contraseñas o un Key Vault) **AHORA MISMO**.
>
> **Nunca podrás volver a ver este valor** después de que salgas o actualices esta página. Si lo pierdes, tendrás que eliminarlo y crear uno nuevo.
