# Series temporales en R
Fernando Villalba Bergado  
24 de abril de 2018  



# INTRODUCCIÓN
Este manual describe cómo trabajar con series temporales y cómo hacer pronósticos con ellas en R. Explicaremos los principales paquetes de $$R$$ que facilitan el uso de series temporales y muchos ejemplos prácticos que muestran casos concretos de análisis, aplicaciones y programación sobre datos temporales en este lenguaje.

Una serie temporal es una secuencia de datos, observaciones o valores, medidos en determinados momentos y ordenados cronológicamente. Los datos pueden estar espaciados a intervalos iguales o irregulares, normalmente las series sobre las que trabajaremos son series de intervalo regular, pues son las más fáciles de representar y analizar con funciones y librerías..

Las series temporales son pares de valores fecha-dato y uno de los usos principales es su análisis para predicción y pronóstico (así se hace por ejemplo con los datos climáticos, las acciones de bolsa, o las series de datos demográficos). 

Las series que utilizamos habitualmente son **univariadas*, lo que quiere decir que los datos se corresponden con observaciones secuenciales sobre intervalos de tiempo iguales (un dato al año, al día , hora). Si el tiempo entre observaciones es igual siempre, el tiempo no es necesario en los datos aunque esté implicito en ellos, y solo necesitamos una fecha inicial. 

## Clasificación de las series temporales
Existen muchas formas de clasificar las series temporales, cada vez que clasificamos algo pretendemos identificar patrones, grupos comunes que nos aporte información del comportamiento de los datos. Una vez identificado el grupo, nos permite focalizar y realizar un mejor análisis más orientado y eficaz.

Visualizar los datos es el punto de partida de cualquier análisis, porque en la gráfica se aprecian de forma  intuitiva las caracteristicas de las series y se determina de manera natural la estructura de la serie, enfocando el análisis.

Existen muchas características de las series temporales, vamos a ver algunas de ellas que incluso son complementarias y yuxtapuestas, pero que se entienden  mejor al comparar con su par inverso:

### Tendendcia - ciclo - aleatoriedad

Una forma de ver las caracteristicas de una serie temporal es según estas tres caracteristicas, que se pueden dar deforma aditiva sobre una misma serie:

    1. **Series de tendencia.** Son aquellas que tienen una evolución creciente o decreciente en el tiempo.
    2. **Series estacionales** Son las que reflejan comportamientos de patrones periódicos asociados al calendario, o formas que se repiten en horas tiempos fijos. Las **Series Cíclicas.** serían las que tienen patrones de repetición que no se corresponden con calendarios fijos, se repiten en ciclos irregulares, mucho más difíciles de predecir.
    4. **Series aleatorias**. Son las que no se pueden predecir, las que son equivalentes a ruido blanco.
    
En las series estacionales, podemos pronosticar las temporadas de picos y valles, pero en las series cíclicas es algo más complicado pues la repetición del ciclo no es previsible por calendario. También interesa apreciar en los datos si existe una tendencia de fondo, de forma que pueda extraerse de los datos y aplicar diferentes modelos de predicción a cada uno de los componentes que forman la serie de observaciones.

Lo habitual es que una serie temporal sea mezcla y convinación de estos comprotamientos (tendencia + estacionalidad + ruido), identificarlos facilitará el trabajo de análisis.

Existen funciones en R que descomponen una serie temporal $$X_t$$ en los 3 componentes principales  $$X_t=T_t+E_t+A_t$$ (tendencia + estacionalidad + ruido). Una de estas funciones es `sdl()` del paquete base y también la función `decompose()`.

Ejemplo: descomponemos los componentes estacionalidad + tendencia + ruido de la serie de cotizaciones del oro entre 1980 y 1985 almacenanda en la variable `gold`.


```r
    require(forecast) # la serie ejem: gold está en esta libreria
    
    # Aplicamos funcion de descomposición sdl()
    des.oro<-stl(gold,s.window = "periodic" )
    #Pintamos el resultado    
    plot(des.oro, main="Descomposición del precio del oro")
    #plot(decompose(gold))
```
![descomposición de la serie de precios del oro](imagenes/descom.png)

### Estacionariedad 
Esta característica de las series incica si sus propiedades cámbian o no cambian con el tiempo, es decir si por ejemplo la media, la varianza o la covarianza son constantes en la serie o por el contrario cambian a lo largo de las observaciones de forma significativa. En resumen, una serie estacionaria es aquella cuyas propiedades (media, varianza, covarianza) no dependen del periodo de observación en el que se miden. 

No confuncir con estacional con estacionario, pues son conceptos diferentes, uno indica ciclos y el otro es una propiedad de los datos.

La *estacionariedad* es una propiedad muy importante, pues existen muchos análisis matemáticos que  describen muy bien las series estacionarias, y no tan bien las que no lo son, de hecho los modelos estadisticos de pronóstico de series temporales **sólo sirven para series estacionarias**.

Existen métodos que veremos para transformar una serie no estacionaria en otra que si lo es, y por tanto sobre la que podemos aplicar pronosticos estadísticos con armamento de análisis potente y fundamentado. Recordemos también que los modelos de pronostico no solo pueden ser estadisticos, hay cientos de modelos físicos y basados en otros conceptos que si funcionan con las series no estacionarias.

La estacionariedad puede darse en unas propiedades y no darse en otras, por lo que las transformaciones serán solo en la dirección que interese. La estacionariedad de la varianza, por ejemplo, se denomina **homoestacidad**.

![Ejemplo de estacionariedad en la media](imagenes/estacionaria.png)

![Ejemplo de estacionariedad en la varianza](imagenes/estacionariaVAR.png)

Las series estacionales pueden abordarse, además de por los sistemas estadísticos, por otros métodos como el **análisis espectral** o de frecuencias. Consisten estos sistemas en la aplicación de la *transformada de Fourier* a los datos para descomponer la serie en combinación lineal de funciones trigonométricas (suma de funciones seno y coseno). 

Las graficas asociadas a los métodos espectrales son el periodograma o espectro de frecuencias que está relacionado con la grafica de autocorrelación. En el ejemplo siguiente se analizan las graficas ACF y el periodograma de la serie de manchas solares anual, y como se ve el perido de maxima autocorrelación de la gráfica de ACF que se corresponde con una retraso de 10 años, es equivalente a la frecuencia máxima del espectro de frecuencias en tormo a 0,1 ($$ 1/0,1=10$$)


```r
    acf(sunspot.year)
```

![](Series_temporales_en_R1_files/figure-html/periodograma-1.png)<!-- -->

```r
    spectrum(sunspot.year)
```

![](Series_temporales_en_R1_files/figure-html/periodograma-2.png)<!-- -->

```r
    # periodogram(sunspot.year)
```

### Autocorrelación
La autocorrelación (ACF) es una medida de la dependencia de los valores actuales respecto de los pasados en una serie temporal. Es una propiedad muy importante cuando se evalua para diferentes peridos de retardo en las graficas de autocorrelación que se generan con la fucnión `acf()`. Cuando los valores son cercanos a 1 indica una relación lineal entre los valores pasado y futuros, si es cero es que no existe relación (lineal) apreciable y si es es -1 es que la dependencia es inverssamente lineal.


### Ruido blanco
Se llama ruido blanco a una serie temporal completamente aleatoria **impredecible**, cuyas observaciones son independientes y están idénticamente distribuidas en el tiempo, con media cero y varianza constante.

Cuando hacemos la suma acumulada de una serie de observaciones de *ruido blanco* se nos genera lo que denominamos un *camino aleatorio* ( *randow walk*).

 

```r
    # Generamos ruido blanco
    y<-rnorm(100)
    par(mfrow=c(2,2))
    # pintamos la serie de ruido blanco
    plot(y, type="l")
    plot(acf(y)) # pintamos la grafica de autocorrelaciones
    # pintamos la serie de camino aleatorio
    plot(cumsum(y),type="l")
```

![](Series_temporales_en_R1_files/figure-html/ruidoblanco-1.png)<!-- -->

```r
    plot(acf(cumsum(y)))
```

![](Series_temporales_en_R1_files/figure-html/ruidoblanco-2.png)<!-- -->

Existe una prueba denominada test de *Ljung-Box* que nos sirve para determiar si una serie temporal se comporta como ruido blanco o no. Si el resultado de `p` es un valor mayor que *0.05* se sugiere que los datos no son significativamente diferentes del ruido blanco, y si es inferio es posible que los datos sean predecibles pues se diferencian del ruido blanco, no son observaciones completamente independientes. Veremos esto en detalle en siguientes capítulos

## Estructura del libro

Se ha ordenado el manual en un formato progresivo, de manera que sirva de libro de consulta constante durante el trabajo y también de guia continua de aprendizaje, desde las nociones básicas, a las más complejas.

En primer lugar haremos un resumen completo de las formas de medida del tiempo en R, lo que nos permitirá entender mejor el funcionamiento de las series temporales, así como obtener conocimientos para realizar analisis alternativos, no solo con objetos de serie temporal sino con otras muy comunes como `data frame`.

El segundo capítul está dedicado a obtener datos, a crear series temporales para ejemplos, o a obtenerlas de los multiples origenes que puedan darse (web, ficheros...). Se aportan ejemplos concretos de librerías para descarga de datos financieros que son unos datos muy demandados por ususarios de $$R$$.

En el tercer capítulo entramos al grano describiendo cómo realizar gráficos de series temporales, haremos un completo repaso por las multiples funciones que existen, y que son más fáciles y atractivas.
También se detallan tipos de graficos exclusivos de series temporales y que realizan analisis graficos muy completos, como los graficos estacionales, los de autocorrelación y retardo ect.

El cuarto capitulo está dedicado a las funciones de pronostico más simples y una introducción al completo paquete `forecast`que tiene elementos interesantes para los pronósticos.

El siguiente caitulo se muestra una introducción al analisis de los residuos, que es una forma de evaluar la bondad de las predicciones que realizamos con los modelos de pronostico.

El sexto capitulo se entra en los metodos de pronostico por regresiñon lineal.

El séptimo detalla las transformaciones en los datos, y cómo se pueden conseguir con ellas mejores pronósticos con simples cambios de base de los datos.


# MEDIDA DEL TIEMPO EN R

Antes de iniciarnos en el uso de series temporales vamos realizar un resumen de la medida del tiempo en R, qué clases existen, cómo se trabaja con fechas y datos de tiempo. Conocer estos tipos de datos no es imprescindible en el manejo de series temporales, pero nos permite tener un conocimiento profundo y mucho más completo de la teoría.

Los datos de tiempo (fechas y horas) son tratados en R Base mediante varios tipos de datos, objetos y clases. Estas clases almacenan internamente la fecha como el número de días o segundos transcurridos desde el 1 de enero de 1970 en la zona UTC.

R Base define principalmente 3 clases de datos fechados:

  1. Clase ***Date*** para las fechas diarias de calendario 
  2. Para las fechas horarias se usan dos clases:
    -  **POSIXct** es un **valor** numérico que representa el número de segundos desde 1/1/1970.
    -  **POSIXlt** es un objeto **lista** con distintas variables (sec,min,hour,mday,mon, year, wday, yday, isdst, zone, gmtoff)  
    
### Fecha diarias (Dates)

La clase `Dates` almacena solo datos _diarios_ que incluyen el año, mes y día de una fecha, pero no los tiempos inferiores a un día (ni horas, ni segundos por ejemplo).

Un fecha del tipo *Date* se puede asignar mediante código de la siguiente manera:


```r
  # Asignar una fecha
  x<- as.Date("1970-01-03")
  x  # se imprime como cadena de texto
```

```
## [1] "1970-01-03"
```

```r
  # pero podemos comprobar que internamente se almacena como 2
  # dos días despues del 1 de enero de 1970.
  unclass(x)
```

```
## [1] 2
```


### Fechas completas `POSIXct` y `POSIXlt`

R almacena las fechas completas usando los estándar *POSIX* que incluyen una serie de características para homogeneizar el tipo complejo de datos de tiempos en diferentes sistemas operativos. POSIX es el acrónimo de *Portable Operating System Interface* y la X procede del sistema operativo $$\UNIX$$.
 
Ambas clases *POSIX* almacenan la fecha y hora como el número de segundos transcurridos desde el _1 de enero de 1970_. La clase `POSIXct` almacena internamente esta cifra como un número entero, mientras que la clase `POSIXlt` la descompone en una lista con elementos para los segundos, minutos, horas, día, mes y año.

En resumen:

    -  **POSIXct** es simplemente un numero entero que guarda el número de segundos desde 1/1/1970. Es útil cuando almacenamos tiempos en un data frame, pues ocupa menos espacio y solo una variable.
    -  **POSIXlt** es un objeto lista que almacena un bloque completo de información diferenciada, como el día, año, dia de la semana, día del mes etc.

Para recordar la diferencia es util la nemotecnia de que el que finaliza en lt es una lista (`POSIXlt` $\rightarrow$ lt  = lista, `POSIXct` = entero) 

### librería `lubridate` funciones para fechas    

Para trabajar con fechas y horas es casi imprescindible cargar la librería `lubridate`. Este paquete nos facilita el uso analitico de los datos fechados:

  * `Sys.Date()` $\rightarrow$ fecha actual del sistema operativo
  * `now()` $\rightarrow$  fecha y hora del momento actual
  * `today()` $\rightarrow$ fecha `Date` de hoy
  * `weekdays()` $\rightarrow$ devuelve el día de la semana
  * `months()` $\rightarrow$ retorna el mes de la fecha
  * `quarters()` $\rightarrow$ retorna el cuatrimestre de la fecha (Q1,Q2,Q3 o Q4)
  
Otras funciones que permiten operaciones de tiempo son:

  * `difftime(fecha1, fecha2, units = "weeks")` $\rightarrow$ la diferencia en tiempo entre dos fechas.
  * `diff()` $\rightarrow$ aplicado a una serie temporal da la diferencia entre dos valores consecutivos de fecha.
  * `seq(dia.inicio, dia.fin,length=10, by=2)`$\rightarrow$ crea una secuencia de fechas.

Una forma alternativa y útil para acceder a un valor concreto dentro de una fecha (por ejemplo el año, el día de una fecha) es hacer la llamada a las propiedades de la variable usando el símbolo `$` en la forma `varfecha$mday`:

  * `sec`: segundos 
  * `min`: minutos de la fecha
  * `hour`: hora
  * `mday`: dia del mes
  * `mon`: mes
  * `year`: año desde 1900. Por lo que debemos sumar esto si queremos el año real
  * `wday`: día de la semana (0 domingo - 6 sábado)

Veamos un ejemplo práctico:

```r
  # guardamos la fecha actual del sistema en x
  x <- Sys.time()
  x
```

```
## [1] "2018-07-12 14:56:16 CEST"
```

```r
  # Convertimos es fecha en la clase POSIXlt (la más completa con la lista)
  fecha_x <- as.POSIXlt(x)
  # imprimimos los nombres de las variables de la clase POSIXlt
  names(unclass(fecha_x))
```

```
##  [1] "sec"    "min"    "hour"   "mday"   "mon"    "year"   "wday"  
##  [8] "yday"   "isdst"  "zone"   "gmtoff"
```

```r
  # Para sacar el dato de cualquiera de ellos basta escribir:
  #  nombre del objeto + $ nombre_variable
  fecha_x$sec
```

```
## [1] 16.19421
```

```r
  fecha_x$wday
```

```
## [1] 4
```

```r
  # cambiamos el objeto a POSIXct
  fecha_x<-as.POSIXct(fecha_x)
  unclass(fecha_x)
```

```
## [1] 1531400176
## attr(,"tzone")
## [1] ""
```

```r
  #solo almacena este gran entero con los segundos desde 1/1/1970
  
  # Generamos una secuencia de días
  dia1=as.Date("25/05/2017",format="%d/%m/%Y")
  dia2=as.Date("10/06/2017",format="%d/%m/%Y")
  seq(dia1,dia2, by=1)
```

```
##  [1] "2017-05-25" "2017-05-26" "2017-05-27" "2017-05-28" "2017-05-29"
##  [6] "2017-05-30" "2017-05-31" "2017-06-01" "2017-06-02" "2017-06-03"
## [11] "2017-06-04" "2017-06-05" "2017-06-06" "2017-06-07" "2017-06-08"
## [16] "2017-06-09" "2017-06-10"
```

```r
  seq(dia1,length=10,by=1)
```

```
##  [1] "2017-05-25" "2017-05-26" "2017-05-27" "2017-05-28" "2017-05-29"
##  [6] "2017-05-30" "2017-05-31" "2017-06-01" "2017-06-02" "2017-06-03"
```

```r
  seq(dia1,dia2, by="week")
```

```
## [1] "2017-05-25" "2017-06-01" "2017-06-08"
```

### Conversión de texto a fecha `strptime` 
Cuando el formato de partida que contiene la fecha es un texto, la función _strptime()_ es una manera sencilla de convertirlo en la clase de tiempo, es decir pasar una cadena en formato texto a la clase `POSIX`. La función toma como parámetros de entrada la lista de texto con las fechas y también hay que decirle qué formato tienen dichos textos de entrada.


```r
  # Convertir texto en fecha
  # Ejemplo 1
  x <- c("1 ene 1960 12:37", "2 abr 1990 23:00", "7 jul 1975 15:15")
  # para convertir estos textos con fechas a formato de tiempo de R
  z <- strptime(x, "%d %b %Y %H:%M")
  z
```

```
## [1] "1960-01-01 12:37:00 CET"  "1990-04-02 23:00:00 CEST"
## [3] "1975-07-07 15:15:00 CET"
```

```r
  class(z)
```

```
## [1] "POSIXlt" "POSIXt"
```

```r
  #ahora podemos acceder a los datos de esta forma sencilla
  z$year # nos da el año
```

```
## [1] 60 90 75
```

```r
  z$mon  # nos devuelve el mes
```

```
## [1] 0 3 6
```

```r
  z$wday # nos devuelve el día de la semana
```

```
## [1] 5 1 1
```

```r
  z$mday # devuelve el día del mes
```

```
## [1] 1 2 7
```

