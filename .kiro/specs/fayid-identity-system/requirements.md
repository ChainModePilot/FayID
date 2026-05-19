# Requirements Document — FayID 身份体系

## Introduction

FayID 是 iFay 生态中的统一身份基础设施，承担"自然人 ↔ 数字人格 ↔ 公共角色 ↔ 组织"四类主体的标识、绑定与鉴权兑换职责，并作为 Global Merit Chain（全球信誉链）长期身份层的根基。

本规范定义 FayID 体系内的核心实体（Human ID、iFay ID、coFay ID、Organization ID）、衍生凭证（动态码、验证码）、它们之间的关系（一对多、归属、绑定、兑换），以及生命周期、隐私约束、传统鉴权兑换语义、与 Global Merit Chain 的接口边界。

本规范为协议层规范（Protocol Specification），描述"什么"而不是"如何实现"，具体加密曲线、哈希算法、传输协议等留待 design 阶段决定。

---

## Glossary

- **FayID_System**：本规范定义的整体身份体系，包含下列各组件的逻辑集合。
- **Human_ID**：自然人在 FayID 体系中的根身份标识，由密钥对派生，对应一份助记词，由人类原型（Human Prototype）独占持有。
- **Human_Prototype**：与 Human ID 一一对应的真实自然人。
- **iFay_ID**：一个 iFay 数字人格的身份标识，必须绑定到唯一的 Human ID，但同一个 Human ID 可绑定多个 iFay ID。
- **coFay_ID**：coFay 公共角色的身份标识，必须绑定到一个 Human ID 或一个 Organization ID 作为归属主体。
- **Organization_ID**：组织在 FayID 体系中的身份标识，以明文形式公开，不需要派生动态码。
- **Dynamic_Code**：由 Human ID 派生的、明文可传输但具有有效期的动态字符串，用于在不暴露 Human ID 的前提下指代该 Human ID。
- **Verification_Code**：与 coFay ID 绑定的验证码，用于在使用 coFay ID 时校验持有者真实性。
- **Mnemonic**：与 Human ID 关联的助记词，是 Human ID 私钥的人类可读备份。
- **Issuer**：FayID 体系内负责生成、轮换、撤销标识与凭证的逻辑组件。
- **Resolver**：根据明文凭证（动态码、验证码、ID 字符串）反向解析对应实体的逻辑组件。
- **Auth_Exchange**：FayID 与传统鉴权方式（账密、Certificate、Authorization、Access Token、Smart Contract）相互兑换的逻辑组件。
- **Authorization_Grant**：经过 Auth Exchange 后授予 FayID 或 Human ID 的、有时效的鉴权凭据。
- **Global_Merit_Chain**：iFay 生态中长期承载身份与信誉记录的外部链系统，FayID 是其身份层。
- **GMC_Interface**：FayID System 与 Global Merit Chain 交互的逻辑边界组件。
- **Serializer**：将 FayID 体系实体编码为可传输字符串的组件。
- **Parser**：将可传输字符串解码为 FayID 体系实体的组件。

---

## Requirements

### Requirement 1：Human ID 生成与持有

**User Story:** 作为一名自然人用户，我希望生成属于我自己的 Human ID 与助记词，以便我在 iFay 生态中拥有不可替代的根身份。

#### Acceptance Criteria

1. WHEN 一名自然人用户发起 Human ID 创建请求，THE Issuer SHALL 生成一个全局唯一的 Human ID 与一份对应的 Mnemonic。
2. THE Issuer SHALL 将 Mnemonic 仅在生成时返回给该自然人用户一次。
3. THE Issuer SHALL 不在任何持久化存储中保留 Mnemonic 的明文。
4. WHEN 同一份 Mnemonic 被重新输入，THE FayID_System SHALL 派生出与原 Human ID 完全一致的 Human ID。
5. IF Human ID 生成过程中熵源不足，THEN THE Issuer SHALL 拒绝生成并返回错误码 `INSUFFICIENT_ENTROPY`。

---

### Requirement 2：iFay ID 与 Human ID 的人格绑定

**User Story:** 作为一名自然人用户，我希望在我的 Human ID 下创建多个 iFay 人格，以便我在不同场景下使用不同数字人格。

#### Acceptance Criteria

