polyv-android-sdk-demo
======================

主要演示polyv视频下载，本地播放，网络播放，视频拍摄和上传功能

配置
--
下载本案例，在eclipse创建android项目，选择"android project from existing code"

额外需要的包有
        ijkmediaplayer.jar
        ijkmediawidget.jar
	commons-codec-1.5.jar
	httpclient-android-4.3.3.jar
	httpmime-4.3.5.jar
	polyvSDK.jar
	universal-image-loader-1.9.3-SNAPSHOT.jar
案列所需要的权限
```xml
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```
注册Service
```xml
   <service  android:name="com.easefun.polyvsdk.server.AndroidService"></service>
```

描述
--
在MyApplication初始化PolyvSDKClient,设置token

```java
	PolyvSDKClient client = PolyvSDKClient.getInstance();
	client.setReadtoken(readtoken);
	client.setWritetoken(writetoken);
	client.setPrivatekey(privatekey);
	client.setUserId(userid);
	client.setSign(true);
	client.setDownloadDir(saveDir);//下载文件的目录
        client.setPort(port);//设置端口,默认是8033
        client.startService(getApplicationContext());//启动服务
```
下载视频
--
使用NewTestActivity类做演示，调用
```java
downloader = new PolyvDownloader(videoId, 1);
downloader.setPolyvDownloadProressListener(new PolyvDownloadProgressListener() {
					@Override
					public void onDownloadSuccess() {
						// TODO Auto-generated method stub
						Log.i("NewTestActivity", "下载完成");
					}

					@Override
					public void onDownload(long current, long total) {
						// TODO Auto-generated method stub
						Message msg = new Message();
						msg.what = DOWNLOAD;
						Bundle bundle = new Bundle();
						bundle.putLong("current", current);
						bundle.putLong("total", total);
						msg.setData(bundle);
						handler.sendMessage(msg);
					}
				});
				
//开始下载
downloader.start()
//删除					
downloader.deleteVideo(videoId, 1);
//删除下载目录
downloader.cleanDownloadDir();
```

如何用小窗口播放在线视频
--

使用IjkVideoActivity类做演示，调用方法：

```java
Intent playUrl = new Intent(NewTestActivity.this,IjkVideoActicity.class);
playUrl.putExtra("vid", videoId);
startActivityForResult(playUrl, 1);

```
  
如何全屏播放在线视频
--

使用IjkFullVideoActivity类做演示，调用方法：

```java
Intent playUrlFull = new Intent(NewTestActivity.this,IjkFullVideoActivity.class);
playUrlFull.putExtra("vid", videoId);
startActivityForResult(playUrlFull, 1);
```

使用IjkVideoView,IjkBaseMediaController构建播放器
--
1.自定义MediaController
  创建一个类MediaController，继承自抽象类IjkBaseMediaController，其中
  父类IjkBaseMediaController.java
  ```java
  public abstract class IjkBaseMediaController extends FrameLayout {
 
	    public IjkBaseMediaController(Context context, AttributeSet attrs) {
	        super(context, attrs);
	    }
	    public IjkBaseMediaController(Context context) {
	        super(context);
	      
	    }
	    protected abstract View makeControllerView();
	    protected abstract void initControllerView(View v) ;
	    public abstract void show();
	    public abstract void show(int timeout); 
	    public abstract void hide();
	    public abstract boolean isShowing();
	    public abstract void setMediaPlayer(MediaPlayerControl player);
	    public abstract void setAnchorView(View view);
	   
	    public interface MediaPlayerControl {
	        void start();
	        void pause();
	        int getDuration();
	        int getCurrentPosition();
	        void seekTo(long pos);
	        boolean isPlaying();
	        int getBufferPercentage();
	        boolean canPause();
	        boolean canSeekBackward();
	        boolean canSeekForward();
	    }
}
  
  ```
  MediaController.java 中的关于View布局的
```java
  protected void initControllerView(View v) {
        mPauseButton = (ImageButton) v.findViewById(R.id.mediacontroller_play_pause);
    	if (mPauseButton != null) {
            mPauseButton.requestFocus();
            mPauseButton.setOnClickListener(mPauseListener);
        }
       mFfwdButton =(ImageButton)v.findViewById(R.id.ffwd);
        mFfwdButton.setOnClickListener(mFfwdListener);
        mRewButton=(ImageButton)v.findViewById(R.id.rew);
        mRewButton.setOnClickListener(mRewListener);
        btn_boardChange=(ImageButton)v.findViewById(R.id.landscape);
        btn_boardChange.setOnClickListener(mBoardListener);
        btn_videoChange=(ImageButton)v.findViewById(R.id.videochange);
        btn_videoChange.setTag("0");
        btn_videoChange.setOnClickListener(mVideoListener);
        mPreButton=(ImageButton)v.findViewById(R.id.prev);
        mNextButton=(ImageButton)v.findViewById(R.id.next);
        if(isUsePreNext){
    	  mPreButton.setVisibility(View.VISIBLE);
    	  mNextButton.setVisibility(View.VISIBLE);
        }
        mPreButton.setOnClickListener(mPreListener);
        mNextButton.setOnClickListener(mNextListener);
        mProgress = (ProgressBar) v.findViewById(R.id.mediacontroller_seekbar);
        if (mProgress != null) {
            if (mProgress instanceof SeekBar) {
                SeekBar seeker = (SeekBar) mProgress;
                seeker.setOnSeekBarChangeListener(mSeekListener);
                seeker.setClickable(false);
                seeker.setThumbOffset(1);
            }
            mProgress.setMax(1000);
        }

        mEndTime = (TextView) v.findViewById(R.id.mediacontroller_time_total);
        mCurrentTime = (TextView) v.findViewById(R.id.mediacontroller_time_current);
    }
```
  具体实现方式参考本案例中的MediaController.java


2.视频播放代码调用
```java
                videoview=(IjkVideoView)findViewById(R.id.videoview);
		progressBar =(ProgressBar)findViewById(R.id.loadingprogress);
		videoview.setMediaBufferingIndicator(progressBar); //在缓冲时出现的loading
		mediaController = new MediaController(this,false);//
		mediaController.setAnchorView(videoview);
		videoview.setMediaController(mediaController);
		videoview.setVid(vid);//		
				

```
