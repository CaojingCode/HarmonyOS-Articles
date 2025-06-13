In HarmonyOS development, the need to modify image colors often arises in scenarios such as theme switching, status indication, or style unification. The following comprehensively covers technical principles, implementation methods, and optimization strategies across various scenarios including vector graphics, bitmaps, and dynamic rendering:  



### I. Modifying Vector Graphic Colors: Flexible Adaptation via `fillColor`
#### 1. Core Principles and Use Cases
Vector graphics (e.g., SVG, AI) describe shapes through mathematical paths, allowing direct modification of fill colors using the `fillColor` property. Advantages include:  

+ **Lightweight Rendering**: No additional resource files required; color changes via property configuration  
+ **Dynamic Responsiveness**: Real-time color switching based on runtime states (e.g., theme mode, interaction state)  
+ **Consistency Maintenance**: Single vector graphic adaptable to multiple color schemes, reducing resource redundancy

#### 2. Advanced Usage and Code Examples
```typescript
// Basic usage: Dynamically switch icon color based on system theme
Image({
  source: $r('app.media.icon_search'), // Reference SVG resource
  fillColor: App.abilityContext?.colorMode === 1 ? '#FFFFFF' : '#000000'
})
.width(24)
.height(24)

// Interaction state color change: Icon color switches on button press
@Entry
@Component
struct IconColorDemo {
  @State isPressed: boolean = false

  build() {
    Button('Click to Switch Color') {
      Image({
        source: $r('app.media.icon_star'),
        fillColor: this.isPressed ? '#FF5500' : '#999999'
      })
      .width(20)
      .height(20)
    }
    .onClick(() => {
      this.isPressed = !this.isPressed
    })
  }
}

// Multi-color gradient fill (supported by some vector formats)
Image({
  source: $r('app.media.icon_compass'),
  fillColor: 'linear-gradient(to right, #3A7BD5, #6B46C1)'
})
.width(32)
.height(32)
```

#### 3. Notes
+ **Resource Format Limitation**: Only vector graphics support the `fillColor` property; bitmaps (PNG/JPG) require alternative methods  
+ **Transparency Compatibility**: If vector graphics contain transparent channels (e.g., `opacity` in `.svg`), `fillColor` will blend with transparency



### II. Modifying Bitmap Colors: Multi-Resource Adaptation and Dynamic Rendering
#### 1. Static Resource Adaptation
Distinguish theme-specific resources via directory structure; the system auto-loads appropriate files:  

```plain
// Resource directory structure
src/main/resources/
├── base/              # Default resources for light mode
│   ├── image_normal.png
│   └── icon_default.svg
└── dark/              # Exclusive resources for dark mode
    ├── image_normal.png  # Dark theme bitmap
    └── icon_default.svg  # Dark theme vector graphic
```

#### 2. Dynamic Color Filtering (ColorFilter)
For real-time color adjustments (e.g., grayscale, inversion), use `ColorFilter`:  

```typescript
// Grayscale processing (suitable for disabled state icons)
Image($r('app.media.image_original'))
  .width(100)
  .height(100)
  .colorFilter({
    type: ColorFilterType.Matrix,
    matrix: [
      0.2126, 0.7152, 0.0722, 0, 0,  // R channel grayscale weight
      0.2126, 0.7152, 0.0722, 0, 0,  // G channel grayscale weight
      0.2126, 0.7152, 0.0722, 0, 0,  // B channel grayscale weight
      0, 0, 0, 1, 0                  // Alpha channel preserved
    ]
  })

// Theme color filtering (map image primary color to app theme)
Image($r('app.media.image_original'))
  .width(120)
  .height(120)
  .colorFilter({
    type: ColorFilterType.Lighting,
    lighting: {
      mul: '#FF0000',  // Color blend value (red)
      add: '#000000'   // Color overlay value (black)
    }
  })
```

#### 3. Runtime Resource Replacement
Dynamically load and modify bitmap colors via `ImageSource` (complex programmatic scenarios):  

```typescript
import image from '@ohos.multimedia.image'

async function modifyImageColor() {
  // 1. Load original image
  const imageSource = await image.createImageSource(
    await fetch($r('app.media.image_original')).arrayBuffer()
  )
  const imageInfo = await imageSource.getImageInfo()
  
  // 2. Create color matrix (enhance red channel by 50%)
  const colorMatrix = new Float32Array([
    1.5, 0, 0, 0, 0,    // R channel
    0, 1, 0, 0, 0,      // G channel
    0, 0, 1, 0, 0,      // B channel
    0, 0, 0, 1, 0       // Alpha channel
  ])
  
  // 3. Apply color filter and generate new image
  const options = {
    colorFilter: {
      type: image.ColorFilterType.MATRIX,
      matrix: colorMatrix
    }
  }
  const newImage = await imageSource.createImage(options)
  
  // 4. Display modified image in UI
  this.modifiedImage = newImage
}
```



### III. Hybrid Approach: Synergistic Optimization of Vector and Bitmap Graphics
#### 1. Automatic Format Detection and Adaptation
```typescript
// Automatically select color modification method based on image format
@Builder
ImageWithColorAdaptation(source: ResourceStr, color: string) {
  // Detect resource extension (simplified logic; actual implementation requires resource manager)
  const isVector = source.endsWith('.svg') || source.endsWith('.ai')
  
  if (isVector) {
    // Use fillColor for vector graphics
    Image({ source: source, fillColor: color })
  } else {
    // Use color filtering for bitmaps
    Image(source)
      .colorFilter({
        type: ColorFilterType.Lighting,
        lighting: { mul: color, add: '#000000' }
      })
  }
}
```

#### 2. Performance Optimization Strategies
+ **Prioritize Vector Graphics**: Use SVG for complex icons to reduce resource size and memory usage  
+ **Caching Mechanism**: Cache processed `PixelMap` objects for frequently used color-filtered images  
+ **Lazy Loading**: Defer color modification operations for non-visible images



### IV. Common Issues and Solutions
| Problem Scenario | Solution |
| --- | --- |
| `fillColor` ineffective for vector graphics | Ensure resource is a pure path graphic (SVG with embedded bitmaps not supported) |
| Blurred bitmap after color filtering | Adjust `colorMatrix` parameters or use `ColorFilterType.LUT` for lookup table optimization |
| Chaotic multi-theme resource management | Use `$r` resource references with conditional compilation (e.g., `#ifdef DARK_MODE`) |


These approaches enable developers to achieve full-scenario color control in HarmonyOS, from simple theme switching to complex image effects, while balancing performance optimization and development efficiency.

