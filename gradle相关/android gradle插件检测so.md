# 检测so
 其根源就是，拿到`implementation`引用的依赖，然后解压依赖文件，拿到依赖文件中的so文件，判断大小及是否适配32和64位  32指得是armeabi  64指的是arm64-v8a
# 检测Manifest
 其根源就是，拿到`implementation`引用的依赖，然后解压依赖文件，拿到依赖文件中的Manifest文件，然后解析xml 拿到权限列表

# 检测res
其根源是，拿到`project.getRootDir().getAbsolutePath() + "/app/src/main/res/` 拿到res中的文件，拿到png图片，判断其大小，然后大于100k的直接进行tinify 进行压缩，然后拿到所有的xml，拿到内容删除空格，然后拿到MD5
 判断md5 是否相同，如果相同则是重复文件

 # 插件功能

 * 检测三方库是否有so，so大小及so是否适配32和64
 * 检测三方库含有权限，是否含有我们不使用的权限
 * 检测Res资源文件 （1） 自动压缩大于100k的图片，（2） 检测是否有重复资源（比如多个shape.xml文件名字不同，但是内容相同）
  
 # 插件使用
 
## 引入插件
  ```java
  apply plugin:'com.match.maker.plugin'
  ```

  ## （1） 检测三方库是否有so，so大小及so是否适配32和64

 命令行执行命令 ./gradlew checkso
 ```java

 ./gradlew checkso

...

[mmm] ---------------[依赖产物开始] group=com.immomo.momomedia:x264encoder----------------
[mmm] so文件 = jni/arm64-v8a/libx264encoder.so   size = 424KB
[mmm] so文件 = jni/armeabi/libx264encoder.so   size = 477KB
[mmm] ---------------[依赖产物结束] group=com.immomo.momomedia:x264encoder----------------
[mmm] ---------------[依赖产物开始] group=com.immomo.momomedia:voaac----------------
[mmm] so文件 = jni/arm64-v8a/libVoAACEncoder.so   size = 74KB
[mmm] so文件 = jni/armeabi/libVoAACEncoder.so   size = 78KB
[mmm] so文件 = jni/armeabi-v7a/libVoAACEncoder.so   size = 78KB
[mmm] ---------------[依赖产物结束] group=com.immomo.momomedia:voaac----------------

...

[mmm] v8a不包含 = libbsdiff.so   group=com.immomo.android.mklibrary:mk   size = 39KB
[mmm] v8a不包含 = libmkjni.so   group=com.immomo.android.mklibrary:mk   size = 7KB
[mmm] v8a不包含 = libsevenz.so   group=com.immomo.android.mklibrary:mk   size = 33KB
[mmm] v8a不包含 = libMOMOPitchShift.so   group=com.immomo.momomedia:mmaudio   size = 76KB
[mmm] v8a不包含 = libMOMOPitchShift.so   group=com.immomo.momomedia:mmaudio   size = 76KB
[mmm] v8a不包含 = libmjni.so   group=MatchMakerAndroid:momsecurity   size = 62KB

 ```

 ###  源码解析

```gradle
  project.task("checkso").doFirst {
            Configuration bb= project.configurations.getByName("implementation")
            bb.canBeResolved=true
            project.gradle.addListener(new EmbedResolutionListener(project, bb))
            def set  = ResolveUtils.resolveArtifacts(bb)
            processArtifacts(set)
        }
```
解析`implementation`引用的依赖，然后解析依赖文件，拿到so文件，并判断大小及是否适配32/64位

## （2）检测三方库含有权限，是否含有我们不使用的权限

命令行执行 `./gradlew checkm`

