---
title: 记一次spring webflux问题
tags:
- spring webflux
- 问题
categories: 
- 问题
- spring webflux
---



#记一次spring webflux问题

## 现象

在一次开发过程中，同事反馈他接收到的请求数据被截断了，怀疑在网关层面的数据修改导致请求被截断。

于是我进行了排查。目前的整个请求流程是这样的：

前端发起GET请求，后台网关层服务通过过滤器读取请求参数，并进行去空格，然后重新塞回请求，转发到对应的后台服务。

在这个过程中，请求体被修改了一次。因此极有可能是在这个去空格的过程中改变了请求信息，导致后台程序获取异常。

让我们来看一下相关的请求信息和代码。



## 请求信息

```http
http://localhost:7000/c/common/genQRCode?url=https%3A%2F%2Flocalhost:7004%2Fshare%2Fshow%3Fappid%3Dwx5c5a1f11111111c%26redirect_uri%3Dhttps%253A%252F%252Flocalhost:7004%252Fxmshow%252F%2523%252FbargainType%252Findex%253Ftype%253Dshare%2526id%253D95%2526broadcastId%253D11111111104%2526appid%253Dwx5c5a1f11111111c%26response_type%3Dcode%26scope%3Dsnsapi_userinfo%26state%3Dstate%23wechat_redirect&width=100&height=100
```



## 代码

### 去空格的代码

```java
private String doTrimGetParams(ServerWebExchange exchange) {
  // GET请求参数
  StringBuilder query = new StringBuilder();
  ServerHttpRequest request = exchange.getRequest();
  Set<String> names = request.getQueryParams().keySet();
  if (!CollectionUtils.isEmpty(names)) {
    names.forEach(name -> {
      String value = request.getQueryParams().getFirst(name);
      query.append(name).append("=");
      if (StringUtils.isNotBlank(value)) {
        query.append(value.trim());
      }
      query.append("&");
    });
    query.delete(query.lastIndexOf("&"), 
                 query.lastIndexOf("&") + 1);
  }
  return query.toString();
}
```

### 过滤器中请求体替换

```java
ServerHttpRequest requestForChange = exchange.getRequest();
String query = doTrimGetParams(exchange);

URI uri = requestForChange.getURI();
URI newUri = UriComponentsBuilder.fromUri(uri)
  .replaceQuery(query)
  .build(false)
  .toUri();
ServerHttpRequest request = requestForChange.mutate()
  .uri(newUri).build();
return chain.filter(exchange.mutate().request(request).build());
```



## 后台服务接收到的错误请求信息

很明显，请求参数中的url被截断了。

```json
{
    "width": "100",
    "decode url": "https://localhost:7004/share/show?appid=wx5c5a1f11111111c",
    "success": "success",
    "url": "https://localhost:7004/share/show?appid=wx5c5a1f11111111c",
    "height": "100"
}
```



## 排查过程

首先确定是否真的是去空格代码的问题，我注释掉去空格的代码做了验证，发现一切都恢复正常，因此问题确实是出在了这块代码。

### 明确认知，正确的结果应该是怎么样的

如果连什么是正确的结果都不知道，那么我们在修改过程中，怎么知道自己是对的呢？

于是，我首先查看正确的请求结果和错误的请求结果的区别是什么，从而进一步推断问题点。于是我将空格代码去除，模拟正确的请求环境，得到了以下结果：

```json
{
    "width": "100",
    "decode url": "https://localhost:7004/share/show?appid=wx5c5a1f11111111c&redirect_uri=https://localhost:7004/xmshow/#/bargainType/index?type=share&id=95&broadcastId=11111111104&appid=wx5c5a1f11111111c&response_type=code&scope=snsapi_userinfo&state=state#wechat_redirect",
    "success": "success",
    "url": "https://localhost:7004/share/show?appid=wx5c5a1f11111111c&redirect_uri=https%3A%2F%2Flocalhost:7004%2Fxmshow%2F%23%2FbargainType%2Findex%3Ftype%3Dshare%26id%3D95%26broadcastId%3D11111111104%26appid%3Dwx5c5a1f11111111c&response_type=code&scope=snsapi_userinfo&state=state#wechat_redirect",
    "height": "100"
}
```

明显的可以看出url被截断了。但这并不能让我有效的知道问题点在哪儿，只是让我有一个对错误的明确认知。因此，我需要查看代码进行排查，但显然代码路基异常简单，只是单纯的去空格，看不出来任何问题，因此需要打断点进行排查。

### 打断点排查

然后我开始打断点。端点排查中发现在代码`String value = request.getQueryParams().getFirst(name);`处，参数url的值就已经发生了变化。

