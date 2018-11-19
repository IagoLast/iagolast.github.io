---
layout: post
title:  "Webmercator y javascript"
date:   2018-02-28 09:00:00 +0100
categories: blog
description: "Implementando y entendiendo la proyección de mercator en javascript."
image: "https://raw.githubusercontent.com/IagoLast/iagolast.github.io/master/static/2018_2_28.png"
math: true
---

<a href="https://bl.ocks.org/iagolast/d6575a79fa279f450947244bba81f098" target="_blank">
    <img 
        src="https://gist.githubusercontent.com/IagoLast/d6575a79fa279f450947244bba81f098/raw/45fec9fe91c3535e99c60ae8dc460d4e850ba917/map.png" 
        alt="Ejemplo de mapa dibujado en javascript" 
        style="max-height:400px; display:block; margin:30px auto;">
</a>



Una de las principales ventajas de dar una charla es que te obliga a documentarte y a aprender cosas sobre el tema. Preparando una charla sobre mapas y javascript y tras leer un poquito sobre proyecciones geográficas no me he podido resistir a <a href="https://bl.ocks.org/iagolast/d6575a79fa279f450947244bba81f098" target="_blank"> intentar pintar un mapa desde cero </a> y escribir sobre ello.

## Proyecciones

Según la [wikipedia](https://es.wikipedia.org/wiki/Proyecci%C3%B3n_(matem%C3%A1ticas)) una proyección es un [mapeo](https://es.wikipedia.org/wiki/Funci%C3%B3n_matem%C3%A1tica) [idempotente](https://es.wikipedia.org/wiki/Idempotencia). Dicho en castellano, es una función que nos sirve para transformar puntos de una esfera (la tierra) en puntos de un plano cartesiano (un mapa!).

Unas proyecciones muy sencillas de entender son las [proyecciones cilíndricas](https://es.wikipedia.org/wiki/Proyecci%C3%B3n_cil%C3%ADndrica_equivalente) como por ejemplo la proyección cilíndrica equivalente.

<img src="http://mathworld.wolfram.com/images/eps-gif/CylindricalProjection3D_700.gif" alt="trigonometria de proyecciones" style="display:block; margin:30px auto;">


En esta proyección tenemos que imaginar que rodeamos la esfera que queremos proyectar con un folio gigante, la anchura de ese folio es igual al diámetro de la esfera en el ecuador ($$2 \cdot \pi \cdot r$$) y la altura del folio será $$2 \cdot r$$.

- El radio de la tierra se representa como $$r$$, para escalar el tamaño de la proyección solamente hay que cambiar su valor.
- La longitud ($$\lambda$$) representa el ángulo que forma el punto que queremos proyectar respecto al meridiano que tomamos como origen.
- La latitud ($$\phi$$) representa el ángulo que forma el punto con el ecuador.


Los puntos se proyectan de forma paralela al ecuador desde el eje de la esfera, dando unas ecuaciones relativamente sencillas.

$$ x = \frac{2 \pi r \lambda}{360} $$

$$ y = r \sin(\phi) $$

Para calcular la $$x$$ simplemente aplicamos [la formula para saber la longitud del arco](https://es.wikipedia.org/wiki/Longitud_de_arco) de la longitud en el ecuador.

Para calcular la $$y$$ aplicamos [trigonometría básica](https://es.wikipedia.org/wiki/Seno_(trigonometr%C3%ADa)) y multiplicamos el radio de la esfera por el seno de la latitud.

El problema de esta proyección es que distorsiona bastante las formas de los paises y los ángulos en el plano no son fieles a los ángulos en la tierra.


<img src="https://upload.wikimedia.org/wikipedia/commons/0/01/Lambert-cylindrical-equal-area-projection.jpg" 
    alt="La proyección cilindrica equivalente distorsiona los ángulos"
    style="max-height:250px; display:block; margin:30px auto;">


## Mercator 

Esto de los ángulos puede parecer una tontería pero en el siglo XVI hacía muy complicado para los navegantes seguir los rumbos de los mapas, por ello en 1569 [Gerardus Mercator](https://es.wikipedia.org/wiki/Gerardus_Mercator) ideo una proyección que conservaba los ángulos de la esfera en el plano.

Mercator es un tipo de [proyección cilíndrica](https://es.wikipedia.org/wiki/Proyecci%C3%B3n_cartogr%C3%A1fica#Proyecci%C3%B3n_cil%C3%ADndrica) y hoy en día es la más utilizada en mapas web. A continuación vamos a explicar como calcular las coordenadas $$(x, y)$$ de la pantalla a partir de una latitud y una longitud usando esta proyección.

### Longitud (x)

La longitud se transforma en la componente $$x$$ de nuestro mapa y es bastante facil de entender:

Como estamos envolviendo la esfera en un cilindro el ecuador tendrá exactamente la mísma longitud que el eje horizontal de nuestro mapa,

Para obtener la coordenada x en el plano cartesiano podemos aplicar una sencilla regla de 3: 

Si 360º se corresponden a $$2 \pi r $$, $$\lambda$$ se corresponden a $$x$$ por lo que podemos deducir:


$$ x = \frac{2 \cdot \pi \cdot r \cdot \lambda}{360} $$


### Latitud (y)

Por desgracia calcular la latitud no es tan sencillo y requiere unas definiciones previas.

<img
    src="https://upload.wikimedia.org/wikipedia/commons/thumb/d/d0/CylProj_infinitesimals2.svg/800px-CylProj_infinitesimals2.svg.png" 
    alt="trigonometria de proyecciones"
    style="display:block; margin:30px auto;">

La figura $$a$$ muestra un punto $$P$$ en la tierra con una longitud $$\lambda$$ y latitud $$\phi$$ y un punto Q cuya longitud es $$\lambda + \delta \lambda$$ y latitud $$ \phi + \delta \phi $$. Recordemos que $$\delta$$ denota incremento o desplazamiento.

La figura $$b$$ representa la proyección de la figura $$a$$ en el plano cartesiano donde el punto $$P'$$ tiene coordenadas $$(x, y)$$ y el punto $$ Q'$$ tiene coordenadas $$(x + \delta x, y + \delta y) $$.

<img 
    src="https://astronavigationdemystifieddotcom.files.wordpress.com/2012/06/diag15-mod.jpg"
    alt="trigonometria de proyecciones"
    style="max-height:200px; display:block; margin:30px auto;">

La longitud del arco $$MQ$$ se calcula aplicando [la fórmula para calcular arcos](https://es.wikipedia.org/wiki/Arco_(geometr%C3%ADa)#Relaci%C3%B3n_entre_arco,_radio_y_%C3%A1ngulo) sobre el meridiano en el que nos encontremos. Recordemos que todos los meridianos tienen radio $$r$$.

$$ MQ = r  \delta \phi $$

Para calcular la longitud del arco $$PM$$ aplicamos de nuevo la fórmula para calcular arcos pero corrigiendo el radio $$r$$ según el coseno de la latitud a la que nos encontremos dado que como se ve en la imagen anterior el radio de los paralelos no es igual al radio del ecuador ($$r$$) sino que varía según la latitud.


$$ PM = r \cos (\phi) \delta \lambda $$


### Factores de escala
El factor de escala en un paralelo se denota $$k$$ y el factor de escala de un meridiano se denota $$ h $$ y nos dicen cómo transformar un punto de la esfera en un punto del plano.

$$ k = \frac{P'M'}{PM} = \frac{\delta x}{r \cos (\phi) \delta \lambda} $$

$$ h = \frac{M'Q'}{MQ} = \frac{\delta y}{r \delta \phi} $$

En el apartado de longitud se explico que el incremento de $$x$$ solamente depende del incremento de la longitud por lo tanto: 

$$  \delta x = r \delta \lambda $$

Al sustituirlo en la ecuación de $$k$$ nos da que $$k$$ es igual a la [secante](https://es.wikipedia.org/wiki/Secante_(trigonometr%C3%ADa)) de la latitud.

$$  k = \frac{\delta x}{r \cos (\phi) \delta \lambda} = \frac{1}{\cos(\phi)} = \sec(\phi) $$

Mercator exige preservar las formas de los polígonos, por lo tanto los factores de escala deben de ser iguales $$ k = h $$

$$ h = \frac{\delta y}{r \delta \phi} = sec(\phi) = k $$

Podemos llamar $$y'$$ al incremento de $$x$$ respecto al incremento de la latitud $$\phi$$  y realizar un cambio de varaible.

$$ h = \frac{y'(\phi)}{r} = sec(\phi) = k$$


Si despejamos $$y'$$ nos da que

$$ y'(\phi)= r \cdot sec(\phi) $$ 


Y $$y'$$ representa el cambio de $$y$$ respecto a $$\phi$$ o dicho de otra forma: su [derivada](https://es.wikipedia.org/wiki/Derivada) respecto a la latitud y por lo tanto la fórmula de $$y$$  se obtiene integrando $$y'$$

$$ y = \int y' $$

<small> (Igual que al integrar la aceleración (incremento de la velocidad respecto al tiempo) obtenemos la velocidad, al integrar la fórmula de $$y'$$ obtenemos la fórmula de $$y$$)</small>

Buscando en una [tabla de integrales trigonométricas](https://es.wikipedia.org/wiki/Anexo:Integrales_de_funciones_trigonom%C3%A9tricas#Integrales_que_contienen_sec) podemos saber la integral de la secante.

$$ y(\phi) = r \cdot \ln(\sec(\phi) + \tan(\phi)) $$


Quedando finalmente las formulas de x e y

$$ x(\lambda) = \frac{2 \cdot \pi \cdot r \cdot \lambda}{360} $$

$$ y(\phi) = r \cdot \ln(\sec(\phi) + \tan(\phi)) $$

### Implementación en Javascript


Como la latitud y la longitud están en grados y las operacioines trigonométricas en javascript aceptan radianes, hay que [pasar de grados a radiantes](https://es.wikipedia.org/wiki/Radi%C3%A1n#Conversiones_entre_grados_y_radianes) antes de realizar operaciones trigonométricas!

$$ x(lon) = \frac{2 \cdot \pi \cdot r \cdot lon}{360} $$

$$ y(lat) = r \cdot \ln(\sec( \frac{lat * \pi}{180}) + \tan(\frac{lat * \pi}{180})) $$


Además, cómo en javascript no hay funcion secante, vamos a reescribir la proyección de la latitud en función del coseno aplicando transformaciones trigonométricas. Recordemos que la secante es la inversa del coseno y la tangente es seno entre coseno.

$$ y(\phi) = r \cdot \ln( \frac{1}{\cos(\phi)} + \frac{\sin(\phi)}{cos(\phi)}) $$

Y por lo tanto:


$$ y(\phi) = r \cdot \ln( \frac{1+\sin(\phi)}{\cos(\phi)}) $$

Si queremos eliminar el coseno para solo calcular el seno:

$$ y(\phi) = \frac{r}{2} \cdot \ln( \frac{1+\sin(\phi)}{1 - \sin(\phi)}) $$

Quedando finalmente:

$$ x(lon) = \frac{2 \cdot \pi \cdot r \cdot lon}{360} $$

$$ y(lat) = \frac{r}{2} \cdot \ln( \frac{1+\sin(\frac{lat * \pi}{180})}{1 - \sin(\frac{lat * \pi}{180})}) $$


```javascript
const MAP_SIZE = 1280; // The size of the map in pixels!

const R = MAP_SIZE / (2 * Math.PI);
const D = R * 2 * Math.PI;

function project([lon, lat]) {
    const sinlat = Math.sin(lat * Math.PI / 180.0);

    const x = D * lon / 360.0;
    const y = R / 2 * Math.log((1 + sinlat) / (1 - sinlat));

    return { x: (D / 2 + x), y: (D - (D / 2 + y)) }
}
```

Podeis ver un ejemplo completo en [este enlace](https://bl.ocks.org/iagolast/d6575a79fa279f450947244bba81f098).


## Referencias

- [http://laaventuradelaciencia.blogspot.com.es/2011/04/la-proyeccion-de-mercator.html](http://laaventuradelaciencia.blogspot.com.es/2011/04/la-proyeccion-de-mercator.html)
- [https://en.wikipedia.org/wiki/Mercator_projection](https://en.wikipedia.org/wiki/Mercator_projection)