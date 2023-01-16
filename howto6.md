# Wien Beispiel HOWTO (Teil 6) - Hotelsuche mit HTML Formular und Autocomplete

Bei 360 Hotels und Unterkünften ist eine Suche nach Hotelnamen sehr hilfreich. Wir implementieren sie mit den HTML Elementen &lt;form>, &lt;input> und &lt;datalist>, einem Event Handler beim Abschicken des Suchformulars und der Leaflet Methode `.eachLayer` zum Finden des gesuchten Hotels in der Hotelfunktion `loadHotels`

## 1. Das Suchformular

Wir beginnen mit dem HTML Code für das Formular. Es besteht aus einem Suchfeld, einem Suchbutton und einem Resetbutton. Wir schreiben den Markup in `index.html` direkt vor dem &lt;div> Element der Karte:

```html
<form id="searchForm">
    <input type="text" name="hotel" placeholder="Hotel suchen ...">
    <input type="button" name="suchen" value="Anzeigen">
    <input type="reset" value="Reset">
</form>
```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/0546ce77b93988e45aca4ce5ff453430fec1f9d1)

* nachdem wir später im Skript auf den Inhalt des Formulars zugreifen wollen, vergeben wir beim &lt;form> Element eine ID (`id="searchForm"`) - über sie werden wir Zugang zum Formular bekommen

* drei &lt;input> Element mit drei verschiedenen Typen komplettieren das Formular

    * `input type="text"` definiert das Suchfeld. Über `name="hotel"` geben wir ihm den Namen `hotel`, den wir später zum Auslesen des gesuchten Hotels verwenden können. Ein Platzhalter (`placeholder`) als Eingabehilfe wird angezeigt, solange das Suchfeld leer ist

    * `input type="button"` definiert den Button zum Abschicken des Formulars. Wir vergeben auch hier eine Namen (`suchen`), den wir später verwenden werden, um auf Klicks auf den Button zu reagieren.  Über `value` setzen wir den Label des Buttons auf *Anzeigen*

    * `input type="reset"` definiert den Button zum Zurücksetzen des Formulars. Er kann bestehenden Text im Textfeld löschen und den Platzhalter damit wieder anzeigen. Über `value` setzen wir den Label des Buttons auf *Reset*

## 2. Eine Autocomplete-Liste erstellen

Über das **&lt;datalist>** Element können wir die Autocomplete-Funktionalität vorbereiten. Dazu wird innerhalb des &lt;datalist> Elements für jeden Wert ein &lt;option> Element mit dem entsprechenden Wert als `value` Attribut definiert. Wir schreiben drei Einträge für die Texte *eins, zwei, drei*. Das &lt;datalist> Element bekommt zusätzlich eine `id="searchList"` die wir gleich noch brauchen werden.

```html
<datalist id="searchList">
    <option value="eins"></option>
    <option value="zwei"></option>
    <option value="drei"></option>
</datalist>
```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/53b0ea2dd00b5fa514d6407a0888f819f12fd3c7)

Wie wir beim Neuladen des Browsers erkennen, werden &lt;datalist> Elemente nicht direkt angezeigt. Damit wir sie beim Suchfeld als Vorschläge auswählen können, müssen wir über `list="searchList"` beim Suchfeld vermerken, dass die Datenliste mit der ID *searchList* für das Autocomplete herangezogen werden soll.

```html
<input type="text" name="hotel" placeholder="Hotel suchen ..." list="searchList">
```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/c3f4bba468dfa81808d302c0197689668cb144fe)

 Das fertige Formular mit Autoccomplete über die Suchliste sieht damit so aus:

```html
<form id="searchForm">
    <input type="text" name="hotel" placeholder="Hotel suchen ...">
    <input type="text" name="hotel" placeholder="Hotel suchen ..." list="searchList">
    <input type="button" name="suchen" value="Anzeigen">
    <input type="reset" value="Reset">
</form>
<datalist id="searchList">
    <option value="eins"></option>
    <option value="zwei"></option>
    <option value="drei"></option>
</datalist>
```

Mit wenigen Buchstaben können wir jetzt auf die passenden Einträge der Suchliste zugreifen.

## 3. Die Suchliste mit den Hotelnamen füllen

Unsere Suchvorschläge *eins, zwei,drei* helfen uns natürlich nicht bei der Hotelsuche, deshalb füllen wir das &lt;datalist> Element mit den tatsächlichen Hotelnamen. Der Platz an dem das passieren kann, ist in `main.js` in der `pointToLayer` Funktion. Dort wird ja für jedes Hotel der Inhalt der Funktion `pointToLayer` ausgeführt. Wir ergänzen deshalb ganz oben in der `pointToLayer` Funktion den folgenden Code:

```javascript
let searchList = document.querySelector("#searchList");
searchList.innerHTML += `<option value="${geoJsonPoint.properties.BETRIEB}"></option>`;
```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/7a9d70570ef73d7c39411dbf4e1872ff64143ed5)

* `document.querySelector()` kennen wir schon vom Neuseelandbeispiel. Es findet über den CSS-Selektor `#searchList` das Element mit der ID `searchList` - in unserem Fall also das &lt;datalist> Element und speichert es in der Varaiablen `searchList`. Über `searchList.innerHTML` können wir dann den Quellcode des &lt;datalist> Elements verändern. Wir verwenden `+=` um Markup an den bestehenden Inhalt anzuhängen.

