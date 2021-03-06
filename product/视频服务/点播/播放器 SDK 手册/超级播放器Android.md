## 简介

Android 超级播放器 SDK 是腾讯云开源的一款播放器组件，简单几行代码即可拥有类似腾讯视频强大的播放功能。包括横竖屏切换、清晰度选择、手势和小窗等基础功能，还支持视频缓存，软硬解切换和倍速播放等特殊功能。相比系统播放器，支持格式更多，兼容性更好，功能更强大。同时还具备首屏秒开、低延迟的优点，以及视频缩略图等高级能力。

## SDK 下载

点播 Android 超级播放器的下载地址是 [SuperPlayer_Android](https://github.com/tencentyun/SuperPlayer_Android)。

## 阅读对象

本文档部分内容为腾讯云专属能力，使用前请开通 [腾讯云](https://cloud.tencent.com) 相关服务，未注册用户可注册账号 [免费试用](https://cloud.tencent.com/login)。

## 快速集成

### aar 集成

1. 下载 SDK + Demo 开发包，下载地址为 [Android](https://github.com/tencentyun/SuperPlayer_Android)。
2. 导入 `SDK/LiteAVSDK_XXX.aar` 以及  `Demo/app/libs/lib_tcsuperplayer.aar`到工程中去。
3. 在 `app/build.gralde` 中添加依赖：
```java
compile(name: 'LiteAVSDK_Professional', ext: 'aar')
compile(name: 'lib_tcsuperplayer', ext: 'aar')
// 超级播放器弹幕集成的第三方库
compile 'com.github.ctiao:DanmakuFlameMaster:0.5.3'
```
4. 在项目`build.gralde`中添加：
```
...
allprojects {
    repositories {
        flatDir {
            dirs 'libs'
        }
        ...
    }
}
...
```
5. 权限声明
```java
<!--网络权限-->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<!--点播播放器悬浮窗权限-->
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
<!--存储-->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

>! `lib_tcsuperplayer.aar` 以 moudle 方式开源，您可在 Demo/lib_tcsuperplayer 中找到所有源代码。

### 使用播放器

播放器主类为`SuperPlayerView`，创建后即可播放视频。
```java
mSuperPlayerView = findViewById(R.id.main_super_player_view);
//通过fileid方式的视频信息配置
SuperPlayerModel model = new SuperPlayerModel();
model.appid = 1252463788;   //AppId
model.fileid = "5285890781763144364"; //视频 FileId
// 开始播放
mSuperPlayerView.playWithMode(model);
```

运行代码，可以看到视频在手机上播放，并且界面上大部分功能都处于可用状态。

![](https://main.qcloudimg.com/raw/128c45edfc77b319475868c21caec2de.png)

### 选择 FileId

视频 FileId 在一般是在视频上传后，由服务器返回：
1. 客户端视频发布后，服务器会返回 fileId 到客户端。
2. 服务端视频上传时，在 [确认上传](https://cloud.tencent.com/document/product/266/9757) 的通知中包含对应的 fileId。

如果文件已存在腾讯云，则可以进入 [视频管理](https://console.cloud.tencent.com/video/videolist) ，找到对应的文件，查看 fileId。如下图所示，ID 即表示 fileId：

![视频管理](https://main.qcloudimg.com/raw/15c5d181b9037b58db5cf192fe831f1b.png)

## 缩略图与打点

在播放长视频时，缩略图（雪碧图）和打点信息有助于观众找到该兴趣的点。使用腾讯云服务 API，能快速对视频处理。
- [截取雪碧图](https://cloud.tencent.com/document/product/266/8101)
- [增加打点信息](https://cloud.tencent.com/document/product/266/14190)

任务执行成功后，播放器的界面会增加新的元素。

![](https://main.qcloudimg.com/raw/55ebce6d0c703dafa1ac131e1852e025.png)

## 播放 DRM 加密的视频

数字版权管理解决方案（Digital Rights Management，DRM），通过技术手段加密内容，用来控制带版权作品的使用、更改和分发，保护带版权内容的安全。适用于音乐、电影等带版权的多媒体内容。

Android 播放器可以播放两种方式加密的输出：

1. 基于 Widevine 加密的 Dash 格式
2. 基于 SimpleAES 加密的 HLS 格式

关于 DRM 的更多详情，您可以参考 [如何对内容做版权保护](<https://cloud.tencent.com/document/product/266/34105#.E5.95.86.E4.B8.9A.E7.BA.A7-drm>) 方案文档。

### 使用方法

首先，App 需要从您的**业务后台**获取 Token，Token 的生成方式请参考 [这里](<https://cloud.tencent.com/document/product/266/34102#token-.E7.94.9F.E6.88.90>) 。然后，通过 FileId + Token 方式进行播放，播放代码如下：

```java
SuperPlayerModel model = new SuperPlayerModel();
String fileId = "15517827183920333646";
model.appId = 1253039488;
model.videoId = new SuperPlayerVideoId();
model.videoId.fileId = fileId;
model.videoId.playDefinition = "20";  // 播放模板
model.videoId.version = SuperPlayerVideoId.FILE_ID_V3; // DRM需要使用V3协议
try {
    // Token需要进行URLEncoder
    String encodedToken = URLEncoder.encode("业务下发的Token", "UTF-8");
    model.token = encodedToken;
} catch (UnsupportedEncodingException e) {
    e.printStackTrace();
}
mSuperPlayerView.playWithModel(model);
```

代码中的`playDefinition`是 [播放模板](https://cloud.tencent.com/document/product/266/34101#.E6.92.AD.E6.94.BE.E6.A8.A1.E6.9D.BF) ID，播放器会根据播放模板指定的行为播放。例如当模板 ID 为20时，先尝试播放商业级加密的输出，若无法播放再降级播放 SimpleAES 方式加密的输出。

## 小窗播放

小窗播放可以悬浮在所有 Activity 之上播放。使用小窗播放非常简单，只需要在开始播放前调用下面代码即可：
```java
// 播放器配置
SuperPlayerGlobalConfig prefs = SuperPlayerGlobalConfig.getInstance();
// 开启悬浮窗播放
prefs.enableFloatWindow = true;
//设置悬浮窗的初始位置和宽高
SuperPlayerGlobalConfig.TXRect rect = new SuperPlayerGlobalConfig.TXRect();
rect.x = 0;
rect.y = 0;
rect.width = 810;
rect.height = 540;
// ...其他配置
```
![](https://main.qcloudimg.com/raw/d6783a450e339526e0ca0b2ed3ef6142.png)

## 退出播放

当不需要播放器时，调用 `resetPlayer` 清理播放器内部状态，释放内存。
```java
mSuperPlayerView.resetPlayer();
```

## 更多功能

完整功能可扫码下载视频云工具包体验，或直接运行工程 Demo。
![Android二维码下载](https://main.qcloudimg.com/raw/344d9d41fc5e305a17e22e104b9305da.png)
