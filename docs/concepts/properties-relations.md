# 속성과 관계

## 속성 (Properties)

### 속성이란?

**속성(Property)**은 사물이 가지는 특성이나 성질입니다.

!!! example "속성의 예"
    - 빨간 사과: "빨강"은 사과의 속성
    - 무거운 돌: "무거움"은 돌의 속성
    - 지혜로운 사람: "지혜로움"은 사람의 속성

### 속성의 존재론적 지위

속성은 실재하는가? 이 물음에 대한 다양한 입장:

```
속성에 대한 입장
├── 실재론 (Realism)
│   ├── 플라톤적 실재론: 속성은 독립적으로 존재
│   └── 아리스토텔레스적 실재론: 속성은 개물 안에 존재
│
├── 유명론 (Nominalism)
│   ├── 술어 유명론: 속성은 언어적 표현일 뿐
│   └── 집합 유명론: 속성은 개물들의 집합
│
└── 트로프 이론 (Trope Theory)
    └── 속성은 개별화된 특수자
```

### 본질적 속성 vs 우연적 속성

| 구분 | 본질적 속성 | 우연적 속성 |
|------|-------------|-------------|
| 정의 | 그것 없이는 그 사물이 아닌 것 | 있어도 되고 없어도 되는 것 |
| 필연성 | 필연적 | 우연적 |
| 예시 | 인간의 이성 | 소크라테스의 대머리 |
| 변화 | 변화 불가 | 변화 가능 |

```python
class Entity:
    def __init__(self, name: str, essential: set, accidental: set):
        self.name = name
        self.essential = essential  # 본질적 속성
        self.accidental = accidental  # 우연적 속성

    def could_lack(self, property: str) -> bool:
        """이 속성 없이도 같은 것일 수 있는가?"""
        return property in self.accidental

    def identity_requires(self, property: str) -> bool:
        """이 속성이 정체성에 필수인가?"""
        return property in self.essential

# 예시
socrates = Entity(
    "소크라테스",
    essential={"인간임", "이성적임"},
    accidental={"대머리임", "아테네 출신"}
)
```

### 내재적 속성 vs 외재적 속성

!!! info "구분"
    - **내재적(intrinsic)**: 그 사물 자체만으로 결정됨
    - **외재적(extrinsic)**: 다른 것과의 관계에서 결정됨

예시:

| 내재적 속성 | 외재적 속성 |
|------------|------------|
| 질량 | 무게 (중력장 의존) |
| 모양 | 위치 |
| 화학적 조성 | 가격 |

## 관계 (Relations)

### 관계란?

**관계(Relation)**는 둘 이상의 사물 사이에 성립하는 연결입니다.

```
관계의 유형 (항수별)
├── 1항 관계: 속성 (x는 빨갛다)
├── 2항 관계: 이항 관계 (x는 y보다 크다)
├── 3항 관계: 삼항 관계 (x는 y와 z 사이에 있다)
└── n항 관계: 다항 관계
```

### 관계의 존재론적 문제

!!! question "브래들리의 역설 (Bradley's Regress)"
    관계 R이 a와 b를 연결한다면, R과 a를 연결하는 관계 R'이 필요하고,
    R'과 R을 연결하는 관계 R''이 필요하고... (무한 퇴행)

해결 시도들:

1. **관계 실재론**: 관계는 그 자체로 연결 기능을 함
2. **관계 환원주의**: 관계는 속성으로 환원됨
3. **트로프 관계론**: 관계는 개별화된 관계-트로프

### 관계의 특성

```python
from typing import TypeVar, Callable

T = TypeVar('T')

class Relation:
    """관계의 논리적 특성"""

    @staticmethod
    def reflexive(R: Callable[[T, T], bool], domain: set) -> bool:
        """반사적: 모든 x에 대해 R(x, x)"""
        return all(R(x, x) for x in domain)

    @staticmethod
    def symmetric(R: Callable[[T, T], bool], domain: set) -> bool:
        """대칭적: R(x, y)이면 R(y, x)"""
        return all(
            R(x, y) == R(y, x)
            for x in domain for y in domain
        )

    @staticmethod
    def transitive(R: Callable[[T, T], bool], domain: set) -> bool:
        """추이적: R(x, y)이고 R(y, z)이면 R(x, z)"""
        return all(
            (not R(x, y) or not R(y, z) or R(x, z))
            for x in domain for y in domain for z in domain
        )
```

| 관계 유형 | 반사적 | 대칭적 | 추이적 | 예시 |
|----------|--------|--------|--------|------|
| 동일성 | ✓ | ✓ | ✓ | = |
| 동치관계 | ✓ | ✓ | ✓ | 합동 |
| 순서관계 | ✓ | ✗ | ✓ | ≤ |
| 엄밀순서 | ✗ | ✗ | ✓ | < |

### 내적 관계 vs 외적 관계

!!! tip "중요한 구분"
    - **내적 관계**: 관계항의 속성에 의해 필연적으로 결정
    - **외적 관계**: 관계항의 속성과 독립적

예시:

| 내적 관계 | 외적 관계 |
|----------|----------|
| "3은 2보다 크다" | "A는 B 옆에 있다" |
| "빨강은 주황과 유사하다" | "철수는 영희를 안다" |

## 보편자 문제

### 하나와 여럿의 문제

!!! question "핵심 물음"
    여러 사물이 같은 속성을 가질 때, 그 "같은 것"은 무엇인가?

    - 이 사과는 빨갛다
    - 저 소방차도 빨갛다
    - "빨강"은 무엇인가?

### 주요 입장

```
보편자 논쟁
│
├── 극단적 실재론
│   └── 보편자는 개물과 독립적으로 존재 (플라톤)
│
├── 온건한 실재론
│   └── 보편자는 개물 안에 존재 (아리스토텔레스)
│
├── 개념론
│   └── 보편자는 마음의 개념 (로크)
│
└── 유명론
    └── 보편자는 이름일 뿐 (오컴)
```

## 다음 단계

서양 존재론의 전통을 더 자세히 알아보려면 [서양 존재론](../traditions/western.md)으로 이동하세요.
