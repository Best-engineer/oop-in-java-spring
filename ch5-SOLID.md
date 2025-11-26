




당신이 제시한 예:

> 하나의 필드가
> 
> - 토지(land)일 때는 면적(area),
> - 건물(building)일 때는 층수(floors)
>     
>     를 의미한다.
>     

이건 **하나의 속성이 여러 개의 역할을 가진 심각한 설계 문제**예요.

---

# ❌ 현재 구조의 문제점

### 1) SRP 위반

속성 1개가 "두 가지 의미"를 가짐 → 단일 책임 X.

### 2) 조건문(if) 난발

프로그래머가 계속 이렇게 작성해야 함:

```java
if (property.type == LAND) {
    // area 로 해석
} else if (property.type == BUILDING) {
    // floor 로 해석
}

```

### 3) Null, unused field 증가

토지는 floors가 null

건물은 area가 null

→ 정규화 위반

### 4) 확장성 부족

나중에 “입주자 수”, “건폐율” 등 타입별 속성이 늘어날수록 모델이 무너짐.

---

# 🔥 어떻게 정규화하고 SRP를 지킬까?

## 🎯 핵심 원칙

> "서로 다른 의미를 가진 데이터는 절대 같은 필드에 담지 않는다."
> 

따라서 **테이블과 객체를 분리**해야 함.

---

# ✅ 해결 방법 1: 테이블을 분리한다 (정규화)

## 🔹 property(공통 테이블)

| id | type | name |
| --- | --- | --- |
| 1 | LAND | “서울 토지 A” |
| 2 | BUILDING | “강남빌딩 B” |

## 🔹 land_property (토지 상세)

| property_id | area |
| --- | --- |
| 1 | 200㎡ |

## 🔹 building_property (건물 상세)

| property_id | floors |
| --- | --- |
| 2 | 30 |

---

## ✔ 자바 모델링 구조도 함께 분리

### 🔹 공통 인터페이스

```java
interface Property {
    String getName();
}

```

### 🔹 토지 클래스

```java
class LandProperty implements Property {
    private String name;
    private double area;

    // getters/setters
}

```

### 🔹 건물 클래스

```java
class BuildingProperty implements Property {
    private String name;
    private int floors;

    // getters/setters
}

```

### 피하는 코드:

```java
int data = property.value;
// 이게 land인지 building인지 if문 없이 알 수 없음 → 나쁜 설계

```

이제는 아래처럼 폴리모피즘 활용 가능:

```java
Property p = repository.find(...);

if (p instanceof LandProperty land) {
    System.out.println(land.getArea());
} else if (p instanceof BuildingProperty building) {
    System.out.println(building.getFloors());
}

```

또는 더 나아가, 비즈니스 로직에서 type 검사 자체가 필요 없는 구조도 가능함.

---

# 👍 해결 방법 2: 단일 테이블 + JSON(파생 속성) → 추천 X

하나의 속성을 JSON으로 저장하는 방식도 있으나

SRP 원칙을 침해할 수 있어서 일반적인 엔터프라이즈 모델에서는 추천하지 않음.

---

# 🚀 해결 방법 3: 상속 구조(OO 모델링)

자바 객체 설계만 보면 다음 구조도 가능:

```
Property
 ├─ LandProperty
 └─ BuildingProperty

```

여기에 공통 로직은 상위 클래스(Property)에 두고,

타입별 로직은 각각에게 책임을 부여.

---

# 🧠 왜 이게 단일 책임 원칙에 맞는가?

- LandProperty는 “토지의 면적 관리”라는 한 가지 책임만 가진다.
- BuildingProperty는 “건물의 층수 관리”라는 한 가지 책임만 가진다.
- Property는 “공통 속성 관리”라는 단일 책임만 가진다.

→ 서로 다른 의미의 데이터가 섞이지 않음

→ type마다 다른 조건문 로직이 줄어듦

→ 확장 시 코드 안정성 증가

---

# 🎯 결론 요약

## ❌ 잘못된 설계

하나의 필드가 여러 의미를 가짐

→ 조건문 남발

→ SRP 위반

→ 정규화 위반

## ✅ 올바른 설계

속성이 다르면 테이블/객체도 분리

→ *property + land_property + building_property*

