<font style="color:rgba(0, 0, 0, 0.85) !important;">Recently, there was a requirement in the project regarding the effect of the bottom indicator of the sliding component Scroll following the gesture sliding and displaying proportionally. The following image shows the effect to be implemented.</font>

![](https://cdn.nlark.com/yuque/0/2025/png/12749040/1749628907440-c58cc911-47aa-441c-abcd-1dfd759edd38.png)

### <font style="color:rgb(0, 0, 0);">Custom Indicator Component</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;">Actually, this is not a progress bar, so we need to customize and draw this component. In HarmonyOS, we can use the Canvas component to draw specified graphics on the page. There are 7 drawing types, namely Circle, Ellipse, Line, Polyline, Polygon, Path, and Rect. Here, we use Rect to draw a rectangle, with a black Rect as the background of the indicator and blue as the progress of the indicator. Then, dynamically set the left margin of the progress.</font>

```typescript
@Entry
@Component
export struct RectIndicator {
  @Prop marginLeft: number = 0 // The left margin of the indicator from the progress, default is 0
  indicatorHeight: number = 20 // The height of the indicator
  indicatorWidth: number = 200 // The background width of the indicator
  indicatorProgressWidth: number = 160 // The progress width of the indicator

  build() {
    Stack() {
      // Draw the rectangular background
      Rect({ width: this.indicatorWidth, height: this.indicatorHeight })
        .radius(this.indicatorHeight / 2)
        .fill($r('app.color.bg_gray'))
        .stroke(Color.Transparent)

      // Draw the rectangular progress
      Rect({ width: this.indicatorProgressWidth, height: this.indicatorHeight })
        .radius(this.indicatorHeight / 2)
        .fill($r('app.color.main_color'))
        .margin({ left: this.marginLeft })
        .stroke(Color.Transparent)
    }.alignContent(Alignment.Start)
  }
}
```

### <font style="color:rgb(0, 0, 0);">Adding Scroll Listener</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;">Create a new RectIndicator class. Since two rectangles need to be stacked together, use the Stack layout for nesting. When calling this custom component, you can set the width (indicatorWidth), height (indicatorHeight), progress width (indicatorProgressWidth) of the component, and dynamically set the left margin (marginLeft) of the progress. You can dynamically set marginLeft by listening to the scroll event of Scroll.</font>

```typescript
Scroll(this.scroll) {
  Row() {
    ForEach(SUB_MENUS, (item: Menu, index) => {
      Column() {
        Image(item.menuIcon).width(28).height(28)
        Text(item.menuText).fontSize(12).fontColor($r("app.color.text_two")).margin({ top: 5 })
      }.width("20%").id("item")
    })
  }
}.scrollable(ScrollDirection.Horizontal)
.margin({ top: 12 })
.scrollBar(BarState.Off)
// Scroll listener
.onDidScroll((_xOffset, _yOffset, scrollState) => {
  // The current state is the scrolling state
  if (scrollState == ScrollState.Scroll) {
    // Get the scrolling offset. It's more accurate to get it through the controller
    const currentOffsetX = this.scroll.currentOffset().xOffset
    LogUtil.debug("Scrolling offset", vp2px(currentOffsetX).toString())
    // Subcomponent width * 2 = the component not displayed / the gray part of the indicator
    let ratio = this.itemWidth * 2 / 10
    // The offset of the indicator progress = scroll offset / ratio
    this.indicatorLeft = vp2px(currentOffsetX) / ratio
  }
})
```



<font style="color:rgba(0, 0, 0, 0.85) !important;">The onDidScroll can set a listener for sliding components (including but not limited to Scroll). Here, we judge that the scrollState is the sliding state, get the current sliding offset currentOffsetX, and calculate the left margin (indicatorLeft) of the indicator through the offset of Scroll.</font>

### <font style="color:rgb(0, 0, 0);">Using the Custom RectIndicator Component</font>
```typescript
RectIndicator({
  marginLeft: this.indicatorLeft, // Left margin
  indicatorHeight: 4,  // The height of the indicator
  indicatorWidth: 30,  // The background width of the indicator
  indicatorProgressWidth: 20  // The progress width of the indicator
}).margin({ top: 8, bottom: 8 })
```



**<font style="color:rgb(0, 0, 0) !important;">Usage method</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">: Call the RectIndicator custom component and pass relevant parameters such as height and width into the component. The progress width here can be calculated through the length of the Scroll component. Here, I just set a width, which does not affect the use.</font>

  


<font style="color:rgba(0, 0, 0, 0.85) !important;">It should be noted that indicatorLeft needs to add a @State annotation to ensure that the component can refresh the UI in real-time according to indicatorLeft.</font>

