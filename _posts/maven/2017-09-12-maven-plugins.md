---
layout:     post
title:      Maven java程序打包插件
category: blog
description:  
---



### Maven项目里如何对java可执行程序打包成规范化可执行包
```
 <build>
	  <plugins>
	          <plugin>
	              <artifactId>maven-compiler-plugin</artifactId>
	              <configuration>
	                  <source>1.8</source>
	                  <target>1.8</target>
	              </configuration>
	 		  </plugin>
	 		  
	 		  <plugin>
	                 <groupId>org.codehaus.mojo</groupId>
	                 <artifactId>appassembler-maven-plugin</artifactId>
	                 <version>2.0.0</version>
	                 <executions>
	                     <execution>
	                         <phase>package</phase>
	                         <goals>
	                             <goal>assemble</goal>
	                         </goals>
	                     </execution>
	                 </executions>
	                 <configuration>
	                     <assembleDirectory>${project.build.directory}/${app.assemble.dirname}
	                     </assembleDirectory>
	                     <binFileExtensions>
	                         <unix>.sh</unix>
	                     </binFileExtensions>
	                     <filterConfigurationDirectory>true</filterConfigurationDirectory>
	                     <configurationSourceDirectory>
	                     	src/main/resources/${profiles.active}
	                     </configurationSourceDirectory>
	                     <copyConfigurationDirectory>true</copyConfigurationDirectory>
	                     <configurationDirectory>conf</configurationDirectory>
	                     <includeConfigurationDirectoryInClasspath>true</includeConfigurationDirectoryInClasspath>
	                     <repositoryLayout>flat</repositoryLayout>
	                     <repositoryName>lib</repositoryName>
	                     <logsDirectory>logs</logsDirectory>
	                     <useWildcardClassPath>true</useWildcardClassPath>
	                     <useAllProjectDependencies>false</useAllProjectDependencies>
	                     <useTimestampInSnapshotFileName>false</useTimestampInSnapshotFileName>
	                     <platforms>
	                         <platform>unix</platform>
	                         <platform>windows</platform>
	                     </platforms>
	                     <programs>
	                         <program>
	                             <id>start</id>
	                             <mainClass>Bootstrap</mainClass>
	                         </program>
	                     </programs>
	                 </configuration>
	           </plugin>
	   </plugins>
   </build>
   
   <profiles>
		<profile>
			<!-- 本地开发环境 -->
			<id>dev</id>
			<properties>
				<profiles.active>dev</profiles.active>
			</properties>
			<activation>
				<activeByDefault>true</activeByDefault>
			</activation>
		</profile>
		<profile>
			<!-- 测试环境 -->
			<id>test</id>
			<properties>
				<profiles.active>test</profiles.active>
			</properties>
		</profile>
		<profile>
			<!-- 预发布环境 -->
			<id>preRelease</id>
			<properties>
				<profiles.active>preRelease</profiles.active>
			</properties>
		</profile>
		<profile>
			<!-- 线上环境 -->
			<id>onLine</id>
			<properties>
				<profiles.active>onLine</profiles.active>
			</properties>
		</profile>
		<profile>
			<!-- 私有化环境 -->
			<id>private</id>
			<properties>
				<profiles.active>private</profiles.active>
			</properties>
		</profile>
	</profiles>

```

