---
layout: post
title:  "Promesas en Javascript"
date:   2016-11-07 16:00:00 +0100
categories: blog
---

Casi todo el mundo ha oido hablar de las promesas en javascript. Aunque han supuesto un gran avance para lidiar con la programación asíncrona poca gente se ha interesado por entenderlas en profundidad.

En este artículo se intenta comentar como se comportan las promesas javascript en algunos casos.

## Programación asíncrona

Antes de leer cualquier cosa sobre promesas, es importante tener claro lo que significa que una operación sea asíncrona y tener unas nociones básicas sobre como javascript gestiona este tipo de operaciones. Si tienes dudas, recomiendo que visites los siguientes enlaces primero:

- [Javascript Asíncrono, la guía definitiva](http://lemoncode.net/lemoncode-blog/2018/1/29/javascript-asincrono#el-modelo-de-javascript)
- [Video: Asincronía en javascript](https://www.youtube.com/watch?v=rgmej4Jx4WM)

## Creando promesas

Podemos crear una promesa utilizando el operador `new` sobre el objecto `Promise` pasando como único parametro una función que llamaremos "ejecutor". Esta función a su vez recibe dos parametros:
un callback que será ejecutado cuando la promesa se ha cumplido (resolve) y otro que será ejecutado cuando no (reject).
```js
const promise = new Promise(function executor(resolve, reject) { 
  // Some async operations here...
});
```

Un caso de uso típico para las promesas es poder ocultar los callbacks de una petición HTTP:

```js
function httpGet(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open("GET", url);
    xhr.onload = () => resolve(xhr.responseText);
    xhr.onerror = () => reject(xhr.statusText);
    xhr.send();
  });
}
```

Con la función anterior, podemos realizar peticiones HTTP encapsuladas en una promesa:

```js
const request = httpGet('https://jsonplaceholder.typicode.com/posts');
```


Hasta aquí todo el mundo suele tenerlo bastante claro. Una vez tenemos la promesa generada podemos tratar con valores asíncronos utilizando la función `then` que recibe dos parámetros: 

- **onSuccess:** El callback que será ejecutado cuando la promesa se resuelva.
- **onError:** El callback que será ejecutado si la promesa devuelve un error.

## Función then

Todas las promesas tienen una función then, que recibe como parametro **una función** que será ejecutada recibiendo como parámetro el resultado de la promesa en cuanto esté disponible. Por lo tanto si en nuestro ejemplo queremos mostrar en la consola la respuesta de nuestra petición HTTP simplemente pasaremos como parámetro la funcion `console.log`.

```js
httpGet('https://jsonplaceholder.typicode.com/posts').then(console.log);
```

Que para los despistados es exactamente lo mismo que:

```js
httpGet('https://jsonplaceholder.typicode.com/posts').then(response => console.log(response));
```


Llegados a este punto muchos lectores se preguntarán cual es la ventaja respecto a los callbacks y eso es lo que veremos a continuación:

### Las promesas se pueden encadenar de forma sencilla.

La función `then` devuelve a su vez una promesa permitiendo que sean encadenadas facilmente. Además el valor que se devuelve en `then` se pasa como parámetro al siguiente.

```js
httpGet('https://jsonplaceholder.typicode.com/posts')
 .then(() => 'hola') // Devolvemos hola
 .then(console.log) // Al siguiente then le llega como parámetro el "hola" devuelto en el paso anterior.
```

Volviendo al ejemplo, nuestra promesa se resuelve con un JSON con una lista de posts. Imaginemos ahora que sólamente queremos mostrar el primer post por consola:

En el primer callback creamos un objeto json a partir del valor devuelto por la primera promesa, en el segundo nos quedamos solamente con el primer post de la lista y en el tercero imprimimos por consola el resultado.

```js
httpGet('https://jsonplaceholder.typicode.com/posts')
  .then(JSON.parse)
  .then(jsonResponse => jsonResponse[0])
  .then(console.log);
```

### El valor que se devuelve dentro de un then puede ser una promesa.

De forma que javascript esperará a que se resuelva antes de ejecutar el siguiente then().

```js
httpGet('https://jsonplaceholder.typicode.com/posts/1')
.then(function(value){
  console.log('Post 1 : ' , value);
  return httpGet('https://jsonplaceholder.typicode.com/posts/2');
})
.then(function(value) {
  console.log('Post 2: ', value);
});
```

Es importante tener esto claro para evitar hacer cosas como la siguiente, que aun siendo equivalente es más dificil de leer y requiere más indentación.

```js
httpGet('https://jsonplaceholder.typicode.com/posts/1')
  .then(function(value){
    console.log('Post 1 : ' , value);
    return httpGet('https://jsonplaceholder.typicode.com/posts/2')
      .then(function(value) {
        console.log('Post 2: ', value);
      });
  })
```

### Si un callback no devuelve nada, se ejecuta la siguiente función

Por lo tanto si un then no devuelve nada se pasa al siguiente **SIN ESPERAR A QUE LA FUNCION ANTERIOR CONCLUYA!!** Esto es un error bastante común ya que mucha gente piensa que siempre se ejecutan en orden.

En el siguiente ejemplo el orden será: 

- Log 1
- Log 3
- Log 2

```js
httpGet('https://jsonplaceholder.typicode.com/posts/1')
  .then(function(value){
    console.log('Log 1 : ' , value);
    httpGet('https://jsonplaceholder.typicode.com/posts/2')
      .then(function(value) {
        console.log('Log 2: ', value);
      });
  })
  .then(value => {
    console.log('Log 3:', value); // undefined 
  });
```

El "Log 3" se ejecuta antes que el log dos porque el primer then no devuelve nada y por lo tanto no se espera a que se resuelvan las promesas.

### Si no le pasas una función a un then, en lugar de fallar pasa al siguiente then.

Esto es un caso algo más raro, pero que esta bien conocer, dado el siguiente código:

```js
httpGet('https://jsonplaceholder.typicode.com/posts/1')
.then(null)
.then(console.log); // post 1
```

Podemos pensar que se loguea "undefined" ¿verdad?, pues no, se ejecuta el último then con el resultado de la primera promesa.

## Errores en promesas

Cuando una promesa no tiene éxito, se utilizan dos opciones **CASI** equivalentes. .then(onSuccess, onError) .catch(onError) Sin embargo hay que tener en cuenta algunos factores.

### Los errores dentro del fragmento de código asíncrono desaparecen!

En el siguiente caso, el error se maneja correctamente y será detectado en la funcion catch.

```js
 function fakeHttpCall(){
  return new Promise(function (resolve, reject){
     throw 'KABOOOOM';
     setTimeout(function() {
       resolve('{"temperatura":10,"lluvia":0}');
     }, 200);
  });
}

fakeHttpCall()
.then(function(value) {
  console.log('La promesa devuelve: ', value);
})
.catch(function(err){
  console.log('Error! ' + err); // KABOOOOM
})
```

Que además es equivalente a:

```js
function fakeHttpCall(){
  return new Promise(function (resolve, reject){
     throw 'KABOOOOM';
     setTimeout(function() {
       resolve('{"temperatura":10,"lluvia":0}');
     }, 200);
  });
}

fakeHttpCall()
  .then(
    function(value) {
      console.log('La promesa devuelve: ', value);
    },
    function(err){
      console.log('Error: ' + err); // KABOOOOM
    });
```

Sin embargo, si el error se produce dentro del código asíncrono no pasará por los error handlers.

```js
 function fakeHttpCall(){
  return new Promise(function (resolve, reject){
     setTimeout(function() {
        throw 'KABOOOOM'; // Uncaught error
        resolve('{"temperatura":10,"lluvia":0}');
     }, 200);
  });
}

fakeHttpCall()
  .then(
    function(value) {
      console.log('La promesa devuelve: ', value);
    },
    function(err){
      console.log('Error: ' + err); // nunca se ejecuta...
    });
```

Esto se arregla, por ejemplo, poniendo un try catch que en caso de error llame explícitamente al reject callback

```js
 function fakeHttpCall() {
    return new Promise(function(resolve, reject) {
        setTimeout(function() {
            try {
                throw 'KABOOOOM';
            } catch (err) {
                reject(err);
            }
            resolve('{"temperatura":10,"lluvia":0}');
        }, 200);
    });
}
fakeHttpCall()
  .then(function(value) {
    console.log('La promesa devuelve: ', value);
  })
  .catch(function(err) {
    console.log('Error: ' + err); // KABOOOOM
  });
```

## Si un then falla se ejecutará el siguiente catch, pero no se interrumpe la ejecución.

Es decir que si a continuación del catch ponemos otro then(), este se ejecutará normalmente con el resultado que devuelva el catch()

```js
 fakeHttpCall()
  .then(function(value) {
    throw 'BOOM'
    return 'v1';
  })
  .catch(function(err) {
    return 'e1';
  })
  .then(function(value){
    console.log(value); // e1
  })
```

Y en caso de que se ejecuten correctamente, se ignora el catch.

```js
fakeHttpCall()
  .then(function(value) {
    return 'v1';
  })
  .catch(function(err) {
    return 'e1';
  })
  .then(function(value){
    console.log(value); // v1
  });
```

Estos son solo algunos ejemplos de cosas que te pueden pasar jugando con promesas en javascript, muchas de ellas pueden resultar dificiles de debuggear o sea que mejor estar preparados.
