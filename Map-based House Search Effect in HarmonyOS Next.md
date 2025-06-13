    
The commonly used map-based house search function involves adding custom markers for regions, business districts, properties, etc., on the map, combined with the filtering logic of the application. Here is a simple implementation of adding region/business district and property markers using HarmonyOS ArkUI.  


![](https://cdn.nlark.com/yuque/0/2025/png/12749040/1749713829856-35773b26-79b6-4d3d-ae11-23800559c10b.png)

### 1. Enable Map Service
Register the application on the Huawei Developer Website, then enable the Map Kit service in **My Projects > My Apps > API Management**.  

### 2. Map-Related Configuration
Configure the `client_id` in the `module.json5` file of the entry module. The `client_id` can be found in **Project Settings > General > App**. You may need to configure the public key fingerprint; refer to the developer website for details.  

```json
"module": {
  "name": "xxxx",
  "type": "entry",
  "description": "xxxx",
  "mainElement": "xxxx",
  "deviceTypes": [
    "phone",
    "tablet"
  ],
  "pages": "xxxx",
  "abilities": [],
  "metadata": [
    {
      "name": "client_id",
      "value": "xxxxxx"  // Configure with the obtained Client ID
    }
  ]
}
```

### 3. Create a Map
Use the `MapComponent` to create a map, which requires two parameters: `mapOptions` and `mapCallback`. Override the lifecycle methods `onPageShow` and `onPageHide` to display and hide the map. If the map does not display, refer to the official website for troubleshooting.  

```typescript
MapComponent(mapOptions: mapCommon.MapOptions, mapCallback: AsyncCallback<map.MapComponentController>)

// Map initialization parameters: set the map center coordinates and zoom level
this.mapOptions = {
  position: {
    target: {
      latitude: 39.9,
      longitude: 116.4
    },
    zoom: 10
  }
};

// Map initialization callback
this.callback = async (err, mapController) => {
  if (!err) {
    // Get the map controller to operate the map
    this.mapController = mapController;
    this.mapEventManager = this.mapController.getEventManager();
    const callback = () => {
      console.info(this.TAG, `on-mapLoad`);
    }
    this.mapEventManager.on("mapLoad", callback);
  }
};

// Triggered each time the page is displayed (including routing and app foreground scenarios), only valid for @Entry-decorated custom components
onPageShow(): void {
  // Bring the map to the foreground
  if (this.mapController) {
    this.mapController.show();
  }
}

// Triggered each time the page is hidden (including routing and app background scenarios), only valid for @Entry-decorated custom components
onPageHide(): void {
  // Send the map to the background
  if (this.mapController) {
    this.mapController.hide();
  }
}

build() {
  Stack() {
    // Initialize the map by calling MapComponent
    MapComponent({ mapOptions: this.mapOptions, mapCallback: this.callback }).width('100%').height('100%');
  }.height('100%')
}
```

### 4. Display My Location Icon on the Map
First, dynamically request location permissions. After obtaining the specific location coordinates, use the map operation class `mapController` to call `setMyLocation` to set the location coordinates and `animateCamera` to move the map to the location (with a configurable zoom level) to display a location icon.  

```typescript
static LocationPermissions: Array<Permissions> =
  ['ohos.permission.LOCATION', 'ohos.permission.APPROXIMATELY_LOCATION'];

// Start location positioning
startLocation() {
  // Request location permissions
  AgentUtil.requestPermissions(AgentUtil.LocationPermissions, async () => {
    // Location permission granted
    // Enable the my-location layer (mapController is obtained as described in the "Display Map" section)
    this.mapController?.setMyLocationEnabled(true);
    // Enable the my-location button
    this.mapController?.setMyLocationControlsEnabled(true);

    // Get the user's location coordinates
    const location = await geoLocationManager.getCurrentLocation();
    // Convert coordinate system (if needed)
    // const wgs84LatLng = AgentUtil.wgs84LatLng(location.latitude, location.longitude);
    // location.latitude = wgs84LatLng.latitude;
    // location.longitude = wgs84LatLng.longitude;
    // Set the user's location
    this.mapController?.setMyLocation(location);

    setTimeout(() => {
      // Move the map to the target location
      this.mapController?.animateCamera(map.newLatLng(location, 15));
    }, 100);
  });
}
```

### 5. Listen for Map Movement Stop
After the map moves to the location coordinates, use `mapController.on("cameraIdle", () => {})` to listen for when the map stops moving. In the callback, implement the logic to display markers based on the map zoom level (e.g., display regions/districts at lower zooms and properties at higher zooms), and further refine business logic such as business district layers.  

```typescript
this.mapEventManager.on("cameraIdle", () => {
  // The map has stopped moving; perform operations
  const position = this.mapController?.getCameraPosition();
  const currentZoom = position?.zoom ?? 0;
  if (currentZoom < 14) {
    // Display region/business district markers
    this.addAreaMarkers();
  } else {
    // Display property markers
    this.addHouseMarkers();
  }
});
```

### 6. Customize Markers
#### Custom Circular Marker
Used to display regions and business districts. Since Map Kit does not support custom styles for markers, only icon settings, use a `@Builder` custom component to draw a circular marker:  

```typescript
@Builder
BuilderMapCircularLabel(text: string, id: string) {
  Stack() {
    // Draw a circular background
    Circle({ width: 70, height: 70 })
      .shadow({ radius: 8 })
      .fill($r("app.color.main_color"));
    Text(text)
      .fontSize(12)
      .fontColor($r("app.color.white"))
      .textAlign(TextAlign.Center)
      .padding({ left: 4, right: 4 });
  }.id(id);
}
```

#### Custom Property Display Marker
Draw a small triangle at the bottom using a `Path` path.  

```typescript
/**
 * Rectangular marker with a small triangle indicator
 * @param text
 */
@Builder
BuilderMapRectLabel(text: string, id: string) {
  Column() {
    Text(text)
      .fontSize(14)
      .fontColor($r("app.color.white"))
      .backgroundColor($r("app.color.main_color"))
      .shadow({ radius: 5, color: $r("app.color.white") })
      .borderRadius(14)
      .padding({ left: 15, right: 15, top: 4, bottom: 4 });
    // Draw a small triangle using path commands
    Path()
      .width(12)
      .height(6)
      .commands('M0 0 L30 0 L15 15 Z')
      .fill($r("app.color.main_color"))
      .strokeWidth(0);
  }.id(id).alignItems(HorizontalAlign.Center);
}
```

### 7. Display Custom Markers on the Map
Use the screenshot class `componentSnapshot` to generate a snapshot of the custom component via `createFromBuilder`. In the callback, obtain the `PixelMap` object, set it as the `icon` in `MarkerOptions`, and call `addMarker` to generate the marker on the map.  

```typescript
componentSnapshot.createFromBuilder(() => {
  // The custom component to generate a PixelMap image
  this.BuilderMapRectLabel(text, houseId);
}, async (error, imagePixelMap) => {
  if (error) {
    LogUtil.debug("snapshot failure: " + JSON.stringify(error));
    return;
  }

  const markerOptions: mapCommon.MarkerOptions = {
    position: latLng,
    icon: imagePixelMap, // Custom component converted to PixelMap
    title: houseId,       // Pass custom parameters to bind with the component for click operations
  };
  if (!this.rectMarkers.find(marker => marker.getTitle() === houseId)) {
    // Add the marker if it doesn't exist in the marker collection
    const marker = await this.mapController?.addMarker(markerOptions);
    marker?.setClickable(true);
    if (marker) {
      this.rectMarkers.push(marker);
    }
  }
});
```

### 8. Listen for Marker Click Events
Use `mapEventManager.on("markerClick", () => {})` to listen for marker clicks. In the callback, handle business logic based on the map level (e.g., navigate to the property layer when a region/business district marker is clicked).  

```typescript
this.mapEventManager.on("markerClick", (marker) => {
  if (marker.getTitle().length === 2) {
    // Clicked a region/business district marker: switch to the property layer and display property information
    this.mapController?.animateCamera(map.newLatLng({
      latitude: 30.513746,
      longitude: 114.403009
    }, 15));
  } else {
    // Clicked a property marker: perform subsequent operations
    ToastUtil.showToast(marker?.getTitle());
  }
});
```

### Summary
The above is the basic process for implementing map-based house search with HarmonyOS ArkUI. Details can be customized according to business needs. Currently, custom markers are displayed by generating images from custom components, which may affect performance with a large number of markers. Optimization can be done by displaying only markers within the current screen, removing those outside the screen when scrolling, and reusing generated `PixelMap` objects to avoid repeated creation.

