# Wien Beispiel HOWTO (Teil 4) - Linien und Flächen verfeinern

## Layer Buslinien verfeinern

Bevor wir mit den Buslinen weiterarbeiten, müssen wir natürlich den Kommentar beim Aufruf `loadLines("https://...")` löschen, damit sie auch angezeigt werden.

[🔗 COMMIT](https://github.com/webmapping/wien/commit/08124c2dc3515f674ef845eb6307117a30fa2510)

Wir erinnen uns an das erste *Usage  example* der [L.geoJSON Dokumentation](https://leafletjs.com/reference.html#geojson) das alles beinhaltet, was wir für unsere Linien benötigen.

```javascript
L.geoJSON(data, {
    style: function (feature) {
        return {color: feature.properties.color};
    }
}).bindPopup(function (layer) {
    return layer.feature.properties.description;
}).addTo(map);
```

* in der [style Option](https://leafletjs.com/reference.html#geojson-style) des Optionenobjekts wird für jedes geoJSON-Objekt einen Funktion ausgeführt, deren Rückgabeobjekt das Aussehen der Linie bestimmt

* mit der [.bindPopup Methode](https://leafletjs.com/reference.html#geojson-bindpopup) wird für jedes geoJSON-Objekt einen Funktion ausgeführt, deren Rückgabeobjekt den Inhalt des Popups bestimmt.

Beginnen wir mit der leichteren Übung, dem Popup

### a) Popup für die Linien hinzufügen

Am Ende unseres `L.geoJSON()` Aufrufs in `loadLines` kopieren den `.bindPopup()` Code der Leafletdokumentation und ändern den Rückgabewert auf `layer.feature.properties.LINE_NAME`. In der Zeile davor, lassen wir uns noch das `layer.feature.properties` Objekt anzeigen - wir erkennen darin die Attribute unserer Linien sobald wir auf eine Linie klicken.

```javascript
L.geoJSON(geojson).bindPopup(function (layer) {
        console.log(layer.feature.properties);
        return layer.feature.properties.LINE_NAME;
    }).addTo(overlay);
```

Mit *Template-Syntax* und *Backticks* können wir das Popup verfeinern und die Stationen des Abschnitts mit angeben

```javascript
return `
    <h4>${layer.feature.properties.LINE_NAME}</h4>
    von: ${layer.feature.properties.FROM_NAME}
    <br>
    nach: ${layer.feature.properties.TO_NAME}
`;
```

[🔗 COMMIT](https://github.com/webmapping/wien/commit/147ddae108bbdcf8eace00576ac7bd7061cb7e09)

### b) Linienfarben unterscheiden

Zur Bestimmung der Linienfarbe dient die [style-Option](https://leafletjs.com/reference.html#geojson-style) des Optionenobjekts beim `L.geoJSON()` Aufruf. Wie bei den Popups, wird für jede Linie die angegebene Funktion ausgeführt und über den Rückgabewert festgelegt, wie die Linie aussehen soll. Die Leafletdokumentation zeigt uns bei den [Path options](https://leafletjs.com/reference.html#path-option), welche Stylingmöglichkeiten zur Verfügung stehen. Wir kopieren den `style`-Block der Dokumentation und geben die Linienfarbe *green* für alle Linien zurück.

```javascript
L.geoJSON(geojson, {
    style: function (feature) {
        return {
            color: "green"
        };
    }
})
```

Wir könnten wieder eine if-Abfrage zur Bestimmung der passenden Liniefarbe machen, **eleganter** ist es allerdings, wenn wir uns ein Javascript-Objekt mit *Key/Value* Paaren basteln, bei denen der *Key* dem Namen der Linie in `feature.properties.LINE_NAME` und der *Value* der gewünschten Farbe entspricht. Wir nennen dieses Objekt `colors`, schreiben es direkt vor die `return` Anweisung und so sieht es mit passenden Farben von <https://clrs.cc/> aus:

```javascript
let colors = {
    "Red Line": "#FF4136",
    "Yellow Line": "#FFDC00",
    "Blue Line": "#0074D9",
    "Green Line": "#2ECC40",
    "Grey Line": "#AAAAAA",
    "Orange Line": "#FF851B"
};
```

Jetzt müssen wir nur noch die Farbe je nach Liniennamen bei der Pfad Option [color](https://leafletjs.com/reference.html#path-color) einsetzen. Steht in `feature.properties.LINE_NAME` "Red Line", wird `colors["Red Line"]` verwendet, steht dort "Blue Line", wird `colors["Blue Line"]` verwendet usw.

```javascript
return {
    color: `${colors[feature.properties.LINE_NAME]}`
}
```

[🔗 COMMIT](https://github.com/webmapping/wien/commit/000ba7a20e1407c12df96898af49a8894ec08c30)

Wir könnten mit der Pfad Option [weight](https://leafletjs.com/reference.html#path-weight) auch die Strichstärke auf 4 Pixel statt 3 Pixel (dem Defaultwert) ändern, oder die Linien über die Pfad Option [dashArray](https://leafletjs.com/reference.html#path-dasharray) strichliert zeichnen

```javascript
return {
    color: colors[feature.properties.LINE_NAME],
    weight: 4,
    dashArray: [10, 6]
};
```

* [dashArray](https://leafletjs.com/reference.html#path-dasharray) funktioniert so, dass in einem Array das Muster definiert wird. Der erste Wert bedeutet dabei *zeichne einen Strich mit dieser Länge*, der zweite Wert bedeutet *zeichne eine Lücke mit dieser Länge*. Diese Arrays können auch mehr als zwei Werte haben - der dritte wären dann wieder *Strich* der vierte *Lücke* usw.

[🔗 COMMIT](https://github.com/webmapping/wien/commit/7813cd66810ff18f04652a633d360ce1ee41ad44)

## Layer Fußgängerzonen verfeinern

Wir löschen wieder den Kommentar beim `loadZones("https://...")` Aufruf und machen dann das Selbe wie bei den Linien

[🔗 COMMIT](https://github.com/webmapping/wien/commit/f066d4195c2feb214248ad6432a86097b33997d3)

Danach können wir den `.bindPopup()` Code der Linien einfach kopieren und ans Ende des `L.geoJSON()` Aufrufs schreiben. Natürlich müssen wir das Popup auch gleich anpassen - wir verwenden die Attribute `ADRESSE`, `ZEITRAUM` und `AUSN_TEXT`. Nachdem bei den Attributen `ZEITRAUM` und `AUSN_TEXT` oft `null`, also keine Angabe steht, verwenden wir den [**Logical OR (||)**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_OR) Operator, um in diesen Fällen eine leere Zeichenkette statt *null* anzuzeigen.

### a) Popup für die Flächen hinzufügen

```javascript
L.geoJSON(geojson).bindPopup(function (layer) {
    return `
        <h4>Fußgängerzone ${layer.feature.properties.ADRESSE}</h4>
        <p>${layer.feature.properties.ZEITRAUM || ""}</p>
        <p>${layer.feature.properties.AUSN_TEXT || ""}</p>
    `;
}).addTo(overlay);
```

[🔗 COMMIT](https://github.com/webmapping/wien/commit/16a46067176b43b5f25324b8b10328af7a5b4060)

### b) Flächen einfärben

Bei den Flächen ändern wir die Standardfarbe ([color](https://leafletjs.com/reference.html#path-color)) auf <https://clrs.cc/> *FUCHSIA (#F012BE)*, die Strichstärke  ([weight](https://leafletjs.com/reference.html#path-weight)) der Linie auf 1 und die Transparenz von Randlinie ([opacity](https://leafletjs.com/reference.html#path-opacity)) und Fläche ([fillOpacity](https://leafletjs.com/reference.html#path-fillopacity)) jeweils auf 0.1

```javascript
L.geoJSON(geojson, {
    style: function(feature) {
        return {
            color: "#F012BE",
            weight: 1,
            opacity: 0.1,
            fillOpacity: 0.1
        }
    }
})
```

[🔗 COMMIT](https://github.com/webmapping/wien/commit/27aef01d4606aec935af305cc388f5ab137fa9be)

