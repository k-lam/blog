###数字音频基础
[http://wenku.baidu.com/view/03c9370790c69ec3d5bb7572.html](http://wenku.baidu.com/view/03c9370790c69ec3d5bb7572.html)

振荡产生声音振幅决定音量，频率决定音调。模拟信号到数字信号的转换是通过A/D来转换的。其中采样率就是每秒钟采样的次数，越高越真实，当然越高，文件大小越大。不过这样采样，肯定就是离散的点了。为了有更加直接的感受，直接上代码！(如果有matlab就好了！！！)

```java
	public class test1 extends Activity implements OnClickListener {

	volatile boolean playing = false;
	final static int SAMPLE_RATE = 11025;//采样频率
	int minSize = 0;
	AudioTrack audioTrack;//播放类
	float frequency = 220;//震动频率
	float angular_frequency = (float)(2 * Math.PI) * frequency / SAMPLE_RATE;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.test);
		minSize = AudioTrack.getMinBufferSize(SAMPLE_RATE,
				AudioFormat.CHANNEL_OUT_MONO, AudioFormat.ENCODING_PCM_16BIT);
		audioTrack = new AudioTrack(AudioManager.STREAM_MUSIC, SAMPLE_RATE,
				AudioFormat.CHANNEL_OUT_MONO, AudioFormat.ENCODING_PCM_16BIT,
				minSize, AudioTrack.MODE_STREAM);
		findViewById(R.id.btn_play).setOnClickListener(this);
		findViewById(R.id.btn_play2).setOnClickListener(this);
	}
	int a = 1;
	@Override
	public void onClick(View v) {
		switch (v.getId()) {
		case R.id.btn_play:
			playing = true;
			Task.callInBackground(new Callable<Void>() {

				@Override
				public Void call() throws Exception {
					audioTrack.play();
					short[] buffer = new short[minSize];
					float angle = 0;
					//while (playing) {
						for(int i = 0; i < buffer.length;i++){
							//可以看到，a越大，振幅变小，声音会变小，
							buffer[i] = (short)(Short.MAX_VALUE * ((float)Math.sin(angle)) / a);
							angle += angular_frequency;
						}
						audioTrack.write(buffer, 0, buffer.length);
					//}
					return null;
				}
			});
			break;
		case R.id.btn_play2:
			playing = false;
			audioTrack.stop();
			a += 2;
			break;
		default:
			break;
		}
	}
	}
```

在音频开发中，可能会经常见到playback这个词，回放。什么事回放？其实"back"（"回"）的意义在于，模拟信号经过A/D转换成数字信号，而在程序中对数字信号play**back**，通过D/A,转回模拟信号播放出来

pcm大小计算 采样率*采样时间*采样位深/8*通道数
采样率通常会用：44100, 22050 and 11025.

###android里的音频 类应用
参考《android多媒体开发 高级编程》

#####[MediaPlayer](http://developer.android.com/reference/android/media/MediaPlayer.html)
简单播放单个.mp3 .wav文件,可以设置音效。视频也可以播放

	MediaPlayer mediaPlayer = MediaPlayer.create(test.this, R.raw.bell);
	mediaPlayer.setOnPreparedListener(new OnPreparedListener() {
					
		@Override
		public void onPrepared(MediaPlayer mp) {
			mediaPlayer.start();
		}
	});

#####[MediaRecorder](http://developer.android.com/reference/android/media/MediaRecorder.html)
录制音频成为wav mp3这样的encoded的文件，视频也可以。

#####[AudioRecord](http://developer.android.com/reference/android/media/AudioRecord.html)
录制音频，二进制格式的

#####[AudioTrack](http://developer.android.com/reference/android/media/AudioTrack.html)
The AudioTrack class manages and plays a single audio resource for Java applications. It allows streaming of PCM audio buffers to the audio sink for playback.

也就是说，要对二进制数据进行修改，然后播放的，用这个类。


#####[SoundPool](http://developer.android.com/reference/android/media/SoundPool.html)

SoundPool是为了播放很多不同声音的场景的，如游戏。SoundPool可以load很多不同的音频，从pool可以看出，是作了缓存处理的。

	void initMusic(){
        soundPool = new SoundPool(10, AudioManager.STREAM_MUSIC, 0);
        Task.callInBackground(new Callable<Void>() {
            @Override
            public Void call() throws Exception {
                String[] fileList = null;
                try {
                    fileList = getAssets().list(SUBDir);
                } catch (IOException e) {
                    e.printStackTrace();
                }
                int i = 0;
                try {
                    for(String file : fileList){
                        Log.i("debug", file);
                            mSoundIds[i++] = soundPool.load(getAssets().openFd(SUBDir + "/"+file), 1);
                            //soundPool.setOnLoadCompleteListener(completeListener);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                    Log.e("debug", e.getMessage());
                }
                return null;
            }
        }).continueWith(new Continuation<Void, Void>() {
            @Override
            public Void then(Task<Void> voidTask) throws Exception {
                soundPool.play(mSoundIds[id],0.4f,0.4f,1,0,2.0f);
                return null;
            }
        });
	}

我们会注意到一个问题，for循环来load文件，如果文件很多，速度回变得很慢。但多线程好像有不适用，因为猜想主要都是争抢系统总线来传输数据。所以通过代码尝试验证


验证过程：

用N条线程加载88个音频，如N = 2，两个线程加载，线程1加载0-43，线程2加载44-87

通过多次测试，用多线程的方法没有改善到加载速度。

*引申问题：对于io密集型的程序，应该如何优化？*

`注：load函数是native实现，暂时还没找到源码。猜想是包括硬盘读入，decode成pcm这两个耗时操作，而decode操作会不会由声卡或DSPs来做？`

#####[AudioEffect](http://developer.android.com/reference/android/media/audiofx/AudioEffect.html)

音效类，据说是根据OpenSL es来的
	
	EnvironmentalReverb  eReverb 
	= new EnvironmentalReverb(0,mediaPlayer.getAudioSessionId());
	eReverb.setEnabled(true);
	eReverb.setDecayHFRatio((short) 1000);
	eReverb.setDecayTime(10000);
	eReverb.setDensity((short) 1000);
	eReverb.setDiffusion((short) 1000);
	eReverb.setReverbLevel((short) -1000);
	mediaPlayer.setAuxEffectSendLevel(1.0f);


###NDK

#####irrKlang
第三方音频库，优秀的有irrKlang，但是android用不了
因为irrKlang发布成已经编译的.so,而他是通过GNC gcc编译的。
android的c library是自己的BIONIC，需要通过android的c编译器编译的代码，才能在andorid上运行。所以irrKlang不能用在android上。

`引申：能在linux上跑的c程序，不一定能在android上面跑，因为c library的不同，就算有源码，通过android的编译工具编译，都不一定能在android上跑`


#####OpenSL es
这个library，已经内嵌在android系统中，其中还有openMax，两个的区别可以看ndk docs

OpenSL es google给出了官方例子在<ndk>/samples/native-audio中。但是例子有点难懂，特别是使用每个方法，对象都要

	slCreateEngine( &engine_obj, 0, nullptr, 0, nullptr, nullptr );
	(*engine_obj)->Realize( engine_obj, SL_BOOLEAN_FALSE );
	(*engine_obj)->GetInterface( engine_obj, SL_IID_ENGINE, &engine );

像这样操作，想想也是醉了！！！！！听说是COM风格的
这个原因可以参考[http://vec3.ca/getting-started-with-opensl-on-android/](http://vec3.ca/getting-started-with-opensl-on-android/)

openSL es部分待续


###高级课题

数字化的音频是离散的周期函数，可以用DFT(离散傅里叶转换)来分析。raw的音频是基于时间的数据，可以通过FFT（快速傅里叶转换）来变成基于频率的数据。www.netlib.org/fftpack/这个网站提供了各种FFT实现，包括java的方法

####to be continued...