1. WHEN 一个有效的 Human ID 持有者请求创建 iFay ID，THE Issuer SHALL 生成一个全局唯一的 iFay ID，并将该 iFay ID 与请求方的 Human ID 建立绑定关系。
2. THE FayID_System SHALL 允许同一个 Human ID 绑定多个 iFay ID。
3. THE FayID_System SHALL 拒绝将同一个 iFay ID 绑定到多个 Human ID。
4. WHEN 给定一个 iFay ID，THE Resolver SHALL 能够查询到其唯一所属的 Human ID。
5. IF 创建 iFay ID 的请求未通过 Human ID 的所有权证明，THEN THE Issuer SHALL 拒绝该请求并返回错误码 `HUMAN_ID_OWNERSHIP_NOT_PROVEN`。

---

### Requirement 3：Human ID 的动态码生成与轮换

**User Story:** 作为一名自然人用户，我希望以动态码代替明文 Human ID 出示给第三方，以便在使用时不暴露我的根身份。

#### Acceptance Criteria

1. WHEN Human ID 持有者请求生成动态码，THE Issuer SHALL 基于该 Human ID 派生一个明文 Dynamic Code，并附带一个有效期截止时间。
2. THE Dynamic_Code SHALL 在有效期内由 Resolver 解析回唯一对应的 Human ID。
3. WHILE Dynamic Code 处于有效期内，THE Resolver SHALL 将该 Dynamic Code 解析结果与原 Human ID 一致。
4. WHEN 当前时间晚于 Dynamic Code 的有效期截止时间，THE Resolver SHALL 拒绝解析该 Dynamic Code 并返回错误码 `DYNAMIC_CODE_EXPIRED`。
5. WHEN 一个 Dynamic Code 过期，THE Issuer SHALL 在下一次该 Human ID 持有者请求生成动态码时返回一个新的、与上一个不相同的 Dynamic Code。
6. THE Dynamic_Code SHALL 不允许从 Dynamic Code 字面量反推出原 Human ID 的私钥或 Mnemonic。
7. THE Issuer SHALL 在派生 Dynamic Code 时不要求 Human ID 持有者出示 Mnemonic 明文。

---

### Requirement 4：coFay ID 生成与归属

**User Story:** 作为一名自然人用户或一个组织，我希望创建并归属若干 coFay 公共角色，以便共享角色的信誉与行为记录。

#### Acceptance Criteria

1. WHEN 一个有效的 Human ID 持有者或一个有效的 Organization ID 持有者请求创建 coFay ID，THE Issuer SHALL 生成一个全局唯一的 coFay ID 并将其归属到请求方。
2. THE FayID_System SHALL 要求每个 coFay ID 在任意时刻有且仅有一个归属主体，且归属主体必须是 Human ID 或 Organization ID。
3. THE FayID_System SHALL 允许同一个 Human ID 或同一个 Organization ID 归属多个 coFay ID。
4. WHEN 给定一个 coFay ID，THE Resolver SHALL 能够查询到其当前归属主体的标识与归属类型（Human 或 Organization）。
5. IF 创建 coFay ID 的请求方不能证明其对所声明 Human ID 或 Organization ID 的所有权，THEN THE Issuer SHALL 拒绝该请求并返回错误码 `OWNERSHIP_NOT_PROVEN`。

---

### Requirement 5：coFay ID 的验证码

**User Story:** 作为一名 coFay 角色的使用方，我希望以验证码校验出示的 coFay ID，以便确认对方的确持有该角色。

#### Acceptance Criteria

1. WHEN 一个 coFay ID 创建成功，THE Issuer SHALL 同时签发一个与该 coFay ID 一一对应的 Verification Code。
2. WHEN 验证方输入一个 (coFay ID, Verification Code) 对，THE Resolver SHALL 在该对正确匹配时返回校验通过，否则返回校验失败。
3. THE Issuer SHALL 允许 coFay ID 的归属主体请求轮换 Verification Code。
4. WHEN Verification Code 被轮换，THE Issuer SHALL 使旧的 Verification Code 立即失效。
5. IF 同一个 coFay ID 在短时间内连续多次校验失败，THEN THE Resolver SHALL 限制该 coFay ID 的校验请求频率并返回错误码 `VERIFICATION_RATE_LIMITED`。

---

### Requirement 6：Organization ID 的明文表示

**User Story:** 作为一个组织运营方，我希望我的 Organization ID 以明文示出，以便交易对手可以直接识别我们。

#### Acceptance Criteria