```r
  # Ejemplo 2 . cambiamos el formato de entrada
  x <- c("1/1/60 12:37", "2/4/90 23:00", "7/7/75 15:15")
  z <- strptime(x, "%d/%m/%y %R") # notese el cambio de formato
  z
```

```
## [1] "2060-01-01 12:37:00 CET"  "1990-04-02 23:00:00 CEST"
## [3] "1975-07-07 15:15:00 CET"
```


### Operaciones con fechas y tiempos

Podemos realizar operaciones con fechas y tiempos siempre que los objetos que intervengan pertenezcan a la misma clase. Es decir, un objeto _Date_ no puede sumarse a otro _POSIXlt_, debemos primero convertirlos a la misma clase.


```r
  x<- as.Date("2015-03-24")
  y <-as.POSIXlt("2016-03-24 12:10")
  
  # probamos a restar dos objetos de diferentes clases
  # y-x # esto da error 
  
  # Convertimos x a clase POSIXlt
  x<-as.POSIXlt(x)
  y-x
```

```
## Time difference of 366.4653 days
```

```r
  # ahora la resta es posible
```

Los objetos POSIX tienen una propiedad que permite especificar la zona horaria en la forma `y<-as.POSIXct("2016-03-24 12:10", tz="GMT")`

### Funciones adicionales de `lubridate`

Este paquete añade funciones que facilitan la manipulación de fechas expresadas en la clase POSIXct. La conversión de caracteres a dicha clase puede hacerse utilizando las funciones definidas por combinaciones de las letras d, m, y (day, month, year) y h, m, s (horas, minutos y segundos). 

Este sistema facilita la entrada de datos a R, por ejemplo $dmy_hms$ espera *día-mes-año-horas-minutos-segundos* en ese orden; si usamos $ymd_h$ espera *año-mes-dia-hora* en el mismo orden

Veamos un ejemplo:


```r
  require(lubridate)
  library(lubridate)
  
  fecha1=ymd_hms("2016-02-23 18:15:09") 
  fecha1
```

```
## [1] "2016-02-23 18:15:09 UTC"
```

```r
  fecha2=dmy_hm("12/3/1975 23:24")
  fecha2
```

```
## [1] "1975-03-12 23:24:00 UTC"
```

```r
  fecha3=dmy("6 7 1975")
  fecha3
```

```
## [1] "1975-07-06"
```

```r
  # Extraemos valores de mes, semana, y día de la semana de fechas
  month(fecha2)
```

```
## [1] 3
```

```r
  week(fecha3)
```

```
## [1] 27
```

```r
  wday(fecha3)
```

```
## [1] 1
```
### Extraer información de un fecha
Para obtener una información concreta de una fecha u hora se utilizan las funciones, second(), minute(), hour(), day(), wday(), yday(), week(), month(), year() y tz(), para la zona horaria.


```r
  fecha.actual<-now()
  
  # Extraemos información de dicha fecha
  year(fecha.actual) # año
```

```
## [1] 2018
```

```r
  month(fecha.actual) # mes
```

```
## [1] 7
```

```r
  wday(fecha.actual,label = TRUE) # dia de la semana
```

```
## [1] jue
## Levels: dom < lun < mar < mié < jue < vie < sáb
```

```r
  minute(fecha.actual) # minutos
```

```
## [1] 56
```

### Aritmétrica con fechas y horas

La suma de fechas con `lubridate` se hace usando la función *difftime(fecha1,fecha2, units="days")*, en units podemos especificar la unidad de fecha para el resultado entre las siguientes:“auto”, “secs”, “mins”, “hours”, “days”, “weeks”


```r
  d1 <- as.Date("01 marzo 1950", "%d %B %Y")
  d2 <- as.Date(c("01 abril 1955", "01 julio 1980"), "%d %B %Y")
  
  difftime(d2,d1,units = "weeks")
```

```
## Time differences in weeks
## [1]  265.2857 1582.8571
```

También pueden ser tratadas en R como una nueva clase de objeto que se llama intervalo.


```r
  # hay que cargar el paquete lubridate
  require("lubridate")
  # Creamos un intervalo
  congreso_inicio <- ymd_hm("20170528 12:00")
  congreso_fin <- ymd_hm("20170604 10:00") 
  duracion_congreso <-  difftime(congreso_fin,congreso_inicio)
  # Para verlo en unidades enteras de fecha es mejor pasarlo a periodo
  as.period(duracion_congreso)
```

```
## [1] "6d 22H 0M 0S"
```

```r
  # Si no solo queremos la diferencia de fechas sino usarlo como intervalo
  # tenemos que crear el objeto 
  duracion_congreso <- interval(congreso_inicio,congreso_fin)
  # Tambien podía definirlo así:
  congreso_inicio %--% congreso_fin
```

```
## [1] 2017-05-28 12:00:00 UTC--2017-06-04 10:00:00 UTC
```

```r
  # Ejem. tengo programada una visita a Madrid en la fecha
  visita_Madrid<-ymd("20170601")
  # quiero saber si coincide con el congreso
  visita_Madrid %within% duracion_congreso
```

```
## [1] TRUE
```

```r
  # Cual es la duración del congreso en días
  duracion_congreso / days(1)
```

```
## [1] 6.916667
```

```r
  # y en semanas
  duracion_congreso / weeks(1)
```

```
## [1] 0.9880952
```
Para convertir entre intervalos, duraciones o periodos se usan las funciones `as.interval()`, `as.period()`, y `as.duration()`.

Con la aritmética simple podemos generar series temporales de manera sencilla añadiendo o sustrayendo periodos de tiempo a una fecha:


```r
  # Inicio de la serie el 15 de mayo de 2017
  inicio<-dmy_hm("15052017 10:00")
  # Sumamos una semana 4 veces
  serie <- inicio + weeks(0:4)
  serie
```

```
## [1] "2017-05-15 10:00:00 UTC" "2017-05-22 10:00:00 UTC"
## [3] "2017-05-29 10:00:00 UTC" "2017-06-05 10:00:00 UTC"
## [5] "2017-06-12 10:00:00 UTC"
```

# OBTENER DATOS
Para empezar y aprender necesitamos datos y ejemplos sencillos en los que podamos manipular y probar el código. En este capitulo mostraremos algunas de las formas de obtener datos de series temporales de manera que podamos iniciar nuestra práctica con elementos reales.

Solo vamos a describir de forma breve solo algunas de las fuentes de datos. En primer lugar los propios datos de R base, que tiene bastantes, y después muchas librarías y funciones útiles para descarga de datos de series web. Otra forma es crear los datos uno mismo, por ello explicaremos también de forma sencilla cómo crear series de prueba aleatorias para uso de enseñanza y aprendizaje.

## Datos ejemplo en R base
El programa básico de R contienen un montón de datos de ejemplo que podemos usar en la práctica. Estos datos están contenidos en el paquete `datasets` y contiene ejemplos de  multiples formatos: data frames, series ts, matrices, vectores.

Para ver todos los datos disponibles hay que ejecutar el comando `data()` en R, así conseguimos que nos muestre una lista completa de todos los objetos de datos almacenados de ejemplo. Entre estos hay varios de series temporales.


Por ejemplo usaremos los datos de *Nile* que contiene observaciones de caudal máximo del río Nilo para este primer script.


```r
    # Ejemplos de datos en datasets:
    # Serie de caudales del Nilo
    str(Nile) # vemos qué contiene Nile
```

```
##  Time-Series [1:100] from 1871 to 1970: 1120 1160 963 1210 1160 1160 813 1230 1370 1140 ...
```

```r
    plot(Nile, main="caudal máximo anual del río Nilo", col="blue") # pintamos la serie temporal
```

![](Series_temporales_en_R1_files/figure-html/datosNilo-1.png)<!-- -->

```r
    # Otro ejemplo de R Base es AirPassengers
    plot.ts(AirPassengers, main="Num pasajeros internacionales por mes entre 1949-1960", xlab="año" )
```

![](Series_temporales_en_R1_files/figure-html/datosNilo-2.png)<!-- -->

```r
    # datos de manchas solares, diarios, mensuales y anuales
    #plot(sunspot.month)
    plot(sunspot.year)
```

![](Series_temporales_en_R1_files/figure-html/datosNilo-3.png)<!-- -->

Muchas de las librarias relacionadas con los datos contienen ejemplos incluídos en el código del paquete. La libraría `forecast`que usaremos profusamente en este manual tiene varias series temporales de diferentes características que pueden usarse de ejemplo (gold, WWWusage...)

### Resumen de datos temporales de ejemplo

    - sunspot, sunspot.month y sunspot.year :Número de manchas solares medias mensuales de 1749 a 1983. Recolectado en el Observatorio Federal Suizo, Zúrich hasta 1960, luego el Observatorio Astronómico de Tokio.
    - Nile : Mediciones del caudal anual del río Nilo en Aswan, 1871-1970, en $$10^8 m^3$$.
    - EuStockMarkets: precios de cierre diarios de los principales índices bursátiles europeos: Alemania DAX (Ibis), Suiza SMI, Francia CAC y Reino Unido FTSE. Los fines de semana y fiestas se omiten. 
    - WWWusage: del paquete `forecast`.Una serie temporal con el número de usuarios conectados a Internet a través de un servidor cada minuto.
    - gold: del paquete `forecast`. Precios diarios de oro por la mañana en dólares estadounidenses. 1 de enero de 1985 - 31 de marzo de 1989.
    - wineind: del paquete `forecast` Ventas totales de vinos australianos por fabricantes de vino en botellas <= 1 litro. Enero de 1980 - agosto de 1994.
    
## Lectura de ficheros locales
Otra opción simple es la lectura de ficheros locales, como hojas excel o ficheros `csv`.
Recordamos en un ejemplo cómo realizar la lectura de un fichero.


```r
    # Lectura de ficheros separados por comas:
    datos<-read.csv("filename.txt", header = FALSE)
    
    # Lectura de ficheros separados por tab
    datos<-read.table("filename.txt", sep="\t", header = FALSE)
    
    # Por defeco se lee columnas de texto como factores, pero si no queremos eso hacemos
    datos<-read.table("filename.txt", sep="\t", header = FALSE, stringsAsFactors = FALSE)
```

Otra opcion interesante es copiar directamnte del portapapeles o de una url web:


```r
# para leer lo copiado en el portapapeles de windwows
datos<-read.table(file="clipboard", sep="\t",header=TRUE)

# para ller de una url web
datos<-read.csv("http://mitablas/aa.csv")
```

Además siempre podemos recurrir al paquete `rio` que simplifica la importación y exportación de ficheros al máximo, con dos únicas funciones (export, import).


```r
    #install.packages("rio")
    library(rio)
    # uso de export
    export(mtcars, "coches.csv")
    
    # uso de import
    x <- import("coches.csv")
```


## Lectura de datos de internet

Existen multitud de webs de *open data* en las que podemos conseguir información de calidad de administraciones y gobiernos. Normalmente hay que bucear en ellas para buscar datos, y una vez que seleccionamos o tenemos a la vista la tabla resultado nos dan la opción de *descarga de ficheros* en un formato estandar como *csv*, o *json*. Copiando estos enlaces a R podemos hacer descargas directas de los datos a nuestro sistema.

Vamos a poner un ejemplo de lectura de datos desde internet de la web de datos abiertos de la Región de Murcia (<http://datosabiertos.regiondemurcia.es>), es una de tantas paginas oficiales con adminstracion transparente y de volcado de datos públicos. Usaremos esta web para descargar el fichero de datos diarios de la estación meteorológica MU62 situada en la Alberca Murcia y pintar una gráfica de temperaturas máximas. Usarems el objeto temporal xts que veremos más adelante en este libro.


```r
    # leemos un fichero de datos meteorologicos obtenido de la web
    #  datosabiertos.regiondemurcia.es 
    # http://datosabiertos.regiondemurcia.es/carm/catalogo/medio-ambiente/informe-meteorologico-diario-carm-anos-2000-a-2017-estacion-meteorologica-mu62
    
    rutafichero<-"https://datosabiertos.carm.es/odata/Agricultura/IMIDA_Diario_MU62.csv"
    
    # Leemos los datos en formato csv directamente desde la url anterior
    datos_meteo<-read.csv(rutafichero)
    # vemos la estructura de los datos leídos
    str(datos_meteo)
```

```
## 'data.frame':	6447 obs. of  14 variables:
##  $ fecha : Factor w/ 6447 levels "01/01/2001","01/01/2002",..: 2335 2547 2760 2972 3185 3396 3608 3820 4032 4245 ...
##  $ tmed  : num  9.6 8 7 7.7 9.3 8.8 9.4 10 8.1 8.7 ...
##  $ tmax  : num  14.3 15.1 10.5 11.7 13 15 17.8 15.7 11.6 15.4 ...
##  $ tmin  : num  6.5 2.9 4.6 4.1 6.2 4.1 3 5.3 4.7 3.5 ...
##  $ hrmed : Factor w/ 719 levels "  20.9","  21.6",..: 533 517 701 653 609 584 426 298 585 544 ...
##  $ hrmax : Factor w/ 532 levels " 28.3"," 30.2",..: 529 529 530 530 530 530 529 439 527 529 ...
##  $ hrmin : Factor w/ 697 levels "   5.2","   7.0",..: 441 292 687 618 508 441 242 282 573 372 ...
##  $ radmed: num  112.6 124 40.4 94.8 117.2 ...
##  $ radmax: num  496 520 280 442 552 526 545 518 194 517 ...
##  $ vvmed : num  0.4 0.4 0.1 0.3 0.4 0.2 0.5 0.8 0.1 0.3 ...
##  $ vvmax : num  3.1 4.2 2.2 3.1 2.9 2.5 2.9 3.8 3.1 2.5 ...
##  $ dvmed : Factor w/ 1978 levels "    5.5","   11.8",..: 1978 1978 1978 1978 1978 1978 1978 1978 1978 1978 ...
##  $ prec  : num  0.1 0 9.4 0 5.9 0.1 0.1 0 0 0 ...
##  $ eto   : Factor w/ 77 levels "    0.1","    0.2",..: 8 8 5 6 8 7 10 12 6 8 ...
```

```r
    require(xts) # para objeto xts
    require(ggplot2) # para autoplot
    
    # creamos un objeto xts con los datos de tmax leídos 
    # para crear un xts(valores, order.by=...fechas)
    stmax <- xts(datos_meteo$tmax, order.by = as.Date(datos_meteo$fecha, format = "%d/%m/%Y")) 
    
    #Graficamos los datos del 2005 al 2008
    autoplot(stmax["2005/2007"], geom = c("line"), color="red") + xlab("año")
```

![](Series_temporales_en_R1_files/figure-html/leerinternet-1.png)<!-- -->


## Crear objetos temporales `ts`
Existen varias opciones para trabajar con series temporales. La más simple es usar el objeto `ts` de R, pensado para series de tipo anual, mensual e intervalos regulares. Esta función sirve para crear y definir una serie temporal a partir de los datos, fechas y frecuencia: `y <- ts(datos, star=fecha_inicio, end= fecha_fin, frecuency=frecuencia)`. Como está pensada para datos anuales, una frecuencia de 12 indicará datos mensuales, de 4 trimestrales.



```r
    require(forecast)
    # Creamos una serie de 24 datos aleatorios para trabajar
    # como suma acumulada de una normal x10
    datos <- cumsum(rnorm(24)*10)
    # pintamos los datos
    plot(datos, type="l")
```

![](Series_temporales_en_R1_files/figure-html/series_temporales-1.png)<!-- -->

```r
    # Creamos la serie temporal asumiendo que son datos trimestrales (cuartos de año)
    # y que se inician en julio de 1990
    # 12= mensuales
    # 4= cada 4 meses
    mits <- ts(datos, start = c(1990,3), frequency = 4)
    mits
```

```
##            Qtr1       Qtr2       Qtr3       Qtr4
## 1990                        -8.545846 -12.213750
## 1991 -13.091255 -18.653718 -34.544790 -30.196711
## 1992 -29.837297 -22.845861 -16.232800 -18.049459
## 1993 -17.394505 -18.288586  -9.634190  -7.196923
## 1994  -1.155177  -5.785935 -17.013314 -18.869801
## 1995 -26.945030 -32.333361 -32.525540 -25.124140
## 1996 -28.351320 -37.230036
```

```r
    # Pintamos nuevamente los datos 
    plot(mits)
```

![](Series_temporales_en_R1_files/figure-html/series_temporales-2.png)<!-- -->

```r
    # otra forma de ver resumen completo de la serie temporal
    tsdisplay(mits)
```

![](Series_temporales_en_R1_files/figure-html/series_temporales-3.png)<!-- -->

```r
    # Calculamos el maximo de la serie
     which.max(mits)
```

```
## [1] 15
```

```r
     frequency(mits)
```

```
## [1] 4
```

En el siguiente ejemplo representaremos varias series temporales en la misma escala y gráfica con la función plot.


```r
    # ejemplo para pintar varias series juntas en la misma grafica
    # Generamos 3 series aleatorias mensuales
    z <- ts(matrix(rnorm(300), 100, 3), start = c(1961, 1), frequency = 12)
    z[,2]<-z[,2]+5 # sumo 5 a la serie=col 3
    z[,3]<-z[,3]-5 # resto 5 a la serie=col 3
    
    class(z)
```

```
## [1] "mts"    "ts"     "matrix"
```

```r
    head(z) # as "matrix"
```

```
##             Series 1 Series 2  Series 3
## Jan 1961 -0.01918973 1.876334 -6.050900
## Feb 1961  0.15292942 3.353859 -5.155634
## Mar 1961  2.40963106 3.726643 -4.631768
## Apr 1961 -0.43204279 4.980736 -5.007940
## May 1961  0.28091097 7.926778 -6.334031
## Jun 1961 -2.51848305 5.004636 -5.121827
```

```r
    # pintamos las series con plot y con plot single
    plot(z)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

```r
    plot(z, plot.type = "single", lty = 1:3, col=c(1:3))
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-11-2.png)<!-- -->


Vamos a crear también una serie temporal cíclica con números aleatorios:


