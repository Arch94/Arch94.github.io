---
title: ionic webview in native(android)
date: 2018-11-25 14:05:43
categories:
- ionic
tags:
- webview
---
其实开发了这么久，webview的优点很多，热更新、跨平台开发、易维护。但是局限性也很大，一些原生的功能还是用原生插件最好。webview webview，功能只是web 用来 view 的。把网页当作native 里的一个模块或插件最好。好久没更新blog了，更新一波让自己记住。
把ionic 部署到android原生应用里面
```
MainActivity
public class MainActivity extends AppCompatActivity {
    private  View cv;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        cv = getWindow().getDecorView();
        findViewById(R.id.ionic).setOnClickListener(cordovaViewClickListener);
    }

    private View.OnClickListener cordovaViewClickListener=new View.OnClickListener() {
        @Override
        public void onClick(View v){
            startCordovaActivity(cv);
        }
    };

    public void startCordovaActivity(View view) {
        // 项目用的方式
        //Intent intent = new Intent(this, TestCordovaActivity.class);
        //ionic 现在应用的方式
        Intent intent = new Intent(this, IonicActivity.class);
        startActivity(intent);
    }
}
```
看看ionic 如何嵌套webview的，activity无需xml
```
public class IonicActivity extends CordovaActivity {

    @Override
    public void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);

        // enable Cordova apps to be started in the background
        Bundle extras = getIntent().getExtras();
        if (extras != null && extras.getBoolean("cdvStartInBackground", false)) {
            moveTaskToBack(true);
        }

        // Set by <content src="index.html" /> in config.xml
        loadUrl("file:///android_asset/ionic/index.html");
    }

}
```
我们项目组嵌套native的方式，效果上和是一样的
```
public class TestCordovaActivity extends Activity {

    SystemWebView webView;
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.content_frame);
        initCordovaWebView();
    }

    public CordovaInterfaceImpl cordovaInterface;
    private CordovaWebView cordovaWebView;
    public final ArrayBlockingQueue<String> onPageFinishedUrl = new ArrayBlockingQueue<String>(5);
    public void initCordovaWebView() {
        cordovaInterface = new CordovaInterfaceImpl(this) {
            @Override
            public Object onMessage(String id, Object data) {
                if ("onPageFinished".equals(id)) {
                    onPageFinishedUrl.add((String) data);
                }
                return super.onMessage(id, data);
            }
        };
        ConfigXmlParser parser = new ConfigXmlParser();
        parser.parse(this);
        webView = (SystemWebView) findViewById(R.id.aiCordovaWebView);
        SystemWebViewEngine systemWebViewEngine = new SystemWebViewEngine(webView);
        //允许JavaScript执行
        webView.getSettings().setJavaScriptEnabled(true);
        this.cordovaWebView = new CordovaWebViewImpl(systemWebViewEngine);
        this.cordovaWebView.init(this.cordovaInterface, parser.getPluginEntries(), parser.getPreferences());
        this.cordovaWebView.loadUrl("file:///android_asset/ionic/index.html");

    }


    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent intent) {
        super.onActivityResult(requestCode, resultCode, intent);
        this.cordovaInterface.onActivityResult(requestCode, resultCode, intent);
    }

}
```
对应的xml
```
<org.apache.cordova.engine.SystemWebView
        android:id="@+id/aiCordovaWebView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
```
上面就是主要代码，至于其他的步骤，网上其他blog 都有，现在有一个native的应用，还有一个ionic build 出来的android 项目
1.  把www文件夹放在android assets 资源目录下，我改名叫ionic 文件夹了
2.  src/main/res/xml 里面的config.xml 文件要从ionic build的android项目里挪过来，这个文件很重要，里面有各种cordova插件所用的java依赖配置,使用command或者ctrl 点击某一项看能不能跳到java文件，不然找不到项目是会报错的。 
3.  AndroidManifest.xml 里面还要加各种各样的权限，同理挪过来，其中有几个注意的地方
```
<!-- android网络请求权限 -->
    <uses-permission android:name="android.permission.INTERNET" />
 <!-- 这个特别重要！！！！！！不然可能相机不能用 -->
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="${applicationId}.provider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/provider_paths" />
        </provider>
```
还有就是 cordova 的sqllite plugin，如果用到了注意把它的2个jar包放进项目依赖里面，sqlite-connector.jar、sqlite-native-driver.jar 别漏了，不然可能会遇到奇怪的报错，当时还是在 stackflow 才知道什么情况
4. 最后一点别忘记，把cordova的jar包以及cordova插件的jar包或者模块拷过来。
5. 直接用android studio 跑起来看一下吧
