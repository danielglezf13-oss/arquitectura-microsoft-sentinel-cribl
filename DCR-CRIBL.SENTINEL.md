# Creación de la Regla de Recolectación de Datos (DCR) mediante Plantilla

Esta guía detalla el proceso para habilitar Microsoft Sentinel en un Log Analytics Workspace (LAW) y, posteriormente, implementar una Regla de Recolectación de Datos (DCR) personalizada utilizando una plantilla JSON.

---

## 📌 Requisito Previo: Habilitar Microsoft Sentinel

Antes de crear la DCR, su Log Analytics Workspace (LAW) debe estar asociado a Microsoft Sentinel.

1.  En la barra de búsqueda del Portal de Azure, busque y seleccione **"Microsoft Sentinel"**.
2.  Haga clic en **"Crear"**.
3.  Aparecerá una lista de sus Log Analytics Workspaces que aún no tienen Sentinel.
4.  Seleccione el LAW que ha estado utilizando y haga clic en **"Agregar"**.
5.  Azure procesará la solicitud y, después de unos momentos, su espacio de trabajo de Sentinel estará listo.

---

## 📋 Paso 1: Recolectar IDs de Recursos Esenciales

Para la plantilla de la DCR, necesitará dos valores críticos: el ID de Recurso del LAW y el ID de Recurso del Data Collection Endpoint (DCE).

> **Sugerencia:** Abra el Portal de Azure en una nueva pestaña o ventana del navegador para recolectar estos IDs sin perder su progreso en la implementación de la plantilla.

### A. Obtener el ID del Log Analytics Workspace (LAW)

1.  Navegue a su **Log Analytics Workspace** (LAW).
2.  En la página de "Información general" (Overview), haga clic en el enlace **"Vista JSON"** (ubicado cerca de la parte superior, junto al nombre del Grupo de Recursos).
3.  Se abrirá un panel lateral.
4.  Copie la cadena completa que aparece debajo de **"ID de recurso"** (`Resource ID`).
5.  Pegue esto en un lugar seguro para usarlo más adelante.

### B. Obtener el ID del Data Collection Endpoint (DCE)

1.  Navegue a su recurso de **"Data Collection Endpoint"** (Punto de conexión de recopilación de datos).
2.  En la página de "Información general" (Overview), haga clic en el enlace **"Vista JSON"**.
3.  Se abrirá un panel lateral.
4.  Copie la cadena completa que aparece debajo de **"ID de recurso"** (`Resource ID`).
5.  Pegue esto en un lugar seguro.

---

## 🚀 Paso 2: Implementar la Plantilla de DCR Personalizada

Ahora usaremos una plantilla ARM para crear la DCR con la configuración exacta que necesitamos.

1.  En la barra de búsqueda del Portal de Azure, busque **"Implementar desde una plantilla personalizada"** y seleccione ese servicio.
2.  Haga clic en la opción **"Cree su propia plantilla en el editor"**.
3.  **Importante:** En la pantalla de edición, elimine todo el código JSON que aparece por defecto (`{ "$schema": ... }`).
4.  Copie y pegue su código JSON de plantilla de DCR en el editor en blanco.

    > **Nota:** Pegue aquí el contenido completo de su plantilla JSON personalizada.
    >
    > ```json
    >// ... Borre todo a partir de este punto ...
    > {
    >   "$schema": "[https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#](https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#)",
    >   "contentVersion": "1.0.0.0",
    >   "parameters": {
    >     
    >   },
    >   "resources": [
    >     
    >   ]
    > }
    > ```

5.  Haga clic en **"Guardar"**.

---

## ✏️ Paso 3: Configurar los Parámetros de Implementación

Después de guardar, será dirigido a una pantalla para completar los parámetros de la plantilla.

1.  **Conceptos básicos:**
    * **Suscripción:** Seleccione la suscripción correcta.
    * **Grupo de recursos:** **(Crítico)** Seleccione el **mismo grupo de recursos** donde residen su LAW y su DCE.
    * **Región:** **(Crítico)** Asegúrese de que la región coincida con la de su LAW.

2.  **Configuración (Parámetros de la plantilla):**
    * **Data Collection Rule Name:** Escriba un nombre único y descriptivo para su DCR (ej. `dcr-cribl-sentinel-syslog`).
    * **Workspace Resource Id:** Pegue la cadena del **ID de recurso del LAW** que copió en el Paso 1.
    * **Endpoint Resource Id:** Pegue la cadena del **ID de recurso del DCE** que copió en el Paso 1.

---

## ✅ Paso 4: Revisar y Crear

1.  Una vez completados todos los campos, haga clic en el botón **"Revisar y crear"** en la parte inferior.
2.  Azure validará la plantilla y los parámetros. Si todo es correcto, verá un mensaje de "Validación superada".
3.  Haga clic en **"Crear"** para iniciar la implementación.
4.  Recibirá una notificación cuando la implementación se haya completado con éxito ("Implementación correcta").

