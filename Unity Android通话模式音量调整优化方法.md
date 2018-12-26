## 问题
Android 手机上，常见的一个问题是进入通话模式后，物理音量键只能调节通话音量，如果此时的游戏背景乐声太大或者太小，就没法很方便的通过音量键进行调节，只能打开游戏的设置面板进行软件调节。

## 解决办法
如果物理音量键在调节音量的时候，能同时设置媒体和通话音量，那么用户体验上会比较直观，相关实现参考代码如下：

这儿提供一个Android的通话音量和媒体音量同步调节范例代码，把以下代码放到项目的主Activity类里即可：

``` java
@Override
public boolean onKeyDown(int keyCode, KeyEvent event)   {
    checkAndInitVolumeControl();
    if(audioMgr!=null && audioMgr.getMode() == AudioManager.MODE_IN_COMMUNICATION){
        switch (keyCode) {
            case KeyEvent.KEYCODE_VOLUME_UP:
                addVolume();
                return true;
            case KeyEvent.KEYCODE_VOLUME_DOWN:
                cutVolume();
                return true;
            default:
                break;
        }
    }
    //return mUnityPlayer.injectEvent(event); //this is for Unity3D
    return super.onKeyDown(keyCode, event);   //this is for normal android
}

private AudioManager audioMgr;
float rate = 1;

public void addVolume(){
    checkAndInitVolumeControl();
    audioMgr.adjustStreamVolume(AudioManager.STREAM_VOICE_CALL, AudioManager.ADJUST_RAISE, AudioManager.FLAG_SHOW_UI);
    int callVolume = audioMgr.getStreamVolume(AudioManager.STREAM_VOICE_CALL);
    audioMgr.setStreamVolume(AudioManager.STREAM_MUSIC,(int)(callVolume * rate), AudioManager.FLAG_PLAY_SOUND);
}

public void cutVolume(){
    checkAndInitVolumeControl();
    int voiceVolume = audioMgr.getStreamVolume(AudioManager.STREAM_VOICE_CALL);
    audioMgr.adjustStreamVolume(AudioManager.STREAM_VOICE_CALL, AudioManager.ADJUST_LOWER, AudioManager.FLAG_SHOW_UI);
    int cutedVoiceVolume = audioMgr.getStreamVolume(AudioManager.STREAM_VOICE_CALL);
    if(voiceVolume == cutedVoiceVolume){
        audioMgr.setStreamVolume(AudioManager.STREAM_MUSIC,0, AudioManager.FLAG_PLAY_SOUND);
    }else{
        audioMgr.setStreamVolume(AudioManager.STREAM_MUSIC,(int)(audioMgr.getStreamVolume(AudioManager.STREAM_VOICE_CALL) * rate), AudioManager.FLAG_PLAY_SOUND);
    }
}

private void checkAndInitVolumeControl(){
    if(audioMgr==null){
        audioMgr = (AudioManager) getSystemService(Context.AUDIO_SERVICE);
        rate = (audioMgr.getStreamMaxVolume(AudioManager.STREAM_MUSIC)+0.0f)/audioMgr.getStreamMaxVolume(AudioManager.STREAM_VOICE_CALL);
        Log.d("volume", "rate:"+rate);
    }
}
```