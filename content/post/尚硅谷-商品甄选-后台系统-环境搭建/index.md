+++
title = '尚硅谷 商品甄选 后台系统 环境搭建'
date = 2024-08-26T00:21:12+08:00
draft = true
+++

## 后台系统-前端工程搭建

### Element-Admin简介

**Vue3 Element Admin** 是一个免费开源的中后台模版。基于`vue3`+`ElementPlus`+`Vite`开发，是一个开箱即用的中后台系统前端解决方案，它可以帮助你快速搭建企业级中后台产品原型。

关于后端管理系统的前端工程，我们没有必要从0~1去开发，可以直接基于Vue3-Element-Admin项目模板进行二次开发，在Vue3-Element-Admin已经提供了一些基础性的功能【登录，首页布局、左侧导航菜单...】。

官网地址：https://huzhushan.gitee.io/vue3-element-admin/v1/guide/



### Element-Admin部署

具体步骤如下所示：

```shell
# Vue3-Element-Admin 要求 Node.js 版本 >= 12 ，推荐Node.js  16.x版本

# 使用git克隆项目 或者 直接下载项目
git clone https://github.com/huzhushan/vue3-element-admin.git

# 进入项目目录
cd vue3-element-admin

# 安装依赖
npm install

# 建议不要直接使用 cnpm 安装依赖，会有各种诡异的 bug。可以通过如下操作解决 npm 下载速度慢的问题
npm install --registry=https://registry.npm.taobao.org

# 启动服务
npm start
```

修改页面项目bug：

```js
# 将permission.js中的相关代码
# 原代码：import { TOKEN } from '@/store/modules/app' // TOKEN变量名
# 更改为如下代码：
import { TOKEN } from '@/pinia/modules/app' // TOKEN变量名
```

启动界面：

![image-20230826111708102](assets\image-20230826111708102.png)



### 项目目录介绍

部署好的前端工程的核心目录结构如下所示：

```java
mock					// 用于测试，模拟后端接口地址
public					// 存储公共的静态资源：图片
src						// 源代码目录，非常重要
    | api				// 提供用于请求后端接口的js文件
    | assets			// 存储静态资源：图片、css
    | components		// 存储公共组件,可重用的一些组件
    | directive			// 存储自定义的一些指令
    | hooks				// 存储自定义的钩子函数
    | i18n				// 存储用于国际化的js文件
    | layout			// 存储首页布局组件
    | pinia				// 用于进行全局状态管理
    | router			// 存储用于进行路由的js文件
    | utils				// 存储工具类的js文件
    | views				// 和路由绑定的组件
    | App.vue			// 根组件
    | default-settings.js // 默认设置的js文件
    | error-log.js		// 错误日志js文件
    | global-components.js // 全局组件的js文件
    | main.js			// 入口js文件(非常重要)
    | permission.js		// 权限相关的js文件(路由前置守卫、路由后置守卫)
vite.config.js			// vite的配置文件，可以在该配置文件中配置前端工程的端口号
```



## 后台系统-后端工程搭建

本章节会给大家介绍一下如何开发尚品甄选项目中的第一个接口。

### 项目结构说明

* 尚品甄选的项目结构如下所示：

![image-20230507105957765](assets\image-20230507105957765.png) 



* 模块说明：

spzx-parent： 尚品甄选项目的父工程，进行项目依赖的统一管理，打包方式为pom

spzx-common:  尚品甄选项目公共模块的管理模块，父工程为spzx-parent

common-util:    工具类模块，父工程为spzx-common

common-service：公共服务模块，父工程为spzx-common

spzx-model:  尚品甄选实体类模块

spzx-manager： 尚品甄选项目后台管理系统的后端服务



* 一个项目中所涉及到的实体类往往有三种：

1、封装请求参数的实体类：这种实体类在定义的时候往往会携带到dto【数据传输对象：Data Transfer Object】字样，会定义在dto包中

2、与数据库对应的实体类：这种实体类往往和数据表名称保证一致，会定义在domain、entity、pojo包中

3、封装响应结果的实体类：这种实体类在定义的时候往往会携带到vo【视图对象：View Object】字样，会定义在vo包中



### 模块依赖说明

模块之间的依赖关系如下图所示：

![image-20230507113452030](assets\image-20230507113452030.png) 

