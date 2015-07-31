
#本文作者#

![](http://7xi6qz.com1.z0.glb.clouddn.com/djlblogpicslrme.jpg)

## [Jiale Dai](https://github.com/daijiale) ##

成都  电子科技大学  本科生

从14年开始一直专研于AndroidWear开发。


>**摘要：** 本文是在学习Google官方视频、开发者文档和实践项目之后整理出来的心得笔记，是以一个个人开发者的角度给大家带来一些侧面对Android Wear开发的看法，不同于一些传统的Android Wear技术开发教程，但是博主希望能通过自己对这些知识的整理和资源的收集，给读者带了一份详尽的、多角度的Android Wear指南。**无论你是程序员，设计师，产品经理，还是手表极客 ，Android Wear用户 or 小白，都能在这篇博文中找到你想要的Android Wear元素。**
> 博文会同时托管到Github上，欢迎更多承载着开源精神的有心人加入，分享你对Android Wear的见解。

#写在开头#
> **自己对AndroidWear的看法：**
> 
> Android Wear的目标就是：不接触手机的前提下，在你需要的时候，它把对你有用的信息呈现给你，扫一眼就够了。ta是
> 一种新的交互模型，有很多有利便捷新潮的交互体验是手机上无法实现的。将你自己置身于一个外部场景，在移动和忙碌中使用这项服务是什么样的体验，你就会发现ta的价值。

**下面我们来欣赏一段Android Wear的应用场景视频**（博主花了大精力才从鹅厂官网漏洞里抓取到的外链地址，**低调、低调**）：

<embed wmode="window" flashvars="vid=o0014kprxll&amp;tpid=28&amp;showend=1&amp;showcfg=1&amp;searchbar=1&amp;shownext=1&amp;list=2&amp;autoplay=1&amp;ptag=m_v_qq_com&amp;outhost=http%3A%2F%2Fv.qq.com%2Fpage%2Fo%2Fl%2Fl%2Fo0014kprxll.html&amp;refer=http%3A%2F%2Fm.v.qq.com%2Fpage%2Fo%2Fl%2Fl%2Fo0014kprxll.html%3Ffrom%3Dtimeline%26isappinstalled%3D0&amp;openbc=0&amp;title=%E8%B0%B7%E6%AD%8CAndroid%20Wear%E6%B7%B1%E5%BA%A6%E8%A7%A3%E6%9E%90" src="http://imgcache.qq.com/tencentvideo_v1/player/TencentPlayer.swf?max_age=86400&amp;v=20140714" quality="high" name="tenvideo_flash_player_1431178166025" id="tenvideo_flash_player_1431178166025" bgcolor="#000000" width="650px" height="472px" align="middle" allowscriptaccess="always" allowfullscreen="true" type="application/x-shockwave-flash" pluginspage="http://get.adobe.com/cn/flashplayer/"> 



##  核心元素： ##

 - Google Now：用户可以和AndroidWear“说话”（语音交互）。
 - Notifications:一个卡片，一个提醒，实现你最想要的服务。具体分为stacks、 pages、 replies、三种性质。
 - WatchFace：表之所以称之为“表”。
 - Data Message：和手机的数据通信机制是重要的桥梁。

## 构建一个Wear Apps的基础（wear app能做到什么？）： ##

**基础API元素:**

 - Custom UI
 - Send Data
 - Control Sensors
 - Voice Actions


下面我会举**四个例子**来说明基于这几个基础元素（Android Wear API）可以实现什么样级别的**Wear App**：
 

### Gmail ###

![](http://7xi6qz.com1.z0.glb.clouddn.com/androidweargmail.PNG)

 - Gmail Base On
    - Notification Bundles
    - RemoteInput for Voice Response
 
大家应该不会陌生Gmail，下面来看看其在wear端的app特性：

1、首先ta有两种邮件提醒类型：

**单页通知卡片（如下图）**

![](http://7xi6qz.com1.z0.glb.clouddn.com/androidwearsinglecard.PNG)

**多页通知卡片（如下图）**

![](http://7xi6qz.com1.z0.glb.clouddn.com/androidwearmuticard.PNG)

2、你可以通过滑动卡片，进一步了解更多信息，而且伴随有 **“语音快速回复” “归档” “在手机端打开回复”**三种操作，这里我们重点谈一下**语音快速回复**这个**具有wear特性**的操作流程：

 
`Android Notification API` 会让你通过远程输入给reply行为做一个注释,而远程输入则会告诉AndroidWear，在执行这个行为之前，你要把文本输入的方式改为语音，因此当Gmail建立一个notification连接时，wear端会给reply行为附加一个远程输入，AndroidWear会看到这条远程输入，然后不会立即发送一个行为，会首先启动一个wear UI界面来收集语音回复信息，然后把转换好的文本变成意图，再发送意图到你的手机上，手机得到意图后，就可以在不触动手机UI的情况下发送/回复邮件了。

下面是关键代码实现过程：

**Add RemoteInput to Reply Action**

```
Action replyAction = new NotificationCompat.Action.Builder(
	R.drawble.ic_reply,getString(R.string.reply), 
	replyPendingIntent)
    addRemoteInput(
	  new RemoteInput.Builder(EXTRA_REPLY_TEXT)
		.setLabel(getString(R.string.replyLabel))
		.build()).build();
```

**Modify Activity to Use Reply Text**

```
Bundle results =
	RemoteInput.getResultsFromIntent(intent);
	if(results!=null){
	 String message = 
		results getString(EXTRA_REPLY_TEXT)
	}
```


3、最后我再来详细介绍下“多卡片重叠式信息提醒”的实现原理，视觉设计上采用的是复线收件箱的风格，ta不再把多条信息压缩到单一的卡片中，我们想做的是每封邮件都有自己的卡片。而这些卡片又放入一个可扩大的堆栈中，这一堆提醒卡片，也叫提醒卡片堆栈，也是notification API的新特性，不会把所有邮件提醒都只能通过一个提醒显示出来，而是按类划分，表明有所关联，ta们在可穿戴设备上组合成一个卡片丛，而用户也可以通过卡片丛去逐个浏览，以提取某一封邮件，并对其回复或者进行其他操作。而卡片丛 即：notification group也有一个分类键，通过设置这个键来控制丛内卡片顺序，并且可以从中标记一个卡片作为组群的整体摘要描述，具体实现代码如下：


```
Notification card1 = 
	new NotificationCompat.Builder(context)
		.setGroup(GROUP_KEY)
		.setSortKey("0")
		.build();

Notification card2 = 
	new NotificationCompat.Builder(context)
		.setGroup(GROUP_KEY)
		.setSortKey("1")
		.build();

Notification card3 = 
	new NotificationCompat.Builder(context)
		.setGroup(GROUP_KEY)
		.setSortKey("2")
		.build();

Notification summary = 
	new NotificationCompat.Builder(context)
		.setGroup(GROUP_KEY)
		.setGroupSummar;
		.build();
```

### Hangouts ###

![](http://7xi6qz.com1.z0.glb.clouddn.com/androidwearhangouts.PNG)


 - Hangouts Base On
    - Custom Wearable Actions
    - Notification Pages
    
1、环聊信息也会自动桥接到可穿戴设备上，在Gmail上面，我们想要的是语音回复，但是环聊的提醒则稍微有点不同，ta并没有回复行为，只是一个内容意图，只需要打开App就可以键入回复了，所以环聊可以很好的不依赖手机而直接在可穿戴设备上进行体验，而且，android wear的 `Notification API`会让你在手机和可穿戴设备上细化不同的操作设置，即手机行为只在手机显示。wear行为只在wear端显示，这就使得我们可以添加一个仅限可穿戴设备使用的回复行为，这一行为涵盖了一个远程输入，却无需变动手机行为。

2、环聊也增加了新的提醒特征：近期会话历史记录。因为在语音回复之前，多出现一些聊天记录总是好的，为了实现这个效果，我们在wear设备扩充器中采用了添加页面的办法：它可以让你为主要提醒内容增加额外的页面，我们把聊天记录放入一个次级大的文本式提醒，然后把它加入到主要提醒中的第二页，并且手机端的提醒体验同时保持不变。关键代码实现如下：

```
Notification chatHistory =
	new NotificationCompat.Builder()
		.setStyle(
			new NotificationCompat.BigTextStyle()
				.bigText(getChatHistory()))
		.build();

firstPageNotification.extend(
	new NotificationCompat.WearableExtender()
		.addPage(chatHistory)
		.build());

```


###  Google Camera ###

![](http://7xi6qz.com1.z0.glb.clouddn.com/androidwearcamera.PNG)

 - Google Camera Base On
    - Wearable DataApi 
	- Wearable MessageAPI
	- WatchActivity 

1、wear端的google camera为相机App增加了好玩有趣的特性：**通过手腕来按下快门**，和很多具有远程遥控的高级相机原理类似，你把手机架在三脚架上或者靠在墙上又或者让其他人帮你拿着，然后你通过按住腕表的一个按键来捕捉一个画面**（代替现在正在热卖而很多大男生却不好意思在大街上用的自拍杆，嘿嘿）**。

2、相比于前面提到的Gmail和环聊（ta们只是用了 `Notification API` 来对手机端的消息做一个整合），Google Camera是一个相对个性而又具有和手机交互的特点，而且对于wear来说，仅仅只需要将**快门**这个按键特殊处理就行，所以，在wear端，光快门按键就霸占整个屏幕这一点也是可以容忍的。

3、通过 `Google Play Service`(以后简称 `GMS`)，实现相机的wear端和手机端通信，在相机手机app准备好拍摄时，手机端端会设置好数据项（意味着它已经做好了接受远程快门信息的准备），这种数据项由手表中wear app内置的服务读取，而wear端则会显示出快门按钮，按住按钮，把信息发回手机来激活手机端的快门键，最后，如何预览你刚才拍到的照片呢？很简单：手机端会创建一张缩略图，然后作为数据项中的一个asset发送回手表端，然后做为wear端全屏来预览。
关键代码如下：

**Setting a DataItem**

```
PutDataMapRequest dataMapRequest =
	PutDataMapRequest.create(DATA_ITEM_NAME);
dataMapRequest.getDataMap().putBoolean(
	FIELD_READY,cameraReady);
Wearable.DataApi.putDataItem(
	mGoogleApiClient,
	dataMapRequest.asPutDataRequest()
);
```
**WearableListenerService**

```
public class CameraListennerService
	extends WearableListennerService{
	@Override
	public void onDataChanged(DataEventBuffer dataEvents){
		for(DataEvent dataEvent:dataEvents){
		    if(dataEvent.getType()== 	DataEvent.TYPE_CHANGED){
			DataMapItem mapDataItem = 
				DataMapItem.fromDataItem(
					dataEvent.getDataItem());
			if(mapDataItem.getDataItem().getBoolean(FIELD_CAMERA_READY,false)){
			postNotification();
		}else{
		stopActivity();	
	}
	)
  }
 }
}
```
**Sending an Asset**

```
PutDataMapRequest dataMapRequest =
	PutDataMapRequest.create(DATA_ITEM_NAME);
dataMapRequest.getDataMap().putBoolean(
	FIELD_READY,cameraReady);

//在DataItem数据项中插入一个判断
if(previewBitmap != null){
	dataMapRequest.getDataMap().putAsset(
	 FIELD_PREVIEW,preview);
	)
}

Wearable.DataApi.putDataItem(
	mGoogleApiClient,
	dataMapRequest.asPutDataRequest()
);
```
### Google Maps ###

![](http://7xi6qz.com1.z0.glb.clouddn.com/androidwearmaps.PNG)

- Google Maps Base On
  - Voice Actions 
  - Custom Display Intent Notifications

1、在Google Maps导航期间，我们想要在手腕上提示导航，这个应用场景在走路的时候尤其有用，因为你要一直拿着你的手机走在大马路上会非常奇怪且占用你的双手，如果把手机放在你的口袋，转而看一下手表的描述来获知导航信息无疑更为便捷和实用。

2、在wear端，google想实现对布局和导航呈现精细的把握，特地搭建了一款wear版本的google maps wear App，让个性抽取式卡片成为限于本地的提醒，通过修改google map手机app的数据项，增加了用于下一次操作的描述与图标以及用于解释导航状态的信息，同时wear端的google maps也增加了这样的数据项，每次变动发生后，ta都会读取新数据，然后更新wear的卡片，提取卡片时，wear app采用可穿戴扩充器的新显示意图特性，你可以指定一个活动来在提醒卡片中绘制内容，这样我们想在卡片上画什么都是可以的，而不是受限制于标准提醒样式。
关键实现代码：

**Custom Notification with Display Intent**

```
Intent displayIntent = 
	createUpdateIntent(data, maneuverBitmap);
displayIntent.setClass(
	this,NotificationDisplayActivity.class);

PendingIntent displayPendingIntent = PendingIntent.getActivity(
	this,0,displayIntent,
	PendingIntent.FLAG_CANCEL_CURRENT);

Notification notification = builder.extend(
	new NotificationCompat.WearableExtender()
		.setHintHideIcon(true)
		.setDisplayIntent(displayPendingIntent)
		.setBackground(background)
		.addPage(secondPage)
		.build();

```

3、google map通过语音指令来开启导航进程，为了实现这一点，可
wear端的google map app会联手意图过滤器（ `Intent`）来为导航声音指令服务，然后需要在 `AndroidManifest.xml`中声明如下：

```
<activity
	android:name=".StarNavigationActivity"
	android:theme="@style/TranslucentTheme">
	<intent-filter>
	 <action
		android:name="android.intent.action.VIEW"/>
		<category			android:name="android.intent.category.DEFAULT"/>
	    <data
			android:scheme="google.navigation"/>
	</intent-filter>
</activity>
```

声明之后，就会产生一个像这样的意图，可穿戴App接收到这个意图之后，就会给手机上的google map发送一条信息，信息包括目的地和导航模式，手机google map app接收到这条信息之后，然后开始导航到目的地，然后就可以出发了，具体通信代码如下:

**Sending a Message**

```
private void startNavigation(Intent intent){
	
	String uriString = intent.getDataString();
	
	mGoogleApiClient.blockingConnect(
		Constants.TIMEOUT_MS,
		TimeUnit.MILLISECONDS);

	DataMap dataMap = new DataMap();
	dataMap.putString(FIELD_URI,uriString）；

	Wearable.MessageApi.sendMessage(
	   mGoogleApiClient,
	   mOtherNodeId,
	   Constants.MESSAGE_PATH_START_NAVIGATION,
		dataMap.toByteArray()).await();
	googleApiClient.diconnect();
}

```

**Receiving a Message**

```
public void onMessageReceived(
	MessageEvent messageEvent){
	if(messageEvent.getPath().equals(
		MESSAGE_PATH_START_NAVIGATION)){
	  DataMap requestData =
			DataMap.fromByteArray(
				messageEvent.getData());
			String uriString =
				requestData.getString(FIELD_URI);
			Intent navIntent = new Intent(
				Intent.ACTION_VIEW,
				Uri.parse(uriString));
			startActivity(navIntent);
	}
}

```


# 搭建云驱动的Android Wear Apps #


![](http://7xi6qz.com1.z0.glb.clouddn.com/djlblog_androidwear_cloud.JPG)


如图：

 - 1、首先，我们需要一款云服务来作为App的后端来进行数据推送和数据处理。
 - 2、其次，移动App会配合这项服务发出一个提醒，而你会在wear设备上看到，而且该提醒也会发送到任何相连的AndroidWear设备上。
 - 3、接着，当然就是AndroidWear App本身了，ta的特效是搭建在移动手机App之内，这样一旦手机App发出一条可以在手机上看见的提醒，这条信息也发给了Android Wear设备，现在的API也能够传送这些提醒，比如说触发回复行为，那么，我们是怎么实现ta们的呢？`Android Studio`实际上把你所需要的一切都给你了，包括用于搭建后端服务的工具包，当然还有Android手机App，现在你还可获得扩展包来操作Android Wear，云后端可以用一款外露的API来搭建，Android Studio工具包可以让你在Java下来进行此类操作，为你处理精细的细节把握，你可以写一段云端代码，通过使用属性，ta能够暴露出运行在你android app中的API，这些属性告诉Android客户端这些代码都是到底在干什么的，比如在执行一种叫quotesApi(引用API）时，并提供一种称为getQuote（获得引用）的方式，ok，一旦你现在搭建好了云服务，借助工具包，事实上你就可以自动创建客户端数据库来进入了，接下来，你要做的当然是搭建你的App了，那如何从你的App进入API呢？其实，这个分类已经自动帮你加载下来了，并且放入Maven库中了，这样你就可以直接在你搭建好的Gradle文件夹下涵盖它们了。

 - 4、最后，我们如何把它拓展来用于Android Wear呢？ 其实很简单，跟google map中的例子一样，通过修改notification和卡片的代码，使用wear端的api，让消息提醒和前端信息视图同时展现在手机客户端和wear端，此时的**手机App**就**变成了wear连接云端**的**中间件**。


# 写给设计师们：如何把握Android Wear下App的设计原则？ #

> PS：自己在DuWear项目组除了做RD研发之外，也对设计比较感兴趣，记得研发表盘那段时期，经常和搭档（MUX-UE-大侯）一起交流wear端的设计理念和心得，这里分享一些给想进军wear端的设计师朋友们：


**首先我们需要知道的是：**

 - Android Wear可以在多种不同的设备上运行，甚至在方形或圆形屏幕上也能正常运行，ta的UI非常简单，而且显示的内容也做了优化以适应小屏幕。
 - Android Wear的核心UI会自动排列卡片的优先级，非常简单，它整合了像Google Now，安卓手机提醒（Notification）和关联性App等资源，从而可以直接在wear端运行，开发师可以创建可以在这种流中正确显示的卡片，在任何点上，我都可以朝左滑动来查看每个项的更多信息，朝右滑动则会移除卡片，
 - 我们也给里面加入了向谷歌说话的能力，用户只需说一句：OK Google来搜索网络寻求答案，也可以进行我们称之为行为（actions）的语音指令，开发师可以深入开发这些语音行为。意味着用户可以直接对你的软件说话。
 - 可穿戴设备提供了一套前所未有的设计理念，
这即是机会，也是挑战，所以相对其他诸如手机或平板之类的设备，要清楚明白地搞懂它们之间的不同之处非常重要。
Android Wear刚好在正确的时间提供了正确的信息，让人们同时与虚拟和现实世界有更好的联系，信息内容会尽可能自动显示在信息流中，对于早已习惯打开App和退出App的我们来讲，这种对于模块的变化是相当巨大的。
## 设计思路： ##
 - **Contextual：**Android Wear会注意到周围环境，而且十分智能，这些设备会让人与计算机设备的亲密关系全新升级，Android Wear不要求用户的关注与输入，相反它会注意到用户所处的环境与状态，然后在正确的时间体贴地提供正确的信息，Android Wear会让人感觉信息及时，提议中肯，无微不至。
 
 - **Glanceable：**这些App只需要一瞥的时间，不算是可穿戴设备处于我们的视线边缘，它们也可以整天使用，高效的App会用最小的嗡嗡声来提供更多的信息提醒，并在细碎的关联信息提醒上做进一步优化，从而在一整天的碎片时间里得以应用。
 
 - **Low Interaction：**快速思考，一针见血，迅捷即时，而且几乎不用与设备互动，在保证小屏幕传送信息优势的同时，Android Wear着重与简单的互动，仅仅在非常必要的时候来需要用户来输入。而且绝大多数输入都是简单的点触、滑动和语音指令，而一般输入所需要的精细操控也得以避免。Android Wear手势简单，操作便捷而迅速

 - **Suggest&Demand：**最后，这些体验都与建议和指令相关，Android Wear就像一位出色的个人助理，它了解你，知道你的喜好，只有在绝对必要的时候才会打扰你，而且它总是近在手边，随时准备为你回答或完成任务。Android Wear贴心、礼貌、有问必答，它把周围世界与用户巧妙联系起来的同时，又极其尊重你的注意力，把你的焦距汇到重点项目上。
 - **Break It Down：**这里所呈现的机会并不是想象中的那样——把智能手机的UI缩小一下就塞进来,相反，需要考虑的是在设计过程中出现的基础性的问题，遵循这些原则之后，要是用户在使用你的产品操作任务时，产品出问题了怎么办？
 
接下来，我们来看一些**出色的设计案例**：
 
 - **Stream Apps**
 
![](http://7xi6qz.com1.z0.glb.clouddn.com/androidwearstreamapps.PNG)
 
 如图，系统中众多App的展现体现在那些垂直的卡片流之中，让小小的屏幕可以为用户尽可能多得展现更多的Wear应用，也是用户界面的核心所在。

 - **Main Interface**
 
![](http://7xi6qz.com1.z0.glb.clouddn.com/androidwearcarddesigner.PNG)
![](http://7xi6qz.com1.z0.glb.clouddn.com/androidweargooglenowcard.PNG)

 如图，这里我们看到主屏幕流中有一些卡片，像这种卡片就会在特定时刻，需要它们该出现的时候才出现。就像 `Google Now`一样，这些卡片无需启动或打开，相反，ta们会基于所在地、走路或者奔跑等活动、实际时间、用户的兴趣爱好以及其他因素来运行，你可以指定某项内容在合适的时候出现，这些卡片与环境息息相关，要注意看这些内容十分简洁，而且布局十分清楚，这些布局也经过了优化，让人能通过**眼角一瞥就能一目了然**。这些卡片也会遵循我们低互动（Low Interaction）的原则，它们没有大量的目录和按钮，用户可以直接滑动屏幕来进入另一页获取更多资料，事实上，通过Android Wear用户界面你应该使用像**滑动和全屏挥动**这样的大手势，不要在一个屏幕上放置多个需要点击的小按钮。


  - 更多Android Wear出色案例和设计资源请参考下文中的：[更多系列教程]()


# Android Wear搭建更高级别的UI #
可穿戴设备的App是Android的标准App，但遵循的是小屏幕的设计理念，它们全屏运行，没有系统UI或状态条，它们的开发过程与安卓App类似，不过在UI的开发理念上却稍有不同，最重要的就是要记住你的界面不必苛求小点触屏或精准的拖拽，举个例子，在可穿戴系统UI中，你会注意到对滑动操作的频繁使用，还有滚动条的运用，它之所以能够流畅运行，是因为它并不需要把注意力花在要去触摸屏幕某个精准的点上，为了帮助大家，Google提供了一个[可穿戴设备App的UI库](http://developer.android.com/training/wearables/ui/index.html)，它提供了异常丰富的元素来用于UI的设计。


![](http://7xi6qz.com1.z0.glb.clouddn.com/androidweargradpagerview.PNG)

这里，我想重点讲一下 `GridViewPager(网格多面控件)` ,ta与主页流类似，大家可以用ta来设计界面，ta与多面控件相似，但是可以水平和垂直同时移动，第一步就是布局的规划，下面这几行代码就是你主行为所需要的全部内容：

**res/layout/pager_example.xml**

```
<?xml version = "1.0" encoding = "utf-8"?>
<android.support.wearable.views.GridViewPager
	xmlns:android = "http://schemas.android.com"
	android:id = "@+id/pager"
	android:layout_width = "match_parent"
	android:layout_height = "match_parent"
/> 

```

在这里，配置多面控件的目的是用于扩展至整个屏幕的，下一步，我们需要一个衔接器（`Adapter`）

**FragmentGridPagerAdapter**

```
int getRowCount()
int getColumnCount(int row)
Fragment getItem(int row , int column)

int getCurrentColumnForRow(
	int row,int currentColum)

```
而这几行代码则是在用户使用导航时为用户提供每张页面所呈现的内容，在这个例子中，我会从一个由片段支持的基本类来进行扩展，要创建一个运行的`Adapter`只需要这三个方法，前两种定义了内容行（`row`）与页的可用大小，注意 `column` 的数量取决与行参数`row`,原因就在于每行可能都有不同的列数量，它的特性就是其选项可以控制每行在页面哪个位置来放置，与固定好的网格布局相比，它给滚动条在平等滚动与垂直滚动间切换提供了可能，在这一布局中，为了实现这点，Google还想了一些办法，但结果看起来就是一个无缝对接，Google为什么要这么做？Google认为每行内容都是单独的存在，在主页流中，这些就成了提醒或Google Now卡片，用户要从一个行为页进入到目标页而进行上下滑动的操作时，如果有不同的项，那么用户在操作时就会迷惑，为了解决这个问题，上翻或下翻总会回到第一列，这个也是网格多面控件的默认模式，为了对此做出调整，你可以用另一种方式进行覆盖，这种方式称为：所想即所见（用户想了解某项信息时，该信息的页面就会呈现在当前页或下一页），它也会提供当前列的位置，为了返回固定的移动系统，你可以选择返回列，或许你还想保存该行上次浏览的那一列，以便下次选择改行时可以直接返回到那一列。最后，最重要的一点就是getItem了，这个就是你要在页面上呈现出片段的位置了，这里，你只需要返回到你的内容片段，然后剩下的事情就由多面控件来处理了。只要需要，内容片段可以长久存储，然后在合适的时候，要么删除，要么重新进入，在这种方式跟多面控件非常类似，只是加了一个垂直的维度而已。


![](http://7xi6qz.com1.z0.glb.clouddn.com/androidwearcontentPage.PNG)

为了帮助大家建立自己的页面内容，Google提供了卡片内容片段，它可以自动应用不同的风格,从而与系统卡片搭配一致，而你要做的只是提供内容就可以，这种内容片段也有许多附加特性，首先，如果你有超过一页的内容，它就会显示滚动条让你拖动到正确的位置，你也可以让它作为一个单独页面开始，这样就可以通过触按来放大到全屏了，在这个案例中，内容溢出会自动得到处理，在你代码样本和文档中会找到更多细节的，还有一些事情要记住：每个页面尽量只放一个单独的行为，如果可能的话，整张卡片应该做成一个触按目标，而需要运行的行为则应该清楚明白，现在，剩下需要做的就是把这些东西整合起来，转接类会处理所有内容片段操作,所以需要给片段管理来一个参数，把转接代码加入到页面代码中就行了。

**SampleActivity.java**

```

public void onCreate(Bundle savedInstanceState){
	 setContentView(R.layout.pager_example);
	 mPager = findViewById(R.id.pager);
	 mAdapter = new ExampleGridPagerAdapter(getFragmentManager());
	 mPager.setAdapter(mAdapter);	
	}

```
# Android Wear 数据层API DevBytes #

首先我们需要知道手机端和wear端的连接是基于Android Wear数据层API的，一旦出现任何变化，核心数据层API会让你的掌上设备，与可穿戴设备自动同步数据，这种进程能够让你的设备充当数据发送器或接收器，或兼而有之，发送器设备设置一些准备发送的数据，一旦这些数据发生变动，那么这些变动就会发送给接收器，接受数据的设备反过来会在检测变动之后，会启动一个反馈功能来告诉程序数据已更新，处于两种设备之间的核心对象被称为**数据项**，它能有效提供设备所需要的数据存储，从基础面来讲，数据项包含一个有效负载对象，它是用来负载实际的数据的，还有一个是路径对象，它为数据项提供唯一的标识符字符串，它在接收端很重要，来表明具体哪些数据得到了更新。

数据项能在可穿戴设别与掌上设备间来回接受和发送小批量数据，例如使用可穿戴设备传感器来收集心率信息，并发送到用户的手机，与前14周的心率信息交叉想比较，并将结果图反映在手表上，因为电流限制的原因，数据项对象能承载小部分数据大约有100k，因此如果你希望数据项承载更大的数据量，你需要附加一个超大容量的对象，这样你就能发送大量的二进制数据到蓝牙传输，如图像，举个例子吧，用掌上设备应用软件下载一个图像，调整大小，然后发送到穿戴式设备上进行展示，掌上设备负责进程中所有的复杂的重量级操作，而穿戴式设备则呈现简单的结果，很不错的是，为了避免数据的重发，资产对象负责缓存数据和保存蓝牙带宽，这就意味着每个缓存的资产对象只能对应一种情况，如果你重复发送多次，也不会浪费带宽或者是电量，更简单的方面，数据有效载荷API也能提供信息API，这样就能完成普通任务，例如告知你的穿戴设备要执行`Activity`，或者是提醒手机切歌曲，在默认的情况下，这些信息可以通过远程过程调用，也就是说，一旦过程改变，则不能确保信息能被接收，但是如果你想要程序更复杂，可以将信息设置成提问（`request`）或回答（`response`）的形式，这样连接的另一方就会告知另外的设备，反过来，另外的设备会完成相关的工作，然后做出回答，相应地，提交答复。更多复杂的DataAPI细节可以到google android developer官网查看。


# Android Wear的提醒（Notification）新特性 #

![](http://7xi6qz.com1.z0.glb.clouddn.com/github_androidwear23.png)


我们来看看wear设备提醒的三个新方面吧：
 
 - 新的显示选项
 - 新的提醒行为
 - 高级自定义设置



这是一个提醒流，它能很好地获取信息并与用户交互，这里有不同外观不同尺寸的垂直提醒列表，获取信息时只需要向上滑动一下表盘就能办到，继续滑动的话就会显示出额外的卡片，信息流中的这些提醒会加入到安卓提醒API中去，如果你已经熟悉了这个API，你可能就会识别出它的一些特性了，比如在独立的屏幕上会出现正确的提醒行为，但跟平板对于手机类似的是，提醒是依然可以被取消的，只需要把提醒卡滑到边上然后释放即可，手机上的提醒会自动同步到你的手表上面，这可以让许多现有安卓APP来在可穿戴设备上发挥自己的价值，它们也可以增加行为和撤销，我们**支持许多现有的提醒风格**，比如**收件箱式、大图式和长文本式**，如果内容太长，用户可以按住提醒来扩展，为了让体验更加丰富，我们也增加了新API来自定义提醒，TA们就成了AndroidSDK和libs库中可穿戴设备扩展类的一部分了。

首先我们来看看，多页面提醒设计`Multi-page notifications`：

## 多页面提醒设计（Multi-page notifications） ##
![](http://7xi6qz.com1.z0.glb.clouddn.com/github_myblogmutinotifi.PNG)

这些页面可以进一步为单条提醒增加详细信息，通过滑动就可以进入，屏幕下方的提示会让你知道TA们停留在哪个片面上，因为页面仅仅是提醒对象，所以他们可以使用任意的提醒风格，

![](http://7xi6qz.com1.z0.glb.clouddn.com/github_myblogmutinotifi2.PNG)

要给一条提醒增加页面的话，使用可穿戴设计新拓展类来增加页面，如下代码段为内容增加了两页新页面：

**Add pages to a notification**

```

Notification page2 = ...
Notification page3 = ...

NotificationCompat.Builder builder = ...
builder.extend(
   new NotificationCompat.WearableExtender(
	.addPage(page2)
	.addPage(page3)
	.setBackground(bitmap)//设置位图
	.setHintShowBackgroundOnly(true));//隐藏图片
NotificationManagerCompat.from(ctx)
	.notify(builder.build());

)

```

总计就是三张卡片了，此外，你可以添加一张全屏图像作为页面，就用不着卡片了，这种办法对地图或照片之类的内容来说非常实用。

## 提醒堆栈（Notification Stacks） ##

![](http://7xi6qz.com1.z0.glb.clouddn.com/github_myblognotifistack.PNG)

它可以把多条提醒归类成一组，用户可以与整个堆栈交互，也可以进入到单独的项中，堆栈本身及子提醒也可以增加行为，这个特性对信息型APP来说非常方便，因为用户可能会一次性查看所有信息，或只看其中的一条，为了创建提醒堆栈，推送一条或多条子提醒，然后把它们全打上组密钥的标签，你可以采用`NotificationCompat.Builder`中的`setGroupMethod`（设置组方式）来实现这一点。来自同一APP相同组密钥的提醒推送会被归类到同一堆栈中，你也可以使用setSortKey(设置类密钥)来处理项，如果你喜欢为一个丛（`bundle`）选择背景图片和行为，你可以推送一个可选择的组概要提醒，在下面代码中，用户会看到为丛设置的“全部归档”行为，取保为每个提醒选择一个唯一的提醒ID或标签，以免它们在推送时会相互覆盖。

**Post group child notifications**

```

//for each child notification
NotificationCompat.Builder builder = ...
build.addAction(R.drawable.archive,
	"Archive",pendingIntent)
	.setGroup("my-group")
	.setSortKey("sort-key");
NotificationManagerCompat.from(ctx)
	.notify(builder.build());

```

目前展示的提醒行为全部使用了默认设置，可以作为附加页面增加到相应卡片之中，下图左手的手表展示了这一设置,把主卡滑走就出现了“播放”行为，右手的手表则展示了直接把行为添加进当前卡片的行为，这样这张卡片就可以直接点击了，使用可穿戴扩展器中的setContentAction(设置内容行为)来为卡片添加行为。这些行为就不会作为单一页面来显示了。

```

//Action Notification
NotificationCompat.Builder builder = ...
build.addAction(R.drawable.pause,"Pause",intent)
build.extend(new NotificationCompat.WearableExtend)
	.setContentAction()

```

![](http://7xi6qz.com1.z0.glb.clouddn.com/github_myblogmusicstatus.PNG)



![](http://7xi6qz.com1.z0.glb.clouddn.com/github_myblogmusicstatus0.PNG)

## 远程输入（Remote Input）##

远程输入则是提醒行为的另外一个新特性，在激活一个行为时，它可以让用户开启文本回复，设备会提供给用户一个短语或让用户从一些选项中进行选择，这种输入的结果就是将含有你意图的行为发送了出去，通过远程输入，AndroidWear与手机、平板或可穿戴设备的APP互动就变得非常简单了，下面的代码就显示出我们为回复行为增加了一个远程输入，用户在卡片流中点击这一行为时，系统就会在`Quick Reply`的标签下，提供给用户一个语音回复的行为，一旦文本回复转换完成而又得到了用户的同意，你的行为意图就会发出，而且目的已经包含在内了，

**Add RemoteInput to a notification action**

```
String EXTRA_QUICK_REPLY = "quick_reply";

NotificationCompat.Builder builder=...
builder.addAction(
	new NotificationCompat.Action.Builder(
		R.drawable.reply,"Reply",pendingIntent
	.addRemoteInput(
		new RemoteInput.Builder(EXTRA_QUICK_REPLY)
			.setLabel("Quick reply").build())
		.build());
...
```

你的意图接收器，可以是一个`Activity`，`Service`，`Broadcast`，就可以使用远程输入API意图功能中的`Get Results`，来重新恢复成目的文本了，在下面的代码中，quickReplyText变量会根据用户的输入来进行设置，在远程输入API中还有许多其他选项可以使用，支持的内容包括预设选择、允许或禁用、自由样式输入，还支持同一行为的多种输入等。


**MyActivity.java**

```
protected void onCreate(Bundle savedInstanceState){
	super.onCreate(savedInstanceState);
	Bundle results = RemoteInput.getResultsFromIntent(
		getIntent());
	if(results != null){
		CharSequence quickReplyText = 
			results.getCharSequence(
				EXTRE_QUICK_REPLY);
		}

}

```

## Custom Display Cards ##

标准的提醒模版或许并不足以展示你想在卡片中出现的内容，所以我们增加了一个API：set Display Intent（设置显示意图），它可以让你使用安卓活动来实时绘制提醒内容，这一特性只对可穿戴设备上运行的APP可用，而且这些APP所用的API需要20以上的版本，定义内嵌到自定义显示卡片的活动时，你必须首先把它标记为exported（导出），这个可以通过在活动中设置导出属性为true，或增加一个意图过滤器来完成，接下来，把这一新属性的“是否潜入”设置为true，这样可以防止活动嵌入到不该嵌入的事件中去，最后，设置关联任务`task Affinity`为空字符串，虽然触控输入并不会在信息流中传播，但这些活动与其他活动一样，可以包含相同内容，这样像按钮那样的控制就可能不再适合了，你的活动写入之后，你可以将其嵌入到信息流中来创建一条提醒，然后使用可穿戴扩展器中的`setDisplayIntent`（设置显示意图）方式来选择该活动，你可以为显示意图增加附加内容来通过活动所需要的任何数据。

**AndroidMainifest.xml**

```
<activity
	android:name="com.example.MyDisplayActivity"
	android:exported="true"
	android:allowEmbeded="true"
	android:taskAffinity="" />
```

下面就是自定义显示提醒的一些模版，信息流中的标准提醒会基于内容自动调整大小，但是自定义显示提醒则需要在不想要默认大尺寸的情况下提供一个尺寸，你可以使用可穿戴拓展器中的设置自定义尺寸预设，或是设置自定义内容高度等方式来选择尺寸。

![](http://7xi6qz.com1.z0.glb.clouddn.com/github_myblognotifiwearmoban.PNG)

## Notification Bridging ##

![](http://7xi6qz.com1.z0.glb.clouddn.com/github_myblogmobile_wear_noti.PNG)

通过设置这些自定义卡片，之前上面提到的`Notification API`可以同时用在可穿戴设备的APP的提醒创建，以及来自手机或平板APP上的提醒桥接，桥接过程在可穿戴设备上是自动进行的，但是这里还有**几种新API是用来自定义桥接行为的**，如下代码所示：首先，你可以使用提醒兼容设置中的新`set Local Only`（设置仅本地）来完全禁用提醒桥接，如果一个提醒仅相关当前设备，那它就很有用处了，第二个特效，就是增加提醒仅可穿戴设备可用的行为，它可以让你为手机和可穿戴设备选择单独的行为设置，仅可穿戴可用行为在可穿戴扩展器的类中添加。


**Disable bridging for a notification**

```
NotificationCompat.Builder builder = ...
builder.setLocalOnly(false);
```

**Add an action for phones,tablets,and wearables**

```
NotificationCompat.Builder builder = ...
builder.addAction(R.drawable.reply,"Archive",pendingIntent);
	.addAction
```

**Add an action for wearables only**

```
NotificationCompat.Builder builder = ...
builder.extend(new NotificationCompat.WearableExtender()
	.addAction(new NotificationCompat.Action(
		R.drawable.reply,"Reply",pendingIntent)));
```



PS：这里有博主自己曾经写过的一个运行在Android手机上的Demo，用来展示Wear端的Notification新特性:[在Github上获取](https://github.com/AndroidWearDemo/AndroidWearNotification)

也可以参考国外大神的一个例子：[Enhanced and Wearable-Ready Notifications on Android](http://code.tutsplus.com/tutorials/enhanced-and-wearable-ready-notifications-on-android--cms-21868)

# Android Wear下的全屏App设计理念 #

为Android Wear设计APP的许多技术因素，你们会觉得非常熟悉，因为它们跟普通的Android APP运行原理是一样的，不过呢，这里主要讲的是**两大不同点**：

## 让用户如何退出APP ##
 
在手机或平板上，用户会使用返回或主页键来推出APP，但这些按钮在Android Wear设备上都不会出现，相反，在wear app上，用户离开你的APP会有如下两种办法：**一种是把页面朝左滑动至边缘退出，另一种是长按APP退出**：
### 滑动退出： ###

通过Android Wear我们引入了一种新的窗口属性：
        
```
< style name ="AppTheme" parent = "Theme.DeviceDefault" >
	< item name = "android.windowSwipeToDismiss">true< /item >
< /style >
```

**即窗口滑动属性**，这种窗口属性可以运用在你具体的活动主题中，一旦窗口滑动退出属性设置为true，那么活动一旦从左滑动至右，它就会退出，这种滑动退出运行方式跟多面控件运行方式类似，如果活动中的内容本身就可以滚动，那么窗口就不会退出，除非用户滚动到该内容边缘后再次滑动，它可以让你创建一些非常出色、类似信息流的体验，这些体验也可以通过滑动来退出。所有的Android Wear APP要么使用设备默认主题，要么使用一个继承默认设备的主题，这样可以确保不同的主题风格都会在你的APP上正常运行，从而让它们在你的wear设备上看起来非常好，

**You get it by default**

```
<activity
	android:name=".ControlRobotsActivity"
	android:theme="Theme.DeviceDefault"
/>
```


当然，我们知道，有些APP没办法使用滑动退出的功能，比如，无限移动的地图应用时永远没有边缘的，如果你不想使用滑动手势，那么你可以通过吧滑动的退出属性设置为false，来在你的主题中禁用ta。对于无法通过滑动来退出的APP。我们可以使用第二个属性：即**长按退出功能**。
    
### 长按退出 ###

这就相当于是一个退出按钮的行为了，为了让用户知道你的APP可以长按退出，在APP首次运行时，你要给用户一个长按退出的提示。打开我们的wear设备，你会发现在屏幕任何地方出现长按行为，都会在APP上出现一个退出按钮。再按下那个按钮来退出活动，用户则会回到主页，为了让你的APP退出变得尽量容易，Google做了一个可以在大多数UI上运行的View,它叫退出覆盖视图。（dismiss overlay view）
     
**activity_control_robots.xml**
     

```
<android.support.wearable.view.DismissOverlayView
	android:id="@+id/dismiss_overlay"
	android:layout_height="match_parent"
      android:layout_width＝"match_parent"
     />
 ```


为了把它集成到你的APP中，首先要把它添加进你的XML活动层中，确保它**增加的位置一定是在其他布局之上的**你还要确保该视图的尺寸能够覆盖整个屏幕，把它高度和宽度设置成与父框架相匹配，这样它就能够确保全屏，而且处于最顶层了，现在我们来看看java类里面怎么写：

**ControlRobotsActivity.java**

```
public void onCreate(Bundle savedState){
	super.onCreate(savedState);
	setContentView(R.layout.activity_control_robots);  	mDismissOverlay = (DismissOverlayView)findViewById(R.id.dismiss_overlay);
	mDismissOverlay.setIntroText(R.string.long_press_intro);
	mDismissOverlay.showIntroIfNecessary();
     mDetector = new GestureDetector(this,new SimpleOnGestureListener(){
       public void onLongPress(MotionEvent ev){
       	mDismissOverlay.show();
       }
     });
```
    
 接下来是`TouchEvent`。  
   
```
public boolean onTouchEvent(MotionEvent ev){
	return mDetector.onTouchEvent(ev) | | super.onTouchEvent(ev);}   
```

在它的`onCreate`中把退出层从你的布局层中拉出来，然后设置为内省文本，这个文本会在第一次运行活动时显示出来，而且会显示在APP其他内容之上，用来告诉用户可以通过长按来返回主页，然后，使用`showIntroIfNecessary`(必要时显示内省文本)，它会，也只会在第一次运行该APP时显示这个内省层，，接下来，如果用户长按了你的app，我们就需要让它激活，使用`GestureDectector`(手势检测器）和`SimpleOnGestureListener`（简易手势接收器）,使用这些框架类会确保所有app感应到手势的时长，在你长按返回时，会激活布局层显示退出行为，会显示一个退出按钮，如果用户点击了该按钮，你的活动就会被结束，但如果你没有点击该按钮，那么这个退出层就会自行隐藏，等着下次出现的命令，最后，还是在你的活动中覆盖一层 `onTouchEvent` (触控事件)，然后让`reveiveTouchEvents` (接受触控事件)连通到`GestureDectector`（手势检测器）,如果 `GestureDetector` 返回为true，你也真的返回主页了，而且不用触动 `onTouchEvent` 方式的正常活动,相反如果为false，那就可以继续使用正常活动的触控。

## 如何设计和运用你的APP，让ta看起来在圆形屏幕(Moto 360)上很不错。 ##

  
![](http://7xi6qz.com1.z0.glb.clouddn.com/github_myblogmoto360_size.PNG)

首先，我们来看看360的屏幕维度吧，这是一个直径为320px的圆圈，下方有30px的`chin`，因此系统会认为它的尺寸为320x290px，在我们自己开发的过程中，我们意识到chin会将一些非计划中的结果导入到现有的布局中，比如我们来看一下信息流中的行为卡片，我们希望给屏幕中央放置一个行为图标，但我们给中央垂直点加了一个层重力机制之后，结果这个蓝圆偏移了15px，但我们还是希望中间的这个蓝圆最好能够处于整个屏幕的中央，在我们之前提到过的默认主题中，`windowOverscan`属性已经设置了，而且整个视图分级结构的源是320X320px，这就导致了你的APP顶级结构视图，依然认为是320X320，而非320X290，然后再把你的局布如预想般放在屏幕中央，如何检测你的活动是运行在圆形屏幕中的呢？你的视图会请求应用窗口插入`insets`,然后会返回一个窗口插入目标，它会告诉你屏幕的形状，在Moto360中，它会告诉你下方插入的窗口为30px，在任何地方只要你要围绕这个`chin`来布局，你就需要经常使用这个值，这里所使用的插入值，会确保你的APP在以后任何可穿戴设备上看起来都很漂亮，为了节省大家敲打这些通用代码的时间，Google增加了一个叫`WatchViewStub`的视图，它可以让你根据APP运行的不同屏幕来扩充一两种不同的布局，如果你想在屏幕上看起来与众不同，就可以使用`WatchViewStub`来作为任何视图分级就够的源，要使用的话，先在你的活动或者onCreate碎片中创建一个新的源，完成之后，你就需要给你的源加上两层布局（`Round、Rect`）,但是有一个问题需要注意：因为这些布局在视图在附加进结构分级前，并没有进行扩充，你就没办法进入子一级的视图，相反，附加一个OnLayoutInflatedListener(布局扩充收听器)，它可以在布局内层进行不合适的扩充时使用，退出布局视图和这个WatchViewStub都可以在可穿戴支持库`Wearable Library`中找到，如下：


```
public WindowInsets onApplyWindowInsets(View view,WindowInsets windowInsets){
	if(windowInsets.isRound()){
		Rect insets = windowInsets.getSystemWindowInsets();
	//insets.bottom = 30
	}
}
```

**ControlRobotsActivity.java**

```
public void onCreate(Bundle savedInstanceState){
   WatchViewStub stub = new WatchViewStub(this);
	stub.setRectLayout(R.layout.activity_control_robots_rect);
	stub.setRoundLayout(R.layout.activity_control_robots_round);
	stub.setOnLayoutInflatedListener(new WatchViewStub.OnLayoutInflatedListener(){
		@Override public void onLayoutInflated(WatchViewStub stub){
			stub.findViewById(R.id.start_invasion).setOnClickListener(mClick);
		}
	});
	setContentView(stub);
}
```

该库同时也提供叫做`盒状插入布局的新布局管理器`，它拓展了框架布局，让开发师能够同时在方形与圆形屏幕上使用同一布局。


# Android Wear表盘（WatchFace）设计与开发 #
不得不承认，Google在表盘方面上还是很鼓励第三方开发者去自由创作的，有别于
[Apple Watch不允许接入第三方watch face应用](http://www.leikeji.com/article?2264)的做法。


要做一款WatchFace应用，你先需要拓展一下`CanvasWatchFaceService`及其引擎，

[AndroidWear CanvasWatchFaceService](http://code.tutsplus.com/tutorials/creating-an-android-wear-watch-face--cms-23718)

![](http://7xi6qz.com1.z0.glb.clouddn.com/canvaswatchface.png)

## onCreate  &  onDraw
你可以加载并缩放任意图片，通过`onCreate`方式来设置表盘风格，它包含一些控制瞥视卡片模式 变量，如上图所示，这种模式是放置OK Google 状态条图标和其他东西的地方，然后你可以用`onDraw`来绘制一个表盘，在个方法中，我们要把显示在表盘的框架的每一帧都渲染出来，因为我们是在画布上绘制，所以我们可以用标准的位图或形状函数，因为这个代码是在每个框架下都能运行的，所以TA的运行状态我们要牢记在心里，

![](http://7xi6qz.com1.z0.glb.clouddn.com/githubblogliangzhongMode.png)

## 交互模式
在交互模式（`interactive`）下，你是可以绘制全彩色和动画的，模板默认每秒钟更新一次，如果你想让TA更新的更加频繁，举个例子，你想播放一个动画时，那么你就要做三件事了：

 - 第一，你要移除`mUpdateTimeHandles`(默认更新时间管理)，要不然`onDraw`只会一秒需要一次。
 
 - 第二，在第一次表盘可见时，需要触发`onDraw`方式，这样就可以让`onVisibilityChange`方式下的框架无效了。
 
 - 第三，你需要在`onDraw`方式最后使框架无效，这样会剔除掉`onDraw`循环，可以让动画流畅播放，现在表盘就会不断更新了，在使框架无效之前，很重要的一点就是查看你的表盘是否处于环境模式下，要不然的话更新循环就会不断在后台运行，就算在环境模式下也是如此，而这么做的话，会**极大地影响电池寿命**。


## 环境模式

在环境模式（`ambient mode`类似于手机的待机模式）下，使用的绘制颜色是有限制的，而且模板默认每分钟才更新一次。

在这个模式下，一般会选用两个模式，
 - 选择灰色图片或者黑白双色。
 - 移除那些更新频率超过1分钟1次的屏幕元素（eg:秒针）
 
 要检测手表有没有进入环境模式，你可以覆盖`onAmbientModeChange`方式，RD会发送实例变量来表明手表处于环境模式下并使当前框架无效，这样会触发重新绘制机制， 
 ```
  public void  onAmbientModeChanged
 	(boolean inAmbientMode){
 	// ...
 		mAmient = inAmbientMode
 	// ... 	
 	}	
 ```
 然后，在下一次的`onDraw`中，RD可以决定他们去做什么。

  ```
  public void onDraw(Canvas canvas ,Rect rect){
    //...draw code....
    if(!mAmbient){
    //additional drawing code....
    }
 }
 ```
 
当然AndroidWearWatchFace API还为开发者们提供 一些高级自定义的情景，来确保表盘在所有情况下都能流畅进行，特别说一下其中两个：

 - 第一个：一些Android Wear设置支持低比特率的环境模式，这就意味着只显示屏幕像素，甚至关掉它，只使用灰度设计，无法在这些屏幕上运行，这就是我们为什么要用黑白代替灰色设计的原因。为了判断设计是否支持低比特率，我们把`onPropertiesChange`进行覆盖。开发者可以查看手表是否支持低比特率的环境模式，

 
 ```
  public void onPropertiesChanged(Bundle properties)
 {
 	super.onPropertiesChanged(properties);
 	mLowBitAmbient=
 	properties.getBoolean
 		(PROPERTY_LOW_BIT_AMBIENT,false); 
 }
 ```
 
 
 - 第二个：表盘能显示瞥视卡片在屏幕下方，我们通常会围绕卡片边缘onDraw一个透明矩形区域——TA可以确保你不会与表盘设计有很差的交互体验，而且在环境模式下尤其重要，要是没有它，卡片信息会被表盘底部的刻度线给遮挡。
 
 这里博主为大家提供了一些[谷歌官方和自己编写的表盘Demo](https://github.com/AndroidWearDemo/AndroidWearWatchface)。

# Android Wear更多系列教程： #

> 博主认为，目前天朝的可穿戴社区仍处于起步阶段，很多资源还不丰富，但是天朝程序猿的力量是强大的，相信随着更多wear developers的加入，可穿戴的社区会愈来愈壮大，最后仅以自己微薄之力，为Android Wear开源做出一点贡献，希望能帮助到更多的人。> 


下面汇总了目前国内比较好的一些Android Wear资源，请参考：

 - 开发类
  - [Android Wear Google官方教程（请翻墙，或者自己搜镜像）](http://developer.android.com/wear/index.html)
  - [Android Wear Google官方教程 `穿戴猫`汉化版本](http://dev.seacat.cn/index.html)
  - [Android Wear `穿戴猫`社区原创基础教程](http://bbs.seacat.cn/forum-106-1.html)
  - [Android Wear - App Structure for Android Wear（应用结构)](http://www.tuicool.com/articles/F7Z3Yj)
  - [benhero博客_Android Wear开发学习指南](http://www.cnblogs.com/benhero/p/4273800.html)
  - [Android Wear_Hands-On](http://code.tutsplus.com/tutorials/introduction-to-android-wear-hands-on--cms-22157)
  - [AndroidWear CanvasWatchFaceService](http://code.tutsplus.com/tutorials/creating-an-android-wear-watch-face--cms-23718)
  - [Introduction to Android Wear: The Basics](http://code.tutsplus.com/articles/introduction-to-android-wear-the-basics--cms-22042)
  - [Enhanced and Wearable-Ready Notifications on Android](http://code.tutsplus.com/tutorials/enhanced-and-wearable-ready-notifications-on-android--cms-21868)
  - [Ask AndroidStudio Wear问答社区](http://ask.android-studio.org/?/explore/category-wear)
 
 - 设计类 
  - [Google 官方Android Wear设计教程（需要翻墙）](http://developer.android.com/design/wear/index.html) 
  - [Google 官方Android Wear表盘（WatchFace）设计教程（需要翻墙）](http://developer.android.com/design/wear/watchfaces.html)
  - [Google Android Wear 设计规范学习心得 ](http://note.youdao.com/share/?id=1a46e80cb5ea07b5c755d38b65ff9576&type=note)
  - [TencentOS智能手表表盘设计大赛](http://tencentos.ui.cn/)
  - [FaceRepo for WatchMaker/Facer表盘引擎收集网站](http://facerepo.com/app/)


# Google Play 上热门的Android Wear应用 #

> 博主收集了一些谷歌市场上比较热门的Android Wear应用，期待更多朋友的补充和意见，大家可以下载下来体验，翻不了墙的同学请默哀。

 - [22款很棒的Android Wear表盘应用](http://note.youdao.com/share/?id=ee853cdb7e4283bade26e485d6ca2c60&type=note)

 - [31款很棒的Android Wear应用](http://note.youdao.com/share/?id=0037856aa8e5e0c2476b9a9f4950baa5&type=note)

 - [Magic WatchFace神奇表盘应用](http://www.magicwatchface.com/zh_cn)


# 热门的可穿戴技术社区 #

> 收录了一些博主目前暂时所知的"可穿戴技术社区"，期待更多朋友的补充和意见！
 
 - 国内：
  - [穿戴猫](http://www.seacat.cn/)
  - [DuWear](http://duwear.baidu.com/)
  - [Ticwear](http://ticwear.com/)
  - [Tencent OS for Watch](http://watch.tos.cn/)
  - [手表控](http://www.watchkong.com/forum/forum.php)
  - [雷科技](http://www.leikeji.com/)
  - [可穿戴设备](http://wearable.hqbpc.com/)
  - [控哪儿网](http://www.kongnar.com/)
  - [出行精灵](http://www.mapelf.com/)
 - 国外：
  - 由于天朝特殊原因，等以后更新。 


#Android Wear相关产品宣传视频#

 - [Google：wear what you want](http://www.cgangs.com/article/3467?source=sinaweibo)
 - [Moto360创意广告](http://www.tudou.com/programs/view/jKv0PSWHdCY/)
 - [Moto360中文应用场景广告](http://baidu.fun.tv/watch/2542550633994583670.html)
 - [华为AndroidWear智能手表官方宣传片1](http://my.tv.sohu.com/us/243481507/79477160.shtml)
 - [华为AndroidWear智能手表官方宣传片2](http://my.tv.sohu.com/us/5747262/78630855.shtml)


#加入我们#

组织在这里(请猛戳链接)：[AndroidWear-CN](https://github.com/AndroidWearDemo)


![](http://7xi6qz.com1.z0.glb.clouddn.com/github_myblogAndroidWear-CN.png)






> **转载**请注明**出处+原文链接+原文作者**，侵权必究，谢谢！
> 持续更新中