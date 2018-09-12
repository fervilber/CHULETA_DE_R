## ggmap
Es una librería que nos permite pintar mapas de manera muy sencilla

para usarla debemos primero bajar un mapa con la función `get_map()` y después dibujarlo con la función `ggmap()`.

la función get_map() es completa tienen muchas maneras de personalizar el mapa que deseamos definir, entre otras cosas la localizción el fondo del mapa las lineas de coordenadas, el zoom por defecto (entre 3 y 21),scale que determina la resolucion del mapa, etc.

```{r}
library(ggmap)
corvallis <- c(lon = -123.2620, lat = 44.5646)

# Get map at zoom level 13: map_5
map_5 <- get_map(corvallis, zoom = 13, scale = 1)

# Plot map at zoom level 5
ggmap(map_5)


##accceso a uno de los poligonos el numro 169
# 169th element of countries_spdf@polygons: one
one<-countries_spdf@polygons[[169]]


# Use ggmap() instead of ggplot()
ggmap(corvallis_map_bw,
      base_layer = ggplot(preds, aes(lon, lat)),extent = "normal", maprange=FALSE) +
  geom_tile(aes(fill=predicted_price),alpha=0.8)
  
  qmplot(lon, lat, data = ward_sales, geom = "polygon", group = group, fill = avg_price)

# Fix the polygon cropping
ggmap(corvallis_map_bw,
      base_layer = ggplot(ward_sales, aes(lon, lat)),extent = "normal", maprange=FALSE) +
  geom_polygon(aes(group = group, fill = ward))

# Repeat, but map fill to num_sales
ggmap(corvallis_map_bw,
      base_layer = ggplot(ward_sales, aes(lon, lat)),extent = "normal", maprange=FALSE) +
  geom_polygon(aes(group = group, fill = num_sales))

# Repeat again, but map fill to avg_price
ggmap(corvallis_map_bw,
      base_layer = ggplot(ward_sales, aes(lon, lat)),extent = "normal", maprange=FALSE) +
  geom_polygon(aes(group = group, fill = avg_price),alpha = 0.8)
```
### agregar capa base

```{r}
# Use base_layer argument to ggmap() to specify data and x, y mappings
ggmap(map_5_bw,
  base_layer= ggplot(ventas, aes(lon, lat, color = ano)))+
  geom_point()

# Use base_layer argument to ggmap() to specify data and x, y mappings
ventas$ano<-as.numeric(ventas$ano)

 ggmap(map_5_bw,
     base_layer= ggplot(ventas, aes(lon, lat)))+
     geom_point(aes(color = precio))+
     facet_wrap(~ano)
```
### formula simplificada `qmplot()`

`ggmap` también proporciona una alternativa rápida al estilo de qplot() en ggplot2. Esta versión es `qmplot()` es menos flexible que una especificación completa, pero a menudo implica menos escritura de código y en una linea  reemplaza  los dos pasos: descargar el mapa y muestra el mapa.

Echemos un vistazo a la versión qmplot () de la gráfica facetada del ejercicio anterior:
```{r}
qmplot(lon, lat, data = sales, 
       geom = "point", color = class) +
  facet_wrap(~ class)

# Plot house sales using qmplot()
qmplot(lon, lat, data = sales, 
       geom = "point", color = bedrooms) +
  facet_wrap(~ month)

# Plot house sales using qmplot()
qmplot(lon, lat, data = ventas, 
  geom = "point", color = precio) +
  facet_wrap(~ ano)
```

## tmap
```{r}
library(tmap)

tm_shape(nyc_tracts_merge) +
  # Add title and change palette
  tm_fill(col = "estimate", 
          title = "Median Income",
          palette = "Greens") +
  # Add tm_borders()
  tm_borders(col = "grey60",lwd = 0.5) +
  tm_shape(water) +
  tm_fill(col = "grey90") +
  tm_shape(manhat_hoods) +
  # Change col and lwd of neighborhood boundaries
  tm_borders(col = "grey40",lwd = 2) +
  tm_text(text = "name", size = 0.5) +
  # Add tm_credits()
  tm_credits("Source: ACS 2014 5-year Estimates, \n accessed via acs package",position = c("right", "bottom"))
  
  save_tmap(filename="nyc_income_map.png",width = 4 , height = 7)
```

