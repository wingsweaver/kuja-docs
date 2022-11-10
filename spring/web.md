# Web 应用



## 业务上下文

``` mermaid
classDiagram

class BusinessContext
class BusinessContextView {
	+ ContextType : 上下文的类型
}
class WebBusinessContextView {
	+ RawRequest : 原始的请求实例
	+ RawResponse : 原始的响应实例
	+ RequestInfo : 请求的信息摘要
	+ ResponseInfo : 响应的信息摘要
}
class ServletBusinessContextView {
	+ HttpServletRequest
	+ HttpServletResponse
}
class WebFluxBusinessContextView {
	+ ServerWebExchange
	+ ServerHttpRequest
	+ ServerHttpResponse
}

<<interface>> BusinessContext

BusinessContext <.. BusinessContextView : 读写
BusinessContextView <|-- WebBusinessContextView : 继承（Web 通用）
WebBusinessContextView <|-- ServletBusinessContextView : 继承（Servlet/webmvc 专用）
WebBusinessContextView <|-- WebFluxBusinessContextView : 继承（webflux 专用）
```





## 返回结果的转换（BodyAdvice）

