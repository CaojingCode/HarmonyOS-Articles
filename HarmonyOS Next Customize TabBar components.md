<font style="color:rgba(0, 0, 0, 0.85) !important;">Tabs in ArkUI are container components that facilitate content view switching through tabs. Each tab corresponds to a specific content view. The content is presented using the </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">TabContent</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> component, which is a child component of </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">Tabs</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">. You can customize the style of the navigation bar by setting the </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">tabBar</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> property on </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">TabContent</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">. Now, let's implement the effect shown in the following UI design based on the reference image:</font>

  
![](https://cdn.nlark.com/yuque/0/2025/png/12749040/1748239655155-52c7d5b4-a91d-46da-838f-417921eed620.png)

<h3 id="analysis-and-implementation-steps"><font style="color:rgb(0, 0, 0);">Analysis and Implementation Steps</font></h3>
<font style="color:rgba(0, 0, 0, 0.85) !important;">Based on the analysis of the above image, achieving the desired effect requires the following steps:</font>

1. <font style="color:rgba(0, 0, 0, 0.85) !important;">Set the background color of the tabBar and the selected tab style.</font>
2. <font style="color:rgba(0, 0, 0, 0.85) !important;">Customize the navigation bar indicator.</font>
3. <font style="color:rgba(0, 0, 0, 0.85) !important;">Configure the indicator to slide smoothly with the content view during transitions.</font>

<h3 id="setting-tabbar-background-color-and-selected-tab-style"><font style="color:rgb(0, 0, 0);">Setting TabBar Background Color and Selected Tab Style</font></h3>
1. <font style="color:rgba(0, 0, 0, 0.85) !important;">First, use the</font><font style="color:rgba(0, 0, 0, 0.85) !important;"> </font>`<font style="color:rgb(0, 0, 0);">@Builder</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> </font><font style="color:rgba(0, 0, 0, 0.85) !important;">decorator to define a custom component.</font>
2. <font style="color:rgba(0, 0, 0, 0.85) !important;">Set the background image and color based on the currently selected tab index. Note that the top-left and top-right corners should have rounded edges, which need to be determined based on the tab's position.</font>
3. <font style="color:rgba(0, 0, 0, 0.85) !important;">Since the selected tab style is a trapezoid with rounded corners, we use three different trapezoid images for this effect.</font>

```typescript
@Builder
tabBuilder(title: string, targetIndex: number, selectImage: ResourceStr) {
  // Create a Column layout
  Column() {
    // Create a Text component to display the title
    Text(title)
      // Set font color based on whether the tab is selected
      .fontColor(this.currentIndex === targetIndex ? $r("app.color.text_one") : $r("app.color.text_two"))
      // Set font size to 14
      .fontSize(14)
      // Set font weight based on selection state
      .fontWeight(this.currentIndex === targetIndex ? 600 : 400)
  }
  // Set Column width to 100%
  .width('100%')
  // Set Column height to 100%
  .height("100%")
  // Vertically center the child component
  .justifyContent(FlexAlign.Center)
  // Set background image if the tab is selected
  .backgroundImage(this.currentIndex == targetIndex ? selectImage : null)
  // Set background color
  .backgroundColor($r("app.color.bg_data_color"))
  // Set rounded corners for top-left and top-right based on tab position
  .borderRadius({ topLeft: targetIndex == 0 ? 8 : 0, topRight: targetIndex == 2 ? 8 : 0 })
  // Set background image to fill the container
  .backgroundImageSize(ImageSize.FILL)
}
```

<h3 id="customizing-the-navigation-bar-indicator"><font style="color:rgb(0, 0, 0);">Customizing the Navigation Bar Indicator</font></h3>
<font style="color:rgba(0, 0, 0, 0.85) !important;">Since the indicator needs to slide smoothly with the content view, it cannot be set within the individual </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">tabBuilder</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">. Instead:</font>

1. <font style="color:rgba(0, 0, 0, 0.85) !important;">Use a</font><font style="color:rgba(0, 0, 0, 0.85) !important;"> </font>`<font style="color:rgb(0, 0, 0);">Column</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> </font><font style="color:rgba(0, 0, 0, 0.85) !important;">component to define the bottom indicator with a blue bar that matches the text width and has a height of 3.</font>
2. <font style="color:rgba(0, 0, 0, 0.85) !important;">The indicator width can be dynamically set to match the text width or a fixed value.</font>
3. <font style="color:rgba(0, 0, 0, 0.85) !important;">The left margin of the indicator should be dynamically adjusted with animation to achieve smooth sliding effects.</font>

```typescript
Stack() {
  Tabs({ barPosition: BarPosition.Start }) {
    TabContent() {
      this.tripPage()
    }.tabBar(this.tabBuilder("Property", 0, $r("app.media.trip_data_start_bg")))
    .align(Alignment.TopStart).margin({ top: 54 })
    
    // Additional TabContent components would follow...
    // ...
    // ...
  }
  .backgroundColor($r("app.color.white"))
  .borderRadius(8)
  .barHeight(44)
  .width("93.6%")
  .height(380)
  .onChange((index) => {
    this.currentIndex = index
  })

  // Custom indicator: a blue bar with dynamic width and position
  Column()
    .width(this.indicatorWidth)
    .height(3)
    .backgroundColor($r("app.color.main_color"))
    .margin({ left: this.indicatorLeftMargin, top: 42 })
    .borderRadius(1)
}
```

<h3 id="adding-indicator-animation"><font style="color:rgb(0, 0, 0);">Adding Indicator Animation</font></h3>
<font style="color:rgba(0, 0, 0, 0.85) !important;">To make the indicator slide smoothly with finger gestures during tab transitions, we need to add animations and listen for tab animation events and gesture interactions.</font>

```typescript
/**
 * Starts an animation to move the indicator to a specified position
 * 
 * @param duration Animation duration in milliseconds
 * @param leftMargin Left margin after animation completes
 * @param width Width after animation completes
 */
private startAnimateTo(duration: number, leftMargin: number, width: number) {
  // Set animation start flag
  this.isStartAnimateTo = true
  animateTo({
    // Animation duration
    duration: duration,
    // Animation curve
    curve: Curve.Linear,
    // Number of iterations
    iterations: 1,
    // Play mode
    playMode: PlayMode.Normal,
    // Callback when animation finishes
    onFinish: () => {
      // Reset animation flag
      this.isStartAnimateTo = false
      // Log animation completion
      console.info('Animation finished')
    }
  }, () => {
    // Update indicator position and width
    this.indicatorLeftMargin = leftMargin
    this.indicatorWidth = width
  })
}
```

<h3 id="1.-animation-start-listener"><font style="color:rgb(0, 0, 0);">1. Animation Start Listener</font></h3>
<font style="color:rgba(0, 0, 0, 0.85) !important;">When the tab switching animation starts, set the target index as the current index and start the indicator animation to dynamically adjust its left margin.</font>

```typescript
Tabs({ barPosition: BarPosition.Start })
// ...
.onAnimationStart((index: number, targetIndex: number, event: TabsAnimationEvent) => {
  // Triggered when tab animation starts. Move indicator with the page transition.
  this.currentIndex = targetIndex
  this.startAnimateTo(this.animationDuration, this.textInfos[targetIndex][0], this.textInfos[targetIndex][1])
})
```

<h3 id="2.-animation-end-listener"><font style="color:rgb(0, 0, 0);">2. Animation End Listener</font></h3>
<font style="color:rgba(0, 0, 0, 0.85) !important;">When the tab switching animation ends, the </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">onAnimationEnd</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> callback is triggered to finalize the indicator position.</font>

```typescript
Tabs({ barPosition: BarPosition.Start })
// ...
.onAnimationEnd((index: number, event: TabsAnimationEvent) => {
  // Triggered when tab animation ends. Stop the indicator animation.
  let currentIndicatorInfo = this.getCurrentIndicatorInfo(index, event)
  this.startAnimateTo(0, currentIndicatorInfo.left, currentIndicatorInfo.width)
})
```

<h3 id="3.-gesture-swipe-listener"><font style="color:rgb(0, 0, 0);">3. Gesture Swipe Listener</font></h3>
<font style="color:rgba(0, 0, 0, 0.85) !important;">This callback is triggered frame by frame during the page swipe gesture, allowing the indicator to follow the user's finger movement.</font>

```typescript
Tabs({ barPosition: BarPosition.Start })
// ...
.onGestureSwipe((index: number, event: TabsAnimationEvent) => {
  // Triggered frame by frame during page swipe.
  let currentIndicatorInfo = this.getCurrentIndicatorInfo(index, event)
  // Update current index
  this.currentIndex = currentIndicatorInfo.index
  // Update indicator left margin
  this.indicatorLeftMargin = currentIndicatorInfo.left
  // Update indicator width
  this.indicatorWidth = currentIndicatorInfo.width
})
```

<h3 id="encapsulating-indicator-information-retrieval"><font style="color:rgb(0, 0, 0);">Encapsulating Indicator Information Retrieval</font></h3>
<font style="color:rgba(0, 0, 0, 0.85) !important;">We encapsulate the logic to calculate the indicator's position and width in a separate method. This method is called during gesture swipes to dynamically update the indicator's properties, ensuring it follows the user's gesture smoothly and achieves the desired UI effect.</font>

```typescript
/**
 * Calculates the current indicator information based on the swipe event
 * 
 * @param index Current tab index
 * @param event Tabs animation event containing swipe information
 * @returns Object containing indicator index, left margin, and width
 */
private getCurrentIndicatorInfo(index: number, event: TabsAnimationEvent): Record<string, number> {
  // Determine next tab index based on swipe direction
  let nextIndex = index

  // Swiping left (from user perspective)
  if (index > 0 && event.currentOffset > 0) {
    nextIndex--
  }
  // Swiping right (from user perspective)
  else if (index < 3 && event.currentOffset < 0) {
    nextIndex++
  }

  // Get information about current and next tabs
  let indexInfo = this.textInfos[index]
  let nextIndexInfo = this.textInfos[nextIndex]

  // Calculate swipe ratio
  let swipeRatio = Math.abs(event.currentOffset / this.tabsWidth)

  // Determine current index based on swipe progress
  // Switch to next tab's style when swiped more than halfway
  let currentIndex = swipeRatio > 0.5 ? nextIndex : index

  // Interpolate left margin and width based on swipe ratio
  let currentLeft = indexInfo[0] + (nextIndexInfo[0] - indexInfo[0]) * swipeRatio
  let currentWidth = indexInfo[1] + (nextIndexInfo[1] - indexInfo[1]) * swipeRatio

  // Return calculated properties
  return { 'index': currentIndex, 'left': currentLeft, 'width': currentWidth }
}
```

<font style="color:rgba(0, 0, 0, 0.85) !important;">This implementation effectively creates a smooth and visually appealing tab navigation system that closely matches modern UI design standards.</font>

