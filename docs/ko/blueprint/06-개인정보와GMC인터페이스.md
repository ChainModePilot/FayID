# 제 6 장 개인 정보와 GMC 인터페이스

본 장은 FayID 체계의 두 가지 하드 제약을 기술한다: **Human ID는 외부로 나가지 않고 로그에 기록되지 않는다**, 그리고 **GMC Interface의 읽기 전용 경계**. 이 두 제약은 FayID가 Global Merit Chain의 장기 신원 계층 역할을 하기 위한 보안 전제 조건이다.

---

## 프라이버시 하드 제약

### 출항 페이로드 규칙

"출항 페이로드"란 FayID System 외부(제 3자 리소스, 전통 인증 소스, Global Merit Chain, 관측성 백엔드 포함)에서 관찰될 수 있는 임의의 바이트 스트림으로 정의한다.

| 엔티티 | 출항 페이로드에 평문으로 등장 허용 여부 |
| --- | --- |
| **Human ID** | **금지**, 단 해당 통신이 명시적으로 Human ID 자기 증명을 목적으로 하는 경우는 제외 |
| **Mnemonic** | **금지**, 어떠한 예외도 없음 |
| **Private Key** | **금지**, 어떠한 예외도 없음 |
| iFay ID / coFay ID / Organization ID | 허용 |
| Dynamic Code | 허용 |
| Verification Code | coFay ID와 짝지어 등장할 때만, 단독 방송 금지 |
| Authorization Grant | 허용 |

### 로그와 관측성

```
log_allow  := { dynamicCode, ifayID, cofayID, organizationID, grantID, errorCode }
log_deny   := { humanID(평문), mnemonic, privateKey, verificationCode(평문) }
```

구현 측의 로거는 반드시 **화이트리스트 방식**으로 필터링해야 한다: 어떠한 직렬화 경로도 로그에 기록되기 전에 redact 단계를 거쳐야 한다.

> 간단히 말해: Human ID 평문과 Mnemonic은 외부에서 관찰 가능한 바이트 스트림에 절대 등장하지 않는다. 네트워크 패킷이든, 로그 파일이든, 감사 출력이든.

---

## Dynamic Code 비상관성

### 설계 의도

외부 관찰자가 두 Dynamic Code 문자열을 가지고 있어도, 그것들이 동일한 Human ID에서 나왔는지 **판단할 수 없다**. 이는 Human ID의 공개 대리자로서의 Dynamic Code의 핵심 프라이버시 성질이다.

### 설계 근거

Dynamic Code의 파생 과정에는 다음이 사용된다:

- **매번 새로운 nonce**: 두 출력이 통계적 의미에서 독립이 되도록 함
- **Issuer 비공개 ikm**: 외부 관찰자가 로컬에서 파생 함수를 재현할 수 없게 함
- **타입 접두사 + 시간 창**: Dynamic Code가 다른 엔티티와 문자열 상에서 혼동되지 않도록 하면서도 추가적인 상관 가능 표지를 도입하지 않음

이로써 외부 관찰자가 두 Dynamic Code 문자열을 가지고 할 수 있는 가장 강력한 통계적 추론도 무작위 추측을 넘지 못한다.

> 이는 design 문서의 Property P9의 암호학적 기초다.

---

## 소유권 목록 조회 접근 제어

### 규칙

- `listIFayIDsOfHuman(proofOfHuman)`은 `proofOfHuman`이 통과된 후에 **반드시** 결과를 반환한다
- 통과되지 않은 경우 `HUMAN_ID_OWNERSHIP_NOT_PROVEN`을 반환한다
- Resolver는 "Human ID로 iFay ID 목록을 역조회"하는 익명 인터페이스를 **제공하지 않는다**

### 설계 동기

외부 관찰자가 역조회 인터페이스를 통해 "어떤 Human ID 명의에 어떤 iFay 인격들이 있는지"의 연관 정보를 수집하는 것을 방지한다. 이는 변형된 형태로 Human ID의 활동 프로필을 노출시키는 것과 같다.

---

## GMC Interface 경계

### 역할 정의

GMC Interface는 FayID System과 Global Merit Chain(글로벌 메리트 체인) 사이의 **유일한 통로**다. 그 설계 원칙은:

> 읽기 전용 + 불가역 + 루트 신원 미노출

### GMC에 노출되는 메서드

