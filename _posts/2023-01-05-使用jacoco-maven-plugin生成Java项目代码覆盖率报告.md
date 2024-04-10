---
title: 使用jacoco-maven-plugin生成Java项目代码覆盖率报告   
date: 2023-01-05 18:39:00 +0800  
categories: [CNBlogs]   
tags: [java,maven]  
---
<a href="https://www.cnblogs.com/hiver/p/17028615.html" target="_blank">点击查看原文</a>

## 1. 参考资料
- [使用JaCoCo Maven插件创建代码覆盖率报告](https://juejin.cn/post/6844903999473188877)  
- [maven单测生成覆盖率报告---Jacoco的使用](https://www.cnblogs.com/fnlingnzb-learner/p/10637802.html)  
- [Maven Failsafe Plugin/ Skipping Tests](https://maven.apache.org/surefire/maven-failsafe-plugin/examples/skipping-tests.html#)  

## 2. 利用[Spring initializr](https://start.spring.io/)生成代码并添加Unit test及Integraiton test类
[https://github.com/hivsuper/study/tree/master/study-java11](https://github.com/hivsuper/study/tree/master/study-java11)
## 3. 添加maven-surefire-plugin,maven-failsafe-plugin及jacoco-maven-plugin配置
<details><summary markdown="span">点击查看代码</summary>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.7.7</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>org.lxp</groupId>
	<artifactId>study-java11</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>study-java11</name>
	<description>Demo project for Java 11</description>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
    <properties>
        <java.version>11</java.version>
        <maven.compiler.release>${java.version}</maven.compiler.release>
        <jacoco-maven-plugin.version>0.8.8</jacoco-maven-plugin.version>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.10.1</version>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <!-- Unit test with Maven Surefire plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>${maven-surefire-plugin.version}</version>
                <configuration>
                    <parallel>method</parallel>
                    <threadCountMethods>10</threadCountMethods>
                    <excludes>
                        <exclude>**/*IT.java</exclude>
                        <exclude>**/IT*.java</exclude>
                    </excludes>
                    <systemPropertyVariables>
                        <listener>org.sonar.java.jacoco.JUnitListener</listener>
                    </systemPropertyVariables>
                </configuration>
            </plugin>
            <!-- Integration test with Maven failsafe plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>${maven-failsafe-plugin.version}</version>
                <executions>
                    <execution>
                        <id>integration-tests</id>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                        <configuration>
                            <excludes>
                                <exclude>**/Test*.java</exclude>
                                <exclude>**/*Test.java</exclude>
                            </excludes>
                            <additionalClasspathElements>
                                <additionalClasspathElement>
                                    ${project.build.directory}/${project.artifactId}-${project.version}.jar.original
                                </additionalClasspathElement>
                            </additionalClasspathElements>
                            <systemPropertyVariables>
                                <listener>org.sonar.java.jacoco.JUnitListener</listener>
                            </systemPropertyVariables>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

    <profiles>
        <profile>
            <id>coverage</id>
            <activation>
                <property>
                    <name>coverage</name>
                    <value>true</value>
                </property>
            </activation>
            <properties>
                <test.listeners>org.sonar.java.jacoco.JUnitListener</test.listeners>
            </properties>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.jacoco</groupId>
                        <artifactId>jacoco-maven-plugin</artifactId>
                        <version>${jacoco-maven-plugin.version}</version>
                        <executions>
                            <execution>
                                <id>pre-unit-test</id>
                                <goals>
                                    <goal>prepare-agent</goal>
                                </goals>
                            </execution>
                            <execution>
                                <id>report</id>
                                <goals>
                                    <goal>report</goal>
                                </goals>
                                <configuration>
                                    <dataFile>target/jacoco.exec</dataFile>
                                    <outputDirectory>target/jacoco-ut</outputDirectory>
                                </configuration>
                            </execution>
                            <execution>
                                <id>pre-integration-test</id>
                                <phase>pre-integration-test</phase>
                                <goals>
                                    <goal>prepare-agent-integration</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
</project>

```
</details>

### 3.1 使用mvn verify + Unit test生成代码覆盖率
Maven命令及输出如下
```
mvn clean verify -P coverage -DskipITs=true -Dmaven.test.skip=false -Dskip.repackage=true -f pom.xml
```
<img src="/assets/img/202301/571584-20230105180150744-234625221.png" width="800" />

如下图所示，Integration test都被跳过
<img src="/assets/img/202301/571584-20230105180442371-702530813.png" width="800" />

打开target/jacoco-ut中的报告
![](/assets/img/202301/571584-20230105180620761-254060611.png)

### 3.2 使用mvn verify + Integration test生成代码覆盖率
修改jacoco-maven-plugin配置
```
<dataFile>target/jacoco-it.exec</dataFile>
```
Maven命令及输出如下
```
mvn clean verify -P coverage -DskipITs=false -Dmaven.test.skip=false -Dskip.repackage=true -f pom.xml
```
<img src="/assets/img/202301/571584-20230105181532021-1336864426.png" width="800" />
如下图所示，Integration test都包括
<img src="/assets/img/202301/571584-20230105184700358-2077180040.png" width="800" />

### 3.3 使用mvn test + Unit test生成代码覆盖率
修改jacoco-maven-plugin配置
```xml
                    <plugin>
                        <groupId>org.jacoco</groupId>
                        <artifactId>jacoco-maven-plugin</artifactId>
                        <version>${jacoco-maven-plugin.version}</version>
                        <executions>
                            <execution>
                                <id>pre-unit-test</id>
                                <goals>
                                    <goal>prepare-agent</goal>
                                </goals>
                            </execution>
                            <execution>
                                <id>report</id>
                                <goals>
                                    <goal>report</goal>
                                </goals>
                                <configuration>
                                    <dataFile>target/jacoco.exec</dataFile>
                                    <outputDirectory>target/jacoco-ut</outputDirectory>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
```
Maven命令及输出如下
```
clean test -P coverage org.jacoco:jacoco-maven-plugin:report -Dmaven.test.skip=false -Dskip.repackage=true -f pom.xml
```
<img src="/assets/img/202301/571584-20230105182846460-466961631.png" width="800" />

<img src="/assets/img/202301/571584-20230105182951795-1214187590.png" width="800" />

在target\site\jacoco中打开报告
![](/assets/img/202301/571584-20230105183740682-408458613.png)
