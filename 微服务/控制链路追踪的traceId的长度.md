---
title: 控制链路追踪的traceId的长度
tags: 
- 微服务
- 链路追踪
- traceId
categories: 微服务
---



#控制链路追踪的traceId的长度

spring cloud sleuth中的`SleuthProperties`这个配置类中存在`traceId128`属性，该属性用于控制traceId的长度。

```java
@ConfigurationProperties("spring.sleuth")
public class SleuthProperties {

	private boolean enabled = true;

	/** When true, generate 128-bit trace IDs instead of 64-bit ones. */
  /** 当是true的时候，构造128位的traceId而不是64位的traceId，默认是false，即默认64位 */
	private boolean traceId128 = false;

	/**
	 * True means the tracing system supports sharing a span ID between a client and
	 * server.
	 */
	private boolean supportsJoin = true;

	/**
	 * List of baggage key names that should be propagated out of process. These keys will
	 * be prefixed with `baggage` before the actual key. This property is set in order to
	 * be backward compatible with previous Sleuth versions.
	 *
	 * @see brave.propagation.ExtraFieldPropagation.FactoryBuilder#addPrefixedFields(String,
	 * java.util.Collection)
	 */
	private List<String> baggageKeys = new ArrayList<>();

	/**
	 * List of fields that are referenced the same in-process as it is on the wire. For
	 * example, the name "x-vcap-request-id" would be set as-is including the prefix.
	 *
	 * <p>
	 * Note: {@code fieldName} will be implicitly lower-cased.
	 *
	 * @see brave.propagation.ExtraFieldPropagation.FactoryBuilder#addField(String)
	 */
	private List<String> propagationKeys = new ArrayList<>();

	public boolean isEnabled() {
		return this.enabled;
	}

	public void setEnabled(boolean enabled) {
		this.enabled = enabled;
	}

	public boolean isTraceId128() {
		return this.traceId128;
	}

	public void setTraceId128(boolean traceId128) {
		this.traceId128 = traceId128;
	}

	public boolean isSupportsJoin() {
		return this.supportsJoin;
	}

	public void setSupportsJoin(boolean supportsJoin) {
		this.supportsJoin = supportsJoin;
	}

	public List<String> getBaggageKeys() {
		return this.baggageKeys;
	}

	public void setBaggageKeys(List<String> baggageKeys) {
		this.baggageKeys = baggageKeys;
	}

	public List<String> getPropagationKeys() {
		return this.propagationKeys;
	}

	public void setPropagationKeys(List<String> propagationKeys) {
		this.propagationKeys = propagationKeys;
	}

}
```

因此我们只需要在配置文件中配置对应数据就行了。

```yaml
spring:
  sleuth:
    # When true, generate 128-bit trace IDs instead of 64-bit ones.
    traceId128: false
```

