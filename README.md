

写在前面：在阅读本文前，请前拥有dubbo基础知识，springboot知识

dubbo官网： http://dubbo.apache.org
dubbo github  源码地址：https://github.com/apache/incubator-dubbo
dubbo 运维项目源码地址：https://github.com/apache/incubator-dubbo-ops

本文项目GitHub： https://github.com/Blankwhiter/dubbo-spring-boot-starter-test
源码对应版本1.0.1：    https://github.com/Blankwhiter/dubbo-spring-boot-starter-test/archive/1.0.1.zip
# 第一步 搭建zookeeper环境
在centos窗口中，执行如下命令，拉取镜像，并启动zookeeper容器
```bash
docker pull zookeeper
docker run -d -v /home/docker/zookeeperhost/zookeeperDataDir:/data -v /home/docker/zookeeperhost/zookeeperDataLogDir:/datalog  -e ZOO_MY_ID=1 -e ZOO_SERVERS='server.1=125.77.116.145:2888:3888'  -p 2182:2181 -p 2888:2888 -p 3888:3888  --name zookeeper --privileged zookeeper
```
*注：1.zookeeper默认连接端口是2181 但本文测试用例时由于被其他程序占走，故使用2182。  2.读者请自行创建映射目录zookeeperDataDir | zookeeperDataLogDir*

# 第二步 springboot集成dubbo
#### 1.项目目录机构
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190328165910927.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbG9uZ2h1YW5nMTU3NDA1,size_16,color_FFFFFF,t_70)

说明： 
- 1.api目录：存放消费者与提供者调用的service接口
- 2.consumer目录：消费者目录 调用提供者远程提供的接口实现
- 3.provider目录：提供者目录 提供给消费者接口实现

读者请自行创建项目目录（创建空项目，然后在空项目中新建三个module）


项目案例说明：业务假设场景=》 产品购买消费金额（consumer）同时并返回所有消费的总金额（需要调用到provider项目中服务实现）。

#### 2.代码编写
2.1 api目录 接口编写
2.1.1.在com.dubbo.api.service（读者请自行创建,下同，package创建将不一一赘述）包下创建**CostService.java**
```java
package com.dubbo.api.service;

public interface CostService {
    /**
     * 成本增加接口
     * @param cost
     * @return
     */
    Integer add(int cost);
}

```

2.1.2.**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.dubbo</groupId>
    <artifactId>api</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>api</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.0.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>
```

2.2 consumer目录 web访问、接口调用以及dubbo配置编写
2.2.1.引入 dubbo-spring-boot-starter 以及 上述的api模块
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.dubbo</groupId>
    <artifactId>consumer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>consumer</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.0.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

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

        <!--引入api模块-->
        <dependency>
            <groupId>com.dubbo</groupId>
            <artifactId>api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
            <scope>compile</scope>
        </dependency>

        <!--引入dubbo环境-->
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>0.2.0</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>

```


2.2.2.在resources目录下 创建**application.yml**,并编写dubbo配置
```xml
dubbo:
  application:
    name:  dubbo-consumer
  registry:
    address: 125.77.116.145:2182
    # 读者请换成自己的zookeeperip
    protocol: zookeeper
    check: false
  monitor:
    protocol: register
  consumer:
    check:  false
    timeout: 3000

server:
  port: 8062

```

2.2.3.使用 **@EnableDubbo**  注解开启dubbo
**ConsumerApplication.java** 启动类
```java
package com.dubbo.consumer;

import com.alibaba.dubbo.config.spring.context.annotation.EnableDubbo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableDubbo
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}


```

2.2.4.编写产品service接口 **ProductService.java**
```java
package com.dubbo.consumer.service;

public interface ProductService {

    /**
     * 获得总消费
     * @param a
     * @return
     */
    Integer getCost(int a);

}

```


2.2.5.编写产品接口的实现，并调用远程服务CostService 。 **ProductServiceImpl.java**
```java
package com.dubbo.consumer.service.impl;

import com.alibaba.dubbo.config.annotation.Reference;
import com.dubbo.api.service.CostService;
import com.dubbo.consumer.service.ProductService;
import org.springframework.stereotype.Service;

/**
 * 产品service
 */
@Service
public class ProductServiceImpl implements ProductService {


    /**
     * 使用dubbo的注解 com.alibaba.dubbo.config.annotation.Reference。进行远程调用service
     */
    @Reference
    private CostService costService;

    @Override
    public Integer getCost(int a) {
        return costService.add(a);
    }
}

```
2.2.6.编写访问类，**ProductController.java**
```java
package com.dubbo.consumer.controller;

import com.dubbo.consumer.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 产品controller
 */
@RestController
public class ProductController {


    @Autowired
    private ProductService productService;

    /**
     * 添加完 返回总共消费
     * @param a
     * @return
     */
    @RequestMapping("/add")
    public String getCost(int a){
        return "该产品总共消费 ："+productService.getCost(a);
    }
}

```

