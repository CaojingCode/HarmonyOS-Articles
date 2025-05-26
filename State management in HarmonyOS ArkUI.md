In the declarative UI programming framework, the UI is the running result of the program state. The user constructs a UI model where the runtime state of the application serves as the parameter. When the parameter changes, the UI, as the returned result, will also change accordingly. The re-rendering of the UI brought about by the changes in these runtime states is collectively referred to as the state management mechanism in ArkUI.

<h2 id="fFuIJ">Overview of Decorators</h2>
ArkUI provides a variety of decorators. According to the scope of influence of the state variables, all the decorators can be roughly divided into:

+ Decorators for managing the state owned by components: Component-level state management can observe changes within components and changes at different component levels, but it needs to be within the same page.
+ Decorators for managing the state owned by the application: Application-level state management can observe the state changes of different pages and even different UIAbility, which is the global state management within the application.

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57510b6d37714cdca111b4911da907a0~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1049&h=650&s=112180&e=png&b=fefefe)  
In the above figure, the decorators in the Components part are for component-level state management, and the Application part is for application state management.

**Managing the state owned by components**, that is, the state management at the Components level in the figure:

+ `@State`: A variable decorated with @State has the state of the component it belongs to and can serve as a data source for one-way and two-way synchronization of its subcomponents. When its value changes, it will cause the rendering refresh of related components.
+ `@Prop`: A variable decorated with @Prop can establish a one-way synchronization relationship with the parent component. The variable decorated with @Prop is mutable, but the modification will not be synchronized back to the parent component.
+ `@Link`: A variable decorated with @Link establishes a two-way synchronization relationship with the state variable of the parent component. The parent component will accept the synchronization of the modification of the variable decorated with @Link, and the update of the parent component will also be synchronized to the variable decorated with @Link.
+ `@Provide/@Consume`: Variables decorated with @Provide/@Consume are used to synchronize state variables across component levels (multiple layers of components), and can be bound through alias or property name without passing through the parameter naming mechanism.
+ `@Observed`: When @Observed decorates a class, the class that needs to observe the multi-layer nested scenario needs to be decorated with @Observed. Using @Observed alone has no effect and needs to be used together with @ObjectLink and @Prop.
+ `@ObjectLink`: A variable decorated with @ObjectLink receives an instance of a class decorated with @Observed and is applied to observe a multi-layer nested scenario and establish a two-way synchronization with the data source of the parent component.

**Managing the state owned by the application**, that is, the state management at the Application level in the figure:

+ `LocalStorage`: Page-level UI state storage, usually used for state sharing within UIAbility and between pages.
+ `AppStorage`: A special singleton LocalStorage object is created by the UI framework when the application starts and provides central storage for the UI state properties of the application.
+ `PersistentStorage`: Persistent storage of UI state, usually used in conjunction with AppStorage. The data stored in AppStorage is selected to be written to the disk to ensure that the values of these properties are the same when the application restarts as when the application is closed.
+ `Environment`: The environmental parameters of the device on which the application runs. The environmental parameters will be synchronized to AppStorage and can be used in conjunction with AppStorage.

**Other State Management Functions**

+ `@Watch` is used to listen to changes in state variables.
+ `$$ operator`: Provides a reference to a TS variable for built-in components, enabling the TS variable to remain synchronized with the internal state of the built-in component.

<h2 id="uM2LN">Managing the State Owned by Components</h2>
<h3 id="eWgVs">@State Decorator: State within Components</h3>
A variable decorated with @State, or a state variable, once it has a state property, is bound to the rendering of the custom component. When the state changes, the UI will have corresponding rendering changes.

_The variables decorated with @State have the following characteristics:_

+ A variable decorated with @State establishes one-way data synchronization with a variable decorated with @Prop in a subcomponent and two-way data synchronization with variables decorated with @Link and @ObjectLink.
+ The lifecycle of a variable decorated with @State is the same as the lifecycle of the custom component it belongs to.
+ Supports multiple data types: Allows Object, class, string, number, boolean, enum, Date types, and arrays of these types.
+ Internally private: A property marked as @State is a private variable and can only be accessed within the component.
+ Supports multiple instances: The internal state data of different instances of the component is independent.
+ Requires local initialization: All @State variables must be assigned an initial value. Keeping a variable uninitialized may cause undefined behavior of the framework. The initial value needs to be a meaningful value. For example, setting the value of a class type to `null` is meaningless and will cause a compilation error.
+ When creating a custom component, it supports setting the initial value through the state variable name: When creating a component instance, you can explicitly specify the initial value of the @State state property through the variable name.

**Decorating Variables of class Object Type**  
When the decorated data type is a class or Object, it can observe changes in its own assignment and changes in the assignment of its properties, that is, all properties returned by Object.keys(observedObject)

Create a Model object

