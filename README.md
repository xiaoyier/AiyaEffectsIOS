# AiyaCameraSDK 说明文档

## 1. SDK 功能说明
哎吖相机SDK的主要功能是在图片或视频中增加特效，可用于相机、直播等多种情景。

## 2. SDK 架构设计
架构设计分为三层.
1. 最外层的是功能模块.功能模块可以快速的方便的集成到项目中.
2. 中间框架层是以AiyaGPUImage为前缀,以Filter为后缀的类组成.这些类运行在GPUImage这个框架中.如果对GPUImage框架熟悉的开发人员,可以自定义丰富的功能.
3. 底层模块是以Effect为后缀的类组成.这些类运行在OpenGl环境中.如果对OpenGL熟悉的开发人员,可以直接应用到图形渲染环境中.

以下是类图
![image](doc/1.png)

## 3. SDK API说明
######AiyaCamera: 相机组件.
    通过设置启动相机时的参数以及设置美颜和滤镜参数,打开相机,对相机输出的数据进行美颜和滤镜处理.使用前要验证License.

######AiyaEffectProcess: 图像数据处理组件(特效,美颜).
    设置美颜和滤镜参数,对图像数据进行美颜和滤镜处理.使用前要验证License.

######AiyaYFBeautifyProcess: 图像数据处理组件(美颜).
    设置美颜参数,对图像数据进行美颜和滤镜处理.使用前不用验证License.

######AiyaGPUImagexxxFilter: 滤镜.
    特效滤镜,美颜滤镜.

######AYxxx: GPUImageSDK中的核心类.
    都以AY为前缀.逻辑无修改.

######AiyaLicenseManager: 验证License.
    在启动时调用.

######AiyaCameraEffect:
    BGRA数据的人脸识别,对纹理数据进行美颜和特效处理.

######AiyaBeautifyEffect:
    对纹理数据进行美颜处理.

## 4. SDK 集成说明
#####1. 导入SDK静态库文件AiyaCameraSDK.framework
#####2. 导入SDK资源文件 AYEffectTrackerData
#####3. 导入依赖的系统库 libz.tbd, libc++.tbd, libc++.1.tdb, CoreMedia.framework, CoreGraphics.framework, MobileCoreServices.framework
#####4. 初始化Lisence.在使用AiyaCameraSDK之前,必须先初始化license,否则会出现无法使用的情况.
```objective-c
[AiyaLicenseManager initLicense:@"对应的licenseKey"];

```
#####5. 具体使用方式如下
 * 使用方式一: 用AiyaCamera对相机数据进行特效加美颜处理
```objective-c
_camera = [[AiyaCamera alloc]initWithPreview:self.view cameraPosition:AVCaptureDevicePositionFront];//设置为前置相机
[self.camera setSessionPreset:AVCaptureSessionPreset1280x720];
[self.camera setEffectPath:path];//设置特效路径
[self.camera setEffectPlayCount:0];//设置特效播放次数.0表示一直重复
[self.camera setBeautyLevel:AIYA_BEAUTY_LEVEL_5];//设置美颜等级
[self.camera setHasAudioTrack:YES];//打开音频数据采集
self.camera.delegate = self;
[self.camera startCapture];//打开相机

#pragma mark AiyaCameraDelegate
/**
 特效渲染完成后的数据回调
 */
- (void)videoCaptureOutput:(AiyaCamera *)capture pixelBuffer:(CVPixelBufferRef)pixelBuffer frameTime:(CMTime)frameTime effectStatus:(AIYA_CAMERA_EFFECT_STATUS)effectStatus{

}

/**
 音频数据的回调
 */
- (void)audioCaptureOutput:(AiyaCamera *)capture audioBuffer:(CMSampleBufferRef)audioBuffer{

}
```
 * 使用方式二: 用AiyaYFBeautifyProcess对像素缓冲区数据(CVPixelBuffer)进行美颜处理
```objective-c
_aiyaYFBeautifyProcess = [[AiyaYFBeautifyProcess alloc]init];//只初始化一次
_aiyaYFBeautifyProcess.beautyLevel = AIYA_YF_BEAUTY_LEVEL_5;

//---对像素缓冲区数据进行美颜处理---
[_aiyaYFBeautifyProcess processWithPixelBuffer:pixelBuffer];
//---美颜处理结束---
```
 * 使用方式三: 用AiyaEffectProcess对像素缓冲区数据(CVPixelBuffer)进行特效加美颜处理
```objective-c
_aiyaEffectProcess = [[AiyaEffectProcess alloc]init];//只初始化一次
_aiyaEffectProcess.beautyLevel = AIYA_BEAUTY_LEVEL_5;
_aiyaEffectProcess.effectPath = path;//设置特效路径
_aiyaEffectProcess.effectPlayCount = 1;//特效只播放一次

//---对像素缓冲区数据进行特效和美颜处理---
[_aiyaYFBeautifyProcess processWithPixelBuffer:pixelBuffer];
//---特效和美颜处理结束---
```

## 5. 常见问题
问题一: 如果项目中已经集成了GPUImageSDK怎么办?会不会产生冲突
答: AiyaCameraSDK中封装的GPUImageSDK,全部类,c函数,宏都做了加前缀的处理.集成到项目中时不会和原有的GPUImageSDK产生冲突.

问题二: 特效只有一分钟.
答: 只有传入正式的License才不会有一分钟的限制.

问题三: SDKDemo中有Podfile文件,但是我的电脑没有装pod怎么办?
答: SDKDemo已经把第三方库打包进去了.不需要下载更新第三方库.双击AiyaCameraSDKDemo.xcworkspace文件,打开后即可运行SDKDemo.

问题四: 模拟器上无法运行.一直报错找不到文件怎么办?
答: 目前SDK没有打包x86版本,无法在模拟器上运行.只能在真机上运行.

问题五: 没有网络时License会失效吗?
答: 用户第一次验证License必须要有网络.以后再次验证时没有网络也可以正常使用.

## 6. 版本更新
最新版本 V1.4

## 7. License说明
1. 每次启动时都要进行验证.
2. 首次验证必须要进行联网.
3. License 验证是同步请求.
4. AiyaLicenseManager的initLicense函数返回码说明:
 YES 认证成功, SDK可以正常使用
 NO  认证失败, SDK不可以使用,请通过下面的联系方式联系我们

## 8. 联系方式
邮箱: liudawei@aiyaapp.com