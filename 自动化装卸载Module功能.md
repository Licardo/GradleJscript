### 自动化装卸载Module功能

#### 1. 功能介绍

> 脚本支持自动装载模块、卸载模块、源码和aar互转功能
>
> 需要少量侵入代码，第一次使用需要确认打出的包是否包含指定依赖

#### 2. 如何引入

1. 添加脚本文件

   > 把dependencyUtil.gradle和wmModuleMavenUrl.gradle文件添加到项目根目录或有权限的子目录

2. 项目build.gradle文件添加依赖

```groovy
apply from: "$rootDir/wmModuleMavenUrl.gradle"

allprojects {
  apply from: "$rootDir/dependencyUtil.gradle"
}
```

3. 修改settings.gradle文件的依赖

```groovy
apply from: "$rootDir/dependencyUtil.gradle"
// 读取模块配置文件
println(">>>项目所有源码依赖的module：\n$sourceModules")
sourceModules.each {
    include(":$it")
//    project(":$it").projectDir = new File(settingsDir, "../$it")
}
```

4. 根目录新建modules配置文件如下：

```groovy
// 注释中不能存在符号 =
// 配置模式为：模块名=0/1/2   0:表示卸载模块 1:表示源码依赖，2：表示maven依赖
// 业务模块
app=1
wmp_user=1
wmp_domain=1
wmp_login=1
wmp_h5=1
wmp_fastmedical=1
pdfreader=1
ocr_ui=1
faceplatform=1
faceplatform-ui=1
// 基础模块
lib_dokit=1
lib_webview=1
lib_share=1
lib_imageload=1
lib_wmagent=1
lib_network=1
wmp_base=1
```

5. 如模块已部署maven仓库，wmMavenUrl.gradle文件中追加maven地址，格式：模块名=maven地址

```groovy
ext.Libs = [
  'base'	:	'com.xxx.lib:base:1.0.1'
]
```

6. module依赖方式修改(modules配置文件中的模块引用都需要用新的依赖方式)

```groovy
// 改造前的依赖方式
api ':user'
api ':base'

// 改造后的依赖方式
wmApi(['user', 'base'])
wmImplementation(['xxx'])
wmCompileOnly(['xxx'])
// 目前支持以上三种依赖方式，如业务需要，可以自定义改造Google提供的其他依赖方式
```



#### 3. 操作指南

1. 新增模块 如：模块名：base,  maven: com.xxx.lib:base:1.0.1

   1. 需要在modules文件中添加: base=1  // :0:表示卸载模块 1:表示源码依赖，2：表示maven依赖

   ```groovy
   base=1
   ```

   2. 如有maven地址，需要在wmMavenUrl.gradle文件中追加maven地址，格式：模块名=maven地址

   ```groovy
   ext.Libs = [
     'base'	:	'com.xxx.lib:base:1.0.1'
   ]
   ```

   3. Sync Project

2. 卸载模块、源码和aar转操作

   1. 修改modules配置文件

   ```groovy
   // 卸载base模块
   base=0
   // 源码依赖
   base=1
   // aar依赖
   base=2
   ```

   2. Sync Project



#### 4. 注意问题

1. 需要注意脚本中引用的目录和开发者自己的目录是否一致
2. wmModuleMavenUrl.gradle脚本中maven依赖的模块一定大于等于modules配置文件中module=2的模块，否则可能找不到maven仓库，编译报错
3. 需要把所有可能出现源码和aar互相切换的模块放在modules配置文件中，只有aar依赖的module不属于该脚本处理范围
4. 如module没有maven依赖地址，只需要操作modules文件，脚本提供装卸载模块的功能

#### 5. 待优化

1. 提升modules配置文件操作的体验
2. 简化流程，比如新加module，开发者需要在modules配置文件中添加模块，如有aar地址，还需要添加wmMavenUrl.gradle文件

#### 6. 原理解析

> 1. 首先读取modules配置文件的内容，根据规则解析文件并添加到源码集合和aar集合中
> 2. 依赖模块时脚本会根据源码集合或aar集合调用相应的官方依赖API

定制依赖引用的核心代码：

```groovy
def addDependencies(DependencyHandler dependencyHandler, String module, String config) {
    if (sourceModules.contains(module)) {
        // 源码依赖
        dependencyHandler.add(config, dependencyHandler.project(path: ":$module"))
    } else if (aarModules.contains(module)) {
        // maven依赖
        if (Libs[module] != null) {
            dependencyHandler.add(config, Libs[module])
        } else {
            throw new GradleException("$module 模块没有对应的maven包")
        }
    } else {
        // 卸载模块
        println("卸载了$module 模块")
    }
}
```