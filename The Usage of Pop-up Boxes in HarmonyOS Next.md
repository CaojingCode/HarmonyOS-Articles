![](https://cdn.nlark.com/yuque/0/2025/png/12749040/1742293177670-f5eb2bda-7107-4734-89f6-e8a53a965a5d.png?x-oss-process=image%2Fformat%2Cwebp)

### Overview and Classification of Pop-up Boxes in HarmonyOS Next
A pop-up box is a modal window that temporarily displays information or actions requiring user attention while maintaining the current contextual environment. Users must complete relevant interactive tasks within the modal pop-up before exiting the modal mode. Pop-up boxes can be independent of any component binding, and their content is typically composed of various components (such as text, lists, input fields, images, etc.) to achieve layout. ArkUI currently provides two categories of pop-up components: custom-style and fixed-style.  

#### Custom Pop-up Boxes
Developers need to pass custom components into the pop-up box based on the usage scenario to implement personalized content. These mainly include:  

+ Basic custom pop-up box (`CustomDialog`)  
+ UI-component-independent global custom pop-up box (`openCustomDialog`)

#### Fixed-style Pop-up Boxes
Developers can use fixed-style pop-up boxes by specifying the text content and button operations to achieve simple interactive effects. These mainly include:  

+ Alert dialog (`AlertDialog`)  
+ Action sheet (`ActionSheet`)  
+ Picker dialog (`PickerDialog`)  
+ Dialog (`showDialog`)  
+ Action menu (`showActionMenu`)

This article focuses on the usage of custom pop-up boxes (fixed-style pop-ups can refer to the official documentation). There are two main implementation methods for custom pop-ups:

1. Basic custom pop-up box (`CustomDialog`) (not recommended)  
2. UI-component-independent global custom pop-up box (`openCustomDialog`) (recommended)

### Basic Custom Pop-up Box (`CustomDialog`)
`CustomDialog` is a custom pop-up box suitable for user interaction scenarios such as advertisements, rewards, warnings, and software updates. It displays custom pop-ups through the `CustomDialogController` class. When using pop-up components, custom pop-ups are preferred for easy customization of styles and content.  

#### 1. `@CustomDialog` Decorator
Use the `@CustomDialog` decorator to define a custom pop-up box, where you can customize the pop-up content (e.g., adding confirm/cancel buttons and data functions):  

#### 2. `CustomContentDialog`
Use `CustomContentDialog` to customize pop-up boxes, supporting both custom content areas and operation-area button styles (available from API version 12).  



| **Name** | **Type** | **Required** | **Decorator Type** | **Description** |
| --- | --- | --- | --- | --- |
| controller | CustomDialogController | Yes | - | Pop-up controller.    _Note_: Not decorated with `@Require`, so parameter validation is not enforced during construction. |
| contentBuilder | () => void | Yes | @BuilderParam | Pop-up content. |
| primaryTitle | ResourceStr | No | - | Pop-up title. |
| secondaryTitle | ResourceStr | No | - | Pop-up auxiliary text. |
| localizedContentAreaPadding | LocalizedPadding | No | - | Inner padding of the pop-up content area. |
| contentAreaPadding | Padding | No | - | Inner padding of the pop-up content area (invalid if `localizedContentAreaPadding` is set). |
| buttons | ButtonOptions[] | No | - | Buttons in the pop-up operation area (supports up to 4 buttons). |
| theme | Theme / CustomTheme | No | - | Theme information (can be a `CustomTheme` or a `Theme` instance from `onWillApplyTheme`). |
| themeColorMode | ThemeColorMode | No | - | Custom pop-up light/dark mode. |


`CustomContentDialog` supports custom content with `contentBuilder`, `buttons`, etc. Similar to the `@CustomDialog` decorator, it can include buttons and data. For fully customized styles, only the `contentBuilder` parameter is needed:  



```typescript
import { CustomContentDialog } from '@kit.ArkUI'

@Entry
@Component
struct Index {
  dialogController: CustomDialogController = new CustomDialogController({
    builder: CustomContentDialog({
      primaryTitle: 'Title',
      secondaryTitle: 'Auxiliary text',
      contentBuilder: () => {
        this.buildContent();
      },
      buttons: [
        { 
          value: 'Button 1',
          buttonStyle: ButtonStyleMode.TEXTUAL, 
          action: () => {
            console.info('Callback when the button is clicked')
          }
        },
        {
          value: 'Button 2',
          buttonStyle: ButtonStyleMode.TEXTUAL,
          role: ButtonRole.ERROR
        }
      ],
    }),
  });

  build() {
    Column() {
      Button("Show custom content pop-up")
        .onClick(() => {
          this.dialogController.open()
        })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
  
  // Custom pop-up content area
  @Builder
  buildContent(): void {
    Column() {
      Text('Content area')
    }
    .width('100%')
  }
}
```

### UI-component-independent Global Custom Pop-up Box (openCustomDialog)  
The official recommendation is to use `openCustomDialog` over `CustomDialogController` for implementing custom pop-up boxes. Due to multiple limitations of `CustomDialogController` (e.g., no support for dynamic creation or refresh), in relatively complex scenarios, it is recommended to use the `openCustomDialog` interface provided by the `PromptAction` object obtained from `UIContext` to implement custom pop-ups.  

There are two parameter passing methods for creating custom pop-up boxes with `openCustomDialog`:  

:::info
#### openCustomDialog (with ComponentContent)
Encapsulating content via `ComponentContent` decouples from the UI, enabling more flexible calls to meet developers' encapsulation needs. It offers stronger flexibility with fully customizable styles and supports dynamically updating pop-up parameters using `updateCustomDialog` after the pop-up is opened.  

#### openCustomDialog (with builder)
Compared to `ComponentContent`, `builder` must bind to the context, introducing some UI coupling. This method uses default pop-up styles, suitable for scenarios where developers want consistency with the system's default pop-up aesthetics.  

:::

#### 1. Initialize Pop-up Configuration
+ Obtain the `PromptAction` object and initialize pop-up configurations (e.g., modify position, animation, and other settings).  
+ Create `ComponentContent` to define the pop-up content, where `wrapBuilder(radioDialogView)` encapsulates custom components.



#### 2. Open the Custom Pop-up Box
Calling the `openCustomDialog` interface opens a pop-up with `customStyle: true` by default, meaning the content style fully follows the `contentNode` customization.  



#### 3. Close the Custom Pop-up Box
After closing the pop-up, call the `dispose()` method of `ComponentContent` to release the corresponding resources.  
_Tip_: The `ComponentContent` object to be closed must be passed in when closing the pop-up.  



#### 4. Update the Custom Pop-up Content
To update the content of custom components in the pop-up, use the `update()` method provided by `ComponentContent`, passing in the same data object used during display.  



#### 5. Complete Code
```typescript
// Import ArkUI dialog operation modules and basic service error types
promptAction?: PromptAction

// Private constructor
private constructor() {
  console.info('Creating custom dialog singleton');
}

/**
 * Get the singleton instance (static method)
 * @returns The global unique instance
 */
public static getInstance(): RadioAppDialog {
  if (!RadioAppDialog.instance) {
    RadioAppDialog.instance = new RadioAppDialog();
  }
  return RadioAppDialog.instance;
}

/**
 * Reset the singleton instance (for testing or special scenarios)
 */
public static resetInstance(): void {
  RadioAppDialog.instance = new RadioAppDialog();
}


// Set dialog content component (supports chaining)
init(context: UIContext, radioSelectBean: RadioSelectBean): RadioAppDialog {
  this.contentNode = new ComponentContent(context, wrapBuilder<[RadioSelectBean]>(radioDialogView), radioSelectBean);
  this.promptAction = context.getPromptAction();
  this.options = {
    alignment: DialogAlignment.Bottom,
    transition: TransitionEffect.move(TransitionEdge.BOTTOM)
      .animation({ duration: 300 }),
  }
  return this
}

// Set dialog options (supports chaining)
setOptions(options: promptAction.BaseDialogOptions): RadioAppDialog {
  this.options = options;
  return this;
}

// Update dialog data
updateData(obj?: Object) {
  this.contentNode?.update(obj)
}


// Show the custom dialog
show() {
  if (this.contentNode !== null) {
    this.promptAction?.openCustomDialog(this.contentNode, this.options)
      .then(() => {
        console.info('Custom dialog opened successfully')
      })
      .catch((error: BusinessError) => {
        // Error handling: get error code and message
        let message = (error as BusinessError).message;
        let code = (error as BusinessError).code;
        console.error(`Dialog opening failed. Error code: ${code}, message: ${message}`);
      })
  }
}

// Close the custom dialog
close() {
  if (this.contentNode !== null) {
    this.promptAction?.closeCustomDialog(this.contentNode)
      .then(() => {
        this.contentNode?.dispose()
        console.info('Custom dialog closed successfully')
      })
      .catch((error: BusinessError) => {
        let message = (error as BusinessError).message;
        let code = (error as BusinessError).code;
        console.error(`Dialog closing failed. Error code: ${code}, message: ${message}`);
      })
  }
}

// Update dialog configuration
updateDialog(options: promptAction.BaseDialogOptions) {
  if (this.contentNode !== null) {
    this.promptAction?.updateCustomDialog(this.contentNode, options)
      .then(() => {
        console.info('Dialog configuration updated successfully')
      })
      .catch((error: BusinessError) => {
        let message = (error as BusinessError).message;
        let code = (error as BusinessError).code;
        console.error(`Dialog update failed. Error code: ${code}, message: ${message}`);
      })
  }
}
```

