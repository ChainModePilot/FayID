# 第 6 章 隐私与 GMC 接口

本章描述 FayID 体系的两条硬约束：**Human ID 不出站不入日志**，以及 **GMC Interface 的只读边界**。这两条约束是 FayID 作为 Global Merit Chain 长期身份层的安全前提。

---

## 隐私硬约束

### 出站载荷规则

定义"出站载荷"为：FayID System 之外（包括第三方资源、传统鉴权来源、Global Merit Chain、可观测性后端）所能观察到的任意字节流。

| 实体 | 是否允许在出站载荷中以明文出现 |
| --- | --- |
| **Human ID** | **禁止**，除非该次通信明确以 Human ID 自证为目的 |
| **Mnemonic** | **禁止**，无任何例外 |
| **Private Key** | **禁止**，无任何例外 |
| iFay ID / coFay ID / Organization ID | 允许 |
| Dynamic Code | 允许 |
| Verification Code | 仅与 coFay ID 配对出现，不单独广播 |
| Authorization Grant | 允许 |

### 日志与可观测性

```
log_allow  := { dynamicCode, ifayID, cofayID, organizationID, grantID, errorCode }
log_deny   := { humanID(明文), mnemonic, privateKey, verificationCode(明文) }
```

实现侧的日志器必须以**白名单方式**过滤：任何序列化路径在写入日志前必须经过 redact 步骤。

> 简言之：Human ID 明文与 Mnemonic 永远不出现在外部可观察的字节流中——无论是网络包、日志文件，还是审计输出。

---

## Dynamic Code 不可关联性

### 设计意图

外部观察者拿到两个 Dynamic Code 字面量，**无法判断**它们是否出自同一 Human ID。这是动态码作为 Human ID 公开代理的核心隐私性质。

### 设计依据

Dynamic Code 的派生过程使用：

- **每次新 nonce**：使两次输出在统计意义上独立
- **Issuer 私有 ikm**：使外部观察者无法本地复现派生函数
- **类型前缀 + 时间窗**：使 Dynamic Code 与其他实体在字面量上不混淆，但不引入额外的可关联标记

由此，外部观察者拿到两个 Dynamic Code 字面量，所能做的最强统计推断不超过随机猜测。

> 这是 design 文档中 Property P9 的密码学基础。

---

## 所有权列表查询的访问控制

### 规则

- `listIFayIDsOfHuman(proofOfHuman)` **必须**在 `proofOfHuman` 通过后才返回结果
- 未通过时返回 `HUMAN_ID_OWNERSHIP_NOT_PROVEN`
- Resolver **不提供**"按 Human ID 反查 iFay ID 列表"的匿名接口

### 设计动机

避免外部观察者通过反查接口收集"某个 Human ID 名下有哪些 iFay 人格"的关联信息——这等同于变相暴露 Human ID 的活动画像。

---

## GMC Interface 边界

### 角色定位

GMC Interface 是 FayID System 与 Global Merit Chain（全球信誉链）之间的**唯一通道**。它的设计原则是：

> 只读 + 不可逆 + 不暴露根身份

### 暴露给 GMC 的方法

```
// 只读
gmcLookupOwnership(ifayIDOrCofayID)
  -> { ownerKind: "HUMAN" | "ORGANIZATION",
       ownerOpaqueRef: string }

gmcResolvePublicEntity(idString)
  -> { kind: "IFAY" | "COFAY" | "ORGANIZATION",
       revoked: bool,
       displayMetadata: opaque }
```

### 显式禁止的方向

```
// 不存在 — 协议层禁止反向写入
// gmcWriteHumanID(humanID)
// gmcWriteMnemonic(mnemonic)
// gmcWritePrivateKey(...)
```

> 通过"该方法在 IDL 中不存在"的方式实现禁止，而非运行期校验。这使得反向写入在协议层就不可能发生。

---

## opaqueRef（不可逆引用）

### 派生方式

```
opaqueRef := encode(prefix="gmcref_",
  payload = HKDF(
    ikm    = gmc_namespace_secret,         // FayID System 持有的命名空间密钥
    salt   = humanID,                       // 仅在 FayID 内部出现
    info   = "fayid/gmc/v1",
    length = 256 bit
  )
)
```

### 关键性质

| 性质 | 说明 |
| --- | --- |
| **稳定性** | 同一 Human ID 派生的 opaqueRef 稳定相等，使 GMC 上的信誉记录可长期累积 |
| **抗碰撞** | 不同 Human ID 派生的 opaqueRef 几乎处处不同 |
| **不可反推** | 仅持有 opaqueRef 不可在多项式时间内反推 Human ID |
| **不可关联** | opaqueRef 不参与 Dynamic Code 派生，二者在外部观察者视角不可关联 |

### 设计意义

opaqueRef 解决了一个核心矛盾：

- Global Merit Chain 需要长期、稳定的自然人引用，才能累积信誉
- 自然人的 Human ID 又必须保持隐私，不能在链上暴露

opaqueRef 正是这两者的折中——它对 Human ID 是"单向函数"的输出：稳定可比，但不可反推。

---

## 信誉关联模式

### iFay / coFay / Organization 信誉

- iFay ID、coFay ID、Organization ID 以明文形式作为信誉记录主体
- GMC 可直接以这些 ID 为索引累积信誉

### Human 信誉

- Human ID **不直接出现**在链上
- 当信誉记录需要关联到自然人时，使用该 Human ID 的 opaqueRef 作为引用
- 同一 Human ID 派生的 opaqueRef 在长期跨度内保持稳定

### 边界查询

当 Global Merit Chain 通过 GMC Interface 查询某个 iFay ID 或 coFay ID 的归属时：

- 返回归属主体的类型（HUMAN / ORGANIZATION）
- 在 HUMAN 情形下，返回的是 opaqueRef 而**不是** Human ID 明文
- 在 ORGANIZATION 情形下，返回 Organization ID 明文（Organization 本身就是公开的）

---

## 隐私边界总结

| 边界 | 内侧（FayID System 内部） | 外侧（出站 / GMC / 日志） |
| --- | --- | --- |
| Human ID 明文 | 仅在 Issuer / Resolver 内部出现 | 禁止 |
| Mnemonic | 仅在生成时一次性返回给持有者 | 禁止 |
| Private Key | 仅在密钥派生路径中存在 | 禁止 |
| Dynamic Code | 内部生成 | 允许公开 |
| opaqueRef | 内部派生 | 仅向 GMC 暴露 |

> 隐私不是"加密一下就好"——FayID 通过协议层的硬约束（IDL 不存在写入方法、出站载荷白名单、日志白名单）从结构上消除了 Human ID 明文出现在外部的可能性。
