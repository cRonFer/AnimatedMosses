Animated Plot Mosses
================
CRonquillo
18/5/2021

> **Paquetes necesarios**

``` r
library(stringr)
library(ggplot2)
library(ggpubr)
library(Hmisc)
library(ggforce)
library(plotly)
library(gganimate)
library(magick)
library(sf)
library(rnaturalearth)
library(rnaturalearthdata)
```

### Preparación del *environment*

``` r
rm(list=ls(all=T)) # clears workspace
set.seed(2)
setwd("C:/Users/cristina/Desktop/MNCN/articulo") #set the working directory
```

-----

## **Create a distributional map of occurrences**

### 1\. Load a map of Iberia as a ‘sf’ polygon from the natural earth data

``` r
iberia_map <- ne_countries(country = c("spain", "portugal"), scale = 'medium', returnclass = "sf")
```

### 2\. Load our dataset of iberian mosses occurrences (GBIF revised):

``` r
bryo_db_year <- read.csv("C:/Users/cristina/Desktop/MNCN/articulo/dataSet_Bryo_revised.csv", sep=";",header=TRUE)

## Filter dataset for established period (here 1970-2000) 
## Select coordinates and year vars
data<-bryo_db_year %>% 
  filter(year>1970 & year<2001) %>% 
  select("Latitude", "Longitude", "year")
data<-unique(data) #keep unique values of coordinates and year
```

### 3\. Transform the dataframe into shapefile

``` r
geo_daff<-data%>%
  st_as_sf(coords = c( "Longitude","Latitude"), 
           crs = 4326, agr = "constant")
```

### 4\. Plot Time\!

#### Animated plot of geographical distribution of occurrences per year

``` r
r<-ggplot() +
  geom_sf(data = iberia_map, fill = "white") + #shp background
  
  geom_sf(data = geo_daff, shape = 16, size = 1.5, #point occurrences shapefile
          colour = "#8AB17D") +
  coord_sf(xlim = c(-10, 5), ylim = c(35, 45))+ #to centre map and avoid the oceanic islands
  transition_states(as.factor(year), state_length = 3)+ 
  shadow_mark(past = TRUE) + # keed previous points
  ggtitle("Records of iberian bryophytes gathered by year\n
        Year: {closest_state}") # the title changes with the year
```

![](C:/Users/cristina/Desktop/MNCN/articulo/distribution_1970to2000.gif)

-----

## **Plot diversity metrics of mosses**

### 1\. Load a dataset of iberian mosses corresponding diversity metrics:

``` r
diversity<-read.csv("C:/Users/cristina/Desktop/MNCN/articulo/bryophytes_diversity.csv", sep=";", header=TRUE)

# Subset for same period of time

div<-diversity %>% filter(year>1970 & year<2001)
```

### *Number of records per year*

### Bar plot:

``` r
s<-ggplot() +
  geom_bar(div, mapping=aes(x = year , y = Counts),stat="identity",fill="seagreen")+
  labs(x="year", y ="Number of records")+
  scale_x_continuous(name= "Year", breaks = seq(1970, 2000, by = 5))
# How to animate the previous plot
plot_s<-s+
  transition_time(year)+ 
  shadow_mark(past=TRUE) # to keep previous bars
```

![](C:/Users/cristina/Desktop/MNCN/articulo/occurrences_per_year1970-2000.gif)

### Point-Line plot:

``` r
plot_s2 <- qplot(x = year,
                 y = Counts,
                 data = div,
                 geom = c("point", "line")) +
  ylab("Number of records") +
  xlab("year") +
  scale_x_continuous(name= "Year", breaks = seq(1970, 2000, by = 5))+
  transition_reveal(year)
```

![](C:/Users/cristina/Desktop/MNCN/articulo/LineNumberOcc_per_year1970-2000.gif)

### *Number of new species accumulated per year*

### Line plot:

``` r
t<-ggplot(div, mapping=aes(x = year , y = acumulado),stat="identity") +
  geom_line(color="indianred")+
  geom_point(color="indianred")+
  labs(x="year", y ="Spp richness accum")+
  scale_x_continuous(name= "Year", breaks = seq(1970, 2000, by = 5))

plot_t<-t+transition_reveal(year) #animation
```

![](C:/Users/cristina/Desktop/MNCN/articulo/richnessAcum70-00.gif)

-----

## **Combine animated plots**

### Render each animated plot as external ‘magick-image’

``` r
gif_distri<-animate(r, width=320,height = 320) # add fps= to change speed
gif_temp<-animate(plot_s, width=320,height = 340)#lil bigger than map
gif_acum<-animate(plot_t, width=320,height = 340)#lil bigger than map
```

### Create a new gif with the first frame of each plot

``` r
new_gif <- image_append(c(gif_distri[1], gif_acum[1],gif_temp[1]))
```

### Now append each frame in a loop

``` r
for(i in 2:100){ # to 100 cause it has 100 frames by default 
  combined <- image_append(c(gif_distri[i], gif_acum[i],gif_temp[i]))
  new_gif <- c(new_gif, combined)
}
```

![](C:/Users/cristina/Desktop/MNCN/articulo/3plotGifAnimatMosses2.gif)