2.3 provider目录 api接口实现以及dubbo配置
2.3.1.引入 dubbo-spring-boot-starter 以及 上述的api模块 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.dubbo</groupId>
    <artifactId>provider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>provider</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.0.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

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
        <!--引入api-->
        <dependency>
            <groupId>com.dubbo</groupId>
            <artifactId>api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
            <scope>compile</scope>
        </dependency>
        <!--引入dubbo环境-->
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>0.2.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>
```

2.3.2.在resources目录下 创建**application.yml**,并编写dubbo配置
```xml
dubbo:
  application:
    name: dubbo-provider
  registry:
    address: 125.77.116.145:2182
    # 读者请自行更改zookeeper地址
    protocol: zookeeper
    check: false
  protocol:
    name: dubbo
    port: 30003
  monitor:
    protocol: register
  consumer:
    check: false
    timeout: 3000

server:
  port: 8061


```
2.3.3.使用 **@EnableDubbo** 注解开启dubbo
**ConsumerApplication.java** 启动类
```java
package com.dubbo.provider;

import com.alibaba.dubbo.config.spring.context.annotation.EnableDubbo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableDubbo
public class ProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}

```

2.3.3.编写CostService服务实现 **CostServiceImpl.java**
```java
package com.dubbo.provider.service.impl;

import com.alibaba.dubbo.config.annotation.Service;
import com.dubbo.api.service.CostService;

/**
 * 花费服务
 */
@Service
public class CostServiceImpl implements CostService {

    /**
     * 假设之前总花费了100
     */
    private final Integer totalCost = 1000;

    /**
     * 之前总和 加上 最近一笔
     * @param cost
     * @return
     */
    @Override
    public Integer add(int cost) {
        return totalCost + cost;
    }
}

```


# 第三步 测试dubbo远程服务调用
编写第二步代码完成后 ，启动consumer项目，以及provider项目
在浏览器中访问  http://localhost:8062/add?a=100
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190401150741900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbG9uZ2h1YW5nMTU3NDA1,size_16,color_FFFFFF,t_70)


出现如上结果即为调用成功

# 第四步 dubbo管理平台
dubbo运维旧版地址： https://github.com/apache/incubator-dubbo-ops/tree/master

1.本文将运维项目代码下载放于 D:\learnplace\incubator-dubbo-ops

2.**这里需要修改一个配置D:\learnplace\incubator-dubbo-ops\dubbo-admin\src\main\resources\application.properties**
```xml
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

server.port=8001
#服务器端口 server.port
spring.velocity.cache=false
spring.velocity.charset=UTF-8
spring.velocity.layout-url=/templates/default.vm
spring.messages.fallback-to-system-locale=false
spring.messages.basename=i18n/message
spring.root.password=root
spring.guest.password=guest
#访问的密码配置 spring.root.password  spring.guest.password
#dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.registry.address=zookeeper://125.77.116.145:2182
#zookeeper地址
```

3.在D:\learnplace\incubator-dubbo-ops\dubbo-admin目录下 ，进入cmd窗口执行
**mvn claen package** 打包项目，
4.然后进入D:\learnplace\incubator-dubbo-ops\dubbo-admin\target ，进入cmd窗口执行
**java -jar dubbo-admin-0.0.1-SNAPSHOT.jar** 运行项目
5.启动成功后 浏览器访问http://localhost:8001 输入账号：root /  密码：root 即可。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190401150837771.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbG9uZ2h1YW5nMTU3NDA1,size_16,color_FFFFFF,t_70)


### 附录：
1.各个软件版本对应
<table width="100%">
<thead>
<tr>
<th>versions</th>
<th>Java</th>
<th>Spring Boot</th>
<th>Dubbo</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>0.2.0</code></td>
<td>1.8+</td>
<td><code>2.0.x</code></td>
<td><code>2.6.2</code> +</td>
</tr>
<tr>
<td><code>0.1.1</code></td>
<td>1.7+</td>
<td><code>1.5.x</code></td>
<td><code>2.6.2</code> +</td>
</tr>
</tbody>
</table>
