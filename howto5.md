# Wien Beispiel HOWTO (Teil 5) - Leaflet Markercluster

Auf der Leaflet [Plugins Seite](https://leafletjs.com/plugins.html) unter [Clustering/Decluttering](https://leafletjs.com/plugins.html#clusteringdecluttering) finden wir das [Leaflet.markercluster Plugin](https://github.com/Leaflet/Leaflet.markercluster) von *Dave Leaver*

Unter [Using the plugin](https://github.com/Leaflet/Leaflet.markercluster#using-the-plugin) sehen wir, dass wir zwei Stylesheets und ein Javascript benötigen:

* MarkerCluster.css
* MarkerCluster.Default.css
* leaflet.markercluster.js

Wir holen uns die Links zu den benötigten Dateien bei <https://cdnjs.com/libraries/leaflet.markercluster> und bauen sie mit *Copy Script Tag* im &lt;head> Bereich von `index.html` ein

```html
<!-- Leaflet markercluster -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.5.3/MarkerCluster.Default.css" integrity="sha512-6ZCLMiYwTeli2rVh3XAPxy3YoR5fVxGdH/pz+KMCzRY2M65Emgkw00Yqmhh8qLGeYQ3LbVZGdmOX9KUjSKr0TA==" crossorigin="anonymous" referrerpolicy="no-referrer" />
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.5.3/MarkerCluster.css" integrity="sha512-mQ77VzAakzdpWdgfL/lM1ksNy89uFgibRQANsNneSTMD/bj0Y/8+94XMwYhnbzx8eki2hrbPpDm0vD0CiT2lcg==" crossorigin="anonymous" referrerpolicy="no-referrer" />
<script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.5.3/leaflet.markercluster.js" integrity="sha512-OFs3W4DIZ5ZkrDhBFtsCP6JXtMEDGmhl0QPlmWYBJay40TT1n3gt2Xuw8Pf/iezgW9CdabjkNChRqozl/YADmg==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/64cb6e0a3d8cc46cdac4e67b9305b390f3226d7f)

Dann müssen wir nur noch beim Code für die Hotels bei der Definition des Overlays eine `markerClusterGroup` statt einer `featureGroup` verwenden

```javascript
let overlay = L.markerClusterGroup();
```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/1616492b9047f8e853f0779ea0efb6a79e61f44c)

**Voilà**, die Icons werden nun gruppiert. Bei den Kreisen der Gruppierung sieht man die Anzahl und Verbreitung der zusammengefassten Icons (als Fläche beim *mouseover*), beim Klick werden die Cluster so gut es geht aufgelöst.

Eine Änderung für bessere Usability können wir noch über das Optionen-Objekt implementieren: ab Zoomfaktor 17, das ist einmal einzoomen von der Startansicht im Zoomlevel 16, ist auf der Karte genügend Platz für die Hotel-Icons und deshalb **unterdrücken wir das Clustering**

```javascript
let overlay = L.markerClusterGroup(
    disableClusteringAtZoom: 17
);
```

[🔗 COMMIT](https://github.com/webmapping22s/wien/commit/f0751161e79889f4acdad4f901a34292bc58e3fa)