```r
    st_estac <- sin(seq(0.1*pi, 10*pi, 0.1*pi))
    
    # generamos una lista de 100 numero aleatorios de media 0 y standard deviation de 0.2
    err <- rnorm(length(st_estac),0, 0.2) #note: length(aSeq)=100
    st_estac <- st_estac + err
    
    	
    #creating a monthly time series, beginning at January 2001
    st_estac<- ts(st_estac, freq = 12, start=c(2005,1))
    plot.ts(st_estac, col="red")
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

### Dividir series temporales ts
En muchas ocasiones nos interesa extraer una parte de la serie temporal, dividir una serie para obtener dos o mas subconjuntos. Esto lo podemos hacer con la función `window()`. Veamos un ejemplo.


```r
    # ejemplo de uso de window para extraer el año 1993 de los datos
    # recordemos que son datos trimestrales (4 datos al año)
    # Usamos la serie generada en el ejmplo anterior
    ano93 <- window(mits, start = 1993, end = c(1993,4))
    ano93
```

```
##            Qtr1       Qtr2       Qtr3       Qtr4
## 1993 -17.394505 -18.288586  -9.634190  -7.196923
```

```r
    plot(ano93)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-13-1.png)<!-- -->
### Objeto `xts`
Aunque existen muchos objetos creados en R para el manejo de Series temporales, es `xts` el estandar que ha permitido la gestion eficiente de las series temporales y ha conseguido aglutinar en un mismo objeto las carcerísticas comunes a todos los que necesitan implementar valores de series en el tiempo.

Uno de los defectos que encontramos en el uso de `ts` como objeto temporal es que muchas veces obtenemos datos fechados sin que tengamos necesidad de indicar el inicio y fin del periodo. Un objeto `xts` almacena los datos como `coredata` y las fechas o tiempo como `index`admitiendo una variedad amplia de formatos de tiempo. En el nucleo de un objeto `xts` está `zoo`.

En el ejemplo siguiente vermos muchas formas de usar `xts`, por ejemplo para extraer los datos de la fecha de una objeto `xts`se usa la función `index(objet_xts)`esto no ahorrará muchos dolores de cabeza buscando la forma trabajar con las fechas de la serie.
 

```r
    library(xts)
    # creamos de nuevo una serie aleatoria temporal
    datostem <- cumsum(rnorm(60)*10)
    datostem <- ts(datostem, start = c(2010,3), frequency = 12)
    
    # convertimos el objeto del ejemplo anterior mits en xts
    xts1<-as.xts(datostem)
    class(xts1)
```

```
## [1] "xts" "zoo"
```

```r
    head(xts1)    
```

```
##                [,1]
## mar 2010  -7.312697
## abr 2010 -11.742784
## may 2010 -23.243811
## jun 2010 -19.193767
## jul 2010 -18.827687
## ago 2010 -30.897851
```

```r
    # La forma habitual de crear una serie xts será así:
    require(rio)
      dat.bit<-import("bitcoinity_data.csv")
      xts1 <- xts(dat.bit$price, order.by=as.Date(dat.bit$Time, "%Y-%m-%d %R"))
    
    # extraemos solo lo datos de xts 
    head(coredata(xts1))
```

```
##            [,1]
## [1,] 0.06445989
## [2,] 0.06457231
## [3,] 0.06190290
## [4,] 0.10889928
## [5,] 0.25884091
## [6,] 0.23844714
```

```r
    # vemos el indice de tiempo almacenado en xts
    index(xts1)
```

```
##  [1] "2010-07-01" "2010-08-01" "2010-09-01" "2010-10-01" "2010-11-01"
##  [6] "2010-12-01" "2011-01-01" "2011-02-01" "2011-03-01" "2011-04-01"
## [11] "2011-05-01" "2011-06-01" "2011-07-01" "2011-08-01" "2011-09-01"
## [16] "2011-10-01" "2011-11-01" "2011-12-01" "2012-01-01" "2012-02-01"
## [21] "2012-03-01" "2012-04-01" "2012-05-01" "2012-06-01" "2012-07-01"
## [26] "2012-08-01" "2012-09-01" "2012-10-01" "2012-11-01" "2012-12-01"
## [31] "2013-01-01" "2013-02-01" "2013-03-01" "2013-04-01" "2013-05-01"
## [36] "2013-06-01" "2013-07-01" "2013-08-01" "2013-09-01" "2013-10-01"
## [41] "2013-11-01" "2013-12-01" "2014-01-01" "2014-02-01" "2014-03-01"
## [46] "2014-04-01" "2014-05-01" "2014-06-01" "2014-07-01" "2014-08-01"
## [51] "2014-09-01" "2014-10-01" "2014-11-01" "2014-12-01" "2015-01-01"
## [56] "2015-02-01" "2015-03-01" "2015-04-01" "2015-05-01" "2015-06-01"
## [61] "2015-07-01" "2015-08-01" "2015-09-01" "2015-10-01" "2015-11-01"
## [66] "2015-12-01" "2016-01-01" "2016-02-01" "2016-03-01" "2016-04-01"
## [71] "2016-05-01" "2016-06-01" "2016-07-01" "2016-08-01" "2016-09-01"
## [76] "2016-10-01" "2016-11-01" "2016-12-01" "2017-01-01" "2017-02-01"
## [81] "2017-03-01" "2017-04-01" "2017-05-01" "2017-06-01" "2017-07-01"
## [86] "2017-08-01" "2017-09-01" "2017-10-01" "2017-11-01" "2017-12-01"
## [91] "2018-01-01" "2018-02-01" "2018-03-01" "2018-04-01" "2018-05-01"
```

```r
    indexClass(xts1) # tipo de fechas usadas
```

```
## [1] "Date"
```

```r
    indexFormat(xts1) # formato de las fechas
```

```
## NULL
```

```r
    tzone(xts1) # Zona Horaria
```

```
## [1] "UTC"
```

```r
    # otra forma de verlo todo a la vez es con el comando
        .index(xts1)
```

```
##  [1] 1277942400 1280620800 1283299200 1285891200 1288569600 1291161600
##  [7] 1293840000 1296518400 1298937600 1301616000 1304208000 1306886400
## [13] 1309478400 1312156800 1314835200 1317427200 1320105600 1322697600
## [19] 1325376000 1328054400 1330560000 1333238400 1335830400 1338508800
## [25] 1341100800 1343779200 1346457600 1349049600 1351728000 1354320000
## [31] 1356998400 1359676800 1362096000 1364774400 1367366400 1370044800
## [37] 1372636800 1375315200 1377993600 1380585600 1383264000 1385856000
## [43] 1388534400 1391212800 1393632000 1396310400 1398902400 1401580800
## [49] 1404172800 1406851200 1409529600 1412121600 1414800000 1417392000
## [55] 1420070400 1422748800 1425168000 1427846400 1430438400 1433116800
## [61] 1435708800 1438387200 1441065600 1443657600 1446336000 1448928000
## [67] 1451606400 1454284800 1456790400 1459468800 1462060800 1464739200
## [73] 1467331200 1470009600 1472688000 1475280000 1477958400 1480550400
## [79] 1483228800 1485907200 1488326400 1491004800 1493596800 1496275200
## [85] 1498867200 1501545600 1504224000 1506816000 1509494400 1512086400
## [91] 1514764800 1517443200 1519862400 1522540800 1525132800
## attr(,"tzone")
## [1] "UTC"
## attr(,"tclass")
## [1] "Date"
```

```r
    # nos dice el día de la semana del indice
    .indexwday(xts1)
```

```
##  [1] 4 0 3 5 1 3 6 2 2 5 0 3 5 1 4 6 2 4 0 3 4 0 2 5 0 3 6 1 4 6 2 5 5 1 3
## [36] 6 1 4 0 2 5 0 3 6 6 2 4 0 2 5 1 3 6 1 4 0 0 3 5 1 3 6 2 4 0 2 5 1 2 5
## [71] 0 3 5 1 4 6 2 4 0 3 3 6 1 4 6 2 5 0 3 5 1 4 4 0 2
```

```r
    # periodicidad de la serie temporal
    periodicity(xts1)
```

```
## Monthly periodicity from 2010-07-01 to 2018-05-01
```

```r
    nmonths(xts1) # cuenta el numero de meses de la serie
```

```
## [1] 95
```

```r
    nyears(xts1) # cuenta el numero de años
```

```
## [1] 9
```

```r
    # seleccion de un año con xts
    xts1["2013"]
```

```
##                 [,1]
## 2013-01-01  15.51720
## 2013-02-01  25.82185
## 2013-03-01  58.39094
## 2013-04-01 128.76183
## 2013-05-01 117.71120
## 2013-06-01 105.98367
## 2013-07-01  86.08022
## 2013-08-01 105.23596
## 2013-09-01 126.69523
## 2013-10-01 153.93089
## 2013-11-01 539.52448
## 2013-12-01 795.18460
```

```r
    # seleccion en un intervalo de fechas
    xts1["20111101/20130705"]
```

```
##                  [,1]
## 2011-11-01   2.739032
## 2011-12-01   3.481064
## 2012-01-01   6.172045
## 2012-02-01   5.154680
## 2012-03-01   4.911478
## 2012-04-01   4.993251
## 2012-05-01   5.052426
## 2012-06-01   5.979181
## 2012-07-01   7.821224
## 2012-08-01  10.942336
## 2012-09-01  11.495739
## 2012-10-01  11.632361
## 2012-11-01  11.360466
## 2012-12-01  13.202405
## 2013-01-01  15.517198
## 2013-02-01  25.821852
## 2013-03-01  58.390943
## 2013-04-01 128.761829
## 2013-05-01 117.711202
## 2013-06-01 105.983668
## 2013-07-01  86.080222
```

```r
    # Tambien se permiten calculos de retraso o diferencia
    head(lag(xts1,2)) # crea una serie con retraso de un intervalo
```

```
##                  [,1]
## 2010-07-01         NA
## 2010-08-01         NA
## 2010-09-01 0.06445989
## 2010-10-01 0.06457231
## 2010-11-01 0.06190290
## 2010-12-01 0.10889928
```

```r
    head(diff(xts1,3)) # crea una serie con la diferencia del 3 intervalo
```

```
##                  [,1]
## 2010-07-01         NA
## 2010-08-01         NA
## 2010-09-01         NA
## 2010-10-01 0.04443939
## 2010-11-01 0.19426860
## 2010-12-01 0.17654424
```

```r
    # conversion de frecuencia de los datos
    datos_ano<-to.period(xts1, period = "years",OHLC = FALSE)
    head(datos_ano )
```

```
##                   [,1]
## 2010-12-01   0.2384471
## 2011-12-01   3.4810637
## 2012-12-01  13.2024049
## 2013-12-01 795.1846021
## 2014-12-01 342.4452384
## 2015-12-01 422.1963177
```

```r
    # uso de splits
    split(xts1 , f = "years")
```

```
## [[1]]
##                  [,1]
## 2010-07-01 0.06445989
## 2010-08-01 0.06457231
## 2010-09-01 0.06190290
## 2010-10-01 0.10889928
## 2010-11-01 0.25884091
## 2010-12-01 0.23844714
## 
## [[2]]
##                  [,1]
## 2011-01-01  0.3737089
## 2011-02-01  0.9227864
## 2011-03-01  0.8498566
## 2011-04-01  1.2143458
## 2011-05-01  6.2275388
## 2011-06-01 18.4706590
## 2011-07-01 13.8333781
## 2011-08-01 10.1313053
## 2011-09-01  5.9026515
## 2011-10-01  3.6650502
## 2011-11-01  2.7390323
## 2011-12-01  3.4810637
## 
## [[3]]
##                 [,1]
## 2012-01-01  6.172045
## 2012-02-01  5.154680
## 2012-03-01  4.911478
## 2012-04-01  4.993251
## 2012-05-01  5.052426
## 2012-06-01  5.979181
## 2012-07-01  7.821224
## 2012-08-01 10.942336
## 2012-09-01 11.495739
## 2012-10-01 11.632361
## 2012-11-01 11.360466
## 2012-12-01 13.202405
## 
## [[4]]
##                 [,1]
## 2013-01-01  15.51720
## 2013-02-01  25.82185
## 2013-03-01  58.39094
## 2013-04-01 128.76183
## 2013-05-01 117.71120
## 2013-06-01 105.98367
## 2013-07-01  86.08022
## 2013-08-01 105.23596
## 2013-09-01 126.69523
## 2013-10-01 153.93089
## 2013-11-01 539.52448
## 2013-12-01 795.18460
## 
## [[5]]
##                [,1]
## 2014-01-01 837.2307
## 2014-02-01 641.6259
## 2014-03-01 593.0073
## 2014-04-01 461.1712
## 2014-05-01 487.0863
## 2014-06-01 614.9025
## 2014-07-01 617.0780
## 2014-08-01 536.3052
## 2014-09-01 445.5712
## 2014-10-01 363.7752
## 2014-11-01 366.5197
## 2014-12-01 342.4452
## 
## [[6]]
##                [,1]
## 2015-01-01 251.2614
## 2015-02-01 234.8412
## 2015-03-01 270.7110
## 2015-04-01 236.0829
## 2015-05-01 239.5750
## 2015-06-01 239.8023
## 2015-07-01 280.2317
## 2015-08-01 253.5103
## 2015-09-01 234.8069
## 2015-10-01 265.4415
## 2015-11-01 349.3576
## 2015-12-01 422.1963
## 
## [[7]]
##                [,1]
## 2016-01-01 411.5218
## 2016-02-01 402.7739
## 2016-03-01 415.8178
## 2016-04-01 432.9478
## 2016-05-01 461.0270
## 2016-06-01 636.3366
## 2016-07-01 659.8910
## 2016-08-01 580.5673
## 2016-09-01 606.8084
## 2016-10-01 641.2744
## 2016-11-01 725.9256
## 2016-12-01 820.6541
## 
## [[8]]
##                  [,1]
## 2017-01-01   910.6948
## 2017-02-01  1058.0350
## 2017-03-01  1133.7862
## 2017-04-01  1215.7601
## 2017-05-01  1884.7714
## 2017-06-01  2636.0463
## 2017-07-01  2504.2073
## 2017-08-01  3855.0189
## 2017-09-01  4125.9778
## 2017-10-01  5260.6432
## 2017-11-01  7748.6436
## 2017-12-01 14973.4488
## 
## [[9]]
##                 [,1]
## 2018-01-01 13070.416
## 2018-02-01  9448.033
## 2018-03-01  9156.124
## 2018-04-01  8001.071
## 2018-05-01  8446.214
```

```r
    # uso de lapply
    lapply(xts1, FUN = cumsum)
```

```
## [[1]]
##                    [,1]
## 2010-07-01 6.445989e-02
## 2010-08-01 1.290322e-01
## 2010-09-01 1.909351e-01
## 2010-10-01 2.998344e-01
## 2010-11-01 5.586753e-01
## 2010-12-01 7.971224e-01
## 2011-01-01 1.170831e+00
## 2011-02-01 2.093618e+00
## 2011-03-01 2.943474e+00
## 2011-04-01 4.157820e+00
## 2011-05-01 1.038536e+01
## 2011-06-01 2.885602e+01
## 2011-07-01 4.268940e+01
## 2011-08-01 5.282070e+01
## 2011-09-01 5.872335e+01
## 2011-10-01 6.238840e+01
## 2011-11-01 6.512744e+01
## 2011-12-01 6.860850e+01
## 2012-01-01 7.478054e+01
## 2012-02-01 7.993522e+01
## 2012-03-01 8.484670e+01
## 2012-04-01 8.983995e+01
## 2012-05-01 9.489238e+01
## 2012-06-01 1.008716e+02
## 2012-07-01 1.086928e+02
## 2012-08-01 1.196351e+02
## 2012-09-01 1.311309e+02
## 2012-10-01 1.427632e+02
## 2012-11-01 1.541237e+02
## 2012-12-01 1.673261e+02
## 2013-01-01 1.828433e+02
## 2013-02-01 2.086651e+02
## 2013-03-01 2.670561e+02
## 2013-04-01 3.958179e+02
## 2013-05-01 5.135291e+02
## 2013-06-01 6.195128e+02
## 2013-07-01 7.055930e+02
## 2013-08-01 8.108290e+02
## 2013-09-01 9.375242e+02
## 2013-10-01 1.091455e+03
## 2013-11-01 1.630980e+03
## 2013-12-01 2.426164e+03
## 2014-01-01 3.263395e+03
## 2014-02-01 3.905021e+03
## 2014-03-01 4.498028e+03
## 2014-04-01 4.959199e+03
## 2014-05-01 5.446286e+03
## 2014-06-01 6.061188e+03
## 2014-07-01 6.678266e+03
## 2014-08-01 7.214571e+03
## 2014-09-01 7.660143e+03
## 2014-10-01 8.023918e+03
## 2014-11-01 8.390437e+03
## 2014-12-01 8.732883e+03
## 2015-01-01 8.984144e+03
## 2015-02-01 9.218985e+03
## 2015-03-01 9.489696e+03
## 2015-04-01 9.725779e+03
## 2015-05-01 9.965354e+03
## 2015-06-01 1.020516e+04
## 2015-07-01 1.048539e+04
## 2015-08-01 1.073890e+04
## 2015-09-01 1.097371e+04
## 2015-10-01 1.123915e+04
## 2015-11-01 1.158850e+04
## 2015-12-01 1.201070e+04
## 2016-01-01 1.242222e+04
## 2016-02-01 1.282500e+04
## 2016-03-01 1.324081e+04
## 2016-04-01 1.367376e+04
## 2016-05-01 1.413479e+04
## 2016-06-01 1.477113e+04
## 2016-07-01 1.543102e+04
## 2016-08-01 1.601158e+04
## 2016-09-01 1.661839e+04
## 2016-10-01 1.725967e+04
## 2016-11-01 1.798559e+04
## 2016-12-01 1.880625e+04
## 2017-01-01 1.971694e+04
## 2017-02-01 2.077498e+04
## 2017-03-01 2.190876e+04
## 2017-04-01 2.312452e+04
## 2017-05-01 2.500929e+04
## 2017-06-01 2.764534e+04
## 2017-07-01 3.014955e+04
## 2017-08-01 3.400457e+04
## 2017-09-01 3.813054e+04
## 2017-10-01 4.339119e+04
## 2017-11-01 5.113983e+04
## 2017-12-01 6.611328e+04
## 2018-01-01 7.918370e+04
## 2018-02-01 8.863173e+04
## 2018-03-01 9.778785e+04
## 2018-04-01 1.057889e+05
## 2018-05-01 1.142351e+05
```


