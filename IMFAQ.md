# IM SDK FAQ

## SDK包问题

1.  IM SDK支持的系统版本?

    > Android 2.2.x 及以上、iOS 7.0 及以上。

2.  SDK中的多人通话功能是否有人数限制？

    > 目前即时通讯服务是没有人数限制的。

3.  我们游戏用的是lua/Js/Java/OC开发的，有对应的SDK包吗？

    > 目前我们提供C++、Unity3D C#、Cocos2d-x、Cocos2d-js、Cocos2d-lua、Android-Java、OC、HTML5 版本的SDK。

4.  我们的游戏在海外运营，游密IMSDK能提供海外服务吗？

    > 游密IM SDK目前已经在美国、香港、日本、韩国、法国、爱尔兰、俄罗斯、澳大利亚、新加坡、印度、巴西等地区部署服务器，满足游戏厂商海外运营的需求。如果需要定制化部署服务，也可与商务沟通。

## AppKey问题

1.  如果获取Appkey和SecretKey？

    > 打开 http://www.youme.im ，注册帐号，注册后即可在控制台自行添加应用，添加了应用以后就可以获取到对应的appkey和secretkey。

2.  如何为我们的另一款游戏产品申请Appkey？

    > 登录[游密官网](https://console.youme.im/user/login)后，在控制台页面新建游戏，便可获得新游戏的Appkey。

3.  iOS与安卓是共用一个Appkey吗？

    > 每一个游戏下，所有平台、区服都共用一个Appkey。

## 接入问题

1.  `msc.so`这个文件有什么作用？

    > `msc.so`是语音转文字功能所需要的库文件，如果不需要语音转文字功能，则可将这个文件删掉。

2.  Android6.0以上系统发生崩溃如何解决？

    > 将原有的`android-support-v4.jar`文件替换成游密IMSDK包中的`android-support-v4.jar`通常可以解决。  

### 环境配置相关问题

1.  Cocos2d Windows 开发环境配置
 
    > Cocos2d引入游密提供的dll、lib库和.h、.cpp源文件；Unity导入插件后即支持在Windows Editor环境使用。
    
2.  真机编译环境配置
  
    > 一般参考SDK包内的文档说明即可完成配置。Cocos2d-x根据项目的自定义程度，配置上可能需要根据实际情况变通，关键是要正确配置库文件和头文件的引入。  
      
### SDK初始化相关问题

1.  初始化的时候发生崩溃如何处理？

    > 优先检查编译选项，`Application.mk`如果使用了非`APP_STL := gnustl_static` 或者`gnustl_shared`，就需要重新匹配客户的编译选项重新编译。

2.  为什Cocos2d-x 项目在部分安卓手机在启动时崩溃？

    > 请先确认java中的`com.youme.im.IMEngine.init(this)` 是否优先于加载游戏引擎so的代码。因为C++接口接入后，在链接时会给so添加依赖信息，被依赖的so必须优先加载，所以必须先调用游密的初始化代码。

3.  为什么部分手机找不到so文件？

    > 首先排查是否将`libyim.so`、`libmsc.so`两个文件放到armeabi中。若已经放置正确，问题仍然存在，请检查是否在项目中第一个启动的`AppActivity`中的`onCreate`中增加了以下Java代码 `com.youme.im.IMEngine.init(this)` 。 
    
4.  初始化与反初始化

    > 目前SDK的初始化都是只能调用一次，多次调用初始化(init)是安全的，切换帐号只需Logout再login即可；不需要调用反初始化。  
    
5.  Cocos2d-x cpp/js/lua 项目为何要求添加Android Java的初始化语句？

    > 针对Cocos2d项目，必须在项目第一个启动的AppActivity添加如下改动，并且必须保证游密的初始化代码优先于加载游戏so的代码执行，原因是 Cococs2d 引擎构建工具在链接期会在生成的so文件里记录各个动态库的依赖关系，必须先加载被依赖的so库，否则部分Android系统（常见于4.x系统）会出现无法加载动态库的情况，所以这个在Java中的AppActivity里的添加顺序一定要把游密的初始化放到最开始：
    ```
    示例：
    @Override
    protected void onCreate(Bundle bundle)
    {
    //下面的两个调用顺序不能错
    com.youme.im.IMEngine.init(this); 
    super.onCreate(bundle);
    }      
    ```
6.  缓存清理

    > 2.0.3.3307版本以前，还需要开发者自行清理。可以配合使用SetAudioCacheDir (for Cocos2d-x)或者SetAudioCachePath (for Unity)，指定缓存到同一个目录，然后启动游戏的时候清理下该目录。
   
7.  初始化和反初始化
    
    > 目前SDK的初始化和反初始化设计都是只能调用一次。多次调用初始化(init)是安全的，但是反初始化只能在测底退出游戏的时候调用。切换帐号只需Logout再login即可。

## 用户管理相关问题

1.  `iUserID`、`strPasswd`、`iChatRoomID`这些参数的设置是否有规则限制？

    > *   `iUserID`：仅可使用字母、数字，字母区分大小写；
    > *   `strPasswd`：不可为空字符串；
    > *   `iChatRoomID`：不可为空字符串，保证游戏内全局唯一。

2.  如何保持SDK的登陆状态和游戏保持同步 

    > SDK内部包含断线重试 ，但是不会一直重试，默认配置会重试5分钟左右，如果依旧连接不上，会通过OnLogout通知。要保持游戏和IM的登陆同步，需要再以下几个地方添加上重试登陆的逻辑：
    
    > 1）判断OnLogin返回的结果，如果不是Success（int值为0），就可以尝试再次调用登陆；
    > 2）在OnLogout的通知里，判断游戏是否在线，如果游戏是在线状态，就尝试再次调用登陆->进频道。注意如果是主动调用的logout接口，就不要重试。
       
## 频道管理相关问题

1.  世界频道怎么实现？

    > 首先为世界频道确定一个频道ID，当这个世界玩家登录游戏时，调用加入频道接口，进入这个频道即可。

2.  单个聊天频道是否有人数限制？

    > 理论上单个频道没有人数上限，但是过多的人同时在一个频道发言容易造成大量消息刷屏，为保证游戏体验，建议单个频道人数设置不超过10000人。
    
3.  两个客户端向同一个Room发送的消息 相互收不到消息如何解决？

    > 需按以下步骤进行检查：
    
    > 1) OnLogin返回状态码 Success
    > 2) OnJoinChatRoom返回状态码 Success
    > 3) 不要登陆相同的userid
    > 4) sendTextMessage时roomid的确是和JoinChatRoom相同的id
    > 5) 调用了SetMessageListen设置过消息回调监听    
    
