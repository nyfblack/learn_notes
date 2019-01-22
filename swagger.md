# 1.yaml

​	Yaml文件是统一定义接口的文件，其中主要包含模型定义(Definitions Object)和接口定义(Paths Object)，模型定义指出了微服务中使用的数据对象(POJO)，接口定义指出了微服务对外调用接口。该文件采用Swagger规范，并在该规范基础上做了很小的修改。

## 1.1 yaml文件结构

```yaml
# 协议的版本
swagger: '2.0' 
# 服务基本信息定义,包括服务名称，版本，服务描述，服务Owner
info:
  # 服务接口版本
  version: v1
  # 定义服务名称
  title: Service
  # 描述服务提供的功能、限制、注意事项等
  description: 
  # 声明服务维护人员信息  
  contact:
    name: username
    email: username@email.com
# 服务支持的访问协议http(s), 当前CloudSOP都是https的
schemes:
  - https 
# Base PATH, 完整的访问路径为 basePath + relativePath，relativePath
  在paths定义
basePath: /rest
# 定义对外调用的接口
paths:
……
# 定义数据对象(POJO对象)
definitions
……

```

## 1.2 paths定义 

paths下每个对外接口定义主要包含如下信息：

```
relativePath路径
请求方式(get、post、delete、put中的一种)
    对接口描述信息
    接口所在的类名称
    接口的名称
    接口输入参数的定义
    接口输出参数的定义
```

---

```yaml
# GET方法定义
  # relativePath路径
  /service/v1/getBookInfo:
    get:  # get、post、delete、put中的一种
      #对接口描述信息
      summary: '根据书名查询book信息'  #综述
      description: '根据条件查询book信息' #具体描述
      # 用于代码生成，声明方法所在类名称
      tags:
        - Service    
      # 用于代码生成，声明方法名称
      operationId: getBookInfo 
      # 声明接口返回值是非基本类型，具体是哪种类型在后续输出参数中定义， 
      # 如果返回值是基本类型，则去掉此部分
      produces:
        - application/json  # 声明返回Json格式数据，非基本类型
      # 声明接口输入参数中包含非基本类型，具体是哪种类型在后续输出参数中
      # 定义，如果输入参数都是基本类型，则去掉此部分
      consumes:
        - application/json
      parameters: #接口输入参数定义
        #参数1 
        - name: bookName  #名称
          #参数类型，常用的类型有body,path,query等类型，指明了调用接口是该
          #参数值传递格式，其中path,query用于基本类型，可用于get、post、
          #delete、put方法，body用于非基本类型，只能用于post、delete、put方法
          in: query  
          #参数描述
          description: 书名 
          # 默认为false，指明调用该方法时是否必须给该输入参数赋值，true表示必须赋值，
          # false为可以不赋值，系统会给默认值，当in为path时，该设置必须为true
          required: true 
          # 指明该数据类型          
          type: string
        #可以继续定义其他参数，格式与参数1格式类似 
      responses: #接口输出参数定义
        200:  #返回编码
          description: Get Service Information #返回值描述
          schema: # 声明返回值类型
            type: string

```

### 1.2.1 参数的说明

| 请求方式 | path            |
| -------- | --------------- |
| get      | /url/{资源标识} |
| post     | /url            |
| put      | /url/{资源标识} |
| delete   | /url/{资源标识} |

- get方法的参数只能是query或path方式，不能是body

- post、delete、put没有限制，可以是body、query或path类型

- body和query或path可以同时存在于parameters的定义中。

  **需要注意**：GET方式提交的数据最多只能是1024字节，实际上URL不存在参数上限的问题，HTTP协议规范没有对URL长度进行限制。这个限制是特定的浏览器及服务器对它的限制，POST是没有大小限制的，HTTP协议规范也没有进行大小限制，起限制作用的是服务器的处理程序的处理能力。