```java

./gradlew checkm

[mmm] root 不包含权限 key = {group=com.huawei.hms:availableupdate}  权限 = {.GET_COMMON_DATA}
[mmm] root 不包含权限 key = {group=com.immomo.cosmos.photon:thirdpush-xiaomi}  权限 = {.MIPUSH_RECEIVE}
[mmm] root 不包含权限 key = {group=com.cosmos.radar:core}  权限 = {.FOREGROUND_SERVICE}
[mmm] root 不包含权限 key = {group=:SecurityGuardSDK-external-release-5.5.15071059}  权限 = {.WRITE_SETTINGS}
[mmm] root 不包含权限 key = {group=:SecurityGuardSDK-external-release-5.5.15071059}  权限 = {.READ_SETTINGS}
[mmm] root 不包含权限 key = {group=com.immomo.cosmos.auth:auth-cucc}  权限 = {.SYSTEM_ALERT_WINDOW}
[mmm] root 不包含权限 key = {group=com.immomo.cosmos.auth:auth-ctcc}  权限 = {.WRITE_SETTINGS}
[mmm] root 不包含权限 key = {group=com.huawei.hms:push}  权限 = {.PROCESS_PUSH_MSG}
[mmm] root 不包含权限 key = {group=com.huawei.hms:push}  权限 = {.PUSH_PROVIDER}
[mmm] root 不包含权限 key = {group=com.huawei.hms:push}  权限 = {.FOREGROUND_SERVICE}
[mmm] root 不包含权限 key = {group=com.liulishuo.filedownloader:library}  权限 = {.FOREGROUND_SERVICE}
[mmm] root 不包含权限 key = {group=com.immomo.android.mmhttp:mmhttp}  权限 = {.MOUNT_UNMOUNT_FILESYSTEMS}
[mmm] root 不包含权限 key = {group=com.immomo.cosmos.photon:thirdpush-oppo}  权限 = {.RECIEVE_MCS_MESSAGE}
[mmm] root 不包含权限 key = {group=com.immomo.cosmos.photon:thirdpush-oppo}  权限 = {.RECIEVE_MCS_MESSAGE}
[mmm] root 不包含权限 key = {group=com.huawei.hms:update}  权限 = {.GET_COMMON_DATA}

```

### 源码解析

其实和so原理一样，解析`implementation`引用的依赖，然后解析依赖文件，拿到`AndroidManifest.xml`文件，解析xml，拿到权限并和主项目权限进行判断


## （3）检测Res资源文件
执行命令 `./gradlew checkres`

### （1） 自动压缩大于100k的图片

```java
...

[mmm] ""/app/src/main/res/drawable-xxhdpi/pay_success_bg.png  size = 262KB
[mmm] 压缩前 size =  262
[mmm] 压缩后 size =  56
[mmm] copy 完成
[mmm] ""/app/src/main/res/drawable-xxhdpi/bg_qr_share.png  size = 582KB
[mmm] 压缩前 size =  582
[mmm] 压缩后 size =  155
[mmm] copy 完成
[mmm] ""/app/src/main/res/drawable-xxhdpi/bg_every_day_pick.png  size = 119KB
[mmm] 压缩前 size =  119
[mmm] 压缩后 size =  32
[mmm] copy 完成
[mmm] ""/app/src/main/res/drawable-xxhdpi/bg_secretary_dialog.png  size = 216KB
[mmm] 压缩前 size =  216
[mmm] 压缩后 size =  43
[mmm] copy 完成
[mmm] 此次共压缩 {7706}

```

### （2） 检测是否有重复资源（比如多个shape.xml文件名字不同，但是内容相同）

```java

...

[mmm] 重复文件
[mmm] ""/app/src/main/res/drawable/bg_10dp_chat_white.xml
[mmm] ""/app/src/main/res/drawable/bg_luck_telephone_part2.xml
[mmm] 重复文件
[mmm] ""/app/src/main/res/drawable-xhdpi/random_match_avatar_12.png
[mmm] ""/app/src/main/res/drawable-xhdpi/random_match_avatar_15.png
[mmm] 重复文件
[mmm] ""/app/src/main/res/drawable/bg_dialog_bublegumpink.xml
[mmm] ""/app/src/main/res/drawable/bg_dialog_negtive_two.xml
[mmm] 重复文件
[mmm] ""/app/src/main/res/drawable/bg_button_redcommon.xml
[mmm] ""/app/src/main/res/drawable/bg_button_enable.xml
[mmm] 重复文件
[mmm] ""/app/src/main/res/drawable/bg_fe377f_radius_29dp.xml
[mmm] ""/app/src/main/res/drawable/bg_29_shape_fe377f.xml
[mmm] 重复文件
[mmm] ""/app/src/main/res/drawable-xxhdpi/single_chat_input_ic_audio_unlock.png
[mmm] ""/app/src/main/res/drawable-xxhdpi/single_chat_input_ic_audio.png
[mmm] 此次共找到重复文件 {15}

```

### 源码解析

（1）其原理就是拿到项目的资源文件，首先判断png图片大小，大于100k 进行压缩，用的是Tinify 进行压缩数据
（2）拿到资源文件后，对每个文件进行MD5储存，如果有相同的MD5说明有相同文件（如果xml文件内容相同，但是换行，空格，也可以准确判断是重复文件）

 