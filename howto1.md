# Wien Beispiel HOWTO (Teil 1) - Grundkarte

## Repo "wien" erstellen und online bringen

* Wir loggen uns bei <https://github.com/username> ein und dann: *Create a new repository* / "wien"

* danach clonen wir <https://github.com/username/wien.git> in VS Code lokal

    * F1 -> Git: Clone
    * Desktop -> Select Repository Location -> Open in New Window

* [Template](https://webmapping22s.github.io/templates/wien.zip) auspacken, Inhalte ins `wien`-Verzeichnis verschieben, alles hinzufügen und committen

    [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/7b448eaa3dfdc5f5d553ca126c65a7c035e6d2a0)

    * was ist neu im Template? In `main.css` verwenden wir *display: flex*, um die drei Bereiche für das Wappen, die Überschrift und die Links gleichmäßig über die Breite zu verteilen.

        ```css
        header {
            display: flex;
            justify-content: space-between;
        }

    * Mit CSS-Flexbox kann man noch viel mehr am Layout feilen - die [MDN Docs Flexbox](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox) geben einen ersten Überblick, was alles möglich ist.


* Push zurück auf <https://github.com/username/wien>

    * bei den drei Punkten oder über F1 -> Git: Push
    * zustimmen, dass *oath* verwendet wird
    * <https://github.com/username/wien> neu laden - wunderbar, unsere Seite ist bei github angekommen

* online verfügbar machen - der 2. Weg nach einem Repo **username.github.io**, das dann automatisch unter https://usernmane.github.io erreichbar ist

    * Beim wien-Repo: Settings -> Pages ->  Source -> "main" wählen und "Save"

    * Your site is ready to be published at <https://username.github.io/wien/>

    * warten, es wird irgendwann verfügbar sein ;-)

    * in der Zwischenzeit bauen wir an der Seite weiter

## Grundkarte erstellen

### a) Layer control über Leaflet.providers Plugin implementieren und Karte initialisieren

* bevor wir `Leaflet.providers` verwenden, bereiten wir den Blick auf den Stephansdom aus deren Wikipedia-Koordinate vor

    ```javascript
    let stephansdom = {
        lat: 48.208493,
        lng: 16.373118,
        title: "Stephansdom"
    };
    ```

    [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/07a31dcc2a8e915ea6ce809d09b46291661e069f)

* Startlayer über Leaflet providers Plugin definieren und die Karte initialisieren

    * Quelle: [Leaflet.providers Plugin](https://github.com/leaflet-extras/leaflet-providers)

    * [Demoapplikation](http://leaflet-extras.github.io/leaflet-providers/preview/index.html) für alle verfügbaren WMTS-Hintergründe des Plugins - wir werden *BasemapAT.grau* verwenden

    * in `index.html`: das Javascript von <https://cdnjs.com/libraries/leaflet-providers> einbinden

        ```html
        <!-- Leaflet providers plugin -->
        <script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet-providers/1.13.0/leaflet-providers.min.js" integrity="sha512-5EYsvqNbFZ8HX60keFbe56Wr0Mq5J1RrA0KdVcfGDhnjnzIRsDrT/S3cxdzpVN2NGxAB9omgqnlh4/06TvWCMw==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
        ```

    * in `main.js`: Startlayer mit *BasemapAT.grau* definieren, die Karte initialisieren, auf den Stephansdom blicken und dabei den Startlayer verwenden.

        ```javascript
        let startLayer = L.tileLayer.provider("BasemapAT.grau");

        let map = L.map("map", {
            center: [ stephansdom.lat, stephansdom.lng ],
            zoom: 12,
            layers: [
                startLayer
            ],
        });
        ```

    [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/700b9b7cbe2f9facfd002f99183bcca13266ca43)


    * Layernavigation über [L.control.layers](https://leafletjs.com/reference.html#control-layers) mit dem Startlayer hinzufügen - wir verwenden `startLayer` als ersten Eintrag wieder, damit das Umschalten gleich funktioniert.

        ```javascript
        let layerControl = L.control.layers({
            "BasemapAT Grau": startLayer,
        }).addTo(map);
        ```

    * alle verfügbaren *basemap.at* Layer ergänzen

        * für einzelne Layer verwenden wir `L.tileLayer.provider`

            ```javascript
            let layerControl = L.control.layers({
                "BasemapAT Grau": startLayer,
                "Basemap Standard": L.tileLayer.provider("BasemapAT.basemap"),
                "Basemap High-DPI": L.tileLayer.provider("BasemapAT.highdpi"),
                "Basemap Gelände": L.tileLayer.provider("BasemapAT.terrain"),
                "Basemap Oberfläche": L.tileLayer.provider("BasemapAT.surface"),
                "Basemap Orthofoto": L.tileLayer.provider("BasemapAT.orthofoto"),
                "Basemap Beschriftung": L.tileLayer.provider("BasemapAT.overlay"),
            }).addTo(map);
            ```

        * für den kombinierten Layer beim Orthofoto mit Beschriftung nützen wir  [L.layerGroup](https://leafletjs.com/reference.html#layergroup) zur Gruppierung der beiden `L.tileLayer.provider` Layer als Array

            ```javascript
            let layerControl = L.control.layers({
                // bestehende Layer ...
                "Basemap mit Orthofoto und Beschriftung": L.layerGroup([
                    L.tileLayer.provider("BasemapAT.orthofoto"),
                    L.tileLayer.provider("BasemapAT.overlay"),
                ])
            }).addTo(map);
            ```
        
        * zusätzlich klappen wir die Layer control gleich aus

            ```javascript
            layerControl.expand();
            ```

        [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/77a23452ac7c7ca442cc3c8d721132780b8c7ec0)


### b) Overlay Layer für den Stephansdom (ein-/ausschaltbar) über L.featureGroup implementieren

* Overlay für den Stephansdom vorbereiten

    * zuerst in der Variablen `sightLayer` eine [L.featureGroup](https://leafletjs.com/reference.html#featuregroup) definieren

        ```javascript
        let sightLayer = L.featureGroup();
        ```

    * dann hängen wir diese *featureGroup* mit einem Label in die Layer control über [.addOverlay](https://leafletjs.com/reference.html#control-layers-addoverlay) ein

        ```javascript
        layerControl.addOverlay(sightLayer, "Sehenswürdigkeiten");
        ```

    [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/adaa7b0ab211af007d3a5e4e065180a3b7c647f3)

* Marker für Stephansdom hinzufügen: der Marker wird jetzt nicht an die `map` sondern an das Overlay `sightLayer`angehängt

    ```javascript
    let mrk = L.marker([ stephansdom.lat, stephansdom.lng]).addTo(sightLayer);
    ```

    [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/6837847905527138fc66080e54edca8901c130bf)

* Overlay an die Karte hängen

    ```javascript
    sightLayer.addTo(map);
    ```

    [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/9fd7c1a1addcf1a81fe12d4ee1f4e3f63e2f9e39)


### c) Maßstab hinzufügen über L.control.scale

* [L.control.scale](https://leafletjs.com/reference.html#control-scale) platziert einen Maßstab im linken, unteren Eck

    ```javascript
    L.control.scale().addTo(map);
    ```

    [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/61c6afc990a0eddf374bc9bda9b30956d4292bb3)

* per Default werden sowohl Meter als auch Meilen angezeigt. Wenn wir nur Meter wollen, hilft uns das *Options-Objekt* beim Aufruf über die Boolean Option *imperial* weiter

    ```javascript
    L.control.scale({
        imperial: false,
    }).addTo(map);
    ```

    [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/e25791c3f71ed73eef5c961f42d8e6c9287237a5)


### d) Fullscreen control über Leaflet.fullscreen Plugin implementieren

* Quelle: [Leaflet.fullscreen Plugin](https://github.com/Leaflet/Leaflet.fullscreen)

* in `index.html`: Javascript und CSS-Stile von <https://github.com/Leaflet/Leaflet.fullscreen> einbinden (siehe *Including via CDN*)

    ```html
    <!-- Leaflet fullscreen plugin -->
    <script src='https://api.mapbox.com/mapbox.js/plugins/leaflet-fullscreen/v1.0.1/Leaflet.fullscreen.min.js'></script>
    <link href='https://api.mapbox.com/mapbox.js/plugins/leaflet-fullscreen/v1.0.1/leaflet.fullscreen.css' rel='stylesheet' />
    ```

* in `main.js`, am Ende des Skripts

    ```javascript
    L.control.fullscreen().addTo(map);
    ```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/00ad605d6cc122cb1ad4f6c723802a4504f2e13e)


### e) Minimap control über Leaflet.MiniMap Plugin implementieren

* Quelle: [Leaflet.MiniMap Plugin](https://github.com/Norkart/Leaflet-MiniMap) ([Demo](https://norkart.github.io/Leaflet-MiniMap/example.html))

* in `index.html`: Javascript und CSS-Stile von <https://cdnjs.com/libraries/leaflet-minimap> einbinden

    ```html
    <!-- Leaflet MiniMap -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet-minimap/3.6.1/Control.MiniMap.min.js" integrity="sha512-WL3nAJlWFKoDShduxQRyY3wkBnQsINXdIfWIW48ZaPgYz1wYNnxIwFMMgigzSgjNBC2WWZ8Y8/sSyUU6abuF0g==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet-minimap/3.6.1/Control.MiniMap.min.css" integrity="sha512-qm+jY0iQ4Xf5RL79UB75REDLYD0jtvxxVZp2RVIW8sm8RNiHdeN43oksqUPrBIshJtQcVPrAL08ML2Db8fFZiA==" crossorigin="anonymous" referrerpolicy="no-referrer" />
    ```

* in `main.js`:

    * in der [Github-Anleitung](https://github.com/Norkart/Leaflet-MiniMap) finden wir unter *Using the MiniMap control* ein Beispiel, wie wir die MiniMap einbinden können. Statt dem `osm2` Layer verwenden wir einen neuen Tilelayer unsers *Leaflet.providers Plugins*. **Wichtig**: auch wenn es verlockend ist, den bereits definierten Startlayer zu verwenden, muss eine **neuer** Layer verwendet werden, sonst funktioniert die MiniMap nicht richtig.

        ```javascript
        let miniMap = new L.Control.MiniMap(
            L.tileLayer.provider("BasemapAT")
        ).addTo(map);
        ```

        [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/87bc556aac44920828d4d9caa714d52c1eb31797)

    * im wahrsten Sinne "neu" ist das **new** vor `L.Control.MiniMap` - es bewirkt, dass ein neues MiniMap-Objekt erzeugt wird und ist gleich zu setzen mit einem Aufruf `L.control.minimap()` **ohne** **new**. Grundsätzlich sollten alle Leaflet Plugins auf diese zwei Arten verwendbar sein, die meisten verwenden allerding die Form ohne *new* mit Kleinschreibung der Methode. Wir könnten also unsere bisherigen Plugins jeweils in zwei Arten verwenden:
    
        ```javascript
        L.control.fullscreen() == new L.Control.Fullscreen()
        L.control.minimap() == new  L.Control.MiniMap()
        ```

    * über das Options-Objekt von `L.Control.MiniMap` können wir die Übersichtskarte noch ein-/ausklappbar machen

        ```javascript
        let miniMap = new L.Control.MiniMap(
            L.tileLayer.provider("BasemapAT"), {
                toggleDisplay: true
            }
        ).addTo(map);
        ```

        [🔗 COMMIT](https://github.com/webmapping22s/wien/commit/5d86a38d5b8017a100c660fe2cae09b5a560b80c)