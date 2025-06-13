When developing an ERP system, department tree lists are often used. The page mainly consists of a search box, a top-level department breadcrumb, and a multi-level department list. Each department list item is composed of a department name and a right-pointing arrow indicating the next level. Clicking on the department name area can pass department data back to the previous page, while clicking on the next-level arrow button can display the next-level department list and add the parent department to the top breadcrumb.

![](https://cdn.nlark.com/yuque/0/2025/jpeg/12749040/1736075821181-47c612d0-33de-44cd-8ef0-1b44dc2f9900.jpeg)

#### Load Department Tree Data
The department tree data consists of multiple department information objects. Each department object contains an array of subordinate departments, which may nest multiple child department objects. The JSON data used here is as follows:

```json
[
  {
    "DepartCode": 0,
    "DepartNo": "",
    "FirstName": "",
    "DepartName": "所有部门",
    "SystemTag": 0,
    "DepartType": 0,
    "DepartStatus": 0,
    "DepartLevel": 0,
    "DepartStatusName": "",
    "DepartmentListZtree": [
      {
        "DepartCode": 50001,
        "DepartNo": "002",
        "FirstName": "测试大区",
        "DepartName": "测试大区",
        "SystemTag": 1,
        "DepartType": 1,
        "DepartStatus": 1,
        "DepartLevel": 1,
        "DepartStatusName": "",
        "DepartmentListZtree": []
      }
    ]
  }
]
```

#### Customize the Top Input Box
The top input box is composed of a TextInput input box component and an Image component wrapped in a Stack component.

```typescript
/**
 * Build the search box
 */
@Builder
BuildSearch() {
  Stack() {
    TextInput({ text: this.depWord, placeholder: "搜索部门" })
      .height(36)
      .placeholderFont({ size: 14 })
      .placeholderColor($r("app.color.text_auxiliary"))
      .backgroundColor($r("app.color.tag_default"))
      .borderRadius(4)
      .padding({ left: 36 })
      .onChange((value) => {
        this.depWord = value
      })
    Image($r("app.media.icon_search")).width(14).height(14).margin({ left: 12 })
  }.align(Alignment.Start)
}
```

#### Customize the Department Breadcrumb
The department breadcrumb consists of a Row component wrapped in a Scroll component. When multiple levels are selected and exceed one screen, it can be scrolled horizontally. The Row component contains Text components for department names and Image components for right-pointing arrows. Clicking on a department name will display all sub-department data of the currently clicked department in the department list. The specific code is as follows:

```typescript
/**
 * Build the top department breadcrumb
 */
@Builder
BuildTopDep() {
  Scroll(this.topScroller) {
    Row() {
      ForEach(this.topDeps, (item: DepartBean, index) => {
        Text(item.DepartName).fontSize(14).fontColor($r("app.color.main_color"))
          .onClick(() => {
            this.topDeps.splice(index + 1, this.topDeps.length - index - 1)
            this.listDeps = item.DepartmentListZtree ?? []
          })
        if (index < this.topDeps.length - 1) {
          Image($r("app.media.icon_back"))
            .width(12).height(12).margin({ left: 4, right: 4 })
            .rotate({ angle: 180 })
        }
      })
    }
  }.scrollable(ScrollDirection.Horizontal) // Horizontal scrolling
  .scrollBar(BarState.Off)
  .margin({ top: 12 })
}
```

#### Draw the Department List
Each item in the department list consists of a department name and a next-level button. Clicking on the department name area will pass the current department back to the previous page, while clicking on the next-level button will display the next-level department list data. The code is as follows:

```typescript
// Build the department list
@Builder
BuildListDep() {
  List({ space: 1, scroller: this.listScroller }) {
    ForEach(this.listDeps, (item: DepartBean, index) => {
      ListItem() {
        Row() {
          Text(item.DepartName).fontSize(14).fontColor($r("app.color.text_two")).layoutWeight(1)
          // If there are sub-departments, display a right-pointing arrow
          if (item.DepartmentListZtree?.length ?? 0 > 0) {
            Image($r("app.media.icon_back"))
              .width(12).height(12).margin({ left: 4, right: 4 })
              .rotate({ angle: 180 })
              .margin({ right: 12 })
              .onClick(() => {
                // Click to jump to the next-level department
                this.topDeps.push(item)
                this.listDeps = item.DepartmentListZtree ?? []
                this.topScroller.scrollPage({ next: true }) // Scroll the breadcrumb to the far right
              })
          }
        }
        .margin({ left: 12 })
      }
      .backgroundColor(Color.White)
      .height(50)
      .width("100%")
      .align(Alignment.Start)
      .onClick(() => {
        // Click to return the selected department information to the previous page
        router.back({
          url: "",
          params: {
            depBean: item
          }
        })
      })
    })
  }.layoutWeight(1)
  .scrollBar(BarState.Off)
  .margin({ top: 8 })
}
```

#### Pass Department Data Back to the Previous Page
After clicking on a department name, call the router.back method to return to the previous page and pass the department data back to the previous page through the params. Note that the url is a required parameter; when returning to the previous page, you can directly pass an empty string.

```typescript
// Click to return the selected department information to the previous page
router.back({
  url: "",
  params: {
    depBean: item
  }
})
```

#### Receive Data Passed Back from the Page
Receive the passed-back data in the onPageShow method of the page lifecycle. You can obtain the passed-back parameters through router.getParams(), and the department object data through params['depBean'], where depBean corresponds to the key of the passed-back parameter.

```typescript
onPageShow(): void {
  let params = router.getParams() as Record<string, DepartBean>
  if (params) {
    this.depBean = params['depBean']
  }
}
```

The complete code is as follows:

```typescript
import { AppUtil, JSONUtil, LogUtil, ToastUtil } from '@pura/harmony-utils'
import { TAB_ARRAY } from '../constant/HomeConstant'
import { Tab } from '../model/HomeModel'
import { createWaterMarkView } from '../view/WaterMarkView'
import { CustomerPage } from './customer/CustomerPage'
import { HomePage } from './home/HomePage'
import { HousePage } from './house/HousePage'
import { MessagePage } from './message/MessagePage'
import { MinePage } from './mine/MinePage'
import { DepartBean } from '../model/DepartBean'
import { router } from '@kit.ArkUI'

@Entry
@Component
struct Index {
  @State currentIndex: number = 0
  @State depBean: DepartBean = new DepartBean()

  onPageShow(): void {
    let params = router.getParams() as Record<string, DepartBean>
    if (params) {
      this.depBean = params['depBean']
    }
  }

  build() {
    Tabs({ barPosition: BarPosition.End }) {
      ForEach(TAB_ARRAY, (item: Tab, index) => {
        TabContent() {
          if (index == 0) {
            HomePage({ depBean: this.depBean })
          } else if (index == 1) {
            HousePage()
          } else if (index == 2) {
            CustomerPage()
          } else if (index == 3) {
            MessagePage()
          } else {
            MinePage()
          }
        }.align(Alignment.Top)
        .tabBar(this.tabBuilder(item.tabText, index, item.tabSelectIcon, item.tabNormalIcon))
      })
    }
    .overlay(createWaterMarkView())
    .scrollable(false)
    .animationDuration(0)
    .backgroundColor($r("app.color.white"))
    .padding({ bottom: px2vp(AppUtil.getNavigationIndicatorHeight()) })
    .onChange((index) => {
      this.currentIndex = index
    })
  }

  @Builder
  tabBuilder(title: string, targetIndex: number, selectedImg: Resource, normalImg: Resource) {
    Column() {
      Image(this.currentIndex === targetIndex ? selectedImg : normalImg)
        .size({ width: 24, height: 24 })
      Text(title)
        .fontColor(this.currentIndex == targetIndex ? $r("app.color.main_color") : $r("app.color.text_auxiliary"))
        .fontSize(10)
        .padding(2)
    }
    .width('100%')
    .padding({ top: 8, bottom: 8 })
    .backgroundColor($r("app.color.white"))
    .borderWidth({ top: 0.5 })
    .borderColor($r("app.color.border_color"))
    .justifyContent(FlexAlign.Center)
  }
}
```

