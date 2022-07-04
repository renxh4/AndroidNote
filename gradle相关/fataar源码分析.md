# fataar 源码分析

## 首先创建embed 配置，可以直接在gradle中引用

```java
    embed 'com.github.bumptech.glide:okhttp3-integration:4.11.0'
```
```java
 //创建embed配置项
    private void createConfigurations() {
        //这个其实就是创建embed
        Configuration embedConf = project.configurations.create(CONFIG_NAME)
        createConfiguration(embedConf)
        FatUtils.logInfo("Creating configuration embed")
        //这个其实就是创建debugEmbed   releaseEmbed
        project.android.buildTypes.all { buildType ->
            String configName = buildType.name + CONFIG_SUFFIX
            FatUtils.printDebugmm("createConfigurations buildTypes" + "///"+configName)
            Configuration configuration = project.configurations.create(configName)
            createConfiguration(configuration)
            FatUtils.logInfo("Creating configuration " + configName)
        }
        //这个其实是创建渠道包的配置项  xxxembed xxxdebugEmbed xxxreleaseEmbed
        project.android.productFlavors.all { flavor ->
            String configName = flavor.name + CONFIG_SUFFIX
            Configuration configuration = project.configurations.create(configName)
            createConfiguration(configuration)
            FatUtils.logInfo("Creating configuration " + configName)
            project.android.buildTypes.all { buildType ->
                String variantName = flavor.name + buildType.name.capitalize()
                String variantConfigName = variantName + CONFIG_SUFFIX
                Configuration variantConfiguration = project.configurations.create(variantConfigName)
                createConfiguration(variantConfiguration)
                FatUtils.logInfo("Creating configuration " + variantConfigName)
            }
        }
    }
```

## 创建Transform

这个主要处理R文件

```java
   //创建Transform
    private registerTransform() {
        transform = new RClassesTransform(project)
        // register in project.afterEvaluate is invalid.
        project.android.registerTransform(transform)
    }
```


## 然后再gradle配置完成之后做一些事情

```java
  project.afterEvaluate {
            doAfterEvaluate()
        }
```

```java
    private void doAfterEvaluate() {
        //embedConfigurations 中存储的是上面生成embed 和 debugEmbed 等
        embedConfigurations.each {
            if (project.fataar.transitive) {
                it.transitive = true
            }
        }

        project.android.libraryVariants.all { LibraryVariant variant ->
            //variant.name = debug 和 release
            FatUtils.printDebugmm("libraryVariants.all = variant  = " +variant)
            FatUtils.printDebugmm("libraryVariants.all = variant.name  = " +variant.name)
            FatUtils.printDebugmm("libraryVariants.all = variant.getFlavorName()  = " +variant.getFlavorName())
            FatUtils.printDebugmm("libraryVariants.all = variant.getBuildType().name  = " +variant.getBuildType().name)
            Collection<ResolvedArtifact> artifacts = new ArrayList()
            Collection<ResolvedDependency> firstLevelDependencies = new ArrayList<>()
            //这个循环的意思是 如果新创建的 embed 或者 debugEmbed 和 variant.name 相同则继续走
            embedConfigurations.each { configuration ->
                FatUtils.printDebugmm("libraryVariants.all = configuration.name = " +configuration.name)
                if (configuration.name == CONFIG_NAME
                        || configuration.name == variant.getBuildType().name + CONFIG_SUFFIX
                        || configuration.name == variant.getFlavorName() + CONFIG_SUFFIX
                        || configuration.name == variant.name + CONFIG_SUFFIX) {
                    FatUtils.printDebug("configuration.name = ${configuration.name}")
                    Collection<ResolvedArtifact> resolvedArtifacts = resolveArtifacts(configuration)
                    artifacts.addAll(resolvedArtifacts)
                    artifacts.addAll(dealUnResolveArtifacts(configuration, variant, resolvedArtifacts))
                    firstLevelDependencies.addAll(configuration.resolvedConfiguration.firstLevelModuleDependencies)
                }
            }
            //最终是把embed的依赖项加入了集合
            if (!artifacts.isEmpty()) {
                def processor = new VariantProcessor(project, variant)
                processor.processVariant(artifacts, firstLevelDependencies, transform)
            }
        }
    }
```
这个方法主要是找到embed 引用的依赖，主要是这个方法`resolveArtifacts`

