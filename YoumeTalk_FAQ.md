# Talk SDK FAQ

## 语音问题

1. 某手机进入通话后，游戏内声音被完全屏蔽掉了？
>部分手机因为硬件设置的问题，会出现这种情况，请将具体手机型号告知游密技术支持，技术人员在后台配置下即可解决。

2. 进入通话后，通话声音和游戏背景音混杂到一起，通话效果不好，怎么办？
>用户进入通话时，会进行回声消除以避免杂音，此时若背景音和音效太高的话，回声消除会使语音被削除一部分，影响通话效果，且几路声音混杂一起难以清晰沟通。
>建议在进入通话时，游戏可以将背景音和音效自动降低；
>如果是直播/指挥场景，因为房间会播放音乐，建议将游戏背景音将为0；
>如果是组队通话场景，建议将游戏背景音降为20%~30%，音效降为50%
>同时，也建议游戏在设置中提供对通话、背景音、音效三路音量调节入口，便于用户根据自己需要实时调整。

3. 进入通话后，部分手机的游戏背景音和音效效果为何受到了影响？
>在通话模式下，会进行回声消除以避免杂音，背景音和音效会受到回声消除的影响，质量会略有降低，而不同机型的硬件有差异，有些能被感知，有些不能被感知。

4. 进入通话后，部分安卓手机上调音量的硬件按钮对游戏音量不起作用了？
>通话模式对应的手机通话音量，游戏背景音音效对应手机媒体音量。
>进入通话模式后，大多手机把通话音量和媒体音量都调用出来供用户调节。但部分安卓机音量键缺省只控制通话音量。
>对此，建议游戏在设置中提供对通话、背景音、音效三路音量调节入口，便于用户根据自己需要实时调整。

5. 测试时，两台机器在一起通话时，为什么会出现啸叫？
>距离，音量跟话筒的不同机型拾音灵敏度是影响啸叫的主要原因。拉开距离、降低音量或者使用耳机，都会降低啸叫。
>QQ或微信在一起通话时也会存此现象，在真实的用户使用场景中，多人在一起同时开启麦克风和扬声器通话的场景也比较少见，因此影响较小。

6. 使用语音流量消耗是多少？
>非静音时平均带宽：上行4.1KB/s,下行3.9KB/s。静音平均宽带：小于0.5KB/s

7. 一个频道能支撑多少人？服务器如何保证支撑海量用户？
>游密语音频道不限制人数，游戏可根据自己的游戏体验决定同一频道里可以有讲话权力的人数。
>服务器这边是分布式无状态部署，且有负载均衡策略，可以支撑大量的游戏在线
>每个游戏都会分配唯一的appkey，游戏之间相互隔离，不会影响游戏服务品质

## SDK包问题

1. 接入通话SDK后，包会增加多少？
>接入完成后，包体会增加1~2M

2. 你们对安卓和iOS的机型支持情况如何？
>Android支持4.0以上，iOS覆盖7.0以上

3. 请问你们支持海外吗？
>游密语音支持全球

4. 网络切换或网络不好的时候，SDK会自动重连吗，需要自己重新加入语音频道么？
>SDK会自动重连（一定次数），如果最终重连失败，则会以回调值YOUME_CALL_TERMINATED通知给游戏方，只有收到这个回调的时候才需要再重新调用加入语音频道的接口。

5. Windows 开发环境配置
>Cocos2d引入游密提供的dll、lib库和.h、.cpp源文件；Unity只需导入插件，想支持在Windows Editor环境直接使用的话联系游密提供个新的（也是临时的）YouMeVoiceAPI.cs文件。
>这里要注意，没有替换那个cs文件的话直接editor跑会报错

6. 真机编译环境配置
>一般参考SDK包内的文档说明即可完成配置。Cocos2d-x根据项目的自定义程度，配置上可能需要根据实际情况变通，关键是要正确配置库文件和头文件的引入。

7. Appkey和SecretKey如何获取
>打开 http://www.youme.im ，注册帐号，注册后即可在控制台自行添加应用，添加了应用以后就可以获取到对应的appkey和secretkey。


## 接入问题

