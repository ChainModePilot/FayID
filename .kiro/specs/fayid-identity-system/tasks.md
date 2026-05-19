# Implementation Plan — FayID 身份体系

## Overview

本任务清单把已定型的 `requirements.md`（12 项需求 / 50 余条验收准则）与 `design.md`（13 条 Correctness Properties / 11 个错误码 / 三层测试策略）拆解为可执行的协议规范层产物。目标产物分四类：

1. **简体中文蓝图**（`docs/zh-CN/blueprint/`）：面向中文读者的设计意图与术语介绍。
2. **Schema 演进**（`schema/draft/`）：在 `schema.ts` / `schema.json` / `schema.mdx` 中沉淀 FayID 类型、错误码、接口签名。
3. **协议规范英文主文档**（`docs/en/specification/draft/`）：FayID 协议的规范级英文文本。
4. **Reference Conformance Test Kit**：默认使用 TypeScript + fast-check（与 `schema/draft/schema.ts` 保持一致），将 13 条 Property 与 11 个错误码各自落地为可执行测试。

任务组顺序：先蓝图沉淀语义 → schema 锁定类型 → 英文规范化 → 测试 kit 验证 → 多语言同步（可选）→ Open Questions 归档。

---

## Tasks

- [x] 1. 简体中文蓝图（docs/zh-CN/blueprint/）
  > 蓝图按章分文件落地于 `docs/zh-CN/blueprint/` 目录下，文件名使用两位章号+章名形式（例如 `01-引言.md`）。导览（阅读顺序 / 双锚定）合并写入 `01-引言.md`，不再单独维护索引文件。

  - [x] 1.1 在第 1 章中合并阅读导览
    - 把"读阅顺序、与 requirements.md Glossary / design.md Architecture 的双锚定"合并到 `docs/zh-CN/blueprint/01-引言.md` 的"阅读指引"小节
    - 不再单独创建索引文件（原 `00-FayID蓝图索引.md` 已删除，避免内容重复）
    - _Requirements: 1.1, 2.1, 4.1, 6.1_

  - [x] 1.2 撰写第 1 章 — 引言
    - 新增 `docs/zh-CN/blueprint/01-引言.md`
    - 阐述 FayID 的定位（iFay 生态身份基础设施）、四类主体（Human / iFay / coFay / Organization）、Global Merit Chain 长期目标
    - _Requirements: 1.1_

  - [x] 1.3 撰写第 2 章 — 术语表
    - 新增 `docs/zh-CN/blueprint/02-术语表.md`
    - 直接继承 requirements.md 中 Glossary 的全部条目，并补充 design 文档中新增的术语（opaqueRef / resourceRef / Issuer / Resolver 等）
    - _Requirements: 1.1, 2.1, 4.1, 6.1_

  - [x] 1.4 撰写第 3 章 — 实体与关系
    - 新增 `docs/zh-CN/blueprint/03-实体与关系.md`
    - 落地 Human ID / iFay ID / coFay ID / Organization ID 四类实体的语义、归属规则与多对一 / 一对一关系
    - 嵌入 design 中的 ER 图（mermaid `erDiagram`）
    - _Requirements: 1.1, 2.2, 2.3, 4.2, 4.3, 6.4_
    - _Property: P1_

  - [x] 1.5 撰写第 4 章 — 凭证与生命周期
    - 新增 `docs/zh-CN/blueprint/04-凭证与生命周期.md`
    - 解释 Mnemonic、Dynamic Code、Verification Code、Authorization Grant 四类凭证的产生、有效期、轮换、撤销
    - 嵌入 design 中的三张状态图（Identity / Dynamic Code / Authorization Grant）
    - _Requirements: 1.2, 1.4, 3.1, 3.4, 3.5, 5.1, 5.3, 5.4, 7.2, 7.4, 7.5, 7.6, 9.1, 9.2, 9.3, 9.5_
    - _Property: P3, P4, P5, P6, P8_

  - [x] 1.6 撰写第 5 章 — 鉴权兑换
    - 新增 `docs/zh-CN/blueprint/05-鉴权兑换.md`
    - 落地传统鉴权五类来源（账密 / Certificate / Authorization / Access Token / Smart Contract）、Authorization Grant 兑换流程、`listGrantsOfHuman` 单点持票
    - 嵌入 design 中的兑换 / 单点持票 / 撤销三张序列图
    - _Requirements: 7.1, 7.3, 7.8, 8.1, 8.2, 8.3, 8.4_
    - _Property: P6, P7_

  - [x] 1.7 撰写第 6 章 — 隐私与 GMC 接口
    - 新增 `docs/zh-CN/blueprint/06-隐私与GMC接口.md`
    - 落地 Human ID 不出站不入日志的硬约束、出站载荷与日志白名单、Dynamic Code 不可关联性
    - 落地 GMC Interface 的只读边界、`opaqueRef` 派生概念、写入方向禁止
    - _Requirements: 10.1, 10.2, 10.3, 10.4, 11.1, 11.2, 11.3, 11.4, 11.5_
    - _Property: P9, P10_

  - [x] 1.8 撰写第 7 章 — Open Questions
    - 新增 `docs/zh-CN/blueprint/07-未决议题.md`
    - 复述 design 第 10 节列出的 10 条 Open Questions（具体算法 / 时间窗 / 限速阈值 / Human ID 撤销 / 跨实例互操作 / namespace_secret 轮换等），并指向后续任务 8 中的归档文件
    - _Requirements: 1.4_

