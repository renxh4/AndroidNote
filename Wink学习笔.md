# 从开始流程分析

## 首先创建一个Wink Task

```java
 public void createWinkTask(Project project) {
        project.getTasks().register("wink", task -> {
            task.doLast(task1 -> JavaEntrance.main(new String[]{"."}));
        }).get().setGroup(Settings.NAME);
    }
```

## 然后添加Wink Version

```java
Settings.data.newVersion = System.currentTimeMillis() + "";
appExtension.getDefaultConfig().addBuildConfigField(new ClassFieldImpl("String",
                "WINK_VERSION", "\"" + Settings.data.newVersion + "\""));
```

### 补丁识别
补丁有个问题：App怎么识别我该不该加载某个补丁

具体做法是：

全量编译时，Hook Task: preDebugBuild，往BuildConfig里面埋入WINK_VERSION

打补丁时，将补丁按规则命名为 “${WINK_VERSION}xxx”
APP启动，获取自身BuildConfig中的WINK_VERSION值，再根据版本号匹配补丁
补丁匹配成功加载，否则忽略


##  资源ID固定 stableId

```java
 appExtension.aaptOptions(aaptOptions -> {
            WinkLog.d("aaptOptions", "开始aapt配置 execute!");
            String stableIdPath = project.getRootDir() + "/.idea/" + "renxh" + "/stableIds.txt";
            String winkFolder = project.getRootDir() + "/.idea/" + "renxh";
            File file = new File(stableIdPath);
            File lbfolder = new File(winkFolder);
            if (!lbfolder.exists()) {
                lbfolder.mkdir();
            }
            if (file.exists()) {
                WinkLog.d("aaptOptions", "开始aapt配置 execute! 文件存在  " + file.getAbsolutePath());
                aaptOptions.additionalParameters("--stable-ids", file.getAbsolutePath());
            } else {
                WinkLog.d("aaptOptions", "开始aapt配置 execute! 文件不存在");
                aaptOptions.additionalParameters("--emit-ids", file.getAbsolutePath());
            }
        });
```
aaptOptions配置，并且采用插件hook方式，帮你完成
https://fucknmb.com/2017/11/15/aapt2%E9%80%82%E9%85%8D%E4%B9%8B%E8%B5%84%E6%BA%90id%E5%9B%BA%E5%AE%9A/ 固定id学习网址

aaptOptions.additionalParameters("--stable-ids", file.getAbsolutePath());
aaptOptions.additionalParameters("--emit-ids", file.getAbsolutePath());
问题：新增资源，重命名为什么之前会报找不到资源ID的错？

其实是如上述所说的打包流程生成ap_的同时是产生了R文件的，此时需要带上所有新编译出的R.class编译java.kotlin代码；所以说是必须先生成资源再编译代码的严格顺序。

## 每次编译完成后做的事情

