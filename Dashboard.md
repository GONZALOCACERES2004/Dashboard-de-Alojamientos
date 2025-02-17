---
title: "Untitled"
author: "Gonzalo Caceres"
date: "2024-09-13"
output: html_document
---

```{r setup, include=FALSE}
library(flexdashboard)
library(ggplot2)
library(dplyr)
library(plotly)
library(DT)
library(crosstalk)
library(htmlwidgets)



# Cargar y preparar datos
df_madrid <- readRDS("df_madrid.rds") %>%
  mutate(Bathrooms = round(Bathrooms, 0)) %>%
  select(-Square.Feet) %>%
  filter(across(everything(), ~!is.na(.) & . != 0))  %>%
  arrange((Price))  # Ordenar por precio de mayor a menor

# Crear un objeto SharedData
sd <- SharedData$new(df_madrid)

# Calcular valores mínimos y máximos para los filtros
min_price <- min(df_madrid$Price, na.rm = TRUE)
max_price <- max(df_madrid$Price, na.rm = TRUE)
```

Column {.sidebar data-width=350}
-----------------------------------------------------------------------

### Filters

```{r}
filter_select("neighborhood", "Select Neighborhoods", sd, ~Neighbourhood, multiple = TRUE)

filter_slider("price", "Price", sd, ~Price, width = "100%")

filter_slider("bedrooms", "Bedrooms", sd, ~Bedrooms,min = 1,max =max(sd$data()$Bedrooms, na.rm = TRUE),step = 1, width = "100%")

filter_slider("bathrooms", "Bathrooms", sd, ~Bathrooms,min = 1,max = max(sd$data()$Bathrooms, na.rm = TRUE),step = 1, width = "100%")

filter_slider("accommodates", "Accommodates", sd, ~Accommodates, step = 1, width = "100%")
```



Column{data-width=650}
-----------------------------------------------------------------------

### - 


```{r}
plot_ly(sd, x = ~Neighbourhood, y = ~Price, type = "bar", height = 300) %>%
  layout(
    title = "Price Vs neighborhood",
    xaxis = list(title = "", tickangle = 90, tickfont = list(size = 8)),
    yaxis = list(title = "Price"),
    margin = list(l = 50, r = 50, b = 100, t = 50),
    dragmode = FALSE,
    clickmode = 'none'
  ) %>%
  config(displayModeBar = FALSE)
```

Review Scores Rating


```{r}
plot_ly(sd, 
        x = ~Review.Scores.Rating, 
        y = ~Price, 
        size = ~Accommodates,
        color = ~Neighbourhood,
        type = "scatter", 
        mode = "markers", 
        marker = list(opacity = 0.7, sizemode = "diameter"),
        text = ~paste("Scores:", Review.Scores.Rating,
                      "<br>Neighbourhood:", Neighbourhood,
                      "<br>Price:", Price,
                      "<br>Accommodates:", Accommodates),
        hoverinfo = "text",
        height = 300) %>%
  layout(
    title = "Price vs Review Scores",
    xaxis = list(title = "Review score"),
    yaxis = list(title = "Price"),
    showlegend = FALSE,
    dragmode = FALSE,
    clickmode = 'none'
  ) %>%
  config(displayModeBar = FALSE)


```

Price Vs Square Meters

```{r}
plot_ly(sd, 
        x = ~Square.Meters, 
        y = ~Price, 
        color = ~Neighbourhood,
        type = "scatter", 
        mode = "markers", 
        marker = list(opacity = 0.7),
        text = ~paste("Neighbourhood: ", Neighbourhood,
                      "<br>Price: ", Price,
                      "<br>Square Meters: ", round(Square.Meters,0)),
        hoverinfo = "text",
        height = 300) %>%
  layout(
    title = "",
    xaxis = list(title = "Square Meters"),
    yaxis = list(title = "Price"),
    showlegend = TRUE,
    hoverlabel = list(bgcolor = "white"),
    dragmode = FALSE,
    clickmode = 'none'
  ) %>%
  config(displayModeBar = FALSE)
```

Accommodation Data Table

```{r}
datatable(sd, 
          options = list(
            pageLength = 10,
            scrollX = TRUE,
            columnDefs = list(
              list(visible = FALSE, targets = which(!(names(sd$data()) %in% c("Neighbourhood", "Price", "Bedrooms", "Bathrooms", "Accommodates", "Review.Scores.Rating", "Square.Meters"))))
            )
          ),
          style = "bootstrap", 
          class = "compact", 
          width = "100%"
) %>%
  formatRound(columns = c("Price", "Square.Meters"), digits = 0) %>%
  formatStyle(columns = names(sd$data()), textAlign = 'center')


```