- 当get、post、delete、put的**in定义为path**时，需要添加路径参数；如上面的示例中输入参数bookName的in参数为path，则relativePath路径修改成如下方式：

  ​	/service/v1/getBookInfo/{bookName}

  如果新增加一个in为path参数bookPrice，现在包含两个in为path输入参数，“relativePath路径”为：

  ​	/service/v1/getBookInfo/{bookName}/{bookPrice}

  更多参数时以此方式类推。

 ### 1.2.2Yaml可使用的数据的类型

| CommonName | type    | format    |
| ---------- | ------- | --------- |
| Integer    | integer | int32     |
| long       | integer | int64     |
| float      | number  | float     |
| double     | number  | double    |
| string     | string  |           |
| byte       | string  | byte      |
| binary     | string  | binary    |
| boolean    | boolean |           |
| date       | string  | date      |
| date time  | string  | date-time |
| password   | string  | password  |

​	基本类型可用于指定输入和输出参数，其中type和format是文件中使用的关键字。可以参数定义是双层结构，某些同一名称的type下可能会有多个format，因此使用这些type值时需要追加format具体指明参数类型；对于具有明确无歧义的type，则不要format关键字，如boolean和string，string比较特殊，string下是存在多个format类型，但如果使用默认的字符串类型，此时format为空，不需要指定format。

```YAML
# 相当于定义int
type: integer #指明大的类型
format: int32 #指明具体类型

# 相当于定义List<String>
type: array
  items:
    type: string  
```

如果输入和输出参数不是上述的基本类型，那在定义输入和输出参数类型是采用如下方式：

- 参数是单对象类型，type和format被置换如下语句：

```yaml
 schema: # 声明返回值Schema
 $ref: '#/definitions/Book'
 # 相当于定义Book类型参量，该部分只能是由definites中定义的类型。
```

- 参数是多对象类型，如List<Book>，type和format被置换如下语句：

```yaml
schema: # 声明返回值Schema
  type: array
    items:
      $ref: '#/definitions/Book'
# 相当于定义List<Book>类型参量，该部分只能是由definites中定义的类型。
```

 ## 1.3 definitions定义 

​      definitions下定义的对象的结构如下，定义的对象在yaml文件编译后会转换成java数据类

```
对象名称:
    对象描述: #可省略
    对象包含的属性:  
      属性1名称
        属性描述  #可省略
        属性数据类型
      属性2名称
        属性描述  #可省略
        属性数据类型
      ……
```

具体示例如下：

```yaml
definitions:
  PipelinePOJO:
    description: Pipeline POJO Object
    properties:
      id:
        type: integer
        format: int64
        description: Pipeline id.
      name:
        type: string
        description: Pipeline name.
      creator:
        type: string
      createTime:
        type: integer
        format: int64
      instance: #表示instance是非基本类型，类型是PipelineInstancePOJO
        $ref: '#/definitions/PipelineInstancePOJO'
      stages: #声明stages是List< PipelineStagePOJO>类型
        type: array
        items:
          $ref: '#/definitions/PipelineStagePOJO'
```

相当于如下java文件：

```java
public class PipelinePOJO  {    
    private Long id = null;
    private String name = null;
    private String creator = null;
    private Long createTime = null;
    private PipelineInstancePOJO instance = null;
    private List<PipelineStagePOJO> stages = new ArrayList<PipelineStagePOJO>();

    public Long getId() {return id;}
    public void setId(Long id){this.id = id;}

    public String getName(){return name;}
    public void setName(String name){this.name = name;}
	......
}
```

每种类型的定义和具体解释如下

1)   如果定义的参数中有包含char数组的，需要定义成

```yaml
gitPassword:
  type: string
  format: password
# 等价：private char[] gitPassword;
```

2)   如果有参数是其它类型的数组的，需要转换成List来定义

```yaml
# 等价：private List<String> value;
values:
  type: array
  items:
    type: string
# 等价: private List<PipelineStagePOJO> values = new ArrayList< PipelineStagePOJO>();
values:
  type: array
  items:
    $ref: '#/definitions/PipelineStagePOJO'
```

3)  如果有Map类型的，我们接口中不支持Map<String, Object>，Object需要有明确的类型，不能是抽象类型 :