## Librería `Quandl`
El paquete `Quandl` da acceso desde R a la ámplia base de datos del mismo nombre. `Quandl` ofrece datos financieros de múltiples fuentes, algunos de forma gratuita y otros en formato premium que requiere inscripción e incluso pago.

El primer paso para usar `Quandl` es registrar un nuevo usuario en el sitio web. Poco después, hay que ir a la configuración de la cuenta y hacer clic en **API KEY**. Esta página debe mostrar un código, como este: *LDgg4c7zsTgLLxyHj*. Este código lo usaremos para la conexión.

La gente `Quandl` ha desarrollado una API especifica para R bastante sencilla de usar. Hay muchos datos libres,como por ejemplo todos los datos economicos mundiales de la [FRED](https://fred.stlouisfed.org/categories). Para obtener el listado completo de datos que ofrece la librería podemo ir a su web y buscar el nombre de la serie especifica que buscamos <https://www.quandl.com/data/>. Veamos un ejemplo.



```r
    # install.packages("Quandl")
    require(Quandl)
    # creamos el API KEY para la conexión
    Quandl.api_key("LDgg4c7zsTgLLxyHj.")

    # obtenemos datos: serie temporal población en España
    poblacionESP<-Quandl("FRED/POPTTLESA148NRUG",type="ts")
    
    # podemos especificar las fechas de inicio y fin:
    # Quandl("FRED/GDPPOT", start_date="2005-01-03",end_date="2013-04-10",type="xts")

    plot(poblacionESP, main="Evolución de la población en España",type= "l",col="red",lwd=2)
```

![Poblacion de España](imagenes/ESP_poblacion.png)

La web de `Quandl` incluso facilita el propio script de R para el dato que buscas, por ejemplo mira aquí: <https://www.quandl.com/data/FRED/GDPPOT-Real-Potential-Gross-Domestic-Product>


## Librería `quandmod`

Para obtener datos de bolsa y valores `quandmod` es una librería interesante que contienen la función `getSymbols()`. Esta función permite extraer de las webs de yahoo, FRED y alguna más los datos diarios de acciones, fondos e índices. Por desgracia *Google* bloqueó recientemente el acceso a *getSymbols*.

Para descargar un valor entre fechas es necesario conocer el TICKER del valor y usar la función como sigue, por cierto los datos se almacenan en un objeto `xts`.


```r
    require(quantmod)

    acc1 <- getSymbols("TEF.MC", 
                       src ="yahoo",
                       periodicity = "monthly",
                       warning= "FALSE",
                       adjust = TRUE,
                       auto.assign=FALSE)
    require(ggplot2)    
    autoplot(Cl(acc1),main="Telefónica MC")
```

![](Series_temporales_en_R1_files/figure-html/ejemplo_quantmod-1.png)<!-- -->

## Librería `BatchGetSymbols`
Esta librería es similar a `quantmod`, pero para análisis de varios valores puede tener alguna ventaja al estructurar mejor los datos en una única *data frame*.


```r
    # esta librería no funcion con R nuevo
    library("BatchGetSymbols")
    # Cargamos varios tickers de empresas
    mis.valores <- c('MSFT','GOOGL','JPM','GE')
    
    # establecemos las fechas
    fecha.ini <- Sys.Date()-30
    fecha.fin <- Sys.Date()
    
    datosvalores<- BatchGetSymbols(tickers = mis.valores,
                             first.date = fecha.ini,
                             last.date = fecha.fin )
```

## ustyc
El paquete `ustyc` permite la descarga de los datos de la curva de interes del  Departamento del Tesoro de EE.UU. La curva de intereses es el tipo que un inversor recibiría por los bonos a diferentes vencimientos. Los valores de los rendimientos se basan en los precios de los instrumentos de deuda del gobierno de EE.UU. negociados en el mercado secundario. 

Dado que estos bonos tienen un riesgo de incumplimiento mínimo, estos valores se utilizan comúnmente como puntos de referencia para rendimientos sin riesgo en los mercados financieros.

Usar el paquete ustyc es muy simple. Todo lo que necesita hacer es llamar a la función `getYieldCurve`. En el siguiente ejemplo, descargaremos la curva de rendimiento para el año 2017.


```r
    library(ustyc)
    # obtener lacurva de rendimiento
    my.yield.curve <- getYieldCurve(year = 2017)
    print(head(my.yield.curve$df))
```

```
##            BC_1MONTH BC_3MONTH BC_6MONTH BC_1YEAR BC_2YEAR BC_3YEAR
## 2017-01-03      0.52      0.53      0.65     0.89     1.22     1.50
## 2017-01-04      0.49      0.53      0.63     0.87     1.24     1.50
## 2017-01-05      0.51      0.52      0.62     0.83     1.17     1.43
## 2017-01-06      0.50      0.53      0.61     0.85     1.22     1.50
## 2017-01-09      0.50      0.50      0.60     0.82     1.21     1.47
## 2017-01-10      0.51      0.52      0.60     0.82     1.19     1.47
##            BC_5YEAR BC_7YEAR BC_10YEAR BC_20YEAR BC_30YEAR
## 2017-01-03     1.94     2.26      2.45      2.78      3.04
## 2017-01-04     1.94     2.26      2.46      2.78      3.05
## 2017-01-05     1.86     2.18      2.37      2.69      2.96
## 2017-01-06     1.92     2.23      2.42      2.73      3.00
## 2017-01-09     1.89     2.18      2.38      2.69      2.97
## 2017-01-10     1.89     2.18      2.38      2.69      2.97
##            BC_30YEARDISPLAY
## 2017-01-03             3.04
## 2017-01-04             3.05
## 2017-01-05             2.96
## 2017-01-06             3.00
## 2017-01-09             2.97
## 2017-01-10             2.97
```

## Datos de Bitcoin y otras criptomonedas
Otra fuente de datos de interés son las nuevas criptomonedas y el Bitcoin (BTC). 

### `coinmarketcapr`
Uno de los paquetes más interesantes es `coinmarketcapr` con el que podemos descargar las series temporales de valoración de criptos que usa la web <https://coinmarketcap.com/> y muchas funciones variadas de control del estado de mercado:


```r
    library(coinmarketcapr)
    # funcion plot_top_5_currencies
    # pinta el valor de las 5 primeras criptos en capitalización
    plot_top_5_currencies()
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

```r
    # funcion get_marketcap_ticker_all
    # nos da el precio actual de cada cripto frente al $ y BTC
    market_today <- get_marketcap_ticker_all()
    head(market_today[,1:8])
```

```
##             id         name symbol rank price_usd  price_btc
## 1      bitcoin      Bitcoin    BTC    1   6201.83        1.0
## 2     ethereum     Ethereum    ETH    2   433.626  0.0700331
## 3       ripple          XRP    XRP    3  0.435309 0.00007030
## 4 bitcoin-cash Bitcoin Cash    BCH    4    682.21   0.110181
## 5          eos          EOS    EOS    5    6.8006 0.00109834
## 6     litecoin     Litecoin    LTC    6   76.4711  0.0123505
##   X24h_volume_usd market_cap_usd
## 1    3613890000.0   106330530396
## 2    1398500000.0  43641581078.0
## 3     179086000.0  17091295547.0
## 4     325895000.0  11756669900.0
## 5     557137000.0   6094354236.0
## 6     253511000.0   4387149565.0
```

### `RBitcoin`

Otro paquete que vale la pena mencionar es [RBitcoin](https://github.com/jangorecki/Rbitcoin). Permite el acceso a los datos comerciales de varias casa de cambio de Bitcoin. Vamos a mostrar un ejemplo simple de importación de datos desde el exchange de intercambio [kraken](https://www.kraken.com/).


```r
    #install.packages("Rbitcoin", repos=c("https://jangorecki.gitlab.io/Rbitcoin","https://cran.rstudio.com"))
    
    library(Rbitcoin)
    # ver precios en exchanges
    market <- "kraken"
    #currency_pair <- c("BTC","EUR")
    currency_pair <- c("LTC","EUR")
    ticker <- market.api.process(market, currency_pair, "ticker")
    ticker
```

```
##    market base quote           timestamp market_timestamp  last     vwap
## 1: kraken  LTC   EUR 2018-07-12 14:56:38             <NA> 65.08 65.31476
##      volume   ask   bid
## 1: 8531.159 65.23 65.09
```

```r
    # CONVERSION DE VALORES
    fromBTC(1) # el precio actual de BTC en USD
```

```
## [1] 6192.334
```

```r
    fromBTC(1, "EUR") # el precio actual de BTC en euros
```

```
## [1] 5293.806
```

```r
    toBTC(150, "EUR") # convertir 150 EUROS a BTC
```

```
## [1] 0.02833707
```

```r
    # acceso a la blockchain
    # a wallet
    monedero <- blockchain.api.process('1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa')
    str(monedero)
```

```
## Classes 'data.table' and 'data.frame':	1 obs. of  10 variables:
##  $ location      : chr "blockchain"
##  $ action        : chr "Single Address"
##  $ timestamp     : POSIXct, format: "2018-07-12 14:57:08"
##  $ currency      : chr "BTC"
##  $ hash          : chr "62e907b15cbf27d5425399ebf6f0fb50ebb88f18"
##  $ address       : chr "1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"
##  $ n_tx          : int 1283
##  $ total_received: num 66.9
##  $ total_sent    : num 0
##  $ final_balance : num 66.9
##  - attr(*, ".internal.selfref")=<externalptr>
```

```r
    # Acceso a las transacciones
    tx <- blockchain.api.process('e5c4de1c70cb6d60db53410e871e9cab6a0ba75404360bf4cda1b993e58d45f8')
    str(tx, max.level=1)
```

```
## List of 10
##  $ location    : chr "blockchain"
##  $ action      : chr "Single Transaction"
##  $ timestamp   : POSIXct[1:1], format: "2013-12-22 21:22:14"
##  $ currency    : chr "BTC"
##  $ double_spend: logi FALSE
##  $ inputs      :Classes 'data.table' and 'data.frame':	1 obs. of  3 variables:
##   ..- attr(*, ".internal.selfref")=<externalptr> 
##  $ hash        : chr "e5c4de1c70cb6d60db53410e871e9cab6a0ba75404360bf4cda1b993e58d45f8"
##  $ tx_index    : int 45125549
##  $ out         :Classes 'data.table' and 'data.frame':	2 obs. of  3 variables:
##   ..- attr(*, ".internal.selfref")=<externalptr> 
##  $ size        : int 258
```

### `coindeskr`
Otro paquete interesante es `coindeskr`, que extrae de la web de noticias mas famosa del sector <https://www.coindesk.com/> el precio del bitcoin y otras cosas interesantes. La informeación de la libraría está en: <https://cran.r-project.org/web/packages/coindeskr/coindeskr.pdf>


```r
    # Intalamos el paquete
    #install.packages('coindeskr')
    library(coindeskr) #R-Package connecting to Coindesk API
    
    # obtenemos el precio de los ultimos 31 días (ultimo mes)
    last31 <- get_last31days_price() 
    # pintamos le resultado
    plot.ts(last31)
```

![](Series_temporales_en_R1_files/figure-html/coindeskr-1.png)<!-- -->

```r
    # obtenemos los precios historicos del cambio BTC EUR
    precios100_dias<-get_historic_price(currency = "EUR", start = Sys.Date() - 100,end = Sys.Date() )
    plot.ts(precios100_dias)
```

![](Series_temporales_en_R1_files/figure-html/coindeskr-2.png)<!-- -->

### Enlaces de datos
Algunas paginas web dan un servicio directo de descarga de datos en formato csv, por ejemplo los datos de cambi de BTC están disponibles en este sistema en: <http://data.bitcoinity.org/markets/price_volume/all/USD?t=lb&vu=curr>

De esta página hemos descargado el fichero *bitcoinity_data.csv* con los precios históricos y volumen de BTC.


```r
    #Leemos el fichero
    require(rio)
    dat.bit<-import("bitcoinity_data.csv")
    plot.ts(dat.bit$price)
```

![](Series_temporales_en_R1_files/figure-html/ejem_rio-1.png)<!-- -->

```r
    # Como vemos no coge bien las fechas por lo que debemos hacer una conversión forzada
    # usando la libraría xts.
    library(xts)
    dat.xts.bit <- xts(dat.bit$price, order.by=as.Date(dat.bit$Time, "%Y-%m-%d %R"))
    # pintamos el logaritmo de la serie para comprender mejor los cambios de escala
    autoplot(log(dat.xts.bit))
```

![](Series_temporales_en_R1_files/figure-html/ejem_rio-2.png)<!-- -->

```r
    #otra opcion es usar zoo
    require(zoo)
    dd<- read.zoo(dat.bit[,1:2],header = TRUE, format = "%Y-%m-%d %R")
    plot(dd)
```

![](Series_temporales_en_R1_files/figure-html/ejem_rio-3.png)<!-- -->

```r
    # agregamos a valores mensuales por año yearamon
    dd.mes<-as.zooreg(aggregate(dd, as.yearmon,max),freq=12)
    tail(dd.mes)
```

```
##  dic 2017  ene 2018  feb 2018  mar 2018  abr 2018  may 2018 
## 14973.449 13070.416  9448.033  9156.124  8001.071  8446.214
```

# GRÁFICAS DE SERIES TEMPORALES
Hemos visto como manejar objetos de tiempo en R, con las clases: `Date`, `POSIXlt` y `POSIXct`, estas clases nos facilitan el trabajo de fechas, pero las series temporales son algo más complejas pues además del fechado está el otro par del conjunto que lo forma el valor.

En las series de intervalo regular nos podemos olvidar del fechado para trabajo analítico y quedarnos solo con el valor y el orden cronológico de dichos datos. Esto facilita el trabajo y amplia las posibilidades de cálculo.

Vamos a ver en este capitulo cómo pintar objetos de serie temporal en R para producir desde gráficos sencillos a otros complejos con tratamiento implicito de los datos como los estacionales o los de ACF.

## Funciones básicas de trazado de series temporales
Cuando usamos el comando `plot()` para representar series temporales R reconoce automáticamente el origen de los datos y ejecuta la funcion `plot.ts()`. Esta función adapta los ejes de abcisas al formato de fecha que tenga la serie temporal así como algunas otras caracteristicas gráficas.

Otra de las funciones que permiten dibujar series temporales es `ggplot`, y en concreto el comando `autoplot()`.


```r
    # Vamos a leer unos datos temporales de la FRED 
    # para su representación grafica posterior:
    # desempleo datos ESP trimestrales
    desESP <- getSymbols("LRUN64TTESQ156N", 
                       src ="FRED",
                       warning= "FALSE",
                       adjust = TRUE,
                       auto.assign=FALSE)  
    # precio vivienda ESP datos trimestrales
    pcasasESP <- getSymbols("QESN628BIS", 
                       src ="FRED",
                       warning= "FALSE",
                       adjust = TRUE,
                       auto.assign=FALSE) 
    
    # Unimos las dos variables
    datosESP<- cbind(desESP,pcasasESP)
    # Eliminamos las filas sin ambos datos
    datosESP<-datosESP[complete.cases(datosESP),]
```


## Gráficas estandar 
Describimos las funciónes sencillas para pintar grafics temporales. Es un asunto delicado, pues nos encontraremos que en muchas ocasiones, las funciones habitaules de trazado no pintan las graficas correctamente, o no interpretan las fechas que están almacenadas en lso objetos `ts`, `xts`, u otros.
En ocasiones manejaremos dataframes, que son los objetos más comunies, y no sabremos cómo pintar las fechas correctamente, bien porque están en los nombres de fila o porque proceden de un objeto xts.
Estas cuestiones las intentaremos acalarar en este apartado que será consulta obligada para el que se inicia en series temporales.

###`plot.xts`
El formato más completo para trabajar con series temporales es `xts`, pues los datos de objetos `ts` están más limitados, por ejemplo requieren datos de frecuencia uniforme y sin huecos mientras que `xts`no.

Por esto la forma habitual de pintar una grafica temporal es como `plot.xts()`, esta función produce gráfias por defecto que detectan las fechas y pinta correctamente los datos.


```r
    # pintamos una grafica como plot.xts
    plot.xts(datosESP)
```


### `autoplot` 
Otro tipo de grafico comodín al que recurrir con frecuiencia es `autoplot`. Es una versión simplificada de `ggplot` que no necesita de la definición detallada de los parámetros y por tanto es más sencilla de usar que la función original de la libreria `ggplot2`. Cuando la funcion detecta una serie temporal la función real que ejecuta es `autoplot.zoo()`.


```r
    # pintamos una grafica como plot.xts
    autoplot(datosESP)
    
    #Calculamos 3 funciones de pronostico de los datos a h=10
        y <-forecast(datosESP$desempleo, h=10) # ets pronostico
        y2<-rwf(datosESP$desempleo,h=10,drift=TRUE)
        y3<-naive(datosESP$desempleo,h=10)
        
    # pintamos los 3 pronosticos en la misma gráfica
        autoplot(y, PI=T) +  #PI=false indica la banda no se muestra
            autolayer(y2, PI=F, col="red") + # añadir capas a autoplot
            autolayer(y3, PI=F, col="green")
```
 
### `plot.zoo`
Las funciones de trazado de `zoo`para serires temporales son de las mejores y están expresamente pensadas para crear graficas claras y atractivas. 


```r
    # Muestra de plot.zoo
    plot.zoo(datosESP,col=c("blue","red"), xlab="Año")
```
### Otras funciones gráficas
Existen multitud de librerías gráficas en R. Para usar con serie temporales suelen ser apropiadas las librerías financieras que contienen funciones grñaficas adaptadas y en algún caso muy buenas, tambien otras librerías especializadas en graficos interacticos como dygraphs, vemos algunas de ellas.

Las funciones `chartSeries` y `barChart` de la librería `quandmod`, o la función `chart.TimeSeries` de la librería  `PerformanceAnalytics` suelen dar buenos resultados vamos a ver ejemplos para nuestro curriculo.


```r
    # Ejemplo de graficas
    # Librería quantmod
    require(quantmod)
        chartSeries(datosESP$preciocasas, TA=NULL,theme=chartTheme('white'))
       # barChart(datosESP$preciocasas,TA=NULL,theme=chartTheme('white'))

    # Librería PerformanceAnalytics
    require(PerformanceAnalytics)
    chart.TimeSeries(datosESP)
    
    # Libraría dygraph, interactiva
    # install.packages("dygraphs")
    require(dygraphs)
    dygraph(datosESP)
```

![chartseries](imagenes/charts.png) 


![chart.TimeSeries](imagenes/chart.TimeSeries.png)

## Gráficas estacionales

La librería `forecast` contiene una funcion para representar *graficas estacionales* usando `ggseasonplot()`o también `seasonplot()`. Lo que hacen estas funciones es trazar una línea para cada ciclo completo y superponer todos juntos en la grafica. Es decir, para el caso de graficas con ciclos anuales, nos pintaría cada año superpuesto.

Una gráfica estacional es similar a una gráfica de tiempo excepto que los datos se dibujan en las "temporadas" individuales en las que se observaron los datos.

Estas funciones tienen una variante interesante que convierte los datos a coordenadas polares, donde el eje de tiempo es circular en lugar de horizontal. Para cambiar este sistema es suficiente con cambiar el argumento polar a VERDADERO.

Otra opción es el formato de grafica de subseries, que hace una mini gráfica de cada temporada y marca el valor medio de cada temporada se muestra como una línea azul.



```r
    require(ggplot2)  # para las graficas
    require(forecast) # para el ggseasonplot

    require(Quandl)   # para los datos
    # Quandl.api_key("lalala")
    # obtenemos los datos de población de España
    # vamos a usar datos de consumo de gas en USA
    gas<-Quandl("EIA/STEO_NGTCPUS_M", api_key="ybWqPvdv3r6bfrsNFkbq", type="ts")
    # podemos especificar las fechas de inicio y fin:
    autoplot(gas,main="US Natural Gas Consumption, Monthly")
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-22-1.png)<!-- -->

