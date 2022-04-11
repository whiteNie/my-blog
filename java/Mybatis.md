## 环境准备

* JDK 1.8
* MySql 5.7
* Maven 3.6.0
* IntelliJ IDEA 2018.3.3

## 简介

* [官方文档](https://mybatis.org/mybatis-3/zh/index.html)

* 学习版本：

```java
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.2</version>
</dependency>

```

## 持久层

Dao 、Service、Controller

* 完成持久化工作的代码块
* 层界限十分明显

## 第一个 Mybatis 项目

### 搭建数据库

```mysql
-- 创建用户表
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
	`id` INT(20) NOT NULL PRIMARY KEY,
	`name` VARCHAR(30) DEFAULT NULL,
	`pwd` VARCHAR(30) DEFAULT NULL
)ENGINE=INNODB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='用户表'; 
-- 插入数据到用户表
INSERT INTO `user`(`id`, `name`, `pwd`) VALUES
(1, '张三', '123456'),
(2, '李四', '123456'),
(3, '王五', '123456');
```

### 搭建项目

* 新建一个普通的 *maven* 项目

* 删除 *src* 目录，此时就相当于是一个父工程

  * pom.xml

    ```java
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
    
        <!-- 我是一个父工程 -->
        <groupId>com.nip</groupId>
        <artifactId>Mybatis-Study</artifactId>
        <packaging>pom</packaging>
        <version>1.0-SNAPSHOT</version>
        <modules>
            <module>mybatis-01</module>
        </modules>
    
        <!-- 导入依赖 -->
        <dependencies>
            <!--Mysql-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.47</version>
            </dependency>
    
            <!-- Mybatis -->
            <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>3.5.2</version>
            </dependency>
    
            <!-- lombok 运行时生产get set 等方法支持 -->
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>1.16.16</version>
            </dependency>
    
            <!-- junit -->
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.12</version>
            </dependency>
        </dependencies>
    
    
    </project>
    ```

    ### 创建一个模块

    * 创建 *mybatis* 的核心配置文件
    * 编写 *mybatis* 工具类

    





