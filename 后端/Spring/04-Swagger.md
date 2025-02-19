# Swagger

Swagger 提供了一套通过代码和注解自动生成文档的方法。

## 添加依赖

```xml
<!-- 自动生成描述 API 的 json 文件 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>

<!-- 将 json 文件解析出来，用一种更友好的方式呈现 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

## 配置参数

```yml
swagger:
  title: API标题
  description: API描述
  version: 1.0.0
  terms-of-service-url: https://chanshiyu.com/
  base-package: com.chanshiyu
  contact:
    name: 蝉時雨
    url: https://chanshiyu.com/
    email: me@chanshiyu.com
```

## 配置类

```java
@Configuration
@EnableSwagger2
@ConditionalOnClass(EnableSwagger2.class)
@Data
@Component
@ConfigurationProperties(prefix = "swagger")
public class SwaggerConfig {

    /** API页面标题 */
    private String title;

    /** API描述 */
    private String description;

    /** API版本号 */
    private String version;

    /** API接口包路径 */
    private String basePackage;

    /** 服务条款地址 */
    private String termsOfServiceUrl;

    /** 联系人 */
    private Contact contact;

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage(basePackage))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title(title)
                .description(description)
                .termsOfServiceUrl(termsOfServiceUrl)
                .version(version)
                .contact(contact)
                .build();
    }
}
```

## 使用

| 注解名称           | 使用说明               |
| ------------------ | ---------------------- |
| @Api               | 描述一个 API 类        |
| @ApiImplicitParam  | 描述一个请求参数       |
| @ApiImplicitParams | 描述一组请求参数       |
| @ApiModel          | 描述一个返回的对象     |
| @ApiModelProperty  | 描述一个返回的对象参数 |
| @ApiOperation      | 描述一个 API 方法      |
| @ApiParam          | 描述一个方法的参数     |
| @ApiResponse       | 描述一个请求响应       |
| @ApiResponses      | 描述一组请求响应       |
| @ApiIgnore         | 表示忽略               |

```java
@Api(value = "商品类目管理", tags = {"商品类目管理Controller"})
public class ProductCategoryController {

    @ApiOperation(value = "类目列表", notes = "类目列表")
    @GetMapping("/list")
    public ResultVO<List<ProductCategory>> list(@ApiParam(value = "页码", defaultValue = "1") Integer pageNum,
                                                @ApiParam(value = "每页大小", defaultValue = "10") Integer pageSize) {
        PageRequest pageRequest = PageRequest.of(pageNum - 1, pageSize);
        Page<ProductCategory> page = productCategoryService.findList(pageRequest);
        ResultAttributesVO resultAttributesVO = new ResultAttributesVO(page.getPageable().getPageNumber() + 1, page.getSize(), page.getTotalElements());
        return ResultVO.ok(page.getContent(), resultAttributesVO);
    }

}
```

```java
@ApiModel(value = "商品类目")
public class ProductCategory {

    @ApiModelProperty(value = "类目ID", dataType = "Integer")
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "category_id")
    private Integer id;

    @ApiModelProperty(value = "类目名称", dataType = "String")
    @Column(name = "category_name")
    @NotBlank(message = "类目名称不能为空")
    private String name;

    @ApiModelProperty(value = "创建时间", dataType = "String")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", locale = "zh", timezone = "GMT+8")
    private Date createTime;

    @ApiModelProperty(value = "更新时间", dataType = "String")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", locale = "zh", timezone = "GMT+8")
    private Date updateTime;

}
```

启动项目访问路径查看文档：`http://192.168.51.242:8080/swagger-ui.html#/`。

参考文章：  
[在 Spring Boot 项目中使用 Swagger 文档](https://www.ibm.com/developerworks/cn/java/j-using-swagger-in-a-spring-boot-project/index.html)