common-service会依赖：common-util、spzx-common

spzx-manager会依赖：common-service



###  环境说明

本次项目开发的时候所使用的软件环境版本如下所示：

| 软件名称     | 版本说明 |
| ------------ | -------- |
| jdk          | jdk17    |
| spring boot  | 3.0.5    |
| spring cloud | 2022.0.2 |
| redis        | 7.0.10   |
| mysql        | 8.0.30   |
| idea         | 2022.2.2 |
|              |          |



### 项目模块创建

![image-20230727201919351](assets\image-20230727201919351.png)

#### spzx-parent

创建一个maven项目，导入如下依赖：

```xml
<!-- 指定父工程 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.0.5</version>
</parent>

<properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <mysql.verison>8.0.30</mysql.verison>
    <fastjson.version>2.0.21</fastjson.version>
    <lombok.version>1.18.20</lombok.version>
    <mybatis.version>3.0.1</mybatis.version>
</properties>

<!-- 管理依赖，版本锁定 -->
<dependencyManagement>
    <dependencies>
        <!-- mybatis和spring boot整合的起步依赖 -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis.version}</version>
        </dependency>

        <!-- mysql驱动 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>

        <!--fastjson-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>${fastjson.version}</version>
        </dependency>

        <!-- lombok依赖 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

注意：

1、设置项目的字符集为utf-8

2、删除src目录



#### spzx-common

在spzx-parent下面创建该子模块，并导入如下依赖：

```xml
<dependencies>
    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>5.5.2</version>
    </dependency>
</dependencies>
```



#### common-util

在spzx-common下面创建该子模块，并导入如下依赖：

```xml
<dependencies>
    <!-- fastjson依赖 -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
    </dependency>

    <dependency>
        <groupId>com.atguigu</groupId>
        <artifactId>spzx-model</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>

    <dependency>
        <groupId>io.minio</groupId>
        <artifactId>minio</artifactId>
        <version>8.5.2</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <scope>provided </scope>
    </dependency>
</dependencies>
```



#### common-service

在spzx-common下面创建该子模块，并导入如下依赖：

```xml
<dependencies>

    <!-- common-util模块 -->
    <dependency>
        <groupId>com.atguigu</groupId>
        <artifactId>common-util</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>

    <!-- spzx-model模块 -->
    <dependency>
        <groupId>com.atguigu</groupId>
        <artifactId>spzx-model</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>

    <!-- spring boot web开发所需要的起步依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
        <scope>provided</scope>
    </dependency>

</dependencies>
```



#### spzx-model

* 在spzx-parent下面创建该子模块，并导入如下依赖：

```xml
<dependencies>

    <!-- lombok的依赖 -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>

    <dependency>
        <groupId>com.github.xiaoymin</groupId>
        <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
        <version>4.1.0</version>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>easyexcel</artifactId>
        <version>3.1.0</version>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
    </dependency>
</dependencies>
```

* 在资料中复制相关实体类

![image-20230727202740833](assets\image-20230727202740833.png)



#### spzx-manager

在spzx-parent下面创建该子模块，并导入如下依赖：

```xml
<dependencies>

    <!-- spring boot web开发所需要的起步依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- redis的起步依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>

    <!-- mybatis的起步依赖 -->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
    </dependency>

    <!-- mysql驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>

    <!-- common-service依赖 -->
    <dependency>
        <groupId>com.atguigu</groupId>
        <artifactId>common-service</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
    </dependency>

    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper-spring-boot-starter</artifactId>
        <version>1.4.3</version>
    </dependency>
</dependencies>
```



### 数据库环境准备

#### 安装Mysql数据库

本地安装mysql数据库使用的是docker安装，对应的步骤如下所示：

* 部署mysql

开发阶段也可以连接本地mysql服务

```shell
# 拉取镜像
docker pull mysql:8.0.30

