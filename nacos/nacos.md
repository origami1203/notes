nacos服务发现与注册中心与配置中心

nacos下载与安装

依赖

```xml
<dependencyManager>
    <dependencies>
        <dependency> 
            <groupId>com.alibaba.cloud</groupId> 
            <artifactId>spring-cloud-alibaba-dependencies</artifactId> 
            <version>2.1.0.RELEASE</version> 
            <type>pom</type> 
            <scope>import</scope> 
        </dependency>
    </dependencies>
</dependencyManager>
```

服务发现

```xml
<dependency> 
    <groupId>com.alibaba.cloud</groupId> 
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId> 
</dependency>
```