Ha creado y configurado exitosamente su Regla de Recolección de Datos (DCR) personalizada.

A continuación se muestra la DCR en formato JSON que debe pegar en el área correspondiente al crear la DCR.
```DCR SENTINEL
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "dataCollectionRuleName": {
            "type": "string",
            "metadata": {
                "description": "Specifies the name of the Data Collection Rule to create."
            }
        },
        "location": {
            "defaultValue": "[resourceGroup().location]",
            "type": "string",
            "metadata": {
                "description": "Specifies the location in which to create the Data Collection Rule."
            }
        },
        "workspaceResourceId": {
            "type": "string",
            "metadata": {
                "description": "Specifies the Azure resource ID of the Log Analytics workspace to use."
            }
        },
        "endpointResourceId": {
            "type": "string",
            "metadata": {
                "description": "Specifies the Azure resource ID of the Data Collection Endpoint to use."
            }
        }
    },
    "resources": [{
        "type": "Microsoft.Insights/dataCollectionRules",
        "apiVersion": "2021-09-01-preview",
        "name": "[parameters('dataCollectionRuleName')]",
        "location": "[parameters('location')]",
        "properties": {
            "dataCollectionEndpointId": "[parameters('endpointResourceId')]",
            "streamDeclarations": {
                "Custom-CommonSecurityLog": {
                    "columns": [{
                            "name": "Activity",
                            "type": "string"
                        },
                        {
                            "name": "AdditionalExtensions",
                            "type": "string"
                        },
                        {
                            "name": "ApplicationProtocol",
                            "type": "string"
                        },
                        {
                            "name": "CollectorHostName",
                            "type": "string"
                        },
                        {
                            "name": "CommunicationDirection",
                            "type": "string"
                        },
                        {
                            "name": "Computer",
                            "type": "string"
                        },
                        {
                            "name": "DestinationDnsDomain",
                            "type": "string"
                        },
                        {
                            "name": "DestinationHostName",
                            "type": "string"
                        },
                        {
                            "name": "DestinationIP",
                            "type": "string"
                        },
                        {
                            "name": "DestinationMACAddress",
                            "type": "string"
                        },
                        {
                            "name": "DestinationNTDomain",
                            "type": "string"
                        },
                        {
                            "name": "DestinationPort",
                            "type": "int"
                        },
                        {
                            "name": "DestinationProcessId",
                            "type": "int"
                        },
                        {
                            "name": "DestinationProcessName",
                            "type": "string"
                        },
                        {
                            "name": "DestinationServiceName",
                            "type": "string"
                        },
                        {
                            "name": "DestinationTranslatedAddress",
                            "type": "string"
                        },
                        {
                            "name": "DestinationTranslatedPort",
                            "type": "int"
                        },
                        {
                            "name": "DestinationUserID",
                            "type": "string"
                        },
                        {
                            "name": "DestinationUserName",
                            "type": "string"
                        },
                        {
                            "name": "DestinationUserPrivileges",
                            "type": "string"
                        },
                        {
                            "name": "DeviceAction",
                            "type": "string"
                        },
                        {
                            "name": "DeviceAddress",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomDate1",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomDate1Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomDate2",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomDate2Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomFloatingPoint1",
                            "type": "real"
                        },
                        {
                            "name": "DeviceCustomFloatingPoint1Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomFloatingPoint2",
                            "type": "real"
                        },
                        {
                            "name": "DeviceCustomFloatingPoint2Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomFloatingPoint3",
                            "type": "real"
                        },
                        {
                            "name": "DeviceCustomFloatingPoint3Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomFloatingPoint4",
                            "type": "real"
                        },
                        {
                            "name": "DeviceCustomFloatingPoint4Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomIPv6Address1",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomIPv6Address1Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomIPv6Address2",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomIPv6Address2Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomIPv6Address3",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomIPv6Address3Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomIPv6Address4",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomIPv6Address4Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomNumber1",
                            "type": "int"
                        },
                        {
                            "name": "DeviceCustomNumber1Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomNumber2",
                            "type": "int"
                        },
                        {
                            "name": "DeviceCustomNumber2Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomNumber3",
                            "type": "int"
                        },
                        {
                            "name": "DeviceCustomNumber3Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomString1",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomString1Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomString2",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomString2Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomString3",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomString3Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomString4",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomString4Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomString5",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomString5Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomString6",
                            "type": "string"
                        },
                        {
                            "name": "DeviceCustomString6Label",
                            "type": "string"
                        },
                        {
                            "name": "DeviceDnsDomain",
                            "type": "string"
                        },
                        {
                            "name": "DeviceEventCategory",
                            "type": "string"
                        },
                        {
                            "name": "DeviceEventClassID",
                            "type": "string"
                        },
                        {
                            "name": "DeviceExternalID",
                            "type": "string"
                        },
                        {
                            "name": "DeviceFacility",
                            "type": "string"
                        },
                        {
                            "name": "DeviceInboundInterface",
                            "type": "string"
                        },
                        {
                            "name": "DeviceMacAddress",
                            "type": "string"
                        },
                        {
                            "name": "DeviceName",
                            "type": "string"
                        },
                        {
                            "name": "DeviceNtDomain",
                            "type": "string"
                        },
                        {
                            "name": "DeviceOutboundInterface",
                            "type": "string"
                        },
                        {
                            "name": "DevicePayloadId",
                            "type": "string"
                        },
                        {
                            "name": "DeviceProduct",
                            "type": "string"
                        },
                        {
                            "name": "DeviceTimeZone",
                            "type": "string"
                        },
                        {
                            "name": "DeviceTranslatedAddress",
                            "type": "string"
                        },
                        {
                            "name": "DeviceVendor",
                            "type": "string"
                        },
                        {
                            "name": "DeviceVersion",
                            "type": "string"
                        },
                        {
                            "name": "EndTime",
                            "type": "datetime"
                        },
                        {
                            "name": "EventCount",
                            "type": "int"
                        },
                        {
                            "name": "EventOutcome",
                            "type": "string"
                        },
                        {
                            "name": "EventType",
                            "type": "int"
                        },
                        {
                            "name": "ExternalID",
                            "type": "int"
                        },
                        {
                            "name": "ExtID",
                            "type": "string"
                        },
                        {
                            "name": "FieldDeviceCustomNumber1",
                            "type": "long"
                        },
                        {
                            "name": "FieldDeviceCustomNumber2",
                            "type": "long"
                        },
                        {
                            "name": "FieldDeviceCustomNumber3",
                            "type": "long"
                        },
                        {
                            "name": "FileCreateTime",
                            "type": "string"
                        },
                        {
                            "name": "FileHash",
                            "type": "string"
                        },
                        {
                            "name": "FileID",
                            "type": "string"
                        },
                        {
                            "name": "FileModificationTime",
                            "type": "string"
                        },
                        {
                            "name": "FileName",
                            "type": "string"
                        },
                        {
                            "name": "FilePath",
                            "type": "string"
                        },
                        {
                            "name": "FilePermission",
                            "type": "string"
                        },
                        {
                            "name": "FileSize",
                            "type": "int"
                        },
                        {
                            "name": "FileType",
                            "type": "string"
                        },
                        {
                            "name": "FlexDate1",
                            "type": "string"
                        },
                        {
                            "name": "FlexDate1Label",
                            "type": "string"
                        },
                        {
                            "name": "FlexNumber1",
                            "type": "int"
                        },
                        {
                            "name": "FlexNumber1Label",
                            "type": "string"
                        },
                        {
                            "name": "FlexNumber2",
                            "type": "int"
                        },
                        {
                            "name": "FlexNumber2Label",
                            "type": "string"
                        },
                        {
                            "name": "FlexString1",
                            "type": "string"
                        },
                        {
                            "name": "FlexString1Label",
                            "type": "string"
                        },
                        {
                            "name": "FlexString2",
                            "type": "string"
                        },
                        {
                            "name": "FlexString2Label",
                            "type": "string"
                        },
                        {
                            "name": "IndicatorThreatType",
                            "type": "string"
                        },
                        {
                            "name": "LogSeverity",
                            "type": "string"
                        },
                        {
                            "name": "MaliciousIP",
                            "type": "string"
                        },
                        {
                            "name": "MaliciousIPCountry",
                            "type": "string"
                        },
                        {
                            "name": "MaliciousIPLatitude",
                            "type": "real"
                        },
                        {
                            "name": "MaliciousIPLongitude",
                            "type": "real"
                        },
                        {
                            "name": "Message",
                            "type": "string"
                        },
                        {
                            "name": "OldFileCreateTime",
                            "type": "string"
                        },
                        {
                            "name": "OldFileHash",
                            "type": "string"
                        },
                        {
                            "name": "OldFileID",
                            "type": "string"
                        },
                        {
                            "name": "OldFileModificationTime",
                            "type": "string"
                        },
                        {
                            "name": "OldFileName",
                            "type": "string"
                        },
                        {
                            "name": "OldFilePath",
                            "type": "string"
                        },
                        {
                            "name": "OldFilePermission",
                            "type": "string"
                        },
                        {
                            "name": "OldFileSize",
                            "type": "int"
                        },
                        {
                            "name": "OldFileType",
                            "type": "string"
                        },
                        {
                            "name": "OriginalLogSeverity",
                            "type": "string"
                        },
                        {
                            "name": "ProcessID",
                            "type": "int"
                        },
                        {
                            "name": "ProcessName",
                            "type": "string"
                        },
                        {
                            "name": "Protocol",
                            "type": "string"
                        },
                        {
                            "name": "Reason",
                            "type": "string"
                        },
                        {
                            "name": "ReceiptTime",
                            "type": "string"
                        },
                        {
                            "name": "ReceivedBytes",
                            "type": "long"
                        },
                        {
                            "name": "RemoteIP",
                            "type": "string"
                        },
                        {
                            "name": "RemotePort",
                            "type": "string"
                        },
                        {
                            "name": "ReportReferenceLink",
                            "type": "string"
                        },
                        {
                            "name": "RequestClientApplication",
                            "type": "string"
                        },
                        {
                            "name": "RequestContext",
                            "type": "string"
                        },
                        {
                            "name": "RequestCookies",
                            "type": "string"
                        },
                        {
                            "name": "RequestMethod",
                            "type": "string"
                        },
                        {
                            "name": "RequestURL",
                            "type": "string"
                        },
                        {
                            "name": "SentBytes",
                            "type": "long"
                        },
                        {
                            "name": "SimplifiedDeviceAction",
                            "type": "string"
                        },
                        {
                            "name": "SourceDnsDomain",
                            "type": "string"
                        },
                        {
                            "name": "SourceHostName",
                            "type": "string"
                        },
                        {
                            "name": "SourceIP",
                            "type": "string"
                        },
                        {
                            "name": "SourceMACAddress",
                            "type": "string"
                        },
                        {
                            "name": "SourceNTDomain",
                            "type": "string"
                        },
                        {
                            "name": "SourcePort",
                            "type": "int"
                        },
                        {
                            "name": "SourceProcessId",
                            "type": "int"
                        },
                        {
                            "name": "SourceProcessName",
                            "type": "string"
                        },
                        {
                            "name": "SourceServiceName",
                            "type": "string"
                        },
                        {
                            "name": "SourceSystem",
                            "type": "string"
                        },
                        {
                            "name": "SourceTranslatedAddress",
                            "type": "string"
                        },
                        {
                            "name": "SourceTranslatedPort",
                            "type": "int"
                        },
                        {
                            "name": "SourceUserID",
                            "type": "string"
                        },
                        {
                            "name": "SourceUserName",
                            "type": "string"
                        },
                        {
                            "name": "SourceUserPrivileges",
                            "type": "string"
                        },
                        {
                            "name": "StartTime",
                            "type": "datetime"
                        },
                        {
                            "name": "ThreatConfidence",
                            "type": "string"
                        },
                        {
                            "name": "ThreatDescription",
                            "type": "string"
                        },
                        {
                            "name": "ThreatSeverity",
                            "type": "int"
                        },
                        {
                            "name": "TimeGenerated",
                            "type": "datetime"
                        }
                    ]
                },
                "Custom-SecurityEvent": {
                    "columns": [{
                            "name": "AccessList",
                            "type": "string"
                        },
                        {
                            "name": "AccessMask",
                            "type": "string"
                        },
                        {
                            "name": "AccessReason",
                            "type": "string"
                        },
                        {
                            "name": "Account",
                            "type": "string"
                        },
                        {
                            "name": "AccountDomain",
                            "type": "string"
                        },
                        {
                            "name": "AccountExpires",
                            "type": "string"
                        },
                        {
                            "name": "AccountName",
                            "type": "string"
                        },
                        {
                            "name": "AccountSessionIdentifier",
                            "type": "string"
                        },
                        {
                            "name": "AccountType",
                            "type": "string"
                        },
                        {
                            "name": "Activity",
                            "type": "string"
                        },
                        {
                            "name": "AdditionalInfo",
                            "type": "string"
                        },
                        {
                            "name": "AdditionalInfo2",
                            "type": "string"
                        },
                        {
                            "name": "AllowedToDelegateTo",
                            "type": "string"
                        },
                        {
                            "name": "Attributes",
                            "type": "string"
                        },
                        {
                            "name": "AuditPolicyChanges",
                            "type": "string"
                        },
                        {
                            "name": "AuditsDiscarded",
                            "type": "int"
                        },
                        {
                            "name": "AuthenticationLevel",
                            "type": "int"
                        },
                        {
                            "name": "AuthenticationPackageName",
                            "type": "string"
                        },
                        {
                            "name": "AuthenticationProvider",
                            "type": "string"
                        },
                        {
                            "name": "AuthenticationServer",
                            "type": "string"
                        },
                        {
                            "name": "AuthenticationService",
                            "type": "int"
                        },
                        {
                            "name": "AuthenticationType",
                            "type": "string"
                        },
                        {
                            "name": "AzureDeploymentID",
                            "type": "string"
                        },
                        {
                            "name": "CACertificateHash",
                            "type": "string"
                        },
                        {
                            "name": "CallerProcessId",
                            "type": "string"
                        },
                        {
                            "name": "CalledStationID",
                            "type": "string"
                        },
                        {
                            "name": "CallerProcessName",
                            "type": "string"
                        },
                        {
                            "name": "CallingStationID",
                            "type": "string"
                        },
                        {
                            "name": "CAPublicKeyHash",
                            "type": "string"
                        },
                        {
                            "name": "CategoryId",
                            "type": "string"
                        },
                        {
                            "name": "CertificateDatabaseHash",
                            "type": "string"
                        },
                        {
                            "name": "Channel",
                            "type": "string"
                        },
                        {
                            "name": "ClassId",
                            "type": "string"
                        },
                        {
                            "name": "ClassName",
                            "type": "string"
                        },
                        {
                            "name": "ClientAddress",
                            "type": "string"
                        },
                        {
                            "name": "ClientIPAddress",
                            "type": "string"
                        },
                        {
                            "name": "ClientName",
                            "type": "string"
                        },
                        {
                            "name": "CommandLine",
                            "type": "string"
                        },
                        {
                            "name": "CompatibleIds",
                            "type": "string"
                        },
                        {
                            "name": "Computer",
                            "type": "string"
                        },
                        {
                            "name": "DCDNSName",
                            "type": "string"
                        },
                        {
                            "name": "DeviceId",
                            "type": "string"
                        },
                        {
                            "name": "DisplayName",
                            "type": "string"
                        },
                        {
                            "name": "Disposition",
                            "type": "string"
                        },
                        {
                            "name": "DomainBehaviorVersion",
                            "type": "string"
                        },
                        {
                            "name": "DomainName",
                            "type": "string"
                        },
                        {
                            "name": "DomainPolicyChanged",
                            "type": "string"
                        },
                        {
                            "name": "DomainSid",
                            "type": "string"
                        },
                        {
                            "name": "EAPType",
                            "type": "string"
                        },
                        {
                            "name": "ErrorCode",
                            "type": "int"
                        },
                        {
                            "name": "ElevatedToken",
                            "type": "string"
                        },
                        {
                            "name": "EventID",
                            "type": "int"
                        },
                        {
                            "name": "EventData",
                            "type": "string"
                        },
                        {
                            "name": "EventSourceName",
                            "type": "string"
                        },
                        {
                            "name": "ExtendedQuarantineState",
                            "type": "string"
                        },
                        {
                            "name": "FailureReason",
                            "type": "string"
                        },
                        {
                            "name": "FileHash",
                            "type": "string"
                        },
                        {
                            "name": "FilePath",
                            "type": "string"
                        },
                        {
                            "name": "FilePathNoUser",
                            "type": "string"
                        },
                        {
                            "name": "Filter",
                            "type": "string"
                        },
                        {
                            "name": "ForceLogoff",
                            "type": "string"
                        },
                        {
                            "name": "Fqbn",
                            "type": "string"
                        },
                        {
                            "name": "FullyQualifiedSubjectMachineName",
                            "type": "string"
                        },
                        {
                            "name": "FullyQualifiedSubjectUserName",
                            "type": "string"
                        },
                        {
                            "name": "GroupMembership",
                            "type": "string"
                        },
                        {
                            "name": "HandleId",
                            "type": "string"
                        },
                        {
                            "name": "HardwareIds",
                            "type": "string"
                        },
                        {
                            "name": "HomeDirectory",
                            "type": "string"
                        },
                        {
                            "name": "HomePath",
                            "type": "string"
                        },
                        {
                            "name": "ImpersonationLevel",
                            "type": "string"
                        },
                        {
                            "name": "IpAddress",
                            "type": "string"
                        },
                        {
                            "name": "IpPort",
                            "type": "string"
                        },
                        {
                            "name": "KeyLength",
                            "type": "int"
                        },
                        {
                            "name": "Level",
                            "type": "string"
                        },
                        {
                            "name": "LmPackageName",
                            "type": "string"
                        },
                        {
                            "name": "LocationInformation",
                            "type": "string"
                        },
                        {
                            "name": "LockoutDuration",
                            "type": "string"
                        },
                        {
                            "name": "LockoutObservationWindow",
                            "type": "string"
                        },
                        {
                            "name": "LockoutThreshold",
                            "type": "string"
                        },
                        {
                            "name": "LoggingResult",
                            "type": "string"
                        },
                      
                        {
                            "name": "LogonHours",
                            "type": "string"
                        },
                        {
                            "name": "LogonID",
                            "type": "string"
                        },
                        {
                            "name": "LogonProcessName",
                            "type": "string"
                        },
                        {
                            "name": "LogonType",
                            "type": "int"
                        },
                        {
                            "name": "LogonTypeName",
                            "type": "string"
                        },
                        {
                            "name": "MachineAccountQuota",
                            "type": "string"
                        },
                        {
                            "name": "MachineInventory",
                            "type": "string"
                        },
                        {
                            "name": "MachineLogon",
                            "type": "string"
                        },
                        {
                            "name": "ManagementGroupName",
                            "type": "string"
                        },
                        {
                            "name": "MandatoryLabel",
                            "type": "string"
                        },
                        {
                            "name": "MaxPasswordAge",
                            "type": "string"
                        },
                        {
                            "name": "MemberName",
                            "type": "string"
                        },
                        {
                            "name": "MemberSid",
                            "type": "string"
                        },
                        {
                            "name": "MinPasswordAge",
                            "type": "string"
                        },
                        {
                            "name": "MinPasswordLength",
                            "type": "string"
                        },
                        {
                            "name": "MixedDomainMode",
                            "type": "string"
                        },
                        {
                            "name": "NASIdentifier",
                            "type": "string"
                        },
                        {
                            "name": "NASIPv4Address",
                            "type": "string"
                        },
                        {
                            "name": "NASIPv6Address",
                            "type": "string"
                        },
                        {
                            "name": "NASPort",
                            "type": "string"
                        },
                        {
                            "name": "NASPortType",
                            "type": "string"
                        },
                        {
                            "name": "NetworkPolicyName",
                            "type": "string"
                        },
                        {
                            "name": "NewDate",
                            "type": "string"
                        },
                        {
                            "name": "NewMaxUsers",
                            "type": "string"
                        },
                        {
                            "name": "NewProcessId",
                            "type": "string"
                        },
                        {
                            "name": "NewProcessName",
                            "type": "string"
                        },
                        {
                            "name": "NewRemark",
                            "type": "string"
                        },
                        {
                            "name": "NewShareFlags",
                            "type": "string"
                        },
                        {
                            "name": "NewTime",
                            "type": "string"
                        },
                        {
                            "name": "NewUacValue",
                            "type": "string"
                        },
                        {
                            "name": "NewValue",
                            "type": "string"
                        },
                        {
                            "name": "NewValueType",
                            "type": "string"
                        },
                        {
                            "name": "ObjectName",
                            "type": "string"
                        },
                        {
                            "name": "ObjectServer",
                            "type": "string"
                        },
                        {
                            "name": "ObjectType",
                            "type": "string"
                        },
                        {
                            "name": "ObjectValueName",
                            "type": "string"
                        },
                        {
                            "name": "OemInformation",
                            "type": "string"
                        },
                        {
                            "name": "OldMaxUsers",
                            "type": "string"
                        },
                        {
                            "name": "OldRemark",
                            "type": "string"
                        },
                        {
                            "name": "OldShareFlags",
                            "type": "string"
                        },
                        {
                            "name": "OldUacValue",
                            "type": "string"
                        },
                        {
                            "name": "OldValue",
                            "type": "string"
                        },
                        {
                            "name": "OldValueType",
                            "type": "string"
                        },
                        {
                            "name": "OperationType",
                            "type": "string"
                        },
                        {
                            "name": "PackageName",
                            "type": "string"
                        },
                        {
                            "name": "ParentProcessName",
                            "type": "string"
                        },
                        {
                            "name": "PartitionKey",
                            "type": "string"
                        },
                        {
                            "name": "PasswordHistoryLength",
                            "type": "string"
                        },
                        {
                            "name": "PasswordLastSet",
                            "type": "string"
                        },
                        {
                            "name": "PasswordProperties",
                            "type": "string"
                        },
                        {
                            "name": "PreviousDate",
                            "type": "string"
                        },
                        {
                            "name": "PreviousTime",
                            "type": "string"
                        },
                        {
                            "name": "PrimaryGroupId",
                            "type": "string"
                        },
                        {
                            "name": "PrivateKeyUsageCount",
                            "type": "string"
                        },
                        {
                            "name": "PrivilegeList",
                            "type": "string"
                        },
                        {
                            "name": "Process",
                            "type": "string"
                        },
                        {
                            "name": "ProcessId",
                            "type": "string"
                        },
                        {
                            "name": "ProcessName",
                            "type": "string"
                        },
                        {
                            "name": "ProfilePath",
                            "type": "string"
                        },
                        {
                            "name": "Properties",
                            "type": "string"
                        },
                        {
                            "name": "ProtocolSequence",
                            "type": "string"
                        },
                        {
                            "name": "ProxyPolicyName",
                            "type": "string"
                        },
                        {
                            "name": "QuarantineHelpURL",
                            "type": "string"
                        },
                        {
                            "name": "QuarantineSessionID",
                            "type": "string"
                        },
                        {
                            "name": "QuarantineSessionIdentifier",
                            "type": "string"
                        },
                        {
                            "name": "QuarantineState",
                            "type": "string"
                        },
                        {
                            "name": "QuarantineSystemHealthResult",
                            "type": "string"
                        },
                        {
                            "name": "RelativeTargetName",
                            "type": "string"
                        },
                        {
                            "name": "RemoteIpAddress",
                            "type": "string"
                        },
                        {
                            "name": "RemotePort",
                            "type": "string"
                        },
                        {
                            "name": "Requester",
                            "type": "string"
                        },
                        {
                            "name": "RequestId",
                            "type": "string"
                        },
                        {
                            "name": "RestrictedAdminMode",
                            "type": "string"
                        },
                        {
                            "name": "RowKey",
                            "type": "string"
                        },
                        {
                            "name": "RowsDeleted",
                            "type": "string"
                        },
                        {
                            "name": "SamAccountName",
                            "type": "string"
                        },
                        {
                            "name": "ScriptPath",
                            "type": "string"
                        },
                        {
                            "name": "SecurityDescriptor",
                            "type": "string"
                        },
                        {
                            "name": "ServiceAccount",
                            "type": "string"
                        },
                        {
                            "name": "ServiceFileName",
                            "type": "string"
                        },
                        {
                            "name": "ServiceName",
                            "type": "string"
                        },
                        {
                            "name": "ServiceStartType",
                            "type": "int"
                        },
                        {
                            "name": "ServiceType",
                            "type": "string"
                        },
                        {
                            "name": "SessionName",
                            "type": "string"
                        },
                        {
                            "name": "ShareLocalPath",
                            "type": "string"
                        },
                        {
                            "name": "ShareName",
                            "type": "string"
                        },
                        {
                            "name": "SidHistory",
                            "type": "string"
                        },
                       
                        {
                            "name": "SourceSystem",
                            "type": "string"
                        },
                        {
                            "name": "Status",
                            "type": "string"
                        },
                        {
                            "name": "StorageAccount",
                            "type": "string"
                        },
                        {
                            "name": "SubcategoryId",
                            "type": "string"
                        },
                      
                        {
                            "name": "Subject",
                            "type": "string"
                        },
                        {
                            "name": "SubjectAccount",
                            "type": "string"
                        },
                        {
                            "name": "SubjectDomainName",
                            "type": "string"
                        },
                        {
                            "name": "SubjectKeyIdentifier",
                            "type": "string"
                        },
                        {
                            "name": "SubjectLogonId",
                            "type": "string"
                        },
                        {
                            "name": "SubjectMachineName",
                            "type": "string"
                        },
                        {
                            "name": "SubjectMachineSID",
                            "type": "string"
                        },
                        {
                            "name": "SubjectUserName",
                            "type": "string"
                        },
                        {
                            "name": "SubjectUserSid",
                            "type": "string"
                        },
                        {
                            "name": "SubStatus",
                            "type": "string"
                        },
                        {
                            "name": "TableId",
                            "type": "string"
                        },
                        {
                            "name": "TargetDomainName",
                            "type": "string"
                        },
                        {
                            "name": "TargetInfo",
                            "type": "string"
                        },
                        {
                            "name": "TargetAccount",
                            "type": "string"
                        },
                        {
                            "name": "TargetLinkedLogonId",
                            "type": "string"
                        },
                        {
                            "name": "TargetLogonId",
                            "type": "string"
                        },
                        {
                            "name": "TargetOutboundDomainName",
                            "type": "string"
                        },
                        {
                            "name": "TargetOutboundUserName",
                            "type": "string"
                        },
                        {
                            "name": "TargetServerName",
                            "type": "string"
                        },
                        {
                            "name": "TargetSid",
                            "type": "string"
                        },
                        {
                            "name": "TargetUser",
                            "type": "string"
                        },
                        {
                            "name": "TargetUserName",
                            "type": "string"
                        },
                        {
                            "name": "TargetUserSid",
                            "type": "string"
                        },
                        {
                            "name": "Task",
                            "type": "int"
                        },
                        {
                            "name": "TemplateContent",
                            "type": "string"
                        },
                        {
                            "name": "TemplateDSObjectFQDN",
                            "type": "string"
                        },
                        {
                            "name": "TemplateInternalName",
                            "type": "string"
                        },
                        {
                            "name": "TemplateOID",
                            "type": "string"
                        },
                        {
                            "name": "TemplateSchemaVersion",
                            "type": "string"
                        },
                        {
                            "name": "TemplateVersion",
                            "type": "string"
                        },
                        {
                            "name": "TimeCollected",
                            "type": "datetime"
                        },
                        {
                            "name": "TimeGenerated",
                            "type": "datetime"
                        },
                        {
                            "name": "TokenElevationType",
                            "type": "string"
                        },
                        {
                            "name": "TransmittedServices",
                            "type": "string"
                        },
                        {
                            "name": "UserAccountControl",
                            "type": "string"
                        },
                        {
                            "name": "UserParametersUserParameters",
                            "type": "string"
                        },
                        {
                            "name": "UserPrincipalName",
                            "type": "string"
                        },
                        {
                            "name": "UserWorkstationsUserWorkstations",
                            "type": "string"
                        },
                        {
                            "name": "VendorIds",
                            "type": "string"
                        },
                        {
                            "name": "VirtualAccount",
                            "type": "string"
                        },
                        {
                            "name": "Workstation",
                            "type": "string"
                        },
                        {
                            "name": "WorkstationName",
                            "type": "string"
                        }
                    ]
                },
                "Custom-Syslog": {
                    "columns": [{
                            "name": "Computer",
                            "type": "string"
                        },
                        {
                            "name": "EventTime",
                            "type": "datetime"
                        },
                        {
                            "name": "Facility",
                            "type": "string"
                        },
                        {
                            "name": "HostIP",
                            "type": "string"
                        },
                        {
                            "name": "HostName",
                            "type": "string"
                        },
                        {
                            "name": "ManagementGroupName",
                            "type": "string"
                        },
                        {
                            "name": "ProcessID",
                            "type": "int"
                        },
                        {
                            "name": "ProcessName",
                            "type": "string"
                        },
                        {
                            "name": "SeverityLevel",
                            "type": "string"
                        },
                        {
                            "name": "SourceSystem",
                            "type": "string"
                        },
                        {
                            "name": "SyslogMessage",
                            "type": "string"
                        },
                        {
                            "name": "TimeCollected",
                            "type": "datetime"
                        },
                        {
                            "name": "TimeGenerated",
                            "type": "datetime"
                        }
                    ]
                },
                "Custom-WindowsEvent": {
                    "columns": [{
                            "name": "Channel",
                            "type": "string"
                        },
                        {
                            "name": "Computer",
                            "type": "string"
                        },
                        {
                            "name": "EventData",
                            "type": "string"
                        },
                        {
                            "name": "EventID",
                            "type": "int"
                        },
                        {
                            "name": "EventLevel",
                            "type": "int"
                        },
                        {
                            "name": "EventLevelName",
                            "type": "string"
                        },
                        {
                            "name": "EventOriginId",
                            "type": "string"
                        },
                        {
                            "name": "ManagementGroupName",
                            "type": "string"
                        },
                        {
                            "name": "Provider",
                            "type": "string"
                        },
                        {
                            "name": "RawEventData",
                            "type": "string"
                        },
                        {
                            "name": "SourceSystem",
                            "type": "string"
                        },
                        {
                            "name": "Task",
                            "type": "int"
                        },
                        {
                            "name": "TimeGenerated",
                            "type": "datetime"
                        }
                    ]
                }
            },
            "destinations": {
                "logAnalytics": [{
                    "workspaceResourceId": "[parameters('workspaceResourceId')]",
                    "name": "logAnalyticsWorkspace"
                }]
            },
            "dataFlows": [{
                    "streams": [
                        "Custom-CommonSecurityLog"
                    ],
                    "destinations": [
                        "logAnalyticsWorkspace"
                    ],
                    "transformKql": "source",
                    "outputStream": "Microsoft-CommonSecurityLog"
                },
                {
                    "streams": [
                        "Custom-SecurityEvent"
                    ],
                    "destinations": [
                        "logAnalyticsWorkspace"
                    ],
                    "transformKql": "source",
                    "outputStream": "Microsoft-SecurityEvent"
                },
                {
                    "streams": [
                        "Custom-Syslog"
                    ],
                    "destinations": [
                        "logAnalyticsWorkspace"
                    ],
                    "transformKql": "source",
                    "outputStream": "Microsoft-Syslog"
                },
                {
                    "streams": [
                        "Custom-WindowsEvent"
                    ],
                    "destinations": [
                        "logAnalyticsWorkspace"
                    ],
                    "transformKql": "source",
                    "outputStream": "Microsoft-WindowsEvent"
                }
            ]
        }
    }],
    "outputs": {
        "dataCollectionRuleId": {
            "type": "string",
            "value": "[resourceId('Microsoft.Insights/dataCollectionRules', parameters('dataCollectionRuleName'))]"
        }
    }
}
```
> [!IMPORTANT]
> **Requisito de Permisos Crítico** 🛑
> Para que la ingestión de datos funcione correctamente, es **indispensable** autorizar a la aplicación.
>
> Debes asignar el rol **Monitoring Metrics Publisher** (Publicador de métricas de supervisión) a tu **Service Principal** (App Registration) dentro del control de acceso (IAM) de la **Data Collection Rule (DCR)**.
