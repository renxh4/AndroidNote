# 注意Mac与Linux精确不到毫秒值,所以这里取秒 快照
private fun getSnapshot(it: File) = ((it.lastModified() / 1000).toString())

# 一些优化思路
* 利用git diff, 但是无法获取未添加到版本管理的文件;
* 利用idea的changlist获取未添加到版本控制的文件;

# 编译
* 最大的难点在于执行编译时classpath的获取，Hook Gradle task，在全量编译后，通过对project的读取获取javaCompileProvider对象，通过此对象可获取classpath
* 增量的文件、R文件、项目已经编译好的class也会加入classpath，保证编译依赖正常
* Kotlinc编译要在Javac之前，要不然无法正确的依赖Kotlinc编译出的java class

# 资源编译
复用gradle task-> processDebugResources

产物会生成在build下各个不同文件夹下，并且是已经做过merge的。产物是ap_和R文件。

现在资源都是用的aapt2

目前是会以offline模式跑一遍Gradle Task。

问题：

build中的资源产物ap_并不包含assets资源，这时我们手动解压ap_带上assets再次打包

# 资源ID固定 stableId
aaptOptions配置，并且采用插件hook方式，帮你完成

aaptOptions.additionalParameters("--stable-ids", file.getAbsolutePath());
aaptOptions.additionalParameters("--emit-ids", file.getAbsolutePath());
问题：新增资源，重命名为什么之前会报找不到资源ID的错？

其实是如上述所说的打包流程生成ap_的同时是产生了R文件的，此时需要带上所有新编译出的R.class编译java.kotlin代码；所以说是必须先生成资源再编译代码的严格顺序。

# 补丁推送
增量补丁生成后，补丁过程

首先adb将补丁推到应用外部存储私有目录 /sdcard/Android/data/packageName，如果没有权限，再推到sdcard目录下
然后推送的代码补丁.jar和资源补丁.apk都将后缀改成.png
应用启动，先读取/sdcard/Android/data/packageName，读不到再读取sdcard目录下
Q：什么情况下/sdcard/Android/data/packageName没有权限
A：小米11实现了自身的沙盒隔离，adb没法PUSH到/sdcard/Android/data/packageName

Q：为什么将后缀改成.png
A：在无法PUSH补丁到/sdcard/Android/data/packageName的情况下，PUSH到sdcard目录下，但安卓11强直使用了分区存储，应用无法访问外部存储，但可以访问图片文件，后缀改成.png成功绕过分区存储权限限制且后缀不影响补丁加载


# 补丁识别
补丁有个问题：App怎么识别我该不该加载某个补丁

具体做法是：

全量编译时，Hook Task: preDebugBuild，往BuildConfig里面埋入WINK_VERSION
```java
    // Embedded WINK_VERSION.
    GradleUtils.getFlavorTask(project, "pre", "DebugBuild").doFirst(task -> {
        Settings.data.newVersion = System.currentTimeMillis() + "";
        ((AppExtension) project.getExtensions().getByName("android"))
                .getDefaultConfig().buildConfigField("String",
                "WINK_VERSION", "\"" + Settings.data.newVersion + "\"");
    });
```java
打补丁时，将补丁按规则命名为 “${WINK_VERSION}xxx”
APP启动，获取自身BuildConfig中的WINK_VERSION值，再根据版本号匹配补丁
补丁匹配成功加载，否则忽略


# 遇到的其他坑（Tips）
Gradle Task 点击执行跟 脚本执行Task的环境不一样，脚本执行用的是系统的环境，点击用的是studio的环境，例如Java版本点击执行和脚本执行用的可能不一样

Java8 File#lastModified() 精确到秒，其他Java版本是毫秒

Gradle Task无法嵌套执行Task，但可以通过脚本执行，但是，脚本执行后会启动新的gradle daemon，速度会非常慢!!

dex工具不支持Java8 lambda，需要换成d8，效果一样

repositories 按顺序遍历库，不需要的干掉，404的话会拖慢gradle同步速度，如果是403会极大的拖慢速度，这块可以是工程优化的空间。PS：赫兹同步要10min，就是这个原因

Gradle版本和对应AGP版本适配，目前测试过5.4.1 ~ 6.1.1

kotlinc 执行 kapt 命令时，变更的文件至少包含一个 kotlin 文件，否则不会进行编译