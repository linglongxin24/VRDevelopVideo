#【Android VR开发】二.给用户播放一个360°全景视频

>VR即Virtual Reality虚拟现实。虚拟现实技术是一种可以创建和体验虚拟世界的计算机仿真系统它利用计算机生成一种模拟环境是一种多源信息融合的交互式的三维动态视景和实体行为的系统仿真使用户沉浸到该环境中。
那么，如何在Android中去开发VR功能的APP呢？我们利用谷歌提供的开源SDK去实现一个360°全景视频的功能

![](https://github.com/linglongxin24/VRDevelopVideo/blob/master/screenshot/Screenshot_2016-12-23-16-01-13-734_VRDevelopVideo.png?raw=true)
![](https://github.com/linglongxin24/VRDevelopVideo/blob/master/screenshot/Screenshot_2016-12-23-16-01-41-994_VRDevelopVideo.png?raw=true)

#一.在build.gradle中引入谷歌VR的SDK依赖

```gradle
     compile 'com.google.vr:sdk-videowidget:1.10.0'
```

#二.注意支持的最小SDK

```gradle
  minSdkVersion 19
  targetSdkVersion 25
```

#三.界面布局文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="cn.bluemobi.dylan.vrdevelopvideo.MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Android开发VR360度全景视频" />

    <com.google.vr.sdk.widgets.video.VrVideoView
        android:id="@+id/vr_video_view"
        android:layout_width="match_parent"
        android:layout_height="250dp"></com.google.vr.sdk.widgets.video.VrVideoView>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <ImageButton
            android:id="@+id/play_toggle"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:background="@android:color/transparent"
            android:paddingStart="0dp"
            android:src="@drawable/pause" />

        <SeekBar
            android:id="@+id/seek_bar"
            style="?android:attr/progressBarStyleHorizontal"
            android:layout_width="0dp"
            android:layout_height="32dp"
            android:layout_weight="8" />

        <ImageButton
            android:id="@+id/volume_toggle"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:background="@android:color/transparent"
            android:paddingStart="0dp"
            android:paddingTop="4dp"
            android:src="@drawable/volume_on" />
    </LinearLayout>
</LinearLayout>


```

#四.加载360°全景视频

```java
    /**
     * 加载360度全景视频
     */
    private void load360Video() {
        vr_video_view = (VrVideoView) findViewById(R.id.vr_video_view);
        seek_bar = (SeekBar) findViewById(R.id.seek_bar);
        volume_toggle = (ImageButton) findViewById(R.id.volume_toggle);
        play_toggle = (ImageButton) findViewById(R.id.play_toggle);

        /**设置加载设置**/
        VrVideoView.Options options = new VrVideoView.Options();
        options.inputType = VrVideoView.Options.TYPE_STEREO_OVER_UNDER;
        /**
         * 设置加载监听
         */
        vr_video_view.setEventListener(new VrVideoEventListener() {
            /**
             * 视频播放完成回调
             */
            @Override
            public void onCompletion() {
                super.onCompletion();
                /**播放完成后跳转到开始重新播放**/
                vr_video_view.seekTo(0);
                setIsPlay(false);
                Log.d(TAG, "onCompletion()");
            }

            /**
             * 加载每一帧视频的回调
             */
            @Override
            public void onNewFrame() {
                super.onNewFrame();
                seek_bar.setProgress((int) vr_video_view.getCurrentPosition());
                Log.d(TAG, "onNewFrame()");
            }

            /**
             * 点击VR视频回调
             */
            @Override
            public void onClick() {
                super.onClick();
                Log.d(TAG, "onClick()");
            }

            /**
             * 加载VR视频失败回调
             * @param errorMessage
             */
            @Override
            public void onLoadError(String errorMessage) {
                super.onLoadError(errorMessage);
                Log.d(TAG, "onLoadError()->errorMessage=" + errorMessage);
            }

            /**
             * 加载VR视频成功回调
             */
            @Override
            public void onLoadSuccess() {
                super.onLoadSuccess();
                /**加载成功后设置回调**/
                seek_bar.setMax((int) vr_video_view.getDuration());
                Log.d(TAG, "onNewFrame()");
            }

            /**
             * 显示模式改变回调
             * 1.默认
             * 2.全屏模式
             * 3.VR观看模式，即横屏分屏模式
             * @param newDisplayMode 模式
             */
            @Override
            public void onDisplayModeChanged(int newDisplayMode) {
                super.onDisplayModeChanged(newDisplayMode);
                Log.d(TAG, "onLoadError()->newDisplayMode=" + newDisplayMode);
            }
        });
        try {
            /**加载VR视频**/
            vr_video_view.loadVideoFromAsset("congo.mp4", options);
        } catch (IOException e) {
            e.printStackTrace();
        }
        /**设置声音按钮点击监听**/
        volume_toggle.setOnClickListener(new View.OnClickListener() {
            public void onClick(View v) {
                setIsMuted(!isMuted);
            }
        });
        /**设置播放暂停按钮点击监听**/
        play_toggle.setOnClickListener(new View.OnClickListener() {
            public void onClick(View v) {
                setIsPlay(!isPlay);
            }
        });

        /**设置进度条拖动监听**/
        seek_bar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            /**
             * 进度条拖动改变监听
             * @param seekBar 拖动条
             * @param progress 进度
             * @param fromUser 是否是用户手动操作的
             */
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                if (fromUser) {
                    /**调节视频进度**/
                    vr_video_view.seekTo(progress);
                }
            }

            @Override
            public void onStartTrackingTouch(SeekBar seekBar) {

            }

            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {

            }
        });

    }

    /**
     * 设置声音开关
     *
     * @param isMuted 开关
     */
    private void setIsMuted(boolean isMuted) {
        this.isMuted = isMuted;
        volume_toggle.setImageResource(isMuted ? R.drawable.volume_off : R.drawable.volume_on);
        vr_video_view.setVolume(isMuted ? 0.0f : 1.0f);
    }

    /**
     * 设置播放暂停
     *
     * @param isPlay 播放暂停
     */
    private void setIsPlay(boolean isPlay) {
        this.isPlay = isPlay;
        play_toggle.setImageResource(isPlay ?R.drawable.pause:  R.drawable.play );
        if(isPlay){
            vr_video_view.playVideo();
        }else{
            vr_video_view.pauseVideo();
        }
    }

```

#五.[GitHub](https://github.com/linglongxin24/VRDevelopVideo)