1. 状态管理方面的建议【2.4.x版本】
>首先需要初始化，并且在收到初始化成功的回调后才能使用其它接口。
>其次在房间状态方面：
>1) 可包含当前房间ID（roomID)，下个房间ID（nextRoomID,初始值为空）,房间状态status（不在房间leaved，正在加入房间joining，在房间joined，正在离开房间leaving，初始值为不在房间leaved）;
>2) 每次调用joinConference的时候做个判断，只有status = leaved的时候能直接调用joinConference（这个时候status更新为joining），否则这个joinConference函数会本地返回失败，不会触发回调(status值不变)
>3) joinConference成功会触发回调，收到回调值YOUME_CALL_CONNECTED的时候，表明加入房间成功，如果status!=leaving，将status值更新为joined，否则status不变（status==leaving的时候说明已经用了离开房间
的操作，就不用管这个回调了）;如果收到回调值YOUME_CALL_FAILED,表明加入房间失败，如果status!=leaving，将status值更新为leaved，否则status不变（status==leaving的时候说明已经用了离开房间的操作，就不用管这个回调了）;
>4) leaveConference这个接口可以随时调用，只要判断status!=leaving，就会本地返回成功并触发回调，这个时候应该把status更新为leaving; 收到回调值YOUME_CALL_TERMINATED的时候，status更新为leaved，判断下nextRoomID
是否为空，空的话不用操作别的了，非空的话说明之前有想直接切换房间的动作，这个时候再joinConference(nextRoomID)就好。
>5) 想直接切换房间（从a房间到b房间），需要将nextRoomID更新为b,再手动调用离开a房间。
>6) status的不同状态可以展现不同的ui，可以体验北凉从主播频道切换到指挥频道的时候，ui上其实会动态显示（退出中－－－>进入中－－－>指挥）。退出中对应status的leaving，进入中对应joining,指挥则是joined

2. YOUME_EVENT_OTHERS_VOICE_OFFYOUME_EVENT_OTHERS_VOICE_OFF joinConference、init之类的接口没进入到回调怎么回事？
>查看该接口函数的直接返回值，只有为YOUME_SUCCESS的时候才会进入回调。返回值为YOUME_ERROR_WRONG_STATE时表明状态错误，一般是此时初始化还没成功或者进出房间操作还没完成所致。

3. 实现实时语音进入频道进行语音收发的流程简介
>init->onevent->YOUME_EVENT_INIT_OK ->JoinChannelSingleMode->onevent->YOUME_EVENT_JOIN_OK->SetSpeakerMute(false)->SetMicrophoneMute(false)
 
4. 状态管理方面的建议【2.5.x版本】
>1) 首先需要初始化SDK，初始化SDK建议到放启动之后做，在App整个生命周期只做一次，并且在收到YOUME_EVENT_INIT_OK的回调后才能使用其它接口，可以设置一个变量保存是否初始化成功的状态；
>2) 根据业务逻辑可以调用setReleaseMicWhenMute（设置是否关闭麦克风时释放录音模块，设置后可以在IM录音前关闭Talk麦克风以防止与IM录音冲突，另一方面做为听众也可以保持背景音乐的原生音质效果）；
>3) 调用加入房间的接口joinChannelSingleMode/joinChannelMultiMode，其中单房间模式是指同一时间只能加入一个房间，多房间模式可以加入多个房间，并可以切换对哪个房间讲话；
>4) 在收到YOUME_EVENT_JOIN_OK事件后表示加入房间成功，在这之后才可以调用改变扬声器/麦克风状态、设置音量、控制他人麦克风、是否监听麦克风或背景音乐、是否开启vad、设置是否通知别人麦克风和扬声器的状态、是否开启麦克风音量等级回调、设置输出到扬声器/听筒（如果没有听筒需求不应该调用该接口）、设置白名单用户、背景音乐管理、设置变声音调（默认关闭，如需开启请联系我们）、连麦控制等通话过程中所需要的功能；
>5) 可根据是否要放到后台通话决定在退到后台/唤醒时是否调用pauseChannel/resumeChannel；
>6) 结束通话调用离开房间接口，单房间为leaveChannelAll、多房间为leaveChannelMultiMode，离开成功后会收到YOUME_EVENT_LEAVED_ALL或YOUME_EVENT_LEAVED_ONE事件；
>7) 3-6步骤根据游戏开局场景、App打电话等场景可重复多次，准备完全退出应用时，调用反初始化接口，生命周期期间只做一次；