```yaml
values:
  type: object
  additionalProperties:
    type: string
# 等价：private Map<String, String> values = new HashMap<String, String>();
# 如果Map里面value是对象的，定义如下：
stuInfo:
  type: object
  additionalProperties:
    $ref: '#/definitions/StuInfo'
# 生成属性如下：private Map<String, StuInfo> stuInfo = new HashMap<String, StuInfo>();
```

4)  List与Map互相嵌套的情形定义:

```yaml
values:
  type: object
  additionalProperties:
    type: array
    items:
      type: string
# 等价：private Map<List<String>> values = new HashMap<List<String >>();
values:
  type: array
  items:
    type: object
    additionalProperties:
      type: string
# 等价：private List<Map<String, String>> values = new ArrayList<Map<String, String >>();
```

5) 对象引用:

```yaml
values:
  $ref: '#/definitions/PipelineStagePOJO'
# private PipelineStagePOJO values;
```

6)      对象可以继承

```yaml
Response:
  description: Basic response with result and message
  properties:
    result:
      type: string
    message:
      type: string
ResponseString:
  description: Response with a simple string value
  allOf: #声明ResponseString继承于Response
  - $ref: '#/definitions/Response'
  - type: object
    properties: #除了Response属性外，额外增加的属性
      entity:
        type: string
        description: return a string
# 这样ResponseString对象就继承了Response对象，并且添加了一个自己的entity属性
```

等价于：

```java
public class Response  {
    
    private String result = null;
    private String message = null;

    public String getResult(){return result;}
    public void setResult(String result){this.result = result;}
    public String getMessage(){return message;}
    public void setMessage(String message){this.message = message;}
}

public class ResponseString extends Response {
    
    private String result = null;
    private String message = null;
    private String entity = null;

    public String getResult(){return result;}
    public void setResult(String result){this.result = result;}
    
    public String getMessage(){return message;}
    public void setMessage(String message){this.message = message;}

    public String getEntity(){return entity;}
    public void setEntity(String entity){this.entity = entity;}
}
```



# 2.swagger

swagger-api定义了一套规范，在github上有：

- swagger-codegen ：从yaml生成接口；一个模板驱动引擎，通过分析用户swagger资源声明以各种语言生成客户端代码
- swagger-ui：一个无依赖的HTML、JS和CSS集合，可以为swagger兼容API动态生成优雅文档；
- swagger-editor：
- swagger-core ：用于java/Scala的Swagger实现。与 JAX-RS（Java API for RESTful Web Services）、Servlets和Play框架进行集成；
- swagger-tools : 提供各种与Swagger进行集成和交互的工具。
- swagger-js ：用于javaScript的Swagger实现；
- swagger-node-express：swagger模块，用于node.js的Express web引用框架。



**swagger的不足：**

1. 不支持复杂或非Java类型，如泛型、JSONObject等。
2. 不支持查询多参数封装为一个bean 
   - JAX-RS 支持@BeanParam注解，可以把?param1=aaa&param2=bbb的多个参数丢到一个bean里面去



swagger的注解@Api、@ApiOperation、@ApiResponses等，如下所示:：

