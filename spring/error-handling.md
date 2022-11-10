# 异常的定义和处理




## 异常的定义

### 异常定义文件的位置

Kuja 默认会从 `classpath:/error-definitions.properties` 文件中，读取异常的定义。

您也可以使用下面的配置项目，指定异常定义文件的位置。

``` properties
kuja.domain.error-definition.locations=classpath:/my-error-definitions-1.properties,classpath:/my-error-definitions-2.properties
```

### 异常定义的格式

- 最简单的定义方式为：

  ``` properties
  # 异常编码=异常消息
  E10000001=目标账户不存在
  E10000002=输入的密码不正确
  # 带 1 个参数的消息的例子
  E10000003=账户 {} 已经被注销
  # 带 2 个参数的消息的例子
  E10000004=账户 {} 已经被 {} 锁定
  ```

- 这种简单的定义方式，跟以下定义方式等价：

  ``` properties
  # 异常编码.message=异常消息
  E10000001.message=目标账户不存在
  E10000002.message=输入的密码不正确
  E10000003.message=账户 {} 已经被注销
  E10000004.message=账户 {} 已经被 {} 锁定
  ```

- 事实上 Kuja 中异常定义 (`ErrorDefinition`) 本质上是一个字典，因此可以在其中定义任何数据。

  ``` properties
  # 异常编码.message=异常消息
  E10000001.message=目标账户不存在
  E10000001.status=404
  E10000002.message=输入的密码不正确
  E10000002.status=500
  E10000002.severity=error
  E10000003.message=账户 {} 已经被注销
  E10000004.message=账户 {} 已经被 {} 锁定
  ```

  > 如果要将自定义的数据保存到返回结果中，需要定制 ReturnValueNormalizer 类型的 Bean。

- 如果在异常的编码中出现了 `.` 符号，那么需要修改以下异常定义的分隔符设置。
  ``` properties
  kuja.domain.error-definition.attribute-delimiter=#
  ```
  配置之后，异常定义文件的定义方式可能如下：
  ``` properties
  # 异常编码=异常消息
  COM.E10000001=目标账户不存在
  COM.E10000002=输入的密码不正确
  COM.E10000003=账户 {} 已经被注销
  COM.E10000004=账户 {} 已经被 {} 锁定
  
  # 异常编码.message=异常消息
  COM.E10000001#message=目标账户不存在
  COM.E10000002#message=输入的密码不正确
  COM.E10000003#message=账户 {} 已经被注销
  COM.E1000004#message=账户 {} 已经被 {} 锁定
  ```





## 异常的抛出

`AbstractSpringController` 和 `AbstractSpringService` 的子类，可以使用 `triggerException` 函数来抛出定义好的异常。

``` java
void triggerBusinessException(String errorDefinitionCode, Object... args) throws BusinessException;

void triggerBusinessException(Throwable cause, String errorDefinitionCode, Object... args) throws BusinessException;

void triggerBusinessException(Locale locale, String errorDefinitionCode, Object... args) throws BusinessException;

void triggerBusinessException(Locale locale, Throwable cause, String errorDefinitionCode, Object... args) throws BusinessException;
```





## 异常的捕获

### spring-webmvc

- **第一道：AbstractSpringController**
  各 `AbstractSpringController` 的子类，可以重载 `handleError` 函数，以实现本 Controller 特有的异常捕获和处理。
  
  ``` java
  protected Object handleError(Throwable error) throws Throwable {
  	......
  }
  ```
  
- **第二道：全局错误处理 GlobalWebMvcErrorHandlerAdvice**
  用于在进入错误页面（/error）重定向之前进行错误拦截和处理，如：不支持指定的请求 METHOD 等。
  可以使用以下配置，关闭 GlobalWebMvcErrorHandlerAdvice。    
  ``` properties
  kuja.web.exception.global-error-advice-enabled=false
  ```
- **第三道：错误页面控制器 DefaultWebMvcErrorController
  用于在错误页面重定向时进行处理，如: 404 找不到相应的资源 等。
  可以使用以下配置，关闭 DefaultWebMvcErrorController。  
  ``` properties
  kuja.web.exception.global-error-handler-enabled=false
  ```

### spring-webflux
- **第一道：AbstractSpringController**
  各 `AbstractSpringController` 的子类，可以重载 `handleError` 函数，以实现本 Controller 特有的异常捕获和处理。
  
  ``` java
  protected Object handleError(Throwable error) throws Throwable {
  	......
  }
  ```
- **第二道：全局错误处理 GlobalWebFluxErrorWebExceptionHandler**
  基于 WebFlux 的 `ErrorWebExceptionHandler` 进行错误拦截和处理，如：不支持指定的请求 METHOD 等。
  可以使用以下配置，关闭 GlobalWebFluxErrorWebExceptionHandler。    
  ``` properties
  kuja.web.exception.global-error-handler-enabled=false
  ```





## 异常的处理

### 响应状态码到业务异常的映射

Kuja 中内置了将特定状态码的错误、映射成指定的业务异常的功能；这样可以提供更有意义的返回结果、而不是冷冰冰的 404 NOT FOUND 这种提示。

