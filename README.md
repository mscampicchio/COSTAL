The map has been created with the following R code:

# 2. Caricare le librerie
library(readxl)
library(leaflet)
library(htmlwidgets)
library(base64enc)
library(htmltools)
library(sf)

# Convertire il logo in formato Base64
logo <- base64enc::dataURI(
  file = "logo_COSTAL.png",
  mime = "image/png"
)

# 3. Importare i dati Excel
list.files()
dati <- read_excel("COSTAL_Atenei_02.xlsx")

# 4. Creare le informazioni per i popup
dati$popup <- paste0(
  "<b>", dati$Ateneo, "</b><br>",
  "Sede: ", dati$Citta, "<br>",
  "Corsi: ", dati$Corso, "<br>",
  "Coordinatore: ", dati$Coordinatore, "<br>",
  "<a href='", dati$Sito, "' target='_blank'>",
  "Visita il sito</a>"
)

# Assegnare un colore in base ai corsi presenti

dati$Colore <- ifelse(
  grepl("L-26", dati$Corso) & grepl("LM-70", dati$Corso),
  "#E69F00",  # Arancione: entrambi
  ifelse(
    grepl("L-26", dati$Corso),
    "#0072B2",  # Blu: L-26
    "#009E73"   # Verde: LM-70
  )
)


# ==================================================
# 5. CREAZIONE DELLA MAPPA COSTAL
# ==================================================

# 5.1 Creare il confine nazionale
italia <- st_sf(
  geometry = st_union(st_geometry(regioni))
)

# 5.2 Creare la mappa
mappa <- leaflet(
  options = leafletOptions(
    zoomControl = TRUE,
    attributionControl = TRUE
  )
) %>%
  
  # A. Regioni italiane
  addPolygons(
    data = regioni,
    color = "#888888",       # Confini regionali
    weight = 0.8,
    opacity = 1,
    fillColor = "#EAF2F8",   # Azzurro chiaro
    fillOpacity = 1,
    smoothFactor = 0,
    highlightOptions = highlightOptions(
      color = "#444444",
      weight = 1.5,
      bringToFront = FALSE
    )
  ) %>%
  
  # B. Confine nazionale
  addPolygons(
    data = italia,
    color = "#333333",       # Grigio scuro
    weight = 2.5,
    opacity = 1,
    fill = FALSE,
    stroke = TRUE,
    options = pathOptions(
      interactive = FALSE
    )
  ) %>%
  
  # C. Marcatori delle sedi universitarie
  addCircleMarkers(
    data = dati,
    lng = ~Lon,
    lat = ~Lat,
    radius = 7,
    color = "#333333",
    weight = 1,
    fillColor = ~Colore,
    fillOpacity = 0.9,
    popup = ~popup,
    label = ~Ateneo
  ) %>%
  
  # D. Inquadratura geografica
  fitBounds(
    lng1 = 6.5,
    lat1 = 35.3,
    lng2 = 18.7,
    lat2 = 47.2
  ) %>%
  
  addLegend(
    position = "bottomright",
    colors = c("#0072B2", "#009E73", "#E69F00"),
    labels = c(
      "L-26 – Laurea triennale",
      "LM-70 – Laurea magistrale",
      "L-26 + LM-70 – Entrambi"
    ),
    title = "Corsi di laurea",
    opacity = 1
  ) %>%

  addControl(
    html = "
    <div style='
      background: white;
      padding: 8px 12px;
      font-size: 12px;
      color: #555555;
      border-radius: 5px;
      box-shadow: 0 1px 5px rgba(0,0,0,0.15);
    '>
      Cliccare sui marcatori per maggiori informazioni.
    </div>
  ",
    position = "bottomleft"
  ) %>%

  addControl(
    html = paste0(
      "<div style='
        background-color: white;
        padding: 8px;
        border-radius: 5px;
      '>",
      "<img src='", logo, "' ",
      "style='width: 90px; height: auto;'>",
      "</div>"
    ),
    position = "topright"
  )

# 5.3 Impostare lo sfondo bianco
mappa <- onRender(
  mappa,
  "
  function(el, x) {
    el.style.backgroundColor = '#FFFFFF';
  }
  "
)

# 5.4 Visualizzazione
mappa

# ==================================================
# 6. ESPORTAZIONE HTML
# ==================================================

saveWidget(
  mappa,
  file = "COSTAL_Map.html",
  selfcontained = TRUE
)

# 6. Visualizzare la mappa in RStudio
mappa

# 7. Esportare in HTML
saveWidget(
  mappa,
  "COSTAL_Map.html",
  selfcontained = TRUE
)
