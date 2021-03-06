# SDK接入指南（iOS）

## 1. 选择 SDK 安装方式

请确保您的 XCode 版本为 7.3 或者其后的版本。

GrowingIO 支持两种 iOS SDK 安装方式： 

A. 使用 CocoaPods 管理依赖 

1. 添加 `pod 'GrowingIO', '~> 1.4.2'` 到 Podfile 中； 
2. 执行 `pod update`，**不要用**`--no-repo-update`**选项**； 
3. 直接进入**安装文档的第五步**； 

B. 手动安装依赖 

1. 点击[1.4.2版](http://assets.growingio.com/sdk/GrowingIO-iOS-SDK-1.4.2.zip)下载 iOS SDK 
2. 进行安装文档的第二步

## 2. 导入 SDK

按照以下步骤将SDK导入到您的项目中。 1. 解压 iOS SDK 压缩文件； 2. 添加 Growing.h 和 libGrowing.a 添加到您的 iOS 工程中；

提醒:

* 记得勾选 "Copy items if needed"

## 3. 添加依赖

在您的工程项目中添加依赖

| 库名称 | 说明 |
| --- | --- |
| Foundation.framework | 基础依赖库 |
| Security.framework | 用于APP连接圈选页面SSL连接 |
| CoreTelephony.framework | 用于读取运营商名称 |
| SystemConfiguration.framework | 用于判断网络状态 |
| AdSupport.framework | 用于来源管理激活匹配 |
| libicucore.tbd | 用于APP连接圈选页面解析 |
| libsqlite3.tbd | 存储日志 |
| CoreLocation.framework | 用于读取地理位置信息（如果您的app有权限） |

提醒：

* 添加项目依赖库的位置在 项目设置target -&gt; 选项卡General -&gt; Linked Frameworks and Libraries  

## 4. 添加编译参数

在您的工程项目中添加编译参数

1. 找到 Linking 设置
2. 在 Other Linker Flags 中添加 -ObjC 参数，请注意大小写

提醒：

* Linking 设置位于 项目设置 target -&gt; 选项卡 Build Settings，左上角选择 All。

## 5. 添加初始化函数

在 AppDelegate 中引入`#import "Growing.h"`并添加启动方法

```text
- (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
      ...
       // 启动GrowingIO
      [Growing startWithAccountId:@"您的项目ID"];

      // 其他配置
      // 开启Growing调试日志 可以开启日志
      // [Growing setEnableLog:YES];
  }
```

请确保将代码添加在上面描述的位置，添加到其他函数中或者异步 block 中可能导致数据不准确。

## 6. 添加URL Scheme

把 URL Scheme 添加到您的项目，以便我们唤醒您的程序，进行圈选。

### URL Scheme 获取方式

1. 添加新产品：登录官网 -&gt; 点击项目选择框 -&gt; 点击“项目管理” -&gt; 点击“应用管理” -&gt; 点击“新建应用”-&gt;选择添加 iOS 应用 -&gt; 填写“应用名称“，点击下一步 -&gt;在第二段中标黄字体。 
2. 现有产品：登录官网 -&gt; 点击项目选择框-&gt; 点击“项目管理” -&gt; 点击“应用管理” -&gt; 找到对应产品的 URL Scheme。

![&#x9879;&#x76EE;&#x7BA1;&#x7406;](../../../.gitbook/assets/image%20%288%29.png)

### URL Scheme 的格式是 growing.xxxxxxxxxxxxxxxx。

1. 添加您的 URL Scheme（growing.xxxxxxxxxxxxxxxx）到项目中，URL Scheme 位于项目设置 target -&gt; 选项卡 Info -&gt; URL Types；
2. 在 AppDelegate 中调用函数 \[Growing handleUrl:\] 来接收 URL   

```text
- (BOOL)application:(UIApplication *)appation openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    if ([Growing handleUrl:url])
    {
        return YES;
    }
    ...
    return NO;
}
```

提醒：

* 如果您的 AppDelegate 中，实现了其中一个或者多个方法，请在已实现的函数中，调用 `[Growing handleUrl:]`:

```text
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(nullable NSString *)sourceApplication annotation:(id)annotation
- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString*, id> *)options
```

* 如果以上所有函数都未实现，则请实现以下方法并调用 `[Growing handleUrl:]`:

```text
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(nullable NSString *)sourceApplication annotation:(id)annotation
```

* 实际情况可能很复杂，请在调试时确保函数 `[Growing handleUrl:]` 会被执行到

### 下方各个配置项将影响统计的准确性，请务必仔细阅读，判断是否需要

## 7. 重要配置项

### 设置界面元素ID

当您的应用界面改版时，可能会导致无法准确地统计已经圈选的元素。

因此，对于应用中的主要流程涉及到的界面元素，建议您对它们设置固定的唯一ID，以保证数据的一致性。

具体要求

1. 主要流程指的是登录、注册、购买、发帖等操作步骤。
2. 设置ID的对象是界面的重要按钮等元素，如“注册”、“结算”、“发布”按钮。
3. 设置ID的方法如下：
4. 接口

   ```text
   @interface UIView(GrowingAttributes)
   @property (nonatomic, copy)   NSString* growingAttributesUniqueTag;
   @end
   ```

5. 代码写法：请加在viewWillAppear或者时机更早的函数里。

```text
-(void)viewWillAppear
{
    UIView * MyView;
    …
    MyView.growingAttributesUniqueTag = @"my_view”;
}
```

* ID只能设置为字母、数字和下划线的组合。
* 对于已经集成过旧版SDK并圈选过的应用，对某个元素设置ID后再圈选它，指标数值会从零开始计算，类似初次集成SDK后发版的效果，但不影响之前圈选的其它指标数据。如果不希望出现这种情况，请不要使用这个方法。

### 采集广告Banner数据

很多应用上方都有横向滚动的Banner广告。

对于这样的广告，如果要收集数据，请在响应点击的控件上添加如下代码

```text
UIView *view;
…
view.growingAttributesValue = 广告的唯一ID;
```

其中view是您的广告元素，请确保两点：

1. 对不同广告图，广告的唯一ID也不相同。
2. 响应点击的控件，与设置ID的控件是同一个

例如，当您的横向滚动广告共有3张广告图时，您可以在3个响应点击的View上分别设置不同的广告唯一ID，类似如下效果：

```text
view1.growingAttributesValue = @“ad1”;
view2.growingAttributesValue = @“ad2”;
view3.growingAttributesValue = @“ad3”;
```

此外，当您想采集一些可能没有文字的控件（比如UIImageView，UIView）时，也可以给属性growingAttributesValue赋值作为文字，用来在圈选的时候区分不同的内容。

### 页面别名

对于iOS应用，页面指的是`UIViewController`。

有些时候，对于完成某个功能的页面，统计时可能需要进一步细分。 比如，对于展示商品列表的页面，需要区分衣物类商品，以及食品类商品的两种列表的访问量。

为处理这种场景，我们提供了取别名的方法来区分这两种情况下的页面，接口如下：

```text
@interface UIViewController(GrowingAttributes)
@property (nonatomic, copy)   NSString* growingAttributesPageName;
@end
```

**我们举例来具体说明它的用法。**

1. 某个应用的商品列表页是用`ListViewController`实现的，所以默认的页面名称都是`ListViewController`。
2. 如果想区分衣物类商品列表和食品类商品列表，分别看它们的浏览量，可以这样做：

   ```text
   //ListViewController类的实现文件
   -(void)viewWillAppear
   {
    if (展示衣物类商品)
    {
        self.growingAttributesPageName = @"clothe_item_list";
    }
    else if (展示食品类商品)
    {
        self.growingAttributesPageName = @"food_item_list";
    }
   }
   ```

请注意

1. 必须在该 UIViewController 的 viewWillAppear 或者更早时机的函数中完成该属性的赋值操作。
2. 页面别名只能设置为字母、数字和下划线的组合。
3. 为查看数据方便，请尽量对iOS和安卓的同功能页面取不同的名称。

### 采集输入框数据

如果您需要采集应用内某个输入框内的文字（例如搜索框），请调用如下接口进行设置

`UIView * view; // view可以是UITextField, UITextView, UISearchBar ... view.growingAttributesDonotTrackValue = NO;`

其中，view代表要被采集的输入框。 当这个输入框失去焦点（包括应用退到后台），且输入框内容跟获取焦点前相比发生变化时，输入框内文字会被发送回GrowingIO。 请注意：对于密码输入框，即便标记为需要采集，SDK也会忽略，不采集它的数据。 

### Facebook广告SDK

如果使用了Facebook广告SDK，请务必添加以下代码来避免冲突，否则可能造成无法创建项目或者统计准确性问题。

请在main函数第一行调用下方函数。APP启动后，将不允许修改采集模式。

`[Growing setAspectMode:GrowingAspectModeDynamicSwizzling]`

### 采集H5页面数据

会自动采集H5页面的数据，不需要特殊配置。

### 采集GPS数据

如果您的应用有相应权限，我们将自动采集您的GPS数据。

### 启用Hashtag识别

对于 1.3.1 及以上 SDK 版本，添加以下方法以启用Hashtag识别：

```text
    // 设置为 YES, 将启用 HashTag
    + (void)enableHybridHashTag:(BOOL)enable;
```

## 8. 其他配置项

### 自定义维度

GrowingIO的数据分析工具提供了例如“应用版本”，“渠道”，“城市”，“设备型号”等等这些通用维度。但在使用过程中，有时无法满足用户对特定数据维度的分析要求。

为了能够让数据分析变得更加的灵活，我们提供了自定义维度的接口，使用者可以对各种对象设置属性。

**这些属性在作图时，将表现为维度。**

#### 用户属性

用户属性只能用来表示登录用户本身的属性，至少包括用户ID。

根据需求，可以用来指定用户的各种属性 1. 自然属性，比如性别、出生年月等。 2. 账户属性，比如等级、类型标签等。 3. 行为特征，比如是否有过购买记录之类。

用户属性被称为CS字段，最多支持十个，从CS1到CS10。

**我们举例说明它的用法。**

总计上传5个用户属性，分别是

CS1: user\_id:100324

CS2: company\_id:943123

CS3: user\_name:张溪梦

CS4: company\_name:GrowingIO

CS5: sales\_name:销售员小王

用户可以填入自定义的数据，该数据不会持久化，在数据变化的时候填入即可

```text
-(void)someMethod
{
    …
    [Growing setCS1Value:@"100324" forKey:@"user_id"];
    [Growing setCS2Value:@"943123" forKey:@"company_id"];
    [Growing setCS3Value:@"张溪梦" forKey:@"user_name"];
    [Growing setCS4Value:@"GrowingIO" forKey:@"company_name"];
    [Growing setCS5Value:@"销售员小王" forKey:@"sales_name"];
    …
}
```

**CS字段设置条件和限制**

1. CS 字段不能是和用户没有直接关系的属性，比如不能是订单 ID，商品 ID 等。
2. CS1 字段：在 GrowingIO 系统中用于识别注册用户的身份，因此 CS1 的 value 必须填写用户的唯一身份标示 ID。
3. CS2 字段：在 GrowingIO 系统中用于识别 SaaS 客户的租户，因此所有的 SaaS 用户必须填写租户的唯一身份标示 ID，非 SaaS 用户不做限定。
4. 对于未登录用户，不要设置任何CS字段。
5. 如果没有用到所有的CS字段，剩下的可以不设置。
6. 同一个CS字段，必须保持在各个平台意义相同。

**对于CS1字段设置时机的说明**

基本原则

1. 当App使用者的登录状态改变时设置CS1字段的值
2. 设置后，在下一次任意`viewDidAppear`被调用时，新的CS1字段的值才会被发送
3. 在CS1字段的值被发送前，重复设值会导致新的值取代旧的，旧值会被丢弃

用户手动登录 1. 如果有多个登录入口，在每一个入口登录成功后，都需要调用`[Growing setCS1Value:forKey:]`来设置用户的唯一标识。 2. 如果有第三方登录，成功登录后需要调用`[Growing setCS1Value:forKey:]`方法。

自动登录：App启动时，自动登录或者用户默认是登录状态，也需要调用`[Growing setCS1Value:forKey:]`方法

注册：有的App注册成功后，默认登录，这种情况下也需要调用`[Growing setCS1Value:forKey:]`方法。

退出：退出登录后，请勿继续上传CS字段，即使是空值也请勿上传。

**其他CS字段遵循相似的设置方法**

**在上传成功两小时后，您需要在「项目管理-项目配置-CS 配置中」进行字段配置和激活，配置成功后便可开始使用 CS 字段进行分析。配置过程请参考** [**属性数据\(CS\)上传配置文档**](https://docs.growingio.com/attribution-data.html)**。**

### 设置元素对象

如果元素自身的内容并不能代表具体的意义，可以使用元素对象来标识。

例如“加入购物车”按钮，可以设置成加入购物车的具体商品名称或ID。

```text
UIView * view;
...
view.growingAttributesInfo = "The New iPad";
```

### 忽略某元素

```text
UIView * view;
...
view.growingAttributesDonotTrack = YES;
```

其中，view是您需要忽略的元素。

### 自定义数据接收服务器

GrowingIO SDK 支持自定义行为数据接收服务器，用户可以选择数据先采集到自己的服务器做一些处理（比如审计）然后转发到 GrowingIO 服务器。详情参见 [CustomTrackerHost](https://github.com/growingio/help_site/tree/f4b4103b288205f6a9b13e0c4692f4d65a2ab386/Features/CustomTrackerHost.html)

### 在 App Store 提交你的应用

集成了 GrowingIO SDK 以后，默认会启用 IDFA，所以在向 App Store 提交应用时，需要：

1. 对于问题 **Does this app use the Advertising Identifier \(IDFA\)**，选择 YES。
2. 对于选项 **Attribute this app installation to a previously served advertisement**，打勾。
3. 对于选项 **Attribute an action taken within this app to a previously served advertisement**，打勾。

至此，您的SDK安装就成功了。您登录 GrowingIO 进入产品安装页面执行“数据检测”几分钟后就可以看到数据了。

 您可以在您的App中激活GrowingIO的SDK，并且进行数据定义。具体的激活以及数据定义的方法，请查看 [移动端圈选](https://docs.growingio.com/shu-ju-shi-shi/quan-xuan/yi-dong-duan-quan-xuan.html)

## 常见问题

### 1.为什么 GrowingIO 使用 IDFA?

GrowingIO 使用 IDFA 来做来源管理激活设备的精确匹配，让你更好的衡量广告效果。如果你不希望跟踪这个信息，可以选择不引入 AdSupport.framework 或者在用 Cocoapods 安装时使用 ‘GrowingIO/without-IDFA' subspec.