- [ ] 2. Schema/Draft 类型与接口骨架
  - [ ] 2.1 在 `schema/draft/schema.ts` 中新增 FayID 实体类型定义
    - 定义 `HumanID` / `IFayID` / `CoFayID` / `OrganizationID` / `DynamicCode` / `VerificationCode` / `AuthorizationGrant` 七类 TypeScript 类型，字段与 design 中"Core Entities & Data Models"章节一致
    - 同时定义共享的 `OwnerKind`、`LegacySourceKind`、`GrantState` 枚举
    - _Requirements: 1.1, 2.1, 4.1, 5.1, 6.1, 7.1, 7.2_
    - _Property: P1_

  - [ ] 2.2 在 `schema/draft/schema.ts` 中新增类型前缀常量与字符串模式
    - 落地 `hid_` / `ifay_` / `cofay_` / `org_` / `dyn_` / `vrf_` / `grt_` 七个前缀常量与每类的字符集 / 长度下界
    - 添加 `FAYID_PREFIXES` 联合类型与 `EntityKind` 枚举
    - _Requirements: 12.1, 12.2, 12.7_
    - _Property: P13_

  - [ ] 2.3 在 `schema/draft/schema.ts` 中新增 11 个错误码常量
    - 沉淀 `INSUFFICIENT_ENTROPY` / `HUMAN_ID_OWNERSHIP_NOT_PROVEN` / `OWNERSHIP_NOT_PROVEN` / `DYNAMIC_CODE_EXPIRED` / `DYNAMIC_CODE_INVALID` / `VERIFICATION_RATE_LIMITED` / `IDENTITY_REVOKED` / `LEGACY_AUTH_FAILED` / `GRANT_EXPIRED` / `GRANT_REVOKED` / `MALFORMED_FAYID_STRING` 为字符串字面量联合类型 + 常量对象
    - _Requirements: 1.5, 2.5, 3.4, 4.5, 5.5, 7.4, 7.6, 7.7, 8.3, 9.4, 12.4_

  - [ ] 2.4 在 `schema/draft/schema.ts` 中新增四个组件接口签名
    - 落地 `Issuer` / `Resolver` / `AuthExchange` / `GMCInterface` 接口（IDL 形式），方法签名与 design 中"Components and Interfaces"小节一致
    - 显式不声明 `gmcWriteHumanID` / `gmcWriteMnemonic` / `gmcWritePrivateKey` 等被禁方向（通过"不存在"实现约束）
    - _Requirements: 1.1, 2.1, 3.1, 4.1, 5.2, 7.1, 8.2, 11.1, 11.5_
    - _Property: P9_

  - [ ] 2.5 同步刷新 `schema/draft/schema.json` 与 `schema/draft/schema.mdx`
    - JSON Schema：把上述类型与错误码以 `$defs` 方式落地，保证字段名与 `schema.ts` 一致
    - MDX：补充 FayID 章节，引用 schema.ts 类型块作为参考来源
    - _Requirements: 12.1, 12.2_

