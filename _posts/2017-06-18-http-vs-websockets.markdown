---
layout: post
title:  "HTTP vs Websockets"
date:   2017-06-18 16:00:00 +0100
categories: blog
---

# HTTP vs Websockets

Esta semana discutía con unos compañeros de equipo el sentido de duplicar una API y exponerla tanto por HTTP como por websockets. Es un tema que ha generado un debate interesante y voy a intentar plasmar mis opiniones en este post.

## El problema
Tenemos un servidor que se encarga de procesar todo el `signaling` de nuestras aplicaciones WebRTC.
Se encarga entre otras cosas de crear una llamada y avisar a todos los invitados o de notificar a los usuarios cuando un contacto se conecta o se desconecta.

Dada la gran cantidad de mensajes asíncronos que el servidor tiene que enviar a los clientes si utilizásemos solamente http estaríamos obligados a utilizar [long polling](https://es.wikipedia.org/wiki/Tecnolog%C3%ADa_Push#Long_polling) para que los clientes reciban estos mensajes.

Es precisamente evitar el uso del long polling el motivo por el que se añaden websockets al servidor. Un evento asíncrono como una llamada entrante será sencillamente un mensaje `incoming-call` a través de los ws.

Ya que nuestro servidor tiene websockets y http, **tendría sentido ofrecer la misma api que se ofrece por http también via websockets?**, **¿tiene sentido utilizar los ws para enviar mensajes desde el cliente al servidor?**

Por ejemplo, para obtener el historial de llamadas o para crear una nueva llamada se hacen peticiones HTTP como las siguientes:

    GET /calls  # Se obtiene una lista con todas las llamadas realizadas por el usuario
    POST /calls # Crea una nueva llamada

¿Tendría algúna ventaja respecto a http ofrecer esta funcionalidad mediante websockets?

```javascript
wss.send('get-calls') // Obtener la lista de llamadas.
wss.send('create-call', data); // Crear una nueva llamada.
```

## Pros & Contras

### Websockets no obliga a que cada mensaje tenga una respuesta asociada.
Imaginemos el caso de crear dos llamadas simultaneas, una se crea correctamente y otra falla por que no hemos escrito bien el nombre del contacto.

Si utilizamos http, tendremos dos peticiones bien diferenciadas, cuyas respuestas serán 201 (creada) y 404 (número no encontrado). Las propias apis de los navegadores para hacer peticiones AJAX ya saben como manejar estas respuestas, por lo que será bastante sencillo de implementar.

Sin embargo, al utilizar websockets y de ser necesario, tendremos que programar algun mecanismo que asocie cada petición a su respuesta generando complejidad extra en nuestro sistema.


### Websockets tiene menor latencia que HTTP
Por cada nueva conexión HTTP, se genera a su vez una conexión TCP/IP cuya negociación inicial es muy costosa.

Podría llegar a tener sentido utilizar websockets para que los clientes envien mensajes al servidor con la excusa de obtener la menor latencia posible, sin embargo, con la llegada de **HTTP/2** este problema se soluciona ya que esta versión del protocolo utiliza una sola conexion TCP.

### Websockets es más ligero que HTTP
Pensemos en una aplicación que genera una gran cantidad de mensajes, por ejemplo una aplicación que envie al servidor todos los movimientos del ratón.

Si utilizamos http además de la latencia de negociar todas las conexiones TCP, en cada mensaje estamos introduciendo un montón de información redundante en las cabezeras HTTP (versión, user-agent, content...) esa información extra no existe en los web sockets, donde podremos mandar mensajes que solamente contegan las coordenadas del ratón `{x: 123, y: 456}`. Es más, mediante websockets podríamos enviar directamente dos números en binario que representen las coordenadas ahorrando así gran cantidad de datos.

De nuevo, estos dos problemas están resueltos en **HTTP/2** que introduce frames binarios y compresión las cabeceras HTTP.

(hasta donde yo se sobre la compresión de cabeceras en HTTP2 que es muy poco, creo que en general siempre va a haber un pequeño overhead por culpa de las cabeceras respecto a los websockets. Por lo que incluso con HTTP2 si la aplicación va a enviar una cantidad loca de mensajes y somos estrictos con la volumen de tráfico generado, podría llegar a tener sentido utilizar wss)

### HTTP encaja bien con la infraestructura de internet.
Puede ser tentador implementar alguna operación como `recuperar las llamadas hechas cierto dia` mediante websockets con la excusa de la baja latencia. Aunque es cierto que mediante wss nos ahorramos muchas negociaciones con respecto a HTTP/1, hay que tener en cuenta que websockets no se beneficia de muchos de los protocolos y sistemas que ofrece la red. A pesar de que en teoría una petición http debería tener más latencia, en la realidad muchas veces esa petición estará cacheada en nuestro navegador o en algún elemento intermedio y la respuesta será inmediata.



## Conclusión.
Con HTTP/2 [funcionando en todos los navegadores actuales](https://caniuse.com/#search=http2) no parece que tenga mucho sentido utilizar websockets para la comunicación de cliente a servidor. No ocurre lo mismo en el caso contrario donde los websockets son una solución perfectamente valida y de baja latencia para enviar mensajes de servidor a cliente.

Es importante saber que existe una api llamada [Server-Sent Events (SSE)](https://hpbn.co/server-sent-events-sse/) pensada precisamente para enviar datos directamente desde el servidor hacia los clientes.



Quizá el problema que nos plateamos no tenga mucho sentido y la pregunta que nos debamos hacer sea **¿Cuando utilizar websockets y cuando utilizr SSE?** pero eso lo dejamos para otro post...


## Referencias

* [WebSockets](https://hpbn.co/websocket/)
* [HTTP/1](https://hpbn.co/websocket/)
* [HTTP/2](https://hpbn.co/http2/)
* [Server-Sent Events (SSE)](https://hpbn.co/server-sent-events-sse/ﬁ)
