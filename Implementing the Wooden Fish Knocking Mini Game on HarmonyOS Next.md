**"Knocking the Wooden Fish" - A Zen-inspired Mini Game**  
This article delves into the core technical aspects of implementing a "knocking wooden fish" game using HarmonyOS's ArkUI framework. Key features include animated interactions, state management, haptic feedback, and sound effects.

![](https://cdn.nlark.com/yuque/0/2025/jpeg/12749040/1741337114932-c51d32dc-cc3d-4f81-bb0a-1d295c0e58b8.jpeg?x-oss-process=image%2Fformat%2Cwebp)

## **I. Architecture Design & Project Setup**
#### **1.1 Project Structure**
The complete project consists of these core modules:  

```plain
├── entry/src/main/ets/
│   ├── components/         // Custom UI components
│   ├── model/              // Data models (e.g., StateArray)
│   ├── pages/              // Page components (WoodenFishGame.ets)
│   └── resources/          // Media assets (wooden fish icons, sound effects)
```

This modular design separates the UI layer (pages), logic layer (model), and resource layer (resources), adhering to HarmonyOS development standards.

#### **1.2 Component-based Development**
Using the `@Component` decorator to create reusable UI units, with `@Entry` marking the page entry. Key states are managed via `@State`:  

```typescript
@Entry
@Component
struct WoodenFishGame {
  @State count: number = 0;                // Merit counter
  @State scaleWood: number = 1;           // Wooden fish scale factor
  @State rotateWood: number = 0;          // Wooden fish rotation angle
  @State animateTexts: Array<StateArray> = []; // Animation queue
  private audioPlayer?: media.AVPlayer;    // Audio player instance
  private autoPlay: boolean = false;       // Auto-tap flag
}
```

`@State` enables **reactive programming**: UI elements automatically re-render when state variables change.



## **II. Animation System Deep Dive**
#### **2.1 Composite Animation for Knocking**
The animation consists of two phases: **compression (100ms)** and **elastic recovery (200ms)**, using `animateTo` for smooth transitions:  

```typescript
playAnimation() {
  // Phase 1: Quick compression + left rotation
  animateTo({
    duration: 100, 
    curve: Curve.Friction // Simulates physical resistance
  }, () => {
    this.scaleWood = 0.9; // Scale X/Y to 90%
    this.rotateWood = -2; // Rotate counter-clockwise 2 degrees
  });

  // Phase 2: Elastic recovery
  setTimeout(() => {
    animateTo({
      duration: 200,
      curve: Curve.Linear // Ensures smooth recovery
    }, () => {
      this.scaleWood = 1; 
      this.rotateWood = 0;
    });
  }, 100); // Delay for animation chaining
}
```

**Curve Selection**:  

+ `Curve.Friction`: Simulates impact resistance  
+ `Curve.Linear`: Ensures consistent recovery speed

#### **2.2 Floating Merit Text Animation**
Manages multiple animations using a dynamic array with unique IDs:  

```typescript
countAnimation() {
  const animId = new Date().getTime(); // Unique timestamp ID
  // Add new animation element
  this.animateTexts = [...this.animateTexts, { 
    id: animId, 
    opacity: 1, 
    offsetY: 20 
  }];
  
  // Fade out and float animation
  animateTo({
    duration: 800,
    curve: Curve.EaseOut // Simulates inertia
  }, () => {
    this.animateTexts = this.animateTexts.map(item => 
      item.id === animId ? 
        { ...item, opacity: 0, offsetY: -100 } : item
    );
  });

  // Clean up after animation completes
  setTimeout(() => {
    this.animateTexts = this.animateTexts.filter(t => t.id !== animId);
  }, 800); // Synchronized with animation duration
}
```

**Key Techniques**:  

1. **Data-driven**: Modifying `animateTexts` triggers UI re-render  
2. **Layered animation**: Control opacity and vertical offset  
3. **Memory optimization**: Remove completed animations to prevent bloat



## **III. Multi-modal Interaction**
#### **3.1 Haptic Feedback**
Utilizes the `@kit.SensorServiceKit` vibration module:  

```typescript
vibrator.startVibration({
  type: "time",       // Time-based vibration
  duration: 50        // 50ms short pulse
}, {
  id: 0,              // Vibrator ID
  usage: 'alarm'      // Resource usage scenario
});
```

**Parameter Tuning**:  

+ **Duration**: 50ms mimics the "crisp" feel of striking wood  
+ **Intensity**: Automatically optimized by HarmonyOS based on usage

#### **3.2 Audio Playback & Resource Management**
Implements sound effects with `media.AVPlayer`:  

```typescript
aboutToAppear(): void {
  media.createAVPlayer().then(player => {
    this.audioPlayer = player;
    this.audioPlayer.url = ""; 
    this.audioPlayer.loop = false; // Disable looping
  });
}

// Reset playback position on tap
if (this.audioPlayer) {
  this.audioPlayer.seek(0);    // Jump to 0ms
  this.audioPlayer.play();     // Play sound
}
```

**Best Practices**:  

1. **Preload resources**: Initialize player during page setup  
2. **Avoid delays**: Use `seek(0)` for instant sound  
3. **Resource cleanup**: Call `release()` in `onPageHide` to prevent leaks



## **IV. Auto-Tap Functionality**
#### **4.1 Timer-State Integration**
Toggle auto-tap mode with the `Toggle` component:  

```typescript
// State toggle callback
Toggle({ type: ToggleType.Checkbox, isOn: false })
  .onChange((isOn: boolean) => {
    this.autoPlay = isOn;
    if (isOn) {
      this.startAutoPlay();
    } else {
      clearInterval(this.intervalId); // Clear timer
    }
  });

// Start timer
private intervalId: number = 0;
startAutoPlay() {
  this.intervalId = setInterval(() => {
    if (this.autoPlay) this.handleTap();
  }, 400); // 400ms interval mimics human tapping
}
```

**Key Improvements**:  

+ Track timer with `intervalId` to ensure cleanup  
+ 400ms interval balances smoothness and performance

#### **4.2 Thread Safety & Performance**
**Risk**: Frequent timer triggers may block the UI thread  
**Solution**:  

```typescript
// Clear timer when page is hidden
aboutToDisappear() {
  clearInterval(this.intervalId);
}
```

Ensures resources are released when the page is inactive.



## **V. UI Layout & Rendering Optimization**
#### **5.1 Layered Layout & Animation Composition**
Use `Stack` for overlapping UI elements:  

```typescript
Stack() {
  // Wooden fish base (bottom layer)
  Image($r("app.media.icon_wooden_fish"))
    .width(280)
    .height(280)
    .margin({ top: -10 })
    .scale({ x: this.scaleWood, y: this.scaleWood })
    .rotate({ angle: this.rotateWood });

  // Floating merit text (top layer)
  ForEach(this.animateTexts, (item, index) => {
    Text(`+1`)
      .translate({ y: -item.offsetY * index }) // Staggered positioning
  });
}
```

**Rendering Optimization**:  

+ Add `fixedSize(true)` to static images to avoid re-layout  
+ Use `translate` instead of `margin` for animations to reduce layout recalculations

#### **5.2 Efficient State-to-UI Binding**
Dynamic styling with chain methods:  

```typescript
Text(`Merit +${this.count}`)
  .fontSize(20)
  .fontColor('#4A4A4A')
  .margin({ 
    top: 20 + AppUtil.getStatusBarHeight() // Adaptive to notch screens
  })
```

**Adaptation Strategies**:  

+ Use `AppUtil.getStatusBarHeight()` to avoid notch interference  
+ Leverage HarmonyOS's **flexible layout** for screen size adaptability



## **VI. Complete Code**
```typescript
import { media } from '@kit.MediaKit';
import { vibrator } from '@kit.SensorServiceKit';
import { AppUtil, ToastUtil } from '@pura/harmony-utils';
import { StateArray } from '../model/HomeModel';

@Entry
@Component
struct WoodenFishGame {
  @State count: number = 0;
  @State scaleWood: number = 1;
  @State rotateWood: number = 0;
  audioPlayer?: media.AVPlayer;
  // Auto-tap feature
  autoPlay: boolean = false;

  // New state variable
  @State animateTexts: Array<StateArray> = []

  aboutToAppear(): void {
    media.createAVPlayer().then(player => {
      this.audioPlayer = player
      this.audioPlayer.url = ""
    })
  }

  startAutoPlay() {
    setInterval(() => {
      if (this.autoPlay) {
        this.handleTap();
      }
    }, 400);
  }

  // Knocking animation
  playAnimation() {
    animateTo({
      duration: 100,
      curve: Curve.Friction
    }, () => {
      this.scaleWood = 0.9;
      this.rotateWood = -2;
    });

    setTimeout(() => {
      animateTo({
        duration: 200,
        curve: Curve.Linear
      }, () => {
        this.scaleWood = 1;
        this.rotateWood = 0;
      });
    }, 100);
  }

  // Tap handler
  handleTap() {
    this.count++;
    this.playAnimation();
    this.countAnimation();
    
    // Add haptic feedback
    vibrator.startVibration({
      type: "time",
      duration: 50
    }, {
      id: 0,
      usage: 'alarm'
    });

    // Play sound effect
    if (this.audioPlayer) {
      this.audioPlayer.seek(0);
      this.audioPlayer.play();
    }
  }

  countAnimation() {
    // Unique ID to prevent animation conflicts
    const animId = new Date().getTime();
    // Initialize animation state
    this.animateTexts = [...this.animateTexts, {id: animId, opacity: 1, offsetY: 20}];
    
    // Execute animation
    animateTo({
      duration: 800,
      curve: Curve.EaseOut
    }, () => {
      this.animateTexts = this.animateTexts.map(item => {
        if (item.id === animId) {
          return { id: item.id, opacity: 0, offsetY: -100 };
        }
        return item;
      });
    });

    // Clean up after animation
    setTimeout(() => {
      this.animateTexts = this.animateTexts.filter(t => t.id !== animId);
    }, 800);
  }

  build() {
    Column() {
      // Counter display
      Text(`Merit +${this.count}`)
        .fontSize(20)
        .margin({ top: 20 + AppUtil.getStatusBarHeight() })

      // Wooden fish container
      Stack() {
        // Tappable area
        Image($r("app.media.icon_wooden_fish"))
          .width(280)
          .height(280)
          .margin({ top: -10 })
          .scale({ x: this.scaleWood, y: this.scaleWood })
          .rotate({ angle: this.rotateWood })
          .onClick(() => this.handleTap())

        // Floating text container
        ForEach(this.animateTexts, (item: StateArray, index) => {
          Text(`+1`)
            .fontSize(24)
            .fontColor('#FFD700')
            .opacity(item.opacity)
            .margin({ top: -100 }) // Initial position
            .translate({ y: -item.offsetY * index }) // Vertical displacement
            .animation({ duration: 800, curve: Curve.EaseOut })
        })
      }
      .margin({ top: 50 })

      // Auto-tap toggle (extended feature)
      Row() {
        Toggle({ type: ToggleType.Checkbox, isOn: false })
          .onChange((isOn: boolean) => {
            this.autoPlay = isOn;
            if (isOn) {
              this.startAutoPlay();
            } else {
              clearInterval();
            }
          })
        Text("Auto-Tap")
      }
      .alignItems(VerticalAlign.Center)
      .justifyContent(FlexAlign.Center)
      .width("100%")
      .position({ bottom: 100 })
    }
    .width('100%')
    .height('100%')
    .backgroundColor('#f0f0f0')
  }
}
```

This implementation showcases HarmonyOS ArkUI's capabilities in building engaging, interactive applications with smooth animations, responsive state management, and multi-modal feedback.