4.  退出时需要调用levelroom吗？

    > logout就不需要再调用leaveRoom，logout之前调用leaveRoom不是必要的。 
    
5.  进出入房间通知

    > 用户进出房间，该房间内的用户可收到其他用户进出房间通知，默认服务端不下发该通知，需要后台配置IM策略启用。       
    
6.  语音、频道在退出时，需要调用levelroom吗？

    > logout就不需要再调用leaveRoom，logout之前调用leaveRoom不是必要的。

## 断网相关问题       
    
1.  断线后无法自动重连如何处理？

    > 当收到YMIM SDK发来的`Logout`通知后，需要重新调用一次登录接口。
    
2.  我断网之后，语音sdk本身会调用哪个接口呢？

    > 断网，IM会自动尝试重连，重连失败会通知OnLogout。如果游戏是断网后会登出再登陆，那也对应的logout IM再Login IM。    
    
3.  断网多久IM和语音会掉线呢？

    > 2.1.3版本断网很快就会检测到掉线， IOS上面稍微久一点，根据心跳而定。

4.  IM断线重连有回调吗？
 
    > 2.1.3.6131版本之前没有回调，除非重连失败，会通知OnLogout；之后的版本有重连回调。

## 语音相关问题

1.  实现语音消息发送的整个流程简介
   
    > init -> login -> onLogin -> joinRoom -> SendAudioMessage -> StopAudioMessage -> OnSendAudioMessageStatus
    
2.  实现语音消息接收的流程简介    

    > init -> login -> onLogin -> joinRoom -> OnRecvMessage -> Download -> OnDownload
    
3.  发送录音失败如何解决？

    > 常见的原因有一下几个：登陆没有成功、没有麦克风（windows比较常见）、没有调用StopAudioMessage、录音时间太短（建议录音限制 1s 以上）。另外可以发送日志给我们定位具体问题。    
    
4.  如何清理本地语音缓存文件

    > 2.0.3.3307版本以前，还需要开发者自行清理。可以配合使用SetAudioCacheDir (for Cocos2d-x)或者SetAudioCachePath (for Unity)，指定缓存到同一个目录，然后需要清理时调用ClearAudioCachePath或者ClearAudioCacheDir(for Cocos2d-x)接口清理该目录。   
    