```r
    # Grafica estacional
    ggseasonplot(gas,year.labels=TRUE, main="Grafico estacional del gas")
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-22-2.png)<!-- -->

```r
    # Grafcia estacional en ByN 
    seasonplot(gas)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-22-3.png)<!-- -->

```r
    # Grafica estacional  sin superposición
    ggsubseriesplot(gas)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-22-4.png)<!-- -->

## Graficas de dispersión o Scatterplots
Si queremos hallar relaciones entre variables de una tabla temporal, tenderemos que hacer muchas gráficas de dispersión comparando cada observación de la variable x con la observación de la misma fecha de la variable y. 


```r
    library(ustyc)
    # obtener lacurva de rendimiento
    my.yield.curve <- getYieldCurve(year = 2017)
    # me guardo la tabla de curva de intereses
    curva.de.interes<-na.omit(my.yield.curve$df)
    # Quito las filas con ceros
    curva.de.interes<-curva.de.interes[apply(curva.de.interes!=0, 1, all),]
    
    qplot(BC_10YEAR, BC_5YEAR, data=curva.de.interes) +
        ylab("Tasa a 5 años") + xlab("Tasa a 10 años") +
        ggtitle("Comparar tasas de interés diferentes periodos 2017")
```

![](Series_temporales_en_R1_files/figure-html/ustyc_ejem-1.png)<!-- -->

```r
    # para ver las relaciones de todos con todos en un data frame   
    # vamos a representar las relaciones de todos 
    require(GGally) # paquete para pintar el grafico 
    curva.de.interes[,c(1,4,7,9,11)] %>% GGally::ggpairs()
```

![](Series_temporales_en_R1_files/figure-html/ustyc_ejem-2.png)<!-- -->


## Autocorrelogramas 
Otra forma de ver los datos de series de tiempo es trazar cada observación contra otra observación que ocurrió algún tiempo antes. Esto es lo que llamamos gráficas de retardo  y puede hacerse con la función `gglagplot()`.

Por ejemplo, trazar $$y_t$$ contra $$y_{t-_1}$$. Esto se llama **diagrama de retardo** porque está trazando la serie temporal contra los retardos (*lag*) de sí misma.

Si calculamos el coeficiente de correlación de los datos de cada gráfica de retardo, obtenemos una gráfica muy interesante que llamamos **grafica de autocorrelación (ACF)** de la serie o **autocorrelogramas**. Esta gráfica nos aporta mucha infomación valiosa para interepretar los datos, la existencia o no de ciclos, la magnitud de estos y la relación entre intervalos temporales. En R la función que usamos para pintarla es `ggAcf()`.

La variación de la gráfica de autocorrelación según el retardo es lo que denominamos **gráfica de autocorrelación parcial (PACF)**, es como la derivada de la función ACF. Las graficas de PACF son importantes como veremos para determinar los parametros de los modelos de regresión lineal ARIMA.

### Autocorrelación en series no estacionales


```r
    library(ggplot2)
    # Pintar la serie temporal completa
    autoplot(Nile)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-23-1.png)<!-- -->

```r
    # Pintar la grafica de autocorrelaciones 
    gglagplot(Nile,do.lines=FALSE)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-23-2.png)<!-- -->

```r
    # Tambien puede hacerse con la función lag.plot
    lag.plot(Nile, lags=10, do.lines =FALSE)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-23-3.png)<!-- -->

```r
    # Pintar la grafica de autocorrelación 
    ggAcf(Nile)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-23-4.png)<!-- -->
### Autocorrelación en series en series cíclicas
Cuando los datos son estacionales o cíclicos, el ACF alcanzará su punto máximo alrededor de los retrasos estacionales o en *la duración promedio del ciclo*.

Vamos a investigar el fenómeno de manchas solares con los datos disponibles en la base de datos `sunspot.year` y  comprobar que exste un ciclo de 10-11 años en los que se repite la insensidad de las mismas.


```r
    require(ggplot2)
    require("forecast")
    # Creamos el grafico de numero de manchas solares con el año.
    #autoplot(sunspot.year)
    autoplot.zoo(sunspot.year)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-24-1.png)<!-- -->

```r
    # Creamos un grafico de restrasos con los datos
    gglagplot(sunspot.year)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-24-2.png)<!-- -->

```r
    # Creamos un grafico ACF 
    ggAcf(sunspot.year)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-24-3.png)<!-- -->

Como vemos en el gráfico, las bandas azules indican los limites de ruido blanco. Si la autocorrelación es menor que esas marcas cuyo valor es $$\pm 2 / \sqrt{T}$$ donde $$T$$ es el numero de observaciones, es idicativo de que se trata de ruido blanco.

## Test de Ruido blanco
El ruido blanco es un término que describe datos puramente aleatorios. Para comprobar si una serie es o no ruido blanco se puede realizar la prueba de *Ljung-Box* utilizando la función `Box.test (serie, lag = 24, fitdf = 0, tipo = "Ljung")`. 

Si el resultado de `p` es un valor mayor que **0.05** se sugiere que los datos no son significativamente diferentes del ruido blanco, y si es inferio es posible que los datos sean predecibles pues se diferencian del ruido blanco.

Vamos a comprobar la "Hipótesis del mercado eficiente" que establece que los precios de los activos reflejan toda la información disponible, por lo que no son predecibles  son independientes de los valores anteriores. Una consecuencia de esto es que los cambios diarios en los precios de las acciones deben comportarse como _ruido blanco_ (ignorando los dividendos, las tasas de interés y los costos de transacción). La consecuencia para los pronosticadores sería que el mejor pronóstico del precio futuro es el precio actual.

Probaremos esto con los precios de google, (la serie `goog`), que contiene el precio de cierre de acciones para Google durante más de 1.000 días hábiles que finaliza el 13 de febrero de 2017. 


```r
    # leemos los datos de la web con quandmod
library("quantmod")
    valor1<-getSymbols("goog", from ="2016-09-15", to ="2017-02-13", src ="yahoo", warning= "FALSE",adjust = TRUE,auto.assign=FALSE)
    # numero de días de los datos
    ndays(valor1)
```

```
## [1] 103
```

```r
    # pintamos la serie origen
    autoplot(Cl(valor1))
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-25-1.png)<!-- -->

```r
    # Pintamos la serie de diferencias
    autoplot(diff(Cl(valor1)))
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-25-2.png)<!-- -->

```r
    #Pintamos la grafica de retrasos de autocorrelación
    ggAcf(diff(Cl(valor1)))
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-25-3.png)<!-- -->

```r
    # Por último realizamos el test de Ljung-Box
    # sobre la serie de diferencias
    Box.test(diff(Cl(valor1)), lag = 1, type = "Ljung")
```

```
## 
## 	Box-Ljung test
## 
## data:  diff(Cl(valor1))
## X-squared = 1.6217, df = 1, p-value = 0.2029
```
Como vemos el valor del test (p-value = 0.20) es bastante superior a 0.05, por lo que podemos afirmar que la serie de diferencias de las accioes de Google se comporta como ruido blanco y por tanto.

# PREDICIONES SIMPLES
Un pronóstico es la estimación del valor fururo de una serie temporal. No es difícil hacer predicciones a futuro en base a unos datos iniciales, la prueba es que existen métodos simples que han sido usados profusamente incluso con buenos resultados. En otras ocasiones es necesario conocer estas fórmulas sencillas de pronóstico para comparar sus resultados con otros modelos complejos y estimar su calidad, recordemos que el *principio de parsimonia* nos indica que las soluciones más simples suelen ser las correctas.

Veremos en este capítulo 4 modelos de pronóstico de formulación muy sencilla, contenidas dentro del paquete `forecast` de R:

    - método de la media: `meanf()`
    - pronóstico simple: `naive()`
    - pronóstico simple estacional: `snaive()`
    - modelo lineal:  `rwf()`
    
## Método de la media
Se trata simplemente de estimar el valor futuro de la serie como el valor medio de las observaciones dadas hasta la fecha de pronóstico. La función de desarrolla el modelo es `meanf()`


```r
    require(forecast)
    # Usamos la serie ts de ejemplo del cap1
    # creamos un pronostico simple de media
    p.med<-meanf(mits,20)
    #Pintamos el pronostico
    plot(p.med)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-26-1.png)<!-- -->



## Pronóstico simple o ingenuo
El método de pronóstico simple consiste en usar la última observación, la más reciente, como valor de predicción para el futuro próximo, esto es lo que se conoce como *pronóstico ingenuo*. Puede parecer tonto, pero existe una función específica para su calculo `naive()`y es, en muchas ocasiones el mejor pronóstico que podemos ofrecer.

Esta estimación sencilla es lo mejor que se puede tener para muchas series temporales, incluida la mayoría de los datos sobre el precio de las acciones, e incluso si no es un buen método de pronóstico, proporciona un punto de referencia útil para comparar con otros.

Para los datos estacionales o cíclicos, una idea similar es establecer como pronóstico el último valor de la misma temporada que la actual. Por ejemplo, si deseamos pronosticar el volumen de ventas para el próximo marzo, usaríamos el volumen de ventas del marzo del año anterior. Esto se implementa en la función `snaive()`(*sesonal naive*, es decir, ingenuo de temporada).


```r
    # pronostico ingenuo en serie tendencial
    naive(y, h = 10) ;

    # pronostico ingenuo en serie cíclica estacional
    snaive(y, h = 2 * frequency(y))`
```
`
Para ambas funciones de pronóstico, puede establecerse un segundo parámetro que indica el número de valores hacia el futuro a pronosticar (argumento `h`).

El paquete de R `forecast`nos facilita el uso de pronósticos sobre series temporales y contiene unas ayudas gráficas que representan la información de manera intuitiva y límpia.
Además la clase principal de objetos en el paquete `forecast` contiene muchas funciones para manejarlos, incluyendo `summary()` y `autoplot()`.



```r
    require(forecast)
    # Usamos naive() para hacer un pronostico ingenuo de la serie por 10 días
    p.simple <- naive(mits, 10)

    # Pintamos el pronóstico y el resumen
    plot(p.simple)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-28-1.png)<!-- -->

```r
    summary(p.simple)
```

```
## 
## Forecast method: Naive method
## 
## Model Information:
## Call: naive(y = mits, h = 10) 
## 
## Residual sd: 6.3414 
## 
## Error measures:
##                     ME     RMSE    MAE       MPE     MAPE      MASE
## Training set -1.247139 6.326138 5.0299 -17.43698 49.24824 0.3898674
##                   ACF1
## Training set 0.2012683
## 
## Forecasts:
##         Point Forecast     Lo 80     Hi 80     Lo 95        Hi 95
## 1996 Q3      -37.23004 -45.33731 -29.12276 -49.62904 -24.83103409
## 1996 Q4      -37.23004 -48.69545 -25.76462 -54.76487 -19.69519920
## 1997 Q1      -37.23004 -51.27224 -23.18783 -58.70574 -15.75433449
## 1997 Q2      -37.23004 -53.44458 -21.01549 -62.02804 -12.43203185
## 1997 Q3      -37.23004 -55.35845 -19.10163 -64.95505  -9.50502447
## 1997 Q4      -37.23004 -57.08872 -17.37136 -67.60127  -6.85880752
## 1998 Q1      -37.23004 -58.67986 -15.78021 -70.03471  -4.42535990
## 1998 Q2      -37.23004 -60.16086 -14.29921 -72.29971  -2.16036207
## 1998 Q3      -37.23004 -61.55185 -12.90822 -74.42704  -0.03302961
## 1998 Q4      -37.23004 -62.86748 -11.59259 -76.43912   1.97905146
```

Otro ejemplo con lectura de fichero Excel:


```r
    require(forecast)
    require(xlsx)
    
    paro.Murcia<-read.xlsx("sec1.xls", sheetIndex = 2, header = T)
    #, colIndex = "1:2")#, rowIndex ="7:120")
    
    paro<-xts(paro.Murcia[,2], order.by = as.Date(paro.Murcia[,1],"%Y-%m-%d %R"))
    colnames(paro)<-"Total_paro"
    head(paro)
```

```
##            Total_paro
## 2009-01-01      97659
## 2009-02-01     101894
## 2009-03-01     105889
## 2009-04-01     107773
## 2009-05-01     106737
## 2009-06-01     106148
```

```r
    # Usamos naive() para hacer un pronostico ingenuo de la serie por 10 días
    p.simple <- naive(paro, 12)

    # Pintamos el pronóstico y el resumen
    autoplot(p.simple)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-29-1.png)<!-- -->

##Pronóstico simple estacional

Para series estacionales la función de pronóstico simple añade una `s` inicial `snaive()`.
Veamos un ejemplo de uso.


```r
    # Usaremos una serie estacinal como wineind
    # seleccionamos un tramo de la serie que acaba en 1985
    vinos<-window(wineind,end=1985)
    # realizamos el pronostico simple estacional para 2 años
    p.simple <- snaive(vinos, 24)

    # Pintamos el pronóstico 
    autoplot(p.simple)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-30-1.png)<!-- -->

## Método lineal
El método lineal extrapola los valores futuros en base a las dos ultimas observaciones, con una simple interpolación lineal de los datos.

Este método está tambien incluido en el paquete `forecast` mediante la función `rwf(y, h, drift=TRUE)`.

```r
    # Pronostico lineal
    # usaremos la serie del oro
    # realizamos un pronostico simple para 50 pasos futuros
    p.simple <- rwf(gold[1:400], 50,drift=TRUE)

    # Pintamos el pronóstico 
    plot(p.simple)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-31-1.png)<!-- -->

# EVALUACIÓN DE PRONÓSTICOS
Vamos a ver fórmulas que nos permitan evaluar la bondad de las predicciones que realizamos.
Estas funciones se basan principalmente en medir la diferencia entre el valor pronosticado y el observado en la realidad.

## Residuos del pronóstico, función tubería
Llamamos *residuos* ($$e_t$$) a las diferencias entre cada valor pronosticado $$\hat{y_t}$$ al aplicar el modelo sobre los datos y su correspondiente valor observado $$y_t$$. Estas diferencias, para cada punto del pronóstico forman una nueva variable aleatoria, que podemos estudiar y que debe cumplir unas características adecuadas para asegurar que el modelo es correcto.

$$ e_t = y_t - \hat{y_t}$$

Por ejemplo, es bueno que los residuos sean pequeños, que tengan un comportamiento de ruido blanco y distribución normal de sus valores, esto indicaría que la parte de residuo **que no pronostica el modelo** es mínima e impredecible. En jerga estadística (regresion lineal) lo que se busca es que se cumplan en los residuos las premisas de linealidad, homocedasticidad y normalidad (<http://tabarefernandez.tripod.com/coco2.pdf>).

Por tanto, para analizar el buen comportamiento del modelo de pronóstico es importante verificar siempre que los **residuos** se comporten bien, esto es: sin valores atípicos, ni patrones y que se asemejen al ruido blanco y que se distribuya normalmente. 

La función `checkresiduals()` se usa para verificar estas características y entre ellas aporta el resultado de la prueba de `Ljung-Box` sobre el ruido blanco.

Para probar esta función es habitual el uso de la función de tubería (*%>%*) en el código R. Esta función es útil cuando hay funciones anidadas, en las que el resultado de una es tomado de entrada en la siguiente. Las funciones de tubería hacen que el código sea mucho más fácil de leer. 

Vamos a calcular los residuos de las funciones de pronóstico anteriores usando tambien la función tubería.


```r
    require(quantmod)
    require(ggplot2)    
    require(forecast)

    acc1 <- getSymbols("MSFT", 
                       src ="yahoo",
                       periodicity = "monthly",
                       warning= "FALSE",
                       adjust = TRUE,
                       auto.assign=FALSE)

    autoplot(Cl(acc1),main="Microsoft") +
        xlab("Mes") + ylab("Precio Microsoft en $")
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-32-1.png)<!-- -->

```r
    # Calculamos los residuos de los pronosticos ingenuos
    res<-residuals(naive(Cl(valor1)))
    # Grafica de residuos
    autoplot(res) + xlab("Mes") + ylab("residuos") +
        ggtitle("Residuos del metodo simple")
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-32-2.png)<!-- -->

