# swagger

前后端分离项目，统一api接口

### 添加依赖

```xml
<!--Swagger-UI API文档生产工具-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.7.0</version>
</dependency>

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.7.0</version>
</dependency>
```

### swagger配置类

```java
@Configuration // 配置类
@EnableSwagger2	//开启swagger2
public class Swagger2Config {
    
    @Bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SWAGGER_2)
            .apiInfo(apiInfo())
            .select()
            
            //为当前包下controller生成API文档
            .apis(RequestHandlerSelectors
                  .basePackage("com.macro.mall.tiny.controller"))
            
            /*
             * 为有@Api注解的Controller类生成API文档
             * .apis(RequestHandlerSelectors
             * .withClassAnnotation(Api.class))
             */
            
            /*
             * 为有@ApiOperation注解的方法生成API文档
             * .apis(RequestHandlerSelectors
             * .withMethodAnnotation(ApiOperation.class))
             */
            
            // 某些路径的可以被扫描到
            .paths(PathSelectors.any())
            .build();
    }

    // 关于项目的描述
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
            .title("SwaggerUI演示")
            .description("mall-tiny")
            .contact("macro")
            .version("1.0")
            .build();
    }
}
```

访问http://localhost:8080/swagger-ui.html即可访问swagger-ui界面

### 常用注解

| 作用范围           | API                | 使用位置                         |
| ------------------ | :----------------- | :------------------------------- |
| 对象属性           | @ApiModelProperty  | 用在出入参数对象的字段上         |
| 协议集描述         | @Api               | 用于controller类上               |
| 协议描述           | @ApiOperation      | 用在controller的方法上           |
| Response集         | @ApiResponses      | 用在controller的方法上           |
| Response           | @ApiResponse       | 用在 @ApiResponses里边           |
| 非对象参数集       | @ApiImplicitParams | 用在controller的方法上           |
| 非对象参数描述     | @ApiImplicitParam  | 用在@ApiImplicitParams的方法里边 |
| 描述返回对象的意义 | @ApiModel          | 用在返回对象类上                 |

##### Api

* @Api(tags="用户相关操作")

* 用于类上，用于描述类的作用

##### ApiOperation

* @ApiOperation(value="获取客服", notes="根据cid获取客服")

* 用于方法上，描述方法的作用
* value,作用，notes详细描述

##### ApiImplicitParam

* @ApiImplicitParam(name = "cid", value = "客户id", required = true, dataType = "String")
* 方法上，对方法上的参数进行描述

```java
@ApiImplicitParams(
    @ApiImplicitParam(name = "cid", value = "客户id", required = true, dataType = "String")
    @ApiImplicitParam(name = "gender", value = "客户性别", required = true, dataType = "String")
)
```

* 用于对接收的参数进行解释

##### ApiModel

* value–字段说明
  name–重写属性名字
  dataType–重写属性类型
  required–是否必填
  example–举例说明
  hidden–隐藏

##### ApiModelProperty

* 对model上的属性进行解释
* value–字段说明 
  name–重写属性名字 
  dataType–重写属性类型 
  required–是否必填 
  example–举例说明 
  hidden–隐藏

##### ApiIgnore

* @ApiIgnore
* 忽略该类或方法