![](https://i.loli.net/2019/05/09/5cd390849d1db.png)

显然，在获取参数的过程中，发生了转码。到这里，可以发现是spring对请求信息做了操作。我们再深入代码进行查看，在`org.springframework.http.server.reactive.AbstractServerHttpRequest`中：

```java
@Override
public MultiValueMap<String, String> getQueryParams() {
  if (this.queryParams == null) {
    this.queryParams = CollectionUtils.unmodifiableMultiValueMap(initQueryParams());
  }
  return this.queryParams;
}
```

发现了这里进行了转码操作。

```java
protected MultiValueMap<String, String> initQueryParams() {
  MultiValueMap<String, String> queryParams = new LinkedMultiValueMap<>();
  String query = getURI().getRawQuery();
  if (query != null) {
    Matcher matcher = QUERY_PATTERN.matcher(query);
    while (matcher.find()) {
      String name = decodeQueryParam(matcher.group(1));
      String eq = matcher.group(2);
      String value = matcher.group(3);
      // 此处对请求值进行了decode转码，然后返回
      value = (value != null ? decodeQueryParam(value) : (StringUtils.hasLength(eq) ? "" : null));
      queryParams.add(name, value);
    }
  }
  return queryParams;
}

private String decodeQueryParam(String value) {
  try {
    return URLDecoder.decode(value, "UTF-8");
  } catch (UnsupportedEncodingException ex) {
    return URLDecoder.decode(value);
  }
}
```



到这里，我们就知道了原来是spring web flux作的妖，他把请求参数用URLDecoder.decode做了一次转码。我一直操作的是转码后的参数，因此重新存入的也是转码后的参数，这就导致数据不对了。

这个时候，我们显然知道如何修改这个问题了。在去空格之后，对参数进行encode处理。

```java
String value = request.getQueryParams().getFirst(name);
try {
  // 上面的getQueryParams方法是 spring web flux 的 AbstractServerHttpRequest 中的方法。
  // 该方法中会对请求参数进行 URLDecoder.decode(value, "UTF-8"); 解码
  // 此处只是去空格处理还需要把原数据转发到不同的内部服务，不需要解码，
  // 因此添加 URLEncoder.encode(value, "UTF-8"); 进行数据还原处理。
  value = URLEncoder.encode(value, "UTF-8");
} catch (UnsupportedEncodingException e) {
  e.printStackTrace();
}
query.append(name).append("=");
if (StringUtils.isNotBlank(value)) {
  query.append(value.trim());
}
query.append("&");
```



到这里，大功告成了吧，然后我去做了验证，然而，结果啪啪打脸。

得到的结果：

```json
{
    "width": "100",
    "decode url": "https://localhost:7004/share/show?appid=wx5c5a1f11111111c&redirect_uri=https%3A%2F%2Flocalhost:7004%2Fxmshow%2F%23%2FbargainType%2Findex%3Ftype%3Dshare%26id%3D95%26broadcastId%3D11111111104%26appid%3Dwx5c5a1f11111111c&response_type=code&scope=snsapi_userinfo&state=state#wechat_redirect",
    "success": "success",
    "url": "https%3A%2F%2Flocalhost%3A7004%2Fshare%2Fshow%3Fappid%3Dwx5c5a1f11111111c%26redirect_uri%3Dhttps%253A%252F%252Flocalhost%3A7004%252Fxmshow%252F%2523%252FbargainType%252Findex%253Ftype%253Dshare%2526id%253D95%2526broadcastId%253D11111111104%2526appid%253Dwx5c5a1f11111111c%26response_type%3Dcode%26scope%3Dsnsapi_userinfo%26state%3Dstate%23wechat_redirect",
    "height": "100"
}
```

正确的结果：

```json
{
    "width": "100",
    "decode url": "https://localhost:7004/share/show?appid=wx5c5a1f11111111c&redirect_uri=https://localhost:7004/xmshow/#/bargainType/index?type=share&id=95&broadcastId=11111111104&appid=wx5c5a1f11111111c&response_type=code&scope=snsapi_userinfo&state=state#wechat_redirect",
    "success": "success",
    "url": "https://localhost:7004/share/show?appid=wx5c5a1f11111111c&redirect_uri=https%3A%2F%2Flocalhost:7004%2Fxmshow%2F%23%2FbargainType%2Findex%3Ftype%3Dshare%26id%3D95%26broadcastId%3D11111111104%26appid%3Dwx5c5a1f11111111c&response_type=code&scope=snsapi_userinfo&state=state#wechat_redirect",
    "height": "100"
}
```

虽然不再被截断，但得到的结果仍然和正确结果略有不同。url中的第一个https后的//被转码了，而正确的结果并没有解码。

很显然，添加的代码起到了作用，但还有其他地方，对请求参数做了处理。这里我仍旧打上端点，一步一步往下走。到这一步，我对新旧两个uri对象进行了对比：

老的URI对uri

![](https://i.loli.net/2019/05/09/5cd39575ee290.png)

新的URI对象newUri

![](https://i.loli.net/2019/05/09/5cd395933c13e.png)



发现两者的query和string参数中的数据不一致，新的URL是`https%253A%252F%252F`，老得URL是`https%3A%2F%2F`。到这里，显而易见了，问题就出在这段代码上

```java
URI newUri = UriComponentsBuilder.fromUri(uri).replaceQuery(query).build(false).toUri();
```

这里调用了4个方法，按照以往的经验，要排查就得一个一个看过去问题出在了哪里。但是，这个时候，我们发现了IDEA这款工具的强大，它为我们显示了入参的名称：

![](https://i.loli.net/2019/05/09/5cd396fe99803.png)

发现build的参数是encoded，那么我们就有理由怀疑，这个encoded表示是否已经加码过。于是我点进去看源码。

```java
/**
 * Build a {@code UriComponents} instance from the various components
 * contained in this builder.
 * @param encoded whether all the components set in this builder are
 * encoded ({@code true}) or not ({@code false})
 * @return the URI components
 */
public UriComponents build(boolean encoded) {
  return buildInternal(encoded ?
                       EncodingHint.FULLY_ENCODED :
                       this.encodeTemplate ? EncodingHint.ENCODE_TEMPLATE : EncodingHint.NONE);
}
```

 根据注释可以看到，这个encoded参数表示所有的uri参数是否被encode过。显然，我已经对其进行过encode。所以需要传一个true。

然后我在此进行了验证，发现结果终于正确了。

大功告成～