```r
    # Grafica de histograma de residuos
    gghistogram(res) + ggtitle("Histograma de residuos")
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-32-3.png)<!-- -->

Podemos pintar un resumen de las anteriores gráficas con la función `checkresiduals()`.


```r
    # función checkresiduals
    Cl(valor1) %>% naive() %>% checkresiduals()
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-33-1.png)<!-- -->

```
## 
## 	Ljung-Box test
## 
## data:  Residuals from Naive method
## Q* = 16.958, df = 10, p-value = 0.07529
## 
## Model df: 0.   Total lags used: 10
```

```r
    # Calculamos el test de Ljung
    Cl(valor1) %>% diff() %>% Box.test(lag=1,type= "Ljung")
```

```
## 
## 	Box-Ljung test
## 
## data:  .
## X-squared = 1.6217, df = 1, p-value = 0.2029
```


## Datos de entrenamiento vs prueba
Antes de comenzar a trabajar en un modelo de pronóstico sobre datos, es preciso dividir la muestra de observaciones o la serie temporal de partida en dos grupos:

 1. un conjunto o varios para el **entrenamiento** del modelo
 2. otro conjunto con datos diferentes de **prueba** del modelo
 
El conjunto de entrenamiento será el que nos de los párámetros finales de clibración del modelo y debe ser diferente al conjunto de prueba, pues de otro modo, estaríamos sobreajustando los parámetros y caeremos en lo que se denomina (en inglés) *overfitting*, sobreajuste.

No se puede evaluar la bondad de un modelo de predicción sobre datos que se han utilizado para calibrar el modelo, pues los resultados serían falsos, no adecuadso. Un modelo sobreajustado  da muy buenos pronósticos en los datos de entrenamiento, pero suele dar malos en los datos de prueba. Esto indica que el modelo no ha detectado la señal principal en los datos y se ha centrado en el ruido, aunque tambien hay que comprobar que no exista un *bias* o desviación en los conjuntos de muestra.

Se logra por tanto un buen resultado, pero a costa de usar demasiados datos observables, lo que suele conllevar fallos en la predicción cuando se incluyen datos nuevos. El sobre-calibrado o sobreajuste es una muestra más del *principio de parsimonia*, o principio de la navaja de Ockham, que dice que en **igualdad de condiciones, la explicación más sencilla suele ser la más probable**. 

Por esto es necesario **siempre** reservar una cantida de datos para descubrir sus posibles relaciones y calibrar el modelo y otro conjunto significativo de observaciones distintas para las pruebas del mismo.

### Funciones para dividir una muestra
Una función que se puede usar para crear los grupos de entrenamiento y de prueba es `subset()` del paquete `forecast` en caso de series `ts`, que devuelve un subconjunto de una serie temporal donde los argumentos opcionales de inicio y fin se especifican usando valores de índice.


```r
    require("forecast")
    # convertimos los datos a ts
    observaciones<-ts(Cl(valor1))
    
    length(observaciones) # comprobamos la longitud de registros
```

```
## [1] 103
```

```r
    str(observaciones) # comprobamos el tipo de objeto
```

```
##  Time-Series [1:103, 1] from 1 to 103: 772 769 766 771 776 ...
##  - attr(*, "index")= atomic [1:103] 1473897600 1473984000 1474243200 1474329600 1474416000 ...
##   ..- attr(*, "tzone")= chr "UTC"
##   ..- attr(*, "tclass")= chr "Date"
##  - attr(*, ".indexTZ")= chr "UTC"
##  - attr(*, ".indexCLASS")= chr "Date"
##  - attr(*, "tclass")= chr "Date"
##  - attr(*, "tzone")= chr "UTC"
##  - attr(*, "src")= chr "yahoo"
##  - attr(*, "updated")= POSIXct[1:1], format: "2018-07-12 14:57:42"
##  - attr(*, "dimnames")=List of 2
##   ..$ : NULL
##   ..$ : chr "GOOG.Close"
```

```r
    # Creamos subconjunto de entrenamiento de las 50 primeras obsevaciones
    train<-subset(observaciones, end = 50)
    
    # Creamos subconjunto de prueba
    test<-subset(observaciones,start = 51, end = 103)
    
    # comprobamos la longitud de cada subconjunto
    length(train)
```

```
## [1] 50
```

```r
    length(test)
```

```
## [1] 53
```
Otra función que nos vale para dividir la muestra es `window()`, o las funciones típicas de data.frames como `head()` o `tail()` que nos permiten dividir series.


```r
    require("forecast")

    # Creamos subconjunto de entrenamiento y test
    # sobre las primeras 40 observaciones
    train<-window(observaciones, end = 40)
    test<-subset(observaciones, start = 41)
    
    # comprobamos la longitud de cada subconjunto
    print(paste("Num datos conjunto entrenamiento=", length(train)))
```

```
## [1] "Num datos conjunto entrenamiento= 40"
```

```r
    print(paste("Num datos conjunto prueba=", length(test)))
```

```
## [1] "Num datos conjunto prueba= 63"
```
## Medidas del error de predicción 
Dado que el residuo o error de las prediciones dependen de las escala de los datos observados, debemos buscar una fórmula que nos permita comparar diferentes modelos de ajuste sin la dependencia esta escala incial de los datos.

Hay varios indicadores en este sentido, por ejemplo los dos siguientes ns permiten comparar diferentes modelos en una misma serie, siendo el MAE más facil de interpretar.

  * **MAE**(Mean absolute error) o valor absoluto medio del error $$MAE=media(|e_t|)$$
  * **RMSE** o raiz cuadrada del error medio cuadrático = $$ RMSE=\sqrt{media(e_t^2)}$$

Otras de las medidas como el MAPE permiten desescalar los resultados a porcentaje lo que pemite comparar diferentes series y modelos. Todas estas medidas tienen sus ventajas y desventajas, por ejemplo MAPE sobrevalora los errres positivos, o las escalas con un cero, por lo que hay otras mediciones como **sMAPE** o incluso el **MASE** que tratan de paliar estas consecuencias con mejores , pero más complejos estimadores.
 
 * **MAPE** (mean absolute mean error) $$MAPE= mean(|100e_t/y_t|)$$ este valor es el mejor par series con estacionalidad.

## Precisión del modelo en series de tendencia

La función `accuracy(f,x,...)` del paquete `forecast`calcula varias estadísticas referentes al ajuste del pronóstico, dados las predicciones y también las observaciones reales correspondientes. Es una función muy completa a la que se le debe aportar como parámetros:

 * **f** es un objeto de clase `forecast`
 * **x** es un vector numérico o una serie temporal

Entre los estadísticos que da están los vistos en el aprtado anterior como RMSE, MSE etc...
Cuanto menor sea el valor de esta medidas, por ejemplo de RMSE, mejor es el pronóstico, más se ajusta a lso datos observados.

Vamos a comparar dos modelos de pronostico con esta función para ver cual es el mejor.


```r
    # Trabajaremos sobre la serie del ventas de vinos en Australia que ya usamos anteriormente
    # pintamos la serie 
    vinos<-window(wineind,end=1985)
    autoplot(vinos)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-36-1.png)<!-- -->

```r
    # Longitud de la serie
    length(vinos)
```

```
## [1] 61
```

```r
    # Creamos el conjunto de entrenamiento
    train <- subset(vinos, end = 50)

    # 1. Metodo simple
    # Calculamos el pronostico ingenuo, para los proximos 11 intervalos
    pronos_s1 <- naive(train, h = 11)
    autoplot(pronos_s1)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-36-2.png)<!-- -->

```r
    # 2. Metodo simple estacional     
    # Calculamos el pronostico de media anterior
    pronos_s2 <- snaive(train, h = 11)
    autoplot(pronos_s2)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-36-3.png)<!-- -->

```r
    # 3. Comparamos resultados
    # Usamos accuracy()  para calcular el RMSE 
    accuracy(pronos_s1, vinos)
```

```
##                     ME     RMSE      MAE       MPE     MAPE     MASE
## Training set  141.6531 5314.319 3833.041 -2.200382 18.18678 2.450546
## Test set     4671.7273 6977.012 5331.182 14.408067 17.98235 3.408340
##                    ACF1 Theil's U
## Training set -0.2390462        NA
## Test set     -0.2334582  1.035587
```

```r
    accuracy(pronos_s2, vinos)
```

```
##                    ME     RMSE      MAE      MPE     MAPE     MASE
## Training set 1284.053 2008.864 1564.158 5.295017 6.491853 1.000000
## Test set     1254.364 1954.547 1608.545 4.253698 5.780763 1.028378
##                    ACF1 Theil's U
## Training set -0.1997403        NA
## Test set     -0.4350287  0.285696
```

```r
    # Podemos acceder a valroes concetor
    accuracy(pronos_s1, wineind)["Test set", "MAPE"]
```

```
## [1] 17.98235
```

```r
    accuracy(pronos_s2, wineind)["Test set", "MAPE"]
```

```
## [1] 5.780763
```

```r
    checkresiduals(pronos_s2)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-36-4.png)<!-- -->

```
## 
## 	Ljung-Box test
## 
## data:  Residuals from Seasonal naive method
## Q* = 21.658, df = 24, p-value = 0.5997
## 
## Model df: 0.   Total lags used: 24
```



## Validación cruzada con `tsCV()`
Otra de las funciones que nos permiten evaluar la bondad de un modelo de pronóstico es aplicar la validación cruzada a los datos de prueba. La función `tsCV()` (*En inglés time serie Cross Validation*) calcula los errores (es decir la diferencia entre el valor esperado y el observado) en un pronóstico realizado con una función determinada, para cada valor de la serie temporal.


```r
    # argumentos de la función tsCV()
    args(tsCV)
```

```
## function (y, forecastfunction, h = 1, window = NULL, ...) 
## NULL
```

La validación cruzada consiste en aplicar calcular los pronósticos con datos de la serie de prueba tomando cada vez un elemento más de la misma serie, y tratandolos como series independientes.
Es decir, considera como conjunto de entrenamiento todos los puntos de la serie hasta uno determinado, y con eso pronostica una serie de pasos adelante, y repite esta misma acción considerando un nuevo intervalo.


![validacion cruzada con h=4](imagenes/cv4-1.png)

La llamada a la función `tsCV()` requiere que se de la *serie temporal*, el *método de pronóstico* y el *horizonte de pronóstico*. El resultado es una nueva serie con el error del valor pronosticado para cada intervalo de la serie temporal.

Haciendo estadísticas de estos errores podemos saber la precisión del modelo de pronóstico.

Veamos un ejemplo en el que calcularemos el MSE (Mean Square Error= erro medio al cuadrático). Lo de elevarlo al cuadrado es para obtener valor absoluto del error.


```r
    require(quantmod)
    #Leemos los datos de Santander en bolsa de Madrid
    acc1 <- getSymbols("SAN.MC", 
                       src ="yahoo",
                       periodicity = "monthly",
                       warning= "FALSE",
                       adjust = TRUE,
                       auto.assign=FALSE)    
    
    autoplot(acc1$SAN.MC.Close)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-38-1.png)<!-- -->

```r
   e <- matrix(NA_real_, nrow = length(acc1$SAN.MC.Close), ncol = 8)
   
    # Creamos una matriz que va a contener los errores de diferentes pronósticos.
    # en este caso haremos la validación cruzada para 8 pasos adelante
    # y necesitamos almacenarlo en una matriz de la misma longitud que la serie y 8 columnas
    # una por cada paso o pronostico
    # Creamos matriz de NA_reales   
    for (h in 1:8){
      e[, h] <- tsCV(acc1$SAN.MC.Close, forecastfunction = naive, h = h)
    }
   
    # Calcularemos pel MSE ara cada pronostico 
    mse <- colMeans(e^2, na.rm = TRUE)
    
    # Pintamos la evolución del MSE contra el horizonte de pronostico.
    data.frame(h = 1:8, MSE = mse) %>%
      ggplot(aes(x = h, y = MSE)) + geom_point()  
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-38-2.png)<!-- -->
Como vemos en el ejemplo anterior a mayor horizonte de pronsóstico, el error MSE es mayor.

## Banda de predicción
Los graficos que estamos usando por defecto para representar pronósticos en la libraría `forecast` incluyen, no solo el valor predicho para cada horizonte de pronostico, sino también unas bandas de colores que indican los intervalos de predición del 80 y 95%.

Se trata de un rango de probabilidad en los que se espera que esté el pronostico, suponiendo que las prediciones tengan una distribución normal.


```r
    # Bandas de predición de pronóstico
    p.acc<-forecast(acc1$SAN.MC.Close,h=20)
    plot(p.acc)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-39-1.png)<!-- -->


# MODELOS DE REGRESIÓN LINEAL

En capítulos anteriores hemos visto algunos modelos muy simples, como el modelo ingenuo que pronostica el último valor observado como predicción futura. Vamoa a estudiar en este capítulo otros modelos para series temporales basados en regresión lineal.

## Regresión lineal simple
Cuando tenemos dos variables relacionadas se puede utilizar la regresión lineal para estimar una de ellas a partir del valor de la otra. Para esto usaremos las funciones gráficas que devuelven directamente el modelo de  regresión de mejor ajuste, o las funciones como `lm()` que aportan los parámetros de la regresión.


```r
    # tomamos dos variables : desempleo en españa y precio de la vivienda
    # aunque pongamos fechas, los datos del FRED se leen enteros y  no hace caso a las fechas de inicio y fin
    require(quantmod)
    require(ggplot2)
    require(dplyr) # para pipe operator %>%
    # desempleo datos ESP trimestrales
    desESP <- getSymbols("LRUN64TTESQ156N", 
                       src ="FRED",
                       warning= "FALSE",
                       from ="2006-01-01", to ="2017-01-01",
                       adjust = TRUE,
                       auto.assign=FALSE)  
    # precio vivienda ESP datos trimestrales
    pcasasESP <- getSymbols("QESN628BIS", 
                       src ="FRED",
                       warning= "FALSE",
                       from ="2006-01-01", to ="2017-01-01",
                       adjust = TRUE,
                       auto.assign=FALSE) 
    
    #    plot(pcasasESP)   
    # Unimos las dos variables
    datosESP<- cbind(desESP,pcasasESP)
    # Eliminamos las filas sin ambos datos
    datosESP<-datosESP[complete.cases(datosESP),]
    head(datosESP)
```

```
##            LRUN64TTESQ156N QESN628BIS
## 2005-10-01             8.8      91.93
## 2006-01-01             9.1      95.35
## 2006-04-01             8.5      99.43
## 2006-07-01             8.1     102.68
## 2006-10-01             8.3     105.34
## 2007-01-01             8.5     107.87
```

```r
    # Cambiamos los nombres de las columnas
    colnames(datosESP)<-c("desempleo","preciocasas")

  # pintamos la grafica
    plot.zoo(datosESP,col=c("blue","red"))
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-40-1.png)<!-- -->

```r
    require(lubridate) # para la fucnion year que convierte a años una fecha
    #Pintamos la relación grafica
    datosESP %>% as.data.frame()  %>% ggplot(aes(x=desempleo, y=preciocasas)) +
        ylab("Precio vivienda en España") +
        xlab("Desempleo en España") +
        geom_point(size=2, color=year(index(datosESP))) +
        geom_smooth(method="lm", se=TRUE)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-40-2.png)<!-- -->

```r
    # Los coeficientes de regresion de la grafica son los obtenidos por a funcion:
    lm(desempleo ~ preciocasas, data=as.data.frame(datosESP))
```

```
## 
## Call:
## lm(formula = desempleo ~ preciocasas, data = as.data.frame(datosESP))
## 
## Coefficients:
## (Intercept)  preciocasas  
##     49.3092      -0.3467
```

```r
    # y= intercep + coeficient x    
    # el primero es el valor de interseccion del eje y
    #el segundo es la pendiente de la linea de regresión
    
    # Para añadir la linea a una grafico podriamos usar:
    # abline(intercep, coeficient) en plot
    # geom_abline(intercep, coeficient) en ggplot
```

## Correlación entre variables
Si tenemos varias variables relacionadas, una forma de ver graficamente los coeficientes de correpación entre cada par de variables es usar la función `ggpairs()` del paquete **GGally**. El resultado nos dá la matriz gráfica de autocorrepaciones entre las series.



```r
    #Añadimos a la serie anterior de datosEsp las exportaciones
    expESP <- getSymbols("ESPEXPORTQDSMEI", 
                       src ="FRED",
                       warning= "FALSE",
                       from ="2006-01-01", to ="2017-01-01",
                       adjust = TRUE,
                       auto.assign=FALSE)

    # añadimos la serie de exportaciones
    datosESP<- cbind(datosESP,expESP)
    
    # Eliminamos las filas sin ambos datos
    datosESP<-datosESP[complete.cases(datosESP),]
     colnames(datosESP)[3]<-"export"
    plot.zoo(datosESP)    
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-41-1.png)<!-- -->

```r
    # También podemos ver la relacioó entre las variables con GGally
    datosESP %>%
        as.data.frame() %>%
        GGally::ggpairs() 
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-41-2.png)<!-- -->

## Regresión múltiple
La función `tlsm()` del paquete forecast, realiza pronósticos de regresion lineal a partir de la descomposición de una serie temporal. El modelo permite especificar una formula de pronostico. 


```r
    require(forecast)
    # Creamos una serie de ejemplo periodica y con tendencia
    y <- ts(rnorm(100,0,5) + 1:100 + 20*sin(2*pi*(1:100)/12), frequency=12, start = 2010)
    plot.zoo(y)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-42-1.png)<!-- -->

```r
    # predicción usando tendencia y estacionalidad
    fit <- tslm(y ~ trend + season)
    plot(forecast(fit, h=24))
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-42-2.png)<!-- -->

```r
    # predicción solo usando la tendencia 
    fit <- tslm(y ~ trend)
    plot(forecast(fit, h=24))
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-42-3.png)<!-- -->


## Suavizamiento exponencial simple
La función `ses()` produce pronósticos de regresión obtenidos usando la suavización exponencial simple (SES). Los parámetros óptimos los obtiene la función internamente minimizando el error por mínimos cuadrados. El uso práctico de la serie es simple, solo se necesita especificar la serie de tiempo y el horizonte de pronóstico. 