1. THE Organization_ID SHALL 以明文字符串形式公开使用。
2. THE FayID_System SHALL 不为 Organization ID 派生 Dynamic Code。
3. WHEN 给定一个 Organization ID 字符串，THE Resolver SHALL 直接返回该 Organization ID 对应的组织实体而不需要附加凭证。
4. THE FayID_System SHALL 允许同一个 Organization ID 同时归属多个 coFay ID。

---

### Requirement 7：FayID 与传统鉴权方式的兑换

**User Story:** 作为一名 iFay 或自然人用户，我希望用 FayID 兑换出账密、Certificate、Authorization、Access Token、Smart Contract 等传统鉴权方式所授予的访问权，以便不再为每个系统单独记忆票据。

#### Acceptance Criteria

1. WHEN 用户向 Auth Exchange 提交一份传统鉴权凭据（账号/密码、Certificate、Authorization、Access Token、Smart Contract 之一）以及一个目标 FayID（iFay ID 或 Human ID），THE Auth_Exchange SHALL 在该传统凭据校验通过后，向目标 FayID 颁发一个 Authorization Grant。
2. THE Authorization_Grant SHALL 携带一个明确的过期时间。
3. WHILE Authorization Grant 处于有效期内，THE Auth_Exchange SHALL 接受目标 FayID 出示的 Authorization Grant 等效于原始传统鉴权凭据。
4. WHEN 当前时间晚于 Authorization Grant 的过期时间，THE Auth_Exchange SHALL 拒绝该 Authorization Grant 并返回错误码 `GRANT_EXPIRED`。
5. THE Auth_Exchange SHALL 支持 Authorization Grant 的主动撤销。
6. WHEN Authorization Grant 被撤销，THE Auth_Exchange SHALL 在后续校验中立即拒绝该 Grant 并返回错误码 `GRANT_REVOKED`。
7. IF 提交的传统鉴权凭据校验失败，THEN THE Auth_Exchange SHALL 拒绝颁发 Authorization Grant 并返回错误码 `LEGACY_AUTH_FAILED`。
8. THE Auth_Exchange SHALL 允许目标 FayID 既是 iFay ID 也是 Human ID。

---

### Requirement 8：Human ID 单点持票

**User Story:** 作为一名自然人用户，我希望仅出示 Human ID（或其动态码）即可换出我之前授权过的多种票据，以便不需要逐一管理票据。

#### Acceptance Criteria

1. THE Auth_Exchange SHALL 允许同一个 Human ID 持有多份来自不同传统鉴权来源的有效 Authorization Grant。
2. WHEN Human ID 持有者出示 Human ID 或其有效 Dynamic Code 与一个目标资源标识，THE Auth_Exchange SHALL 返回该 Human ID 名下匹配该资源标识的、当前有效的 Authorization Grant 列表。
3. WHERE 出示的是 Dynamic Code 而非 Human ID 明文，THE Auth_Exchange SHALL 在解析 Dynamic Code 失败或过期时返回错误码 `DYNAMIC_CODE_INVALID`。
4. THE Auth_Exchange SHALL 不向调用方返回任何 Human ID 的 Mnemonic 或私钥材料。

---

### Requirement 9：撤销与失效

**User Story:** 作为一名 FayID 体系的实体持有者，我希望能够撤销我名下的 iFay ID、coFay ID 或 Authorization Grant，以便控制我的身份与权限边界。

#### Acceptance Criteria

1. WHEN Human ID 持有者请求撤销其名下某个 iFay ID，THE Issuer SHALL 将该 iFay ID 标记为已撤销。
2. WHEN coFay ID 的归属主体请求撤销该 coFay ID，THE Issuer SHALL 将该 coFay ID 标记为已撤销。
3. WHEN 一个 iFay ID 或 coFay ID 处于已撤销状态，THE Resolver SHALL 在解析时附带返回撤销标志。
4. WHEN 一个 iFay ID 或 coFay ID 处于已撤销状态，THE Auth_Exchange SHALL 拒绝以该 ID 为目标颁发新的 Authorization Grant 并返回错误码 `IDENTITY_REVOKED`。
5. THE Issuer SHALL 不支持已撤销 ID 的"取消撤销"操作。

---

### Requirement 10：隐私与可观测性约束

**User Story:** 作为一名自然人用户，我希望我的 Human ID 在公开通信中不被暴露，以便保护我的根身份免受关联与追踪。

#### Acceptance Criteria