```typescript
class Model {
  public value: string;

  constructor(value: string) {
    this.value = value;
  }
}
```

Initialize the object decorated with @State in the parent component, and the initialization of the parent component will overwrite the local initialization.

```typescript
@Entry
@Component
struct EntryComponent {
  build() {
    Column() {
      // The parameters specified here will overwrite the locally defined default values during the initial rendering. Not all parameters need to be initialized from the parent component.
      MyComponent({ count: 1, increaseBy: 2 })
        .width(300)
      MyComponent({ title: new Model('Hello World 2'), count: 7 })
    }
  }
}
```

Initialize the object decorated with @State locally, and the update of the @State variable will trigger the update of the component UI.

```typescript
@Component
struct MyComponent {
  @State title: Model = new Model('Hello World');
  @State count: number = 0;
  private increaseBy: number = 1;

  build() {
    Column() {
      Text(`${this.title.value}`)
        .margin(10)
      Button(`Click to change title`)
        .onClick(() => {
          // The update of the @State variable will trigger the update of the content of the above Text component.
          this.title.value = this.title.value === 'Hello ArkUI' ? 'Hello World' : 'Hello ArkUI';
        })
        .width(300)
        .margin(10)

      Button(`Click to increase count = ${this.count}`)
        .onClick(() => {
          // The update of the @State variable will trigger the update of the content of this Button component.
          this.count += this.increaseBy;
        })
        .width(300)
        .margin(10)
    }
  }
}
```



![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6809035a5a5c48728b9574055248e54b~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=294&h=291&s=161367&e=gif&f=57&b=067ff8)

<h3 id="lTE0u">@Prop Decorator: Parent-Child One-Way Synchronization</h3>
A variable decorated with @Prop can establish a one-way synchronization relationship with the parent component. The variable decorated with @Prop is mutable, but the change will not be synchronized back to its parent component.

_The variables decorated with @Prop have the following characteristics:_

+ Supports simple data types: Only supports `number`, `string`, `boolean`, `Object`, `class`, `enum` types;
+ Internally private: A property marked as `@Prop` is a private variable and can only be accessed within the component.
+ Supports multiple instances: The internal state data of different instances of the component is independent.
+ The @Prop decorator cannot be used in custom components decorated with @Entry.

```typescript
@Component
struct CountDownComponent {
  @Prop count: number = 0;
  costOfOneAttempt: number = 1;

  build() {
    Column() {
      if (this.count > 0) {
        Text(`You have ${this.count} Nuggets left`)
      } else {
        Text('Game over!')
      }
      // The variable decorated with @Prop will not be synchronized to the parent component.
      Button(`Try again`).onClick(() => {
        this.count -= this.costOfOneAttempt;
      })
    }
  }
}

@Entry
@Component
struct ParentComponent {
  @State countDownStartValue: number = 10;

  build() {
    Column() {
      Text(`Grant ${this.countDownStartValue} nuggets to play.`)
      // The modification of the data source of the parent component will be synchronized to the subcomponent.
      Button(`+1 - Nuggets in New Game`).onClick(() => {
        this.countDownStartValue += 1;
      })
      // The modification of the parent component will be synchronized to the subcomponent.
      Button(`-1  - Nuggets in New Game`).onClick(() => {
        this.countDownStartValue -= 1;
      })

      CountDownComponent({ count: this.countDownStartValue, costOfOneAttempt: 2 })
    }
  }
}
```

<h3 id="ifKhM">@Link Decorator: Parent-Child Two-Way Synchronization</h3>
A variable decorated with @Link in a subcomponent establishes a two-way data binding with the corresponding data source in its parent component. A variable decorated with @Link shares the same value as the data source in its parent component.

_The variables decorated with @Link have the following characteristics:_

+ Object, class, string, number, boolean, enum types, and arrays of these types. Supports the Date type.
+ Internally private: A property marked as `@Link` is a private variable and can only be accessed within the component.
+ Supports multiple instances: The internal state data of different instances of the component is independent.
+ Does not support internal initialization: When creating a new instance of a component, a value must be passed to the variable decorated with `@Link` for initialization, and internal initialization within the component is not supported.