```java
@Path("promotionmanage/v1/promotions")	//访问路径
@Api(tags = {"ApiPromotionResource"})	//tags-name:用于代码生成，声明方法所在类名称
@Produces(MediaType.APPLICATION_JSON)  	//声明接口返回值是非基本类型
@Consumes(MediaType.APPLICATION_JSON)	//声明接口输入参数中包含非基本类型
public class PromotionResourceApi {

    @GET
    @ApiOperation(value = "查询促销活动列表",	//summary:综述
            notes = "返回 QueryPromotionListRsp",	//description:具体描述
            response = QueryPromotionListRsp.class)	//正常相应的数据
    @ApiResponses(value = {@ApiResponse(code = 400, message = "Invalid ID supplied"),
            @ApiResponse(code = 404, message = "Pet not found")})  //异常相应的错误码和描述
    public QueryPromotionListRsp getPromotions(@BeanParam QueryPromotionListReq req) {
        //输入参数使用@BeanParam注解，生成的yaml的输入为，该对象的每一个属性；
        //如果没有@BeanParam注解，生成的yaml的输入为该对象，同时会在yaml中生成对象定义；
        return null;
    }

    @GET
    @Path("/{promotionId}")
    @ApiOperation(value = "查询促销详情",
            notes = "返回 PromotionInfoRsp",
            response = PromotionInfoRsp.class)
    @ApiResponses(value = {@ApiResponse(code = 400, message = "Invalid ID supplied"),
            @ApiResponse(code = 404, message = "Pet not found")})
    public PromotionInfoRsp getPromotionDetails(@PathParam("promotionId") @NotNull @Size(min = 0, max = 64) String promotionId) {
        //@PathParam 该注解表明，此输入参数为路径参数；
        return null;
    }

    @POST
    @ApiOperation(value = "创建促销",
            notes = "返回 Result",
            response = Result.class)
    @ApiResponses(value = {@ApiResponse(code = 400, message = "参数不合法"),
            @ApiResponse(code = 404, message = "创建失败")})
    public Result createPromotion(CreatePromotionReq createPromotionReq) {
        return null;
    }

    @PUT
    @ApiOperation(value = "编辑促销",
            notes = "返回 Result",
            response = Result.class)
    @ApiResponses(value = {@ApiResponse(code = 400, message = "参数不合法"),
            @ApiResponse(code = 404, message = "编辑促销失败")})
    public Result editPromotion(EditPromotionReq editPromotionReq) {
        return null;
    }

    @POST
    @Path("/check")
    @ApiOperation(value = "校验促销资格",
            notes = "校验某个客户是否具有参与促销活动的资格",
            response = BaseResult.class)
    @ApiResponses(value = {@ApiResponse(code = 400, message = "Invalid ID supplied"),
            @ApiResponse(code = 404, message = "Pet not found")})
    public BaseResult checkPromotionQualification(CheckPromotionQualificationReq req) {
        return null;
    }
}
```





## 2.1 Api Swagger-ui

1. 创建一个springBoot项目；

```java
//springBoot的启动类
@EnableWebMvc
@SpringBootApplication
public class App {
    public static void main( String[] args ) {
        SpringApplication.run(App.class,args);
    }
}
```

2. 导入swagger相关依赖；

```xml
<!--swagger api 依赖-->
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
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.6.6</version>
</dependency>
```

3. 编写swagger配置类

```java
//这些注解必须存在
@Configuration
@EnableSwagger2
@EnableWebMvc
@ComponentScan(basePackages = "com.nyf.controller") //必须扫描Controller包
public class SwaggerConfig {
    @Bean
    public Docket customDocket(){
        return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo());
    }
    private ApiInfo apiInfo(){
        Contact contact=new Contact("NYF","http://nyfblack.github.io","13659292181@163.com");
        return new ApiInfoBuilder().title("nyf的swagger demo 项目")
                .description("api接口").contact(contact).version("1.0.0").build();
    }
}
```

4. 编写webMVC配置文件

```java
@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        //注册映射
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
    }
}
```

5. 控制器类

```java
package com.nyf.controller;

import com.nyf.model.User;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiOperation;
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

@Api(value="用户模块",description = "用户接口模块")
@RestController
public class UserController {

    //模拟数据库储存数据
    public static List<User> users = new ArrayList<>();
    static {
        users.add(new User("zs","123"));
        users.add(new User("ls","456"));
    }

    @ApiOperation(value="获取用户列表",notes = "获取所有用户信息")
    @ResponseBody
    @GetMapping("/users")
    public Object index(){
        HashMap<String,List> map = new HashMap<>();
        map.put("users",users);
        return users;
    }

    @ApiOperation(value = "根据id查询用户",notes = "获取单个用户信息")
    @ApiImplicitParam(value = "用户id",paramType = "path")
    @ResponseBody
    @GetMapping("/users/{id}")
    public Object getUserById(@PathVariable("id") String id){
        int uid = Integer.parseInt(id);
        return users.get(uid);
    }

    @ApiOperation(value="添加用户")
    @ApiImplicitParam(value = "用户类",paramType = "query")
    @ResponseBody
    @PostMapping("/user")
    public Object addUser(User user){
        System.out.println(user);
        return users.add(user);
    }

    @ApiOperation(value="删除用户",notes = "根据url的id来指定删除对象")
    @DeleteMapping(value="/users/{id}")
    public void delete(@PathVariable int id){
        users.remove(id);
    }
} 
```



