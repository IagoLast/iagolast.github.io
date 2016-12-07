---
layout: post
title:  "Implementacion de GF(256)"
date:   2016-11-07 16:00:00 +0100
categories: blog
---

Este artículo trata sobre la implementación en javascript de un campo de Galois $$GF(256)$$, si tienes dudas sobre cómo funcionan los campos de Galois te recomiendo que empieces por [aquí]({% post_url 2016-11-06-campos-galois %}).



...

### Suma y Resta
La suma y la resta no tienen mayor dificultad. Al todos los coeficientes binaros un [xor binario](https://es.wikipedia.org/wiki/Disyunci%C3%B3n_exclusiva) será suficiente para sumar o restar dos números.
```javascript
function add(a, b) {
	return a ^ b;
}
```

### Generadores

Un número $$ \alpha $$ es un generador  de $$GF(q)$$ $$\iff \forall x_i \in GF(q)$$ se cumple que $$x_i = \alpha^n $$


Esto quiere decir que todos los elementos distintos de cero pertenecientes a $$GF(q)$$ pueden ser escritos como $$\alpha^n$$ siendo $$n$$ un entero positivo.


Por ejemplo $$2$$ sería un generador de $$GF(5)$$ ya que las potencias de 2 nos dan siempre elementos de $$\{0,1,2,3,4 \}$$

	GF(5) = {0,1,2,3,4}

	2^0 = 1
	2^1 = 2
	2^2 = 4
	2^3 = 8 mod 5 = 3
	2^4 = 16 mod 5 = 1
	2^5 = 32 mod 5 = 2
	...



$$ \alpha = 2 $$ es un generador de GF(256) por lo que cualquier número del Campo puede ser expresado como $$2^n$$

### Multiplicación

En lugar de realizar los cálculos de la forma tradicional, multiplicando los dos números paso a paso, utilizaremos las propiedades de los logaritmos para transformar esta operación en una sencilla suma. Además para ahorrar tiempo, todos los posibles logaritmos del campo y sus inversas (exponentes) estarán precalculadas en dos tablas (logTable y expTable). En el último apartado se explica como calcular esas tablas.

En la función para multiplicar dos números se realizan comprobaciones básicas para ver que no estamos multiplicando por uno o por cero, si no se da ninguno de estos casos realizaremos la multiplicación mediante suma de logaritmos.

```javascript
function multiply(a, b) {
	// Si cualquiera de los dos operandos es cero devolvemos cero.
	if (a == 0 || b == 0) {
		return 0;
	}
	// Si uno de los dos operandos es 1 devolvemos el otro operando.
	if (a == 1) {
		return b;
	}
	if (b == 1) {
		return a;
	}
	return expTable[logTable[a] + logTable[b] % 255];
}
```
Comprender la última linea requiere una explicación detallada. Primero de todo recordemos la definición de logaritmo.

$$ \log_\alpha x = n \iff x = \alpha^n $$

Si definimos $$i$$ y $$j$$ como los logaritmos de $$a$$ y de $$b$$ respectivamente:

$$i = \log_\alpha a \qquad  j = \log_\alpha b$$

Podemos reescribir la ecuación de la siguiente forma:

$$ a \times b = \alpha^i \times \alpha^j$$

Y por las [propiedades de las potencias](https://es.wikipedia.org/wiki/Potenciaci%C3%B3n#Multiplicaci.C3.B3n_de_potencias_de_igual_base) podemos escribir

$$\alpha^i \times \alpha^j = \alpha^{i+j} = \alpha^t$$

En resumen, podemos calcular $$a \times b $$ de la siguiente forma

$$ a \times b = \alpha^{i+j} = \alpha^t$$


Donde $$t$$ se define como la suma de $$i$$ y $$j$$, valores que también tenemos disponibles en la tabla de logaritmos. Como en las tablas solamente tenemos valores precalculados para los elementos del campo, habrá que operar módulo 255 para garantizar que el valor de t se encuentra en la tabla de exponenetes.

$$logTable[a] = i \qquad logTable[b] = j \qquad t = a + b \bmod 255$$

Para conocer el valor de $$\alpha^t$$ sólo tenemos que buscar en la tabla donde tenemos todas las exponenciaciones precalculadas.

$$expTable[t] = \alpha^t$$

Siendo verbosos, podríamos escribir esa última linea de la siguiente forma:

```javascript
var i = logTable[a];
var j = logTable[b];
var t = i + j % 255;
return expTable[t];
```

### División

Para la división aplicamos exactamente el mismo razonamiento que en el paso anterior, pero restando los logaritmos en lugar de sumarlos:

$$ a / b = \alpha^{i-j} = \alpha^t$$

Al principio de la función se realizan algunas comprobaciónes:  división entre cero y casos particulares como $$0/n$$ o $$n/1$$.

```javascript
function divide(a, b) {
	if (b == 0) {
		throw RangeError('divide() cannot divide by zero');
	}
	if (a == 0) {
		return 0;
	}
	if (b == 1) {
		return a;
	}
	return expTable[(logTable[a] - logTable[b]) % 255];
}
```




### Inversa
Para calcular el inverso de un número aplicamos una técnica similar.

A partir de la definición de logaritmo obtenemos la siguiente ecuación:

$$ a^{-1} = \alpha^{\log_\alpha a^{-1}} $$

Y por [Las propiedades de los logaritmos](https://es.wikipedia.org/wiki/Logaritmo#Propiedades_algebraicas) sabemos que


$$ \alpha^{(-1) \times \log_\alpha a} $$

o lo que es lo mismo en nuestro campo

$$ \alpha^{255 - \log_\alpha a} = \alpha^t$$

Si recordamos que expTable(t) = $$\alpha^t$$ ya tendremos todos los pasos resueltos para calcular la inversa.

```javascript
function inv(a) {
	if (a == 0) {
		throw RangeError('inv() argument must be positive');
	}
	return expTable[255 - logTable[a]];
}
```

### Cálculo de tablas


Para generar la **tabla de exponenciación** simplemente vamos elevando 2 a sus sucesivas potencias, el problema es que
a partir de $$2^7$$ obtenemos elementos mayores a $$255$$ por lo que se salen del rango de $$GF(256)$$.

Necesitamos que esos elementos volvan a estar en el rango de $$GF(256)$$ sin que se repitan, por lo que no podemos hacer sencillamente el módulo del elemento, en lugar de ello se hace un xor con el `polinomio irreducible`, obteniendo de nuevo elementos pertenecientes al campo de Galois.



```javascript
const IRREDUCTIBLE_POLINOMIAL = 0x11D; // m(x) = x^8 + x^4 + x^3 + x^2 + 1
function createExponentialsTable() {
	var expTable = new Array(256);
	var x = 1;
	for (var i = 0; i <= 256 - 1; i++) {
		expTable[i] = x;
		x = x * 2;
		if (x >= SIZE) {
			x = x ^ IRREDUCTIBLE_POLINOMIAL;
		}
	}
	return expTable;
}
```
En nuestro caso utilizamos el polinomio definido en el estandar ISO 18004 para códigos QR cuya representación hexadecimal es `0x11D`.

$$ C(x) =  x^8 + x^4 + x^3 + x^2 + 1$$

La **tabla de logaritmos** será sencillamente la inversa de la tabla de exponenciación.

```javascript
function createLogarithmsTable(expTable) {
	var logTable = new Array(256);
	for (var i = 0; i < 256 - 1; i++) {
		logTable[expTable[i]] = i;
	}
	logTable[0] = undefined;
	return logTable;
}
```