ver proyecciones de la capa:

# proj4string() on nyc_tracts and neighborhoods
proj4string(neighborhoods)
proj4string(nyc_tracts)
  
```{r}
library(tmap)

# gsub() to replace " " with "\n" linea nueva
manhat_hoods$name<-gsub(" ","\n",manhat_hoods$NTAName)

# gsub() to replace "-" with "/\n"
manhat_hoods$name<-gsub("-","/\n",manhat_hoods$name)

# Edit to map text to name, set size to 0.5
tm_shape(nyc_tracts_merge) +
    tm_fill(col = "estimate") +
  tm_shape(water) +
    tm_fill(col = "grey90") +
  tm_shape(manhat_hoods) +
    tm_borders() +
    tm_text(text = "name",size=0.5)
    
# Merge nyc_tracts and nyc_income: nyc_tracts_merge
nyc_tracts_merge<-merge(nyc_tracts,nyc_income,by.x = "TRACTCE",by.y = "tract")

# Call summary() on nyc_tracts_merge
summary(nyc_tracts_merge)    
  tm_shape(nyc_tracts_merge) +
  tm_fill(col = "estimate") +
  # Add a water layer, tm_fill() with col = "grey90"
  tm_shape(water) +
  tm_fill(col = "grey90")  +
  # Add a neighborhood layer, tm_borders()
  tm_shape(neighborhoods)     +
  tm_borders()  
  
  
 # Find unique() nyc_tracts_merge$COUNTYFP
unique(nyc_tracts_merge$COUNTYFP)
# el valor unico es 061
# Add logical expression to pull out New York County
manhat_hoods <- neighborhoods[neighborhoods$CountyFIPS=="061", ]

tm_shape(nyc_tracts_merge) +
  tm_fill(col = "estimate") +
  tm_shape(water) +
  tm_fill(col = "grey90") +
  # Edit to use manhat_hoods instead
  tm_shape(manhat_hoods) +
  tm_borders() +
  # Add a tm_text() layer
  tm_text(text="NTAName")  
  
  

# A set of intuitive colors
intuitive_cols <- c(
  "darkgreen",
  "darkolivegreen4",
  "goldenrod2",
  "seagreen",
  "wheat",
  "slategrey",
  "white",
  "lightskyblue1"
)

# Use intuitive_cols as palette
tm_shape(land_cover) +
  tm_raster(palette = intuitive_cols)+
  tm_legend(position = c("left", "bottom"))  
```

## ggvis
```{r}
mtcars %>% ggvis(~wt, ~mpg, fill:=hp) %>% layer_points() %>% layer_smooths()  
 pressure %>% ggvis(~temperature, ~pressure,fill = ~temperature) %>% layer_points()
 ```
 
 ## Lectura en un shapefile

Los archivos shape son una de las formas más comunes en que los datos espaciales se comparten y se leen fácilmente en R usando readOGR () del paquete rgdal. readOGR () tiene dos argumentos importantes: dsn y layer. Exactamente lo que pasa a estos argumentos depende del tipo de datos que está leyendo. Aprendió en el video que para shapefiles, dsn debería ser la ruta al directorio que contiene los archivos que componen el shapefile y la capa es el nombre del archivo del shapefile particular (sin ninguna extensión).

Para su mapa, quiere límites del vecindario. Descargamos las Áreas de Tabulación de Vecindarios<https://www1.nyc.gov/site/planning/data-maps/open-data/dwn-nynta.page>, según la definición de la Ciudad de Nueva York, desde la Plataforma de Datos Abiertos del Departamento de Planificación de la Ciudad. La descarga fue en forma de archivo zip y hemos puesto el resultado de descomprimir el archivo descargado en su directorio de trabajo.

Utilizará la función dir() desde la base R para examinar el contenido de su directorio de trabajo, y luego leerá en el shapefile a R.

