---
layout: post
title: Visualisation Schmetterlingsdaten - Teil 2
abstract: Abstract
author: Markus Dumke
tags: Visualisation Schmetterlingsdaten
comments: true
---

## Erstellen einer Shiny App
Zunächst müssen wir das Paket Shiny installieren. Dieses erstellt im Folgenden die Visualisation. Eine Shiny App besteht immer aus 2 Teilen: einem ui.R Skript, dass die grafische Benutzeroberfläche (Buttons etc., die der Nutzer klicken kann) erstellt und ein server.R Skript, dass festlegt, was passieren soll, wenn z.B. ein Button geklickt wird. Zunächst müssen wir das Peket laden und die Daten einlesen. Ausserdem brauchen wir noch das leaflet Paket, das die Karten zur Verfügung stellt. Platziere dafür folgenden Code in deiner ui.R **und** in deiner server.R Datei:

```r
library(shiny)
library(leaflet)

# load data
data <- read.csv2("data_proprocessed.csv", encoding = "utf8")
```

Zunächst fangen wir mit einer einfachen App an, die einfach eine Karte anzeigt. Dafür fügen wir in ui.R einen leafletOutput ein, damit wird standardmässig die Openstreetmap Karte eingebunden. Die Breite der Karte setzen wir auf 100% der Bildschirmbreite, die Höhe auf 700 Pixel (Wichtig: Es ist nicht möglich, beide auf 100% zu setzen!)

```r
ui <- fluidPage(
  leafletOutput("map", width = "100%", height = "700")
)
```

In server.R fügen wir dann folgenden Code ein, der die Karte dann tatsächlich erstellt. Mit `setView()` setzen wir den aktuellen Kartenausschnitt. Die ersten Argumente bestimmen Höhen- und Breitengrad des Kartenmittelpunkts, die letzte Zahl den Zoom Faktor. Ändere diese nach Belieben.

```r
shinyServer(
  function(input, output, session) {
    
    output$map <- renderLeaflet({
      leaflet() %>% addTiles()  %>%
        setView(11, 49, 7)
    })
  }
)
```
Jetzt haben wir bereits eine lauffähige App, wenn du in Rstudio auf **Run App** klickst, sollte sich ein Fenster öffnen, in dem eine leere Openstreetmap Karte gezeigt wird.

Als nächstes wollen wir die Koordinaten als Punkte auf der Karte darstellen lassen. Dafür ändern wir server.R, sodass folgender Code dort steht:

```r
shinyServer(
  function(input, output, session) {
    
    points <- reactive({cbind(data$longitude, data$latitude)})
        
    output$map <- renderLeaflet({
      leaflet() %>% addTiles()  %>%
        setView(11, 49, 7) %>% 
        addCircleMarkers(data = points(), fillOpacity = 1, 
                         opacity = 1)
    })
  }
)

```

Jetzt sollte der Output mit **Run App** etwa so aussehen:

![Shiny App]({{ site.url }}/assets/app2.JPG)

Als nächster Schritt wäre es cool, eine Möglichkeit zu haben, welche Daten angezeigt werden sollen, z.B. nur eine bestimmte Art oder nur Beobachtungen ab einem bestimmten Jahr darzustellen.
Dafür fügen wir in ui.R eine sidebar ein, die es uns erlaubt genau diese Auswahlen zu treffen. In Shiny sind zahlreiche Inputs möglich, z.B. Buttons, Checkboxes und TextInputs, für mehr siehe hier: <a href="http://shiny.rstudio.com/gallery/widget-gallery.html" target="_blank">Shiny Widgets</a> 

```r
ui <- fluidPage(
  sidebarPanel(
    selectizeInput("Art", label = "Art", selected = "Polygonia c-album",
                   choices = levels(df$Art), multiple = TRUE)),
    mainPanel(leafletOutput("Karte", width = "100%", height = "700"))
)
```

Damit bei Auswahl einer Art, auch nur die Punkte dieser Art auf der Karte angezeigt werden, müssen wir in server.R ein subset bilden.

```r
shinyServer(
  function(input, output, session) {
    
    data_subset <- reactive({
      data[data$Art %in% input$Art, ]
    })
    
    points <- reactive({cbind(data_subset()$longitude, data_subset()$latitude)})
        
    output$Karte <- renderLeaflet({
      leaflet() %>% addTiles()  %>%
        setView(11, 49, 7) %>% 
        addCircleMarkers(data = points(), fillOpacity = 1, opacity = 1)
    })
  }
)
```


```r

```


```r

```




The code can be found here: [Github](https://github.com/markdumke/lepivis)
{% include disqus.html %}