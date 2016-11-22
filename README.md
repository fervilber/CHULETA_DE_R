---
title: "r_sort_code"
author: "Fernando Villalba"
date: "16 de noviembre de 2016"
output:
  html_document:
    toc: yes
    toc_depth: '3'
    number_sections: true
    theme: united
    highlight: tango
---

```{r setup, include=FALSE, eval=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
## Table of contents
1. [INTRODUCCION](#INTRODUCCION)
2. [ENTORNO DE TRABAJO](#ENTORNO_DE_TRABAJO)
3. [DATA FRAMES](#df)
4. [DATA TABLES](#dt)
5. [LECTURA DE FICHEROS](#lectura_de_ficheros)
6. [WEB SCRAPING](#wscraping)
7. [CONSULTA BASES DE DATOS](#bd)
8. [EXRPRESIONES LÓGICAS](#logexp)
9. [FUNCIONES DE TEXTO](#text)
10. [INSTALACION DE PAQUETES](#inst_paq)


## INTRODUCCION <a name="INTRODUCCION"></a>
Some pieces of code that help begginers to write their first programs in R.
Some of the comments are in spanish.
<http://rmarkdown.rstudio.com/html_document_format.html>

## ENTORNO DE TRABAJO <a name="ENTORNO_DE_TRABAJO"></a>

```{r eval=FALSE}
# Borrar todas las varibles del entorno en R
rm(list=ls())

```

## DATA FRAMES <a name="df"></a>
Seleccionar correctamente o filtrar un data frame es una de las tareas mas necesarias de R.
### Descartar NA <a name="df1"></a>
para seleccionar los valores no NA podemos usar esto

```{r eval=FALSE}
#data es un df
is.na(data) # es true para todos los valores NA
!is.na(data) # es true para todos los valores no NA

# Por ejemplo convertir los ceros 0 a NA
df[df == 0]<-NA

#nombrecol es el nombre de la columna o num de columna
#entonces filtro almacena los valores que no son NA
    filtro <- data[ !is.na(data[nombrecol]), ]
    colsinNA <- data[, nombrecol]
    
#si son valoresnumericos podemos saber el max min etc de este modo
    rowNum <- which.min(colsinNA) ##return the nrow of the min value 
    nombre <- data[rowNum, ]$name
   
   
    rowNum <- which.max(colsinNA) ##return the nrow of the max value   

```
### ordenar un DF <a name="df2"></a>
para ordenar según criterios columan 1 y columna 3 hacer esto:

```{r eval=FALSE}
#data es un df
# esto ordena el df con dos criterios, primero según los valores de la col 1 y despues caso de empate por col 3
  data_ordenado <- data[order(data[1],data[3],decreasing = FALSE),]
```
### Seleccion de valores
```{r eval=FALSE}
 ## selec char values per col
    data <- data[data[outcome] != 'Not Available', ]
```
### Convertir a numeric
```{r eval=FALSE}
## Convierte a numeric la columna 3 del df
    data[3] <- as.data.frame(sapply(data[3], as.numeric))
```
## DATA TABLES <a name="dt"></a>
### comandos de Data Table
```{r eval=FALSE}
# leer un dt con fread
#1. cargar la librería de data table
library(data.table)
DT <- fread(input="./data/e5.csv", sep=",")
file.info("./data/e5.csv")$size
#calcular media de una columna y split de otra
DT[,mean(colA),by=colB])
# que es lo mismo que:
sapply(split(DT$colA,DT$colB),mean)
# lomismo con split
subset1<-split(DT$colA,DT$colB)
#esto lo dibide en dos, y podemos sacar la media de cada uno
summary(subset1[[1]])

#Guardar datos como un fichero
DT<-write.table(big_df, file=file, row.names=FALSE, col.names=TRUE, sep="\t", quote=FALSE)
system.time(fread(file))



