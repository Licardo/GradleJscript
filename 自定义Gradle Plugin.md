### 自定义Gradle Plugin

#### 自定义gradle plugin目前有三种方式

1. build.gradle文件中自定义Plugin

   ```java
   class PaofanPlugin implements Plugin<Project> {
   
       @Override
       void apply(Project target) {
           target.task('trace') {
               println('this is a gradle plugin')
           }
       }
   }
   ```

   ```groovy
   apply plugin: PaofanPlugin
   ```

   

2. 创建buildSrc模块，实现方式和第三种自定义插件几乎相同，区别在于该方式不需要上传maven，只能用在自己项目中
3. 新建Plugin模块，在模块中实现插件，重点讲这种方式

#### 实现方式

1. 新建Module项目，并在build.gradle中加入groovy引用:

```groovy
apply plugin:groovy

dependencies {
    implementation gradleApi()
    implementation localGroovy()
}
```



2. 修改目录，src-main-java改为src-mian-groovy,结构如下:<img src="/Users/liepu/Downloads/gradle_plugin_category.png" alt="目录" style="zoom:50%;" />

3. 新建Plugin文件，比如：LTracePlugin.groovy。groovy文件需要手动加上package,否则后面会找不到插件

```java
package com.liepu.plugin
import org.gradle.api.Plugin
import org.gradle.api.Project

public class LTracePlugin implements Plugin<Project> {

    @Override
    void apply(Project project) {
        project.getExtensions().create('liepu', LConfig)
        project.task('trace') {
            doLast {
                println(project['liepu'].param)
            }
        }
    }
}
```

4. 新建resources/META-INF/gradle-plugins目录，该目录下新建xxx.properties文件，该文件名就是将来引用的插件的ID，比如com.liepu.trace.properties, 引用插件方式：apply plugin: com.liepu.trace

5. xxx.properties是插件入口，内容是自定义的Plugin

```groovy
implementation-class=com.liepu.plugin.LTracePlugin
```

6. 在build.gradle中加入maven引用和task,需要把写好的插件上传到maven本地或者远程仓库

```groovy
apply plugin:maven

uploadArchives {
    repositories {
        mavenDeployer {
            //设置插件的GAV参数
            pom.groupId = 'com.liepu.plugin'
            pom.artifactId = 'trace'
            pom.version = '1.0.0'
            //文件发布到下面目录
            repository(url: uri('../repo'))
        }
    }
}
```

7. 项目根build.gradle中加入插件的引用

```groovy
buildscript {
    repositories {
        google()
        jcenter()
        maven {
            url uri("repo/")
        }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.3.2'
        classpath 'com.liepu.plugin:trace:1.0.0'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
```

8. app主模块中引入插件,到此自定义Gradle 插件完成

```groovy
apply plugin: 'com.liepu.trace-plugin'
```

#### 需要注意的点

1. xxx.properties文件名xxx和apply plugin: 'xxx'必须保持一致
2. xxxPlugin必须手动加上package com.xxx,否则会报插件找不到错误
3. maven上传的groupid、artifactId、version等必须和classpath:xxx保持一致