# 创建容器
docker run -d --name mysql -p 3306:3306 -v mysql_data:/var/lib/mysql -v mysql_conf:/etc/mysql --restart=always --privileged=true -e MYSQL_ROOT_PASSWORD=1234 mysql:8.0.30
```

docker安装完成mysql8，如果使用sqlyog或者navite连接，需要修改密码加密规则，因为低版本客户端工具不支持mysql8最新的加密规则。如果使用客户端连接，需要修改：

* docker exec 进入mysql容器

* mysql -uroot -p 登录你的 MySQL 数据库，然后 执行这条SQL：

```sql
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '1234';
```

然后再重新配置SQLyog的连接，重新填写密码，则可连接成功了。



####  初始化数据库

具体步骤如下所示：

1、使用工具连接到mysql中，创建数据库db_spzx

2、导入课程资料中的db_spzx.sql文件



#### 表结构介绍

登录功能相关的表：sys_user【后台管理系统用户表】

```sql
CREATE TABLE `sys_user` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '会员id',
  `username` varchar(20) NOT NULL DEFAULT '' COMMENT '用户名',
  `password` varchar(32) NOT NULL DEFAULT '' COMMENT '密码',
  `name` varchar(50) DEFAULT NULL COMMENT '姓名',
  `phone` varchar(11) DEFAULT NULL COMMENT '手机',
  `avatar` varchar(255) DEFAULT NULL COMMENT '头像',
  `description` varchar(255) DEFAULT NULL COMMENT '描述',
  `status` tinyint NOT NULL DEFAULT '1' COMMENT '状态（1：正常 0：停用）',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `is_deleted` tinyint NOT NULL DEFAULT '0' COMMENT '删除标记（0:不可用 1:可用）',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='用户表';
```



#### 部署Redis

* 使用docker部署Redis，具体的操作如下所示：

开发阶段也可以连接本地redis服务

```shell
#1 拉取镜像
docker pull redis:7.0.10

#2 在宿主机的 /var/lib/docker/volumes/redis-config/_data/目录下创建一个redis的配置文件，
vim redis.conf
# 内容如下所示
#开启持久化
appendonly yes
port 6379
# requirepass 1234
bind 0.0.0.0

#3 如果/var/lib/docker/volumes没有redis-config，创建数据卷 
docker volume create redis-config

#4 创建容器
docker run -d -p 6379:6379 --restart=always \
-v redis-config:/etc/redis/config \
-v redis-data:/data \
--name redis redis \
redis-server /etc/redis/config/redis.conf
```



### 统一结果实体类

#### 响应结果分析

通过对前端登录功能的源码查看，可以看到登录请求成功以后，前端需要从返回结果中解析出来三部分的数据：

```javascript
// code:自定义的业务状态码，前端会根据具体的业务状态码给出不同的处理。比如：200表示成功、非200都是失败
// message：响应消息。比如：登录成功、用户名或者密码错误、用户无权限访问
// data：后端返回给前端的业务数据
const { code, data, message } = await Login(state.model)
```



#### Result实体类

针对上述的三部分的数据，我们可以定义一个实体类来进行封装，后期尚品甄选项目中所有的接口的返回值统一都会定义为Result。

* 在spzx-model模块，找到Result实体类，定义如下所示：

```java
// com.atguigu.spzx.model.vo.common
@Data
public class Result<T> {

    //返回码
    private Integer code;

    //返回消息
    private String message;

    //返回数据
    private T data;

    // 私有化构造
    private Result() {}

    // 返回数据
    public static <T> Result<T> build(T body, Integer code, String message) {
        Result<T> result = new Result<>();
        result.setData(body);
        result.setCode(code);
        result.setMessage(message);
        return result;
    }
}
```



为了简化Result对象的构造，可以定义一个枚举类，在该枚举类中定义对应的枚举项来封装code、message的信息，如下所示：

```java
// com.atguigu.spzx.model.vo.common
@Getter // 提供获取属性值的getter方法
public enum ResultCodeEnum {

    SUCCESS(200 , "操作成功") ,
    LOGIN_ERROR(201 , "用户名或者密码错误");

    private Integer code ;      // 业务状态码
    private String message ;    // 响应消息

