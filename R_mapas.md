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

## ggvis
```{r}
mtcars %>% ggvis(~wt, ~mpg, fill:=hp) %>% layer_points() %>% layer_smooths()  
 pressure %>% ggvis(~temperature, ~pressure,fill = ~temperature) %>% layer_points()
 ```
