# Building a Chat App in Swift Using Multipeer Connectivity Framework
## 使用Multipeer Connectivity框架构建一个聊天app

***

> * 原文链接 : [Building a Chat App in Swift Using Multipeer Connectivity Framework](http://www.appcoda.com/chat-app-swift-tutorial/)
> * 原文作者 : [GABRIEL THEODOROPOULOS](http://www.appcoda.com/author/gabrielth/)
> * 译者 : [yrq110](https://github.com/yrq110)

***

`Multipeer Connectivity`-`多点连接`

在iOS编程中，总有一些SDK相比其它而言更有意思更吸引开发者的注意，Multipeer Connectivity就是其中一个。如你所知，MPC框架并不是iOS8中的新东西，
在7中就出现了。之前我写过一些相关的教程，不过说实话，我很惊讶大家对它如此的感兴趣。现在过了一段时间后，又写了一篇新的教程，相信这其中还
有一些东西需要说明下。

你也许会疑惑为何会说这个有些古老的话题，而不是iOS8中的新特性。好吧，我这么做有三个原因：

1. 很多读者都通过邮件告诉我想知道在multipeer connectivity框架中如何处理多种不同的任务，在回答这些问题时，我发现了一些以前没注意到的问题，而且有些是不容易发现的。
2. 在之前的教程中我使用了一个默认的、创建好的视图控制器来实现用户的邀请与建立连接。读者反馈想要看看手动的实现，这也是接下来要做的。
3. 我觉得使用Swift来实现MPC是很有用的，对初学者也很有帮助。

![](http://www.appcoda.com/wp-content/uploads/2015/01/mpc-swift-featured.jpg)

自multipeer connectivity诞生以来，为大量开发者提供了很多实现新点子的可能性。使用一种简单的方式去进行设备间的连接是很诱人的，这也是为何开发者会乐于将它集成在app中。不过如果你还没用过MPC框架的话，我必须提醒你：它有时不是你所期望的那么稳定、健壮，我在我的项目中发现过类似问题，有些开发者也告知过我同样的问题。MPC使用蓝牙和WiFi来连接附近的设备，虽然这听起来很棒很有前景，但是有时由于通信过程中的问题，连接也会失败或变得很慢。若你要传输重要数据的话，这种情况是需要重点考虑的。我建议你使用一个备用的通信解决方案(像一个web服务)，确保有一个备选途径，当MPC失效时app也能继续工作。尽管我这么说，我依然相信MPC是一个对所有iOS开发者都很不错的工具，值得俺再写一篇教程。

我不打算深入MPC的细节中。如果想尝尝鲜的话，看看这篇[教程]()。否则的话，看看我总结的这个MPC的概述。

MPC中包含4个重要的类，分别对应4个概念:

1. Peer-节点 (MCPeerID): 一个节点实则就是一台设备，这通常是需要优先设置的，因为它会作为下一个类实例初始化时的一个参数使用。它包含一个重要的属性 - displayName，这是一台设备对于附近的节点所显示的名字。
2. Session-会话 (MCSession class): 这是两个节点间建立的连接，前提是第一个节点邀请了第二个，并且第二个接受了邀请。一个会话只与两台设备有关。一个第三方设备不能连接进一个已存在的会话。
3. Browser-搜索器 (MCNearbyServiceBrowser class): 这个类的方法被用来寻找附近的设备并邀请他们加入一个会话。作为前提条件，其他设备必须先广播他们自己。在这篇教程中用使用这个类去手动邀请其他设备。
4. Advertiser-广播器 (MCNearbyServiceAdvertiser): 这个类负责广播一个设备，控制这个设备对其他设备可见或不可见，接受或拒绝其他节点的会话邀请。


MPC的逻辑是很简单的:  一台设备(一个节点)使用它的搜索器，寻找周围的其他设备。一台设备必须广播它自己，否则不会被发现。一旦它在搜索时发现了一个或多个节点，就会向其发送一个会话连接的邀请。这个邀请可以在发现周围节点时自动发送，也可以用户手动决定是否发送，这完全取决于应用中国是如何实现的。无论任何情况，只要邀请被接受，会话就会建立，两个节点间就可以收发数据、资源与流。

本文中我的目标是给你演示如果编程实现搜索、邀请与连接其它节点。记住，之后看到的只是其中一种代码实现的方式，很显然，你可以通过使用MPC中提供的所有工具，以你自己想要的方式去实现适合你app的功能。下面的内容只是一个MPC框架实现的例子，实现了精简的从头到尾的编程过程，没有使用任何额外的SDK。我希望在这篇教程最后你们能找到一些所寻找的答案。

最后，如果你从未使用过MPC框架，我建议你扫一眼苹果的官方文档与我之前的教程。哦了不浪费时间了，开始编程吧。

**目录**
* 关于Demo App
* 一个自定义类
* 搜索节点
* 显示搜索到的节点
* 处理广播
* 邀请节点
* 连接到会话
* 一个发送数据的简便方法
* 在节点间发送数据]
* 接收数据
* 终止聊天
* 最后修整
* 测试app

## 关于Demo App

来稍微说下demo应用，一个聊天app。好吧这与之前MPC教程中的聊天app有很大的共同之处，不过让我解释一下:不管我怎么想，都找不到一个比这个更合适的类型，聊天app一直会是首选，我相信在教程的最后你会认同我的看法的。

开始工程在[这里](https://www.dropbox.com/s/adbuyk2j1vgmbmd/MPCRevisitedTemplate.zip?dl=0)下载，将在这个的基础上进行编程，下好后先大致浏览下，以便能快速的上手。

如我所讲，将构建一个聊天app，这个app分为两个视图控制器，第二个会呈现给用户。如果你看看工程中的Interface Builder会发现第一个视图控制器中有一个列表视图，在这个列表视图中将要显示的是MPC框架所搜索到的所有附近的设备，列表中的节点是动态刷新的，消失的已存在节点与新发现的节点都会动态显示在列表中。这里还有一个包含一个按钮的工具栏，后面会使用这个按钮去打开和关闭设备的可发现性，针对MPC中广播的特性所设计。

在点击一个列表项节点时，会询问另一端的用户是否开始聊天。如果他拒绝则什么都不会发生，如果接受的话会在屏幕中显示第二个视图控制器，名为ChatViewController，那么两台设备就可以交换文本信息了。这里包含一个用来撰写信息的textfield和一个显示所有信息的列表视图。在顶部工具栏的按钮用来终止聊天。差不多就是所有细节了，来看看具体的实现吧。

在看app的最终效果图之前，我还要说一件事。尽管我们可以用MPC连接7个甚至更多的设备，在这里只连接一个节点进行多次测试，因为使用会话来连接节点与发现和列出所有附近节点是两码事。在下面的部分你会看到逐一实现的步骤，现在先感受一下吧:

![](http://www.appcoda.com/wp-content/uploads/2015/01/t27_2_ask_chat_alert.png)

## 一个自定义类

知道这个工程是做什么的后让我们来创建一个自定义类，在这个类中会处理所有需要实现的MPC框架方法，在最后我们会创建一个使用代理模式的新协议。

当务之急，在Xocde中按住Cmd与N组合键，在出现的引导视图中，选择创建一个新的Cocoa Touch Class:

![](http://www.appcoda.com/wp-content/uploads/2015/01/t27_4_class_template_1.png)

第二步中，使新的类继承自NSObject类，命名为MPCManager

![](http://www.appcoda.com/wp-content/uploads/2015/01/t27_5_class_template_2.png)

跟着向导走，确保你正在操作的是MPCManager.swift文件。

在代码的第一行，需要导入multipeer connectivity框架，因此在文件头部添加如下代码:
```swift
import MultipeerConnectivity
```
接着来声明需要用到的MPC框架中的类的对象，在类中的顶部添加如下行:
```swift
var session: MCSession!
 
var peer: MCPeerID!
 
var browser: MCNearbyServiceBrowser!
 
var advertiser: MCNearbyServiceAdvertiser!
```
除了上面那些，还需要声明两个以后要用到的变量:
```swift
var foundPeers = [MCPeerID]()
 
var invitationHandler: ((Bool, MCSession!)->Void)!
```
在foundPeers数组中会存放所有发现的节点。注意此时与这些节点间不会构建任何连接，只需要知道周围有这些节点。这个数组在声明的同时进行了初始化，因此不需要额外设置为nil，期望数组在发现新对象时已经准备就绪。

invitationHandler是一个完整声明的处理程序，不过现在不会管它，等用的时候再说。

接下来需要使自定义类满足特定的MPC协议。这些协议的委托方法使我们可以去处理multipeer connectivity的相关操作，比如搜索、广播、会话等。如下修改类的第一行，添加协议:
```swift
class MPCManager: NSObject, MCSessionDelegate, MCNearbyServiceBrowserDelegate, MCNearbyServiceAdvertiserDelegate
```
我觉得没必要解释每个协议的用途了。

现在来创建一个初始化器来初始化所有的MPC对象。一个一个来，从peer对象开始，它代表了，在初始化时需要提供一个显示的名称(display name)，这个显示名称将会对其他节点可见，并且可以设置为任意字符串。为了使事情简单点，我直接将设备名称作为显示名称，不过我不建议在一个实际的app中这样做，也许你应该让用户自己来输入想要的名字，或者使用其他方法来生成唯一的节点名称。在代码中，如下进行初始化:
```swift
override init() {
    super.init()
 
    peer = MCPeerID(displayName: UIDevice.currentDevice().name)
}
```
通过UIDevice类获得设备名称。

在peer对象成功初始化后，接着进行其他对象的操作。注意peer必须最先被初始化，因为之后的对象都需要使用它。session对象:
```swift
override init() {
    ...
 
    session = MCSession(peer: peer)
    session.delegate = self
}
```
如你所见，一个session的初始化只需要一个参数，就是之前的peer。并且在初始化过程中将session对象委托给了当前类。

接着是搜索器对象:
```swift
override init() {
    ...
 
    browser = MCNearbyServiceBrowser(peer: peer, serviceType: "appcoda-mpc")
    browser.delegate = self
}
```
这个对象的初始化接受两个参数：第一个是peer. 第二个是一个在初始化后无法改变的值，指明了搜索器能搜索到的服务类型(service type)。简单地讲，它用于在大量节点中的准确识别，这样MPC就知道需要搜索什么了，而广播器也必须设置成相同的服务类型(一会儿你会见到)。在设置这个值时需要遵守两条规则：(a)不能超过15个字符(b)只能包含小写的ASCII字符、数字与连字符。如果你不符合这个规则会跳出一个runtime的异常并且app将会崩溃。

对于广播器:
```swift
override init() {
    ...
 
    advertiser = MCNearbyServiceAdvertiser(peer: peer, discoveryInfo: nil, serviceType: "appcoda-mpc")
    advertiser.delegate = self
}
```
注意，在这里设置与之前相同的服务类型，还有一个叫discoveryInfo的参数，这个参数是一个字典，当你想给发现的其他节点发送额外信息时所需要设置的，注意，这个字典的键值都必须是字符串类型。方便起见，将这个参数设为nil。

初始化方法准备好了，这里是完整的代码:
```swift
override init() {
    super.init()
 
    peer = MCPeerID(displayName: UIDevice.currentDevice().name)
 
    session = MCSession(peer: peer)
    session.delegate = self
 
    browser = MCNearbyServiceBrowser(peer: peer, serviceType: "appcoda-mpc")
    browser.delegate = self
 
    advertiser = MCNearbyServiceAdvertiser(peer: peer, discoveryInfo: nil, serviceType: "appcoda-mpc")
    advertiser.delegate = self
}
```
在结束这部分之前，来创建一个实现委托模式的协议。注意我们会声明当前在app的开发过程中所需要的所有委托方法，之后将不会再处理这个协议，直到需要它们的时候。

在自定义类的上方添加如下代码段:
```swift
protocol MPCManagerDelegate {
    func foundPeer()
 
    func lostPeer()
 
    func invitationWasReceived(fromPeer: String)
 
    func connectedWithPeer(peerID: MCPeerID)
}
```
接着会讨论上面每一个函数的作用，在下一个部分中会详细介绍。

最后，在MPCManager类中声明一个delegate对象:
```swift
var delegate: MPCManagerDelegate?
```
目前为止成功完成了项目的第一个重要部分，可以歇歇脚了。我想说下，现在在Xcode中提示的错误是正常情况，在实现MPC的委托方法后就会消失了。

## 搜索节点

MCNearbyServiceBrowserDelegate协议中有三个方法允许我们处理发现与丢失的节点，还有在搜索过程中可能发生的错误。下面要实现这些方法来继续开发这个demo app，不过这太简单了以至于不用花多少工夫。

来从第一个方法开始吧，当发现附近的节点时(换句话说，当发现其他设备时)由MPC调用。先看看实现的代码:
```swift
func browser(browser: MCNearbyServiceBrowser!, foundPeer peerID: MCPeerID!, withDiscoveryInfo info: [NSObject : AnyObject]!) {
    foundPeers.append(peerID)
 
    delegate?.foundPeer()
}
```

首先需要着手的一个重要的行为就是在foundPeers数组中添加找到的节点(在之前已经声明了，记得吗?)，之后将这个数组作为ViewController类中列表视图的数据源，列出所有找到的节点。要这么做的话需要调用MPCManagerDelegate协议中的foundPeer委托方法，这个方法需要在ViewController类中实现(下一节中)，在那儿会重载列表数据，给用户显示最新发现的节点。

目前处理了发现一个节点时的情况，同样也需要考虑到相反的情况，需要关注若节点不再可用(可发现)时的状况。因此，来实现下一个代理方法:
```swift
func browser(browser: MCNearbyServiceBrowser!, lostPeer peerID: MCPeerID!) {
    for (index, aPeer) in enumerate(foundPeers){
        if aPeer == peerID {
            foundPeers.removeAtIndex(index)
            break
        }
    }
 
    delegate?.lostPeer()
}
```
没什么可说的，代码已经表达地很清楚了。首先找到了节点在foundPeers数组中的位置，然后移除它，这样在我们的列表里就不存在这个节点了，因此需要提醒ViewController刷新列表所显示的节点。因此需要调用lostPeer代理方法，在它的实现过程中会重载列表的数据。

最后，还有一个代理方法需要实现，这个是用来管理可能发生的错误信息，搜索不可用的情况。显然，不会在这里处理严重的错误，只是显示错误的信息，像如下代码所示来实现它:
```swift
func browser(browser: MCNearbyServiceBrowser!, didNotStartBrowsingForPeers error: NSError!) {
    println(error.localizedDescription)
}
```
在这一节结束前，来注意一下：使用上面的委托方法来提示ViewController类对节点变化的响应，应使用键值编码与观察者机制来追踪foundPeers数组的变化。不过由于有MPCManagerDelegate协议，就不需要去写额外的代码了。如果不需要这个协议中的其他方法，使用KVC与KVO好一些而不是委托模式。在这里的情况，我们坚持实现有着更快速并简洁的执行过程的委托方法的做法(PS:我承认这里有问题(´･ω･｀))。

做完上面的工作后，app就能够搜索节点了。 现在，来添加一些必要的代码，显示出找到的节点吧。

## 显示搜索到的节点
至此这个demo应用可以发现周围的节点，并且添加(或移除)进foundPeers数组，接下来让我们在ViewController中的列表中显示它们吧。在此之前需要关注一下AppDelegate类，在那里声明MPCManager对象，这样在其他类中就可以通过app中的delegate实例来访问它了。

在工程导航栏中点击AppDelegate.swift文件打开它。在类的顶部添加如下代码:
```swift
var mpcManager: MPCManager!
```
进入application(application:didFinishLaunchingWithOptions:)方法，进行MPCManager的初始化:
```swift
func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
    // Override point for customization after application launch.
 
    mpcManager = MPCManager()
 
    return true
}
```
现在，打开ViewController.swift文件，里面已经有了一个最小实现。显然还不够，需要添加更多代码，来填充与列表有关的方法，这样一切就能正常工作了。在类的顶部，从同时声明与初始化应用的delegate对象开始:
```swift
let appDelegate = UIApplication.sharedApplication().delegate as AppDelegate
```
在声明的同时实例化或初始化一个对象时Swift的一个特点，因为这样会使代码变得很简洁，提高开发效率。下一步就是MPCManager类的delegate设置成ViewController，在ViewDidLoad方法中添加如下行:
```swift
override func viewDidLoad() {
    ...
 
    appDelegate.mpcManager.delegate = self
 
}
```
在这行Xcode会显示一条错误，原因很简单:还没引入MPCManagerDelegate协议，在类的第一行如下添加一下即可:
```swift
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource, MPCManagerDelegate
```
接着来实现之前章节中使用过的两个委托方法，foundPeer和lostPeer。不管哪种情况下都需要要刷新列表即可，因此它们的实现代码很简单:
```swift
func foundPeer() {
    tblPeers.reloadData()
}
 
 
func lostPeer() {
    tblPeers.reloadData()
}
```
上面这部分是关键的一步，不过如果我们不“告诉”列表数据源是什么的话依旧是空白的，不做适当的修改会直接显示周围节点的名字。因此，需要从修改列表的行数与区数开始。显然，行数就是所找到的所有节点数:
```swift
func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return appDelegate.mpcManager.foundPeers.count
}
```
是时候显示每个节点的名字了:
```swift
func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
    var cell = tableView.dequeueReusableCellWithIdentifier("idCellPeer") as UITableViewCell
 
    cell.textLabel?.text = appDelegate.mpcManager.foundPeers[indexPath.row].displayName
 
    return cell
}
```
注意在上面的代码中访问了每个节点在foundPeers数组中的displayName属性。

最后来设置每个行的高度:
```swift
func tableView(tableView: UITableView, heightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat {
    return 60.0
}
```
上面的都搞定后，列表就能正常工作了。

虽然在这里已经搞定了一项重要的工作了，不过还没有考虑另一个重要的细节: 搜索功能默认是关闭的，如果不进行指示，设备是不会去搜索的。可以随时启动搜索，取决于由你的应用需求。这里，我们在app启动后立刻开启，进入viewDidLoad方法添加如下行:
```swift
override func viewDidLoad() {
    ...
 
    appDelegate.mpcManager.browser.startBrowsingForPeers()
}
```
要执行停止搜索的话，需要调用搜索器的stopBrowsingForPeers()方法。

有关搜索器的都搞定了, 接下来实现设备的广播功能。

## 处理广播

除了搜索，multipeer connectivity框架还允许设备对附近的节点广播自身，没有广播的话搜索就没有意义。实际上，它们是相辅相成，同等重要的。广播，意味着一个设备是否对其他设备可见。如果app中启用广播功能，则该设备就对附近的其他节点可见，不启用的话其他节点就不会发现。在这一节中来看看其中的细节，启动与禁用demo app的广播功能。

若你已经熟悉过开始工程了的话，肯定会记得在ViewController场景中有个包含一个按钮的工具栏。接着将使用这个按钮，通过实现startStopAdvertising(sender:)方法来进行开关广播功能的操作。点击按钮时会显示一个action表单，在这个action表单中有两个按钮：一个进行两种广播状态的切换，另一个控制是否打开action表单对话框。为了更有趣，使第一个按钮上的文字随着当前状态变化。

在用代码实现上述之前，先声明一个新的Bool属性，用它来判断设备是否在广播状态。确保你是在操作ViewController.swift文件，在类的顶部添加如下行:

```swift
var isAdvertising: Bool!
```
在viewDidLoad中会给这个变量赋值，设为true，意味着设备当前在广播自己，不过同时也需要打开这个功能，因此在viewDidLoad方法需要同时实现:
```swift
override func viewDidLoad() {
    ...
 
    appDelegate.mpcManager.advertiser.startAdvertisingPeer()
 
    isAdvertising = true    
}
```
现在可以到action方法中实现它了。先亮出代码，之后再讲解:
```swift
@IBAction func startStopAdvertising(sender: AnyObject) {
        let actionSheet = UIAlertController(title: "", message: "Change Visibility", preferredStyle: UIAlertControllerStyle.ActionSheet)
 
        var actionTitle: String
        if isAdvertising == true {
            actionTitle = "Make me invisible to others"
        }
        else{
            actionTitle = "Make me visible to others"
        }
 
        let visibilityAction: UIAlertAction = UIAlertAction(title: actionTitle, style: UIAlertActionStyle.Default) { (alertAction) -> Void in
            if self.isAdvertising == true {
                self.appDelegate.mpcManager.advertiser.stopAdvertisingPeer()
            }
            else{
                self.appDelegate.mpcManager.advertiser.startAdvertisingPeer()
            }
 
            self.isAdvertising = !self.isAdvertising
        }
 
        let cancelAction = UIAlertAction(title: "Cancel", style: UIAlertActionStyle.Cancel) { (alertAction) -> Void in
 
        }
 
        actionSheet.addAction(visibilityAction)
        actionSheet.addAction(cancelAction)
 
        self.presentViewController(actionSheet, animated: true, completion: nil)
    }
```
简要说下上面的实现代码都做了啥:

* 首先，使用一条信息与合适的风格来初始化了一个action表单控制器。
* 接着，根据isAdvertising的当前值，通过分配对应的值到actionTitle这个本地变量中来设置action表单中第一个按钮的标题。
* 有了正确的标题后，创建一个新的提醒行为，当用户点击第一个按钮时触发。
* 最重要的部分: 根据isAdvertising的值，停止或开始设备的广播。当然不要忘了将isAdvertising值设为相反的值。
* 为cancel按钮创建一个(空的)action。
* 两个action都添加倒了action表单控制器中。
* 最后，开启显示动画设置跳转视图。

在之后的app测试中会看到上面的action方法时如何工作的。做完上面的工作后，就可以改变设备的可被发现状态了。

## 邀请节点

multipeer connectivity的目的就是使两个设备间建立连接然后交换数据，我们现在只做了一半工作，仅实现了节点的发现与广播功能。下一步就是邀请节点加入到会话中，这样，两个设备间就可以进行通信了。

这里最重要的是手动来实现MPC，理解这个过程。何时来邀请找到的节点，并连接到会话中，都是由你决定的。并没有一个规定的、默认的时间，需要根据app的运行环境来选取合适的时机。比如说，当我们点击ViewController中的列表项时就会邀请对应的附近节点，也有其他情况，可能需要在搜索器发现节点后就发送邀请。何时来邀请取决于app中的实际需求。另外，还有一件同等重要的事你需要知道，发送邀请与监理连接可以在不请求用户许可的情况下进行，可以在后台执行，不过我建议你在连接其他设备或收发数据等情况时要提醒一下用户。

回到demo中，下面是这一节的目标:

1. 点击列表项时向对应名字的节点发送一个邀请。
2. 接受邀请。
3. 询问用户是否接受聊天邀请。
4. 接受或拒绝邀请

依旧在ViewController.swift文件中，来让列表来对点击事件产生响应。接着实现列表的委托方法(tableView:didSelectRowAtIndexPath:)，在这里需要执行一项重要的任务：给选择的节点发送邀请。如你所见，仅仅需要一行代码。来看看实现方法，后面会讨论它:
```swift
func tableView(tableView: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath) {
    let selectedPeer = appDelegate.mpcManager.foundPeers[indexPath.row] as MCPeerID
 
    appDelegate.mpcManager.browser.invitePeer(selectedPeer, toSession: appDelegate.mpcManager.session, withContext: nil, timeout: 20)
}
```
讲真，如果我们将选择的节点分配给一个本地变量的话就可以省去第一行代码。不管如何，第一行代码表达的更清楚，留着它吧。

使用搜索器对象的invitePeer(peerID:toSession:withContext:timeout:)方法，使用multipeer connectivity框架给选择的节点发送一个邀请。接收如下参数:

1. peerID: 想要发送邀请的节点。
2. toSession: 在MPCManager类中初始化的session对象。
3. withContext: 若要给邀请的节点发送额外数据时可以用这个参数，需要一个NSData对象。
4. timeout: 这里设置邀请者等待节点答复的最长时间。默认是30秒，这里设置为20。在一个真正的app中需要“告诉”你具体的等待时间。

现在跳转到类的顶部，导入MPC框架:
```swift
import MultipeerConnectivity
```
上述是第一步，接着必须实现MCNearbyServiceAdvertiserDelegate协议中的一对delegate方法，这样app就能处理一个接收到的邀请了。打开 MPCManager.swift文件。

接下来看到的方法包含一个邀请处理者(invitation handler)，用来回复发送邀请的节点。这个处理者接受两个参数，第一个是指示邀请是否被接受的bool值，第二个是session对象，对应接受邀请的情况。在demo中不会立即给邀请者一个答复，需要先询问用户是否想要接受邀请。因此必须先将邀请处理暂时储存到一个属性中，在用户响应后再回复这个邀请。之前声明过如下的方法:
```swift
var invitationHandler: ((Bool, MCSession!)->Void)!
```
需要储存邀请处理者，像之前说的那样，第一个delegate方法实现如下代码所示:
```swift
func advertiser(advertiser: MCNearbyServiceAdvertiser!, didReceiveInvitationFromPeer peerID: MCPeerID!, withContext context: NSData!, invitationHandler: ((Bool, MCSession!) -> Void)!) {
    self.invitationHandler = invitationHandler
 
    delegate?.invitationWasReceived(peerID.displayName)
}
```
使invitation handler与invitationHandler属性保持相同。注意这两个处理者名字相同，因此self的使用时必须的。

此外，调用了MPCManagerDelegate协议中的另一个方法。 提醒ViewController类已经接收到了一个邀请，给用户显示出一个alert控制器来询问是否聊天。之后就会看到它的实现，现在只需要关注显示节点名称就行了。

MCNearbyServiceAdvertiserDelegate协议中的第二个委托方法被用来处理无法打开广播器的情况，在控制台打印出错误信息:
```swift
func advertiser(advertiser: MCNearbyServiceAdvertiser!, didNotStartAdvertisingPeer error: NSError!) {
    println(error.localizedDescription)
}
```
现在回到ViewController.swift文件，来实现invitationWasReceived(fromPeer:)委托方法。它会显示一条信息，提醒用户聊天的对象是什么(显示节点名称)，并且提供2种选择：接受与拒绝。不论用户选择的是什么，都要使用MPCManager类的invitationHandler属性来回复邀请者。来看看如何实现的:
```swift
func invitationWasReceived(fromPeer: String) {
    let alert = UIAlertController(title: "", message: "\(fromPeer) wants to chat with you.", preferredStyle: UIAlertControllerStyle.Alert)
 
    let acceptAction: UIAlertAction = UIAlertAction(title: "Accept", style: UIAlertActionStyle.Default) { (alertAction) -> Void in
        self.appDelegate.mpcManager.invitationHandler(true, self.appDelegate.mpcManager.session)
    }
 
    let declineAction = UIAlertAction(title: "Cancel", style: UIAlertActionStyle.Cancel) { (alertAction) -> Void in
        self.appDelegate.mpcManager.invitationHandler(false, nil)
    }
 
    alert.addAction(acceptAction)
    alert.addAction(declineAction)
 
    NSOperationQueue.mainQueue().addOperationWithBlock { () -> Void in
        self.presentViewController(alert, animated: true, completion: nil)
    }
}
```
上面是一个典型的alert控制器的实现，没什么难的地方。接受邀请时，调用mpcManager对象的invitationHandler属性，将邀请的回复设为true，并提供session对象。如果用户不想聊天，则将邀请处理者的第一个参数设为false，第二个参数为空，这种情况下不需要发送session。

又完成了一个关键步骤，接着来看看一个session的状态与当连接建立时会发生什么。

## 连接到会话

会话使用MPCManager类中的session对象表示，是在用multipeer connectivity时的最终目标。2个节点都连接到一个会话中时，它们就可以互相交换数据与资源、执行流传输了。一个会话有三种状态：；已连接(Connecting)、正在连接(Connecting)与断开(Not Connected)。

multipeer connectivity框架使我们对每一个状态都有控制权，通过MCSessionDelegate协议的委托方法来实现。通常这个方法的实现并不难，只需要对应每一种状态设置其相应的行为即可。在下面的代码段中，对于已连接状态，调用了一个其他的MPCManagerDelegate协议委托方法，而对于其他两种状态，仅仅在控制台显示了一条信息，这样就可以在测试过程中准确的得知当前会话的状态。粘贴下面代码的时候注意当前操作的文件是否是MPCManager.swift。
```swift
func session(session: MCSession!, peer peerID: MCPeerID!, didChangeState state: MCSessionState) {
    switch state{
    case MCSessionState.Connected:
        println("Connected to session: \(session)")        
        delegate?.connectedWithPeer(peerID)
 
    case MCSessionState.Connecting:
        println("Connecting to session: \(session)")        
 
    default:
        println("Did not connect to session: \(session)")
    }
}
```
使用connectedWithPeer(peerID:)委托方法通知ViewController类：设备已经与附近的一个节点(由上述的peerID参数决定)连接到了一个会话中了。

再次回到ViewController.swift文件, 来实现connectedWithPeer(peerID:)方法。在这个demo中想要在节点连接到会话时就开始聊天，因此仅需要跳转到ChatViewController场景即可。很简单:
```swift
func connectedWithPeer(peerID: MCPeerID) {
    NSOperationQueue.mainQueue().addOperationWithBlock { () -> Void in
        self.performSegueWithIdentifier("idSegueChat", sender: self)
    }
}
```
注意这会使两个设备(邀请者与被邀者)都执行segue，两者的会话状态都会变为已连接。

之后会去操作ChatViewController类，在操作前让我来强调一件事：我们不会去处理用户终止聊天邀请的情况，我觉得这不是很重要，交给你自己来实现吧。

## 一个发送数据的简便方法

在这一节我们会创建MPCManager类的一个自定义方法，用来将数据发送给其他节点。实际上是调用MCSession类中一个负责发送数据的特殊方法，不过首先需要准备并配置好这个方法所需要的所有参数。创建这个自定义方法的原因是因为避免在多个地方进行配置，一次搞定。

先给你实现的方法 (确保打开了MPCManager.swift文件):
```swift
func sendData(dictionaryWithData dictionary: Dictionary<String, String>, toPeer targetPeer: MCPeerID) -> Bool {
    let dataToSend = NSKeyedArchiver.archivedDataWithRootObject(dictionary)
    let peersArray = NSArray(object: targetPeer)
    var error: NSError?
 
    if !session.sendData(dataToSend, toPeers: peersArray, withMode: MCSessionSendDataMode.Reliable, error: &error) {
        println(error?.localizedDescription)
        return false
    }
 
    return true
}
```
上面的sendData(data:toPeers:withMode:error:)就是所调用的MPCSession方法，它接受如下参数:

* data: 一个NSData对象，实际发送的数据。
* toPeers: 一个接收数据的节点数组 (NSArray)。
* withMode: 数据发送模式，有两种模式: 可靠与不可靠。如果没收到的数据不会引起任何问题，这种数据不重要的情况下可以使用第二种模式。
* error: 包含可能发生error的一个NSError对象。

现在来看看上面的实现代码发生了什么，如你所见，这个方法有两个参数：(a)一个字典对象 (b)目标节点。首先，使用NSKeyedArchiver类对字典进行归档，将其转化成一个NSData对象，然后定义一个仅包含目标节点的单元素peersArray数组。声明error变量后，将上面提供的变量作为输入参数，调用session对象的数据发送方法。

若在发送数据时有错误，会将其显示出来然后返回false，否则返回true，意味着一切正常。

上面这个方法会在下一节用到。

## 在节点间发送数据

在聊天会话的过程中节点发送与接收的所有消息，都会显示在ChatViewController的列表中。最靠后显示的消息总是时刻最近的那条，每当一条消息被发送与接收时都会使列表重载。

你可能猜到了，会使用一个数组来储存所有消息，显然这个数组会被作为列表的数据源。重要的是，这个数组中的每一个对象都是一个包含字符串类型键值的字典。为何是字典?因为需要为每个发送或接收的消息分配一对数据：消息的发送方与消息本身。当我们的设备是消息发送方时，设备中的消息发送方就会被设为"self"，我们的节点名就会发送给其他设备。

回到编码的工作中，现在有一些事情要做了。第一步就是要声明并初始化消息数组(列表的数据源), 之后会声明并初始化一个应用的delegate对象，因此会访问APPDelegate类的mpcManager属性。打开ChatViewController.swift文件，在类顶部添加如下两行:
```swift
var messagesArray: [Dictionary<String, String>] = []
 
let appDelegate = UIApplication.sharedApplication().delegate as AppDelegate
```
messagesArray初始化为一个空数组，我们不打算将ChatViewController设为mpcManager的委托对象，将使用另一种途径来从MPCManager类中获取消息。

在聊天时，每当一个新消息完成编辑并且发送按钮被按下时会触发一些行为，下面这些是我们需要做的:
1. 隐藏键盘。
2. 根据消息创建一个字典，调用上一节的自定义方法将消息发送给其他节点。
3. 创建另一个字典，将发送方与消息作为其内容，然后储存进messagesArray数组中。
4. 刷新列表。
5. 消息发送后清空输入栏。

上述这些发生在UITextFieldDelegate协议中的委托方法textFieldShouldReturn(textField:)中。如果看看viewDidLoad方法，会发现ChatViewController类已经被设为了textfield的委托对象。

代码实现如下:
```swift
func textFieldShouldReturn(textField: UITextField) -> Bool {
    textField.resignFirstResponder()
 
    let messageDictionary: [String: String] = ["message": textField.text]
 
    if appDelegate.mpcManager.sendData(dictionaryWithData: messageDictionary, toPeer: appDelegate.mpcManager.session.connectedPeers[0] as MCPeerID){
 
        var dictionary: [String: String] = ["sender": "self", "message": textField.text]
        messagesArray.append(dictionary)
 
        self.updateTableview()
    }
    else{
        println("Could not send data")
    }
 
    textField.text = ""
 
    return true
}
```
需要注意的点: 调用了自定义方法sendData(dictionaryWithData:toPeer:)，输入之前创建的messageDictionary。有趣的是我们使用appDelegate.mpcManager.session.connectedPeers[0]对象来指明目标节点，需要说明的是MCSession类包含一个叫connectedPeers的数组属性，每个连接到我们设备的节点都会被记录下来。在我们的代码中，由于已经知道只有1个节点会加入到会话中， 因此直接访问数组的第一个元素即可。

如果数据被成功发送了，则为发送方与消息准备一个新的字典。如果是我们的消息，则sender被设为“self”，接着使用messagesArray的append方法将字典添加到数组中。最后调用updateTableview方法刷新列表，这是个自定义方法，一会儿会实现它。

如果有错误发生，会在控制台显示一条信息。不管发生什么，在上述方法的最后都会清空输入栏。

要编写的updateTableview方法有两个目的:首先是刷新列表，显示新的消息。其次，自动滚动到列表底部，这样最新的消息就能时刻显示了 。代码如下:
```swift
func updateTableview(){
    self.tblChat.reloadData()
 
    if self.tblChat.contentSize.height > self.tblChat.frame.size.height {
        tblChat.scrollToRowAtIndexPath(NSIndexPath(forRow: messagesArray.count - 1, inSection: 0), atScrollPosition: UITableViewScrollPosition.Bottom, animated: true)
    }
}
```
当列表的content size大于列表的框体高度时，则必须要滚动，使用上面的方法实现。

现在，在类的顶部导入multipeer connectivity框架， 修复在textFieldShouldReturn(textField:)中的错误:
```swift
import MultipeerConnectivity
```
在这一节结束前还需要做一件事，需要完善列表相关的方法。首先需要将messagesArray数组中已存在的元素个数分配给行数:
```swift
func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return messagesArray.count
}
```
在tableView(tableView:cellForRowAtIndexPath:)方法中需要检查发送方的消息。如果发送方的值是“self”，则将子标签设为紫色，会显示消息的前缀“I said:”。否则设为橘黄色，显示“X said:”，X是其他节点的名称。代码如下:
```swift
func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
    var cell = tableView.dequeueReusableCellWithIdentifier("idCell") as UITableViewCell
 
    let currentMessage = messagesArray[indexPath.row] as Dictionary<String, String>
 
    if let sender = currentMessage["sender"] {
        var senderLabelText: String
        var senderColor: UIColor
 
        if sender == "self"{
            senderLabelText = "I said:"
            senderColor = UIColor.purpleColor()
        }
        else{
            senderLabelText = sender + " said:"
            senderColor = UIColor.orangeColor()
        }
 
        cell.detailTextLabel?.text = senderLabelText
        cell.detailTextLabel?.textColor = senderColor
    }
 
    if let message = currentMessage["message"] {
        cell.textLabel?.text = message
    }
 
    return cellc
}
```
上面的方法中没什么难的地方，就不讨论它了。

需要关注一个列表行的高度，很明显我们不能确定高度是多少，因为消息的长度是变化的，每行的高度都需要动态变化。因此使用一个iOS 8的特性，叫做self sizing cells(ps:早知道这个就好了...)，可以看看Simon的这个教程。将列表栏的文本标签的行数设为0，然后在viewDidLoad方法中设置以下两个属性:
```swift
tblChat.estimatedRowHeight = 60.0
tblChat.rowHeight = UITableViewAutomaticDimension
```
上述代码在viewDidLoad中已经有了，交给iOS处理吧。

把上面的搞定后就可以进行接收数据的工作了。

## 接收数据
现在app可以发送消息了，现在需要处理接收数据的环节了。接着要操作MPCManager与ChatViewController类。首先，来实现一个MPCSessionDelegate协议中的一个新方法。

打开MPCManager.swift文件，添加如下方法:
```swift
func session(session: MCSession!, didReceiveData data: NSData!, fromPeer peerID: MCPeerID!) {
    let dictionary: [String: AnyObject] = ["data": data, "fromPeer": peerID]
    NSNotificationCenter.defaultCenter().postNotificationName("receivedMPCDataNotification", object: dictionary)
}
```
只包括仅仅两行却至关重要的代码。首先将接收到的数据与发送方节点添加到一个字典中，接着提交一个名为receivedMPCDataNotificaton的通知(NSNotification)，在ChatViewController中监听这个通知，作出适当的处理，在列表中显示发送方的名称与消息。在上一节中我说过不要将ChatViewController设为MPCManager的委托对象，会通过另一种途径从这个类中获取消息，这里的不同的途径就是上面我们提交的通知。

现在让我们回到ChatViewController.swift文件中，在viewDidLoad方法中来监听上面的通知。这很简单，只需要下面几行代码即可:
```swift
override func viewDidLoad() {
    ...    
 
    NSNotificationCenter.defaultCenter().addObserver(self, selector: "handleMPCReceivedDataWithNotification:", name: "receivedMPCDataNotification", object: nil)
}
```
这么做了后，每当收到新数据时都会提交一个通知，ChatViewController会做出响应。

还剩最后一步：实现handleMPCReceivedDataWithNotification(notification:)，当检测到通知时会调用它。

在实现它之前，先来大概说说在其中发生了什么:
* 第一步，需要得到通知中带有的字典，“提取”其中的数据与节点。
* 将数据对象转换为字典，即可访问其中的数据。
* 设置一个规则，指明聊天终止的一个特殊短语，这个短语即为“end_chat”消息。
* 若消息不是上述的特殊值，则创建一个包含发送方名称与消息的字典，将其添加进messagesArray数组中，并刷新列表。
* 若消息表示聊天的终点，即提醒用户另一个节点终止了聊天，则会返回当前视图控制器。在下一节中会编写这一步的代码。

来看看上述代码的实现，代码中的额外注释会帮助你理解:
```swift
func handleMPCReceivedDataWithNotification(notification: NSNotification) {
    // Get the dictionary containing the data and the source peer from the notification.
    let receivedDataDictionary = notification.object as Dictionary<String, AnyObject>
 
    // "Extract" the data and the source peer from the received dictionary.
    let data = receivedDataDictionary["data"] as? NSData
    let fromPeer = receivedDataDictionary["fromPeer"] as MCPeerID
 
    // Convert the data (NSData) into a Dictionary object.
    let dataDictionary = NSKeyedUnarchiver.unarchiveObjectWithData(data!) as Dictionary<String, String>
 
    // Check if there's an entry with the "message" key.
    if let message = dataDictionary["message"] {
        // Make sure that the message is other than "_end_chat_".
        if message != "_end_chat_"{
            // Create a new dictionary and set the sender and the received message to it.
            var messageDictionary: [String: String] = ["sender": fromPeer.displayName, "message": message]
 
            // Add this dictionary to the messagesArray array.
            messagesArray.append(messageDictionary)
 
            // Reload the tableview data and scroll to the bottom using the main thread.
            NSOperationQueue.mainQueue().addOperationWithBlock({ () -> Void in
                self.updateTableview()
            })
        }
        else{
 
        }
    }
}
```
准备好了。现在若有消息来到时，app就会在列表中显示它了。

## 终止聊天
离应用中功能全部实现的工作所剩无几了，其中一个就是终止聊天，发生在其中一个节点想要终止或会话不是已连接状态。

在ChatView Controller场景中最顶部的工具栏上有一个按钮，点击后会调用endChat(sender:)方法。使用这个方法给其他节点发送终止消息告诉它们聊天结束，然后返回到上一个视图控制器。当然，这个消息即为上一节所说的 “_end_chat”短语。

来看看具体实现:
```swift
@IBAction func endChat(sender: AnyObject) {
    let messageDictionary: [String: String] = ["message": "_end_chat_"]
    if appDelegate.mpcManager.sendData(dictionaryWithData: messageDictionary, toPeer: appDelegate.mpcManager.session.connectedPeers[0] as MCPeerID){
        self.dismissViewControllerAnimated(true, completion: { () -> Void in
            self.appDelegate.mpcManager.session.disconnect()
        })
    }
}
```
如你所见，在这里dismiss了视图控制器然后使用MCSession类的disconnect()方法断开了与节点间的会话。在跳转动画完成后才断开会话连接，给了会话足够的存活时间。

在最后一节中实现了自定义方法handleMPCReceivedDataWithNotification(notification:)中的部分代码，为何我会说“部分”，因为还没有添加任何处理当发送终止聊天消息时的代码。是时候添加了，在做之前需要注意，要给用户显示一个alert控制器提醒他其它节点结束了聊天，在那之后，我们会从会话中断开并dismiss视图控制器。这里是代码中缺失的部分:
```swift
func handleMPCReceivedDataWithNotification(notification: NSNotification) {
    ...
 
    // Check if there's an entry with the "message" key.
    if let message = dataDictionary["message"] {
        ...
        else{
            // In this case an "_end_chat_" message was received.
            // Show an alert view to the user.
            let alert = UIAlertController(title: "", message: "\(fromPeer.displayName) ended this chat.", preferredStyle: UIAlertControllerStyle.Alert)
 
            let doneAction: UIAlertAction = UIAlertAction(title: "Okay", style: UIAlertActionStyle.Default) { (alertAction) -> Void in
                self.appDelegate.mpcManager.session.disconnect()
                self.dismissViewControllerAnimated(true, completion: nil)
            }
 
            alert.addAction(doneAction)
 
            NSOperationQueue.mainQueue().addOperationWithBlock({ () -> Void in
                self.presentViewController(alert, animated: true, completion: nil)
            })            
        }
    }
}
```
当其它节点关闭app时也会导致聊天被终止，或者由于某些原因两设备间的连接丢失，这时需要考虑的情况。要做的就是在MPCManager.swift类的browser(browser:lostPeer:)代理方法中添加一个新的通知，然后还需要监听并处理它，就像之前的那个通知一样。不过这次我就不给你看具体的实现了，作为练习麻烦你自己动手完成吧。

快成功了，再进行一些小小的增补就可以测试了。

## 最后修整
离测试app只剩一步了，还有一些事没有做。需要在MPCManager类中定义一些MCSessionDelegate协议中的委托方法，虽然用不上。这样做了的话，一些Xcode中的错误就会消失了。

不要犹豫，打开MCPManager.swift文件copy下面的方法即可:
```swift
func session(session: MCSession!, didStartReceivingResourceWithName resourceName: String!, fromPeer peerID: MCPeerID!, withProgress progress: NSProgress!) { }
 
func session(session: MCSession!, didFinishReceivingResourceWithName resourceName: String!, fromPeer peerID: MCPeerID!, atURL localURL: NSURL!, withError error: NSError!) { }
 
func session(session: MCSession!, didReceiveStream stream: NSInputStream!, withName streamName: String!, fromPeer peerID: MCPeerID!) { }

```

## 测试app
可以在两台设备(至少)上运行demo，也可以在一台设备与一个虚拟机上运行。首先通过开关app中的广播功能来操作搜索器与广播器，然后选择一个节点来开始聊天，发送、接收消息，最后终止对话。如果你完成了测试，可以改改代码，添加一些你想要的功能。

我要展示一些app的截图，注意我是在iPhone模拟器上运行的。

发现一个附近的节点:

![](http://www.appcoda.com/wp-content/uploads/2015/01/t27_1_list_peers.png)

关闭广播器:

![](http://www.appcoda.com/wp-content/uploads/2015/01/t27_6_turn_off_advertiser.png)

询问用户是否聊天:

![](http://www.appcoda.com/wp-content/uploads/2015/01/t27_2_ask_chat_alert.png)

进行聊天:

![](http://www.appcoda.com/wp-content/uploads/2015/01/t27_3_chatting.png)

结束聊天:

![](http://www.appcoda.com/wp-content/uploads/2015/01/t27_7_end_of_chat.png)

可以在[这里](https://www.dropbox.com/s/sdi6h5xosssbkei/MPCRevisited.zip?dl=0)下载最后的工程。
