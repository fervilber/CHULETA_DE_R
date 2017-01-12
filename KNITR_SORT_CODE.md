

## INDICE
1. [INTRODUCCION](#INTRODUCCION)
2. [ENTORNO DE TRABAJO](#ENTORNO_DE_TRABAJO)
3. [DATA FRAMES](#df)

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
```{r fig.width = 0.4*a4width, fig.height = 0.2*a4height}
plot(1:34)
```

