---
layout: post
title:  "Campos de Galois"
date:   2016-11-06 14:42:29 +0100
categories: jekyll update
---

Una explicación más detallada (en inglés) puede encontrar en [este video](https://www.youtube.com/watch?v=x1v2tX4_dkQ) sobre el que se basa este texto.

Un campo de Galois es una [estructura algebráica](https://es.wikipedia.org/wiki/Estructura_algebraica) utilizada entre otras muchas cosas en algoritmos como el [AES](https://es.wikipedia.org/wiki/Advanced_Encryption_Standard), o el [Reed-Solomon](https://es.wikipedia.org/wiki/Reed-Solomon). La intención de este artículo es dotar al lector de unos conocimientos mínimos que le permitan comprenderlos e implementarlos en algún lenguaje de programación.


## Estructuras algebraicas

Una estructura algebraica viene dada por una [tupla](https://es.wikipedia.org/wiki/Tupla) de elementos y un conjunto de operaciones aplicables a estos elementos.

$$ E = (\mathbb{Z}, \{+, -\}) $$

Un ejemplo de estructura ($$E$$) podría ser los números enteros ($$\mathbb{Z}$$) como túpla, y la suma ($$+$$) y la resta ($$-$$) como operaciones.

## Cuerpos

Son un tipo de estructura algebráica más estricta:

Las operaciones `adición` `sustracción` `multiplicación` y `división` se pueden realizar siempre que se cumplan las siguientes propiedades:

- Existe un **elemento neutro** para la adición y la multiplicación:
- El orden en que se ejecuten *las operaciones* no altera el resultado. [(Propiedad Asociativa)](https://es.wikipedia.org/wiki/Asociatividad_(%C3%A1lgebra))
- El orden de los sumandos no altera la suma, o el orden de los factores no altera el producto. [(Propiedad Conmutativa)](https://es.wikipedia.org/wiki/Propiedad_conmutativa)
- El resultado de un número multiplicado por la suma de dos o más sumandos, es igual a la suma de los productos de cada sumando por ese número. [(Propiedad Distributiva)](https://es.wikipedia.org/wiki/Propiedad_distributiva)

## Cuerpos finitos
También llamados **campos de Galois** son un cuerpo definido sobre un conjunto finito de elementos donde el número de elementos ($$n$$) debe ser primo o una potencia de un número primo.

$$ n = p^m $$


Si $$m=1$$ estaremos ante un caso particular llamado campo primo.

### Campos primos

Conocidos como $$GF(p^1)$$ (del ingles Galois Field) están formado por un conjunto de $$p-1$$ elementos y las operaciones de los cuerpos definidas como:

- Suma: Relizar la suma normalmente módulo p
- Resta: Realizar la resta normalmente módulo p
- Multiplicación: Realizar la multiplicación normalmente módulo p.
- División: Realizar la división normalmente módulo p.


#### Ejemplo $$GF(7)$$

	GF(7)
	Elementos: {0,1,2,3,4,5,6}
	Operaciones: {+, -, x, /}

Suma:

	1 + 1 = 2 mod 7 = 2
	1 + 6 = 7 mod 7 = 0
	2 + 6 = 8 mod 7 = 1

Multiplicación:

	2 * 3 = 6 mod 7 = 6
	3 * 3 = 9 mod 7 = 2

Hasta aquí todo es bastante simple, el problema es cuando $$m>1$$.

## Extension fields

Es un cuerpo finito donde el número de elementos es una potencia de un número primo y los elementos del conjunto son polinómios:

$$ a_{m-1} X^{m-1} + ... + a_0 X + a_0 $$

Donde $$ a_i \in GF(p^m) $$

Para acompañar las explicaciones con ejemplos, tomaremos el caso concreto donde $$p=2$$  $$m=3$$.

#### Sea $$GF(2^3)$$ donde

	p = 2
	m = 3
	GF(p) = GF(2) = {0,1}

Todos polinomios de $$GF(2^3)$$ tendrán la forma:

$$ ax^2 + ax + a $$

Donde a puede tener dos posibles valores $$ a_i \in \{0, 1\} = GF(2) = GF(p)$$

Es decir que la lista completa de polinomios que pertenecen al $$GF(2^3)$$ será:

$$ 0x^2 + 0x + 0 $$

$$ 0x^2 + 0x + 1 $$

$$ 0x^2 + 1x + 0 $$

$$ 0x^2 + 1x + 1 $$

$$ 1x^2 + 0x + 0 $$

$$ 1x^2 + 0x + 1 $$

$$ 1x^2 + 1x + 0 $$

$$ 1x^2 + 1x + 1 $$

Esta naturaleza permite representar los polinomios utilizando números binarios para indicar el valor de sus
coeficientes.

$$ 000 $$

$$ 001 $$

$$ 010 $$

$$ 011 $$

$$ 100 $$

$$ 101 $$

$$ 110 $$

$$ 111 $$


### Sumas & resta
Para sumar o restar polinomios en un $$GF(p^n)$$ sólo hay que sumar/restar cada coeficiente módulo $$p$$.

En los campos de Galois donde $$p=2$$ ambas operaciones son equivalentes a un [xor binario](https://es.wikipedia.org/wiki/Disyunci%C3%B3n_exclusiva).

	Ejemplos de suma de polinomios en GF(8)

	000 + 000 = 000
	111 + 001 = 110
	101 + 101 = 010


### Multiplicación
La multiplicación se vuelve más compleja ya que si se hace de forma habitual, obtendremos polinomios que están fuera del campo

$$ (x^2 + x + 1) * (x^2 + 1) = x^4 + x^3 + x + 1 $$

por lo que se hace necesario reducir el resultado módulo un polinomio $$C(x)$$ que actuará como `elemento neutro`.

$$ (x^2 + x + 1) * (x^2 + 1) \equiv (x^4 + x^3 + x + 1) \bmod C(x) $$

La principal caracteristica de estos polinomios es que son irreducibles (no se pueden factorizar). También son conocidos como `primitivas` y para todo $$GF(2^m)$$ se pueden encontrar múltiples primitivas.
