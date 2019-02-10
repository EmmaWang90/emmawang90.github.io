---
title: maven插件入门
date: 2019-02-09 12:05:56
tags:
    - maven
    - plugin
categories:
    - maven
---

# 必须配置

maven的本质是一个执行插件的框架，所有的工作都是由插件完成的。插件有两种类型：

- 执行插件build plugins，在执行build时被调用，应该被配置在pom文件的build元素中。
- 报告插件reporting plubins，在生成站点时被调用，应该被配置在pom文件的reporting元素中。因为reporting plugin的执行结果是站点的一部分，所以需要被国际化。



所有的插件都需要被设置groupId`, `artifactId` 和 `version。推荐对每个插件设置version。

最佳实践：在parent pom的build的pluginManagement中声明所有插件的version。



# 通用配置

## 背景

在每一个maven插件中，都有一个或多个Mojo类，Mojo是maven plain old java ojbect，继承org.apache.maven.plugin.AbstractMojo，这个接口保证插件可以被正确执行、输出日志、与控制台交互。

mojo中包含任意多个字段或字段的setters，这些字段可以被命令行、配置文件进行配置。插件的goal和phase也在Mojo类的注解中标志。

比如：

```Java
/**
 * @goal query
 */
public class MyQueryMojo
    extends AbstractMojo
{
    /**
     * @parameter expression="${query.url}"
     */
    private String url;
 
    /**
     * @parameter default-value="60"
     */
    private int timeout;
 
    /**
     * @parameter
     */
    private String[] options;
 
    public void execute()
        throws MojoExecutionException
    {
        ...
    }
}
```

## 一般配置

### 在pom文件中配置

在pom文件的插件中，通过configuration元素对插件进行配置，比如：

```xml
<project>
  ...
  <build>
    <plugins>
      <plugin>
        <artifactId>maven-myquery-plugin</artifactId>
        <version>1.0</version>
        <configuration>
          <url>http://www.foobar.com/query</url>
          <timeout>10</timeout>
          <options>
            <option>one</option>
            <option>two</option>
            <option>three</option>
          </options>
        </configuration>
      </plugin>
    </plugins>
  </build>
  ...
</project>
```

其中configuration的子元素与Mojo类中字段的名称相同。

### 在系统变量中配置

通过Mojo类字段上的@parameter expression，可在命令行中添加配置，如：

```bash
mvn myquery:query -Dquery.url=http://maven.apache.org
```

注意，在使用命令行配置前，到插件中确认配置的名称。可以使用help goal查询参数和类型，比如：

```bash
mvn javadoc:help -Ddetail -Dgoal=javadoc
```



## 配置Build插件

### 使用executions

使用executions标签，最常用于配置作用于build生命周期某个阶段的插件，比如：

```xml
<project>
  ...
  <build>
    <plugins>
      <plugin>
        <artifactId>maven-myquery-plugin</artifactId>
        <version>1.0</version>
        <executions>
          <execution>
            <id>execution1</id>
            <phase>test</phase>
            <configuration>
              <url>http://www.foo.com/query</url>
              <timeout>10</timeout>
              <options>
                <option>one</option>
                <option>two</option>
                <option>three</option>
              </options>
            </configuration>
            <goals>
              <goal>query</goal>
            </goals>
          </execution>
          <execution>
            <id>execution2</id>
            <configuration>
              <url>http://www.bar.com/query</url>
              <timeout>15</timeout>
              <options>
                <option>four</option>
                <option>five</option>
                <option>six</option>
              </options>
            </configuration>
            <goals>
              <goal>query</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  ...
</project>
```

这里有两个execution配置。其中，executions绑定到test阶段，execution2没有配置phase，使用默认阶段。如果这个goal有默认阶段，则插件会被执行，否则不会被执行。

注意，在一个pom文件的某个插件中，execution的id必须是独一无二的，在不同的pom文件的同一插件的exexution的id可以不一样，这些配置会被合并。这个规则同样适用于profile中定义的executions。

如果一个插件中的两个execution被绑定到不同的phase，如下所示，这个插件会被执行多次。

```xml
<project>
  ...
  <build>
    <plugins>
      <plugin>
        ...
        <executions>
          <execution>
            <id>execution1</id>
            <phase>test</phase>
            ...
          </execution>
          <execution>
            <id>execution2</id>
            <phase>install</phase>
            <configuration>
              <url>http://www.bar.com/query</url>
              <timeout>15</timeout>
              <options>
                <option>four</option>
                <option>five</option>
                <option>six</option>
              </options>
            </configuration>
            <goals>
              <goal>query</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  ...
</project>
```

maven 3.3.1版本之后，可以在命令行中直接对某个execution进行配置：

```bash
mvn myqyeryplugin:queryMojo@execution1
```

### 使用dependencies

可以配置build插件的依赖，以使用更新版本的依赖。

比如maven antrun插件1.2依赖ant 1.6.5，如果想使用其他版本，可以这样配置:

```xml
<project>
  ...
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-antrun-plugin</artifactId>
        <version>1.2</version>
        ...
        <dependencies>
          <dependency>
            <groupId>org.apache.ant</groupId>
            <artifactId>ant</artifactId>
            <version>1.7.1</version>
          </dependency>
          <dependency>
            <groupId>org.apache.ant</groupId>
            <artifactId>ant-launcher</artifactId>
            <version>1.7.1</version>
          </dependency>
         </dependencies>
      </plugin>
    </plugins>
  </build>
  ...
</project>
```

### 使用inherited

默认情况，插件配置会被继承到子pom中，为了打破这种继承关系，可以使用inherited。

```xml
<project>
  ...
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-antrun-plugin</artifactId>
        <version>1.2</version>
        <inherited>false</inherited>
        ...
      </plugin>
    </plugins>
  </build>
  ...
</project>
```

