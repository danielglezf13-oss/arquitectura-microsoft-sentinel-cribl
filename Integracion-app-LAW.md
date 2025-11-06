# Guía: Asignación de Roles de IAM a un Log Analytics Workspace

Este documento detalla el procedimiento estándar para asignar una Entidad de Servicio (App Registration) de Azure a un Log Analytics Workspace (LAW) con los permisos adecuados.

Este paso es fundamental para habilitar integraciones de terceros, como la ingesta de datos desde plataformas como **Cribl**, permitiendo que la aplicación interactúe de forma segura con los recursos de Log Analytics.

> **Prerrequisitos:**
> * Una suscripción de Azure activa.
> * Un **Log Analytics Workspace** (LAW) previamente implementado.
> * Una **Entidad de Servicio** (App Registration) ya creada.
> * Permisos de `Propietario` (Owner) o `Administrador de acceso de usuarios` (User Access Administrator) en el LAW para poder asignar roles.

---

### 🎯 1. Navegar al Control de Acceso (IAM)

1.  Inicie sesión en el **Portal de Azure** (`portal.azure.com`).
2.  En la barra de búsqueda principal, localice y navegue a su **Log Analytics Workspace** de destino.
3.  En el menú de navegación izquierdo del workspace, seleccione **Control de acceso (IAM)**.

### ⚙️ 2. Iniciar Asignación de Rol

1.  Dentro del panel de **Control de acceso (IAM)**, haga clic en el botón **"+ Agregar"** en la parte superior.
2.  En el menú desplegable, seleccione **"Agregar asignación de roles"**. Esto iniciará el asistente de asignación.

### 🛡️ 3. Configurar la Asignación

El asistente le guiará a través de tres pestañas principales:

#### A. Pestaña "Rol"
1.  Verá una lista de roles integrados. En el cuadro de búsqueda "Buscar por nombre", escriba: **`Colaborador de Log Analytics`**.
2.  Seleccione el rol **"Colaborador de Log Analytics"** (`Log Analytics Contributor`) de la lista.
3.  Haga clic en **"Siguiente"**.

#### B. Pestaña "Miembros"
1.  En la sección "Asignar acceso a", asegúrese de que esté seleccionada la opción **"Usuario, grupo o entidad de servicio"**.
2.  Haga clic en el enlace azul **"+ Seleccionar miembros"**.
3.  Se abrirá un panel lateral. En la barra de búsqueda "Seleccionar", **escriba el nombre de la Entidad de Servicio** (App Registration) que creó anteriormente.
4.  Seleccione su aplicación de la lista de resultados y haga clic en el botón **"Seleccionar"**.

#### C. Pestaña "Revisar y asignar"
1.  Verifique que el **Rol** (`Colaborador de Log Analytics`) y el **Miembro** (su aplicación) son correctos.
2.  Haga clic en el botón **"Revisar y asignar"** para finalizar el proceso.

### ✅ 4. Verificación Final

Una vez completada la asignación, es crucial verificar que los permisos se hayan aplicado correctamente.

1.  Permanezca en el panel **Control de acceso (IAM)** de su LAW.
2.  Seleccione la pestaña **"Asignaciones de roles"**.
3.  En la barra de búsqueda "Buscar en esta lista", **filtre por el nombre de su aplicación**.
4.  Deberá ver su aplicación listada, confirmando que el rol de **"Colaborador de Log Analytics"** ha sido asignado exitosamente.

---

**Resultado:** Su Entidad de Servicio ahora tiene los permisos necesarios para colaborar con el Log Analytics Workspace. La configuración de permisos en Azure para la integración está completa.
