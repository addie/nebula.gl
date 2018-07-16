# EditableGeoJsonLayer

The Editable GeoJSON layer accepts a [GeoJSON](http://geojson.org) `FeatureCollection` and renders the features as editable polygons, lines, and points.

```js
import DeckGL from 'deck.gl';
import { EditableGeoJsonLayer } from 'nebula.gl';

const myFeatureCollection = {
  type: 'FeatureCollection',
  features: [/* insert features here */]
};

class App extends React.Component {
  state = {
    mode: 'modify',
    selectedFeatureIndex: 0,
    data: myFeatureCollection
  };

  render() {
    const layer = new EditableGeoJsonLayer({
      id: 'geojson-layer',
      data: this.state.data,
      mode: this.state.mode,
      selectedFeatureIndex: this.state.selectedFeatureIndex,

      onEdit: ({
        updatedData,
        updatedMode,
        updatedSelectedFeatureIndex
      }) => {
        this.setState({
          data: updatedData,
          mode: updatedMode,
          selectedFeatureIndex: updatedSelectedFeatureIndex
        });
      }
    });

    return (<DeckGL {...this.props.viewport} layers={[layer]} />);
  }
}
```

## Properties

Inherits all [deck.gl's Base Layer](https://uber.github.io/deck.gl/#/documentation/deckgl-api-reference/layers/layer) properties.

#### `data` (Object, optional)

* Default: `null`

A [GeoJSON](http://geojson.org) `FeatureCollection` object. The following types of geometry are supported:

* `Point`
* `LineString`
* `Polygon`
* `MultiPoint`
* `MultiLineString`
* `MultiPolygon`
* `GeometryCollection` is not supported.

*Note: passing a single `Feature` is not supported. However, you can pass a `FeatureCollection` containing a single `Feature` and pass `selectedFeatureIndex: 0` to achieve the same result.*

#### `mode` (String, optional)

* Default: `modify`

The `mode` property dictates what type of edits the user can perform and how to handle user interaction events (e.g. pointer events) in order to accomplish those edits.

* `view`: no edits are possible, but selection is still possible.

* `modify`: user can move existing points, add intermediate points along lines, and remove points.

* `drawLineString`: user can draw a `LineString` feature by clicking positions to add.

  * If no feature is selected, clicking will create a new feature `Point` feature and select it (by passing its index as `updatedSelectedFeatureIndex`).

  * If a `Point` feature is selected, clicking will convert it to a `LineString` and add the clicked position to it.

  * If a `LineString` feature is selected, clicking will add the clicked position to it.

* `drawPolygon`: user can draw a new `Polygon` feature by clicking positions to add then closing the polygon.

  * If no feature is selected, clicking will create a new feature `Point` feature and select it (by passing its index as `updatedSelectedFeatureIndex`).

  * If a `Point` feature is selected, clicking will convert it to a `LineString` and add the clicked position to it.

  * If a `LineString` feature is selected, clicking will add the clicked position to it. If that clicked position is the first position, the feature will be converted to a `Polygon` feature.

#### `onEdit` (Function, optional)

The `onEdit` event is the core event provided by this layer and must be handled in order to accept and render edits. The `event` argument includes the following properties:

* `updatedData` (Object): A new `FeatureCollection` with the edit applied.

  * To accept the edit as is, supply this object into the `data` prop on the next render cycle (e.g. by calling React's `setState` function)

  * To reject the edit, do nothing

  * You may also supply a modified version of this object into the `data` prop on the next render cycle (e.g. if you have your own snapping logic).

* `updatedMode` (String): A suggested value to use for `mode` after the edit is applied. Often this is the same value. But occasionally, an edit will need to transition the layer from one mode to another.

* `updatedSelectedFeatureIndex` (Number): A suggested value to use for `selectedFeatureIndex` after the edit is applied. Often this is the same value. But occasionally, an edit will change the selected feature (e.g. when creating a new feature, it goes from `null` to the index of the newly created feature).

* `editType` (String): The type of edit requested. One of:

  * `movePosition`: A position was moved.

  * `addPosition`: A position was added (either at the beginning, middle, or end of a feature's coordinates). Note: this may result in a feature being upgraded from one type to another (e.g. `Point` to `LineString` or `LineString` to `Polygon`).

  * `removePosition`: A position was removed.

  * `addFeature`: A new feature was added. Its index is reflected in `featureIndex`.

* `featureIndex` (Number): The index of the edited/added feature.

* `positionIndexes` (Array): An array of numbers representing the indexes of the edited position within the features' `coordinates` array

* `position` (Array): An array containing the ground coordinates (i.e. [lng, lat]) of the edited position (or `null` if it doesn't apply to the type of edit performed)

##### Example

Consider the user removed the third position from a `Polygon`'s first ring, and that `Polygon` was the fourth feature in the `FeatureCollection`. The `event` argument would look like:

```js
{
  updatedData: {...},
  updatedMode: 'modify',
  updatedSelectedFeatureIndex: 3,
  editType: 'removePosition',
  featureIndex: 3,
  positionIndexes: [1, 2],
  position: null
}
```

#### `pickable` (Boolean, optional)

* Default: `true`

Defaulted to `true` for interactivity.

### GeoJsonLayer Options

The following properties from [GeoJsonLayer](https://uber.github.io/deck.gl/#/documentation/deckgl-api-reference/layers/geojson-layer) are supported and function the same:

* `filled`
* `stroked`
* `lineWidthScale`
* `lineWidthMinPixels`
* `lineWidthMaxPixels`
* `lineJointRounded`
* `lineMiterLimit`
* `pointRadiusScale`
* `pointRadiusMinPixels`
* `pointRadiusMaxPixels`
* `fp64`
* `getRadius`
* `getLineWidth`
* TODO: `lineDashJustified`
* TODO: `getLineDashArray`

### Edit Handles Options

Edit handles are the points rendered on a feature to indicate interactive capabilities (e.g. vertices that can be moved). Edit handle objects have the following properties:

* `type` (String): either `existing` for existing positions or `intermediate` for positions half way between two other positions.

#### `editHandlePointRadiusScale` (Number, optional)

* Default: `1`

#### `editHandlePointRadiusMinPixels` (Number, optional)

* Default: `4`

#### `editHandlePointRadiusMaxPixels` (Number, optional)

* Default: `Number.MAX_SAFE_INTEGER`

#### `getEditHandlePointColor` (Function|Array, optional)

* Default: `handle => handle.type === 'existing' ? [0xc0, 0x0, 0x0, 0xff] : [0x0, 0x0, 0x0, 0x80]`

#### `getEditHandlePointRadius` (, optional)

* Default: `handle => (handle.type === 'existing' ? 5 : 3)`

### Data Accessors

#### `getLineColor` (Function|Array, optional)

* Default: `(feature, isSelected) =>
    isSelected ? [0x90, 0x90, 0x90, 0xff] : [0x0, 0x0, 0x0, 0xff]`

The rgba color of line string and/or the outline of polygon for a GeoJson feature, depending on its type.
Format is `r, g, b, [a]`. Each component is in the 0-255 range.

* If an array is provided, it is used as the line color for all features.
* If a function is provided, it is called on each feature to retrieve its line color.
  * The `isSelected` argument indicates if the given feature is the selected feature

#### `getFillColor` (Function|Array, optional)

* Default: `(feature, isSelected) =>
    isSelected ? [0x90, 0x90, 0x90, 0x90] : [0x0, 0x0, 0x0, 0x90]`

The solid color of the polygon and point features of a GeoJson.
Format is `r, g, b, [a]`. Each component is in the 0-255 range.

* If an array is provided, it is used as the fill color for all features.
* If a function is provided, it is called on each feature to retrieve its fill color.
  * The `isSelected` argument indicates if the given feature is the selected feature

Note: This accessor is only called for `Polygon`, `MultiPolygon`, `Point`, and `MultiPoint` features.

## Methods

These methods can be overridden in a derived class in order to customize event handling.

### `onClick`

The pointer went down and up without dragging. This method is called regardless if something was picked.

#### `event` argument

* `picks` (Array): An array containing [deck.gl Picking Info Objects](https://uber.github.io/deck.gl/#/documentation/developer-guide/adding-interactivity?section=what-can-be-picked-) for all objects that were under the pointer when clicked, or an empty array if nothing from this layer was under the pointer.
* `screenCoords` (Array): `[x, y]` screen pixel coordinates relative to the deck.gl canvas.
* `groundCoords` (Array): `[lng, lat]` ground coordinates.

### `onStartDragging`

The pointer went down on something rendered by this layer and the pointer started to move.

* `picks` (Array): An array containing [deck.gl Picking Info Objects](https://uber.github.io/deck.gl/#/documentation/developer-guide/adding-interactivity?section=what-can-be-picked-) for all objects that were under the pointer when it went down.
* `screenCoords` (Array): `[x, y]` screen pixel coordinates relative to the deck.gl canvas where the pointer was when it was considered to start dragging (should be very close to `dragStartScreenCoords`).
* `groundCoords` (Array): `[lng, lat]` ground coordinates where the pointer was when it was considered to start dragging (should be very close to `dragStartGroundCoords`).
* `dragStartScreenCoords` (Array): `[x, y]` screen pixel coordinates relative to the deck.gl canvas where the pointer went down.
* `dragStartGroundCoords` (Array): `[lng, lat]` ground coordinates where the pointer went down.

*Note: this method is not called if nothing was picked when the pointer went down*

### `onDragging`

The pointer went down on something rendered by this layer and the pointer is moving.

* `picks` (Array): An array containing [deck.gl Picking Info Objects](https://uber.github.io/deck.gl/#/documentation/developer-guide/adding-interactivity?section=what-can-be-picked-) for all objects that were under the pointer when it went down.
* `screenCoords` (Array): `[x, y]` screen pixel coordinates relative to the deck.gl canvas where the pointer is now.
* `groundCoords` (Array): `[lng, lat]` ground coordinates where the pointer is now.
* `dragStartScreenCoords` (Array): `[x, y]` screen pixel coordinates relative to the deck.gl canvas where the pointer went down.
* `dragStartGroundCoords` (Array): `[lng, lat]` ground coordinates where the pointer went down.

### `onStopDragging`

The pointer went down on something rendered by this layer, the pointer moved, and now the pointer is up.

* `picks` (Array): An array containing [deck.gl Picking Info Objects](https://uber.github.io/deck.gl/#/documentation/developer-guide/adding-interactivity?section=what-can-be-picked-) for all objects that were under the pointer when it went down.
* `screenCoords` (Array): `[x, y]` screen pixel coordinates relative to the deck.gl canvas where the pointer went up.
* `groundCoords` (Array): `[lng, lat]` ground coordinates where the pointer went up.
* `dragStartScreenCoords` (Array): `[x, y]` screen pixel coordinates relative to the deck.gl canvas where the pointer went down.
* `dragStartGroundCoords` (Array): `[lng, lat]` ground coordinates where the pointer went down.

### `onPointerMove`

The pointer moved, regardless of whether the pointer is down, up, and whether or not something was picked

* `screenCoords` (Array): `[x, y]` screen pixel coordinates relative to the deck.gl canvas where the pointer is now.
* `groundCoords` (Array): `[lng, lat]` ground coordinates where the pointer is now.
* `isDragging` (Boolean): `true` if the pointer is moving but it is considered a drag, in this case you likely want to handle `onDragging`