→ 자바에서는 상속 or 인터페이스로 분리

→ if문 제거, 타입별 책임 분리

---

원하시면

✔ 이 구조로 Spring JPA에서 사용하는 실제 코드 예제

✔ OOP 다형성을 활용해 if문을 완전히 제거하는 고급 구조

까지도 만들어 드릴게요!

### 나의 말:

위에서 설계한 테이블 정규화를 좀 더 자세하게 알려주ㅓ

### ChatGPT의 말:

좋아요, 아까 이야기한 **토지/건물 예시**를 가지고

“DB 정규화 관점에서” 좀 더 깊게 풀어서 설명해볼게요.

---

## 1. 문제 상황 다시 정리

처음 설계(안 좋은 설계):

```
property
---------------------------
id (PK)
type         -- 'LAND' 또는 'BUILDING'
name
value        -- 토지면 적일 때는 면적, 건물일 때는 층수

```

- `type = 'LAND'` → `value` = 면적(㎡)
- `type = 'BUILDING'` → `value` = 층수

### 이게 왜 문제냐?

1. **하나의 컬럼(value)이 여러 의미를 가짐**
    - 어떤 행은 면적
    - 어떤 행은 층수
        
        → **단일 책임 원칙 위반 + 의미가 불명확**
        
2. **데이터 타입도 사실 다름**
    - 면적: `DOUBLE` 또는 `DECIMAL`
    - 층수: `INT`
        
        → 억지로 하나로 묶기 때문에 타입도 왜곡됨
        
3. **조회/로직이 항상 if/else 의존**
    
    ```sql
    SELECT
      CASE
        WHEN type = 'LAND' THEN CONCAT(value, '㎡')
        WHEN type = 'BUILDING' THEN CONCAT(value, '층')
      END AS info
    FROM property;
    
    ```
    
    자바 코드도 계속:
    
    ```java
    if (property.getType() == LAND) { ... } else { ... }
    
    ```
    

---

## 2. 정규화의 목표: “서로 다른 의미는 분리하자”

여기서 적용되는 핵심 아이디어:

> “하나의 속성(컬럼)은 항상 한 가지 의미만 가져야 한다.”
> 

그래서 **“공통 부분”과 “타입별 상세”를 분리**하는 정규화가 필요합니다.

---

## 3. 정규화된 테이블 설계

### 3-1. 1단계: 공통 속성 테이블 `property`

먼저, **토지든 건물이든 공통으로 가지는 정보만** 모읍니다.

```
property
---------------------------------
id (PK)
type          -- 'LAND', 'BUILDING', ...
name          -- 이름
address       -- 주소(있다면)
...

```

- 이 테이블은 “부동산(Property)라는 개체 자체”를 나타냅니다.
- 아직 **토지만의 정보, 건물만의 정보는 넣지 않습니다.**

---

### 3-2. 2단계: 토지 전용 테이블 `land_property`

```
land_property
---------------------------------
property_id (PK, FK -> property.id)
area                -- 면적(㎡)
...

```

- `property_id`는 `property.id`를 참조하는 **1:1 관계**.
- 이 레코드는 “이 property는 토지이고, 그 토지의 면적은 얼마다”를 표현.
- 이 테이블에는 **토지에만 해당하는 속성**만 둡니다.

---

### 3-3. 3단계: 건물 전용 테이블 `building_property`

```
building_property
---------------------------------
property_id (PK, FK -> property.id)
floors              -- 층수
...

```

- 역시 `property_id`가 `property.id`를 참조.
- 이 테이블에는 **건물에만 해당하는 속성**만 둡니다.

---

## 4. 이 구조가 왜 “정규화”에 부합하는가?

### 1) 제1정규형(1NF) – 컬럼은 더 이상 쪼개지지 않는 단일 의미

- 예전 설계에서는 `value` 컬럼의 의미가 두 개(면적/층수) → 애매.
- 정규화 후:
    - `land_property.area` → 항상 면적
    - `building_property.floors` → 항상 층수
        
        → 각 컬럼이 **한 가지 의미만** 가짐 → 1NF에 부합.
        

### 2) 제2, 제3 정규형 – 부분 함수 종속/이행적 종속 제거

