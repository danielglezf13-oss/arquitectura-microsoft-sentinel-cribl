# Agregar event hub a la integración
---
Algunos complementos dentro deazure como la recolezipon de logs de XDR no pueden tener una integración con cribl, en este caso se hace uso de un event hub, un complemento
de azure que nos permite hacer la recopilación de algunos logs que se necesitan enviar a azure pero que no existe una forma directa de hacerlo, asi que se recurre a un
event hub para hacer la recoleccion y despues enviarlos a sentienl por medio del LAW

A continuación se muestran lospasos para crear un event hub
