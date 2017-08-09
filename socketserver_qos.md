Socket Server 服务质量监控
===================
基于现有系统结构，客户端持续检测，服务端收集客户端报告的数据，实时描绘用户体验。

目的
-------------------
额度拍卖系统，用户数量众多，并且客户端运行环境多样。常有用户报告“不稳定”，但我们不能确定存在相同情况用户占比，以及发生问题的时间。  
为了掌握真实的用户体验，设计此功能，让客户端主动报告。运维人员则化被动为主动，可以有针对性调整，实时了解相关影响，持续提高整个系统运行表现。

理论基础
-------------------
当前系统最被关注的部分是公开信息，其实时性要求非常高，而且数量多，作为性能检测的对象比较合适。  
每个公开消息均包含时间戳Ts（代表此消息发出时间），经过网络路径传输后，由客户端接收，记录接收时间Tc。
```text
                         +-----------+
                         |  Message  | +----->
+----------+             +-----------+                     +----------+
|          |                                               |          |
|  Server  | +--+-------------Network path------------+--+ |  Client  |
|          |    |                                     |    |          |
+----------+    |                                     |    +----------+
                |                                     |
                |                                     |
                |                                     |
                +                                     +
                Ts                                    Tc
     (Message send time from server)      (Message arrive time to client)
```
Ts与Tc的差值定义为∆t，包含了服务端与客户端固有本地时差∆t0、消息从服务端发出到客户端接收耗时te。
```text
Ts1 +-----te1------+ Tc1
Ts2 +-----te2------+ Tc2
Ts3 +-----te2------+ Tc3
          .
          .
          .
Tsn +-----ten------+ Tcn
```
每个数据包的te会在te0附近波动，那么∆t(∆t0+te)会在∆t0(∆t0+te0)附近波动，两者波动幅度完全一致。  
我们不能直接获取te的值，但通过∆t就可以了解te的波动，从而评估（网络）环境稳定程度，波动幅度越小则代表越稳定。  
客户端记录最近的几个∆t，计算标准偏差σ，量化反映稳定程度。

概要设计
-------------------
### 心跳数据附加内容
原心跳数据形式如下
```json
{
  "ts": "151045613"
}
```
增加（可选）节点`atts`，类型为数组。数组内数据项可以为多种类型，由`type`确定。当前仅存在一种类型`qos-report`。
```json
{
  "ts": "151045613",
  "atts": [
    {
      "type": "qos-report",
      "ct": 61462371600000,
      "dev": 0.25
    }
  ]
}
```
字段说明
* `ct`: clientTime，本报告的客户端时间戳。
* `dev`: STDEV，最近客户端消息接收时间波动σ，参见[上文](#理论基础)。

### 客户端功能
* 建立公开消息接收时间记录数据结构`MessageReceiveLog`。
```actionscript
public class MessageReceiveLog 
{
	/**
	 * 消息接收时间
	 */
	public var receivedTime:Date;
	/**
	 * 消息服务器时间戳
	 */
	public var messageServerTime:Date;
}
```
* 当收到公开消息时即存储相关信息，若存在超过`30`秒之前的数据则将其清除，或者总数大于`10`个则清除较旧的。
* 当存储的消息接收记录达到`10`个，则计算标准偏差σ，更新待发送质量报告。
* 当客户端发送心跳时检查是否存在待发送质量报告，若存在则将其附加在心跳数据中，并清除。
```text
                                       Max capacity = 1

                                             +
                                             |
                                             |
                                             |
+--------------------+                       +                    +--------------------+
|                    |                                            |                    |
|       Meter        |   (Re)Place     +-------------+   Attach   |  HeartBeatRequest  |
|                    |                 |             |            |                    |
|         +--------------------------> |  QoSReport  | +-------------------->          |
|                    |                 |             |            |                    |
|                    |             +   +-------------+   +        |                    |
|                    |             |                     |        +--------------------+
+--------------------+             +---------------------+

                                        Ready to send.
```
* 与服务器连接成功后，初始化相关计数，避免数据不一致带来的问题。

### 服务端功能
* 数据模型变更

心跳数据中添加附加数据`attachs`
```java
public class HeartBeatRequestCore implements RequestCore {
	/**
	 * 时间戳
	 */
	private final String ts;

	/**
	 * 附加数据
	 */
	private final List<AttachedBase> attachs;

	@JsonCreator
	public HeartBeatRequestCore(@JsonProperty("ts") String ts,
			@JsonProperty("atts") List<AttachedBase> attachs) {
		this.ts = ts;
		this.attachs = attachs;
	}

	public String getTs() {
		return ts;
	}

	public List<AttachedBase> getAttachs() {
		return attachs;
	}
}
```
附加数据类型定义
```java
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, include = JsonTypeInfo.As.PROPERTY, property = "type")
@JsonSubTypes({ @Type(name = "qos-report", value = AttachedQoSReport.class) })
public abstract class AttachedBase {
	private final String type;

	@JsonCreator
	protected AttachedBase(@JsonProperty("type") String type) {
		this.type = type;
	}

	public String getType() {
		return type;
	}
}

public class AttachedQoSReport extends AttachedBase {
	private final Date clientTime;
	private final double stdev;

	protected AttachedQoSReport(@JsonProperty("ct") Date clientTime, @JsonProperty("dev")double stdev) {
		super("qos-report");
		this.clientTime = clientTime;
		this.stdev = stdev;
	}

	public Date getClientTime() {
		return clientTime;
	}

	public double getStdev() {
		return stdev;
	}
}
```
* 当接收到心跳时，检查是否存在附加数据。若存在质量报告附加数据，则存储数据，包括`clientTime`, `stdev`, `clientIP`, `serverTime`, `bidNumber`, `sessionId`，存储在Redis(?)中。

### 监视器
* 大数据工具处理报告，显示实时数据图表，并且可以追溯历史。