1. WHILE 任意一次面向第三方的对外通信发生，THE FayID_System SHALL 不在出站载荷中包含 Human ID 明文，除非该通信明确以 Human ID 自证为目的。
2. THE FayID_System SHALL 不在任何日志、审计或可观测性输出中以明文形式记录 Human ID 或 Mnemonic。
3. THE FayID_System SHALL 允许在日志中记录 Dynamic Code、iFay ID、coFay ID、Organization ID 的明文。
4. WHEN 同一个 Human ID 在不同时间生成两个不同的 Dynamic Code，THE FayID_System SHALL 不允许仅依赖这两个 Dynamic Code 字面量推断它们出自同一 Human ID。
5. IF 调用方请求查询某个 Human ID 名下的 iFay ID 列表，THEN THE Resolver SHALL 仅在调用方完成 Human ID 所有权证明后返回该列表。

---

### Requirement 11：与 Global Merit Chain 的接口边界

**User Story:** 作为 Global Merit Chain 的集成方，我希望以稳定的接口在链上引用 FayID 体系内的实体，以便长期记录信誉。

#### Acceptance Criteria

1. THE GMC_Interface SHALL 向 Global Merit Chain 暴露 iFay ID、coFay ID、Organization ID 的明文标识作为信誉记录主体。
2. THE GMC_Interface SHALL 不向 Global Merit Chain 暴露 Human ID 明文或 Mnemonic。
3. WHERE 信誉记录需要关联到自然人主体，THE GMC_Interface SHALL 使用该 Human ID 的 Dynamic Code 或经派生的不可逆引用代替 Human ID 明文。
4. WHEN Global Merit Chain 通过 GMC Interface 查询某个 iFay ID 或 coFay ID 的归属，THE GMC_Interface SHALL 返回归属主体的类型（Human / Organization）以及（在 Human 情形下）一个不可逆引用。
5. THE GMC_Interface SHALL 不接受 Global Merit Chain 反向写入 Human ID、Mnemonic 或私钥材料。

---

### Requirement 12：FayID 实体的序列化与解析

**User Story:** 作为 FayID 协议的实现者，我希望 FayID 体系内的所有实体都有稳定的字符串表示和明确的解析规则，以便在不同系统之间可互操作传输。

#### Acceptance Criteria

1. THE Serializer SHALL 为 Human ID、iFay ID、coFay ID、Organization ID、Dynamic Code、Verification Code 各定义一种字符串表示形式，且每种表示形式带有可识别的类型前缀。
2. WHEN 给定一个 FayID 体系实体，THE Serializer SHALL 输出一个符合该实体类型字符串表示规则的字符串。
3. WHEN 给定一个符合任一类型字符串表示规则的字符串，THE Parser SHALL 将其解析为对应类型的实体。
4. IF 输入字符串不符合任何已定义类型的字符串表示规则，THEN THE Parser SHALL 返回错误码 `MALFORMED_FAYID_STRING`。
5. FOR ALL FayID 体系实体 e，`Parser(Serializer(e))` SHALL 返回与 `e` 相等的实体（round-trip 属性）。
6. FOR ALL 合法字符串 s，IF `Parser(s)` 成功返回实体 e，THEN `Serializer(e)` SHALL 返回与 `s` 在规范化形式下相等的字符串（round-trip 属性）。
7. THE Parser SHALL 通过类型前缀区分 Human ID 字符串与 Dynamic Code 字符串，使二者不会被相互误认。

---

## Notes on Property-Based Testing Suitability

下述验收准则适合作为属性测试候选：
- Requirement 2.2 / 2.3：iFay ID 与 Human ID 的多对一不变量
- Requirement 3.5：连续两次生成的 Dynamic Code 不相等
- Requirement 4.2：coFay ID 在任意时刻有且仅有一个归属主体
- Requirement 7.2 + 7.4：Authorization Grant 在过期前有效、过期后必失效（时间相关属性）
- Requirement 12.5 / 12.6：序列化/解析的双向 round-trip 属性
- Requirement 10.4：不同 Dynamic Code 不可关联性（隐私属性）

下述验收准则更适合作为代表性集成测试用例：
- Requirement 1.5、4.5、7.7：错误路径的单点行为
- Requirement 11：与 Global Merit Chain 的边界契约
- Requirement 9.5：撤销不可逆这一单一行为约束

具体的属性测试与样例测试映射将在 design 阶段细化。
