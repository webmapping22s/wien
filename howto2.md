# Wien Beispiel HOWTO (Teil 2) - GeoJSON

## bevor wir loslegen, deaktivieren wir das Overlay für den Stephansdom samt dem Marker mit einem mehrzeiligen Kommentar

```javascript
/*
let sightLayer = L.featureGroup();
layerControl.addOverlay(sightLayer, "Sehenswürdigkeiten");

let mrk = L.marker([ stephansdom.lat, stephansdom.lng]).addTo(sightLayer);

sightLayer.addTo(map);
*/
```

[🔗 COMMIT](https://github.com/webmapping/wien/commit/adfb3a21c9836771681a8e395e8bec698203a1f3)

## GeoJSON als Datenformat

### a) was ist GeoJSON?

* <https://geojson.org/> liefert die Antwort: *GeoJSON is a format for encoding a variety of geographic data structures*

* <https://de.wikipedia.org/wiki/GeoJSON> zeigt uns ein paar Beispiele für die einzelnen Geometrietypen

* mehr müssen wir vorerst nicht wissen und wir holen uns ein erstes **Real World Beispiel** vom OpenData Auftritt der Stadt Wien - links unten in unserer Wienkarte finden wir bei Datenquellen den Link dorthin (<https://digitales.wien.gv.at/open-data/>), wir öffnen ihn in einem neuen Tab.

### b) wie sieht so ein Real World Beispiel aus?

* über die suche *Sehens* finden wir den Datensatz **Sehenswürdigkeiten Standorte Wien** bei <https://data.gv.at>, dem OpenData Portal von Österreich

* auf der Katalog Seite der Sehenswürdigkeiten finden wir den Link zum GeoJSON bei **WFS GetFeature (JSON) JSON**

* über **Mehr Information** landen wir auf der Detailsseite mit der gewünschten [URL](https://data.wien.gv.at/daten/geo?service=WFS&request=GetFeature&version=1.1.0&typeName=ogdwien:SEHENSWUERDIGOGD&srsName=EPSG:4326&outputFormat=json)

* wir sehen uns die Datei `SEHENSWUERDIGOGD.json` in VS Code an und verwenden natürlich **F1 Beautify file** damit wir die Struktur besser erkennen

* wir erkennen, dass es sich um eine Sammlung von *Features* handelt, wieviele es insgesamt sind und in welcher Projektion sie vorliegen

    ```json
    {
        "type": "FeatureCollection",
        "totalFeatures": 63,
        "features": [],
        "crs": {
            "type": "name",
            "properties": {
                "name": "urn:ogc:def:crs:EPSG::4326"
            }
        }
    }
    ```

* die einzelnen *Features* sind im `features`-Array zu finden - dessen Einträge sind wiederum Javascript-Objekte

    ```json
    {
        "type": "Feature",
        "id": "SEHENSWUERDIGOGD.411009",
        "geometry": {
            "type": "Point",
            "coordinates": [16.373675865880358, 48.210760467880625]
        },
        "geometry_name": "SHAPE",
        "properties": {
            "OBJECTID": 411009,
            "NAME": "Ankeruhr",
            "ADRESSE": "Hoher Markt 10/11",
            "WEITERE_INF": "http://www.wien.info/article.asp?IDArticle=2018",
            "THUMBNAIL": "https://www.wien.gv.at/viennagis/bubble/ankeruhr.jpg",
            "SE_ANNO_CAD_DATA": null
        }
    }
    ```

    * wir erkennen die Geometrie in `geometry` und die Attribute in `properties`

    * würde dieses Feature-Objekt in einer Variablen `geoJsonPoint` gespeichert sein, könnten wir so auf die einzelnen Werte zugreifen

        ```js
        geoJsonPoint.geometry.coordinates     // [16.373675865880358, 48.210760467880625]
        geoJsonPoint.geometry.coordinates[0]  // 16.373675865880358
        geoJsonPoint.geometry.coordinates[1]  // 48.210760467880625
        geoJsonPoint.properties.NAME          // Ankeruhr
        geoJsonPoint.properties.ADRESSE       // Hoher Markt 10/11
        geoJsonPoint.properties.THUMBNAIL     // https://www.wien.gv.at/viennagis/bubble/ankeruhr.jpg
        geoJsonPoint.properties.WEITERE_INF   // http://www.wien.info/article.asp?IDArticle=2018
        ```

    * Leaflet versteht das GeoJSON-Format und kann es automatisch in Marker, Linien und Flächen umwandeln. Dazu müssen wir die Daten aber zuerst laden und das geht mit Hilfe einer Funktion, die noch dazu asynchron ausgeführt wird - wir müssen uns also zuerst noch kurz mit Funktionen beschäftigen

## Sidestep: Javascript Funktionen

Eine Funktion ist vereinfacht gesagt ein Codeblock mit einem Namen, der "irgend etwas tut" wenn wir ihn aufrufen. Beim Aufruf können wir Parameter für "das Tun" innerhalb der Funktion mitschicken und wenn der Codeblock abgearbeitet ist, kann die Funktion auch wieder einen einzelnen Wert, einen Array oder eine Objekt zurückliefern.

* eine einfach Funktion zum Addieren von zwei Zahlen sieht damit so aus:

    ```javascript
    funtion addiere (zahl1, zahl2) {
        let summe = zahl1 + zahl2;
        return summe;
    }
    ```

* um die Zahlen 2 und 4 zu addieren, müssen wir die Funktion mit den beiden Zahlen als Parameter aufrufen. Das Ergebnis speichern wir in der Variablen `ergebnis` und zeigen es mit `console.log` an

    ```javascript
    let ergebnis = addiere(2, 4);
    console.log(ergebnis);
    ```

* genau so eine Funktion werden wir jetzt schreiben, um die GeoJSON-Daten der Sehenswürdigkeiten zu laden

### b) GeoJSON-Daten asynchron mit einer Funktion laden

* so sieht diese Funktion zum Laden aus :-o

    ```javascript
    // Sehenswürdigkeiten laden
    async function loadSites(url) {
        let response = await fetch(url);    
        let geojson = await response.json();
        console.log(geojson);
    }
    loadSites("https://data.wien.gv.at/daten/geo?service=WFS&request=GetFeature&version=1.1.0&typeName=ogdwien:SEHENSWUERDIGOGD&srsName=EPSG:4326&outputFormat=json");
    ```

    [🔗 COMMIT](https://github.com/webmapping/wien/commit/4fb66807b69cddea912dce9fe61cf730f9211e5d)

* was passiert dabei?

    * in der ersten Zeile wird eine asynchrone (`async`) Funktion (`function`) mit dem Namen `loadSites` definiert, die als Parameter eine Internetadresse (`url`) erwartet

    * der Codeblock der Funktion wird durch geschwungene Klammern (`{}`) abgegrenzt

    * in der Funktion wird im ersten Schritt auf die Antwort (`response`) des Servers gewartet (`wait`) die er beim Aufruf (`fetch`) der Internetadresse (`url`) zurückliefert

    * ist die Antwort angekommen, kann im zweiten Schritt das GeoJSON-Objekt der Antwort (`response.json`) in der Variablen `geojson` gespeichert werden. Auch das kann dauern, deshalb warten wir wieder, bis es soweit ist (`wait`)

    * **F12** zeigt uns schließlich in der Konsole das geladenen GeoJSON-Objekt

    * **Wichtig**: die Funktion wird natürlich nur ausgeführt, wenn wir sie aufrufen (`loadSites("https://...")` und dabei die Internetadresse des gewünschten GeoJSON Files übergeben - diese "landet" in der Funktion dann als Variable `url`

### c) GeoJSON-Daten mit L.geoJSON() visualisieren

Zum Visualisieren benötigen wir noch eine Zeile Code. Wir verwenden die Leaflet Methode [L.geoJSON](https://leafletjs.com/reference.html#geojson), die wir in der [Leaflet Dokumentation](https://leafletjs.com/reference.html) unter **Other Layers / GeoJSON** finden. Im Abschnitt **Factory** sehen wir, dass wir das GeoJSON-Objekt als ersten Parameter übergeben können, die Optionen brauchen wir vorerst nicht. Wie schon bei früheren Layern, verwenden wir `.addTo(map)` um den GeoJSON-Layer an die Karte zu hängen.

```javascript
async function loadSites(url) {
    // ...
    L.geoJSON(geojson).addTo(map);
}
```

[🔗 COMMIT](https://github.com/webmapping/wien/commit/d3f2b70585b285efc70fa509c4569032d09127c5)

Die Marker für die Sehenswürdigkeiten sind damit auf der Karte sichtbar

### d) den GeoJSON-Layer in ein Overlay schreiben

Noch eleganter ist es, den GeoJSON-Layer in ein eigenes Overlay zu schreiben. Vor dem `L.geoJSON`-Aufruf definieren wir das Overlay als [L.featureGroup](https://leafletjs.com/reference.html#featuregroup), fügen es zur Karte hinzu und hängen es mit einem Label in die Layercontrol ein

```javascript
let overlay = L.featureGroup();
layerControl.addOverlay(overlay, "Sehenswürdigkeiten");
overlay.addTo(map);
```

Den `L.geoJSON()` Layer müssen wir jetzt nur noch in das Overlay und nicht in die Karte direkt schreiben

```javascript
L.geoJSON(geojson).addTo(overlay);
```

[🔗 COMMIT](https://github.com/webmapping/wien/commit/7f12bbdd0676023f168c3d62e66dcdb227a8ae9a)

Damit kann der Sehenswürdigkeiten Layer ein- und ausgeschaltet werden

## Übungsaufgabe: wir erstellen 4 weitere Layer

Über die Suche bei [DigitalesWien](https://digitales.wien.gv.at/open-data/) finden wir Layer und implementieren sie mit den jeweiligen Vorgaben

1. Funktion `loadStops`
    * Suche: *sightseeing* 
    * Datensatz *Touristische Kraftfahrlinien Haltestellen Vienna Sightseeing Linie Standorte Wien*
    * Overlay Label "Haltestellen Vienna Sightseeing"

    * [🔗 COMMIT](https://github.com/webmapping/wien/commit/1817ace1bb115bf27858867717b8fb69100a011e)

2. Funktion `loadLines`
    * Suche: *sightseeing*
    * Datensatz *Touristische Kraftfahrlinien Liniennetz Vienna Sightseeing Linie Wien*
    * Overlay Label "Liniennetz Vienna Sightseeing"

    * [🔗 COMMIT](https://github.com/webmapping/wien/commit/58050e63908e0235642a991116e8d80dce71e454)

3. Funktion `loadZones`
    * Suche: *Fußgängerzonen*
    * Datensatz *Fußgängerzonen Wien*
    * Overlay Label "Fußgängerzonen"

    * [🔗 COMMIT](https://github.com/webmapping/wien/commit/7eacbcde4b3ba3f2b273f3cefb4735207d0be82b)

4. Funktion `loadHotels`
    * Suche: *Hotels*
    * Datensatz *Hotels und Unterkünfte Standorte Wien*
    * Overlay Label "Hotels und Unterkünfte"

    * [🔗 COMMIT](https://github.com/webmapping/wien/commit/6cfb837a8f68dbf43214cb6a9d961069ad19c9b4)


## Layer Sehenswürdigkeiten verfeinern

### a) Popup hinzufügen

* zur besseren Übersicht deaktivieren wir die vier anderen Layer indem wir den Aufruf der jeweiligen Funktion auskommentieren und die Funktion im VS Code einklappen

    [🔗 COMMIT](https://github.com/webmapping/wien/commit/bff2d9deee04e7c97c2fe1baa31f1b7f746c6b9a)


* wir könnten, wie in der Leaflet Dokumentation von [L.geoJSON](https://leafletjs.com/reference.html#geojson) im *Usage example* gezeigt, das Popup einfach vor dem `.addTo(overview)` hinzufügen, das machen wir aber nicht, denn in einem späteren Schritt würde uns das nur Probleme bereiten. Deshalb verwenden wir das Options-Objekt und werden bei [pointToLayer](https://leafletjs.com/reference.html#geojson-pointtolayer) fündig. Die *Description* zeigt uns ein passendes Beispiel, wie wir den Marker besser kontrollieren können.

    ```javascript
    L.geoJSON(geojson, {
        pointToLayer: function(geoJsonPoint, latlng) {
            return L.marker(latlng);
        }
    }).addTo(overlay);;
    ```

    [🔗 COMMIT](https://github.com/webmapping/wien/commit/3ecbb36ad365048ab2a01ce484d6feecc864beb1)

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

    [🔗 COMMIT](https://github.com/webmapping/wien/commit/192c2bb193dced58e003575f267cb815f2b8e4c3)

* damit ist das Popup fertig, als nächstes verwenden wir ein eigenes Icon an Stelle der Marker

### b) ein eigenes Icon statt dem Marker verwenden

* als Quelle für unser Icon verwenden wir die [Map Icons Collection - 1000+ free & customizable icons for maps](https://mapicons.mapsmarker.com/)

    * Suche *photo* -> Photo wählen

    * wir versuchen gleich das *customizable* Feature und wählen Weiß (`#ffffff`) als Farbe beim Farbwähler und klicken auf `Generate`

    * die Seite wird neu geladen, die Icons entsprechend angepasst und wir finden unsere Farbe in der URL <https://mapicons.mapsmarker.com/markers/media/photo/?custom_color=ffffff> wieder - dieses Feature werden wir später bei den Icons für die Bushaltestellen noch brauchen ...

    * wir wählen das linke Icon und speichern es im `icons/`-Verzeichnis als `photo.png`

    [🔗 COMMIT](https://github.com/webmapping/wien/commit/3b23c65190f2b901f12d73b1d76466c55a05616d)

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

    [🔗 COMMIT](https://github.com/webmapping/wien/commit/850d71e3d85d0d730e6cb9854cf03aa0f534f5d4)


## Layer Bushaltestellen verbessern

Bevor wir weiterarbeiten, setzen wir den Zoomfaktor beim Laden der Karte auf 16 und Blicken damit auf die Wiener Innenstadt

[🔗 COMMIT](https://github.com/webmapping/wien/commit/5f7d2982fae965831eabfc46c0f853d1fa6e84c6)

* als Quelle für das Bus Icon verwenden wir wieder die [Map Icons Collection - 1000+ free & customizable icons for maps](https://mapicons.mapsmarker.com/), suchen nach *photo*, stellen `#ffffff` als Farbe ein und speichern uns das neu generierte Icon als `bus.png` in dass Verzeichns `icons`

    [🔗 COMMIT](https://github.com/webmapping/wien/commit/6080f910af7b83cef98826ea88c534ce8d88d919)

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

[🔗 COMMIT](https://github.com/webmapping/wien/commit/47c965d8991fa746f90a4584ff13642f0f449238)

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


[🔗 COMMIT](https://github.com/webmapping/wien/commit/9eab797a2cb853b6db3c61cbd0c89bf502ad325b)

Damit bleibt noch, die Icons je nach `LINE_ID` zu verwenden, wir bedienen uns wieder der *Template-Syntax* mit *Backticks* bei der `iconUrl` und erzeugen Links zu den Icons `bus_1.png` bis `bus_6.png`. Die Zahl, und damit die Farbe, folgt dem Attribut `LINE_ID`

```javascript
icon: L.icon({
    iconUrl: `icons/bus_${geoJsonPoint.properties.LINE_ID}.png`,
    // ...
})
```

[🔗 COMMIT](https://github.com/webmapping/wien/commit/b88d64f0af82c119d27d360e03a1cc65b61ae52b)