* für jedes Hotel wird dann ein &lt;option> Element mit dessen Namen in `geoJsonPoint.properties.BETRIEB` als `value` angehängt

Das Resultat können wir zwar wieder nicht direkt sehen, aber im Suchfeld testen - alle Hotels sind jetzt über Autocomplete mit wenigen Buchstaben wählbar.

## 4. Hotel suchen, anzeigen und Popup ausklappen

Der Platz an dem das alles passiert ist am Ende von `loadHotels`. Damit wir dort mit dem Formular kommunizieren können, müssen wir uns zuerst mit `document.querySelector` eine Referenz zum Formular mit der ID `searchForm` in einer Variablen `form` speichern.

```javascript
let form = document.querySelector("#searchForm");
```

Danach können wir bei der Variablen `form` einen sogenannten **Event-Listener** definieren, dessen Funktion ausgeführt wird, wenn wir auf den Anzeigen-Button `form.suchen` klicken (`onclick`). In dieser Funktion lassen wir uns den aktuellen Wert des Suchfelds in `form.hotel.value` mit `console.log` anzeigen.

```javascript
form.suchen.onclick = function() {
    console.log(form.hotel.value);
}
```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/fb8e5fc23091db7a4f97d4d18e71744a7b47f1f4)

In der Variablen `form.hotel.value` wissen wir jetzt also, welches Hotel in der Suche ausgewählt wurde. Wie finden wir den dazugehörigen Marker mit Popup in unserer Karte? Dazu müssen wir alle gezeichneten Marker im GeoJSON Layer durchgehen und deren Namen mit dem gewünschten Namen vergleichen.

Für dieses *Durchgehen*, stellt Leaflet die Methode [.eachLayer](https://leafletjs.com/reference.html#geojson-eachlayer) zur Verfügung. Wir können sie anwenden, sobald der `L.geoJSON()` Aufruf in einer Variablen gespeichert ist. Wir ergänzen also vor dem `L.geoJSON()` Aufruf die Deklaration für die Variablen `hotelsLayer`

```javascript
let hotelsLayer = L.geoJSON(geojson, {
    // bestehender Code zum Zeichnen der Marker
})
```

In der `form.suchen.onclick` Funktion können wir dann `hotelsLayer.eachLayer()` einsetzen, um jeden Marker zu untersuchen. Wir lassen uns den marker, die Koordinate über [marker.getLatLng()](https://leafletjs.com/reference.html#marker-getlatlng), das Popup über [marker.getPopup()](https://leafletjs.com/reference.html#marker-getpopup) und den Namen über `marker.feature.properties.BETRIEB` mit `console.log` anzeigen.

```javascript
form.suchen.onclick = function() {
    console.log(form.hotel.value);
    hotelsLayer.eachLayer(function(marker) {
        console.log(marker)
        console.log(marker.getLatLng())
        console.log(marker.getPopup())
        console.log(marker.feature.properties.BETRIEB)
    })
}
```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/1d8aa14ba27f20f68951159314d1a6ceb33a95ed)

In einer `if-Abfrage` überprüfen wir schließlich noch, ob der aktuelle Name des Hotels in `marker.feature.properties.BETRIEB` dem gesuchten Namen in `form.hotels.value` entspricht. Sind beide gleich, positionieren wir die Karte mit [map.setView()](https://leafletjs.com/reference.html#map-setview) auf die Koordinate des Icons. Als Zoomlevel wählen wir dabei 17, denn in diesem Level sind alle Markercluster aufgelöst. Mit [marker.openPopup()](https://leafletjs.com/reference.html#marker-openpopup) öffnen wir noch das Popup. Die `console.log` Zeilen kommentieren wir aus, oder löschen sie ganz.

```javascript
// Hotel suchen und anzeigen
let form = document.querySelector("#searchForm");
form.suchen.onclick = function() {
    hotelsLayer.eachLayer(function (marker) {
        if (form.hotel.value == marker.feature.properties.BETRIEB) {
            map.setView(marker.getLatLng(), 17);
            marker.openPopup();
        }
    });
}
```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/f4efa8539c3d1e258e6cb3ba74c215ddb7a39fa8)

Unsere Testeintäge *eins, zwei, drei* im &lt;datalist> Element können wir auch noch löschen

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/56d21d8fa9968bab2cdaba9c3e6778b0adf6ea79)

## 5. Leider ist die Suchliste nicht sortiert :-(

Leider sind die Einträge der Suchliste nicht nach dem Attribut `BETRIEB` sortiert. Deshalb sortieren wir die Einträge im `geojson.features` Array einfach selber. Wie geht das? Gar nicht so einfach, aber hier ist die Lösung ;-)

```javascript
geojson.features.sort(function(a, b) {
    return a.properties.BETRIEB > b.properties.BETRIEB;
});
```

Details dazu hat die [MDN Hilfe](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)

Noch besser ist es, die Namen gleich *case insensitive* zu vergleichen. Die Javascript String Methode [.toLowerCase()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/toLowerCase) erlaubt das und unsere fertige Sortierfunktion sieht damit so aus:

```javascript
geojson.features.sort(function(a, b) {
    return a.properties.BETRIEB.toLowerCase() > b.properties.BETRIEB.toLowerCase()
})
```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/5d856433cf6eda6c86bbf103fcd283f75a1c2c5a)