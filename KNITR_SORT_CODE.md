

## INDICE
1. [INTRODUCCION](#INTRODUCCION)
2. [INSERTAR IMAGENES](#INSERT_IMAG)
3. [CREAR FICHERO DE IMAGENES](#FICHERO)

## INTRODUCCION <a name="INTRODUCCION"></a>
Para generar informes he empezado a usar R junto con el paquete knitr. Esto facilita el trabajo y la salida es realmente buena.
Luego lo suelo imprimir en documentos pdf como resultado final.

Knitr transforma todo a LaTeX y luego genera el documento pdf.
El problema de esto, y empiezo claro, es que lapersonalizacion es un poco compleja, y cuesta pillar el ptruco de las cosas, po eso voy a ir apuntando aquí todas las cosillas que estoy mirando para tenerlas a mano como en una especie de hoja rápida o manual de usuario breve.

## Configuracion general
en las primeras lineas de los documentos en KnitR se especifican muchas de las caracteristicas generales de los mismos. Hay varias formas de especificarlas, bien po código KnitR o también se puede hacer mediante código Latex.

Ya sabemos que LateX permite hacer de todo, pero en este caso existen muchos problemas de compatibilidad con KnitR, que hacen dificil algunas cuestiones.

### Ejemplos
Iniciamos explicando el preambulo del documento, que configura las principales caractristicas del mismo.
```{}
---
title: "Cómo insertar imágenes en Markdown  \\  Guia básica"
author: "FVB"
date: "2017"
lang: "es"
output:
  pdf_document:
    fig_caption: yes
    fig_crop: no
    fig_height: 2
    fig_width: 3
    highlight: haddock
    keep_tex: yes
    number_sections: yes
    toc: yes
    toc_depth: 2
  html_document:
    fig_caption: yes
    theme: journal
    toc: yes
    toc_depth: 2
header-includes: 
- \usepackage{graphicx}
- \usepackage{subfig}
- \usepackage[labelfont=bf]{caption}
- \usepackage{fancyheadings}
- \usepackage[utf8]{inputenc}
- \pagestyle{fancy}
- \rhead{\includegraphics[height=0.1\textwidth]{logo1.jpg}}
- \geometry{verbose,tmargin=2.5cm,bmargin=2.5cm,lmargin=2.5cm,rmargin=2.5cm}  
---
```
Al poner TOC= tes nos hace el indice de contenido, pero no el de figuras, que podemos sacarlo añadiendo tras el preambulo esta sencilla linea de código LateX:

```{}
\listoffigures
\pagebreak
```` 
En mi caso al poner *lang: "es"* en el preambulo me pone todo en español, por lo que no me funciona los parametros de renombrar, pero si a alguno no le va puede añadir en el aparte de header-includes:,lo siguiente:
- \renewcommand{\contentsname}{Índice}
- \renewcommand{\listfigurename}{Índice de figuras}
- \renewcommand{\listtablename}{Índice de tablas}

### Variables genericas
Vienen muy bien hacer una variable global con el ancho y el largo de la pagina que luego podemos usar en la definición de figuras.
Para ello y dentro del cógigo R podemos poner lo siguiente tras el preambulo
```{r}
 library(knitr)
  options(formatR.arrow=TRUE,width=50)
  opts_chunk$set(fig.path='figure/graphics-', 
                 cache.path='cache/graphics-', 
                 fig.align='center',
                 external=TRUE,
                 echo=TRUE,
                 warning=FALSE
                )
  a4width<- 8.3
  a4height<- 11.7
```
De esta forma para insertar una gráfica podemos definir su ancho y alto en % respecto al total de la paguna:
```{}
{r fig.width = 0.4*a4width, fig.height = 0.2*a4height}
plot(1:34)
```
## INSERTAR IMAGENES <a name="INSERT_IMAG"></a>

Métodos de insertar imagenes en markdown y KnitR
### Con markdown directamente
```{}
![Foto del puente \label{Foto 1}](foto1.jpg)
```
![Foto del puente \label{Foto 1}](foto1.jpg)

### Con markdown y algo de LateX
En esta caso añadimos una label o etiqueta a la que luego hacemos referencia poneinedo \ref{Foto 2}
```{}
![Foto del puente \label{Foto 2}](foto1.jpg)
```
![Foto del puente \label{Foto 2}](foto1.jpg)

### Con LateX solo

Se añade esta liena tal cual o entre dobre dollar: $$
```{}
\includegraphics[width=0.43\linewidth]{imag/mapa.jpg}
```
tambien se puede insterta entre los simbolos de $$
```{}
$$
\includegraphics[width=0.43\linewidth]{imag/mapa_ulea.jpg}
\hspace{0.3cm}
\includegraphics[width=0.43\linewidth]{imag/mapa_ulea.jpg}
\break
$$
```
### Con KnitR
Donde a4width y a4height son parámetros globales que hemos definido al principio, con el ancho y largo de la hoja.
```{}
{r fig.width = 0.4*a4width, fig.height = 0.2*a4height}
plot(1:34)
```
### Con código R

Tambien podemos hacerlo con R directamente lo que nos permite programar.
Aquí insetamos 6 imagenes copias de una misma:
```{}
{r fotomat,fig.cap='Varias fotos en una matriz',fig.width=6,fig.height = 0.5*a4height, out.width = '\\textwidth'}
```
```{r fotomat,fig.cap='Varias fotos en una matriz',fig.width=6,fig.height = 0.5*a4height, out.width = '\\textwidth'}
library(jpeg)
library(grid)

setwd("C:/R/proyectos/imagenes")
img <- readJPEG("foto1.jpg")

# pinta una matriz con la imagen un numero de veces especificado
pintaRaster<- function(img,i=4) {
  par(mfrow=c(i/2, 2), mar=c(0, 0, 1, 0))
# mar= vector con margenes(bottom, left, top, right)
  for(j in 1:i*2){
  plot.new()
  plot.window(xlim=c(0, 1), ylim=c(0, 1), asp=1)
  rasterImage(img, 0, 0, 1, 1)
  title(paste0(letters[j], '. Image ', j), font.main=1)
}
}

pintaRaster(img,6)
```

### Con código R 2
Esta es una funcion para insertar un png desde código R:
```{r}
library(grid)
library("png")
# leemos la imagen
img <- readPNG("myfile.png")

# definimos la funcion para pintar un png
pintaRaster<- function(img) {
  par(mar=c(0,0,0,0))
  plot.new()
  plot.window(xlim=c(0, 1), ylim=c(0, 1), asp=1)
  rasterImage(img, 0, 0, 1, 1)
}

# llamamos ala funcion
pintaRaster(img)
dev.off()
```

# CREAR ENTRADAS DESDE CODIGO
 escribe un fichero desde R
```{r}
################################################################
## crea código LATEX para insertar las imagenes de un directorio
################################################################

setwd("C:/R/proyectos/SCADA/ulea")
imagenes<-list.files("./imag/" )
#imagenes
length(imagenes)
## writeLines(line,"figurasLatex.tex", sep = "\n", useBytes = FALSE)
limpia<- function (x) gsub('([[:punct:]])|\\s+','_',x);



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