5.  我完全不需要语音转文字功能，能否精简sdk？

    > 我们有同步编译的不带语音转文字功能的sdk，可以联系我们获取，QQ：2448236656。   
    
6.  录音反回PTT_Fail如何解决？

    > 一般有以下几种情况会返回PTT_Fail：
  
    > 1) 没有麦克风
    > 2) 上一次SendAudioMessage后还没有StopAudioMessage就又调用了StopAudioMessage
    > 3) 使用带语音识别的接口，但是并没有说话
    > 4) StopAudioMessag后立即调用了SendAudioMessage，由于我们的接口是全异步设计，每次录音之间最好有1s左右的间隔，这样才能正常重开讯飞语音识别引擎。    
    
7.  语音播放听不清或者语音转文字完全不正确如何解决？

    > 建议在播放语音时暂停或者调低游戏背景音乐音量。常见的听不清是因为播放语音时，游戏背景音乐没有停止，游戏音乐一般经过专门调制，声音连续且音量很高，会掩盖语音，导致语音听不清楚。
游密2.0.3.3xxx 及其以后的版本的 IM SDK的语音音量是经过AGC处理，音量水平对标微信语音音量。受限于录音设备精度，音量水平如果再提高会引起明显的失真。
还有重要的一点，建议在录制语音时暂停游戏背景音乐以及游戏音效。 

8.  讯飞录音返回10407错误

    > 这个目前发现过两例，都是因为客户已经接入过讯飞，替换为游密的SDK后，原先的讯飞登录代码并没有删除，导致讯飞的SDK包和帐号不一致。解决办法是查找讯飞的登录代码"SpeechUtility.createUtility"，找到后，把原来调用讯飞的代码都注释掉。   
    
9.  调用StopAudioMessage()后，有时不能收到OnStartSendAudioMessage()回调，是什么问题呢？

    > 1）检查调用StopAudioMessage()是否成功；
    > 2）使用语言识别的接口，如果没有检测到语音 / 录音时间太短或者其它异常会失败；出现这种情况只会通知OnSendAudioMessageStatus(), 正常发送语音才会收到OnStartSendAudioMessage()回调。
      发送语音不论成功还是失败都会都OnSendAudioMessageStatus()的通知；正常发送语音的情况下，在时间上收到OnStartSendAudioMessage()比OnSendAudioMessageStatus()快
    > 3）检查是否连续调用StopAudioMessage()和CancelAudioMessage()接口，如果连续调用时间间隔很短，是以CancelAudioMessage()的状态来回调，而CancelAudioMessage()没有回调。    
    
10.  iOS平台录音操作会导致APP瞬间卡顿大概1~3秒钟是什么问题？

    > 可以通过配置优化，就是第一次录音后保持在录音模式，不过iOS的静音键会失效，这是系统api的限制；或者全异步，但是开始200ms左右会录不到。
    
11.  如果打电话比较长时间语音会断么？

    > 时间长不会断，如果断了语音会一直重试连接（最大500次），如果失败会通知YOUME_EVENT_RECONNECTED ,错误码是非0，表示重连失败。    
    
12.  iOS下报bitcode相关错误如何解决？

    > 语音识别库不支持bitcode，ios下配置 Enable Bitcode为NO。
    
13.  调用SendOnlyAudioMessage返回YIMErrorcode_PTT_Fail是什么问题？

    > 1）可能是没有录音权限，或者写文件失败
    > 2）如果有录音权限，且没有其它调用错误，可能是麦克风开着（调用了实时语音），PS：此种可能针对安卓而言，因为安卓某些机型不支持同时两个录音对象
     此种可能的解决办法：调用Talk的 pauseChannel(); 暂停通话再启动录音    
    
    
14.  关于录音与语音播放控制的优化建议。

    > 如遇录音时还有语音播放，录音时先停止播放。同时建议录音时暂停游戏背景音。

15.  语音消息播放声音小如何解决？

    > 建议先提取录音文件在pc上播放检查是否正常。

16.  PTT对讲语音流量大小。

    > 新版sdk在不使用语音翻译的情况下，采用amr格式编码音频，60秒语音约50k byte流量。

