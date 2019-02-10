---
title: maven打包
date: 2019-02-10 11:50:13
tags:
   - maven
   - assemlby
   - 打包
categories:
   - maven
---

## 基本用法

一般常用的打包插件为maven-assembly-plugin，使用时可以在pom文件中添加如下配置，这里把配置文件dep.xml放在src/assembly下面，这个路径是assembly配置文件的标准路径。

其中executions中create-archive将assembly:single绑定到package阶段，因此在执行mvn package时会执行mvn assembly:single进行打包，日志中可以找到create-archive。

```xml
<build>
    <plugins>
      <plugin>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>2.5.3</version>
        <configuration>
          <descriptor>src/assembly/dep.xml</descriptor>
        </configuration>
        <executions>
          <execution>
            <id>create-archive</id>
            <phase>package</phase>
            <goals>
              <goal>single</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
```

典型的assembly配置文件如下：

```xml
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2 http://maven.apache.org/xsd/assembly-1.1.2.xsd">
  <id>bin</id>
  <formats>
    <format>tar.gz</format>
    <format>tar.bz2</format>
    <format>zip</format>
  </formats>
  <fileSets>
    <fileSet>
      <directory>${project.basedir}</directory>
      <outputDirectory>/</outputDirectory>
      <includes>
        <include>README*</include>
        <include>LICENSE*</include>
        <include>NOTICE*</include>
      </includes>
    </fileSet>
    <fileSet>
      <directory>${project.build.directory}</directory>
      <outputDirectory>/</outputDirectory>
      <includes>
        <include>*.jar</include>
      </includes>
    </fileSet>
    <fileSet>
      <directory>${project.build.directory}/site</directory>
      <outputDirectory>docs</outputDirectory>
    </fileSet>
  </fileSets>
</assembly>
```

配置好后，使用命令行打包：

```bash
mvn package
```

## 快捷配置

maven-assembly-plugin打包了四个长用的配置文件，可以使用descriptorRefs进行快捷配置。如：

```xml
<plugins>
      <plugin>
        <!-- NOTE: We don't need a groupId specification because the group is org.apache.maven.plugins ...which is assumed by default.
         -->
        <artifactId>maven-assembly-plugin</artifactId>
        <version>3.1.1</version>
        <configuration>
          <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
          </descriptorRefs>
        </configuration>
         <executions>
          <execution>
            <id>create-archive</id>
            <phase>package</phase>
            <goals>
              <goal>single</goal>
            </goals>
          </execution>
        </executions>
```

插件中包含了四种配置：

1. bin，打包项目的二进制版本，包含三种格式：tar.gz、tar.bz2和zip。
2. jar-with-dependencies，打包项目和二进制版本和打包的依赖，生成文件格式为jar，jar包内所有依赖都作为class存在。只支持uber-jar（不明白）。
3. src，将src的所有内容打包，包含三种格式：tar.gz、tar.bz2和zip。
4. project，将打包项目中除target下的所有内容，包含三种格式：tar.gz、tar.bz2和zip。