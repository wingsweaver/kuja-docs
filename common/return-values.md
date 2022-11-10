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