## 2.2 java->yaml

pom.xml

```xml
<dependencies>
    <!--swagger 的依赖-->
    <dependency>
        <groupId>io.swagger</groupId>
        <artifactId>swagger-jersey2-jaxrs</artifactId>
        <version>1.5.10</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.1.7</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
    </dependency>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-validator</artifactId>
        <version>5.2.4.Final</version>
    </dependency>
</dependencies>
```

web.xml

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app version="2.4" xmlns="http://java.sun.com/xml/ns/j2ee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee  http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

  <servlet>
    <servlet-name>jersey</servlet-name>
    <servlet-class>org.glassfish.jersey.servlet.ServletContainer</servlet-class>
    <init-param>
      <param-name>jersey.config.server.provider.packages</param-name>
      <param-value>
        io.swagger.jaxrs.listing,
        com.demo.swagger.jaxrs
      </param-value>
    </init-param>
    <init-param>
      <param-name>jersey.config.server.wadl.disableWadl</param-name>
      <param-value>true</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>jersey</servlet-name>
    <url-pattern>/api/*</url-pattern>
  </servlet-mapping>

  <filter>
    <filter-name>ApiOriginFilter</filter-name>
    <filter-class>com.demo.swagger.jaxrs.util.ApiOriginFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>ApiOriginFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
</web-app>
```



## 2.3 yaml->java

1. 下载jar包

   swagger-codegen-cli.jar 

2. 查看Swagger Codegen的帮助信息

   ```shell
   java -jar swagger-codegen-cli-2.2.1.jar help generate
   ```

3. 查看Swagger Codegen支持的具体某个语言的使用帮助，拿java举例

   ```shell
   java -jar swagger-codegen-cli-2.2.1.jar config-help -l java
   ```

4. 利用Swagger Codegen根据服务生成客户端代码

   ```shell
   java -jar swagger-codegen-cli-2.2.1.jar generate 
   -i http://petstore.swagger.io/v2/swagger.json 
   -l java 
   -o samples/client/pestore/java
 
 #-i指定swagger描述文件的路径,url地址或路径文件;
 #  该参数为必须(http://petstore.swagger.io/v2/swagger.json是官方的一个例子，我们可以改成自己的服务)
 #-l指定生成客户端代码的语言,该参数为必须
 #-o指定生成文件的位置(默认当前目录)
 
 #springBoot的使用
 java -jar modules/swagger-codegen-cli/target/swagger-codegen-cli.jar generate \
  -i http://petstore.swagger.io/v2/swagger.json \
  -l spring \
  -o samples/server/petstore/springboot
  #也可以添加json格式的配置文件
  #{"basePackage":"io.swagger","configPackage":"io.swagger.config"}
  #使用命令 -c myOptions.json 指出；
  #To Use-it : in the generated folder try mvn package for build jar.
  #Start your server java -jar target/swagger-springboot-server-1.0.0.jar
  #SpringBoot listening on default port 8080
   ```

   除了可以指定上面三个参数，还有一些常用的：

   -c json格式的配置文件的路径;文件为json格式,支持的配置项因语言的不同而不同

   -a 当获取远程swagger定义时,添加授权头信息;URL-encoded格式化的name,逗号隔开的多个值

   --api-package 指定生成的api类的包名

   --artifact-id 指定pom.xml的artifactId的值

   --artifact-version 指定pom.xml的artifact的版本

   --group-id 指定pom.xml的groupId的值

   --model-package 指定生成的model类的包名

   -s 指定该参数表示不覆盖已经存在的文件

   -t 指定模版文件所在目录











