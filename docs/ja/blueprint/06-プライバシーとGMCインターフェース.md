# 第 6 章 プライバシーと GMC インターフェース

本章は FayID 体系における 2 つのハード制約について述べる。すなわち、**Human ID は出方向に流出せず、ログにも記録されない**こと、および **GMC Interface の読み取り専用境界**である。これら 2 つの制約は、FayID が Global Merit Chain の長期 ID 層として機能するための安全前提である。

---

## プライバシーのハード制約

### 出方向ペイロードのルール

「出方向ペイロード」とは、FayID System の外部（第三者リソース、従来型認証ソース、Global Merit Chain、可観測性バックエンドを含む）が観測しうる任意のバイト列を指す。

| エンティティ | 出方向ペイロードに平文で出現することを許容するか |
| --- | --- |
| **Human ID** | **禁止**。ただし、当該通信が明確に Human ID 自身を証明することを目的とする場合を除く |
| **Mnemonic** | **禁止**、いかなる例外もない |
| **Private Key** | **禁止**、いかなる例外もない |
| iFay ID / coFay ID / Organization ID | 許容 |
| Dynamic Code | 許容 |
| Verification Code | coFay ID とペアでのみ出現し、単独でブロードキャストしない |
| Authorization Grant | 許容 |

### ログと可観測性

```
log_allow  := { dynamicCode, ifayID, cofayID, organizationID, grantID, errorCode }
log_deny   := { humanID(平文), mnemonic, privateKey, verificationCode(平文) }
```

実装側のロガーは**ホワイトリスト方式**でフィルタリングしなければならない。すなわち、いかなるシリアライズ経路もログ書き込み前に redact ステップを通過する必要がある。

> 端的に言えば、Human ID 平文と Mnemonic は外部観測可能なバイト列に決して出現しない——ネットワークパケットであれ、ログファイルであれ、監査出力であれ。

---

## Dynamic Code の関連付け不能性

### 設計意図

外部観察者が 2 つの Dynamic Code 文字列を入手しても、それらが同一の Human ID から出たものか**判定できない**。これは Dynamic Code を Human ID の公開プロキシとして用いる際の中核的なプライバシー性質である。

### 設計根拠

Dynamic Code の派生プロセスは次を用いる。

- **毎回新規の nonce**：2 回の出力を統計的に独立にする
- **Issuer の私的 ikm**：外部観察者がローカルに派生関数を再現できないようにする
- **型プレフィックス + 時間ウィンドウ**：Dynamic Code が他のエンティティと文字列上で混同されないようにするが、追加の関連付け可能なマーカーを導入しない

これにより、外部観察者が 2 つの Dynamic Code 文字列を得たときに行える最強の統計的推定はランダム推測を超えない。

> これは design 文書における Property P9 の暗号学的基礎である。

---

## 所有権リストクエリのアクセス制御

### ルール

- `listIFayIDsOfHuman(proofOfHuman)` は `proofOfHuman` の検証通過後に**のみ**結果を返さなければならない
- 通過しない場合は `HUMAN_ID_OWNERSHIP_NOT_PROVEN` を返す
- Resolver は「Human ID から iFay ID リストを逆引き」する匿名インターフェースを**提供しない**

### 設計動機

外部観察者が逆引きインターフェースを通じて「ある Human ID 名義下にどのような iFay 人格があるか」という関連付け情報を収集することを回避する——これは Human ID のアクティビティプロファイルを変則的に露出することと等しい。

---

## GMC Interface の境界

### 役割

GMC Interface は FayID System と Global Merit Chain（全球信用チェーン）の間の**唯一のチャネル**である。設計原則は次のとおり。

> 読み取り専用 + 不可逆 + 根 ID を露出しない

### GMC に公開するメソッド

```
// 読み取り専用
gmcLookupOwnership(ifayIDOrCofayID)
  -> { ownerKind: "HUMAN" | "ORGANIZATION",
       ownerOpaqueRef: string }

gmcResolvePublicEntity(idString)
  -> { kind: "IFAY" | "COFAY" | "ORGANIZATION",
       revoked: bool,
       displayMetadata: opaque }
```

### 明示的に禁じる方向

```
// 存在しない — プロトコル層で逆方向書き込みを禁止
// gmcWriteHumanID(humanID)
// gmcWriteMnemonic(mnemonic)
// gmcWritePrivateKey(...)
```

> 「該当メソッドが IDL に存在しない」という形で禁止を実現するのであり、実行時検証ではない。これにより、逆方向書き込みはプロトコル層で発生不可能となる。

---

## opaqueRef（不可逆参照）

### 派生方式

```
opaqueRef := encode(prefix="gmcref_",
  payload = HKDF(
    ikm    = gmc_namespace_secret,         // FayID System が保持する名前空間鍵
    salt   = humanID,                       // FayID 内部にのみ出現
    info   = "fayid/gmc/v1",
    length = 256 bit
  )
)
```

### 主要性質

| 性質 | 説明 |
| --- | --- |
| **安定性** | 同じ Human ID から派生する opaqueRef は安定的に同一であり、GMC 上の信用記録が長期にわたり累積可能となる |
| **耐衝突性** | 異なる Human ID から派生する opaqueRef はほぼいたるところで異なる |
| **逆推不能** | opaqueRef のみを保持しても、多項式時間で Human ID を逆推することはできない |
| **関連付け不能** | opaqueRef は Dynamic Code の派生に関与せず、両者は外部観察者の視点から関連付け不能である |

### 設計上の意義

opaqueRef は次の中核的矛盾を解決する。

- Global Merit Chain は信用を累積するため、長期かつ安定した自然人参照を必要とする
- 一方、自然人の Human ID はプライバシーを保つ必要があり、チェーン上で露出してはならない

opaqueRef はこの両者の折衷である——Human ID に対する「一方向関数」の出力として、安定的に比較可能であるが逆推不能である。

---

## 信用関連付けのパターン

### iFay／coFay／Organization の信用

- iFay ID、coFay ID、Organization ID は平文形式で信用記録の主体となる
- GMC はこれらの ID をインデックスとして直接信用を累積できる

### Human の信用

- Human ID は**チェーン上に直接出現しない**
- 信用記録を自然人に関連付ける必要がある場合、当該 Human ID の opaqueRef を参照として使用する
- 同一の Human ID から派生する opaqueRef は長期にわたって安定を維持する

### 境界クエリ

Global Merit Chain が GMC Interface 経由である iFay ID または coFay ID の帰属を照会するとき、

- 帰属主体の種別（HUMAN／ORGANIZATION）を返却する
- HUMAN の場合、返却するのは opaqueRef であり、Human ID 平文では**ない**
- ORGANIZATION の場合、Organization ID 平文を返却する（Organization 自身がもとより公開のため）

---

## プライバシー境界の総括

| 境界 | 内側（FayID System 内部） | 外側（出方向／GMC／ログ） |
| --- | --- | --- |
| Human ID 平文 | Issuer／Resolver 内部にのみ出現 | 禁止 |
| Mnemonic | 生成時に保持者へ 1 度のみ返却 | 禁止 |
| Private Key | 鍵派生経路にのみ存在 | 禁止 |
| Dynamic Code | 内部生成 | 公開許可 |
| opaqueRef | 内部派生 | GMC へのみ露出 |

> プライバシーは「暗号化すれば済む」ものではない。FayID はプロトコル層のハード制約（IDL に書き込みメソッドが存在しない、出方向ペイロードのホワイトリスト、ログのホワイトリスト）により、Human ID 平文が外部に出現する可能性を構造的に排除している。
