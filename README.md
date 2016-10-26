# R_SORT_CODE
learning R by doing.

Sort code examples is a bunch of sort examplos of code that will be veru usefull for beginners.


# Leer los ficheros de un directorio


# leer los datos exif de fotos

```{r}

install.packages("exifr")

setwd("C:/TITAN/FOTOS")
files_full<-list.files(pattern = '.jpg',full.names=TRUE)
exifinfo <- exifr(files_full)
```

# leer los datos exif de fotos
