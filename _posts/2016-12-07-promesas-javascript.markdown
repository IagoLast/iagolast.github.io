---
layout: post
title:  "Promesas en Javascript"
date:   2016-11-07 16:00:00 +0100
categories: blog
---

Casi todo el mundo utiliza promesas en javascript y lo cierto es que han supuesto un gran avance para lidiar con la programación asíncrona, sin embargo, siguen teniendo una cara oculta que poca gente se atreve a investigar. En este artículo se pretenden tocar unos temas un poco más retorcidos para lograr comprender con profundidad el comportamiento de las promesas en javacript. Vamos a suponer que estamos llamando a una api REST que nos devuelve un Json con información meteorológica, esto es una operación asíncrona que simularemos con un setTimeout.

```javascript
 function httpGetWheather(){
  return new Promise(function (resolve, reject){
     setTimeout(function() {
       resolve('{"temperatura":10,"lluvia":0}');
     }, 200);
  });
}
```

Hasta aquí todo el mundo tiene claro los principios básicos de las promesas, permiten tratar con valores asíncronos utilizando la función `.then(onSuccess, onError)` que recibe dos parámetros: **onSuccess** El callback que será ejecutado cuando la promesa se resuelva. **onError** El callback que será ejecutado si la promesa devuelve un error.

## Método .then()

Todas las promesas tienen un método then, que reciben como parametro una función que será ejecutada en cuanto el valor de la promesa este disponible. Por lo tanto si en nuestro ejemplo queremos mostrar en la consola la respuesta de nuestra promesa simulada.

```javascript
 httpGetWheater().then(function(value){
  console.log('La promesa ha devuelto:' , value);
});
```

## El método then() devuelve otra promesa permitiendo que podamos encadenar múltiples promesas.

Nuestra promesa devuelve información meteorológica dentro de un string que contiene un objeto JSON. Imaginemos ahora que sólamente queremos mostrar la temperatura de forma que en un primer paso parseamos el json, y en un segundo paso, mostramos el resultado por consola. En el primer callback creamos un objeto json a partir del valor devuelto por la primera promesa y devolvemos la temperatura que se pasará como parámetro al siguiente then().

```javascript
httpGetWheather()
.then(function(jsonResponse) {
    var jsonObject = JSON.parse(jsonResponse);
    return jsonObject.temperatura; // Este valor se pasa al siguiente then!
})
.then(function(temperatura){
    console.log('La temperatura es: ' + temperatura);
});
```

## El valor que se devuelve dentro de un then puede ser una promesa.

De forma que javascript esperará a que se resuelva antes de ejecutar el siguiente then().

```javascript
httpGetWheather()
.then(function(value){
  console.log('La promesa1 ha devuelto : ' , value);
  return httpGetWheather();
})
.then(function(value) {
  console.log('La promesa2 devuelve: ', value);
});
```

Un ejemplo más complicado, recordemos que si se devuelve una promesa, no se ejecutará el then hasta tener el valor resuelto por la promesa anterior y que then() en si devuelve una promesa.

```javascript
 httpGetWheather()
.then(function(value){
  console.log('La promesa1 ha devuelto : ' , value);
  return httpGetWheather().then(function(jsonValue){
    return JSON.parse(jsonValue).temperatura;
   });
})
.then(function(value) {
  console.log('La promesa2 devuelve: ', value); // 10
});
```

## Si un callback no devuelve nada, se ejecuta la siguiente función

Por lo tanto si un then no devuelve nada se pasa al siguiente **SIN ESPERAR A QUE LA FUNCION ANTERIOR CONCLUYA!!** Esto es un error bastante común ya que mucha gente piensa que siempre se ejecutan en orden.

```javascript
 httpGetWheather()
.then(function(value){
  console.log('La promesa1 ha devuelto : ' , value);
  // NO SE ESPERA A QUE SE TERMINE!
  httpGetWheather().then(function(){console.log('Promesa intermedia');});
})
.then(function(value) {
  console.log('La promesa2 devuelve: ', value); // undefined
});
```

## Si no le pasas una función a un then, en lugar de fallar pasa al siguiente then.

Esto es un caso algo más raro, pero que esta bien conocer, dado el siguiente código:

```javascript
httpGetWheather()
.then(null)
.then(function(value) {
  console.log(value); // {"temperatura":10,"lluvia":0}
});
```

Podemos pensar que se loguea "undefined" ¿verdad?, pues no, se ejecuta el último then con el resultado de la primera promesa. conclusión: Pasar siempre funciones como parámetro a los then().

## Errores en promesas

Cuando una promesa no tiene éxito, se utilizan dos opciones **CASI** equivalentes. .then(onSuccess, onError) .catch(onError) Sin embargo hay que tener en cuenta algunos factores.

## Los errores dentro del fragmento de código asíncrono desaparecen!

En el siguiente caso, el error se maneja correctamente y será detectadp en la funcion catch.

```javascript
 function httpGetWheather(){
  return new Promise(function (resolve, reject){
     throw 'KABOOOOM';
     setTimeout(function() {
       resolve('{"temperatura":10,"lluvia":0}');
     }, 200);
  });
}

httpGetWheather()
.then(function(value) {
  console.log('La promesa devuelve: ', value);
})
.catch(function(err){
  console.log('Error! ' + err); // KABOOOOM
})
```

Que además es equivalente a:

```javascript
function httpGetWheather(){
  return new Promise(function (resolve, reject){
     throw 'KABOOOOM';
     setTimeout(function() {
       resolve('{"temperatura":10,"lluvia":0}');
     }, 200);
  });
}

httpGetWheather()
.then(function(value) {
  console.log('La promesa devuelve: ', value);
},function(err){
  console.log('Error: ' + err); // KABOOOOM
});
```

Sin embargo, si el error se produce dentro del código asíncrono no pasará por los error handlers.

```javascript
 function httpGetWheather(){
  return new Promise(function (resolve, reject){
     setTimeout(function() {
        throw 'KABOOOOM'; // Uncaught error
        resolve('{"temperatura":10,"lluvia":0}');
     }, 200);
  });
}

httpGetWheather()
.then(function(value) {
  console.log('La promesa devuelve: ', value);
},function(err){
  console.log('Error: ' + err); // nunca se ejecuta...
})
```

Esto se arregla, por ejemplo, poniendo un try catch que en caso de error llame explícitamente al reject callback

```javascript
 function httpGetWheather() {
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
httpGetWheather().then(function(value) {
    console.log('La promesa devuelve: ', value);
}).catch(function(err) {
    console.log('Error: ' + err); // KABOOOOM
});
```

## Si un then falla se ejecutará el siguiente catch, pero no se interrumpe la ejecución.

Es decir que si a continuación del catch ponemos otro then(), este se ejecutará normalmente con el resultado que devuelva el catch()

```javascript
 httpGetWheather()
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

```javascript
httpGetWheather()
.then(function(value) {
    return 'v1';
})
.catch(function(err) {
    return 'e1';
})
.then(function(value){
  console.log(value); // v1
})
```

Estos son solo algunos ejemplos de cosas que te pueden pasar jugando con promesas en javascript, muchas de ellas pueden resultar dificiles de debuggear o sea que mejor estar preparados.
