# R_SORT_CODE
learning R by doing.

Sort code examples is a bunch of sort examplos of code that will be veru usefull for beginners.


# Leer los ficheros de un directorio
Como leer los nombres de los ficheros de un dir
```{r}
setwd("C:/....") #establece el directorio de trabajo
#almacena en files_full la ruta relativa de cada fichero, 
# si solo queremos el nombre poner full.names=FALSE
files_full<-list.files(pattern = '.jpg',full.names=TRUE) #lee todos los ficheros en el dir

```

# leer los datos exif de fotos

```{r}
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

#leer fichero *.csv y descartar los NA
```{r}
#lee un fichero de datos
tmp <-read.csv(nombrefichero)

#numero de filas totales que es igual al num de casos completos
# con datos completos en todas las columnas de la fila
 nobs<- nrow(na.omit(tmp))
```
# Bucles
## Bucle que recorra un vector
```{r}

#creamos un vector
x<-1:100
#seq_along(x) crea una secuencia de 1 a el lenght(x)..
    for (i in seq_along(x)) {   
        x2 <- x[i]*2
    }
```


