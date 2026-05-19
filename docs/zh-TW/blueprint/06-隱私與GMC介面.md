# 第 6 章 隱私與 GMC 介面

本章描述 FayID 體系的兩條硬約束：**Human ID 不出站不入日誌**，以及 **GMC Interface 的唯讀邊界**。這兩條約束是 FayID 作為 Global Merit Chain 長期身份層的安全前提。

---

## 隱私硬約束

### 出站載荷規則

定義「出站載荷」為：FayID System 之外（包括第三方資源、傳統鑑權來源、Global Merit Chain、可觀測性後端）所能觀察到的任意位元組流。

| 實體 | 是否允許在出站載荷中以明文出現 |
| --- | --- |
| **Human ID** | **禁止**，除非該次通訊明確以 Human ID 自證為目的 |
| **Mnemonic** | **禁止**，無任何例外 |
| **Private Key** | **禁止**，無任何例外 |
| iFay ID / coFay ID / Organization ID | 允許 |
| Dynamic Code | 允許 |
| Verification Code | 僅與 coFay ID 配對出現，不單獨廣播 |
| Authorization Grant | 允許 |

### 日誌與可觀測性

```
log_allow  := { dynamicCode, ifayID, cofayID, organizationID, grantID, errorCode }
log_deny   := { humanID(明文), mnemonic, privateKey, verificationCode(明文) }
```

實作側的日誌器必須以**白名單方式**過濾：任何序列化路徑在寫入日誌前必須經過 redact 步驟。

> 簡言之：Human ID 明文與 Mnemonic 永遠不出現在外部可觀察的位元組流中——無論是網路封包、日誌檔，還是稽核輸出。

---

## Dynamic Code 不可關聯性

### 設計意圖

外部觀察者拿到兩個 Dynamic Code 字面量，**無法判斷**它們是否出自同一 Human ID。這是動態碼作為 Human ID 公開代理的核心隱私性質。

### 設計依據

Dynamic Code 的派生過程使用：

- **每次新 nonce**：使兩次輸出在統計意義上獨立
- **Issuer 私有 ikm**：使外部觀察者無法本地復現派生函式
- **類型前綴 + 時間窗**：使 Dynamic Code 與其他實體在字面量上不混淆，但不引入額外的可關聯標記

由此，外部觀察者拿到兩個 Dynamic Code 字面量，所能做的最強統計推斷不超過隨機猜測。

> 這是 design 文件中 Property P9 的密碼學基礎。

---

## 所有權清單查詢的存取控制

### 規則

- `listIFayIDsOfHuman(proofOfHuman)` **必須**在 `proofOfHuman` 通過後才返回結果
- 未通過時返回 `HUMAN_ID_OWNERSHIP_NOT_PROVEN`
- Resolver **不提供**「按 Human ID 反查 iFay ID 清單」的匿名介面

### 設計動機

避免外部觀察者透過反查介面收集「某個 Human ID 名下有哪些 iFay 人格」的關聯資訊——這等同於變相暴露 Human ID 的活動畫像。

---

## GMC Interface 邊界

### 角色定位

GMC Interface 是 FayID System 與 Global Merit Chain（全球信譽鏈）之間的**唯一通道**。它的設計原則是：

> 唯讀 + 不可逆 + 不暴露根身份

### 暴露給 GMC 的方法

```
// 唯讀
gmcLookupOwnership(ifayIDOrCofayID)
  -> { ownerKind: "HUMAN" | "ORGANIZATION",
       ownerOpaqueRef: string }

gmcResolvePublicEntity(idString)
  -> { kind: "IFAY" | "COFAY" | "ORGANIZATION",
       revoked: bool,
       displayMetadata: opaque }
```

### 顯式禁止的方向

```
// 不存在 — 協定層禁止反向寫入
// gmcWriteHumanID(humanID)
// gmcWriteMnemonic(mnemonic)
// gmcWritePrivateKey(...)
```

> 透過「該方法在 IDL 中不存在」的方式實作禁止，而非執行期校驗。這使得反向寫入在協定層就不可能發生。

---

## opaqueRef（不可逆引用）

### 派生方式

```
opaqueRef := encode(prefix="gmcref_",
  payload = HKDF(
    ikm    = gmc_namespace_secret,         // FayID System 持有的命名空間金鑰
    salt   = humanID,                       // 僅在 FayID 內部出現
    info   = "fayid/gmc/v1",
    length = 256 bit
  )
)
```

### 關鍵性質

| 性質 | 說明 |
| --- | --- |
| **穩定性** | 同一 Human ID 派生的 opaqueRef 穩定相等，使 GMC 上的信譽記錄可長期累積 |
| **抗碰撞** | 不同 Human ID 派生的 opaqueRef 幾乎處處不同 |
| **不可反推** | 僅持有 opaqueRef 不可在多項式時間內反推 Human ID |
| **不可關聯** | opaqueRef 不參與 Dynamic Code 派生，二者在外部觀察者視角不可關聯 |

### 設計意義

opaqueRef 解決了一個核心矛盾：

- Global Merit Chain 需要長期、穩定的自然人引用，才能累積信譽
- 自然人的 Human ID 又必須保持隱私，不能在鏈上暴露

opaqueRef 正是這兩者的折中——它對 Human ID 是「單向函式」的輸出：穩定可比，但不可反推。

---

## 信譽關聯模式

### iFay / coFay / Organization 信譽

- iFay ID、coFay ID、Organization ID 以明文形式作為信譽記錄主體
- GMC 可直接以這些 ID 為索引累積信譽

### Human 信譽

- Human ID **不直接出現**在鏈上
- 當信譽記錄需要關聯到自然人時，使用該 Human ID 的 opaqueRef 作為引用
- 同一 Human ID 派生的 opaqueRef 在長期跨度內保持穩定

### 邊界查詢

當 Global Merit Chain 透過 GMC Interface 查詢某個 iFay ID 或 coFay ID 的歸屬時：

- 返回歸屬主體的類型（HUMAN / ORGANIZATION）
- 在 HUMAN 情形下，返回的是 opaqueRef 而**不是** Human ID 明文
- 在 ORGANIZATION 情形下，返回 Organization ID 明文（Organization 本身就是公開的）

---

## 隱私邊界總結

| 邊界 | 內側（FayID System 內部） | 外側（出站 / GMC / 日誌） |
| --- | --- | --- |
| Human ID 明文 | 僅在 Issuer / Resolver 內部出現 | 禁止 |
| Mnemonic | 僅在生成時一次性返回給持有者 | 禁止 |
| Private Key | 僅在金鑰派生路徑中存在 | 禁止 |
| Dynamic Code | 內部生成 | 允許公開 |
| opaqueRef | 內部派生 | 僅向 GMC 暴露 |

> 隱私不是「加密一下就好」——FayID 透過協定層的硬約束（IDL 不存在寫入方法、出站載荷白名單、日誌白名單）從結構上消除了 Human ID 明文出現在外部的可能性。
