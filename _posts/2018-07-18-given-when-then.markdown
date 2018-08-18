---
layout: post
title:  "Given When Then"
date:   2018-07-17 000:00:00 +0100
categories: blog
description: "Una breve reflexión sobre como escribir tests descriptivos"
image: "https://blog.recruitloop.com/wp-content/uploads/2013/05/homer-simpson-instructions1.jpg"
---
<img 
    src="https://blog.recruitloop.com/wp-content/uploads/2013/05/homer-simpson-instructions1.jpg" 
    alt="No es bueno leer especificaciones con prisa" 
    style="max-height:400px; display:block; margin:30px auto;"/>


## La importancia de unos tests descriptivos

Por suerte cada cada día somos mas los desarrolladores convencidos de la importancia de tener tests automatizados en nuestro código, sin embargo todavía mucha gente sigue tratando el código de los tests como si fuese menos importante. 

No voy a entrar en temas sobre como organizar correctamente los tests, qué se debe probar o como debe ser un buen test de unidad. Simplemente voy a dar unos consejos básicos sobre como (en mi opinión), crear unos tests que además de garantizar que una parte de nuestro código funciona ayuden a los programadores a **entender la funcionalidad que están probando**.


## Given When Then

Hace ya algunos años Martin Fowler [un artículo](https://martinfowler.com/bliki/GivenWhenThen.html) en el que se basa este post.

En él sugiere dividir los tests en 3 etapas:

- **Given:** Describe el escenario base en el que se va a realizar el test
- **When:** Añade alguna condición específica al escenario.
- **Then:** Describe la estado final en el que debe quedar el programa.


En el mundo del javascript es común utilizar [test runners](https://medium.com/dailyjs/javascript-test-runners-benchmark-3a78d4117b4) que suelen ofrecer una funcion para definir un test y otra para agruparlos en bloques, en el caso de [Jasmine.js](https://jasmine.github.io/api/edge/global.html#describe) serían `it` y `describe` respectivamente.


## IRTD

- Intention revealing test description
- Es una forma de organizar los tests que ayuda a entender la funcionalidad
- Normalmente en tests de aceptacion
- Se define tambien para tests de unidad
- Normas:
    - A primer nivel indicamos el caso de uso/clase
    - De ser necesario presentamos el segundo nivel (la funcion si estamos probando una clase)
    - [precondiciones]
        - La primera va precedida de `when`.
        - Las siguientes van precedidas de `and`.
    - expectativas
        - Deben empezar por `should`
        - Las expectativas pueden acabar con una precondicion
            - > deberia ser mayor **cuando el parametro es un entero postivo**

- Asegurarse de cubrir todo el espectro de casos posibles
    - Cuando el numero es mayor que cero
    - Cuando el numero es menor que cer
    - QUe pasa cuando es cero?

Si aplicamos esta norma, nuestros tests son mucho más fáciles de leer y inmediatamente somos capaces de entender la función del sistema que esta siendo probado, por ejemplo un sistema de login:


```js
describe('Login:', () => {
    describe('when the username/password is invalid', () => {
        it('should return a descriptive error providing feedback', () => {});
    });

    describe('when the username/password is valid', () => {
        describe('and the user is disabled', () => {
            it('should return a descriptive error providing feedback', () => {});
            it('should update error logs', () => {});
        })

        describe('and the user is enabled', () => {
            it('should login the user', () => {});
            it('should update login logs', () => {});
        })
    });
});
```

Como podemos ver a continuación, al ejecutar los tests obtenemos un log muy explicito acerca de la intención del sistema.

```
Login:
    when the username/password is invalid
        ✔ should return a descriptive error providing feedback
    when the username/password is valid
        and the user is disabled
            ✔ should return a descriptive error providing feedback
            ✔ should update error logs
        and the user is enabled
            ✔ should login the user
            ✔ should update login logs
```


En el caso de tests unitarios, donde estemos probado una función o una clase podemos proceder de la siguiente forma:

Nombre de la clase -> .nombre del metodo probado -> 

```js
describe('Calendar:', () => {
    describe('.getDate', () => {
        describe('when called with no parameters', () => {
            it('should return the current date', () => {});
        });
    });
});
```


```js
describe('Calendar:', () => {
    describe('.getDate', () => {
        describe('when called with no parameters', () => {
            describe('and system language is english', () => {
                it('should return the current date', () => {});
            });

            describe('and system language is english', () => {
                it('should return the current date', () => {});
            });

        });
    });
});
```



## Referencias

[http://www.kgolev.com/revealing-code-intent-tests/](http://www.kgolev.com/revealing-code-intent-tests/)