Se usa este método de pronostico en series que no reflejan claramente ni tendencia ni estacionalidad, series de difécil pronóstico, en las que pueden dar mejores resultados incluso los métodos ingenuos.

La función `fitted()` se puede utilizar para pintar los datos pronosticados en un paso adelante con el modelo.


```r
    # Uso de ses() en el pronostico de una serie a h=20
    dat<-window(AirPassengers,start=c(1960,3))
    plot(dat)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-43-1.png)<!-- -->

```r
    # Creamos el pronostico al horizonte 3
    fc <- ses(dat, h =3)
    # Usamos summary() para ver los resultados
    head(summary(fc))
```

```
## 
## Forecast method: Simple exponential smoothing
## 
## Model Information:
## Simple exponential smoothing 
## 
## Call:
##  ses(y = dat, h = 3) 
## 
##   Smoothing parameters:
##     alpha = 0.9999 
## 
##   Initial states:
##     l = 418.996 
## 
##   sigma:  56.8325
## 
##      AIC     AICc      BIC 
## 109.8280 113.8280 110.7358 
## 
## Error measures:
##                    ME     RMSE      MAE       MPE     MAPE MASE      ACF1
## Training set 1.300111 56.83252 47.70159 -0.340655 9.726179  NaN 0.3659019
## 
## Forecasts:
##          Point Forecast    Lo 80    Hi 80    Lo 95    Hi 95
## Jan 1961       431.9958 359.1620 504.8296 320.6061 543.3855
## Feb 1961       431.9958 328.9984 534.9932 274.4749 589.5167
## Mar 1961       431.9958 305.8524 558.1392 239.0761 624.9155
```

```
##          Point Forecast    Lo 80    Hi 80    Lo 95    Hi 95
## Jan 1961       431.9958 359.1620 504.8296 320.6061 543.3855
## Feb 1961       431.9958 328.9984 534.9932 274.4749 589.5167
## Mar 1961       431.9958 305.8524 558.1392 239.0761 624.9155
```

```r
    # Pintamos la serie con el pronostico y la serie ajustada
    autoplot(fc) + autolayer(fitted(fc))
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-43-2.png)<!-- -->

Como vemos el modelo SES es bastante simple y los resultados difieren poco del modelo ingenuo (`naive()`). 

## Holt para tendendicas
El método de tendencia local de Holt se implementa en la función `holt()`, es aplicable a series con una clara tendencia, no es aplicable a series estacionales. La función tiene el parametro damped que sirve para aplicar un suavizado a la curva de pronostico.


```r
    # Pronostico de hotl para la serie diaria de DAX en 1991-1994
    dat<-window(EuStockMarkets[,"DAX"], start=1991, end=1994)
    # str(dat)    
    # plot(dat)
    
    #Realizamos un pronostico a 100 días por holt
    fc <- holt(dat	,h=100) # esto para tendencias de pendiente constante
    
    # Pronostico con version con suavizado
    fc1 <- holt(dat,h=100, damped = TRUE)
    #summary(fc)
    autoplot(fc, PI=F) + autolayer(fc1, PI=F, lwd=2, col="red")
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-44-1.png)<!-- -->

```r
    # probamos que los residuos sean ruido blanco
    checkresiduals(fc)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-44-2.png)<!-- -->

```
## 
## 	Ljung-Box test
## 
## data:  Residuals from Holt's method
## Q* = 530.97, df = 516, p-value = 0.3148
## 
## Model df: 4.   Total lags used: 520
```

## Holt-Winters para datos estacionales
La función `hw()` produce pronósticos usando el método de Holt-Winters específico para cualquier serie estacional. Hay que especificar el método elegido de pronostico si aditivo  multiplicativo. El primero añade ciclos iguales a los actuales y el segundo son ciclos ampliados.



```r
    # Trabajaremos con la serie del oro de ejemplo
    # queremos un pronostico a 3 años
    fc <- hw(gold, seasonal = "additive", h =12*3)

    #fc1 <- hw(gold, seasonal = "multiplicative", h =12*3)
    autoplot(fc)
    #  autoplot(fc) + autolayer(fc1, PI=F, lwd=2, col="red")
    checkresiduals(fc)
```

![prediccion de oro con holtwinter](imagenes/holtwinter.png)


## Predicción automática con `ets`
La función ETS (alisamiento exponencial), proporciona una forma completamente automática de realizar pronósticos para una amplia gama de series de temporales. La llamada de la funcion es algo diferente, pues `ets()` ajusa los parámetros del modelo, pero para realizar el pronóstico. Digamos que selecciona previamente qué modelo elegir que se ajuste mejor, y después lousaremos como un función más a través de la llamada a `forecast()`.


```r
    # Calculamos un pronóstico automático con ETS
    fc <- forecast(ets(gold),h=3*12)
    # Pintamos los datos  del pronóstico 
    autoplot(fc) 
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-46-1.png)<!-- -->

```r
    # Análisis de residuos
    #checkresiduals(fc)
```
Esta función realiza el ajuste con métodos internos, por lo que automatiza mucho el proceso y los resultados son ajustados, aunque a veces el método ingenuo tenga menor error como podemos ver en el siguiente ejemplo.


```r
    # CReamos una función para almacenar los resultados de 
    # pronosticos ETS 
    p_ets <- function(y, h) {
      forecast(ets(y), h = h)
    }
    
    # aplicamos la validación cruzada tsCV() a los métodos para horizonte de 6 meses
    e1 <- tsCV(gold, forecastfunction = p_ets, h = 6)
    e2 <- tsCV(gold, forecastfunction = snaive, h = 6)
    
    # calculamos el MSE de cada resultado)
    mean(e1^2, na.rm = TRUE)
    mean(e2^2, na.rm = TRUE)
```
Como vemos el pronostico de `ets()` no siempre es bueno, incluso en series cíclicas no estacionales funciona bastante mal como por ejemplo con la serie *lynx* que contiene el número de trampas de lince entre 1821-1934.


```r
    # Pintamos la serie lynx{datasets}
    require(ggplot2)
    autoplot(lynx)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-48-1.png)<!-- -->

```r
    # Ajustamos un modelo ets a la serie
    fit <- ets(lynx)
    
    # veos los parametros del calculo 
    summary(fit)
```

```
## ETS(M,N,N) 
## 
## Call:
##  ets(y = lynx) 
## 
##   Smoothing parameters:
##     alpha = 0.9999 
## 
##   Initial states:
##     l = 169.2223 
## 
##   sigma:  0.9489
## 
##      AIC     AICc      BIC 
## 2052.369 2052.587 2060.578 
## 
## Training set error measures:
##                    ME     RMSE      MAE       MPE     MAPE      MASE
## Training set 28.30726 1182.181 824.4878 -44.94341 94.83425 0.9923325
##                   ACF1
## Training set 0.3785704
```

```r
    # Pintamos un pronostico a 20 años
    fit %>% forecast(h=20) %>% autoplot()
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-48-2.png)<!-- -->

```r
    # sin embargo con series estacionales si va bien
    autoplot(AirPassengers)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-48-3.png)<!-- -->

```r
    fit <- ets(AirPassengers)
    fit %>% forecast(h=20) %>% autoplot()
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-48-4.png)<!-- -->

# TRANSFORMACIONES y DESCOMPOSICIÓN DE SERIES TEMPORALES

El propósito de la transformación de series temporales es simplificar los patrones en los datos  y quitar las fuentes de variación evidentes. Se trata en definitiva, de poder aplicar los métodos de pronóstico estadístico y por tanto de obtener con la transformación una serie estacionaria sobre la que trabajar mejor.

Las transformaciónes de las series temporales extraen la esencia de los datos, y permiten analizar los componentes de tendencia, ciclo y variabilidad de forma independiente y mejorar la predicción.

La descomposición es la separación en componentes de una serie temporal,buscando tambien la simplificación del analisis en grupos sencillos. La principal descomposición que realizamos es extraer de la serie original las componentes de tendencia, ciclo y remanente o parte que no se puede explicar.

## Diferenciación
La diferrenciación de una serie consiste en tranformar la original en otra con las diferencias o incrementos en cada x pasos. Es decir, la resta de dos valores consecutivos de la serie. Con esto conseguimos en muchos casos **eliminar la tendencia de la serie** y obtener una variable estacionaria.
La diferenciación elimina patrones sistemáticos como tendencia y estacionalidad de los datos. 

La función `diff()` aplicada sobre una serie temporal nos devuelve la serie diferenciada. Como veremos en el siguiente ejemplo en la serie diferenciada la gráfic de acf cae inmediatamente a valores próximos a cero, lo que indica que se trata de una serie estacionaria. Por contra las series no estacionarias tienen valores altos y positivos como la serie original. 


```r
    require(quantmod)
    require(ggplot2) 
    # leemos los ultimos 10 años del ibex en diarios
    acc1 <- getSymbols("^IBEX", 
                       src ="yahoo",
                       periodicity = "daily",# "monthly",
                       warning= "FALSE",
                       adjust = TRUE,
                       auto.assign=FALSE)
    
    acc.dif<-diff(Cl(acc1))
    #length(acc.dif)
    acc.dif<-na.omit(acc.dif)
    acc1<-na.omit(Cl(acc1))
    png('imagenes/grafibex.png',height = 500, width = 600, units = "px")
        par(mfrow=c(2,2),mar=c(3,3,2,2))
        #spectrum(acc.dif)
        plot.zoo(acc1,main="IBEX 35")
        acf(acc1)
        plot.zoo(acc.dif,main="Diff del IBEX 35")
        acf(acc.dif, ylim=c(-0.1,0.1))
    dev.off()
    
    Box.test(acc.dif, lag=10, type="Ljung-Box")
```

![Diferenciación de la serie daria del IBEX y autocorrelación](imagenes/grafibex.png)

Cuando la diferenciación de un retardo no proporciona una serie estacionaria, puede probarse la segunda derivada - `diff(diff(serie))`- o también la diferenciación por retardos diferentes por ejemplo para desestacionalizar una serie. Si el ciclo es anual, puede hacerse `diff(serie, lag=12)` por ejemplo.

Para saber qué intervalo es el adecuado para diferenciar existen algunos test como el `kpss`.


```r
    library(urca)
    acc1 %>% ur.kpss() %>% summary()

    # Calculo del lag apropiado
    ndiffs(acc1)
```

```r
####################### 
# KPSS Unit Root Test # 
####################### 

Test is of type: mu with 9 lags. 

Value of test-statistic is: 8.4046 

Critical value for a significance level of: 
                10pct  5pct 2.5pct  1pct
critical values 0.347 0.463  0.574 0.739
```
El valor del test para la serie original de 8.4046 es muy alto indicando que habría que diferenciar la serie. PAra ver que valor de retardo lag de diferencia es apropiado podemos usar la función `ndiffs(serie)`.



## Transformaciones de Box-Cox 
La transformación de Box-Cox se usa para estabilizar la *varianza* de la serie, esto es, para que las variaciones sean comparables en escala a lo largo del tiempo. Se aplica por tanto en series estacionales o con ciclos. 
La función `BoxCox()` tiene dos argumentos principales, la serie origen y `lambda`. El calculo de lambda que hace a la serie menos variable puede hacerse de forma automática con la función `BoxCox.lambda(serie)`. 

Por ejemplo, en la serie `AirPassengers` vemos ciclos estacionales que además crecen con el tiempo, para estabilizar la varianza aplicamos al ejemplo una transformación Box COx con lambda=-0.2947156, calculado automáticamente por la función BoxCox.lambda. Como vemos en las gráficas los ciclo resultantes son más iguales.


```r
    # Serie de num de pasajeros
    autoplot(AirPassengers) 
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-52-1.png)<!-- -->

```r
    # Cálculo de lambda=BoxCox.lambda(AirPassengers)
    autoplot(BoxCox(AirPassengers,lambda=BoxCox.lambda(AirPassengers)))
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-52-2.png)<!-- -->

## Logaritmo
Una de las transformaciones clasicas de la series con comportamientos exponenciales o de gran variabilidad es el logaritmo. Haciendo un log de la serie conseguimos cambiar las tendencias exponenciales a lineales y las variaciones e vuelven más uniformes por lo que podemos trabajar con las series de manera habitual.

Vamos a ver los resultados para una serie como el precio del Bitcoin con eneorme variación de escala


```r
    # Intalamos el paquete

    library(coindeskr) #R-Package connecting to Coindesk API

    # obtenemos los precios historicos del cambio BTC EUR
    pBTC<-get_historic_price(currency = "USD", start = "2013-01-01" ,end = Sys.Date() )
    plot.ts(pBTC)
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-53-1.png)<!-- -->

```r
    plot.ts(log(pBTC))
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-53-2.png)<!-- -->

Vamos a ver un ejemlo:


```r
    y<-ts(na.omit(log(pBTC)))
    y1<- window(y, end =1600)
    
    autoplot(forecast(ets(y1),300), PI=F) + autolayer(window(y, start=1600))
```

![](Series_temporales_en_R1_files/figure-html/unnamed-chunk-54-1.png)<!-- -->


## Descomposición de series
### `decompose`
Es un método clásico para series con estacionalidad. Se puede aplicar con dos tipos, el aditivo y el multiplicativo, uno suma y otro divide la estacionalidad en proporción al valor observado de tendencia.
El resultado de aplicar las funiones de descomposición de series `decompose()` es una terna de series formado por el componente de tendencia, el de ciclo y el que resta que podemos decir es el aleatorio.

Veamos un ejemplo:


```r
AirPassengers %>% decompose(type="multiplicative") %>%
  autoplot() + xlab("Year") +
  ggtitle("Descomposición de la serie de pasajeros")
```

![](Series_temporales_en_R1_files/figure-html/decompose-1.png)<!-- -->

### `seasonal`
Otro método de descomposición de series es el `X11` que en R está programado dentro del paquete `seasonal`. Tienen parecidas características, pero aporta algunos gráficos interesantes que vamos a mostrar. Una de las novedades es que el componente estacional no es constante.


```r
    require(seasonal)
    AirPassengers %>% seas(x11="") -> fit
    
    autoplot(fit) +
  ggtitle("X11 decomposicion")
```

![](Series_temporales_en_R1_files/figure-html/x11-1.png)<!-- -->

```r
    autoplot(AirPassengers, series="Datos") +
        autolayer(trendcycle(fit), series="Tendendica")    +
        autolayer(seasadj(fit), series="Estacional ajustada")    +
        xlab("Año") + ylab("Pasajeros") +
        ggtitle("Evolución de pasajeros - descomposición") +
        scale_colour_manual(values=c("gray","blue","red"),
             breaks=c("Datos","Estacional ajustada","Tendendica"))
```

![](Series_temporales_en_R1_files/figure-html/x11-2.png)<!-- -->

```r
    # otro grafico interesante
    fit %>% seasonal() %>% ggsubseriesplot() + ylab("Temporada") 
```

![](Series_temporales_en_R1_files/figure-html/x11-3.png)<!-- -->
El paquete `seasonal` da acceso a las series descompuestas con las funciones  `seasadj()`, `trendcycle()` and `remainder()` (estacional, tendencia y resto).

### `stl`
Como última alterntiva veremos la funcion `stl()` que también realiza la descomposción de una serie estacional, con la ventaja de que no solo es válido como las anteriores para series de ciclos mensuales, anuales o semanales, es una formulación más aséptica, y válida para cualquier frecuencia.

Además permite que el componente estacional varíe en el tiempo, no como `decompose`, y es más robusto frente a outliers o valores fuera de rango, pero hay que tener cuidado con estas funciones, pues no tienen ajuste automatico a calendarios y por tanto en series con secuencias discontinuas como datos diarios de bolsa (donde faltan los fines de semana y festivos) hay -previamente- que hacer alguna transformación temporal.

Los parámetros de la función son:

 - *t.window* que controla la tendencia (indica el lag que se toma en consideración para la tendencia)
 - *s.window* que controla el ciclo. (si se pone ="periodic" lo calcula automáticamente)
 

```r
AirPassengers %>%
  stl(t.window=12, s.window="periodic", robust=TRUE) %>%
  autoplot()
```

![](Series_temporales_en_R1_files/figure-html/stl_decompose-1.png)<!-- -->


## Pronosticos sobre descomposiciones de series
Una vez hemos realizado una descomposición de la serie que creemos adecuada, pordemos realizar pronosticos sobre los datos, y modular estos según cada componente de la descomposición. Esto es, podemos ejecutar un pronostico de tendencia sobre la parte de tendencia de la descomposición, y aplicarle luego la modulación que aporta la arte de estacionalidad.


```r
    # creamos una descomposición de la serie gold 
    smodel<- stl(gold, s.window="periodic") 
    autoplot(smodel)
    # Creamos un pronostico con el metodo drif de la parte de estacionalidad ajustada de la serie descompuesta
    smodel %>% seasadj() %>%  rwf(24,drift=TRUE)  %>% autoplot()
    
    # Idem pronostico con el metodo drif de la parte de tendencia de la descomposición
    # para 12 meses
    smodel %>% trendcycle() %>%  rwf(12,drift=TRUE)  %>% autoplot()
    
    # lo mismo usandola función forecast
    smodel %>% forecast(method="rwdrift") %>% autoplot()
    # lo mismo usando la función stlf en la que hay que indicar el método de pronostico
    autoplot(stlf(gold, method='rwdrift'))
    
    #Podemos usar un modelo ets
    smodel<- ets(gold) 
    smodel %>% forecast() %>% autoplot()