```



## LECTURA DE FICHEROS <a name="lectura_de_ficheros"></a>
### Leer los ficheros de un directorio <a name="l1"></a>
Como leer los nombres de los ficheros de un dir
```{r eval=FALSE}
setwd("C:/....") #establece el directorio de trabajo
#almacena en files_full la ruta relativa de cada fichero, 
# si solo queremos el nombre poner full.names=FALSE
files_full<-list.files(pattern = '.jpg',full.names=TRUE) #lee todos los ficheros en el dir

```
### leer los datos exif de fotos <a name="Leer_exif"></a>

```{r eval=FALSE}
#instalamos el paquete que lee los datos exif
install.packages("exifr")
#cargamo la librería
library("exifr", lib.loc="~/R/win-library/3.3")

setwd("C:/TITAN/FOTOS") #establece el directorio de trabajo donde están las fotos
files_full<-list.files(pattern = '.jpg',full.names=TRUE) #lee todos los ficheros en el dir
#llamamos a exifr que es la funcion del paquete que lee las fotos y extrae un data frame con todos los 
#datos EXIF
exifinfo <- exifr(files_full)
```

### leer fichero *.csv y descartar los NA <a name="Leer_sin_NA"></a>

Una forma rápida es descartar los casos no completos, es decir las filas que contengan algún NA. Ojo pues nos quita muchos casos con datos si falta en una de las columnas, por lo que no es conveniente si queremos hacer estadisticas de columnas concretas más tarde
```{r eval=FALSE}
#lee un fichero de datos
data <-read.csv(nombrefichero)

#quitamos los NA de la columna 4
    data1 <- data[!is.na(data[4]), ]

#alternativamente si quiero solo casos completos:
# las filas que tienen datos validos en todas las columnas 
    data1<-na.omit(data)
    # si quiero sabe cuantas observaciones tengo
    nobs<- nrow(na.omit(data))
```
Otra forma es leer todo, y luego en el proceso seleccionar las columnas concretas descartando NA. PAra ello usaremos, esto:
Tambien mostramos como leer un fichero que contiene NA pero con otro formato. En este caso el fichero contiene datos como "Not Available" que al leer le decimos que equivale a NA, lo que nos facilita despues el tratamiento
```{r eval=FALSE}
data <- read.csv("data/outcome-of-care-measures.csv", stringsAsFactors=FALSE, na.strings="Not Available")
# seleccionamos de la columna 3 los datos validos , no NA
data1 <- data[!is.na(data[3]), ]
```

```{r eval=FALSE}
#creamos un vector
x<-1:100
#seq_along(x) crea una secuencia de 1 a el lenght(x)..
    for (i in seq_along(x)) {   
        x2 <- x[i]*2
    }
```
## WEB SCRAPING <a name="wscraping"></a>
### Descargar una tabla de una pagina web  <a name="tabla_web"/>
 descargar una tabla de una web
```{r eval=FALSE}
#EJEMPLO 1
# descargamos los datos de invertia del IBEX35
library("XML", lib.loc="~/R/win-library/3.3")
library(XML)

#direccion url con al menos una tabla
t_invertia ="http://www.invertia.com/mercados/bolsa/indices/ibex-35/acciones-ib011ibex35"

# con "which" indicamos qué tabla de la web descargamos, en este caso la 2 que corresponde con los valores del IBEX
t_invertia.table = readHTMLTable(t_invertia, header=T, which=2,trim = TRUE,stringsAsFactors=F)
t_invertia.table
names(t_invertia.table)
nom<-c("TKR","último", "Dif","Dif. %", "Open","Max","Min", "Vol","Cap", "Rt/Div",  "BPA", "PER","Hora")
names(t_invertia.table)<-nom
#quitar los n.a y n.d de la tabla y poner como NA
t_invertia.table[t_invertia.table == "n.d."]<-NA
t_invertia.table[t_invertia.table == "n.a."]<-NA
## Bucles <a name="bucles"></a>
### Bucle que recorra un vector <a name="b1"></a>
```
### Descargar un fichero csv desde la web
```{r eval=FALSE}
# Descargar de la web el fichero siguiente csv
fileUrl<-"https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Fss06hid.csv"
# lo guardamos en la carpeta local como p1.csv
download.file(fileUrl, destfile="p1.csv")