- [ ] 3. 协议规范英文主文档（docs/en/specification/draft/）
  - [ ] 3.1 创建 `fayid.mdx` 与 Overview / Architecture 章节
    - 新建 `docs/en/specification/draft/fayid.mdx`，包含 Front-matter、目录、Overview、Architecture（含 design 中的 `flowchart LR` 组件交互图与信任边界表）
    - _Requirements: 1.1, 11.1_

  - [ ] 3.2 撰写 Core Entities & Identifier Format 章节
    - 落地七类实体的字段表 + ER 图 + 类型前缀表 + 字符集 / 长度下界 + `normalize` 规则
    - _Requirements: 1.1, 2.1, 2.4, 4.1, 4.4, 5.1, 6.1, 12.1, 12.2, 12.3, 12.7_
    - _Property: P1, P11, P12, P13_

  - [ ] 3.3 撰写 Cryptographic Primitives & Lifecycle 章节
    - 落地 Human ID / Dynamic Code / Verification Code / opaqueRef 四个派生过程的伪代码与抽象性质
    - 嵌入 Identity / Dynamic Code / Authorization Grant / Verification Code 四张状态图
    - _Requirements: 1.4, 1.5, 3.1, 3.2, 3.4, 3.5, 3.6, 3.7, 5.4, 7.4, 7.6, 9.1, 9.2, 9.3, 9.5_
    - _Property: P2, P3, P4, P5, P6, P8, P10_

  - [ ] 3.4 撰写 Auth Exchange Protocol & Privacy & GMC Interface 章节
    - 落地兑换 / 单点持票 / 撤销三段 sequence diagram、resourceRef 命名空间约定、出站载荷允许 / 禁止表、GMC IDL 与不可逆引用派生
    - _Requirements: 7.1, 7.3, 7.5, 7.7, 7.8, 8.1, 8.2, 8.3, 8.4, 10.1, 10.2, 10.3, 10.4, 10.5, 11.1, 11.2, 11.3, 11.4, 11.5_
    - _Property: P6, P7, P9, P10_

  - [ ] 3.5 撰写 Error Codes 集中表与传播原则
    - 完整列出 11 个错误码、触发场景、来源组件与关联需求
    - 添加错误传播原则四条（稳定字符串、不携带敏感字段、同类错误码一致、限速错误的最小信息）
    - _Requirements: 1.5, 2.5, 3.4, 4.5, 5.5, 7.4, 7.6, 7.7, 8.3, 9.4, 10.2, 12.4_

  - [ ] 3.6 撰写 Correctness Properties & Open Questions 章节
    - 把 design 中 13 条 Property 的标题、全称量化形式、validates 列表整段迁入
    - 把 design 中 10 条 Open Questions 整段迁入并标注"非规范性附录"
    - _Requirements: 12.5, 12.6_
    - _Property: P1, P2, P3, P4, P5, P6, P7, P8, P9, P10, P11, P12, P13_

- [ ] 4. 协议层规范 / Schema / 文档自洽性检查点
  - 同步检查 `requirements.md` / `design.md` / `schema/draft/*` / `docs/en/specification/draft/fayid.mdx` / `docs/zh-CN/blueprint/fayid.md` 五处的字段名、错误码、Property 编号是否完全一致
  - Ensure all tests pass, ask the user if questions arise.
  - _Requirements: 12.1_