```typescript
class GreenButtonState {
  width: number = 0;

  constructor(width: number) {
    this.width = width;
  }
}

@Component
struct GreenButton {
  @Link greenButtonState: GreenButtonState;

  build() {
    Button('Green Button')
      .width(this.greenButtonState.width)
      .height(40)
      .backgroundColor('#64bb5c')
      .fontColor('#FFFFFF，90%')
      .onClick(() => {
        if (this.greenButtonState.width < 700) {
          // Update the property of the class, and the change can be observed and synchronized back to the parent component.
          this.greenButtonState.width += 60;
        } else {
          // Update the class, and the change can be observed and synchronized back to the parent component.
          this.greenButtonState = new GreenButtonState(180);
        }
      })
  }
}

@Component
struct YellowButton {
  @Link yellowButtonState: number;

  build() {
    Button('Yellow Button')
      .width(this.yellowButtonState)
      .height(40)
      .backgroundColor('#f7ce00')
      .fontColor('#FFFFFF，90%')
      .onClick(() => {
        // The simple type of the subcomponent can be synchronized back to the parent component.
        this.yellowButtonState += 40.0;
      })
  }
}

@Entry
@Component
struct ShufflingContainer {
  @State greenButtonState: GreenButtonState = new GreenButtonState(180);
  @State yellowButtonProp: number = 180;

  build() {
    Column() {
      Flex({ direction: FlexDirection.Column, alignItems: ItemAlign.Center }) {
        // Simple type is synchronized from @State in the parent component to @Link in the subcomponent.
        Button('Parent View: Set yellowButton')
          .width(312)
          .height(40)
          .margin(12)
          .fontColor('#FFFFFF，90%')
          .onClick(() => {
            this.yellowButtonProp = (this.yellowButtonProp < 700) ? this.yellowButtonProp + 40 : 100;
          })
        // Class type is synchronized from @State in the parent component to @Link in the subcomponent.
        Button('Parent View: Set GreenButton')
          .width(312)
          .height(40)
          .margin(12)
          .fontColor('#FFFFFF，90%')
          .onClick(() => {
            this.greenButtonState.width = (this.greenButtonState.width < 700) ? this.greenButtonState.width + 100 : 100;
          })
        // Initialize @Link for the class type.
        GreenButton({ greenButtonState: $greenButtonState }).margin(12)
        // Initialize @Link for the simple type.
        YellowButton({ yellowButtonState: $yellowButtonProp }).margin(12)
      }
    }
  }
}
```

1. Click the Button in the subcomponents GreenButton and YellowButton, and the subcomponents will change accordingly, and the changes will be synchronized to the parent component. Since @Link is two-way synchronized, the changes will be synchronized to @State.
2. When clicking the Button in the parent component ShufflingContainer, the @State changes and will also be synchronized to @Link, and the subcomponents will also have corresponding refreshes.

<h2 id="Sqkxr">Managing the State Owned by the Application</h2>
<h3 id="VFpjl">LocalStorage: Page-Level UI State Storage</h3>
LocalStorage is a page-level UI state storage. The parameters received through the @Entry decorator can share the same LocalStorage instance within the page. LocalStorage supports state sharing between multiple pages within a UIAbility instance.  
LocalStorage is an in-memory "database" provided by ArkTS for storing page-level state variables.

+ The application can create multiple LocalStorage instances. The LocalStorage instances can be shared within the page or across pages and within a UIAbility instance through the GetShared interface.
+ The root node of the component tree, that is, the @Component decorated with @Entry, can be assigned a LocalStorage instance, and all subcomponent instances of this component will automatically obtain access to this LocalStorage instance.
+ A component decorated with @Component can access at most one LocalStorage instance and AppStorage. A component not decorated with @Entry cannot be independently assigned a LocalStorage instance and can only accept the LocalStorage instance passed by the parent component through @Entry. One LocalStorage instance can be assigned to multiple components on the component tree.
+ All properties in LocalStorage are mutable.

According to the different synchronization types with the component decorated with @Component, LocalStorage provides two decorators:

+ _@LocalStorageProp_: A variable decorated with @LocalStorageProp establishes a one-way synchronization relationship with a given property in LocalStorage.
+ _@LocalStorageLink_: A variable decorated with @LocalStorageLink establishes a two-way synchronization relationship with a given property in LocalStorage created in @Component.

<h3 id="A6wDk">AppStorage: Global UI State Storage for the Application</h3>
AppStorage is the global UI state storage for the application, which is bound to the process of the application. It is created by the UI framework when the application starts and provides central storage for the UI state properties of the application.

Different from AppStorage, LocalStorage is at the page level and is usually used for data sharing within the page. And AppStorage is the global state sharing at the application level, which is equivalent to the "hub" of the entire application. The persistent data PersistentStorage and the environmental variable Environment are both transferred through AppStorage to interact with the UI.

<h4 id="WDm5x">@StorageProp</h4>
+ One-way synchronization: From the corresponding property of AppStorage to the state variable of the component.
+ Object, class, string, number, boolean, enum types, and arrays of these types.
+ Local modification of the component is allowed, but once the given property in AppStorage changes, it will overwrite the local modification.
+ @StorageProp does not support initialization from the parent node. It can only be initialized with the property corresponding to the key in AppStorage. If there is no corresponding key, it will be initialized with the local default value.

<h4 id="UzR0P">@StorageLink</h4>
The variable decorated with `@StorageLink(key)` is the internal state data of the component. When these state data are modified, the `build()` method of the component will be called for UI refresh. The component establishes a two-way data binding

