android studio 引用aar包 冲突
===


# 文件冲突

## 文件错误提示

       一般类似这样的Error:Execution failed for task ‘:app:transformResourcesWithMergeJavaResForDebug’.> com.android.build.api.transform.TransformException: com.android.builder.packaging.DuplicateFileException: Duplicate files copied in APK META-INF/maven/com.squareup.okio/okio/pom.xml File1: C:\Users\WX_JIN.gradle\caches\modules-2\files-2.1\com.squareup.okio\okio\1.6.0\98476622f10715998eacf9240d6b479f12c66143\okio-1.6.0.jar File2: D:\Android\workspace\wxj\YK\app\build\intermediates\exploded-aar\YK\umenglibrary\unspecified\jars\classes.jar 
上面提示Duplicate files copied in APK META-INF/maven/com.squareup.okio/okio/pom.xml 
重复这个文件，我们只要去掉一个或者忽略一个就行了 
![提示错误类似这个](http://img.blog.csdn.net/20160311100913124)

## 解决方案

       在主项目中添加build->android->添加packagingOptions exclude 包含重复的文件 
![解决方案截图](http://img.blog.csdn.net/20160311101044328)

# jar冲突

## 冲突提示

![错误提示](http://img.blog.csdn.net/20160311101529330)

## 解决方案

![解决一](http://img.blog.csdn.net/20160311101549219) 
![解决二](http://img.blog.csdn.net/20160311101606657) 
使用上面这种忽略掉重复的依赖包

# 资源冲突

有时候我们在集成第三方aar包时会发现aar里面引用的资源和自己工程的里面的某些资源文件名称一样，这会在打包时会报错，并提示某个资源文件重复 
怎样解决问题呢？我们可以在aar文件找到和自己工程的那些资源重复，并删除重新生成一个新的aar即可，步骤如下

```java
//解压aar文件到tmpDir目录下
unzip myLib.aar -d tmpDir 

//删除tmpDir中和工程中重复资源文件

//将tmpDir重新打包成一个新的aar
jar cvf myNewLib.aar -C tmpDir/ .
```