6. 进入房间后没有声音
检查硬件：是否静音、音量是否最大；检查接口：是否调用打开麦克风和扬声器开关之类的(要加入房间成功之后调用)；检查逻辑：房间里还有别人么（用户名不一样），加入房间真的成功了么（收到成功的回调connected）
没听到通话可能的原因：
>1) Join的时候要 不同 的userid 进入 相同 的channel；
>2) OnEvent收到到JOIN_OK事件后，setMicrophoneMute(false); setSpeakerMute(false);
>3) 如果有开启vpn，可先关闭vpn测试；
>4) Windows 常见的问题是可能没有正常连接麦克风，可以先用其他录音软件测试录音是否正常。

7. 实时语音的过程中想调用类似im按住说话的功能怎么办
>加入房间的接口参数needMic理解成占有麦克风设备。如果needMic为true，那进入语音频道后会全程占有麦克风设备（知道退出语音频道），这个时候你想用其他的sdk去抢mic是无法实现的（例如用im的按住说话啊之类的），有如下几个va方案可以释放mic设备：
在2.5.5.3772版本以下，在按住说话前可以调用 PauseChannel( ); 接口暂停会话（效果跟离开房间差不多，除了保持与网络端的心跳）,等需要占用回来的时候再调用ResumeChannel( );。注意这两个接口都有相关回调通知:YOUME_EVENT_PAUSED 和 YOUME_EVENT_RESUMED，收到通知后才完成暂停。
在2.5.5.3772版本及以上，最好调用ReleaseMicSync( )和ResumeMicSync( )来强制释放麦克风，在调用时最好设置冷却时间，避免短时间内频繁调用这两个接口。
另外一个方案是： 在加入房间之前调用SetReleaseMicWhenMute( true )；通话过程中需要释放麦克风时调用SetMicrophoneMute( true )；对麦克风静音的同时释放麦克风。

8. 网络切换或网络不好的时候，SDK会自动重连吗，需要自己重新加入语音频道么？
>SDK会自动重连（默认500次，基本上网络能恢复都可以重连成功），开始重连会通知：YOUME_EVENT_RECONNECTING ，重连结束会通知：YOUME_EVENT_RECONNECTED ，如果状态码不是 YOUME_SUCCESS ，表示最终重连失败。

9. 初始化和反初始化
>目前SDK的初始化和反初始化设计都是只能调用一次。建议在开启游戏的时候初始化，不推荐调用反初始化。

10. Cocos2d-x cpp/js/lua 项目为何要求添加Android Java的初始化语句
>针对Cocos2d项目，必须在项目第一个启动的AppActivity添加如下改动，并且必须保证游密的初始化代码优先于加载游戏so的代码执行，该要求是受限于部分Android（常见于4.x系统）系统对so加载顺序的处理，必须要先加载被依赖的so。
```
示例：
@Override
protected void onCreate(Bundle bundle)
{
//下面两个的调用顺序不能错
YouMeManager.Init(this);
super.onCreate(bundle);
//启动服务用来监控网络的变化
Intent intent = new Intent(this,VoiceEngineService.class);
startService(intent);
}
```

>11. 主播模式下监听怎么没有效果？
监听需要打开麦克风，并且戴耳机才有效果。

12. 日志文件位置
>安卓机：/sdcard/Android/data/包名/files/ymrtc_log.txt
>苹果机：用xcode把container下载下来，路径是 AppData/Documents/ymrtc_log.txt
>windows：我的文档／exe名/ymrtc_log.txt

13. iPhone上，进入语音房间之前，把手机上的静音按钮置于静音状态，游戏背景音是听不到的，但进入语音房间之后，游戏背景音就出来了，静音按钮对游戏背景音和语音都不起作用
>iPhone上有几种声音模式，包括 Ambient, Play, PlayAndRecord。其中，当播放的声音不是app主要的业务功能时（如游戏的背景音），选择的是Ambient模式（这是缺省的模式），在这种模式下，静音按钮是起作用的。当播放的声音是app的主要业务功能时，选择的是Play模式（如音乐播放器）或者PlayAndRecord（如网络电话），这时静音按钮是不起作用的。我们的语音使用的就是PlayAndRecord模式，所以，当进入语音房间时，因为模式从Ambient切换到了PlayAndRecord, 静音键就失去作用了。具体可参考苹果的官方文档： 
>https://developer.apple.com/reference/avfoundation/avaudiosessioncategoryambient 
>https://developer.apple.com/reference/avfoundation/avaudiosessioncategoryplayandrecord

