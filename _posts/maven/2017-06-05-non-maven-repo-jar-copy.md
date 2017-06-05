---
layout:     post
title:      Maven非仓库JAR的引入打包并使用的方法
category: blog
description:  有些第三方服务提供商给出的jar包并没有在Maven中央仓库中，又不想手动安装jar到本地仓库，或者上传到私服，那么就可以这样搞。
---



### Maven项目里如何使用第三方非maven仓库的jar

方法1：
```
一种办法就是手动安装这个jar到本地仓库，可以看下边链接
```
[如何手动安装Jar到本地仓库](http://www.ioend.com/maven-cmd "手动安装")
缺点就是如果部署到多台server，就得分别在每一个上边执行。

方法2： 
```
如果有maven私服，并且打包的server也都可以访问到私服, 那么就可以设置上传Jar到私服上去，pom中引用，需要时即可下载。
```

方法3：
如果不想受制于私服，也不想手动安装，就可以将所需要的Jar放到项目的lib目录下。
借助于maven的插件，进行打包时的copy。

```
	 <properties>
	        <appassembler.target.dir.name>application-1.0</appassembler.target.dir.name>
	 </properties>
	<build>
        <plugins>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy-nonmavenjar-resources</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <encoding>UTF-8</encoding>
                            <outputDirectory>${project.build.directory}/${appassembler.target.dir.name}/lib
                            </outputDirectory>
                            <overwrite>true</overwrite>
                            <resources>
                                <resource>
                                    <directory>${basedir}/lib/</directory>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

        </plugins>
    </build>


上边插件就会在package阶段进行lib目录下资源的copy

那么为了在IDE代码编辑中能引用到，可以将lib目录增加到IDE的buildpath中即可。



```