- [ ] 5. Reference Conformance Test Kit（默认 TypeScript + fast-check）
  - [ ] 5.1 创建 testkit 目录骨架与依赖配置
    - 新建 `schema/draft/testkit/`（或同级独立目录），落地 `package.json` / `tsconfig.json` / `vitest.config.ts` / `README.md`，固定 `fast-check` 与 `vitest` 依赖；任务开始时与用户再次确认语言与库版本
    - _Requirements: 12.1_

  - [ ] 5.2 实现实体生成器与 Serializer / Parser 参考实现
    - 提供 7 类实体的 `fast-check` Arbitrary 生成器与对应的 `serialize` / `parse` 参考函数
    - 同时提供"非法字符串"反向生成器（错误前缀 / 越界字符 / 长度极端）
    - _Requirements: 12.1, 12.2, 12.3, 12.4, 12.7_
    - _Property: P11, P12, P13_

  - [ ] 5.3 PBT — Property 1：标识创建唯一性 + 归属一致性
    - 在 testkit 中新增 `properties/p01-creation-uniqueness.spec.ts`，对随机操作序列断言全局唯一与归属一致
    - _Requirements: 1.1, 2.1, 2.2, 2.3, 2.4, 4.1, 4.2, 4.3, 4.4, 5.1_
    - _Property: P1_

  - [ ] 5.4 PBT — Property 2：Mnemonic 确定性派生 Human ID
    - 在 testkit 中新增 `properties/p02-mnemonic-determinism.spec.ts`
    - _Requirements: 1.4_
    - _Property: P2_

  - [ ] 5.5 PBT — Property 3：Dynamic Code 解析与时窗
    - 在 testkit 中新增 `properties/p03-dynamic-code-window.spec.ts`，使用可控时钟覆盖 `[issuedAt, expiresAt]` 与 `> expiresAt` 两段
    - _Requirements: 3.1, 3.2, 3.3, 3.4_
    - _Property: P3_

  - [ ] 5.6 PBT — Property 4：Dynamic Code 不可碰撞
    - 在 testkit 中新增 `properties/p04-dynamic-code-uniqueness.spec.ts`
    - _Requirements: 3.5_
    - _Property: P4_

  - [ ] 5.7 PBT — Property 5：Verification Code 单版本有效
    - 在 testkit 中新增 `properties/p05-verification-code-versioning.spec.ts`
    - _Requirements: 5.2, 5.4_
    - _Property: P5_

  - [ ] 5.8 PBT — Property 6：Authorization Grant 状态机一致性
    - 在 testkit 中新增 `properties/p06-grant-state-machine.spec.ts`，基于操作序列模型 + 可控时钟
    - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.6, 7.8, 9.4_
    - _Property: P6_

  - [ ] 5.9 PBT — Property 7：listGrantsOfHuman 等价于过滤
    - 在 testkit 中新增 `properties/p07-list-grants-equivalence.spec.ts`，覆盖 Human ID 明文 / 有效 Dynamic Code / 无效 Dynamic Code 三种 case
    - _Requirements: 8.1, 8.2, 8.3_
    - _Property: P7_

  - [ ] 5.10 PBT — Property 8：撤销单调性
    - 在 testkit 中新增 `properties/p08-revocation-monotonicity.spec.ts`
    - _Requirements: 9.1, 9.2, 9.3, 9.5_
    - _Property: P8_

  - [ ] 5.11 PBT — Property 9：Human ID 不出站不入日志
    - 在 testkit 中新增 `properties/p09-human-id-non-disclosure.spec.ts`，并实现可注入的出站载荷拦截器与日志拦截器作为 harness
    - _Requirements: 10.1, 10.2, 11.2_
    - _Property: P9_

  - [ ] 5.12 PBT — Property 10：GMC opaqueRef 稳定且不可逆推
    - 在 testkit 中新增 `properties/p10-gmc-opaque-ref.spec.ts`，覆盖确定性 + 抗碰撞
    - _Requirements: 11.3, 11.4_
    - _Property: P10_

  - [ ] 5.13 PBT — Property 11：Round-trip 实体侧
    - 在 testkit 中新增 `properties/p11-roundtrip-entity.spec.ts`
    - _Requirements: 12.5_
    - _Property: P11_

  - [ ] 5.14 PBT — Property 12：Round-trip 字符串侧
    - 在 testkit 中新增 `properties/p12-roundtrip-string.spec.ts`
    - _Requirements: 12.6_
    - _Property: P12_

  - [ ] 5.15 PBT — Property 13：类型前缀不混淆 + 非法字符串拒绝
    - 在 testkit 中新增 `properties/p13-prefix-disambiguation.spec.ts`
    - _Requirements: 12.2, 12.3, 12.4, 12.7_
    - _Property: P13_

  - [ ] 5.16 错误码样例测试集中归档
    - 在 testkit 中新增 `samples/error-codes.spec.ts`，为 11 个错误码每个提供至少一个正向触发样例（含 `INSUFFICIENT_ENTROPY` / `HUMAN_ID_OWNERSHIP_NOT_PROVEN` / `OWNERSHIP_NOT_PROVEN` / `DYNAMIC_CODE_EXPIRED` / `DYNAMIC_CODE_INVALID` / `VERIFICATION_RATE_LIMITED` / `IDENTITY_REVOKED` / `LEGACY_AUTH_FAILED` / `GRANT_EXPIRED` / `GRANT_REVOKED` / `MALFORMED_FAYID_STRING`）
    - _Requirements: 1.5, 2.5, 3.4, 4.5, 5.5, 7.4, 7.6, 7.7, 8.3, 9.4, 12.4_

  - [ ] 5.17 GMC 接口契约测试
    - 在 testkit 中新增 `samples/gmc-interface.spec.ts`，通过类型层 / 反射断言 `gmcWriteHumanID` / `gmcWriteMnemonic` / `gmcWritePrivateKey` 不存在；正向断言 `gmcLookupOwnership` / `gmcResolvePublicEntity` 不返回 Human ID 明文
    - _Requirements: 11.1, 11.2, 11.4, 11.5_
    - _Property: P9, P10_

  - [ ] 5.18 限速与时间相关样例测试
    - 在 testkit 中新增 `samples/rate-limit.spec.ts`，基于可控时钟与受控失败注入覆盖 `VERIFICATION_RATE_LIMITED` 的窗口与退避字段最小信息
    - _Requirements: 5.5, 10.2_

  - [ ] 5.19 不可属性化条款的样例测试集
    - 在 testkit 中新增 `samples/non-pbt-coverage.spec.ts`，集中覆盖 design Testing Strategy 中点名"不被 PBT 覆盖"的条款（1.2, 1.3, 2.5, 3.7, 6.1, 6.2, 6.3, 7.5, 8.4, 10.3, 10.5, 11.1, 11.5, 12.1）
    - _Requirements: 1.2, 1.3, 2.5, 3.7, 6.1, 6.2, 6.3, 7.5, 8.4, 10.3, 10.5, 11.1, 11.5, 12.1_

