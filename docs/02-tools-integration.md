\# 工具集成与接口定义

\## 1. 整体通信协议

\### 1.1 调用方式

Dify智能Agent 通过 \*\*HTTP POST Webhook\*\* 调用n8n子工作流工具，实现业务数据查询与操作。



\### 1.2 统一全局响应规范

所有n8n工具返回数据强制遵循统一JSON格式，便于Dify统一解析、异常捕获：

```json

{

&#x20; "success": true,

&#x20; "data": {},

&#x20; "error": "",

&#x20; "trace\_id": "trace-唯一追踪ID"

}

```



\## 2. 六大核心工具接口字典

| 工具名称 | 输入参数 | 核心输出数据 | 底层业务逻辑 |

| ---- | ---- | ---- | ---- |

| get\_order\_info | order\_id | 订单状态、商品名称、订单金额 | 查询orders订单数据表 |

| get\_logistics | order\_id | 物流单号、快递公司、物流状态 | 查询logistics物流数据表 |

| apply\_free\_return | order\_id、退货原因 | 退货单号、申请状态、系统提示文案 | 写入return\_orders退货数据表，提交售后申请 |

| get\_user\_vip\_level | user\_id | 用户VIP等级、累计消费金额 | 查询users用户信息表 |

| query\_size\_chart | 商品品类 | 尺码、胸围、腰围等尺码数据 | 查询size\_charts尺码参数表 |

| get\_coupon\_info | user\_id | 优惠券编码、优惠金额、有效期 | 查询coupons用户优惠券表 |



\## 3. 全局异常处理机制

\### 3.1 n8n 服务端异常处理

通过Code节点校验入参、查询结果，空数据/参数异常时，统一返回失败格式：

```json

{"success": false, "error": "未查询到对应信息，请核对参数"}

```



\### 3.2 Dify 客户端异常处理

Agent识别到`success: false`时，触发止损Prompt机制，停止重复重试调用工具，自动向用户返回友好兜底话术，提升交互体验。