14. iOS 设备上，有些设备进入房间后，声音会变大或者变小
>iOS上的音量控制跟android是类似的，分媒体，通话等不同音轨，但不像android那样可以让你分别在UI上设置，而是在播放某种音轨的时候音量键就对某条通道的声音起作用。进入房间之前，因为没有录音，播放的声音是媒体音量，这时音量键设置的就是媒体音量。进入房间后，启动了录音模式，系统会认为这时播放的声音就是通话音量了，所以音量就会自动调整为之前用户设置的通话音量。如果之前设置的通话的音量很大，而媒体音量很小，你就会听到声音变大了。可以这样实验一下，在进房间之前，把音量调大，再进房间，然后把音量调低；这是你退出房间，声音就变大了，再进入房间，声音就变小了。 相反，如果进房间之前，把音量调小，再进房间，把音量调大；这是再退出房间，声音就会变小，再进入房间声音就会变大。

15. Android 通话进行当中，如果来了电话，电话声音会有异常
>来电话时，MainActivity会收到onPause()事件，游戏app应该在onPause里面退出语音房间。打完电话后，MainActivity会收到onResume()事件，游戏应该在onResume里面重新进入语音房间。

16. 接入之后，发现很多Android手机声音都很小
>常见的原因是配置问题，需求检查一下 MODIFY_AUDIO_SETTINGS 这个权限是否有加上，如果没有这个权限，声音就会很小。 

17. 进入通话后，通话声音和游戏背景音混杂到一起，通话效果不好，怎么办？
>用户进入通话时，会进行回声消除以避免杂音，此时若背景音和音效太高的话，回声消除会使语音被削除一部分，影响通话效果，且几路声音混杂一起难以清晰沟通。 建议在进入通话时，游戏可以将背景音和音效自动降低； 如果是直播/指挥场景，因为房间会播放音乐，建议将游戏背景音将为0； 如果是组队通话场景，建议将游戏背景音降为15%~20%，音效降为40% 左右，也建议游戏在设置中提供对通话、背景音、音效三路音量调节入口，便于用户根据自己需要实时调整。

18. 进入通话后，安卓手机上调音量的硬件按钮对游戏音量不起作用了？
>通话模式对应的手机通话音量，游戏背景音音效对应手机媒体音量。 进入通话模式后，大多手机把通话音量和媒体音量都调用出来供用户调节。但部分安卓机音量键缺省只控制通话音量。 对此，可以参考我们提供的音量控制代码，在Java代码里同步通话和媒体音量。

19. 程序切换到后台，仍然能继续聊天，怎么处理？
>需要客户端自己响应切换后台事件来调取PauseChannel( ) /ResumeChannel( )接口。特别注意： 在Unity3D下，由于检查录音权限可能触发Unity的OnApplicationPause事件，PauseChannel 之前先判断一下是否已经完成ResumeChannel，避免死循环。

21. 可以修改麦克风音量吗？
>由于移动平台并没有硬件API可以调节麦克风灵敏度，而如果开放软件控制接口容易导致缩放失控，比如放大后超过音量最大值会被导致声音失真。所以一般都是调整播放端的音量，可以通过接口：setVolume(int volume)设置。

22. 如何知道其他人是否在讲话？
>可以通过接口 SetVadCallbackEnabled(bool enabled); 开启VAD检测。SDK在打开麦克风情况下，检测到开始讲话时，会给频道内其用户发送 YOUME_EVENT_OTHERS_VOICE_ON 事件，检测到结束讲话时会通知频道内其他用户YOUME_EVENT_OTHERS_VOICE_OFF 事件。

