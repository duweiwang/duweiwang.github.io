---
title: Android的maven插件和maven-publish插件
date: 2024-05-01 21:58:50
tags:
categories:
description:
---


#### 概述
由于maven插件已经废弃，被maven-publish插件替代了，有些老项目还在使用maven插件，所以需要知道如何升级，这里做一个对比。

#### maven插件

以下是总体配置，具体还会涉及：是否上传源码

```groovy
apply plugin: 'maven'

uploadArchives {
    repositories {
        mavenDeployer {
            //release 还是snapshot打包
            def isRelease = project.AAR_BUILD_TYPE == "release"
            
            if (isRelease) {
                repository(url: project.RELEASE_URL) { //正式仓库地址
                    authentication(userName: project.MAVEN_USERNAME, password: project.MAVEN_PASSWORD)
                }
                pom.version = project.AAR_VERSION//eg: 1.0.0
            } else {
                repository(url: project.SNAPSHOT_URL) {//测试仓库地址
                    authentication(userName: project.MAVEN_USERNAME, password: project.MAVEN_PASSWORD)
                }
                pom.version = project.AAR_SNAPSHOT_VERSION//eg: 1.0.0-shapshot
            }
            pom.groupId = project.POM_GROUPID//eg：com.github.bumptech.glide
            pom.artifactId = project.POM_ARTIFACTID//eg: glide
            pom.name = project.POM_NAME// eg: glide
            pom.packaging = project.POM_PACKAGING//eg: aar/jar
        }
    }
}

```



#### maven-publish插件

```groovy
apply plugin: 'maven-publish'

publishing {
    repositories {//仓库
        maven {
            url = "https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/"
            credentials {
                username = "sonatypeUsername"
                password = "sonatypePassword"
            }
        }
    }
    
    publications {//发布产物
        //下面这个名字可以自己定义如：debug、release
        maven(MavenPublication) {
            
            //配置产物，三种方式
            //1、依赖bundleReleaseAar任务，并上传产物
            afterEvaluate{
                artifact(tasks.getByName("bundleReleaseAar"))
            }
            //2、直接指定路径
            artifact "$buildDir/outputs/aar/${project.name}-release.aar"
            //3、结合agp插件
            
            groupId = 'com.example'
            artifactId = 'my-library'
            version = '1.0.0'

            pom {
                name = 'My Library'
                description = 'A description of my library'
                url = 'https://github.com/example/my-library'
            }
        }
    }

}

```


结合agp插件，指定产物：
```groovy
afterEvaluate {
    publishing {
        repositories {//仓库
        
        }
    }

    publications {//发布产物
        maven(MavenPublication) {
            from components.release//发布内容
            artifact sourcejar//上传源码
        }
    }
    
}

```


#### 参考资料
[maven插件 和 maven-publish 插件的区别](https://blog.csdn.net/h_bpdwn/article/details/122479437)
[Maven 插件与 Maven-Publish 插件的差异](https://www.bytezonex.com/archives/R8trm3Wz.html)