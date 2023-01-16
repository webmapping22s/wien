# Wien Beispiel HOWTO (Teil 3) - Icons für Punktlayer verwenden

## Layer Sehenswürdigkeiten verfeinern

### a) Popup hinzufügen

* zur besseren Übersicht deaktivieren wir die vier anderen Layer indem wir den Aufruf der jeweiligen Funktion auskommentieren und die Funktion im VS Code einklappen

    [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/bff2d9deee04e7c97c2fe1baa31f1b7f746c6b9a)


* wir könnten, wie in der Leaflet Dokumentation von [L.geoJSON](https://leafletjs.com/reference.html#geojson) im *Usage example* gezeigt, das Popup einfach vor dem `.addTo(overview)` hinzufügen, das machen wir aber nicht, denn in einem späteren Schritt würde uns das nur Probleme bereiten. Deshalb verwenden wir das Options-Objekt und werden bei [pointToLayer](https://leafletjs.com/reference.html#geojson-pointtolayer) fündig. Die *Description* zeigt uns ein passendes Beispiel, wie wir den Marker besser kontrollieren können.

    ```javascript
    L.geoJSON(geojson, {
        pointToLayer: function(geoJsonPoint, latlng) {
            return L.marker(latlng);
        }
    }).addTo(overlay);;
    ```

    [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/3ecbb36ad365048ab2a01ce484d6feecc864beb1)

    * der Funktion, die bei `pointToLayer` für jeden Punkt des Datensatzes ausgeführt wird, wird von Leaflet automatisch der aktuelle Punkt (`geoJsonPoint`) und seine Koordinate (`latlng`) übergeben

    * am Ende der Funktion müssen wir den Marker mit `return` zurückgeben damit er in der Karte auch gezeichnet wird

* das Popup definieren in *Template-Syntax* mit *Backticks* vor dem Marker in einer Variablen `popup` und hängen es mit `bindPopup` an den Marker. Wir erinnern uns daran, wie wir die Attribute des Punkts ansprechen können (z.B. `geoJsonPoint.properties.NAME`) und verwenden die Attribute `NAME`, `ADRESSE`, `THUMBNAIL`, `WEITERE_INF` für das Popup

    ```javascript
    let popup = `
        <img src="${geoJsonPoint.properties.THUMBNAIL}" alt=""><br>
        <strong>${geoJsonPoint.properties.NAME}</strong>
        <hr>
        Adresse: ${geoJsonPoint.properties.ADRESSE}<br>
        <a href="${geoJsonPoint.properties.WEITERE_INF}" target="Wien">Weblink</a>
    `;
    return L.marker(latlng).bindPopup(popup);
    ```

    * `alt=""` bedeutet, dass unser Bild rein dekorativ ist. Wir wissen ja auch gar nicht, was auf den einzelnen Bildern zu sehen ist.
    
    * `target="Wien"` bewirkt wie beim Neuseelandbeispiel, dass alle Weblinks im selben Tab geöffnet werden

    [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/192c2bb193dced58e003575f267cb815f2b8e4c3)

* damit ist das Popup fertig, als nächstes verwenden wir ein eigenes Icon an Stelle der Marker

### b) ein eigenes Icon statt dem Marker verwenden

* als Quelle für unser Icon verwenden wir die [Map Icons Collection - 1000+ free & customizable icons for maps](https://mapicons.mapsmarker.com/)

    * Suche *photo* -> Photo wählen

    * wir versuchen gleich das *customizable* Feature und wählen Weiß (`#ffffff`) als Farbe beim Farbwähler und klicken auf `Generate`

    * die Seite wird neu geladen, die Icons entsprechend angepasst und wir finden unsere Farbe in der URL <https://mapicons.mapsmarker.com/markers/media/photo/?custom_color=ffffff> wieder - dieses Feature werden wir später bei den Icons für die Bushaltestellen noch brauchen ...

    * wir wählen das linke Icon und speichern es im `icons/`-Verzeichnis als `photo.png`

    [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/3b23c65190f2b901f12d73b1d76466c55a05616d)

* eingebaut wird das Icon beim Erzeugen des Markers - die Dokumentation von [L.marker](https://leafletjs.com/reference.html#marker) zeigt bei den Options, dass wir `icon` als Option definieren müssen. Wie dieses Icon von der Syntax her aussehen muss, zeigt der Link zur Dokumentation von [L.icon](https://leafletjs.com/reference.html#icon) in der *Description* der Option `icon`. Wir folgen dem Link und sehen ein komplettes *Usage example* von dem wir zunächst nur die Option `iconUrl` verwenden - sie zeigt auf unser Icon `photo.png` im Verzeichnis `icons/`

    ```javascript
    return L.marker(latlng, {
        icon: L.icon({
            iconUrl: "icons/photo.png",
        })
    }).bindPopup(popup);
    ```

* die Optionen `iconAnchor`, `popupAnchor` beim Beispiel der Dokumentation weisen darauf hin, dass die Positionierung des Icons verändert werden kann. Per Default werden die Koordinaten des Punkts auf die linke, obere Ecke des Icons gesetzt - das schauen wir uns an und fügen temporär einen zweiten default Marker an der selben Position hinzu

    ```javascript
    L.marker(latlng).addTo(map);    // temporär zum Testen des Anfasspunkts
    let mrk = L.marker(latlng, {
        // ...
    });
    ```

* die Spitze unseres Icons mit der Größe **32x37** Pixel liegt von Links, Oben gesehen bei **16** und **37**, diese Werte verwenden wir beim `iconAnchor` Array

    ```javascript
    icon: L.icon({
        iconUrl: "icons/photo.png",
        iconAnchor: [16, 37],
    })
    ```

* wenn wir auf das Popup klicken, verdeckt das Popup das Icon, deshalb schieben wir es beim `popupAnchor` Array um 37 Pixel nach Oben

    ```javascript
    icon: L.icon({
        iconUrl: "icons/photo.png",
        iconAnchor: [16, 37],
        popupAnchor: [0, -37]
    })
    ```

* damit sind Icon und Popup richtig positioniert und wir können den temporären Marker löschen oder auskommentieren. der fertige Code für unser Icon sieht so aus:

    ```javascript
    return L.marker(latlng, {
        icon: L.icon({
            iconUrl: "icons/photo.png",
            iconAnchor: [16, 37],
            popupAnchor: [0, -37]
        })
    }).bindPopup(popup);
    ```

    [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/850d71e3d85d0d730e6cb9854cf03aa0f534f5d4)


## Layer Bushaltestellen verfeinern

Bevor wir weiterarbeiten, setzen wir den Zoomfaktor beim Laden der Karte auf 16 und Blicken damit auf die Wiener Innenstadt

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/5f7d2982fae965831eabfc46c0f853d1fa6e84c6)

* als Quelle für das Bus Icon verwenden wir wieder die [Map Icons Collection - 1000+ free & customizable icons for maps](https://mapicons.mapsmarker.com/), suchen nach *photo*, stellen `#ffffff` als Farbe ein und speichern uns das neu generierte Icon als `bus.png` in dass Verzeichns `icons`

    [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/6080f910af7b83cef98826ea88c534ce8d88d919)

### a) Farbige Icons für die Bushaltestellen erzeugen

Den Haltestellenlayer erzeugen wir mit *Copy-Paste* des Codes der Sehenswürdigkeiten und ein paar kleinen Änderungen in **vier Schritten**

1. wir kopieren den geamten `L.geoJSON()` Code der Sehenswürdigkeiten und überschreiben damit den bestehenden `L.geoJSON()` Aufruf in der Funktion `loadStops` 

2. wir bessern den Popup-Code aus und verwenden die Attribute `STAT_NAME` und `LINE_NAME`

3. wir bessern die `iconUrl` aus und verwenden `icons/bus.png` statt `icons/photo.png`

4. wir löschen den Kommentar beim Funktionsaufruf `loadStops("https://...")` und aktivieren damit das Zeichnen des Haltestellen Layers

```javascript
L.geoJSON(geojson, {
    pointToLayer: function(geoJsonPoint, latlng) {
        let popup = `
            <strong>${geoJsonPoint.properties.LINE_NAME}</strong><br>
            Station ${geoJsonPoint.properties.STAT_NAME}
        `;
        return L.marker(latlng, {
            icon: L.icon({
                iconUrl: "icons/bus.png",
                iconAnchor: [16, 37],
                popupAnchor: [0, -37]
            })
        }).bindPopup(popup);
    }
}).addTo(overlay);
loadStops("https://data.wien.gv.at/daten/geo?service=WFS&request=GetFeature&version=1.1.0&typeName=ogdwien:TOURISTIKHTSVSLOGD&srsName=EPSG:4326&outputFormat=json");
```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/47c965d8991fa746f90a4584ff13642f0f449238)

Ein Blick auf die Attribute der Bushaltestellen zeigt, dass zusätzlich zum Namen der Linie in `LINIE_NAME`, jede Linie eine `LINIE_ID`besitzt, deren Werte von 1 bis 6 gehen.

Diese `LINE_ID` wird entscheiden, welches Icon wir verwenden und deshalb generieren wir bei der [Map Icons Collection](https://mapicons.mapsmarker.com/markers/media/photo/?custom_color=ffffff) sechs Icons mit passenden Farben der Webseite [Colors - A nicer color palette for the web](https://clrs.cc/)

**Übungsaufgabe: 6 Icons nach folgenden Vorgaben erzeugen**

* Zielverzeichnis: `icons/`
* Farben nach Namen der Linien: Farbwerte siehe <https://clrs.cc/>
* Namen: `bus_1.png` bis `bus_6.png`
* wir erinnern uns, dass wir die Farben direkt beim Aufruf der Map Icons Seite verwenden können ...

**Lösung für die URLs bei der Map Icons Collection**

* [bus_1.png](https://mapicons.mapsmarker.com/markers/road-transportation/bus/?custom_color=FF4136) (RED)
* [bus_2.png](https://mapicons.mapsmarker.com/markers/road-transportation/bus/?custom_color=FFDC00) (YELLOW)
* [bus_3.png](https://mapicons.mapsmarker.com/markers/road-transportation/bus/?custom_color=0074D9) (BLUE)
* [bus_4.png](https://mapicons.mapsmarker.com/markers/road-transportation/bus/?custom_color=2ECC40) (GREEN)
* [bus_5.png](https://mapicons.mapsmarker.com/markers/road-transportation/bus/?custom_color=AAAAAA) (GRAY)
* [bus_6.png](https://mapicons.mapsmarker.com/markers/road-transportation/bus/?custom_color=FF851B) (ORANGE)


[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/9eab797a2cb853b6db3c61cbd0c89bf502ad325b)

Damit bleibt noch, die Icons je nach `LINE_ID` zu verwenden, wir bedienen uns wieder der *Template-Syntax* mit *Backticks* bei der `iconUrl` und erzeugen Links zu den Icons `bus_1.png` bis `bus_6.png`. Die Zahl, und damit die Farbe, folgt dem Attribut `LINE_ID`

```javascript
icon: L.icon({
    iconUrl: `icons/bus_${geoJsonPoint.properties.LINE_ID}.png`,
    // ...
})
```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/b88d64f0af82c119d27d360e03a1cc65b61ae52b)

## Layer Hotels und Unterkünfte verfeinern

Bevor wir die Funktion `laodHotels` verändern, besorgen wir uns drei Icons für die Hotels und Unterkünfte und speichern sie im Verzeichnis `icons/`

* Icon Hotel, Farbe PURPLE - `#B10DC9`, Icon [hotel_0star](https://mapicons.mapsmarker.com/markers/restaurants-bars/hotels/hotel/)

* Icon Pension, Farbe PURPLE - `#B10DC9`, Icon [lodging_0star](https://mapicons.mapsmarker.com/markers/restaurants-bars/hotels/lodging/)

* Icon Appartment, Farbe PURPLE - `#B10DC9`, Icon [apartment-2](https://mapicons.mapsmarker.com/markers/friends-family/apartment/)

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/b0782a9123a6a7622c4622a19a0373a17dea113a)

Danach erzeugen wir den Layer für Hotels und Unterkünfte wieder mit *Copy-Paste*, diesmal des Codes der Haltestellen und ein paar kleinen Änderungen in den bekannten **vier Schritten**

1. wir kopieren den geamten `L.geoJSON()` Code der Haltestellen und überschreiben damit den bestehenden `L.geoJSON()` Aufruf in der Funktion `loadHotels`

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/5ef429cda320a06e2df9a4407df5761e6da70eb3)

2. wir bessern den Popup-Code aus und verwenden die Attribute `ADRESSE`, `BETRIEB`, `BETRIEBSART_TXT`, `KATEGORIE_TXT`, `KONTAKT_EMAIL`, `KONTAKT_TEL`, und, `WEBLINK1`

    ```javascript
    let popup = `
        <p>
            <strong>
                ${geoJsonPoint.properties.BETRIEB} -
                ${geoJsonPoint.properties.BETRIEBSART_TXT}
                ${geoJsonPoint.properties.KATEGORIE_TXT}
            </strong>
            <br>
            ${geoJsonPoint.properties.ADRESSE}
        </p>
        <hr>
        <address>
            Tel.:${geoJsonPoint.properties.KONTAKT_TEL}<br>
            E-Mail: <a href="mailto:${geoJsonPoint.properties.KONTAKT_EMAIL}">${geoJsonPoint.properties.KONTAKT_EMAIL}</a><br>
            Web: <a href="${geoJsonPoint.properties.WEBLINK1}" target="Wien">${geoJsonPoint.properties.WEBLINK1}</a>
        </address>
    `;
    ```

    * `target="Wien"` bewirkt wieder, dass alle Weblinks im selben Tab geöffnet werden

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/939dec53c7fb1e24a61c915763e3a21f942f2064)

3. wir bessern die `iconUrl` aus und verwenden vorerst `icons/hotel_0star.png` statt `icons/bus.png` für alle Icons. Die Unterscheidung nach Typen folgt später

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/80e9c62226d7d33bb0cff515cb5c5f4bae65ddc3)

4. wir löschen den Kommentar beim Funktionsaufruf `loadHotels("https://...")` und aktivieren damit das Zeichnen des Hotel Layers

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/fabee948902f6595f06cc0d9af6ba3481de94a58)

Bleibt noch, die Hotels und Unterkünfte nach Typen zu unterscheiden. Dazu verwenden wird das Attribut `BETRIEBSART` und ermitteln in einem `if, else if, else` Block das passende Icon für die Werte **H** (Hotel), **P** (Pension) und **A** (Appartment). Wir speichern den gefundenen Icon-Namen in einer Variablen `icon`, die wir vor der if-Abfrage initialisieren.

```javascript
let icon;
if (geoJsonPoint.properties.BETRIEBSART == "H") {
    icon = "hotel_0star";
} else if (geoJsonPoint.properties.BETRIEBSART == "P") {
    icon = "lodging_0star";
} else {
    icon = "apartment-2";
}
```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/45f40531104fe01aa0dfab536ae8757f243ff073)

Den gefundenen Icon-Namen setzen wir schließlich bei der `iconUrl` ein. Damit werden je nach Typ, verschiedene Icons angezeigt.

```javascript
icon: L.icon({
    iconUrl: `icons/${icon}.png`,
    // ...
})
```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/0c2da982920ec2817f1286991c38f1528c2185bf)