23. 如何获取自己讲话的音量？
>在通过接口：setMicLevelCallback( 10 ); 后 ，在开启麦克风的情况下，会收到 YOUME_EVENT_MY_MIC_LEVEL 事件，如果设置的10 ，YOUME_EVENT_MY_MIC_LEVEL 事件的errorcode 值会根据音量大小缩放到 0 -10的范围。（注意：setMicLevelCallback只有在加入房间后设置才有效）

24. 吃鸡类游戏接口调用建议
>1) 在YOUME_EVENT_INIT_OK事件后，建议先调用setReleaseMicWhenMute（关闭麦克风时保持原有游戏的高音质），再调用加入房间joinChannelSingleMode接口，其中YouMeUserRole为YOUME_USER_TALKER_FREE，bCheckRoomExist为false；
>2) 在加入房间成功事件YOUME_EVENT_JOIN_OK之后，立即调用setWhiteUserList（必须）、setMicrophoneMute：true（默认关闭麦克风，强烈建议）、setSpeakerMute：false（默认打开扬声器）、setAutoSendStatus（不建议）、setVadCallbackEnable（可选）、setMicLevelCallback（可选）、setVolume（可选）。
>3) 特别说明：setWhiteUserList接口是指将自己的语音只转发给白名单列表里的玩家，其它人收不到。其WhiteUserList参数建议写成小队成员+距离最近的敌人的形式，小队成员一定要排在前面，如果没有队友，可以只列出距离最近的敌人。一定要在加入房间成功之后立即调setWhiteUserList，否则会造成房间内100人同时说话的情况（默认开麦时）。白名单的最大长度游戏应该控制一下，我们在后台也可以做限制，建议不大于5个。
>4) setMicrophoneMute和setSpeakerMute接口的调用和UI显示保持同步。
>5) 运行期间初始化和反初始化应该只做一次，中间可以反复进出频道。

25. Android播放背景音乐接口返回失败
>1) 检查一下mp3文件是否存在
>2) 检查一下App是否有存储权限

26. 两个手机太近产生了啸音
>同时开启麦克风和扬声器，近距离时容易产生啸叫。原理是由于声音增益循环导致的
近距离的时候，戴耳机或者降低语音音量都可以减轻这个影响   实在不行我们这边可以后台配置整体降低声音，但是玩家就没法调更大了

27. 外置声卡
>sdk 2.6.5.494 以上支持外置声卡模式：
 
>1) 增加setForceDisableAudioProcess接口，设置是否强制关闭软件音频信号前处理，声卡内录模式播放音乐的场景中，戴耳机的情况下可以强制关闭，提高音乐质量
>2) 增加setChannelAudioMode接口，可设置普通通话（CHANNEL_AUDIO_MODE_CALL）和高音质通话模式（CHANNEL_AUDIO_MODE_HQ_MUSIC）
 
另外需要注意几个问题：
>1) 房间主持人的角色设置为YOUME_USER_HOST，只有他发的语音包才有高音质的效果（因为要播放音乐，需要更高的码率支持）；其它连麦者不需要太高的码率，角色设置为YOUME_USER_TALKER_FREE；不能上麦的角色设置为YOUME_USER_LISTENER即可
>2) 切换高音质时需要房间主持人触发，调用setChannelAudioMode，传CHANNEL_AUDIO_MODE_HQ_MUSIC，同时app层可以使用sendMessage接口发送信令，房间内其它人收到信令之后也调用setChannelAudioMode，传CHANNEL_AUDIO_MODE_HQ_MUSIC即可

28. 为什么有的机型声音特别小？
>部分Android手机上需检查一下 MODIFY_AUDIO_SETTINGS这个权限是否有加上，如果没有这个权限，声音就会很小。其他情况，您也可以将有问题的机型反馈给游密技术支持，技术人员在后台配置下即可解决。