```java
 //这里configuration参数，指的是 embed debugEmbed releaseEmbed
    private Collection<ResolvedArtifact> resolveArtifacts(Configuration configuration) {
        //这个方法的主要作用是，拿到embed引用的依赖，并判断他是aar或者jar包，然后放入集合
        def set = new ArrayList()
        if (configuration != null) {
            configuration.resolvedConfiguration.resolvedArtifacts.each { artifact ->
                if (ARTIFACT_TYPE_AAR == artifact.type || ARTIFACT_TYPE_JAR == artifact.type) {
                    //
                } else {
                    FatUtils.printDebugmm("error artifact name = ${artifact.name}, type = ${artifact.type}")
                    throw new ProjectConfigurationException('Only support embed aar and jar dependencies!', null)
                }
                FatUtils.printDebugmm("resolvedConfiguration.resolvedArtifacts artifact ="+"artifact name = ${artifact.name}, type = ${artifact.type}")
                FatUtils.printDebugmm("resolvedConfiguration.resolvedArtifacts artifact1 ="+artifact)
                //这里的 artifact 指的就是  如 ： okhttp3-integration-4.11.0.aar
                set.add(artifact)
            }
        }
        return set
    }
```

最后通过`processor.processVariant`来进行解析依赖

## processor.processVariant 解析依赖，把aar数据拷贝到主aar

```java
 //artifacts embed的解析后的依赖产物，dependencies embed 的依赖
    void processVariant(Collection<ResolvedArtifact> artifacts,
                        Collection<ResolvableDependency> dependencies,
                        RClassesTransform transform) {
        //拿到preDebugBuild
        String taskPath = 'pre' + mVariant.name.capitalize() + 'Build'
        TaskProvider prepareTask = mProject.tasks.named(taskPath)
        if (prepareTask == null) {
            throw new RuntimeException("Can not find task ${taskPath}!")
        }
        //这里拿到bundleDebug
        TaskProvider bundleTask = VersionAdapter.getBundleTaskProvider(mProject, mVariant)
        preEmbed(artifacts, dependencies, prepareTask)
        //处理产物，把依赖拷贝到build中
        processArtifacts(artifacts, prepareTask, bundleTask)

        processClassesAndJars(bundleTask)
        if (mAndroidArchiveLibraries.isEmpty()) {
            return
        }
        processManifest()
        processResources()
        processAssets()
        processJniLibs()
        processConsumerProguard()
        processGenerateProguard()
        processDataBinding(bundleTask)
        processRClasses(transform, bundleTask)
    }
```
这里就很清楚了，依次处理 Manifast，Resources，assets，jni，等等

接下来主要分析这个方法

## preEmbed

```java
 private void preEmbed(Collection<ResolvedArtifact> artifacts,
                          Collection<ResolvedDependency> dependencies,
                          TaskProvider prepareTask) {
        //创建predebugEmbde 之前打印出依赖
        TaskProvider embedTask = mProject.tasks.register("pre${mVariant.name.capitalize()}Embed") {
            doFirst {
                printEmbedArtifacts(artifacts, dependencies)
            }
        }
        //preDebugBuild 依赖 preDebugEmbde
        prepareTask.configure {
            dependsOn embedTask
        }
    }
```
这个就是创建了predebugEmbde 任务 ，preDebugBuild依赖predebugEmbde，在Task执行之前打印出依赖


## processArtifacts

