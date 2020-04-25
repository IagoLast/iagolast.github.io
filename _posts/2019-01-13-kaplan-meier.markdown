---
layout: post
title:  "Análisis de supervivencia: Curvas de Kaplan Meier con R"
date:   2019-01-13 14:00:00 +0100
categories: blog
math: true
description: "Explicación y ejemplos de como implementar curvas de Kaplan Meier en R para el análisis de supervivencia."
image: "https://starecat.com/content/wp-content/uploads/graph-happiness-intelligence-the-simpsons-lisa.jpg"
---

<amp-img layout="responsive" src="https://starecat.com/content/wp-content/uploads/graph-happiness-intelligence-the-simpsons-lisa.jpg" width="324" height="243" alt="Imagen de los Simpsons con una gráfica"></amp-img>

## Análisis de supervivencia.

El [análisis de supervivencia](https://es.wikipedia.org/wiki/An%C3%A1lisis_de_la_supervivencia) es una parte de la estadística que estudia los procesos relacionados con la muerte de organismos vivos y se puede utilizar en Medicina para comprobar los efectos de un tratamiento.

## Función de supervivencia

La [función de supervivencia](https://es.wikipedia.org/wiki/Funci%C3%B3n_de_supervivencia) <amp-mathml layout="container" inline data-formula="$$ S $$"></amp-mathml> nos indica la probabilidad de sobrevivir más allá de un periodo de tiempo <amp-mathml layout="container" inline data-formula="`t`"></amp-mathml>.


<amp-mathml layout="container" data-formula="$$ S(t) = ProbabilidadSupervivencia $$]">
</amp-mathml>

En su forma más frecuente su representación gráfica será similar a la imagen que ilustra este artículo, donde a medida que pasa el tiempo (eje X) la probabilidad de sobrevivir disminuye (eje Y).


## Estimador de Kaplan Meier

Al estudiar un tratamiento solamente se tiene una pequeña [muestra](https://es.wikipedia.org/wiki/Muestra_estad%C3%ADstica) de pacientes por lo que es imposible conocer la forma real de la función de supervivencia y es necesario un [estimador](https://es.wikipedia.org/wiki/Estimador) para aproximar el valor real de la función.

El [estimador de Kaplan Meier](https://es.wikipedia.org/wiki/Estimador_de_Kaplan-Meier) es especialmente interesante para estudios médicos porque entre otras cosas tiene en cuenta la [censura de los datos](https://es.wikipedia.org/wiki/Censura_(estad%C3%ADstica)). Esta censura puede ocurrir cuando un paciente abandona un estudio, se pierde el seguimiento o sigue vivo una vez finaliza el estudio.

Para generar un estimador de Kaplan Meier hacen falta como mínimo dos datos de cada paciente: el tiempo en el que se realizó la última observación `(tiempo)` y el estado del paciente en el momento de dicha observación `(evento)`.

- **tiempo**: Es un valor numérico que representa el número de dias que han pasado hasta cierto evento.
- **evento**: Es un valor [booleano](https://es.wikipedia.org/wiki/Tipo_de_dato_l%C3%B3gico) que indica si en ese instante de tiempo ha ocurrido un cierto evento, por ejemplo la muerte del paciente.


## Implementación en R

R tiene un paquete específico para análisis de supervivencia llamado [survival](https://cran.r-project.org/web/packages/survival/index.html) que nos ofrece la función [Surv](https://www.rdocumentation.org/packages/survival/versions/2.11-4/topics/Surv) para generar datos de supervivencia de forma sencilla.

```R
survObject <- Surv(data$tiempo, data$evento)
```

### Kaplan-Meier en R

La librería [survival](https://cran.r-project.org/web/packages/survival/index.html) también nos ofrece la función [survfit](https://stat.ethz.ch/R-manual/R-devel/library/survival/html/survfit.html) que dados unos datos de supervivencia nos generará una aproximación de Kaplan-Meier.

```R
fit <- survfit(survObject ~ 1, data = data)
```

Podemos visualizar estos datos mediante la función [plot](https://www.rdocumentation.org/packages/graphics/versions/3.5.2/topics/plot).

```R
plot(fit)
```

### Comparando la supervivencia de variables

Lo interesante de todo esto, es saber como afectan diferentes variables a la supervivencia de los pacientes. Imaginemos que tenemos un dataset con las siguientes columnas:

- **tiempo:** Es un valor numérico que representa el número de dias que han pasado hasta cierto evento.
- **evento:** Es un valor si/no (booleano) que indica si en ese instante de tiempo ha ocurrido un cierto evento (por ejemplo la muerte del paciente).
- **Tipo_de_Tratamiento:** Valor categórico que indica el tipo de tratamiento que recibió el paciente: `A/B/C`.


Será interesante ver como afecta la variable  `Tipo_de_Tratamiento` a la función de supervivencia.


Para ello, utilizaremos la funcion `survfit` pero el primer parámetro será una [fórmula](https://www.datacamp.com/community/tutorials/r-formula-tutorial) en la que indicamos que la variable dependiente de supervivencia depende de la variable `Tipo_de_Tratamiento`.

```R
fit <- survfit(survObject ~ Tipo_de_Tratamiento, data = data)
```

De nuevo podemos representar el resultado utilizando la función `plot`.

<amp-img src="http://dwoll.de/rexrepos/content/assets/figure/rerSurvivalKM03.png" 
  layout="responsive"
  width="504"
  height="504"
  alt="Curva de kaplan meier comparando 3 tratamientos">
</amp-img>

De esta imagen podemos deducir que el tratamiento B representado en color rojo es el que mejores supervivencias ofrece mientras que el tratamiento C ofrece los peores resutados.

### Es la diferencia estadisticamente significativa?

Aunque las curvas sean diferentes necesitamos demostrar formalmente que esa diferencia es estadisticamente significativa y que no es producto del azar. Para ello se suele utilizar el método conocido como [log-rank](https://es.wikipedia.org/wiki/Prueba_de_Mantel%E2%80%93Cox) ofrecido por defecto en la función [survdiff](https://stat.ethz.ch/R-manual/R-devel/library/survival/html/survdiff.html).

```R
logRank <- survdiff(survObject ~ Tipo_de_Tratamiento, data = data)
```

de donde puede extraer el [p-valor](https://es.wikipedia.org/wiki/Valor_p) de la siguiente forma:

```R
pval <- p.val <- 1 - pchisq(logRank$chisq, length(logRank$n) - 1) 
```

### Mostrando los resultados graficamente
Se puede utilizar la funcion [ggsurvplot](https://www.rdocumentation.org/packages/survminer/versions/0.4.3/topics/ggsurvplot) contenida en la libreria [survminer](https://cran.r-project.org/web/packages/survminer/index.html) para generar gráficas más estéticas.

```R
ggsurvplot(fit, data=data, pval = pval)
```


<amp-img src="http://www.sthda.com/sthda/RDoc/figure/survminer/surminer-ggplot2-survival-plot-line-types-colors-2.png" 
  layout="responsive"
  width="774"
  height="573"
  alt="Curva de kaplan meier comparando supervivencia por sexos y mostrando el p valor.">
</amp-img>

## Función en R

A continuación se muestra una función en R que realiza todos los pasos que hemos explicado previamente. Recuerda que para utilizarla es necesario tener cargadas las librerias `survival` y `survminer`.

```R
kaplanMeier <- function(data, timeLabel="time", eventLabel="event", trueLabel="SI", falseLabel="NO", variable = "1") {
  timeData <- data[[timeLabel]]
  eventData <- data[[eventLabel]]
  
  # Transform the eventData vector into a binary vector
  eventData <- sapply(eventData, function(item) {
    if(item == trueLabel) {
      return(1)
    }
    return(0)
  })
  
  # Create the survival Object 
  survObject = Surv(timeData, eventData)
  
  # Define the formula used for the estimation (https://www.datacamp.com/community/tutorials/r-formula-tutorial)
  formulae <- as.formula(paste("survObject ~", variable))
  
  # Create the kaplan meier estimation 
  fit <- do.call(survfit, args = list(formula = formulae, data = data))
  
  if(variable != "1") {
    # Compute the log-rank for the estimation
    logRank <- do.call(survdiff, args = list(formula = formulae, data = data))
    
    # Extract the p-value from the log-rank (https://stat.ethz.ch/pipermail/r-help/2007-April/130676.html)
    pval <- p.val <- 1 - pchisq(logRank$chisq, length(logRank$n) - 1) 
    
    # Plot the result
    ggsurvplot(fit, data=data, pval = pval)
  }
  else {
    ggsurvplot(fit, data=data)
  }
}
```