    private ResultCodeEnum(Integer code , String message) {
        this.code = code ;
        this.message = message ;
    }
}
```



在Result类中扩展获取Result对象的方法

```java
// 通过枚举构造Result对象
public static <T> Result build(T body , ResultCodeEnum resultCodeEnum) {
    return build(body , resultCodeEnum.getCode() , resultCodeEnum.getMessage()) ;
}
```



### 整合Swagger

####  Swagger

* Swagger是一种基于OpenAPI规范的API文档生成工具，它可以根据Java代码中的注解自动生成API接口文档，并提供UI界面进行在线测试和调试。

* Swagger为开发人员提供了更加方便、直观的API管理方式，有助于提升API的可读性和可维护性。

* Swagger的主要特点包括：

1、自动生成API文档：通过在Java代码中添加Swagger注解，Swagger能够自动地解析API接口的参数、响应等信息，并生成相应的API文档。

2、在线测试接口：Swagger提供了UI界面，可以方便地进行API接口的测试和调试，无需单独使用HTTP客户端来测试接口。

3、支持多种语言和框架：Swagger不仅支持Java语言和Spring框架，还支持多种其他语言和框架，如PHP、Python、Go等。

4、扩展性强：Swagger提供了多种扩展机制和插件，可以满足各种项目的需要，如集成OAuth2、自定义UI等。

Swagger提供的UI界面相比于另外一款Api文档生成工具**Knife4j**较为简陋。



####  Knife4j

##### Knife4j简介

官方文档：https://doc.xiaominfo.com/

* Knife4j是一种基于Swagger构建的增强工具，它在Swagger的基础上增加了更多的功能和扩展，提供了更加丰富的API文档管理功能。相比于原版Swagger，Knife4j的主要特点包括：

1、更加美观的UI界面：Knife4j通过对Swagger UI的修改和优化，实现了更加美观、易用的UI界面，提升了开发人员的体验感。

2、支持多种注解配置方式：除了支持原版Swagger的注解配置方式外，Knife4j还提供了其他几种注解配置方式，方便开发人员进行不同场景下的配置。

3、提供多种插件扩展：Knife4j提供了多种插件扩展，如knife4j-auth、knife4j-rate-limiter等，可以满足不同项目的需求。

4、集成Spring Boot Starter：Knife4j发布了spring-boot-starter-knife4j，可以实现更加便捷的集成，并支持配置文件中的动态属性调整。

##### Knife4j使用

官方文档使用地址：https://doc.xiaominfo.com/docs/quick-start

具体的步骤：

在common-service模块中添加knife4j所需要的配置类

```java
@Configuration
public class Knife4jConfig {

    @Bean
    public GroupedOpenApi adminApi() {      // 创建了一个api接口的分组
        return GroupedOpenApi.builder()
                .group("admin-api")         // 分组名称
                .pathsToMatch("/admin/**")  // 接口请求路径规则
                .build();
    }

    /***
     * @description 自定义接口信息
     */
    @Bean
    public OpenAPI customOpenAPI() {

        return new OpenAPI()
                 .info(new Info()
                 .title("尚品甑选API接口文档")
                 .version("1.0")
                 .description("尚品甑选API接口文档")
                 .contact(new Contact().name("atguigu"))); // 设定作者
    }

}
```

启动项目就可以访问到knife4j所生成的接口文档了。访问地址：**http://项目ip:端口号/doc.html**

![image-20230509194447547](assets\image-20230509194447547.png) 

##### 常见注解

在Knife4j中也提供了一些注解，让我们对接口加以说明，常见的注解如下所示：

```shell
@Tag： 用在controller类上，对controller进行说明
@Operation: 用在controller接口方法上对接口进行描述
@Parameters：用在controller接口方法上对单个参数进行描述
@Schema： 用在实体类和实体类属性上，对实体类以及实体类属性进行描述
```

举例说明：

```java
@Tag(name = "首页接口")
public class IndexController {


    @Operation(summary = "用户登录")
    public Result<LoginVo> login(@RequestBody LoginDto loginDto) {
        ...
    }

    @Operation(summary = "用户退出")
    @Parameters(value = {
            @Parameter(name = "令牌参数" , required = true)
    })
    @GetMapping(value = "/logout")
    public Result logout(@RequestHeader(value = "token") String token) {
        ...
    }

}


@Data
@Schema(description = "用户登录请求参数")
public class LoginDto {

    @Schema(description = "用户名")
    private String userName ;

    @Schema(description = "密码")
    private String password ;

    @Schema(description = "提交验证码")
    private String captcha ;

    @Schema(description = "验证码key")
    private String codeKey ;

}
```



#### 导出

Knife4j生成的接口文档是一个在线的接口文档使用起来不是特别的方便，当然Knife4j也支持离线接口文档，并且支持导出json格式的数据，如下所示：

![image-20230509203546813](assets\image-20230509203546813.png) 