# leemos su contenido
p1 <- read.table("p1.csv", sep="," , header= TRUE)
# Leemos la variable VAL que continene el valor de las casas
tmp<-p1$VAL
tmp<-tmp[!is.na(tmp)] # seleccionamos los validos quitamos NA

# leemos enel code book que un valor de 24 significa que el
# valor  de la casa es de 1 millon o más.
tmp<-tmp[tmp>=24]
# La respuesta de cuantos hay de mas de un millon es 53
pregunata1<- length(tmp)
```
### Descargar web y leer lineas
```{r eval=FALSE}
con <- url("http://biostat.jhsph.edu/~jleek/contact.html")
htmlcode <- readLines(con)
close(con) # SIEMPRE CERRAR CONEXION

nchar(htmlcode[10])
nchar(htmlcode[20])
nchar(htmlcode[30])
nchar(htmlcode[100])
```


## CONSULTA BASES DE DATOS <a name="bd"></a>
### Conectar con MySQL
```{r eval=FALSE}
# open a connection to the database
ucscDb<-dbConnect(MySQL(),user = "genome",host = "genome-mysql.cse.ucsc.edu")
db = "hg19" #= select specific database
result<- dbGetQuery(ucscDb,"show databases;")
dbDisconnect(ucscDb)
```
### Conectar a fichero y consultar con SQL

```{r eval=FALSE}
# install sqldf package using below command
# install.packages("sqldf")
# Load sqldf package
library(sqldf)

fileurl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Fss06pid.csv"
download.file(fileurl, destfile="./US_community_survey_data.csv")
acs <- read.csv("./US_community_survey_data1.csv", header=TRUE)

sqldf("select distinct AGEP from acs")
```


## EXRPRESIONES LÓGICAS <a name="logexp"></a>
Muestra expresiones logicas que pueden ser muyy utiles.
### Validar valores
outcome=resultado en inglés
Ejemplo de una funcion en la que se validan los valores de entrada
```{r eval=FALSE}
best <- function(state, outcome) {
    ## Error handling
    ## Validate the outcome string. Valid values:
    possibleoutcomes = c("heart attack", "heart failure", "pneumonia")
    if( outcome %in% possibleoutcomes == FALSE ) stop("invalid outcome")
    
    ## Validate the state string
    states <- data[, 2]
    states <- unique(states) ##use unique to extract just one value without repeat
    if( state %in% states == FALSE ) stop("invalid state")    
    
    ## Validate the num value 
    ## &&=AND  num%%1
    if( num != "best" && num != "worst" && num%%1 != 0 ) stop("invalid num")

}

```
! indicates logical negation (NOT).

& and && indicate logical AND and | and || indicate logical OR. The shorter form performs elementwise comparisons in much the same way as arithmetic operators. The longer form evaluates left to right examining only the first element of each vector. Evaluation proceeds only until the result is determined. The longer form is appropriate for programming control-flow and typically preferred in if clauses.

xor indicates elementwise exclusive OR.

isTRUE(x) is an abbreviation of identical(TRUE, x), and so is true if and only if x is a length-one logical vector with no attributes (not even names).

## FUNCIONES DE TEXTO <a name="text"></a>
### Dar la vuelta a un string
```{r eval=FALSE}
#cortar los espacios del inicio y fin
trim <- function (x) gsub("^\\s+|\\s+$", "", x)

#dar la vuelta a una palabra
strReverse <- function(x){
  sapply(lapply(strsplit(x, NULL), rev), paste, collapse = "")
}

#substituir caracteres de un texto
descripcion<-"sajsasjj jlakjs lajs lajs \n asa"
descripcion <- gsub("\n"," ",descripcion, fixed = TRUE)

```
## INSTALACION DE PAQUETES <a name="inst_paq"></a>
### Instalar bioconductor
```{r eval=FALSE}
source("https://bioconductor.org/biocLite.R")
biocLite()
```
