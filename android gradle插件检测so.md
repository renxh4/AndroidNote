# 检测so
 其根源就是，拿到`implementation`引用的依赖，然后解压依赖文件，拿到依赖文件中的so文件，判断大小及是否适配32和64位  32指得是armeabi  64指的是arm64-v8a
# 检测Manifest
 其根源就是，拿到`implementation`引用的依赖，然后解压依赖文件，拿到依赖文件中的Manifest文件，然后解析xml 拿到权限列表

# 检测res
其根源是，拿到`project.getRootDir().getAbsolutePath() + "/app/src/main/res/` 拿到res中的文件，拿到png图片，判断其大小，然后大于100k的直接进行tinify 进行压缩，然后拿到所有的xml，拿到内容删除空格，然后拿到MD5
 判断md5 是否相同，如果相同则是重复文件