# Configuración de la herramienta de destino que envia logs a sentinel
---

Lo primero es serciorarse que los lohs llegan al syslog del pack, para esto es necesario configurar un equipo para enviar logs

en un documento anexo se detalla como configurar un equipo fortinet para el envio de logs y como comprobar si se reciben correctamente

Una vez comprobado que los logs llegan a cribl por medio del equipo de prueba, se configura el destino que asi es comunmente llamado en cribl

el destino se configura entrando en el apartado de "Destinatios" dentro del pack descargado anteriormente

aqui es donde se pueden configurar varios destinos, ya sea de tipo blob storage, azure data explorer, azure  monitor logs, event hub, y sentinel como este caso

seleccionamos el destino de sentinel, al hacer esot se abre una ventana donde tenemos que agregar es detino, para esto damos click en el boton de "add"

en la opcion de outpu ID, podemos poner el nombre que sea, teniendo en cuenta que los espacios deben de sustituirse por guines bajos

En el apartado de descripcion podemos agregar la descripcion que deseemos

en el selector de endpoint configuration seleccionamos el de tipo "ID", al seleccionarlo se abrira unos apartados que debemos llenar

en el nuevo apartado de data collection endopint, debemos poner la url de ingesta de registros que se obtiene: ingresando al DCE anteriormente creado
y el el apartado de info se nos mostrara la ur de ingesta de registros, la cual vamos a copear y pegar en el apartado de Data collection endpoint


en el apartado de Data collection rule ID, ingresamos a la DCR que creamos, y en el apartado de info semuestra el ID inmutable el cual debe,os copiar y pegar en el apartadoantes mencionado de cribl

y en el apartado de Stream name, debemos de poner el nombre de la tabla a donde se van a enviar los logs, al crear la DCR se crearon  4 tablas nuevas que se pueden ver en el apartado de configuración > origenes de datos, para este caso que es syslog, es necesario usar la tabla llamada Custom-Syslog, que es a donde se envian los logs de los diferentes dispositivos que envian logs desde una ip

eso seria todo en el apartado de general settings, ahora vamos al apartado de authentication, esta es la parte importante debido que se manejan diferentes tipos de llaves

en el apartado de Login URL, debemos buscar en el registro de aplicaciones de azure la aplicacion que se creo anteriormente, damos click y buscamos el apartado de puntos de conexion, damos click y de la lista que sale, buscamos el Punto de conexión de token de OAuth 2.0 (v2), una vez que lo encontramos copiamos el url que nos muestra y pegamos en el apartado de cribl antes mencionado

En el aprtado de OAuth secret, que esta en cribl, debemos de ingresar el valor del secreto creado cuando se realizo la configuracion de la app

En el apartado de Client ID, ponemos primero comillas dobles,y dentro de ellas pegamos el Id. de aplicación (cliente), que se encuentra en el apartado de info cuando se abre la app antes registrada en azure

Una vez completado estos pasos damos click en "Save", seguido de esto empezara la autenticación entre cribl y azure, 
