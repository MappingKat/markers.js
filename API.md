## The Markers Layer

`mapbox.markers()` is a markers library that makes it easier to add HTML elements
on top of maps in geographical locations and interact with them. Internally,
markers are stored as [GeoJSON](http://www.geojson.org/) objects, though
interfaces through CSV and simple Javascript are provided.

<a href='#wiki-markers'>#</a> mapbox.<b>markers</b>()

`mapbox.markers()` is the singular entry point to this library - it creates a
new layer into which markers can be placed and which can be added to
a Modest Maps map with `.addLayer()`

<a href="#wiki-markers_factory">#</a> markers.<b>factory</b>([<i>value</i>])

Defines a new factory function, and if the layer already has points added to it,
re-renders them with the new factory. Factory functions are what turn GeoJSON feature
objects into HTML elements on the map.

The argument should be a function that takes a
[GeoJSON feature object](http://geojson.org/geojson-spec.html#feature-objects)
and returns an HTML element.

Due to the way that `markers.js` allows multiple layers of interactivity, factories
that want their elements to be interactive **must** either set `.style.pointerEvents = 'all'` on
them via Javascript, or have an equivalent CSS rule with `pointer-events: all` that affects
the elements.

If value is not specified, returns the current factory function.

<a href="#wiki-markers_features">#</a> markers.<b>features</b>([<i>value</i>])

This is the central function for setting the contents of a markers layer: it runs the provided
features through the filter function and then through the factory function to create elements
for the map. If the layer already has features, they are replaced with the new features.

The argument should be
a array of [GeoJSON feature objects](http://geojson.org/geojson-spec.html#feature-objects).
An empty array will clear the layer of all features

If the value is not specified, returns the current array of features.

<a href="#wiki-markers_sort">#</a> markers.<b>sort</b>([<i>value</i>])

Markers are typically sorted in the DOM in order for them to correctly visually overlap. By default,
this is a function that sorts markers by latitude value - `geometry.coordinates[1]`.

The argument should be a function that takes two GeoJSON feature objects and returns a number indicating
sorting direction - fulfilling the [Array.sort](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Array/sort)
interface.

If the value is not specified, returns the current function used to sort.

The default sorting function is:

```javascript
function(a, b) {
    return b.geometry.coordinates[1] -
      a.geometry.coordinates[1];
}
```

<a href='#wiki-markers_filter'>#</a> markers.<b>filter</b>([<i>value</i>])

Markers can also be filtered before appearing on the map. This is a purely presentational filter -
the underlying features, which are accessed by `.features()`, are unaffected. Setting a new
filter can therefore cause points to be displayed which were previously hidden.

The argument should be a functoin that takes a GeoJSON feature object and returns `true`
to allow it to be displayed or `false` to hide it.

If the value is not specified, returns the current function used to filter features.

The default filter function is:

```javascript
function() { return true; }
```

<a href='#wiki-markers_url'>#</a> markers.<b>url</b>(<i>url</i> [<i>, callback</i>])

This provides another way of adding features to a map - by loading them from a remote GeoJSON file.

The first argument should be a URL to a GeoJSON file on a server. If the server is remote, the
GeoJSON file must be served with a `.geojsonp` extension and respond to the JSONP callback `grid`.

The second argument is optional and should be a callback that is called after the request finishes,
with the features array (if any) and the layer instance as arguments.

<a href='#wiki-markers_extent'>#</a> markers.<b>extent</b>()

Return the extent of all of the features provided.
Returns an array of two `{ lat: 23, lon: 32 }` objects compatible with
Modest Maps's `extent()` call. If there are no features, the extent is set to
`Infinity` in all directions, and if there is one feature, the extent is set
to its point exactly.

<a href='#wiki-markers_addCallback'>#</a> markers.<b>addCallback</b>(<i>event, callback</i>)`

Adds a callback that is called on certain events by this layer. These are primarily used by `mmg_interaction`, but otherwise useful to support more advanced bindings on mmg layers that are bound at times when the mmg object may not be added to a map - like binding to the map's `panned` event to clear tooltips.

Event should be a String which is one of the following:

* `drawn`: whenever markers are drawn - which includes any map movement
* `markeradded`: when markers are added anew

Callback is a Function that is called with arguments depending on what `event` is bound:

* `drawn`: the layer object
* `markeradded`: the new marker

<a href='#wiki-markers_removeCallback'>#</a> markers.<b>removeCallback</b>(<i>event, callback</i>)`

This removes a callback bound by `.addCallback(event, callback)`.

The `callback` argument must be the same Function as was given in `addCallback`. The `event` argument must be the same String that was given in `addCallback`

## Interacting with Markers

Classic interaction, hovering and/or clicking markers and seeing their details,
is supported by `marker_interaction` and customizable through its methods.
This supports both mouse & touch input.

<a href='#wiki-marker_interaction'>#</a> mapbox.<b>marker_interaction</b>(<i>value</i>)`

Adds tooltips to your markers, for when a user hovers over or taps the features.

The single argument must be a markers layer. This returns an `interaction` instance which provides methods for customizing how the layer behaves.

<a href='#wiki-interaction_hide_on_move'>#</a> interaction.<b>hide_on_move</b>([<i>value</i>])`

Determines whether tooltips are hidden when the map is moved. The single argument should be `true` or `false` or not given in order to retrieve the current value.

<a href='#wiki-interaction_exclusive'>#</a> interaction.<b>exclusive</b>([<i>value</i>])

Determines whether a single popup should be open at a time, or unlimited. The single argument should be `true` or `false` or not given in order to retrieve the current value.

<a href='#wiki-interaction_formatter'>#</a> interaction.<b>formatter</b>([<i>value</i>])

Set or get the formatter function, that decides how data goes from being in a feature's `properties` to the HTML inside of a tooltip. This is a getter setter that takes a Function as its argument.

The default formatter is:

```javascript
function(feature) {
    var o = '', props = feature.properties;
    if (props.title) {
        o += '<h1 class="mmg-title">' + props.title + '</h1>';
    }
    if (props.description) {
        o += '<div class="mmg-description">' + props.description + '</div>';
    }
    if (typeof html_sanitize !== undefined) {
        o = html_sanitize(o,
            function(url) {
                if (/^(https?:\/\/|data:image)/.test(url)) return url;
            },
            function(x) { return x; });
    }
    return o;
}
```

## CSV Support

[GeoJSON](http://www.geojson.org/) is a simple and convenient format, but requires
familiarity with [JSON](http://www.json.org/) to master. [CSV](http://en.wikipedia.org/wiki/Comma-separated_values) is also supported by converters which attempt to find the geodata -
the columns with coordinates - in CSV files, and convert them into GeoJSON objects
that are usable by the markers API.

<a href='#wiki-mapbox_csv_to_geojson'>#</a> mapbox.<b>csv_to_geojson</b>(<i>value</i>)

[CSV](http://en.wikipedia.org/wiki/Comma-separated_values) support is provided by a translation layer that reads CSV files, attempts to pick up latitude and longitude columns (by any columns named with `lat` or `lon` at the beginning, in upper or lower case), and converting these into an array of GeoJSON features which you can easily use with `.feature()`.

Takes a string of CSV text and returns an array of CSV features. This will _throw_ if it cannot find a latitude & longitude field, and return `[]` if given a blank string.

<a href='#wiki-mapbox_csv_url_to_geojson'>#</a> mapbox.<b>csv_to_geojson</b>(<i>url, callback</i>)

While `csv_to_geojson` expects a string of CSV data, `csv_url_to_geojson` can take a URL to a CSV file - which must be locally hosted due to [cross domain restrictions](http://en.wikipedia.org/wiki/Same_origin_policy)

The first argument must be a URL to the CSV file, while the second argument is a Function that receives the array of GeoJSON features as its first argument.