```{r}
library(sp)
library(rgdal)

# Use dir() to find directory name
dir()

# Call dir() with directory name
dir("nynta_16c")

# Read in shapefile with readOGR(): neighborhoods
# readOGR(dir, capa)
neighborhoods<-readOGR("nynta_16c","nynta")

# summary() of neighborhoods
summary(neighborhoods)

# Plot neighboorhoods
plot(neighborhoods)
```
## raster
Los archivos ráster se leen con mayor facilidad en R con la función ráster () del paquete ráster. Simplemente ingrese el nombre de archivo (incluida la extensión) del ráster como primer argumento, x.

La función raster () utiliza algunas funciones de paquete de ráster nativo para leer en ciertos tipos de archivos (según la extensión en el nombre del archivo) y de lo contrario pasa la lectura del archivo a readGDAL () desde el paquete rgdal. El beneficio de no usar readGDAL () directamente es simplemente que raster () devuelve un objeto RasterLayer.

Un tipo común de archivo ráster es el GeoTIFF, con la extensión de archivo .tif o .tiff. Hemos descargado un ráster mediano de ingresos del censo de EE. UU. Y lo hemos colocado en su directorio de trabajo.
Echemos un vistazo y léalo.
```{r}
library(raster) 

# Call dir()
dir()

# Call dir() on the directory
dir("nyc_grid_data")

# Use raster() with file path: income_grid
income_grid<-raster("nyc_grid_data/m5602ahhi00.tif")

# Call summary() on income_grid
summary(income_grid)

# Call plot() on income_grid
plot(income_grid)
```

## crear paleta de color para raster
```{r}
library(RColorBrewer)
brewer.pal(9, "BuPu")

tm_shape(prop_by_age) +
  tm_raster("age_18_24", palette = vir) +
  tm_legend(position = c("right", "bottom"))
# creamos una escala de color 
# primero vemos las que hay en RColorBrewer
display.brewer.all
# nos gusta la RdGy y generamos una escala de 7 colores con ella

red_gray <- brewer.pal(7,"RdGy")

# Print migration
print(migration)

# Diverging "RdGy" palette
red_gray <- brewer.pal(7,"RdGy")

# Use red_gray as the palette 
tm_shape(migration) +
  tm_raster(palette = red_gray) +
  tm_legend(outside = TRUE, outside.position = c("bottom"))

# Add fixed breaks 
tm_shape(migration) +
  tm_raster(palette = red_gray,
            style="fixed",
            breaks=c(-5e6, -5e3, -5e2, -5e1, 5e1, 5e2, 5e3, 5e6)) +
  tm_legend(outside = TRUE, outside.position = c("bottom"))

# Plot land_cover
tm_shape(land_cover) +
  tm_raster() 


# Palette like the ggplot2 default
hcl_cols <- hcl(h = seq(15, 375, length = 9), 
                c = 100, l = 65)[-9]

# Use hcl_cols as the palette
tm_shape(land_cover) +
  tm_raster(palette = hcl_cols)
```

## Ejemplos
```{r}
      library(sp)
      library(rgdal)
      library(maptools)
      
      # Bajamos los datos de la web de la chsegura 
      mapa.rios <- readShapeSpatial("capas/CHS_Red_Hidrografica.shp")
      plot(mapa.rios)
      summary(mapa.rios)

tm_shape(mapa.rios) +
  tm_lines(col="Shape_len",palette="Dark2",lwd =1)+
  tm_text(text = "NOMBRE", size = 0.4,auto.placement=T)

list.files(pattern="*.shp$", recursive=TRUE, full.names=TRUE) 

library(ggmap)
murcia <- get_googlemap(center = c(lon = -1.1636353 , lat = 37.9439891), zoom = 10, scale = 2)
# murcia1 <- get_map(location = "murcia", zoom = 14, source = "osm", color = "bw")

rios.wgs84 <- spTransform(rios, CRS("+proj=longlat +datum=WGS84"))
rios.wgs84 <- fortify(rios.wgs84)

ggmap(murcia)+
  geom_polygon(data = capa, aes(long, lat),alpha = .7)

ggmap(murcia)+
  geom_path(data = rios.wgs84, aes(long, lat, group = group))
```