17.  iOS系统在录音说话时发生闪退是什么原因？

    > 在SDK集成过程中需要为iOS10配置录音权限，详情参考对应语言下API Guides中相关指引，路径如：`Cocos`->`C++`->`API Guides`->`导入IM SDK`->`IOS：需要为iOS10以上版本添加录音权限配置`。

18.  语音消息发送很慢或者经常发送失败是什么原因？

    > 首先确认测试游戏产品的运营地区；若是海外运营的游戏，由于网络因素，出现语音消息发送慢或失败的情况属于正常现象，请使用质量较好的vpn进行测试；若是国内运营的游戏，请先确认测试环境网络状况是否正常，然后确认测试设备是否成功加入频道且处于一个频道内。若问题仍然存在，请将根目录下`YouMeIMLog.txt`与问题详细情况、问题发生的具体时间点发送给游密的商务同事，我们收到后会尽快给您答复。

19.  无法录音或点击录音按钮发生崩溃是什么原因？

    > 请检查是否打开录音权限；若打开后问题仍然存在，请将logcat日志发送给游密的商务同事，我们收到后会尽快给您答复。

20.  IM SDK是自动上传语音、播放语音吗？

    > 发送语音消息是自动上传的。SDK会提供语音文件的地址，由游戏自己进行播放操作。

21.  在iOS上没有开启我们游戏的麦克风权限，在发语音的时候没有弹出请求麦克风权限的提示，导致无法正常发送语音如何解决？

    > iOS如果在首次询问“是否允许访问麦克风”点击拒绝或者手动关闭麦克风权限的话，系统将不再弹授权的提示，需要游戏自己提示。如果`SendOnlyAudioMessage`返回的errorcode为PTT_fail，可以提示用户检查录音授权是否开启。

22.  语音消息播放

     > 从2.1.0版本开始提供了播放接口，可以播放本地wav音频文件。

23.  发送消息收到 UnknowError 错误
    
     > 常见于发送房间消息，房间号并不存在。修正的方法是往自己joinRoom的房间id发消息，就可以发送成功。
    
## LBS相关问题

1.  定位的功能，获取附近的人可以在PC上使用吗？

    > PC上不能使用，只有Android和IOS才行。
    
    > 使用顺序：
    > 1）SetUserInfo 设置自己的用户信息，其中的ServerID字段会用在步奏4的区分需要哪个服的玩家
    > 2）现获取当前自己的地理位置：GetCurrentLocation()
    > 3）OnUpdateLocation 回调成功后
    > 4）可以查询附近在线的玩家（需要先联系游密技术支持同学开通该功能）：GetNearbyObjects, 默认是获取附近在线玩家，需要的话也可以配置附近一定时间内在线的玩家。 
    
## 公告相关问题

1.  三种公告的获取方式差异是什么？

    > 如果公告下发时段未在线 在公告显示时段登录 可以拉到聊天框和置顶公告， 跑马灯只能下发在线情况下有通知，主动获取不到。

2.  进出入房间通知

    > 用户进出房间，该房间内的用户可收到其他用户进出房间通知，默认服务端不下发该通知，需要后台配置IM策略启用。       
    
## 常见错误码处理说明
*** 特别说明：所有参数外部要确保不会为null ***

1.  错误码`0`表示成功还是失败？

    > `0`表示成功， 其余错误码具体含义请参考错误码定义，路径如：`Cocos`->`C++`->`API Guides`->`错误码定义`。

2.  发送消息收到 UnknowError 错误如何解决？

    > 常见于发送房间消息，房间号并不存在。修正的方法是往自己joinRoom的房间id发消息，就可以发送成功。  
    
3.  IMAPI.Instance().Login：

    > 如果同步返回的错误码非Success，不会有OnLogin的回调。返回错误码：AlreadyLogin 表示已经是登陆成功的状态。OnLogin返回失败的情况，一般是长时间网络不通或者接口参数错误，调试能过后，只会在网络长时间无法连接和服务器异常的情况下可能发生，可以重试3-5次。

4.  IMAPI.Instance().Logout：

    > 可以认为肯定成功，除非本来就不在登陆状态。
    
5.  IMAPI.Instance().JoinChatRoom：

    > 如果同步返回的错误码非Success，不会有OnJoinChatRoom的回调，非登陆状态join会返回NotLogin错误码。

6.  IMAPI.Instance().LeaveChatRoom：

    > 可以认为肯定成功，除非本来就没有在这个频道。

