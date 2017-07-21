
# 视频常见知识点

参考

[视音频编解码技术零基础学习方法](http://blog.csdn.net/leixiaohua1020/article/details/18893769)

[Android视频开发进阶(part1-关于视频的那些术语)](http://www.jianshu.com/p/10e357946447)

---

## 流媒体协议

    流媒体协议是服务器与客户端之间通信遵循的规定
    RTSP+RTP/RTMP/RTMFP/MMS/HTTP

## 视频编码/解码
    
    将视频像素数据（RGB，YUV等）压缩成为视频码流，从而降低视频的数据量
    HEVC(H.265)/H.264/MPEG4/MPEG2/..

## 视频格式（封装）

    把视频码流和音频码流按照一定的格式存储在一个文件中
    AVI/MP4/TS/FLV/MKV/RMVB

## 视频播放器流程

> 视频播放器步骤：解协议，解封装，解码视音频，视音频同步

![](http://img.blog.csdn.net/20140201120523046?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVpeGlhb2h1YTEwMjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


# SRS搭建

    
    直播服务器使用SRS(运营级的互联网直播服务器集群)
    使用百度LLS推流SDK和拉流SDK进行测试srs
    

> 注意：SRS主要运行在Linux系统上，譬如Centos和Ubuntu，包括x86、x86-64、ARM和MIPS。MacOS支持代码编辑和编译。其他Unix-like系统不支持，SRS也不支持Windows系统。

官网：https://github.com/ossrs/srs

## 安装步骤

**step1. 获取srs**
    
    git clone https://github.com/ossrs/srs
    cd srs/trunk
    
**step2. 编译srs**    
    
    ./configure && make

**step3. 编写SRS配置文件**

    gedit conf/rtmp.conf
    
    ##############
    # conf/rtmp.conf
    listen              1935;
    max_connections     1000;
    vhost __defaultVhost__ {
    }
    ##############

（此处我没有改动，具体配置更改可自行百度/Google）

**step4. 启动SRS**
    
    cd srs/trunk
    ./etc/init.d/srs start 

## 使用推流SDK/拉流SDK测试srs

    使用百度LSS推流SDK和拉流SDK
    更改推流拉流地址 rtmp://(your ip)/...（参考SRS官网）
    即可
    
    注意：如果使用的是虚拟机搭建，责IP需要独立于本机，可使用虚拟机桥接模式

## extra:虚拟机桥接模式

[VMware设置桥接上网](http://www.jb51.net/article/103768.htm)

    步骤按照网上参考来
    注意点
    1. 防火墙关闭
    2. /etc/init.d/networking restart 重启网关不一定成功，直接重启系统
    3. sudo gedit /etc/network/interfaces 可以更IP配置，可以在UI界面里配置（设置-网络-有线-选项-IPV4设置） 
    4. sudo gedit /etc/resolv.conf 配置DNS 同样可以在UI界面里配置（设置-网络-有线-选项-IPV4设置） 
    5. 每次重启系统 ，dns要重新设置（最新的VMware会自动保存上次配置，所以不需要）
    6. 查看ifconfig 的inet addr是不是当前配置的ip
    7. 使用p2p终结者查看当前网络那个IP没有使用
    8. ping 主机 主机ping虚拟IP ping测试网络

以上步骤完成后，就可以用推流SDK进行直播，拉流SDK进行播放（开森）

# 使用FFmpeg进行视音频编解码

官网：https://ffmpeg.org/

[FFMPEG视音频编解码零基础学习方法](http://blog.csdn.net/leixiaohua1020/article/details/15811977)

---

> 多媒体视频处理工具FFmpeg有非常强大的功能包括视频采集功能、视频格式转换、视频抓图、给视频加水印等

## As编译调用FFmpeg库(Cmake)

### ndk安装

**step1. 下载并解压**

    下载所需对应的系统NDK包
    解压到home/xxx
    
**step2. 配置环境变量**

    $sudo gedit ~/.bashrc
    添加代码
    export   NDK=/home/xxx/(ndk解压的路径文件名)
    export   PATH=${PATH}:$NDK 
    source  ~/.bashrc使其修改的文件生效
    ndk-build 查看是否安装成功（无command not found即可）

### 编译ffmpeg的so库 

**step1. 下载并且解压**

参考：
https://github.com/dxjia/ffmpeg-compile-shared-library-for-android

下载并且解压最新的linux包ffmpeg-3.3.2.tar.bz2

**step2. 修改ffmpeg/configure**
    
    SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
    LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
    SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
    SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR)$(SLIBNAME)'
    
    改为
    SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
    LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
    SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
    SLIB_INSTALL_LINKS='$(SLIBNAME)'

这样编译出来的so库的名字才能够符合android所需

**step3. 建立build_android_arm.sh**

进入解压的文件目录，建议build_android_arm.sh（编译arm平台库sh文件）


    #!/bin/bash
    NDK_HOME=/home/xxx/android-ndk-r14b
    #echo It is ok
    SYSROOT=$NDK_HOME/platforms/android-14/arch-arm
    TOOLCHAIN=$NDK_HOME/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64
    
    export NDK
    export SYSROOT
    export TOOLCHAIN
    
    CPU=arm
    ADDI_CFLAGS="-marm"
    PREFIX=$(pwd)/android/$CPU
    
    
    function build_one
    {
    ./configure \
    --prefix=$PREFIX \
    --enable-shared \
    --disable-static \
    --disable-doc \
    --disable-ffserver \
    --enable-cross-compile \
    --cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
    --target-os=linux \
    --arch=arm \
    --sysroot=$SYSROOT \
    --extra-cflags="-Os -fpic $ADDI_CFLAGS" \
    --extra-ldflags="$ADDI_LDFLAGS" \
    $ADDITIONAL_CONFIGURE_FLAG
    make clean  
    make  
    make install 
    }
    
    build_one


NDK=/home/xxx/android/android-ndk-r10 更改为自己安装的ndk位置

SYSROOT=$NDK/platforms/android-16/arch-arm/指定的ndk platform的路径，一定要选择比你的目标机器使用的版本低的，比如你的手机是android-15版本，那么就选择低于15的

TOOLCHAIN=/home/xxx/android/android-ndk-r10/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64 更改为自己的toolchains对应的arm-linux

**step4. 执行build_android_arm.sh**
    
    ./build_andrioid_arm.sh

解压的文件目录内生成android文件夹，查看android/arm/lib目录下的so文件库，成功编译成SO库
    
### As调用ffmpeg

参考：[编译Android下可执行命令的FFmpeg](http://blog.csdn.net/mabeijianxi/article/details/72904694)



**step1. new project**

    创建新项目，记得勾选“include C++ project”
    会自动生成CmakeList.txt,cpp目录下对应的native-lib等

**step2. build.gradle**

    externalNativeBuild {
        cmake {
            cppFlags "-std=c++11"//包含c++11特性
        }
        ndk {
            abiFilters "armeabi"//android平台
        }
    }
    
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
            //可放到cpp目录下则改为src/main/cpp/CMakeLists.txt）
        }
    }
    
**step2. 导入so以及include文件内容**

so放置到src/main/jniLibs/armeabi目录下，也可以随意放置目录，不过build.gradle需要进行配置

    sourceSets {
        main {
            jniLibs.srcDirs = ['./libs/jni']
        }
    }

include（ffmpeg编译build_android_arm.sh时候生成的，位于android/arm/include） 放置到src/main/cpp目录下，也可以随意放置，但是需要在cmakelists.txt中配置

**step3. CmakeLists.txt配置**

    # For more information about using CMake with Android Studio, read the
    # documentation: https://d.android.com/studio/projects/add-native-code.html
    
    # Sets the minimum version of CMake required to build the native library.
    
    cmake_minimum_required(VERSION 3.4.1)
    
    # 设置支持gnu++11
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=gnu++11")
    
    # 设置so包位置
    set(lib_src_DIR ${CMAKE_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI})
    
    
    #include位置
    include_directories(
            ${CMAKE_SOURCE_DIR}/src/main/cpp/include
    )
    
    # add so
    add_library(avcodec-57_lib SHARED IMPORTED)
    set_target_properties(avcodec-57_lib PROPERTIES IMPORTED_LOCATION
                                 ${lib_src_DIR}/libavcodec-57.so)
    
    
    add_library(avdevice-57_lib SHARED IMPORTED)
    set_target_properties(avdevice-57_lib PROPERTIES IMPORTED_LOCATION
                            ${lib_src_DIR}/libavdevice-57.so)
    
    
    add_library(avformat-57_lib SHARED IMPORTED)
    set_target_properties(avformat-57_lib PROPERTIES IMPORTED_LOCATION
                            ${lib_src_DIR}/libavformat-57.so)
    
    add_library(avutil-55_lib SHARED IMPORTED)
    set_target_properties(avutil-55_lib PROPERTIES IMPORTED_LOCATION
                            ${lib_src_DIR}/libavutil-55.so)
    
    add_library(swresample-2_lib SHARED IMPORTED)
    set_target_properties(swresample-2_lib PROPERTIES IMPORTED_LOCATION
                            ${lib_src_DIR}/libswresample-2.so)
    
    
    add_library(avfilter-6_lib SHARED IMPORTED)
    set_target_properties(avfilter-6_lib PROPERTIES IMPORTED_LOCATION
                            ${lib_src_DIR}/libavfilter-6.so)
    
    
    add_library(swscale-4_lib SHARED IMPORTED)
    set_target_properties(swscale-4_lib PROPERTIES IMPORTED_LOCATION
                            ${lib_src_DIR}/libswscale-4.so)
    
    
    
    
    # Creates and names a library, sets it as either STATIC
    # or SHARED, and provides the relative paths to its source code.
    # You can define multiple libraries, and CMake builds them for you.
    # Gradle automatically packages shared libraries with your APK.
    
    add_library( # Sets the name of the library.
                 native-lib
    
                 # Sets the library as a shared library.
                 SHARED
    
                 # Provides a relative path to your source file(s).
                 src/main/cpp/native-lib.cpp)
    
    # Searches for a specified prebuilt library and stores the path as a
    # variable. Because CMake includes system libraries in the search path by
    # default, you only need to specify the name of the public NDK library
    # you want to add. CMake verifies that the library exists before
    # completing its build.
    find_library(log-lib
                  log )
    
    # Specifies libraries CMake should link to your target library. You
    # can link multiple libraries, such as libraries you define in this
    # build script, prebuilt third-party libraries, or system libraries.
    
    target_link_libraries( # Specifies the target library.
                           native-lib
                           # Links the target library to the log library
                           # included in the NDK.
                           avcodec-57_lib
                           avdevice-57_lib
                           avfilter-6_lib
                           avformat-57_lib
                           avutil-55_lib
                           swresample-2_lib
                           swscale-4_lib
                           ${log-lib})

注意：CMAKE_SOURCE_DIR 代表最顶层CMakeLists.txt所在的文件夹的路径
https://landchanning.github.io/2016/11/10/android_touch_ndk_cmake/

**step4. loadLibrary**

    System.loadLibrary("native-lib");

由于测试机子android版本4.2.2，出现

> Cannot load library: soinfo_link_image(linker.cpp:1635): could not load library "libavcodec-57.so" needed by "libnative-lib.so"; caused by load_library(linker.cpp:745): library "libavcodec-57.so" not

参考： https://stackoverflow.com/questions/28806373/android-4-2-ndk-library-loading-crash-load-librarylinker-cpp750-soinfo-l

改为

    System.loadLibrary("avutil-55");
    System.loadLibrary("swresample-2");
    System.loadLibrary("avcodec-57");
    System.loadLibrary("avformat-57");
    System.loadLibrary("swscale-4");
    System.loadLibrary("avfilter-6");
    System.loadLibrary("avdevice-57");
    System.loadLibrary("native-lib");
    
**step5. native**

    //添加所需的native方法
    public native int stream(String param1,String param2);
    
可用javah 生成native方法对应的h文件

    /* DO NOT EDIT THIS FILE - it is machine generated */
    #include <jni.h>
    /* Header for class com_lseven_test_testffmpegplayer_MainActivity */
    
    #ifndef _Included_com_lseven_test_testffmpegplayer_MainActivity
    #define _Included_com_lseven_test_testffmpegplayer_MainActivity
    #ifdef __cplusplus
    extern "C" {
    #endif
    /*
     * Class:     com_test_testffmpegplayer_MainActivity
     * Method:    stringFromJNI
     * Signature: ()Ljava/lang/String;
     */
    JNIEXPORT jstring JNICALL Java_com_test_testffmpegplayer_MainActivity_stringFromJNI
      (JNIEnv *, jobject);
    
    /*
     * Class:     com_lseven_test_testffmpegplayer_MainActivity
     * Method:    stream
     * Signature: (Ljava/lang/String;Ljava/lang/String;)I
     */
    JNIEXPORT jint JNICALL Java_com_test_testffmpegplayer_MainActivity_stream
      (JNIEnv *, jobject, jstring, jstring);
    
    #ifdef __cplusplus
    }
    #endif
    #endif

则native-lib.cpp添加

    JNIEXPORT jint JNICALL Java_com_test_testffmpegplayer_MainActivity_stream
    (JNIEnv *env, jobject, jstring param1, jstring param2){
      //TODO 
    }

测试运行即可，android studio版本支持断点调试jni

# Ffmpeg 推流
参考：

[最简单的基于FFmpeg的移动端例子：Android 推流器](http://blog.csdn.net/leixiaohua1020/article/details/47056051)

[ffmpeg超详细综合教程——摄像头直播](http://blog.csdn.net/xyblog/article/details/50433199)

```c
AVOutputFormat *ofmt = NULL;  
AVFormatContext *ifmt_ctx = NULL, *ofmt_ctx = NULL;  
AVPacket pkt;  

int ret, i;  
char input_str[500]={0};  
char output_str[500]={0};  
char info[1000]={0};  
sprintf(input_str,"%s",(*env)->GetStringUTFChars(env,input_jstr, NULL));  
sprintf(output_str,"%s",(*env)->GetStringUTFChars(env,output_jstr, NULL));  

//input_str  = "cuc_ieschool.flv";  
//output_str = "rtmp://localhost/publishlive/livestream";  
//output_str = "rtp://233.233.233.233:6666";  

//FFmpeg av_log() callback  
av_log_set_callback(custom_log);  

av_register_all();  
//Network  
avformat_network_init();  

//Input  
if ((ret = avformat_open_input(&ifmt_ctx, input_str, 0, 0)) < 0) {  
LOGE( "Could not open input file.");  
goto end;  
}  
if ((ret = avformat_find_stream_info(ifmt_ctx, 0)) < 0) {  
LOGE( "Failed to retrieve input stream information");  
goto end;  
}  

int videoindex=-1;  
for(i=0; i<ifmt_ctx->nb_streams; i++)   
if(ifmt_ctx->streams[i]->codec->codec_type==AVMEDIA_TYPE_VIDEO){  
    videoindex=i;  
    break;  
}  
//Output  
avformat_alloc_output_context2(&ofmt_ctx, NULL, "flv",output_str); //RTMP  
//avformat_alloc_output_context2(&ofmt_ctx, NULL, "mpegts", output_str);//UDP  

if (!ofmt_ctx) {  
LOGE( "Could not create output context\n");  
ret = AVERROR_UNKNOWN;  
goto end;  
}  
ofmt = ofmt_ctx->oformat;  
for (i = 0; i < ifmt_ctx->nb_streams; i++) {  
//Create output AVStream according to input AVStream  
AVStream *in_stream = ifmt_ctx->streams[i];  
AVStream *out_stream = avformat_new_stream(ofmt_ctx, in_stream->codec->codec);  
if (!out_stream) {  
    LOGE( "Failed allocating output stream\n");  
    ret = AVERROR_UNKNOWN;  
    goto end;  
}  
//Copy the settings of AVCodecContext  
ret = avcodec_copy_context(out_stream->codec, in_stream->codec);  
if (ret < 0) {  
    LOGE( "Failed to copy context from input to output stream codec context\n");  
    goto end;  
}  
out_stream->codec->codec_tag = 0;  
if (ofmt_ctx->oformat->flags & AVFMT_GLOBALHEADER)  
    out_stream->codec->flags |= CODEC_FLAG_GLOBAL_HEADER;  
}  

//Open output URL  
if (!(ofmt->flags & AVFMT_NOFILE)) {  
ret = avio_open(&ofmt_ctx->pb, output_str, AVIO_FLAG_WRITE);  
if (ret < 0) {  
    LOGE( "Could not open output URL '%s'", output_str);  
    goto end;  
}  
}  
//Write file header  
ret = avformat_write_header(ofmt_ctx, NULL);  
if (ret < 0) {  
LOGE( "Error occurred when opening output URL\n");  
goto end;  
}  

int frame_index=0;  

int64_t start_time=av_gettime();  
while (1) {  
AVStream *in_stream, *out_stream;  
//Get an AVPacket  
ret = av_read_frame(ifmt_ctx, &pkt);  
if (ret < 0)  
    break;  
//FIX：No PTS (Example: Raw H.264)  
//Simple Write PTS  
if(pkt.pts==AV_NOPTS_VALUE){  
    //Write PTS  
    AVRational time_base1=ifmt_ctx->streams[videoindex]->time_base;  
    //Duration between 2 frames (us)  
    int64_t calc_duration=(double)AV_TIME_BASE/av_q2d(ifmt_ctx->streams[videoindex]->r_frame_rate);  
    //Parameters  
    pkt.pts=(double)(frame_index*calc_duration)/(double)(av_q2d(time_base1)*AV_TIME_BASE);  
    pkt.dts=pkt.pts;  
    pkt.duration=(double)calc_duration/(double)(av_q2d(time_base1)*AV_TIME_BASE);  
}  
//Important:Delay  
if(pkt.stream_index==videoindex){  
    AVRational time_base=ifmt_ctx->streams[videoindex]->time_base;  
    AVRational time_base_q={1,AV_TIME_BASE};  
    int64_t pts_time = av_rescale_q(pkt.dts, time_base, time_base_q);  
    int64_t now_time = av_gettime() - start_time;  
    if (pts_time > now_time)  
        av_usleep(pts_time - now_time);  

}  

in_stream  = ifmt_ctx->streams[pkt.stream_index];  
out_stream = ofmt_ctx->streams[pkt.stream_index];  
/* copy packet */  
//Convert PTS/DTS  
pkt.pts = av_rescale_q_rnd(pkt.pts, in_stream->time_base, out_stream->time_base, AV_ROUND_NEAR_INF|AV_ROUND_PASS_MINMAX);  
pkt.dts = av_rescale_q_rnd(pkt.dts, in_stream->time_base, out_stream->time_base, AV_ROUND_NEAR_INF|AV_ROUND_PASS_MINMAX);  
pkt.duration = av_rescale_q(pkt.duration, in_stream->time_base, out_stream->time_base);  
pkt.pos = -1;  
//Print to Screen  
if(pkt.stream_index==videoindex){  
    LOGE("Send %8d video frames to output URL\n",frame_index);  
    frame_index++;  
}  
//ret = av_write_frame(ofmt_ctx, &pkt);  
ret = av_interleaved_write_frame(ofmt_ctx, &pkt);  

if (ret < 0) {  
    LOGE( "Error muxing packet\n");  
    break;  
}  
av_free_packet(&pkt);  
  
}  
//Write file trailer  
av_write_trailer(ofmt_ctx);  
end:  
avformat_close_input(&ifmt_ctx);  
/* close output */  
if (ofmt_ctx && !(ofmt->flags & AVFMT_NOFILE))  
avio_close(ofmt_ctx->pb);  
avformat_free_context(ofmt_ctx);  
if (ret < 0 && ret != AVERROR_EOF) {  
LOGE( "Error occurred.\n");  
return -1;  
}  
return 0;  

```

## Ffmpeg常见结构体和函数

参考： [FFmpeg常见结构体](http://blog.csdn.net/leixiaohua1020/article/details/41181155)

### 常见结构体

AVFormatContext：统领全局的基本结构体。主要用于处理封装格式（FLV/MKV/RMVB等）。

AVIOContext：输入输出对应的结构体，用于输入输出（读写文件，RTMP协议等）

AVStream，AVCodecContext：视音频流对应的结构体，用于视音频编解码。

AVFrame：存储非压缩的数据（视频对应RGB/YUV像素数据，音频对应PCM采样数据）

AVPacket：存储压缩数据（视频对应H.264等码流数据，音频对应AAC/MP3等码流数据）


---

**解协议（http,rtsp,rtmp,mms—）**

AVIOContext，URLProtocol，URLContext主要存储视音频使用的协议的类型以及状态。URLProtocol存储输入视音频使用的封装格式。每种协议都对应一个URLProtocol结构。（注意：FFMPEG中文件也被当做一种协议“file”）

**解封装（flv,avi,rmvb,mp4）**

AVFormatContext主要存储视音频封装格式中包含的信息；AVInputFormat存储输入视音频使用的封装格式。每种视音频封装格式都对应一个AVInputFormat 结构

**解码（h264,mpeg2,aac,mp3）**

每个AVStream存储一个视频/音频流的相关数据；每个AVStream对应一个AVCodecContext，存储该视频/音频流使用解码方式的相关数据；每个AVCodecContext中对应一个AVCodec，包含该视频/音频对应的解码器。每种解码器都对应一个AVCodec结构


**存数据**

视频的话，每个结构一般是存一帧；音频可能有好几帧。
解码前数据：AVPacket
解码后数据：AVFrame

---

**注册/销毁**

    AVFormatContext: avformat_alloc_context/avformat_free_context
    AVIOContext: avio_alloc_context/av_free
    AVStream：avformat_new_stream/AVFormatContext.avformat_free_context
    AVFrame: av_frame_alloc/av_frame_free
    AVPacket: av_init_packet，av_new_packet/av_free_packet
    
---

**其余结构体**

**URLProtocol/URLContext**

**URLProtocol**

    libavformat\url.h
    url_open()：打开协议
    url_read()：读数据
    url_write()：写数据
    url_close()：关闭协议

FILE协议（“文件”在FFmpeg中也被当做一种协议）的结构体ff_file_protocol的定义位于libavformat\file.c

LibRTMP协议的结构体ff_librtmp_protocol的定义位于libavformat\librtmp.c

UDP协议的结构体ff_udp_protocol的定义位于libavformat\udp.c


### 常见函数

    ...

## Android Camera视频采集

参考：[Android中直播视频技术探究之---摄像头Camera视频源数据采集解析](http://blog.csdn.net/jiangwei0910410003/article/details/52057543)

[android平台，利用ffmpeg对android摄像头采集编码](http://blog.csdn.net/zh_ang_hua/article/details/47276357)


[安卓手机摄像头编码](http://blog.csdn.net/xyblog/article/details/50433183)

1. camera数据收集
2. 如何让ffmpeg对camera数据格式转换（转为flv h26.4）
3. 如何上传(camera帧数据上传到rtmp服务器?)
4. 结束上传
5. extra:进度条进度控制/暂停退出继续播放/水印美颜效果（考虑如何使用ffmpeg配合使用）

