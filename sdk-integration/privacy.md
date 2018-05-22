# 安全性说明

GrowingIO除了先进的数据分析理念和技术外，还从硅谷带来了对于数据安全极为重视的态度。我们将在每一个环节保证您的数据安全。在下面的说明中，我们将分别介绍「JS SDK」和「移动端SDK」的工作方式，以及安全性能。

## 1.工作方式与采集安全

### JS SDK

GrowingIO 的 JS SDK 是运行于用户网站一段机器代码，使用「cookies」——是一些在用户的计算机上储存的文本文件，来帮助网站分析用户是如何使用网站的。这些由 cookie 生成的信息包括用户的使用情况（包括用户的 IP 地址）会被传输并存储在 GrowingIO 的云端服务器上。GrowingIO 通过使用这些信息来评估客户网站用户的使用情况，生成网站使用报告，提供跟网站行为相关的服务。

GrowingIO JS SDK 会在用户加载网页后自动执行，开始收集用户的行为数据，建议集成时把代码放在`<head> </head>`之间。JS SDK 采用异步方式加载，不会影响网站自身的加载数据。

目前 SDK 主要采集三类数据:

* 访问数据:网站访客在何时何地访问了哪个网页，收集信息包括域名、页面路径、浏览器、操作系统、屏幕分辨率、访问来源、用户唯一标识 ID、访问唯一标识 ID、访问时间、页面标题。如果客户集成时设置了自定义维度，也会一并收集。
* 内容数据:当用户访问网站时，用户看到的内容即页面出现的元素，会被自动采集，包括内容所在的页面信息、元素的标记 ID、文本内容、超链接、位置信息。
* 行为数据:用户在网站上的交互行为，比如点击链接、提交表单、修改选择，都会被自动采集，内容包括交互元素的页面信息、交互行为类型、交互元素的标记 ID、交互元素的内容、交互元素的超链接、交互元素的位置信息。GrowingIO 不采集任何用户在表单里输入的信息，唯一例外是搜索框。

### 移动端SDK

移动端SDK需要在应用打包时，被加载在您的应用当中。GrowingIO的「移动端SDK」会随着客户应用的启动而自动开始进行用户行为数据。当用户关闭应用时，SDK会随着客户应用的关闭而关闭，不会在后台做任何额外动作。

#### 时间延迟

经过我们反复的测量，移动端SDK的数据发送仅仅会带来10ms以内的时间延迟，用户感知不到任何的差异。GrowingIO真正的做到了用户无感知的数据采集，不会对应用的用户体验带来任何的降低。

#### 稳定性

我们非常注重SDK的稳定性，每个版本的SDK我们都会进行大量的稳定性测试，以确保您的应用一如既往的稳定。从目前客户集成SDK的结果来看，应用的崩溃率没有因为集成而提高。

#### SDK不会被普通用户（消费者）唤醒

如果没有您的GrowingIO账户，普通用户（消费者）无法唤醒圈选界面。  
GrowingIO目前主要通过两种方式打开SDK的圈选界面。

**iOS**

1. 对应用打包时，修改SDK配置项，把SDK圈选界面默认打开。  
2. 通过在浏览器中输入特殊的字符串，调起SDK以及其中的圈选界面。

**安卓**

1. 对应用打包时，修改SDK配置项，把SDK圈选界面默认打开。  
2. 通过GrowingIO的BI应用调用SDK以及其中的圈选界面。

#### 移动端SDK采集的数据类型

与「JS SDK」一样，移动端SDK主要采集三类数据：访问数据，内容数据，行为数据。并且，不采应用文本框里的数据，也就不会主动记录普通用户填写的账户/电话/银行卡等隐私信息，在采集环节保证安全。
## 2.GrowingIO SDK支持GDPR 
### 概述
作为一家专注于用户行为数据分析的大数据公司，GrowingIO始终注重用户隐私数据的保护，在本次发布的版本中，GrowingIO SDK（Android、iOS、JS）针对欧盟区的一般数据保护法(GDPR)提供了以下的API供开发者调用。

GrowingIO SDK提供默认是否开启数据采集的配置项
GrowingIO SDK提供关闭或开启全局数据采集的接口，开发者可在APP中任何场景时调用该接口
GrowingIO SDK提供获取该设备的设备ID接口，开发者可配合数据侧提供的接口删除或导出该设备的行为数据
### 接口
接口名称 | Android|iOS| JS
---|---|---|---
全局配置项 | .disableDataCollect()|  |  
关闭或开启全局数据采集 | // 不采集数据<br>GrowingIO.getInstance().disableDataCollect();<br>// 收集数据<br>GrowingIO.getInstance().enableDataCollect()| disableDataCollect <br><br><br>enableDataCollect | // 开启gdpr，停止数据采集<br>window.gio('config',{"dataCollect":"false"});<br>// 关闭gdpr，开始数据采集<br>window.gio('config',{"dataCollect":"true"});<br>放在send之前
获取访问用户ID | GrowingIO.getInstance().getVisitUserId();|getVisitUserId |  window.gio('getVisitUserId');<br>放在send之后
#### Andorid 样例

```
GrowingIO.startWithConfiguration(this, new Configuration()
        .disableDataCollect()        // 开启GDPR， 不采集数据。 默认采集
        .useID()
        .trackAllFragments());
//  不采集数据
GrowingIO.getInstance().disableDataCollect();
// 收集数据
GrowingIO.getInstance().enableDataCollect();
// 获取设备Id
GrowingIO.getInstance().getVisitUserId();
```
#### iOS 样例
```
// 开启GDPR
[Growing disableDataCollect];
 
// 关闭GDPR
[Growing enableDataCollect];
 
// 获取设备ID
NSString *viId = [Growing getVisitUserId];
```
#### JS 样例
```
// 开启gdpr，停止数据采集
 
window.gio('config',{"dataCollect":"false"});
 
// 关闭gdpr，开始数据采集
window.gio('config',{"dataCollect":"true"});
 
获取访问用户ID
window.gio('getVisitUserId')   放在send之后
```