### 初始环境
这些都是获取系统的一些参数，比如获取 java home sdkversion 等
```java
   private void createEnv(Project project) {
        this.project = project;

        AppExtension androidExt = (AppExtension) project.getExtensions().getByName("android");

        Env env = Settings.env;
        env.javaHome = getJavaHome();
        env.sdkDir = androidExt.getSdkDirectory().getPath();
        env.buildToolsVersion = androidExt.getBuildToolsVersion();
        env.buildToolsDir = FileUtils.join(androidExt.getSdkDirectory().getPath(),
                "build-tools", env.buildToolsVersion);
        env.compileSdkVersion = androidExt.getCompileSdkVersion();
        env.compileSdkDir = FileUtils.join(env.sdkDir, "platforms", env.compileSdkVersion);
        。。。。
        //遍历所有的moudle 然后把每个moudle的基本编译参数记录下来
        findModuleTree2(project, "");

        //记录version
        if (!Settings.data.newVersion.isEmpty()) {
            WinkLog.d("version","有vesion");
            env.version = Settings.data.newVersion;
            Settings.data.newVersion = "";
            WinkLog.d("version","查看version"+Settings.env.version);
        }else {
            WinkLog.d("version","无vesion"+Settings.data.newVersion);
        }
        //缓存整个evn 数据到本地
        Settings.storeEnv(env, project.getRootDir() + "/.idea/" + Settings.NAME + "/env");
        //重新赋值data，最主要的是赋值每个moudle的编译参数
        initData(project);
    }
```
这个主要是获取不同moudle的 一些基本信息，比如 dir kotlin编译参数，java编译参数的获取
```java
    private void initProjectData(ProjectFixedInfo fixedInfo, Project project, boolean foreInit) {
        WinkLog.d("initProjectData",foreInit+"/");
        long findModuleEndTime = System.currentTimeMillis();
        fixedInfo.name = project.getName();
//        fixedInfo.isProjectIgnore = isIgnoreProject(fixedInfo.name);
        WinkLog.d("initProjectData","0");
        if (fixedInfo.isProjectIgnore && !foreInit) {
            return;
        }

       。。。。。
        Settings.env.kaptProcessingClasspath = apSb.toString().replaceFirst(":", "");


        Settings.env.jvmTarget = "-jvm-target " + getSupportVersion(javaCompile.getTargetCompatibility());
        kotlinArgs.add("-jvm-target");
        kotlinArgs.add(getSupportVersion(javaCompile.getTargetCompatibility()));

        kotlinArgs.add("-d");
        kotlinArgs.add(Settings.env.tmpPath + "/tmp_class");

        StringBuilder sbKotlin = new StringBuilder();
        for (int i = 0; i < kotlinArgs.size(); i++) {
            sbKotlin.append(" ");
            sbKotlin.append(kotlinArgs.get(i));

        }

        fixedInfo.kotlincArgs = sbKotlin.toString();
    }
```
### 产生快照

```kotlin
   scanPathCode = "${project.fixedInfo.dir}/src/main/java"
   scanPathRes = "${project.fixedInfo.dir}/src/main/res"

    csvPathCode = "${diffDir}/md5_code.csv"
    csvPathRes = "${diffDir}/md5_res.csv"
```

```kotlin
private fun genSnapshotAndSaveToDisk(path: String, csvPath: String) {
//        Log.v(TAG, "[${project.fixedInfo.name}]:遍历目录[$path]下的java,kt文件,生成md5并保存到csv文件[$csvPath]")

        val csvFile = File(csvPath)
        if (!csvFile.exists()) {
            csvFile.parentFile.mkdirs()
            csvFile.createNewFile()
        }

        val timeBegin = System.currentTimeMillis()
        File(path).walk()
            .filter {
                it.isFile
            }
            .filter {
                it.extension in extensionList
            }
            .forEach {
                val row = listOf(it.absolutePath, getSnapshot(it))
                csvWriter.open(csvPath, append = true) {
                    writeRow(row)
                }
            }

        WinkLog.d(TAG, "[${project.fixedInfo.name}]:耗时:${System.currentTimeMillis() - timeBegin}ms")

    }
```
其实就是直接遍历 `/src/main/java  src/main/res` 下的文件，然后把取文件的更改时间戳 然后存入 csv 文件中 ，之后就对比时间戳是否一样来判断是否跟更改

到这里初始化的事情就完成了


## 运行WinkTask 进行增量编译

```java
 public static void main(String[] args) {

           ...
           //判断本地是否有env 环境缓存
            if (helper.isEnvExist(path)) {
                // Increment
                WinkLog.d("isEnvExist");
                //如果有缓存就把env加载到内存中
                helper.initEnvFromCache(path);
            }else {
                WinkLog.d("isEnvExistretrue");
                return;
            }

            // Diff file changed
            WinkLog.d("mmm","rundiff");
            //进行差异对比，这个其实就是把之前缓存的快照取出，然后和本地的快照进行对比，看是否有改变
            runDiff();
            //判断是否有资源
            new ResourceHelper().checkResourceWithoutTask(); // 内部判断：Settings.data.hasResourceChanged
            new CompileHelper().compileCode();
            if (new IncrementPatchHelper().patchToApp()) {
                updateSnapShot();
            }

    }
```