- “토지의 면적”이라는 속성은 **토지라는 개념에만 종속**되어야 함.
- 지금은:
    - `property`는 “부동산 전체”에 대한 테이블
    - 면적은 그 중 “토지라는 subtype”에만 해당
- 그래서 “전체 개체(Property)” 테이블에서 쓸데없이 면적/층수 등을 관리하지 않고
    
    **각 subtype 테이블로 분리하면서 종속관계를 명확하게 분리**한 것.
    

단순화해서 말하면:

> “토지 정보는 토지 테이블에, 건물 정보는 건물 테이블에”
> 
> 
> 각자 자기 주인에게 가게 한 것이 2NF/3NF 관점에서 올바름.
> 

---

## 5. 실제 SQL DDL 예시

```sql
CREATE TABLE property (
    id          BIGINT PRIMARY KEY,
    type        VARCHAR(20) NOT NULL, -- 'LAND', 'BUILDING', ...
    name        VARCHAR(100) NOT NULL,
    address     VARCHAR(255)
);

CREATE TABLE land_property (
    property_id BIGINT PRIMARY KEY,
    area        DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (property_id) REFERENCES property(id)
);

CREATE TABLE building_property (
    property_id BIGINT PRIMARY KEY,
    floors      INT NOT NULL,
    FOREIGN KEY (property_id) REFERENCES property(id)
);

```

---

## 6. 조회 예시

### 6-1. 토지 목록 조회

```sql
SELECT p.id, p.name, l.area
FROM property p
JOIN land_property l ON p.id = l.property_id
WHERE p.type = 'LAND';

```

### 6-2. 건물 목록 조회

```sql
SELECT p.id, p.name, b.floors
FROM property p
JOIN building_property b ON p.id = b.property_id
WHERE p.type = 'BUILDING';

```

여기서 중요한 점:

- 이제 **같은 컬럼을 두고 “이건 상황에 따라 의미가 다름”이라고 해석할 필요가 없음.**
- 왜냐하면
    - `area`는 언제나 면적
    - `floors`는 언제나 층수로만 사용되니까.

---

## 7. 자바 코드 관점에서의 장점

정규화된 DB 설계는 자바 코드(도메인 모델)에도 그대로 반영됩니다.

```java
class Property {
    Long id;
    PropertyType type;
    String name;
    String address;
}

class LandProperty {
    Long propertyId;
    double area;
}

class BuildingProperty {
    Long propertyId;
    int floors;
}

```

- 더 이상 `int value;` 같은 이상한 필드가 없음.
- “이 값이 뭘 의미하는지” if/else로 해석할 필요가 없음.
- 각 클래스가 **자기 타입에 맞는 책임만 가짐** → 단일 책임 원칙도 만족.

---

## 8. “단일 테이블로 놔두고 싶으면?” (비추천 설계와 비교)

가끔 이렇게 하고 싶은 유혹이 있음:

```
property
---------------------------------
id
type          -- 'LAND', 'BUILDING'
name
area          -- 토지일 때만 사용, 건물일 때는 NULL
floors        -- 건물일 때만 사용, 토지일 때는 NULL

```

단순히 nullable 컬럼으로 해결하는 방식.

### 이 방식의 문제

- NULL이 잔뜩 생김 → “이 컬럼은 어떤 행에서는 의미 없음”
- 타입이 늘어날수록(예: 상가, 공장, 아파트…) 컬럼이 폭증
- 정규화 관점에서 “타입별로 분리할 수 있는 정보가 한 테이블에 다 섞여있다”는 문제

그래서 **정석적인 설계는 subtype 테이블 분리**입니다.

지금 공부하시는 **단일책임 원칙 + 정규화 학습** 관점에서도,

토지와 건물을 나누는 지금 예시는 **슈퍼타입/서브타입 모델링의 전형적인 예**라고 보면 됩니다.

---

## 정리 한 번만 더

1. **하나의 컬럼에 여러 의미를 넣는 설계** → 정규화 위반 + SRP 위반
2. 공통 속성은 `property` 테이블로 묶고
3. 타입별 속성은 `land_property`, `building_property` 같은 **별도 테이블**로 분리
4. 각 테이블/컬럼은 **한 가지 의미만** 갖게 설계
5. 자바 코드에서도 각 클래스가 한 가지 책임만 가지도록 자연스럽게 연결

---