```

![autoplot(smodel)](imagenes/stl1.png)

![smodel %>% seasadj() %>%  rwf(24,drift=TRUE)  %>% autoplot()](imagenes/stl2.png)

![smodel %>% trendcycle() %>%  rwf(12,drift=TRUE)](imagenes/stl3.png)

![stl modelo](imagenes/stlrwdrift.png)

![pronosico con ets modelo](imagenes/stl_ets.png)

# MODELOS ARIMA 

Los modelos de regresión exponencial y los ARIMA (o modelos de media movil integrada autoregresiva) son los más ampliamente utilizados para el pronóstico de series temporales. Los primeros se basan en la tendencia y estacionalidad de los datos y los ARIMA en la autocorrelación de las serie.

## AR
Los modelos ARMA predicen los valores futuros mediante la combinación lineal de los valores pasados. Un modelos autoregresivo sigue la siguiente ecuación: $$y_t=c+a_1y_t_−1+a_2y_t_−2+ ⋯+a_ y_t−p+e_t$$

## MA 
Los modelos MA predicen valores futuros mediante la combinación lineal de los errores de pronostico pasados.

## ARIMA 
Combinando diferenciación con modelos autoregesivos con media movil tenermos los ARIMA. La nomeclatura de los modelos ARIMA(p,d,q) significa:

- p= orden de la parte autorregresiva
- d = grado de diferenciación del modelo
- q = orden de la media movil

Muchos de los modelos que hemos descrito son casos especiales de ARIMA:

| ruido blanco| ARIMA(0,0,0) |
| paseo aleatorio | ARIMA(0,1,0) |
| paseo aleatorio con deriva | 	ARIMA(0,1,0) |
| AR 	| ARIMA(p,0,0) |
| MA | 	ARIMA(0,0,q) |

Veamos un ejemplo de uso:


```r
    # Ajustamos un modelo ARIMA automáticamente  a la serie de oro
    fit <- auto.arima(gold)
    
    # Calculamos los residuos y el resumen 
    checkresiduals(fit) 
```

![](Series_temporales_en_R1_files/figure-html/ARIMA-1.png)<!-- -->

```
## 
## 	Ljung-Box test
## 
## data:  Residuals from ARIMA(1,1,2) with drift
## Q* = 10.087, df = 6, p-value = 0.121
## 
## Model df: 4.   Total lags used: 10
```

```r
    summary(fit)
```

```
## Series: gold 
## ARIMA(1,1,2) with drift 
## 
## Coefficients:
##           ar1      ma1      ma2   drift
##       -0.1628  -0.1744  -0.0552  0.0713
## s.e.      NaN      NaN      NaN  0.1151
## 
## sigma^2 estimated as 32.47:  log likelihood=-3410.82
## AIC=6831.64   AICc=6831.7   BIC=6856.69
## 
## Training set error measures:
##                         ME     RMSE      MAE          MPE      MAPE
## Training set -0.0005567974 5.774345 3.097751 -0.008251843 0.7782533
##                  MASE        ACF1
## Training set 1.007258 0.003941559
```

```r
    # Pintamos el pronostico para un año
    fit %>% forecast(h = 12) %>% autoplot()
```

![](Series_temporales_en_R1_files/figure-html/ARIMA-2.png)<!-- -->



```r
    # para un modelo concreto
    arima(lh, order = c(1,0,1))
    arima(USAccDeaths, order = c(0,1,1), seasonal = list(order = c(0,1,1)))
    
    autoplot(forecast(arima(lh, order = c(1,0,1))))

    fit <- auto.arima(presidents)
    fit %>% forecast(h = 12) %>% autoplot()
    predict(fit, 3)
```


## Modelos ARIMA automáticos 
La función `auto.arima()` da como resultado una formula del tipo siguiente: `ARIMA (p,d,q) (P,D,Q) [m]`, pr ejemplo ARIMA (0,1,4) (0,1,1) [12].

el significado de cada letra puede verse en la ayuda de la función, pero básicamente d es el orden de la primera diferenciación de la serie, D el de la frecuencia estacional
 
La función `auto.arima()` también funciona con datos estacionales, aunque hay que establecer `lambda = 0` (de BoxcCox) en la función para que la transformación de estaciionalidad de deshaga, es decir que el modelo se ajustará a los datos transformados, y que los pronósticos se transformarán de nuevo a la escala original.


Vamos a ver un ejemplo sobre los datos de precios del bitcoin.


```r
    # usamos los datos leidos de BTC
    pBTC1<-ts(na.omit(log(pBTC))) # quitamos NA y hacemos log
    # pintamos la grafica de log del BTC
    pBTC1 %>% autoplot()

    # Ajustamo un modelo ARIMA con lambda = 0
    fit <- auto.arima(pBTC1, lambda=0)

    # Resumen de resultados
    summary(fit)

    # Pintamos el pronostico para 60 días
    fit %>% forecast(h=60) %>% autoplot() #+ coord_cartesian(xlim = c(1800, 2050)) 
```

![arima modelos](imagenes/arima.png)

La función `auto.arima()` necesita estimar muchos modelos diferentes, y se usan varios atajos para tratar de hacer la función lo más rápido posible. Esto puede hacer que se devuelva un modelo que en realidad no tiene el valor de *AICc* más pequeño. Para hacer que `auto.arima()` trabaje más duro para encontrar un buen modelo, agregue el argumento opcional `stepwise = FALSE` para ver una colección de modelos mucho más grande.


```r
# Find an ARIMA model for euretail
fit2 <- auto.arima(pBTC1,stepwise = FALSE)
summary(fit2) 

fit2 %>% forecast(h=60) %>% autoplot()
```

### Comparando auto.arima() y ets()
Vamos a comparar los dos métodos semi automáticos de clclo de pronosticos, el modelo ets() y el auto.arima(). Lo haremos sobre los datos históricos de caudales maximos del río Nilo. Distinguiremos dos conjuntos de datos una de ensayo y otro de test.


```r
str(Nile) # vemos la serie de Nile
# Seleccionamos dos conjuntos de entrenamiento y prueba
train <- window(Nile, start =1871, end = 1960)
test<-window(Nile, start =1960, end = 1970)

# Ajustamos un modelos de arima y de ets a los datos de entrenamiento.
fit1 <- auto.arima(train)
fit2 <- ets(train)

# Vemos los residuos de cada caso. y como son parecidos
checkresiduals(fit1)
checkresiduals(fit2)

# HAcemos el pronostico a 10 años de cada modelo
fc1 <- forecast(fit1, h = 10)
fc2 <- forecast(fit2, h = 10)

# Use accuracy() to find better model based on RMSE
accuracy(fc1, Nile)
accuracy(fc2, Nile)

#Pintamos los resultados
autoplot(fc1,PI=F) + autolayer(fc2,PI=F,col="red") + autolayer(test)
```

# REGRESION DINÁMICA
La regreesión dinámica incorpora al conocimiento de los modelos relaciones de la serie temporal con otras series, ajustando los datos sgún la correlación que exista entre ellas. Se incorpora de esta manera información adicional

La función `auto.arima()` que ajusta un modelo de regresión puede incorporar información adicional de otras series que se aporta como parámetro adicional con el el argumento `xreg` que contiene una matriz de variables de regresión. 

Vamos a ver un caso de ejemplo para tener las cosas más claras. 
La dataframe `datosESP` incluye datos de desempleo, precio de casas y exportaciones en España.


```r
    # Vemos la variable datosESP y la pintamos
    str(datosESP)
    plot.zoo(datosESP, col=c("red", "blue","brown"), lwd=2)
    
    # Ajustamos un modelo ARIMA
    fit <- auto.arima(datosESP[, "preciocasas"], xreg = datosESP[, "desempleo"], stationary = TRUE)
    
    fit <- auto.arima(datosESP$preciocasas, xreg = datosESP$desempleo, stationary = TRUE)

    # Chequeamos el modelo.
    # Este parametro concreto nos da el incremento de precio casa
    #por cada unidad de incremento de desempleo
    coefficients(fit)[3]
```
Ahora que tenemos el modelo, vamos a hacer el pronostico condicionado. Para ello primero calcularemos el pronostico de desempleo según la función lineal `ets()` para 2 años (8 trimestres), y con los datos de esta predicción haremos un pronostico del precio de la vivienda.


```r
    # 1. Ejecutamos un modelo ets() para el desempleo
    fit1<-ets(datosESP[, "desempleo"])
    pronostico_desempleo<-forecast(fit1,h=8)
    autoplot(pronostico_desempleo, main="desempleo ESP")
```

![](Series_temporales_en_R1_files/figure-html/regdinamica1-1.png)<!-- -->

```r
    # 2. Predecimos el precio de la vivienda con el pronóstico anterior de desempleo en cuenta.
    fc <- forecast(fit1, xreg = pronostico_desempleo$mean)
    # Pintamos la predicción condicionada
    autoplot(fc, main="Pronostico reecio vivienda") +
        xlab("trimestres") + ylab("precio vivienda Index")
```

![](Series_temporales_en_R1_files/figure-html/regdinamica1-2.png)<!-- -->

```r
# y si el desempleo se mantien constante en el 20% esos 2 años    
    pronostico_desempleo<-rep(20,8)
    fc <- forecast(fit1, xreg = pronostico_desempleo)
    autoplot(fc, main="Pronostico reecio vivienda") +
        xlab("trimestres") + ylab("precio vivienda Index")
```

![](Series_temporales_en_R1_files/figure-html/regdinamica1-3.png)<!-- -->

```r
# Y si el desempleo baja de forma escalonada hasta el 5% en 2 años    
    fc <- forecast(fit1, xreg = c(18,15,10,8,5,5,5,5))
    autoplot(fc, main="Pronostico reecio vivienda") +
        xlab("trimestres") + ylab("precio vivienda Index")
```

![](Series_temporales_en_R1_files/figure-html/regdinamica1-4.png)<!-- -->

## Pronóstico de la demanda de electricidad
También puede modelar la demanda de electricidad diaria en función de la temperatura. Como puede haber visto en su factura de electricidad, se usa más electricidad en los días calurosos debido al aire acondicionado y en los días fríos debido a la calefacción.

En este ejercicio, se ajustará a un modelo de regresión cuadrática con un error ARMA. Un año de datos diarios se almacenan como elec, incluida la demanda diaria total, una variable indicadora para los días de trabajo (un día de trabajo se representa con 1, y un día no laborable se representa con 0) y las temperaturas máximas diarias. Como hay estacionalidad semanal, la frecuencia se ha establecido en 7.


Vamos a ver otro ejemplo de ajuste dinámico, en los datos de demanda electrica y temperatura. Como es sabido, cuando hace calor se enchufa el aire acondicionado, y cuando hace frio la calefacci´n, por lo que la demanda de energía guarda cierta relación física con la temperatura.

Tenemos un fichero que continen un año de datos diarios de demanda, temperatura y si es o no un dia normal o festivo almacenados en `elec.csv`


```r
    elec<-read.csv("elec.csv",sep = ";")
    # Time plots of demand and temperatures
    head(elec)
    str(elec)
    elec<-ts(elec, start = 1, frequency=7)
    autoplot(elec[, c("Demanda", "Temperatura")], facets = TRUE)
    
    # Matriz de regresión que usamos tem y wday
    xreg <- cbind(MaxTemp = elec[, "Temperatura"], 
                  Workday = elec[, "Wday"])
    
    # Ajustamos el modelo de prediccion de la demanda
    fit <- auto.arima(elec[,"Demanda"], xreg = xreg)
    
    # Realizamos un pronostico para los 5 días siguientes 
    # suponemos que la tem se queda en 30 
    # y que son días normales 3 entre semana y 2 festivos seguidos
    prono5<-forecast(fit, xreg = cbind(rep(30,5),c(0,0,0,1,1)))
    autoplot(prono5, xlim = c(40, 55))
```

# FOURIER 
Cuando tenemos serie cíclicas y estacionales la predicción se hace más compleja y debemos recurrir a la transformada de Fourier. Un video interesante para informarse sobre el tema es este <https://www.youtube.com/watch?v=FjmwwDHT98c>. Las transformadas de Fourier también son utiles en series estacionales dificiles de detectar como en series diarias de datos con estacionalidad mensual, en las que hay muchos datos y la estacionalidad es lejana o de frecuencias . La regresión armónica que usa senos y cosenos puede ser una solución para modelizar estas series.

Antes vamos a ver unas cuestiones técnicas. Llamamos frecuencia a la magnitud temporal del ciclo, y periodo o periodo de retorno (T) al inverso de la frecuencia.

La función `fourier(x,K,h=NULL))` facilita la generación de los armónicos necesarios. Cuanto mayor sea el orden (K), más "ondulado" será el patrón estacional. Con K = 1, es una curva sinusoidal simple. Puede seleccionar el valor de K minimizando el valor del estimador del error **AICc**.

Vamos a ver un ejemplo de predicción de demanda electrica dependinte de la temperatura. Es facilpensar que a mayor temperatura más aires acondicionados y más demanda electrica, y a menor temperatura en invierno tambien por calefacción.

Los datos de demanda están en un fichero csv en formato tabla por año y mes que hay primero que tranasforar.


```r
    elec<-read.csv("elec.csv",sep = ";")
    # Time plots of demand and temperatures
    
    elec<-ts(elec, start = 1, frequency=7)
    
    elec1<-window(elec[,"Demanda"],star=40)
    #spectrum(elec)
    autoplot(elec1)
    # Set up harmonic regressors of order 13
    armonicos <- fourier(elec1)
    
    # Fit regression model with ARIMA errors
    fit <- auto.arima(elec1, xreg = armonicos, seasonal = FALSE)
    fc <- forecast(fit, xreg = armonicos, h=7*2)
    
    # Plot forecasts fc
    autoplot(fc)
```


## Regresión armónica para la estacionalidad múltiple
Las regresiones armónicas también son útiles cuando las series temporales tienen múltiples patrones estacionales. La función `auto.arima()` toma mucho tiempo para ajustarse a series largas, por lo que en su lugar es preferible usar un modelo de regresión estándar con términos de Fourier utilizando la función `tslm()`. Esto es muy similar a `lm()`, pero está diseñado para manejar series temporales.

Con la estacionalidad múltiple, debe especificar el orden K para cada uno de los períodos estacionales.


```r
    # Veamos los argumentos de la funcion:
    args(tslm)
```

```
## function (formula, data, subset, lambda = NULL, biasadj = FALSE, 
##     ...) 
## NULL
```


```r
# Fit a harmonic regression using order 10 for each type of seasonality , 
# K es un vector pues hay doble ciclo estacinal diario y semanal
fit <- tslm(taylor ~ fourier(taylor, K = c(10, 10)))

# Forecast 20 working days ahead
fc <- forecast(fit, newdata = data.frame(fourier(taylor, K = c(10, 10), h = 960)))

# Plot the forecasts
autoplot(fc)

# Check the residuals of fit
checkresiduals(fit)
```

Veamos otro ejemplo de serie de fourier. Por desgracia en la versión de R con la que he realizado el manual no funciona correctamente la función fourier, por lo que tan solo xpongo el código sin ejecutarlo.


```r
    # Plot the calls data
    autoplot(lynx)
    
    # Set up the xreg matrix
    xreg <- fourier(Nile, 2)
    c(4,4))
    
    McSpatial::fourier(Nile,minq=1,maxq=10,crit="gcv")
    fit_g<-McSpatial::fourier(y~x,minq=1,maxq=qmax-1,crit="gcv")
    fit_g$q
    # Fit a dynamic regression model
    fit <- auto.arima(calls, xreg = xreg, seasonal = FALSE, stationary = TRUE)
    
    # Check the residuals
    checkresiduals(fit)
    
    # Plot forecasts for 10 working days ahead
    fc <- forecast(fit, xreg =  fourier(calls, c(10, 0), h = 1690))
    autoplot(fc)
```

## Modelo TBATS

Otro de los modelos de predicción del paquete `forecast` es el tbats (*Exponential smoothing state space model with Box-Cox transformation, ARMA errors, Trend and Seasonal component*). Se trata de un tipo especial de modelo de predicción de series temporales que puede ser muy lento de estimar, especialmente con varias series temporales estacionales, pero muy completo, esto es que es apropiado para series complejas, en las que hay cambios de varianza y tendencia apreciables.

### Ejemplo Gas
Tenemos un fichero csv, con los datos de producción de gas mensual en Australia.En este ejemplo priemro leeremos el fichero y lo transformaremos en uno limpio sobre el que podamos leer datos, ya que el original contienen datos en tablas por meses y necesitamos con estos datos hacer una serie temporal. Una vez transformado pintaremos la gráfica y vemos que la varianza de la serie ha cambiado mucho con el tiempo, por lo que necesita una transformación. La estacionalidad también ha cambiado de forma con el tiempo, y hay una fuerte tendencia. Esto lo convierte en una serie ideal para probar la función `tbats()` que está diseñada para manejar estas características.


```r
    # LEEMOS Y LIMPIAMOS LOS DATOS
    # Leemos los datos del fichero
    gas1<-read.csv("gas.csv")
    str(gas1)
    head(gas1)
    #names(gas1)[1]<-"ano"
    names(gas1)<-c("ano","ene","feb","mar","abr", "may", "jun", "jul", "ago", "sep", "oct", "nov", "dic")
    # Vamos a juntar los datos en uan tbla utilizable
    require(tidyr)
    # con el siguiente script limpiamos la tabla y la hacemos dfecha datos
    df.gas <- melt(gas1, id=c("ano"),na.rm = T)
    #df.gas <- gather(gas1,key=ano,value=2:13,na.rm=T) 
    str(df.gas)
    names(df.gas)<-c("ano","mes","valor")
    head(df.gas)
    
    df.gas$fecha<-paste0("1", df.gas$mes,df.gas$ano)
    df.gas$fecha<-as.Date(df.gas$fecha,"%d%b%Y")

    # HACEMOS EL MODELO DE PRONOSTICO
    require(xts)
    require(ggplot2)
    ts.gas<-xts(df.gas$valor, order.by = df.gas$fecha)
    
    # Pintamos los datos
    autoplot(ts.gas)
    # Ajustamos el modelo TBATS a los datos del gas
    # ojo que debemos transformarlo en ts antes
    fit <- tbats(as.ts(ts.gas))
    # Realizamos el pronóstico a 5 años
    fc <- forecast(fit, h=5*12)
    # Pintamos el resultados
    autoplot(fc)
```

# Enlaces

Ponemos varios enlaces interesantes al respecto de la series temporales en R.

 * <https://robjhyndman.com/hyndsight/tspackages/>
 * <https://www.stat.pitt.edu/stoffer/tsa4/R_toot.htm>
 * <https://otexts.org/fpp2/appendix-using-r.html>
 * <https://a-little-book-of-r-for-time-series.readthedocs.io/en/latest/src/timeseries.html>
