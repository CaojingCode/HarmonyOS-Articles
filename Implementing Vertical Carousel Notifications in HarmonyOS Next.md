![](https://cdn.nlark.com/yuque/0/2025/gif/12749040/1749887935393-408d0afc-8189-4a92-86e8-d449b3d15810.gif)

<font style="color:rgb(28, 31, 35);">To achieve the automatic vertical scrolling effect for text in ArkUI, the Swiper component can be used. The Swiper component provides the capability to display content in a sliding carousel. As a container component, Swiper can carousel through its child components when multiple ones are configured.</font>

### 1. Vertical Carousel Implementation
```typescript
Swiper() {
  ForEach(this.transactionList, (item: TransactionInfo) => {
    Row() {
      Image(StrUtil.isBlank(item.TitleImage) ? $r("app.media.icon_circular_default_head") : item.TitleImage)
        .width(44)
        .height(44)
        .borderRadius(22)
      Text(item.Title)
        .fontSize(14)
        .fontColor($r("app.color.text_one"))
        .maxLines(2)
        .ellipsisMode(EllipsisMode.END)
        .textOverflow({ overflow: TextOverflow.Ellipsis })
        .lineHeight(22)
        .margin({ left: 9 })
        .layoutWeight(1)
    }.onClick(() => {
      ToastUtil.showToast("Clicked on transaction report")
    })
  })
}
.vertical(true) // Enable vertical carousel direction
.loop(true)    // Enable loop playback
.autoPlay(true) // Enable auto-play
.indicator(false) // Whether to show navigation dots
.height(44)
.margin({ left: 9 })
.layoutWeight(1)
```

**Key Properties**:

+ **vertical**: When `true`, the carousel slides vertically; when `false`, it slides horizontally (default: `false`).
+ **loop**: Controls whether the carousel loops (default: `true`).
+ **autoPlay**: Enables automatic carousel of child components (default: `false`, interval: 3000ms).
+ **interval**: Carousel interval in milliseconds (default: 3000ms).
+ **indicator**: Whether to display navigation dots.

### 2. Implementing a Custom Dialog
![](https://cdn.nlark.com/yuque/0/2025/png/12749040/1745547090590-a5f71ee5-d5bd-4eb0-902a-99f11be7051d.png?x-oss-process=image%2Fformat%2Cwebp)

Use the `@CustomDialog` decorator to create a custom dialog where you can define the dialog's content, add confirm/cancel buttons, and include data functions.

```typescript
import { StrUtil } from "@pura/harmony-utils"
import { TransactionInfo } from "../model/TransactionInfo"

@Preview
@CustomDialog
export struct DealInfoDialog {
  dealDialogController?: CustomDialogController
  @Prop transaction?: TransactionInfo = undefined

  build() {
    Column() {
      Text("Breaking News: Important Announcement")
        .width("60%")
        .margin({ top: 92 })
        .fontSize(16)
        .fontWeight(600)
        .fontColor($r("app.color.yellow_aureus"))
        .textAlign(TextAlign.Center)
      Row() {
        Image(StrUtil.isBlank(this.transaction?.TitleImage) ? $r("app.media.placeholder") :
          this.transaction?.TitleImage).width(68).height(68).borderRadius(6)
        Column() {
          Text(this.transaction?.Title)
            .fontSize(13)
            .fontWeight(600)
            .fontColor($r("app.color.yellow_aureus"))
            .maxLines(1)
            .width("70%")
            .ellipsisMode(EllipsisMode.END)
            .textOverflow({ overflow: TextOverflow.Ellipsis })
          Text("Property ID: " + StrUtil.toStr(this.transaction?.PropertyNo))
            .fontSize(12)
            .fontColor($r("app.color.yellow_aureus"))
          Row() {
            Text(this.transaction?.Price.toString()).fontColor("#DC2424").fontSize(18).fontWeight(800)
            Text("CNY").fontColor("#DC2424").fontSize(12).fontWeight(600).baselineOffset(-2)
          }.backgroundColor($r("app.color.yellow_aureus"))
          .padding({
            left: 10,
            right: 10,
            top: 1,
            bottom: 1
          })
          .borderRadius(4)
        }.height(68)
        .alignItems(HorizontalAlign.Start)
        .justifyContent(FlexAlign.SpaceBetween)
        .margin({ left: 12 })
      }.width("65%")
      .margin({ top: 14 }).alignItems(VerticalAlign.Top)

      Text(this.transaction?.ShareInfo)
        .width("65%")
        .margin({ top: 20 })
        .fontSize(13)
        .fontColor($r("app.color.yellow_aureus"))
    }
    .width("100%")
    .height(529)
    .backgroundImage($r("app.media.icon_deal_dialog_bg"))
    .backgroundImageSize({ width: "100%", height: "100%" })
  }
}
```

**Create a Controller to Link with the Decorator**:

```typescript
this.dealDialogController = new CustomDialogController({
  builder: DealInfoDialog({ transaction: this.transaction }),
  alignment: DialogAlignment.Center,
  customStyle: true,
  offset: { dx: 0, dy: -20 },
  maskColor: 0x66000000
})
```

**Usage**:

```typescript
this.dealDialogController?.open()
```