7.  IMAPI.Instance().SendTextMessage 和 IMAPI.Instance().SendGift：

    > 同步返回错误码：NotLogin 没有登陆（可能是断线重连中），可以主动尝试调用一次login接口重新登陆；ParamInvalid 参数错误，一般开发期间调试通过后不会发生。返回Success表示成功启动发送，最终发送是否成功在OnSendMessageStatus 回调里判断。

8.  IMAPI.Instance().SendAudioMessage：
  
    > 同步返回错误码：PTT_Fail表示没有录音权限或者没有录音设备（比如windows没有插麦），NotLogin 没有登陆（可能是断线重连中），可以主动尝试调用一次login接口重新登陆；ParamInvalid 参数错误，一般开发期间调试通过后不会发生。返回Success表示成功启动录音成功，非Success不需要再调用StopAudioMessage()或者CancleAudioMessage()。

9.  IMAPI.Instance().StopAudioMessage： 

    > 同步返回错误码： NotLogin 没有登陆（也可能是断线重连中）,PTT_Fail表示没有录音权限（部分安卓设备只有异部判断是能正常录音，所以stop的时候也可能返回没有录音权限的错误）。返回Success表示录音正常，并开始语音前处理并发送语音消息，发送结果通过：OnSendAudioMessageStatus 通知是否发送成功，特别说明，如果OnSendAudioMessageStatus错误码是 PTT_SpeechTooShort 表示没有检测到说话的声音或者说话时间太短。
    注意：每次 SendAudioMessage 和 StopAudioMessage 之间，建议交互上做一个1s的限制，小于1s提示录音太短，这种情况下用调用 CancleAudioMessage，而不是 StopAudioMessage。

10.  DownloadAudioFile：

    > 同步返回 ParamInvalid 表示有不能为空的参数是空，返回Success后，会收到  OnDownload  回调，回调的错误码为Success表示下载成功，失败可能的错误码：ParamInvalid  传入的requestID不正确，找不到对应的消息、PTT_DownloadFail 表示下载失败（一般是网络原因导致）。      

## 其它

1.  日志文件位置

    > windows: c:\Users\用户名\AppData\Local\YouMeIMLogV2.txt
    > android: sd卡/Android/data/包名/yimcache/YouMeIMLogV2.txt
    > iOS: 请下载应用沙盒目录，参考：http://www.jianshu.com/p/a11437e68083 ,在沙盒的 Library/Caches/YouMeIMLogV2.txt
   
2.  在哪里可以看到更新日志？
 
    > SDK包带了一个更新说明。   

3.  关键词过滤功能里，如何自己修订需要的词条？  

    > 将需要过滤的词条一行一个录入一个txt文件里发送给游密的商务同事；收到后，我们会在24小时内为您配置完成。
    
4.  获取私聊消息记录包括自己发送的消息吗？
  
    > 包括。 
    
5.  有没有方法可以清除所有在线房间的数据和用户数据？
     
    > 频道不用清理都没关系，不在线2分钟就会IM 会清理出去，IM异常离线会通知OnLogout。   
    
6.  如何屏蔽用户的消息？

    > 如果是要禁言某个用户，可以用服务器端的禁言接口。sdk的屏蔽接口，是用户的黑名单，黑名单内的用户消息就都屏蔽了。  
    
7.  屏蔽用户消息包括私聊和频道消息吗？
 
    > 包括被屏蔽用户发的私聊和频道消息。  
    
8.  拉取频道聊天记录的消息会根据本地已存做一个选择拉取吗?

    > 调用QueryRoomHistoryMessageFromServer()会返回服务器能存储的全部消息记录（包含最新的消息记录），不会根据本地已存的消息记录做一个计算返回本地没有的。 
    
9.  房间默认支持2000人在线，频道需要更大人数的支持可以沟通商务再让我们开通。      

10. IMsdk更新到9022版本的时候.语音功能使用不了 日志显示对应的Leave  Fail（2017）  

    > 这个问题是因为语音引擎未启动导致的.让其检查下运维网站配置是否开启了语音转文字功能 以及语音识别是否是对应的识别   如果没有问题检查下是否有使用其他录音接口或者sdk等 都没有问题 可以让其检查打包出来的apk是否有对应识别的动态库以及jar包（一般都是对应的识别动态库缺失导致的,很多更新了sdk 但是新版的sdk可能有删除添加什么的 svn没有识别出来 需要手动添加后提交才行）。


