---
layout: post
title:  "Campos de Galois"
date:   2016-11-06 14:42:29 +0100
categories: blog
description: "Explicación para programadores de los Campos de Galois, también llamados cuerpos finitos." 
math: true
---

Una explicación más detallada (en inglés) puede encontrar en [este video](https://www.youtube.com/watch?v=x1v2tX4_dkQ) sobre el que se basa este texto.

Un campo de Galois es una [estructura algebráica](https://es.wikipedia.org/wiki/Estructura_algebraica) utilizada entre otras muchas cosas en algoritmos como el [AES](https://es.wikipedia.org/wiki/Advanced_Encryption_Standard), o el [Reed-Solomon](https://es.wikipedia.org/wiki/Reed-Solomon). La intención de este artículo es dotar al lector de unos conocimientos mínimos que le permitan comprenderlos e implementarlos en algún lenguaje de programación.


## Estructuras algebraicas

Una estructura algebraica viene dada por una [tupla](https://es.wikipedia.org/wiki/Tupla) de elementos y un conjunto de operaciones aplicables a estos elementos.
Un ejemplo de estructura (<amp-mathml layout="container" inline data-formula="`E`"></amp-mathml>) podría ser la tupla formada por los números enteros (<amp-mathml layout="container" inline data-formula="$$\mathbb{Z}$$"></amp-mathml>) y el conjunto de operaciones formado por la suma (<amp-mathml layout="container" inline data-formula="$$+$$"></amp-mathml>) y la resta (<amp-mathml layout="container" inline data-formula="$$-$$"></amp-mathml>).

<amp-mathml layout="container" data-formula="$$  E = (\mathbb{Z}, \{+, -\})  $$]"></amp-mathml>




## Cuerpos

Son un tipo de estructura algebráica más estricta:

Las operaciones `adición` `sustracción` `multiplicación` y `división` se pueden realizar siempre que se cumplan las siguientes propiedades:

- Existe un [elemento neutro](https://es.wikipedia.org/wiki/Elemento_neutro) para la adición y la multiplicación.
- El orden en que se ejecuten *las operaciones* no altera el resultado. [(Propiedad Asociativa)](https://es.wikipedia.org/wiki/Asociatividad_(%C3%A1lgebra))
- El orden de los sumandos no altera la suma, o el orden de los factores no altera el producto. [(Propiedad Conmutativa)](https://es.wikipedia.org/wiki/Propiedad_conmutativa)
- El resultado de un número multiplicado por la suma de dos o más sumandos, es igual a la suma de los productos de cada sumando por ese número. [(Propiedad Distributiva)](https://es.wikipedia.org/wiki/Propiedad_distributiva)

## Cuerpos finitos
También llamados **campos de Galois** son un cuerpo definido sobre un conjunto finito de elementos donde el número de elementos (<amp-mathml layout="container" inline data-formula="$$n$$"></amp-mathml>) debe ser primo o una potencia de un número primo.


<amp-mathml layout="container" data-formula="$$ n = p^m $$"></amp-mathml>



Si <amp-mathml inline layout="container" data-formula="$$ m = 1 $$"></amp-mathml> estaremos ante un caso particular llamado campo primo.

### Campos primos

Conocidos como <amp-mathml inline layout="container" data-formula="$$GF(p^1)$$"></amp-mathml> (del ingles Galois Field) están formado por un conjunto de <amp-mathml inline layout="container" data-formula="$$p - 1$$"></amp-mathml> elementos y las operaciones de los cuerpos definidas como:

- Suma: Relizar la suma normalmente módulo p
- Resta: Realizar la resta normalmente módulo p
- Multiplicación: Realizar la multiplicación normalmente módulo p.
- División: Realizar la división normalmente módulo p.


#### Ejemplo

	GF(7)
	Elementos: {0,1,2,3,4,5,6}
	Operaciones: {+, -, *, /}

Suma:

	1 + 1 = 2 mod 7 = 2
	1 + 6 = 7 mod 7 = 0
	2 + 6 = 8 mod 7 = 1

Multiplicación:

	2 * 3 = 6 mod 7 = 6
	3 * 3 = 9 mod 7 = 2

Hasta aquí todo es bastante simple, pero ¿Qué pasa cuando <amp-mathml inline layout="container" data-formula="$$ m > 1 \; $$"></amp-mathml> ?

## Extension fields

Llamaremos extension field a un cuerpo finito donde el número de elementos es una potencia de un número primo y los elementos del conjunto son polinómios.

<amp-mathml layout="container" data-formula="$$ a_{m-1} X^{m-1} + ... + a_0 X + a_0 $$"></amp-mathml>


Donde <amp-mathml inline layout="container" data-formula="$$ a_i \in GF(p^m) \;\;\;\; $$"></amp-mathml>  

Pongamos como ejemplo el caso donde <amp-mathml inline layout="container" data-formula="$$ p = 2 \;\;\;\;$$"></amp-mathml> y <amp-mathml inline layout="container" data-formula="$$ m = 3 \; \; \; \; $$"></amp-mathml>

```
p = 2
m = 3
GF(p) = GF(2) = {0,1}
```

Sabemos que todos polinomios <amp-mathml inline layout="container" data-formula="$$ GF(2^3) \;$$"></amp-mathml> tendrán la forma:

<amp-mathml layout="container" data-formula="$$ ax^2 + ax + a $$"></amp-mathml>



y que a solamente puede tener dos posibles valores <amp-mathml inline layout="container" data-formula="$$ a_i \in \{0, 1\} $$"></amp-mathml>, por lo que la lista completa de polinomios que pertenecen al <amp-mathml inline layout="container" data-formula="$$GF(2^3) \; $$"></amp-mathml>   será:

<amp-mathml layout="container" data-formula="$$ 0x^2 + 0x + 0 $$"></amp-mathml>

<amp-mathml layout="container" data-formula="$$ 0x^2 + 0x + 1 $$"></amp-mathml>

<amp-mathml layout="container" data-formula="$$ 0x^2 + 1x + 0 $$"></amp-mathml>

<amp-mathml layout="container" data-formula="$$ 0x^2 + 1x + 1 $$"></amp-mathml>

<amp-mathml layout="container" data-formula="$$ 1x^2 + 0x + 0 $$"></amp-mathml>

<amp-mathml layout="container" data-formula="$$ 1x^2 + 0x + 1 $$"></amp-mathml>

<amp-mathml layout="container" data-formula="$$ 1x^2 + 1x + 0 $$"></amp-mathml>

<amp-mathml layout="container" data-formula="$$ 1x^2 + 1x + 1 $$"></amp-mathml>


Esta naturaleza permite representar los polinomios utilizando solamente el valor de sus coeficientes.


<amp-mathml layout="container" data-formula="$$ 000 $$"></amp-mathml>

<amp-mathml layout="container" data-formula="$$ 001 $$"></amp-mathml>

<amp-mathml layout="container" data-formula="$$ 010 $$"></amp-mathml>

<amp-mathml layout="container" data-formula="$$ 011 $$"></amp-mathml>

<amp-mathml layout="container" data-formula="$$ 100 $$"></amp-mathml>

<amp-mathml layout="container" data-formula="$$ 101 $$"></amp-mathml>

<amp-mathml layout="container" data-formula="$$ 110 $$"></amp-mathml>

<amp-mathml layout="container" data-formula="$$ 111 $$"></amp-mathml>


### Sumas y restas
Para sumar o restar polinomios en un <amp-mathml inline layout="container" data-formula="$$GF(p^n)$$"></amp-mathml> sólo hay que sumar/restar cada coeficiente módulo <amp-mathml inline layout="container" data-formula="$$p$$"></amp-mathml>. En los campos de Galois donde <amp-mathml inline layout="container" data-formula="$$ p=2  \;  \;$$"></amp-mathml> ambas operaciones son equivalentes a un [xor binario](https://es.wikipedia.org/wiki/Disyunci%C3%B3n_exclusiva).

	Ejemplos de suma de polinomios en GF(8)

	000 + 000 = 000
	111 + 001 = 110
	101 + 101 = 010


### Multiplicación
La multiplicación se vuelve más compleja ya que si se hace de forma habitual, obtendremos polinomios que están fuera del campo

<amp-mathml layout="container" data-formula="$$ (x^2 + x + 1) * (x^2 + 1) = x^4 + x^3 + x + 1 $$"></amp-mathml>

por lo que se hace necesario reducir el resultado módulo un polinomio <amp-mathml inline layout="container" data-formula="$$C(x) \; $$"></amp-mathml> que actuará como `elemento neutro`.


<amp-mathml layout="container" data-formula="$$ (x^2 + x + 1) * (x^2 + 1) \equiv (x^4 + x^3 + x + 1) \bmod C(x) $$"></amp-mathml>


La principal caracteristica de estos polinomios es que son irreducibles (no se pueden factorizar). También son conocidos como `primitivas` y se puede demostrar que para todo <amp-mathml inline layout="container" data-formula="$$GF(2^m) \;$$"></amp-mathml> existen múltiples primitivas.
