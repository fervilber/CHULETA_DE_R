---
title: "VILBER R CHEAT SHEET"
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
## INDICE
1. [INTRODUCCION](#INTRODUCCION)
2. [ENTORNO DE TRABAJO](#ENTORNO_DE_TRABAJO)
3. [DATA FRAMES](#df)
4. [DATA TABLES](#dt)
5. [LECTURA DE FICHEROS](#lectura_de_ficheros)
6. [WEB SCRAPING](#wscraping)
7. [CONSULTA BASES DE DATOS](#bd)
8. [EXRPRESIONES LÓGICAS](#logexp)
9. [FUNCIONES DE TEXTO](#text)
10.[FUNCIONES DE FECHA](#fecha)
11. [INSTALACION DE PAQUETES](#inst_paq)
12. [SUBSETTING](#subset)
13. [GENERA SERIES ALEATORIAS](#seralea)

## INTRODUCCION <a name="INTRODUCCION"></a>
Some pieces of code that help begginers to write their first programs in R.
Some of the comments are in spanish.
<http://rmarkdown.rstudio.com/html_document_format.html>

## ENTORNO DE TRABAJO <a name="ENTORNO_DE_TRABAJO"></a>

```{r eval=FALSE}
getwd() # nos dice el directorio de trabajo actual
setwd("c:/R/PROYECTOS") #establece la ruta dada como directorio de trabajo

ls() #lista los objetos actuales en memoria
list.files() #lista los ficheros en el directorio actual
dir() # lo mismo que list.files


# CREAR UN DIRECTORIO
dir.create("testdir") # crea un directorio en el wd actual con el nombre testdir

file.exists("mytest.R") # si TRUE es el que fichero existe
file.rename("mytest.R", "mytest2.R")# cambia el nombre de  a 2
file.copy("mytest2.R", "mytest3.R")
dir.create(file.path("testdir2","testdir3"),recursive = TRUE) # CREA dos directorios recursivamente

# BORRAR DIRECTORIOS CUIDADIN!!
unlink("testdir",recursive = TRUE) #BORRA LOS DIRECTORIOS

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
### Quitar las columnas que son todo ceros o NA

```{r eval=FALSE}
# Quita las columnas que son todo NA
DT[,which(unlist(lapply(DT, function(x)!all(is.na(x))))),with=F]

#Estas otras opciones para quitar los ceros
df[,colSums(is.na(df))< nrow(df)])
select(df,df[,colSums(is.na(df))< nrow(df)])
df<-df[,colSums(df != 0) != 0]
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
### Espacio que ocuparía una matriz
Para saber el espacio de memoria que ocupará una matriz podemos usar esta función.

```{r estima_mem}
# Función que estima la cantidad de memoria usada por una matriz de filas x col
# según sea integer o numeric

predict_data_size <- function(numeric_size, number_type = "numeric") {
  if(number_type == "integer") {
    byte_per_number = 4 # 4 bytes por numero entero
  } else if(number_type == "numeric") {
    byte_per_number = 8  # 8 bytes por numero
  } else {
    stop(sprintf("Unknown number_type: %s", number_type))
  }
  estimate_size_in_bytes = (numeric_size * byte_per_number)
  class(estimate_size_in_bytes) = "object_size"
  print(estimate_size_in_bytes, units = "auto")
}
# Ejemplos
predict_data_size(1304287*28, "numeric")
predict_data_size(1518*1518, "integer")
```

## LECTURA DE FICHEROS <a name="lectura_de_ficheros"></a>
El comando básico para leer un fichero es el siguiente:
```{r eval=FALSE}
datos_leidos <- read.table("nombre_fichero.txt", 
                skip = numero_de_fila_desde_donde_empieza_a_leer,
                nrows = num_filas_que leer,
                sep = ";", na.strings = "?",
                col.names= vector_nombres_de_col )
```
### Leer un txt
Leer un fichero txtx de texto y asignar los nombres de las columnas según la primera fila
El fichero de muestra contiene comentarios en las filas que empiezan con # y los datos están separados por | .
```{r eval=FALSE}
pm1<- read.table("nombre_fichero.txt", comment.char = "#", header = FALSE, sep = "|", na.strings = "")
# vemos la dimension del fichero
dim(pm0)
head(pm0)
# vamos a asignar los nombres de las columnas, que vemos que estaban en la primera fila del fichero
cnames<- readLines("nombre_fichero.txt",1) # lee la fila 1
# vemos que los nombres están separados por |, hacemos un 
# split de los datos
cnames<-strsplit(cnames, "|", fixed = TRUE)
# asignamos como nombres de col el primer elemento de la lista de cnames
names(pm0)<-cnames[[1]]
```

### Leer los nombres de los ficheros de un directorio <a name="l1"></a>
Como leer los nombres de los ficheros de un directorio:
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
### Limpiar espacios en nombres de ficheros
El objeto es quitar los espacios y caracteres raros en los nombres de ficheros de un directorio.
En el ejemplo solo seleccionamos los ficheros de extension *.jpg
```{r eval=FALSE}
getwd()
ruta<-"C:/R/proyectos/INFORMES/OJOS"
setwd(ruta)
#Cambiar espacio por _ en los nombres de ficheros de un directorio
imagenes<-list.files("./imag/",pattern = ".jpg")
imagenes
length(imagenes)
limpia<- function (x) gsub('([[:punct:]])|\\s+','_',x); # quita los simbolos raros
limpian<- function (x) gsub('ñ','n',x); # quita las ñ por n

setwd("C:/R/proyectos/INFORMES/OJOS/imag")
## Código para cambiar el nombre de los ficheros de un directorio
## eliminando espacios en blanco

for (i in seq_along(imagenes)){
    a<-limpia(substring(imagenes[i],1,nchar(imagenes[i])-4))
    a<-limpian(a)
    a<- paste(a,".jpg", sep = "")
    file.rename(imagenes[i],a)# cambia el nombre de  a 2
}
```

### ESCRIBIR EN UN FICHERO de MARKDOWN CON FIGURAS
 Escribe en un fichero desde R.
 Este programa lee los nombres de los ficheros de imagen de un directorio, y escribe en otro fichero el código markdown para insertar dichas imagenes en un informe.

```{r}
################################################################
## crea código LATEX para insertar las imagenes de un directorio
################################################################

ruta<-"C:/R/proyectos/INFORMES/OJOS"
setwd(ruta)
getwd()
imagenes<-list.files("./imag/",pattern = ".jpg")

sink("imagenes.rmd") #abro la conexion con el fichero
cat("# cod para insertar las imagenes de un directoio en markdown")
cat("\n") #salto de linea
## Empieza la figura

# hago un loop para todas las subfiguras
for (i in seq_along(imagenes)){
    #i<-1
    nom<- substring(imagenes[i],1,nchar(imagenes[i])-4)
    nom
    line<-paste("![", nom," \\label{fig_", i , "}](imag/", imagenes[i] ,")", sep="")
    line
    cat(line)
    cat("\n")
}

sink()# cierro la conexion
```
## escribir fichero de cod latex
Lo mismo pero en codigo Latex
```{r}
################################################################
## crea código LATEX para insertar las imagenes de un directorio
## en un informe R markdown.
################################################################

ruta<-"C:/R/proyectos/SCADA/ulea"
setwd(ruta)
imagenes<-list.files("./imag/" )
length(imagenes)
## writeLines(line,"figurasLatex.tex", sep = "\n", useBytes = FALSE)

limpia<- function (x) gsub('([[:punct:]])|\\s+','_',x);
limpian<- function (x) gsub('ñ','n',x); # cambia las ñ por n en los nombres
#Escribir en fichero con CAT
fecha<-Sys.Date()
sink("figurasLatex.tex") #abro la conexion con el fichero
cat("% NUEVO ANEXO FOTOGRAFICO")
cat("\n")
cat(paste("% fer -",fecha,sep=""))
cat("\n") #salto de linea
## Empieza la figura
line<-"\\begin{figure}[h] \\centering"
cat(line)
cat("\n")
# hago un loop para todas las subfiguras
for (i in seq_along(imagenes)){
    line<-""
    line<-paste(line,"\\subfigure[",imagenes[i],"]{\\includegraphics[width=0.45\\linewidth]{imag/", imagenes[i],"}}", sep="")
    #write(line,file="figurasLatex.tex",append=TRUE)
    cat(line)
    cat("\n")
    if(i%%2==0){
        cat("\\\\")
        cat("\n")
    }
}

cat("\\caption{figuras}\\label{fig.fig1}")
cat("\\end{figure}")
sink()# cierro la conexion
```

## WEB SCRAPING <a name="wscraping"></a>
### Descargar y descomprimir un zip de la web
# Download data files
```{r eval=FALSE}
library(httr) 
url <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
file <- "datos.zip"
if(!file.exists(file)){
    print("descargando")
    download.file(url, file, method = "wininet")
}

# Unzip and create folders 
datafolder <- "UCI_HAR_Dataset"
resultsfolder <- "results"
if(!file.exists(datafolder)){
    print("unzip file")
    unzip(file, list = FALSE, overwrite = TRUE)
} 

if(!file.exists(resultsfolder)){
    print("create results folder")
    dir.create(resultsfolder)
} 

# 0) Read txt and convert to data.frame in folder UCI HAR Dataset

setwd("C:/R/proyectos/C3_course_project/UCI HAR Dataset/")

# Read general data
features     = read.table('features.txt',header=FALSE); #imports features.txt
activityType = read.table('activity_labels.txt',header=FALSE); #imports activity_labels.txt
```
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

# Buscar un patron de texto
#da como resultado el id del elemento del vector que lo contiene
txt <- c("arm","foot","lefroo", "baFoobar")
grep("foo", txt,ignore.case=TRUE)
# txt[2] y txt[4] lo contienen, el resultado es un vector (2, 4)

grep("[a-z]", letters)
# obtiene un vector con las id de las letras de 1 a 26
```
### split texto
para separar en elementos de un vector una linea e texto con separadores
```{r eval=FALSE}
cnames<-"col1 | col2 | nombre tres| asa"
cnames<-strsplit(cnames, "|", fixed = TRUE)
# como algunos nombres tienen espacios lo arreglamos con la funcion make.names
# que cambia espacios por puntos
cnames<- make.names(cnames[[1]])
```
## FUNCIONES DE FECHA <a name="fecha"></a>
### convertir datos de fecha y hora en POSIXlt
Muchas veces al leer un fichero nos dan los datos de fecha y hora separados en columnas, una manera de transformarlos en formato POSIXlt es la siguiente:
Lod datos originales del fichero son:
|Date|Time|
|---|---|
|16/12/2006|17:24:00|
|16/12/2006|17:25:00|
|16/12/2006|17:26:00|

```{r}
# crear una nueva variable class POSIXlt
data$DateTime <- strptime(paste(data$Date, data$Time, sep=" "), "%d/%m/%Y %H:%M:%S")
```
### Paquete stringr
paquete con funciones de manejo de textos
```{r eval=FALSE}
library(stringr)

#Quitar espacios delante y detras de un string
str_trim(string) # si se especifica side=c("left")
```

## INSTALACION DE PAQUETES <a name="inst_paq"></a>
Para instalar un paquete hacemos:
  install.packages("nombredelpaquete");
Si no sabemos si está instalado, lo mejor es usar la siguiente funcion con el paquete que queramos, en este caso "png":
De este modo nos instalará incluso las dependencias al mismo.
```{r}  
if (!require("png")) {
   install.packages("png", dependencies = TRUE)
   library(png)
   } 
 ```
### RTOOLS
Para instalar RTOOL debemos bajar el fichero de la siguiente web:
<http://cran.r-project.org/bin/windows/Rtools/>

```{r eval=FALSE}
# 1. comprobamos que no está
find.package("devtools")

# 2. si está lo cargamos así 
library(devtools)
find_rtools()
```
### Instalar bioconductor
```{r eval=FALSE}
source("https://bioconductor.org/biocLite.R")
biocLite()

#Paquetes disponibles 
#a<-- available.packages()
   head(rownames(a),3) ## muestra solo los nombres de las primeras lineas

# Para saber los argumentos de una funcion:
args(list.files) # da la lista de argumentos de una fucnion
args(grep)

```
### 

## SUBSETTING <a name="subset"></a>

Repaso las principales formas de crear subconjuntos dentro de un data frame o de un data table.

```{r eval=FALSE}
#########################################
##  SUBDIVIDISION DE  VARIABLES
##  CREACION DE SUBCONJUNTOS
#########################################

#Creamos un data frame
set.seed(13435)
x<- data.frame("var1"= sample(1:5),"var2"= sample(6:10),"var3"= sample(11:15))
x<-x[sample(1:5),];x$var2[c(1,3)]=NA
x

# Seleccionar una columna por numero
x[,1]
# Seleccionar columna por name
x[,"var1"]

# Seleccionar parte de una fila y columna a la vez
x[1:2,"var2"]

# Seleccionar con operadores lógicos
# selecciono los valores que cumplen esta condición en todas las columnas
x[(x$var1<=3 & x$var3 >11),] # AND 

x[(x$var1<=3 | x$var3>15),] # OR


# Seleccionar con which
x[which(x$var2>8),]
x[(x$var2>8),]  # ver diferencia respecto a NA

# subconjunto de una columna que ademas contenga ciertos valores
x2<-x[x$var3 %in% c(4,56,23)] # crea nueva tabla con la columna var3
                              # pero solo las filas que valen 4, 56 y 23

#########################################
##  ORDENAR VARIABLES
#########################################

# Ordenar una columna ascendiente por defecto
sort(x$var1)

# ordenar por orden decreciente
sort(x$var1, decreasion=TRUE)

# ordenar y poner los NA al final
sort(x$var1, na.last=TRUE)

# Ordenar toda el data frame según una columna
x[order(x$var1),] # ordeno por la columna var1


#Ordenar por varias variables
x[order(x$var1,x$var3),] # ordeno por la columna var1

#####################################
# Añadir una columna nueva al data frame directamente
x$var4<- rnorm(5)

#Añadir columnas nuevas
y<- cbind(x,rnorm(5)) # la añade en la ultima columna
y<- cbind(rnorm(5),x) # añade una columna en la primera columna

# Añade una fila nueva 
y<-rbind(c(1,2,3,4,5,6),x)

###########################
# SUBTOTALES CON TAPPLY
#########################
  # por ejemplo si queremos la suma de la colA, según cada valor de la colB de un data frame
  tapply(df$colA,df$colB,sum)
  #probar dcast(df, df$colA ~ df$colB, sum)
  #Otra opcion es con split
  res<- lapply(split(df$colA,df$colB),sum)
  unlist(res)
  sapply(res, sum)

#####################################
# sumar elementos de un split doble
# lista de listas
# hacemos un doble split
df_split<-split(BaltCity,list(BaltCity$year,BaltCity$type))

# rutina para obtener la suma de un elemento de segundo nivel en una lista de listas.
# cogemos la suma de cada elemento 1 de cada lista principal del split
resum_df<- data.frame()

for (i in seq(df_split)){
    # cambia 1 por la columna de la sublista que queramos resumir
    resum_df<-rbind(resum_df,sum(df_split[[i]][,1]));#,names(df_split)[i]); 
}
# añadimos el nombre
resum_df<-cbind(resum_df,names(df_split))
resum_df

# crear tabla relacional split doble con xtabs
# here are 3 ways of calculate the cros data table
# 1. xtabs
    xt<-xtabs(Emissions~year + type,BaltCity) # emision year y taype son columnas del data frame BaltCity
    # to test the results take a look at some
    # sum(BaltCity[BaltCity$year == 1999 & BaltCity$type == "POINT",1])
# 2. aggregatee
     xta<-aggregate(Emissions ~ year + type, BaltCity, sum)
# 3. library(dplyr)
     xta <- nei %>%
        filter(fips == 24510) %>%
        group_by(year, type) %>%
        summarize(total.emissions = sum(Emissions))

```
## GENERA SERIES ALEATORIAS <a name="seralea"></a>
En muchas ocasiones queremos generar numeros aleatorios y series para modeliza:
La semilla reprodicible es con seed

```{r}
set.seed(10) # genera semilla aleatoria
x <- rnorm(100) # genera 100 numeros normales entre 0-1

# creamos un factor
f <- rep(0:1,each=50) # repite 0 50 veces y 1 otras 50
#para convertir en factor
f<-factor(f,labels=c("Grupo 1", "Grupo 2"))

#crear variable aleatoria de yes y nos
yesno<- sample(c("yes", "no"), size=10, replace=TRUE)
#crear factor
yesnofac<-factor(yesno, levels=c("yes", "no"))
as.numeric((yesnofac))
#yes=1, no 2...

# si lo quiero cambiar el orden de los factores
yesnofac<-relevel(yesnofac, ref="no")
as.numeric((yesnofac))
 

```
### MONEDA AL AIRE
para simular el lanzamiento de una moneda usaremos la binomial.

```{r}
set.seed(1)
n<-1000 # numero de lanzamientos de moneda o tirada
# size es el numero de monedas que lanzamos en la misma tirada, y lo que da es la suma de los resultados. p ejm size 5 =3 caras + 2 cruces...
lanzamientos=rbinom(n,size=1,prob = 0.5)
table(lanzamientos)
```

### FACTORES
Dividir una variable numerica en 3 partes como factores o categorias
```{r}
# dividir en 3 partes iguales una variable numerica diamonds$carat
# calculamos los puntos de corte 3+1=4
cutpoints<-quantile(diamonds$carat,seq(0,1,length=4),na.rm=TRUE)

#creamos la nueva variable en la dataframe diamons
# cut(variable_a_cortar, puntos_de_corte)
diamonds$car2<-cut(diamonds$carat,cutpoints)
#ojo que los valores iguales al min los excluye en otra categoría
levels(diamonds$car2).

```