29. iPhone上，进入语音房间之前，把手机上的静音按钮置于静音状态，游戏背景音是听不到的，但进入语音房间之后，游戏背景音就出来了，静音按钮对游戏背景音和语音都不起作用，是什么原因？
>iPhone上有几种声音模式，包括 Ambient, Play, PlayAndRecord。其中，当播放的声音不是app主要的业务功能时（如游戏的背景音），选择的是Ambient模式（这是缺省的模式），在这种模式下，静音按钮是起作用的。当播放的声音是app的主要业务功能时，选择的是Play模式（如音乐播放器）或者PlayAndRecord（如网络电话），这时静音按钮是不起作用的。我们的语音使用的就是PlayAndRecord模式，所以，当进入语音房间时，因为模式从Ambient切换到了PlayAndRecord, 静音键就失去作用了。为了改善用户体验，可以在启动应用的时候就把模式设为Play, 而不是缺省的Ambient，这样不管是否进入语音房间，用户都能有一致的体验。具体可参考苹果的官方文档：[AVAudioSessionCategoryAmbient](https://developer.apple.com/reference/avfoundation/avaudiosessioncategoryambient )、[avaudiosessioncategoryplayandrecord](https://developer.apple.com/reference/avfoundation/avaudiosessioncategoryplayandrecord)

30. iOS 设备上，有些设备进入房间后，声音会变大或者变小，是什么原因？
>iOS上的音量控制跟android是类似的，分媒体，通话等不同音轨，但不像android那样可以让你分别在UI上设置，而是在播放某种音轨的时候音量键就对某条通道的声音起作用。进入房间之前，因为没有录音，播放的声音是媒体音量，这时音量键设置的就是媒体音量。进入房间后，启动了录音模式，系统会认为这时播放的声音就是通话音量了，所以音量就会自动调整为之前用户设置的通话音量。如果之前设置的通话的音量很大，而媒体音量很小，你就会听到声音变大了。可以这样实验一下，在进房间之前，把音量调大，再进房间，然后把音量调低；这时你退出房间，声音就变大了，再进入房间，声音又变小了。

31. Android 通话进行当中，如果来了电话，电话声音会有异常，是什么原因？
>来电话时，MainActivity会收到onPause()事件，游戏app应该在onPause里面退出语音房间。打完电话后，MainActivity会收到onResume()事件，游戏应该在onResume里面重新进入语音房间。

32. 如何获取iOS手机的日志？
>iPhone上取日志的方法：用xcode, 进菜单 Windows->Devices， 点击左栏你的iPhone设备，在右下角会有一个“installed app“列表，点中你们的app，然后在最下边右击齿轮图标，会出现一个菜单，点击”Download Container", 把Container目录下载到本地，然后在本地打开Container包，日志文件相对路径 AppData/Documents/youme_av_log.txt（文件名也可能是ymrtc_log.txt）。

33. 如果已经加入了语音频道，还能加入IM的聊天频道么？能收听和发送聊天频道的语音消息吗？
>语音频道和IM聊天消息频道对应游密的两个SDK，因此，是没有任何冲突的。您可以在加入语音频道的同时，加入多个IM聊天频道。在这种情况下，收听聊天频道的语音消息是不受任何影响的，但是，录入语音消息要看具体情况，如果在语音频道中已经占用了麦克风设备（比如：自由通话模式，和主播／指挥模式下的主播／指挥），那么就不允许在IM聊天频道中录入语音消息

## 其他问题

1. 有人声检测的接口吗，比如A用户说了话，可以识别到，并返回到B用户？
>目前没有人声检测的接口。
建议可使用麦克风或扬声器开启/关闭检测做替代，仍然可以起到互动引导的作用。
在Joinconference函数中将autoSendStatus设为true即可实现。

2. 语音频道何时创建和消失？
>第一个人加入频道成功（收到回调值YOUME_CALL_CONNECTED）就相当于创建了频道；等这个频道所有的人都退出了，语音频道就消失了。

3. 主播Windows电脑上跑Android模拟器，玩家能听到背景音乐，但听不到主播声音，但用别的软件如yy是能发出声音的，是什么原因？
>解决办法是让主播退出游戏和模拟器，然后重启模拟器和游戏，声音就有了。

4. 已经进入A频道，再进入B频道，最终会进入到哪个频道？
>如果A频道没主动退出的话，B频道是会进入失败的，意味着还留在A频道；如果目的是希望进入频道B，那么需要先调用退出频道的逻辑，收到退出成功的消息后，再进入B频道。。

5. 如果因为某些原因，多次调用加入同一频道的操作（或离开频道等类似情况），而没有退出，会怎样？
>虽然SDK会让你后面的不合理的函数（重复加入频道等）操作直接返回失败，但游戏方应该做好状态管理，去积极避免这种情况发生。
