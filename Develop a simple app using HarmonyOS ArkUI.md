An HarmonyOS application developed based on ArkUI. It calls the open API [玩android](https://www.wanandroid.com/) and implements functions such as simple page navigation, login, preservation of the login status, data display, and H5 page loading.![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fbc15436b91a4d26b31f999e5324c1f8~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=944&h=2038&s=246824&e=png&b=ffffff)

<h3 id="LsCN7">Bottom Navigation Bar on the Home Page</h3>
The bottom navigation is implemented using Tabs. The page composition of the Tabs component consists of two parts, namely TabContent and TabBar. TabContent is the content page, and TabBar is the navigation tab bar. Each TabContent should have a tab corresponding to its content, which can be configured through the tabBar property of TabContent.

```typescript
Tabs({ barPosition: BarPosition.End }) {
  TabContent() {
    HomeComponent()
  }.align(Alignment.Top)
  .tabBar(this.tabBuilder("Home", 0, $r("app.media.main_index_select_icon"), $r("app.media.main_index_icon")))

  TabContent() {
    SystemComponent()
  }.align(Alignment.Top)
  .tabBar(this.tabBuilder("System", 1, $r("app.media.main_house_select_icon"), $r("app.media.main_house_icon")))

  TabContent() {
    ProjectComponent()
  }.tabBar(this.tabBuilder("Project", 2, $r("app.media.main_message_select_icon"), $r("app.media.main_message_icon")))

  TabContent() {
    MyComponent()
  }.align(Alignment.Top)
  .tabBar(this.tabBuilder("My", 3, $r("app.media.main_me_select_icon"), $r("app.media.main_me_icon")))
}.scrollable(false)
.divider({ strokeWidth: 0.3 })
.animationDuration(0)
.onChange((index) => {
  this.currentIndex = index
})
```

The barPosition controls the position of the navigation bar. BarPosition.Start represents the top navigation bar, and BarPosition.End represents the bottom navigation bar;  
The divider can define the style of the navigation bar dividing line; animationDuration sets the switching animation duration of the navigation page; onChange listens to the navigation page switching.

HomeComponent, SystemComponent, ProjectComponent, and MyComponent are four custom page components. Switching the navigation can switch to the corresponding page.

<h3 id="TVOPp">Custom Component on the Home Page (HomeComponent)</h3>
The HomeComponent on the home page is composed of the Swiper component for the carousel image and the list component, which is used to display the article list. It is nested with a pull-to-refresh component PullToRefresh. The approximate structure is as follows:

```typescript
Pull-to-refresh/Load-more on pull-up
PullToRefresh({
  data: this.articleList,
  scroller: this.scroller,
  onRefresh: () => {
    // Callback for refreshing
  },
  onLoadMore: () => {
   // Callback for loading more
  },
  customList: () => {
   // Refresh the page content
    Scroll(this.scroller) {
       Column(){
          // Carousel image
          Swiper(){}
          // Article list
          ForEach(this.articleList, (item: ArticleData) => {
             ArticleListItem({ article: item })
          })
       }
    }
})
```

The third-party library [@ohos/pulltorefresh](https://ohpm.openharmony.cn/#/cn/detail/@ohos%2Fpulltorefresh) is used for the refresh component. The sub-component of the refresh component must be a scrollable component. Here, since the page consists of a carousel image and a list component, a Scroll component is nested on the outer layer to ensure the normal operation of the page. SystemComponent is a constructor @Builder within a custom component, which can abstract reusable UI elements into a method and is used here to display the layout of each item in the list.

Tips: The Scroll component and PullToRefresh need to share a controller Scroller.

<h3 id="IWwpA">Custom Component for the System (SystemComponent)</h3>
The system page is composed of top Tabs nested with Grid and a pull-to-refresh pulltorefresh list. The Grid component is a component provided by the system for displaying grids.  
The page structure is as follows:

```typescript
// Top navigation bar
Tabs({ barPosition: BarPosition.Start }){
  TabContent(){
    Column(){
    // Top grid component
      Grid(){}
      // Pull-to-refresh component
      PullToRefresh({
        customList: () => {
        // Scrollable component
          Scroll(){
            Column(){
            // Article list item
              ArticleListItem()
            }
          }
        }
      })
    }
  }
}
```

Below the navigation bar of this page, a grid component is nested, and below the grid component, a pull-to-refresh component is used to nest the list for display. When switching the navigation, call the interface to refresh the grid and list data, and selecting the grid component page will refresh the list data.

<h3 id="wGZ6t">Custom Component for the Project (ProjectComponent)</h3>
The project page is composed of Tabs directly nested with a pull-to-refresh component, and the pull-to-refresh component is nested with a Grid grid component. The page structure is as follows:

```plain
// Top navigation bar
Tabs({ barPosition: BarPosition.Start }){
   TabContent(){
     // Pull-to-refresh component
      PullToRefresh({
        customList: () => {
           Grid(this.scroller) {
             ForEach(this.projectList, (item: ArticleData) => {
               GridItem() {
                 // Style of the grid item
               }
             })
           }
           .columnsTemplate("1fr 1fr")
           .width("95%")
           .columnsGap(10)
           .scrollBar(BarState.Off)
           .padding(1)
        }
      })
   }
}
```

The columnsTemplate method can set the number of columns of the grid. One 1fr represents one column. columnsGap sets the spacing between lists, and scrollBar sets the style of the progress bar.

<h3 id="BDds0">Network Request @ohos/axios</h3>
The third-party library [@ohos/axios](https://ohpm.openharmony.cn/#/cn/detail/@ohos%2Faxios) is used for network requests and is simply encapsulated as follows:

```typescript
export class HttpUtils {
  private static instance: HttpUtils = new HttpUtils()
  private axiosInstance: AxiosInstance

  private constructor() {
    this.axiosInstance = axios.create({
      baseURL: 'https://www.wanandroid.com/',
      timeout: 10 * 1000,
      // Set the global request header
      headers: { "Content-Type": "application/x-www-form-urlencoded" },
      // Set the proxy
      // proxy: { host: "192.168.10.229", port: 8888, exclusionList: [] }
    });
    
    // Intercept the request data and handle some business logic, such as adding cookies to the request
    this.axiosInstance.interceptors.request.use(async (config: InternalAxiosRequestConfig) => {
      if (config.url != WanAndroidUrl.Login) {
        // If it is not the login interface, add cookies to the request header
        const cookies = await PFUtils.get<string>("cookies").then()
        config.headers["Cookie"] = cookies
      }
      console.log("Request data: ", JSON.stringify(config.headers))
      return config;
    });
    
    // Intercept the response data and handle some business logic, such as saving cookies
    this.axiosInstance.interceptors.response.use((response: AxiosResponse) => {
      console.log("Response data: ", response.config.url)
      console.log("Response data: ", JSON.stringify(response.data))

      if (response.config.url == WanAndroidUrl.Login) {
        // If it is the login interface, save the cookies
        const cookies = response.headers["set-cookie"] as string[]
        // Save the cookies to the cache
        PFUtils.put<string>("cookies", cookies.toString())
        // Save the login status
        PFUtils.put<boolean>("LoginState", true)
        // Notify other pages that the user has logged in successfully
        emitter.emit({ eventId: Constant.USER_LOGIN })
      }

      return response;
    }, (error: AxiosError) => {
      // Do something with the response error
      console.log("Request error data: ", JSON.stringify(error))
    });
  }
}
```

Project address: [HarmonyOS Version of WanAndroid](https://gitee.com/caojingCode/WanAndroid)