```
// 읽기 전용
gmcLookupOwnership(ifayIDOrCofayID)
  -> { ownerKind: "HUMAN" | "ORGANIZATION",
       ownerOpaqueRef: string }

gmcResolvePublicEntity(idString)
  -> { kind: "IFAY" | "COFAY" | "ORGANIZATION",
       revoked: bool,
       displayMetadata: opaque }
```

### 명시적으로 금지된 방향

```
// 존재하지 않음 — 프로토콜 계층에서 역방향 쓰기 금지
// gmcWriteHumanID(humanID)
// gmcWriteMnemonic(mnemonic)
// gmcWritePrivateKey(...)
```

> "해당 메서드가 IDL에 존재하지 않는다"는 방식으로 금지를 구현하며, 런타임 검증이 아니다. 이를 통해 역방향 쓰기는 프로토콜 계층에서 애초에 발생할 수 없게 된다.

---

## opaqueRef (불가역 참조)

### 파생 방식

```
opaqueRef := encode(prefix="gmcref_",
  payload = HKDF(
    ikm    = gmc_namespace_secret,         // FayID System이 보유하는 네임스페이스 키
    salt   = humanID,                       // FayID 내부에서만 등장
    info   = "fayid/gmc/v1",
    length = 256 bit
  )
)
```

### 핵심 성질

| 성질 | 설명 |
| --- | --- |
| **안정성** | 동일한 Human ID에서 파생된 opaqueRef는 안정적으로 동일하여, GMC 상의 평판 기록을 장기 누적할 수 있다 |
| **충돌 저항성** | 서로 다른 Human ID에서 파생된 opaqueRef는 거의 모든 곳에서 다르다 |
| **불가역추적** | opaqueRef만 보유한 상태에서 다항 시간 내에 Human ID를 역추적할 수 없다 |
| **비상관성** | opaqueRef는 Dynamic Code 파생에 참여하지 않으며, 둘은 외부 관찰자 시점에서 상관 관계가 없다 |

### 설계 의의

opaqueRef는 핵심 모순을 해결한다:

- Global Merit Chain은 평판을 누적하기 위해 자연인에 대한 장기적이고 안정적인 참조를 필요로 한다
- 자연인의 Human ID는 프라이버시를 유지해야 하며 체인 상에 노출될 수 없다

opaqueRef는 정확히 이 둘 사이의 절충점이다. Human ID에 대한 "단방향 함수"의 출력으로서 안정적으로 비교 가능하지만 역추적할 수는 없다.

---

## 평판 연결 모델

### iFay / coFay / Organization 평판

- iFay ID, coFay ID, Organization ID는 평문 형태로 평판 기록 주체가 된다
- GMC는 이 ID들을 인덱스로 직접 사용하여 평판을 누적할 수 있다

### Human 평판

- Human ID는 체인 상에 **직접 등장하지 않는다**
- 평판 기록을 자연인에 연결할 필요가 있을 때, 해당 Human ID의 opaqueRef를 참조로 사용한다
- 동일한 Human ID에서 파생된 opaqueRef는 장기간에 걸쳐 안정적으로 유지된다

### 경계 조회

Global Merit Chain이 GMC Interface를 통해 어떤 iFay ID나 coFay ID의 귀속을 조회할 때:

- 귀속 주체의 타입(HUMAN / ORGANIZATION)을 반환한다
- HUMAN인 경우 반환하는 것은 opaqueRef이며, Human ID 평문이 **아니다**
- ORGANIZATION인 경우 Organization ID 평문을 반환한다 (Organization 자체가 공개적이기 때문)

---

## 프라이버시 경계 요약

| 경계 | 내측 (FayID System 내부) | 외측 (출항 / GMC / 로그) |
| --- | --- | --- |
| Human ID 평문 | Issuer / Resolver 내부에서만 등장 | 금지 |
| Mnemonic | 생성 시 일회성으로 보유자에게 반환 | 금지 |
| Private Key | 키 파생 경로에만 존재 | 금지 |
| Dynamic Code | 내부 생성 | 공개 허용 |
| opaqueRef | 내부 파생 | GMC에만 노출 |

> 프라이버시는 "암호화 한 번으로 끝"나는 것이 아니다. FayID는 프로토콜 계층의 하드 제약(IDL에 쓰기 메서드 부재, 출항 페이로드 화이트리스트, 로그 화이트리스트)을 통해 Human ID 평문이 외부에 등장할 가능성을 구조적으로 제거한다.