```java
    /**
     * exploded artifact files 分解依赖
     */
    private void processArtifacts(Collection<ResolvedArtifact> artifacts, TaskProvider<Task> prepareTask, TaskProvider<Task> bundleTask) {
        if (artifacts == null) {
            return
        }
        for (final ResolvedArtifact artifact in artifacts) {
            if (FatAarPlugin.ARTIFACT_TYPE_JAR == artifact.type) {
                if (artifact.file.exists()) {
                    long len = artifact.file.length()
                    long size = len / 1024
                    String gp = artifact.getModuleVersion().getId().getGroup()
                    String na = artifact.getModuleVersion().getId().getName()
                    FatUtils.printDebugmm("[依赖产物][jar] group=${gp}:${na}, size=${size}")
                }
                addJarFile(artifact.file)
            } else if (FatAarPlugin.ARTIFACT_TYPE_AAR == artifact.type) {
                AndroidArchiveLibrary archiveLibrary = new AndroidArchiveLibrary(mProject, artifact, mVariant.name)
                // 获取 build/exploded-aar/com.github.bumptech/glide/debug/classes.jar
                File jarFile = archiveLibrary.classesJarFile
                if (jarFile.exists()) {
                    long len = jarFile.length()
                    long size = len / 1024
                    FatUtils.printDebug("[依赖产物][aar] group=${archiveLibrary.group}:${archiveLibrary.name}, size=${size}")
                }
                addAndroidArchiveLibrary(archiveLibrary)
                Set<Task> dependencies

                if (getTaskDependency(artifact) instanceof TaskDependency) {
                    dependencies = artifact.buildDependencies.getDependencies()
                } else {
                    CachingTaskDependencyResolveContext context = new CachingTaskDependencyResolveContext()
                    getTaskDependency(artifact).visitDependencies(context)
                    if (context.queue.size() == 0) {
                        dependencies = new HashSet<>()
                    } else {
                        dependencies = context.queue.getFirst().getDependencies()
                    }
                }
                //build/intermediates/exploded-aar/com.github.bumptech/glide/debug/
                final def zipFolder = archiveLibrary.getRootFolder()
                zipFolder.mkdirs()
                def group = artifact.getModuleVersion().id.group.capitalize()
                def name = artifact.name.capitalize()
                String taskName = "explode${group}${name}${mVariant.name.capitalize()}"
                //这个就是把依赖数据拷贝到缓存
                Task explodeTask = mProject.tasks.create(taskName, Copy) {
                    from mProject.zipTree(artifact.file.absolutePath)
                    into zipFolder

                    doFirst {
                        // Delete previously extracted data.
                        zipFolder.deleteDir()
                    }
                }

                if (dependencies.size() == 0) {
                    explodeTask.dependsOn(prepareTask)
                } else {
                    explodeTask.dependsOn(dependencies.first())
                }
                Task javacTask = mVersionAdapter.getJavaCompileTask()
                javacTask.dependsOn(explodeTask)
                bundleTask.configure {
                    dependsOn(explodeTask)
                }
                mExplodeTasks.add(explodeTask)
            }
        }
    }

```
首先遍历`artifacts`集合 ，然后判断是`jar还是aar`,
* 如果是jar 就直接打印出大小和名称，然后放入jar集合
* 如果是aar,首先创建`AndroidArchiveLibrary`这个之后再说， 接下来就是把依赖数据，直接拷贝到`build/intermediates/exploded-aar/com.github.bumptech/glide/debug/`,集体到吗如下

```java
  //build/intermediates/exploded-aar/com.github.bumptech/glide/debug/
                final def zipFolder = archiveLibrary.getRootFolder()
                zipFolder.mkdirs()
                def group = artifact.getModuleVersion().id.group.capitalize()
                def name = artifact.name.capitalize()
                String taskName = "explode${group}${name}${mVariant.name.capitalize()}"
                //这个就是把依赖数据拷贝到缓存
                Task explodeTask = mProject.tasks.create(taskName, Copy) {
                    from mProject.zipTree(artifact.file.absolutePath)
                    into zipFolder

                    doFirst {
                        // Delete previously extracted data.
                        zipFolder.deleteDir()
                    }
                }
``` 

## processClassesAndJars

