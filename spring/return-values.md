# 返回结果的定义和加工



## 字段定义

### 基本字段

Kuja 中返回结果的基本定义 `ReturnValue` 如下：

``` java
public class ReturnValue extends AbstractPojo implements Serializable {
    /**
     * 错误码。
     */
    private Object code;

    /**
     * 错误消息。
     */
    private String message;

    /**
     * 附加数据（如：异常信息、请求参数等）。
     */
    private Map<String, Object> additional;
}
```

> Kuja 中做了特殊处理，如果 `附加数据 (additional)` 字段没有设置的话，将不会被序列化到响应结果中。

大部分情况下，`code` 和 `message` 字段都没什么用（默认成功）；只是在发生错误时，才需要根据错误类型、设置为相应的结果，以便于调用者使用。

`additional` 字段则可以按需来保存定制的数据；当然，也要根据。

> 发生异常时，Kuja 中推荐：
>
> - message 中写入便于理解的返回结果，而不是直接使用异常的消息（对调用者来说没有意义）
> - 在 additional 字段中按需存入异常相关信息，便于调试分析 —— Kuja 已经内置了此功能

### 返回数据

在此基础上，Kuja 还提供了一个带有返回数据的 `ReturnValueData` 定义：

``` java
public class ReturnValueData<T> extends ReturnValue {
    /**
     * 实际数据。
     */
    private T data;
}
```

> Kuja 中做了特殊处理，如果 `实际数据 (data)` 字段没有设置的话，将不会被序列化到响应结果中。

在实际的项目中，返回结果到底是继承 ReturnValue、还是直接使用 ReturnValueData，是个见仁见智的问题。

``` java
// 1. 继承 ReturnValue 的用法
public class AuthTokenDto extends ReturnValue {
    private String token;
    private long expiresAt;
}

public AuthTokenDto auth(AuthQuery query) {
    ......
}

// 2. 使用 ReturnValueData 的用法
public class AuthTokenData {
    private String token;
    private long expiresAt;
}

public ReturnValueData<AuthTokenData> auth(AuthQuery query) {
    ......
}
```

Kuja 中也没有特殊要求，只要定义明确、便于序列化和反序列化即可。





## 数据加工

### `code` 和 `message` 字段的设置

一种办法是，生成返回结果的时候，明确地设置 `code` 和 `message` 字段的值。

``` java
return ReturnValue.code("succeed").message("操作成功");
// 或者
return ReturnValueData.data(someData).code("succeed").message("操作成功");
```

当然，您也可以封装本项目专用的工具类，以减少重复的代码量。

或者，您**可以使用 Kuja 来自动补全这两个字段的值** —— 只需要在配置文件中设置如下字段即可。

``` properties
kuja.domain.model.normalizer.default-return-value.succeed.code=0
kuja.domain.model.normalizer.default-return-value.succeed.message=操作成功
kuja.domain.model.normalizer.default-return-value.fail.code=-1
kuja.domain.model.normalizer.default-return-value.fail.message=操作失败
```



### `data` 字段的设置

一种办法是，生成返回结果的时候，明确地设置 `data` 字段的值。

``` java
return ReturnValueData.data(someData).code("succeed").message("操作成功");
```

不过，如果您使用 ReturnValueData 来返回数据的话，**可以使用 Kuja 来进行自动转换** —— 这个功能是默认开启的，不需要做额外设置。

``` java
// 1. 生成结果时，明确地在 ReturnValueData 中设置 data 字段的值
public ReturnValueData<AuthTokenData> auth(AuthQuery query) {
    ......
}

// 2. 直接返回 data 字段的值，让 Kuja 自动转换成 ReturnValueData 类型
public AuthTokenData auth(AuthQuery query) {
    ......
}
```



### `additional` 字段的设置

#### 保存错误信息

只需要打开以下设置，就可以在发生错误时自动保存错误信息到返回结果的 `additional` 字段中。

``` properties
# 可设置的值如下：
# onerror: 发生错误时保存错误信息
# never: 从不保存错误信息
kuja.domain.model.normalizer.error.mode=onerror
```

#### 保存请求信息

只需要打开以下设置，就可以按需保存请求信息到返回结果的 `additional` 字段中。

``` properties
# 可设置的值如下：
# always: 即使不发生错误，也要保存请求信息
# onerror: 发生错误时保存请求信息
# never: 从不保存请求信息
kuja.domain.model.normalizer.request.mode=onerror
```

#### 定制保存信息

如果要定制保存到 `additional` 字段的信息，除了在代码中明确地设置之外，还可以自定义以下接口的实现类的 Bean。

``` java
public interface ReturnValueNormalizer {
    /**
     * 对返回结果进行规范化处理。
     *
     * @param returnValue 返回结果
     */
    void normalize(ReturnValue returnValue);
}
```





## 参考资料

- [Kuja 底层的返回结果定义和加工](../common/return-values.md)

