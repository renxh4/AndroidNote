# 什么是插件
IntelliJ插件其实就是平时我们使用Ide时使用的一些快捷工具，例如：我们平时经常使用的，快速生成get和set方法

# 怎么创建项目


# 基本信息介绍
[ id ] 插件id，类似于Android项目的包名，不能和其他插件项目重复，所以推荐使用com.xxx.xxx的格式
[ name ] 插件名称，别人在官方插件库搜索你的插件时使用的名称
[ version ] 插件版本号
[ vendor ] 插件发布者信息，可以添加邮箱链接
[ description ] 插件描述信息，在这里可以介绍你的插件内容，支持HTML标签
[ change-notes ] 插件版本变更日志，支持HTML标签
[ idea-version ] 支持的idea版本  分为since-build最低版本 和 until-build最高版本，两个属性可以任选一或者同时使用  官网有详细介绍 www.jetbrains.org/intellij/sd…  大体规则为 since-build <= 支持版本 < until-build
[ extensions ] 自定义扩展，
[ actions ] 具体的插件动作


# 开始编写插件