### 首先说一下资源编译

如果有资源改动的话，就进行资源编译


复用gradle task-> `processDebugResources`

产物会生成在build下各个不同文件夹下，并且是已经做过merge的。产物是ap_和R文件。

现在资源都是用的aapt2

资源的打包过程分为两个阶段

编译Compile：将资源文件编译为二进制格式，也就是*.flat，对应aapt2 compile xxx
链接Link: 合并所有已编译的文件并将它们打包到一个软件包中。对应aapt2 link xxx
对于values资源，因为之前全量编译的产物是合并过的，所以不能单独使用单个模块修改的.flat替换合并过的.flat

目前是会以offline模式跑一遍Gradle Task。

```shell
sehllcmd:./gradlew processDebugResources --offline
```

问题：

build中的资源产物ap_并不包含assets资源，这时我们手动解压ap_带上assets再次打包

```java
unzip -o -q /Users/momo/Desktop/github/gradle/app/build/intermediates/processed_res/debug/out/resources-debug.ap_ -d /Users/momo/Desktop/github/gradle/.idea/renxh/tempResFolder
cp -R /Users/momo/Desktop/github/gradle/app/build/intermediates/merged_assets/debug/out/. /Users/momo/Desktop/github/gradle/.idea/renxh/tempResFolder/assets
cd /Users/momo/Desktop/github/gradle/.idea/renxh/tempResFolder
zip -r -o -q /Users/momo/Desktop/github/gradle/.idea/renxh/1654677276413_resources-debug.apk *

```
到此为止资源编译完成



### java 和 kotlin 编译

就是带上java和kotlin的编译环境 对有改动的类进行编译，然后利用d8命令编译成dex 其中如果有资源改动，需要加上r文件进行编译

```       
 cmds += " " + Settings.env.appProjectDir + "/build/intermediates/compile_and_runtime_not_namespaced_r_class_jar/" + Settings.env.variantName + "/R.jar";     
```

## 推送到机器

这个没什么好说的，就是把编译后的数据直接推到机器，然后重启app


首先adb将补丁推到应用外部存储私有目录 /sdcard/Android/data/packageName，如果没有权限，再推到sdcard目录下
然后推送的代码补丁.jar和资源补丁.apk都将后缀改成.png
应用启动，先读取/sdcard/Android/data/packageName，读不到再读取sdcard目录下
Q：什么情况下/sdcard/Android/data/packageName没有权限
A：小米11实现了自身的沙盒隔离，adb没法PUSH到/sdcard/Android/data/packageName

Q：为什么将后缀改成.png
A：在无法PUSH补丁到/sdcard/Android/data/packageName的情况下，PUSH到sdcard目录下，但安卓11强直使用了分区存储，应用无法访问外部存储，但可以访问图片文件，后缀改成.png成功绕过分区存储权限限制且后缀不影响补丁加载

## 加载补丁文件


### App加载补丁
App启动后，向PathClassLoader的dexElements进行插入新补丁的jar包，达到优先加载增量编译补丁的目的


### 加载资源产物
在application启动时（ContentProvider的奇技淫巧）

替换AssetManager成我们自己的xx-debug.apk纯资源的apk。

遍历ResourceManager内resource缓存集合，都换成我们新的AssetManager

关键代码：
```java
for (WeakReference<Resources> wr : references) {
  final Resources resources = wr.get();
  if (resources == null) {
    continue;
  }
  // Set the AssetManager of the Resources instance to our brand new one
  try {
    //pre-N
    assetsFiled.set(resources, newAssetManager);
  } catch (Throwable ignore) {
    // N
    final Object resourceImpl = resourcesImplFiled.get(resources);
    // for Huawei HwResourcesImpl
    final Field implAssets = findField(resourceImpl, "mAssets");
    implAssets.set(resourceImpl, newAssetManager);
  }

  clearPreloadTypedArrayIssue(resources);

  resources.updateConfiguration(resources.getConfiguration(), resources.getDisplayMetrics());
}
```
遗留问题与改进
非values资源单个增量，继续尝试aapt2 link