只需要按照以下格式，指定 `kuja.web.exception.status-error-definition-mappings` 配置项目即可。

``` properties
# 示例：将 404 错误映射为编码为 E00000001 的错误
kuja.web.exception.status-error-definition-mappings.404=E00000001
# 示例：将 405 错误映射为编码为 E00000002 的错误
kuja.web.exception.status-error-definition-mappings.405=E00000002
```

### 错误定义中返回响应状态码的设置

您可以在错误定义文件中、使用以下的方式来设置特定的业务异常对应的响应状态码。

``` properties
# 错误编码.status=返回的响应状态码
E10000001.status=400
E10000002.status=500
```


### 错误消息的 i18n 处理

如果您需要对错误消息进行 i18n 处理，您需要实现 `ErrorDefinitionMessageResolver` 接口，并且注册相应的 Bean。

``` java
@Configuration
public class LocalizationConfiguration {
    @Bean
    public ErrorDefinitionMessageResolver errorDefinitionMessageResolver() {
        return new MyErrorDefinitionMessageResolver();
    }
}
```

Kuja 中内置了简易的实现类 `SimpleI18nErrorDefinitionMessageResolver`，可以基于以下的错误定义方式、返回对应的消息文本。

> 错误编码.message.语言环境=该语言环境下的错误消息

``` properties
# 默认的错误消息
E10000001.message=Good morning!
# 中文环境下的错误消息
E10000001.message.zh-CN=早上好！
# 日语环境下的错误消息
E10000001.message.ja-JP=おはようございます！
```





## 异常的处理（进阶）

Kuja 已经内置了简易版的异常处理功能，以基于异常定义返回相应的结果。

如果不需要定制异常处理过程，可以跳过以下章节。

### 处理流程

捕获到异常之后，Kuja 按照以下的流程、处理异常，并且返回处理结果。

``` mermaid
%% graph TD; Kuja 的异常处理流程
graph LR;
ErrorNormalizer --> ErrorHandlerContextFactory --> PreErrorHandler --> ErrorHandler --> PostErrorHandler --> ErrorHandlerResultResolver
```

上述各接口的功能如下：

| 名称                       | 功能                                                         |
| -------------------------- | ------------------------------------------------------------ |
| ErrorNormalizer            | 用于对要处理的异常进行加工处理。<br>比如：把 `HttpRequestMethodNotSupportedException` 异常转换成 `ResponseStatusException`，以便于后续处理。 |
| ErrorHandlerContextFactory | 用于创建 `异常处理上下文（ErrorHandlerContext）` 的实例，以便在处理过程中传递数据。<br>详细情况请参照下文的 `异常处理上下文` 章节。 |
| PreErrorHandler            | 正式的异常处理之前，对 `异常处理上下文` 进行必要的初始化处理。<br>比如：将当前的 HttpServletRequest 实例设置到异常处理上下文中。 |
| ErrorHandler               | 正式的异常处理。<br>比如：根据不同的异常类型，设置特定的返回结果。 |
| PostErrorHandler           | 异常处理后的辅助处理。<br>比如：更新 HTTP 响应的状态码、Header 等。 |
| ErrorHandlerResultResolver | 从 `异常处理上下文` 中提取错误处理结果。                     |

### 异常处理上下文（ErrorHandlerContext）

异常处理上下文，即单次异常处理过程中共享的上下文，以便于在各种不同的处理接口中传递数据。

异常处理上下文可以简单理解成一个字典，其中可以包含任何数据；

对于数据的读写，则是由各种 `ErrorHandlerContextView` 来提供的 —— 有些类似于数据库中的 View。

``` mermaid
classDiagram

class ErrorHandlerContext
class ErrorHandlerContextFactory {
	+ CreateErrorHandlerContext : 创建异常处理上下文
}
class ErrorHandlerContextView {
	+ BusinessContext : 业务上下文
	+ Error : 要处理的异常
	+ IsHandled : 是否处理
	+ Result : 处理结果
}
class WebErrorHandlerContextView {
	+ ResponseStatus : 响应状态码
	+ ResponseHaders : 响应 Headers
}
class WebServletErrorHandlerContextView {
	+ HttpServletRequest : HTTP 请求
	+ HttpServletResponse : HTTP 响应
}
class WebFluxErrorHandlerContextView {
	+ ServerWebExchange : 服务网络交换 
}

<<interface>> ErrorHandlerContext
<<interface>> ErrorHandlerContextFactory

ErrorHandlerContext <.. ErrorHandlerContextFactory : 生成
ErrorHandlerContext <.. ErrorHandlerContextView : 读写
ErrorHandlerContextView <|-- WebErrorHandlerContextView : 继承（Web 通用）
WebErrorHandlerContextView <|-- WebServletErrorHandlerContextView : 继承（webmvc 专用）
WebErrorHandlerContextView <|-- WebFluxErrorHandlerContextView : 继承（webflux 专用）
```



## 参考资料

- [Web 处理](./web.md)
- [Kuja 底层的错误定义](../common/error-handling.md)
