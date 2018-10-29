---
layout: post
title:  "E2E testing con Travis,NightWatch y SauceLabs"
date:   2017-10-29 16:00:00 +0100
categories: blog
---

# E2E testing con Travis,NightWatch y SauceLabs

Repo de ejemplo: [https://github.com/IagoLast/e2e-travis-saucelabs](https://github.com/IagoLast/e2e-travis-saucelabs)

Cuando estoy programando un side-project en lo último que pienso es en los navegadores en los que va a funcionar mi código. Normalmente ya es un milagro que funcione en mi máquina! Pero en el mundo real si queremos ser medianamente serios lo mínimo que deberíamos dar es una **lista de navegadores compatibles** con nuestra app o librería.

En este artículo voy a hablar sobre como crear un entorno de integración continua con tests e2e utilizando [Travis](travis-ci.org), [Nightwatch](http://nightwatchjs.org/) y [SauceLabs](https://saucelabs.com/) 

## Aplicación de ejemplo.

La aplicación que vamos a probar, es una web-app que simplemente muestra `Hello World` utilizando características de `es6` como  [const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const) o [template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals).

Por simplicidad toda mi aplicación cabe en un simple `index.html`:

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>

</head>

<body>
    <script>
        const name = 'WORLD';
        const helloElement = document.createElement('h1');
        helloElement.innerText = `Hello ${name}`;
        document.body.appendChild(helloElement);
    </script>
</body>

</html>
```

Como se puede ver lo único que hace el esta "app" pintar un `h1` con el contenido `Hello WORLD` a el body.

## Sirviendo la app en local

[serve](https://github.com/zeit/serve) es una librería de `node` que permite crear un servidor web facilmente para probar una web en local.

Se puede instalar escribiendo:


    yarn add --dev serve


Con el comando `serve` se crea un servidor web que sirve el `index.html` de mi web-app en el puerto `5000`.

```

 $(npm bin)/serve

   ┌──────────────────────────────────────────────────┐
   │                                                  │
   │   Serving!                                       │
   │                                                  │
   │   - Local:            http://localhost:5000      │
   │   - On Your Network:  http://192.168.0.13:5000   │
   │                                                  │
   │   Copied local address to clipboard!             │
   │                                                  │
   └──────────────────────────────────────────────────┘

```

Efectivamente, abriendo chrome en `http://localhost:5000` puedo ver mi aplicación funcionando correctamente

<amp-img layout="responsive" width="500" height="341" src="https://iagolast.files.wordpress.com/2017/10/screen-shot-2017-10-29-at-21-54-31.png"></amp-img>



## Configurando Nightwatch.js

Lo habitual es que por cada cambio que haga en mi app, vuelva a abrir el navegador y comprobar a mano que todo funciona correctamente,
[Nightwatch.js](http://nightwatchjs.org/) es una herramienta que permite automatizar este proceso de forma que podemos dedicar ese tiempo a otras cosas más útiles.

Para instalar nightwatch basta con escribir

```
    yarn add --dev nightwatch
```

Para ejecutar nightwatch hace falta un archivo de configuración `nightwatch.conf.js`:

```javascript
// nightwatch.conf.js

module.exports = {
    src_folders: ['test'], // Array de carpetas donde se encuentran los tests
    test_settings: {
        default: {
            desiredCapabilities: {
                browserName: 'chrome', // Navegador que va a ser controlado
            }
        },
    }
};
```

 y una carpeta donde se encuentran los tests, en este caso solamente hay un test `test/basic.js`:

```javascript
// test/basic.js

module.exports = {
  basicTest: function (browser) { // Define a simple test
    // Navigate to web-app url
    browser.url('http://localhost:5000'); 
    // Wait the page to be loaded
    browser.waitForElementVisible('body', 1000);
    // Expect to have a h1 element
    browser.expect.element('h1').to.be.present;
    // Expect the element to have text Hello WORLD
    browser.expect.element('h1').text.to.equal('Hello WORLD');
    // Finish browser session.
    browser.end();
  },
};
```

Este test navega a la url donde se esta siriviendo la webapp, espera a que el contenido se cargue y comprueba que el `h1`esta presente con el tenxto `Hello WORLD`.

Para ejecutar las pruebas en local hace falta tener un servidor de selenium corriendo junto con un [Browser Driver](http://nightwatchjs.org/gettingstarted#browser-drivers-setup).

En mac se puede instalar selenium mediante:

    brew install selenium-server-standalone

Y el driver de chrome:

    brew install chromedriver


Para ejecutar selenium basta con escribir:

    selenium-server


Y con esto ya podemos ejecutar `nightwatch`, que controlará nuestro navegador a traves del browser driver que tengamos instalado y ejecutará los tests de forma automática.

```bash
MacBook-Pro-de-CARTO:e2e-travis-saucelabs iago$ $(npm bin)/nightwatch

[Basic] Test Suite
======================

Running:  basicTest
 ✔ Element <body> was visible after 45 milliseconds.
 ✔ Expected element <h1> to be present
 ✔ Expected element <h1> text to equal: "Hello WORLD"

OK. 3 assertions passed. (1.363s)
```

Como era de esperar los tests pasan.

## Sauce Labs

Sauce labs es una herramienta gratis para proyectos open source que nos permite ejecutar pruebas de `selenium` contra diferentes plataformas de forma moderadamente sencilla.



### Configurando usuario y contraseña

Para utilizar `sauce-labs` tendremos que crearnos una cuenta y obtener nuestro nombre de usuario y access-key. Una vez las tengamos tendremos que editar nuestro archivo de configuración de nightwatch añadiendo el puerto, el host y nuestro usuario y access-key.

Para **NO subir claves secretas a github** guardo mis variables de entorno en un archivo `.env` y las cargo mediante `dotenv` 


```javascript
require('dotenv').config(); // Carga la informacion secreta.

module.exports = {
    src_folders: ['test'], // Array de carpetas donde se encuentran los tests
    test_settings: {
        default: {
            desiredCapabilities: {
                browserName: 'chrome', // Navegador que va a ser controlado
            },
            selenium_port: 80, // Puerto en el que sauce sirve selenium
            selenium_host: 'ondemand.saucelabs.com', // Url de saucelabs
            username: process.env.SAUCE_USERNAME, // Nombre de usuario de saucelabs
            access_key: process.env.SAUCE_ACCESS_KEY, // Api key de sauce labs
        },
    }
};
```

### sauce-connect

Como nuestro servidor web esta en nuestra máquina local y no es accesible desde el exterior, necesitamos comunicar los servidores de 
sauce con nuestra web-app, para ello utilizaremos [sauce connect](https://wiki.saucelabs.com/display/DOCS/Sauce+Connect+Proxy), hay que bajarse el binario  y ejecutarlo pasandole como parametros usuario y api-key

    bin/sc -u <SAUCE_USERNAME> -k <SAUCE_ACCESS_KEY>

Si se ha ejecutado correctamente veremos que hay un tunel activo en el dashboard de `sauce`.


<amp-img layout="responsive" width="250px" height="277px" src="https://iagolast.files.wordpress.com/2017/10/screen-shot-2017-10-29-at-22-34-43.png"></amp-img>

Si ejecutamos de nuevo `nightwatch` este se ejecutará contra los servidores de sauce.


```bash

$(npm bin)/nightwatch

[Basic] Test Suite
======================

Running:  basicTest
 ✔ Element <body> was visible after 1013 milliseconds.
 ✔ Expected element <h1> to be present
 ✔ Expected element <h1> text to equal: "Hello WORLD"

OK. 3 assertions passed. (6.88s)


```

## Configurando diferentes navegadores

La gracia de todo esto es que con unos pequeños cambios en el archivo de configuracion de nightwatch, podemos
ejecutar nuestras pruebas contra diferentes navegadores, en este caso:

- Firefox55
- Internet explorer 11
- MS Edge 15

```javascript
require('dotenv').config(); // Carga la informacion secreta.

module.exports = {
    src_folders: ['test'], // Array de carpetas donde se encuentran los tests
    test_settings: {
        default: {
            desiredCapabilities: {
                browserName: 'chrome', // Navegador que va a ser controlado
            },
            selenium_port: 80, // Puerto en el que sauce sirve selenium
            selenium_host: 'ondemand.saucelabs.com', // Url de saucelabs
            username: process.env.SAUCE_USERNAME, // Nombre de usuario de saucelabs
            access_key: process.env.SAUCE_ACCESS_KEY, // Api key de sauce labs
        },
        // Añadimos diferentes navegadores para probar
        firefox55: {
            desiredCapabilities: {
                browserName: 'firefox',
                version: 55,
            }
        },
        ie11: {
            desiredCapabilities: {
                browserName: 'internet explorer',
                version: 11
            }
        },
        edge15: {
            desiredCapabilities: {
                browserName: 'MicrosoftEdge',
                version: 15,
            }
        },
    }
};
```

Para ejecutar las pruebas, debemos pasarle el nombre de los entornos al parametro `env` de nightwatch

```bash
$(npm bin)/nightwatch --env default,ie11,edge15,firefox55
Started child process for: default environment 
Started child process for: ie11 environment 
Started child process for: edge15 environment 
Started child process for: firefox55 environment 

  >> default environment finished.  


  >> firefox55 environment finished.  


  >> ie11 environment finished.  


  >> edge15 environment finished.  

 default   [Basic] Test Suite
======================
 default   
 default   Results for:  basicTest
 default   ✔ Element <body> was visible after 1065 milliseconds.
 default   ✔ Expected element <h1> to be present
 default   ✔ Expected element <h1> text to equal: "Hello WORLD"
 default   OK. 3 assertions passed. (7.634s)
 default   
 ie11   [Basic] Test Suite
======================
 ie11   
 ie11   Results for:  basicTest
 ie11   ✔ Element <body> was visible after 1094 milliseconds.
 ie11   ✖ Expected element <h1> to be present - element was not found  - expected "present" but got: "not present"
 ie11       at Object.basicTest (/Users/iago/Workspace/personal/e2e-travis-saucelabs/test/basic.js:8:20)
    at _combinedTickCallback (internal/process/next_tick.js:67:7)
 ie11   FAILED:  1 assertions failed and 1 passed (17.289s)
 ie11   
 ie11    _________________________________________________
 ie11   TEST FAILURE:  1 assertions failed, 1 passed. (17.416s)
 ie11    ✖ basic
 ie11   - basicTest (17.289s)
 ie11      Expected element <h1> to be present - element was not found  - expected "present" but got: "not present"
 ie11          at Object.basicTest (/Users/iago/Workspace/personal/e2e-travis-saucelabs/test/basic.js:8:20)
       at _combinedTickCallback (internal/process/next_tick.js:67:7)
 ie11   
 edge15   [Basic] Test Suite
======================
 edge15   
 edge15   Results for:  basicTest
 edge15   ✔ Element <body> was visible after 1183 milliseconds.
 edge15   ✔ Expected element <h1> to be present
 edge15   ✔ Expected element <h1> text to equal: "Hello WORLD"
 edge15   OK. 3 assertions passed. (20.673s)
 edge15   
 firefox55   [Basic] Test Suite
======================
 firefox55   
 firefox55   Results for:  basicTest
 firefox55   ✔ Element <body> was visible after 1039 milliseconds.
 firefox55   ✔ Expected element <h1> to be present
 firefox55   ✔ Expected element <h1> text to equal: "Hello WORLD"
 firefox55   OK. 3 assertions passed. (11.596s)
 firefox55   

```
En menos de 12 segundos, hemos probado nuestra app en 4 navegadores diferentes y **internet explorer 10 ha fallado** por que entre otras cosas no tiene soporte para los string literals que utilizamos en nuestra web-app.

## Notificando a sauce el resultado

En el dashboard de sauce labs podemos ver nuestros tests en los 4 navegadores sin embargo no tenemos ningun tipo de feedback acerca de su han sido exitosos o no.

<amp-img layout="responsive" width="500" height="192" src="https://iagolast.files.wordpress.com/2017/10/screen-shot-2017-10-29-at-22-50-41.png"></amp-img>


Se puede utilizar la [REST API](https://wiki.saucelabs.com/display/DOCS/Job+Methods) de saucelabs para actualizar la información de los jobs (tests) desde nightwatch. Para ello vamos a crear un pequeño snippet llamado `sauce-feedback.js`


```javascript
var request = require('request');

function uploadSauceResults(browser, done) {
  // Finish browser session;
  browser.end();

  var user = browser.options.username;
  var key = browser.options.accessKey;
  var jobId = browser.sessionId;
  var passed = browser.currentTest.results.failed === 0;

  if (user && key && jobId) {
    var url = 'https://saucelabs.com/rest/v1/' + user + '/jobs/' + jobId;
    return request.put({
      url: url,
      auth: { username: user, password: key },
      headers: { 'content-type': 'application/json' },
      body: JSON.stringify({ passed: passed })
    }, done);
  } else {
    console.warn('No user/key/jobId provided.');
    done();
  }
}

module.exports = uploadSauceResults;
```

Y lo vamos a utilizar en nuestro `basic.js` test.

```javascript
// basic.js
const after = require('./sauce-feedback');
module.exports = {
    basicTest: function (browser) { // Define a simple test
        // Navigate to web-app url
        browser.url('http://localhost:5000');
        // Wait the page to be loaded
        browser.waitForElementVisible('body', 1000);
        // Expect to have a h1 element
        browser.expect.element('h1').to.be.present;
        // Expect the element to have text Hello WORLD
        browser.expect.element('h1').text.to.equal('Hello WORLD');
    },
    after: after,
};
```

Si vemos ahora el dashboard despues de ejecutar los tests, observamos los ticks verdes. (Por algun motivo que desconozco IE se muestra como completado en lugar de "error", pero algo es algo! )

<amp-img layout="responsive" width="500" height="142" src="https://iagolast.files.wordpress.com/2017/10/screen-shot-2017-10-29-at-22-58-23.png"></amp-img>


## Integración con Travis.

Sería genial poder correr estos tests en nuestro sistema de integración continua, para probar de forma automática cada PR/commit.

Para ello crearemos un archivo `travis.yml`

```yaml
language: node_js

node_js:
  - 8

cache: yarn

addons:
  sauce_connect: true

before_script:
  - yarn serve &
```

Por defecto travis utiliza los npm scripts para la CI por lo que habra que actualizar el `package.json` y añadir dos comandos, `test` y `serve` 

```javascript
// package.json

"scripts": {
    "test": "nightwatch --env default,ie11,edge15,firefox55",
    "serve": "serve"
}
```

El addon `sauce_connect` requiere que esten definidas en travis las variables de entorno para el `SAUCE_ USERNAME` y el `SAUCE_ACCESS_KEY` la forma más sencilla es [definirlas en el cliente web](https://docs.travis-ci.com/user/environment-variables/#Defining-Variables-in-Repository-Settings)

También tenemos que actualizar el nighwatch.conf.js con la informacion del `build` y el `tunel-id`.


```javascript
// nightwatch.conf.js
const TRAVIS_JOB_NUMBER = process.env.TRAVIS_JOB_NUMBER; // Variable de entorno definida automaticamente por travis
require('dotenv').config();

module.exports = {
    src_folders: ['test'],
    test_settings: {
        default: {
            desiredCapabilities: {
                browserName: 'chrome',
                build: `build-${TRAVIS_JOB_NUMBER}`, // <----- importante para travis
                'tunnel-identifier': TRAVIS_JOB_NUMBER, // <----- importante para travis
            },
            selenium_port: 80,
            selenium_host: 'ondemand.saucelabs.com',
            username: process.env.SAUCE_USERNAME,
            access_key: process.env.SAUCE_ACCESS_KEY,
        },
        firefox55: {
            desiredCapabilities: {
                browserName: 'firefox',
                version: 55,
            }
        },
        ie11: {
            desiredCapabilities: {
                browserName: 'internet explorer',
                version: 11
            }
        },
        edge15: {
            desiredCapabilities: {
                browserName: 'MicrosoftEdge',
                version: 15,
            }
        },
    }
};
``` 

Con esto travis probara automaticamente cada commit o PR de la rama maste

<amp-img layout="responsive" width="499" height="32" src="https://iagolast.files.wordpress.com/2017/10/screen-shot-2017-10-29-at-23-20-07.png"></amp-img>

En este caso nos avisa de que los tests están fallando en IE10 por lo que voy a reescribir la webapp para que funcione en todos los navegadores.


```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>

</head>

<body>
    <script>
        var name = 'WORLD'; // No se usa const
        var helloElement = document.createElement('h1');
        helloElement.innerText = 'Hello ' + name; // No se usan string literals!
        document.body.appendChild(helloElement);
    </script>
</body>

</html>
``` 


Tras estos cambios vuelvo a ejecutar mis pruebas en local:

```bash
 yarn test
yarn run v1.2.1
$ nightwatch --env default,ie11,edge15,firefox55
Started child process for: default environment 
Started child process for: ie11 environment 
Started child process for: edge15 environment 
Started child process for: firefox55 environment 

  >> default environment finished.  


  >> firefox55 environment finished.  


  >> ie11 environment finished.  


  >> edge15 environment finished.  

 default   [Basic] Test Suite
======================
 default   
 default   Results for:  basicTest
 default   ✔ Element <body> was visible after 1009 milliseconds.
 default   ✔ Expected element <h1> to be present
 default   ✔ Expected element <h1> text to equal: "Hello WORLD"
 default   OK. 3 assertions passed. (7.259s)
 default   [Sauce Feedback] Test Suite
===============================
 default   
 default   OK. 3  total assertions passed. (8.641s)
 ie11   [Basic] Test Suite
======================
 ie11   
 ie11   Results for:  basicTest
 ie11   ✔ Element <body> was visible after 1086 milliseconds.
 ie11   ✔ Expected element <h1> to be present
 ie11   ✔ Expected element <h1> text to equal: "Hello WORLD"
 ie11   OK. 3 assertions passed. (13.08s)
 ie11   [Sauce Feedback] Test Suite
===============================
 ie11   
 ie11   OK. 3  total assertions passed. (14.456s)
 edge15   [Basic] Test Suite
======================
 edge15   
 edge15   Results for:  basicTest
 edge15   ✔ Element <body> was visible after 1013 milliseconds.
 edge15   ✔ Expected element <h1> to be present
 edge15   ✔ Expected element <h1> text to equal: "Hello WORLD"
 edge15   OK. 3 assertions passed. (20.44s)
 edge15   [Sauce Feedback] Test Suite
===============================
 edge15   
 edge15   OK. 3  total assertions passed. (21.878s)
 firefox55   [Basic] Test Suite
======================
 firefox55   
 firefox55   Results for:  basicTest
 firefox55   ✔ Element <body> was visible after 1124 milliseconds.
 firefox55   ✔ Expected element <h1> to be present
 firefox55   ✔ Expected element <h1> text to equal: "Hello WORLD"
 firefox55   OK. 3 assertions passed. (11.573s)
 firefox55   [Sauce Feedback] Test Suite
===============================
 firefox55   
 firefox55   OK. 3  total assertions passed. (12.949s)
✨  Done in 22.57s.
```
Y tal y como esperabamos, la web app funciona perfectamente en los 4 navegadores.

<amp-img layout="responsive" width="500" height="350" src="https://iagolast.files.wordpress.com/2017/10/screen-shot-2017-10-30-at-20-57-51.png"></amp-img>

## Resumen

Hemos montado un entorno que nos permite probar aplicaciones web de forma automática en múltiples navegadores. SauceLabs nos permite también elegir las plataformas sobre las que corren esos navegadores (windows, osx) e incluso permite dispositivos móviles.

Ese entorno se puede configurar junto a travis para probar de forma automática cada commit o Pull request de forma individual.


## Referencias

- [https://saucelabs.com/](https://saucelabs.com/)
- [http://nightwatchjs.org/](http://nightwatchjs.org/)
- [https://docs.travis-ci.com/user/sauce-connect/](https://docs.travis-ci.com/user/sauce-connect/)
- [https://medium.com/@mikaelberg/zero-to-hero-with-end-to-end-tests-using-nightwatch-saucelabs-and-travis-e932c8deb695](https://medium.com/@mikaelberg/zero-to-hero-with-end-to-end-tests-using-nightwatch-saucelabs-and-travis-e932c8deb695)