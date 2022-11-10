# Spring



在 Spring 项目中，只需按照以下步骤，即可快速使用 Kuja 提高生产性。

1. 导入 kuja 依赖
2. 定义业务异常
3. 编写业务逻辑




## 导入 Kuja 依赖

- 如果使用的是 `spring-webmvc`，那么需要导入 `kuja-spring-stater-webmvc` 组件库：
  ```xml
  <dependency>
      <groupId>com.wingsweaver.kuja</groupId>
      <artifactId>kuja-spring-starter-webmvc</artifactId>
      <version>版本号</version>
  </dependency>
  ```
- 如果使用的是 `spring-webflux`，那么需要导入 `kuja-spring-stater-webflux` 组件库：
  ```xml
  <dependency>
      <groupId>com.wingsweaver.kuja</groupId>
      <artifactId>kuja-spring-starter-webflux</artifactId>
      <version>版本号</version>
  </dependency>
  ```

> 除了依赖的组件库不一样，使用方式完全一样。





## 定义业务异常
在 resources 目录中，新建 `error-definitions.properties` 文件；

在其中，可以按照以下方式来定义要使用的业务异常。

``` properties
# 异常编码=异常消息
E10000001=目标账户不存在
E10000002=输入的密码不正确
# 带 1 个参数的消息的例子
E10000003=账户 {} 已经被注销
# 带 2 个参数的消息的例子
E10000004=账户 {} 已经被 {} 锁定
```





## 编写业务逻辑

使用 Kuja 后，您可以专注在业务逻辑上。

遇到业务异常时，只需简单使用 `triggerException` 函数将其抛出即可；Kuja 将会做好后续的全方位的、全局的异常处理。

1. 将各个 `Controller` 实现类的基类改为 `AbstractSpringController`：
``` java
public class AuthController extends AbstractSpringController {
    ......
}
```
2. 将各个 `Service` 实现类的基类改为 `AbstractSpringService`：
``` java
public class AuthService extends AbstractSpringService {
    ......
}
```
3. 实现业务逻辑，使用 `triggerException` 抛出指定编码的业务异常：
``` java
public AuthTokenDto login(LoginQuery query) {
    // 查找相同账户名称的记录
	TbUser tbUser = this.tbUserMapper.selectByAccount(query.getAccount());
	if (tbUser == null) {
		// 如果找不到对应账户，抛出异常 E10000001
		this.triggerBusinessException("E10000001");
	}

	// 检查账户的状态
	UserStatus status = UserStatus.valueOf(tbUser.getStatus());
	switch (status) {
		case UNREGISTERED:
			// 如果用户已经被注销，抛出异常 E10000003（带 1 个参数）
			this.triggerBusinessException("E10000003", tbUser.getAccount());
			break;
		case LOCKED:
			// 如果用户已经被锁定，抛出异常 E10000004（带 2 个参数）
			this.triggerBusinessException("E10000004", tbUser.getAccount(), tbUser.getLockedBy());
			break;
		default:
			break;
	}

	// 匹配账户和密码
	// 仅作示例使用，简单使用了明文密码
	if (!Objects.equals(query.getPassword(), tbUser.getPassword())) {
		// 如果密码不匹配，抛出异常 E10000002
		this.triggerBusinessException("E10000002");
	}

	// 返回
	AuthTokenDto authTokenDto = new AuthTokenDto();
	authTokenDto.setToken(UUID.randomUUID().toString());
	return authTokenDto;
}
```
4. Kuja 会自动捕获相关异常，并返回下面这样的返回结果：
``` json
{
	"code": "E10000001",
	"message": "目标账户不存在"
}
```

事实上，Kuja 还会全方位捕获所有异常（包含 Spring 异常），并根据设置进行处理。

例如：在发生 404 错误时，Kuja 默认会返回：
``` json
{
	"code": "failed",
	"message": "404 NOT_FOUND"
}
```

如果找不到合适的错误设置的话，Kuja 默认会返回：
``` json
{
	"code": "failed",
	"message": "An unexpected error has happened"
}
```





## 参考资料

### 示例项目

- [webmvc](https://github.com/wingsweaver/kuja/tree/main/kuja-spring/kuja-spring-starter-webmvc/examples/)
- [webflux](https://github.com/wingsweaver/kuja/tree/main/kuja-spring/kuja-spring-starter-webflux/examples/)

### 进阶用法

- [异常的定义和处理](../spring/error-handling.md)
- [返回结果的定义和加工](../spring/return-values.md)