- [ ] 6. Conformance Test Kit 检查点
  - 全量运行 PBT 与样例测试，记录每条 Property 的随机种子与样本数（≥ 100）；输出一份覆盖矩阵（Property × Requirement）作为 testkit `README.md` 附录
  - Ensure all tests pass, ask the user if questions arise.
  - _Requirements: 12.5, 12.6_

- [ ] 7. 多语言文档同步
  - [ ] 7.1* 翻译 `fayid.mdx` 至 `docs/zh-TW/specification/draft/`
    - 保持章节结构与 mermaid 图原样，仅本地化文字
    - _Requirements: 12.1_

  - [ ] 7.2* 翻译 `fayid.mdx` 至 `docs/ja/specification/draft/`
    - _Requirements: 12.1_

  - [ ] 7.3* 翻译 `fayid.mdx` 至 `docs/ko/specification/draft/`
    - _Requirements: 12.1_

  - [ ] 7.4* 翻译 `fayid.mdx` 至 `docs/es/`、`docs/fr/`、`docs/de/`、`docs/ru/` 四个目录的 `specification/draft/`
    - 可分批交付；每个目标语言独立可验收
    - _Requirements: 12.1_

- [ ] 8. Open Questions 归档
  - 在 `.kiro/specs/fayid-identity-system/open-questions.md` 中归档 design 第 10 节列出的 10 条议题，每条标注 owner / 影响范围 / 解决里程碑（v1 / v2 / 实现 spec / ADR），并把 design 与 spec 主文档中的相应链接指向该归档
  - _Requirements: 1.4, 3.1, 5.5, 7.2, 9.5, 11.3, 11.4, 12.1_

## Notes

- 标 `*` 的子任务为可选，可在需要更快交付时跳过；其余子任务均需实现。
- 所有任务的"产物"列在第一行的简短描述中：要么是某个具体文件的新增 / 修改，要么是某段章节 / 某个测试文件的落地。
- 13 条 Correctness Property 与 11 个错误码均有专属任务覆盖，详见任务 5.3–5.15、5.16。
- GMC Interface 契约由任务 2.4（接口签名层面）+ 5.17（运行期 / 类型层断言）联合覆盖。
- Conformance test kit 默认语言为 TypeScript + fast-check，与 `schema/draft/schema.ts` 一致；任务 5.1 开始时如需调整，再与用户确认。
- 本工作流到任务清单完成为止；执行各任务请打开本文件并点击对应 "Start task"。
