# Wien Beispiel HOWTO (Teil 2) - GeoJSON asynchron laden und default Visualisierung

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


