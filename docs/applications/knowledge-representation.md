# 지식 표현

## 개요

**지식 표현(Knowledge Representation, KR)**은 인공지능에서 지식을 컴퓨터가 사용할 수 있는 형태로 표현하는 분야입니다.

!!! quote "Davis, Shrobe, & Szolovits (1993)"
    "지식 표현은 다섯 가지 역할을 한다:
    대리물(surrogate), 존재론적 개입, 단편화 이론, 계산 매개, 인간 표현 매체"

## 지식 표현 방식

### 1. 의미망 (Semantic Network)

```
          [동물]
            │
        is-a│
            ▼
          [새]──── has ────→[날개]
            │                  │
        is-a│              part-of
            ▼                  │
         [참새]◄───────────────┘
            │
        instance-of
            ▼
        [영희의 참새]
```

### 2. 프레임 (Frame)

```python
class Frame:
    """Minsky의 프레임 표현"""

    bird_frame = {
        "name": "새",
        "is_a": "동물",
        "slots": {
            "날개": {
                "value": 2,
                "type": "integer",
                "default": 2
            },
            "can_fly": {
                "value": True,
                "type": "boolean",
                "default": True,
                "if_needed": lambda: check_wing_condition()
            },
            "habitat": {
                "type": "string",
                "facets": ["forest", "urban", "wetland"]
            }
        },
        "methods": {
            "fly": lambda self: "날아갑니다" if self.can_fly else "못 날아요"
        }
    }
```

### 3. 논리 기반 표현

#### 명제 논리

```
명제:
P: "비가 온다"
Q: "우산을 가져간다"

규칙: P → Q (비가 오면 우산을 가져간다)
사실: P (비가 온다)
추론: Q (따라서 우산을 가져간다)
```

#### 술어 논리 (First-Order Logic)

```prolog
% 지식 베이스
human(socrates).
mortal(X) :- human(X).

% 쿼리
?- mortal(socrates).
% 결과: true

% 더 복잡한 예
parent(tom, mary).
parent(tom, jack).
parent(mary, ann).

grandparent(X, Z) :- parent(X, Y), parent(Y, Z).

?- grandparent(tom, ann).
% 결과: true
```

### 4. 규칙 기반 시스템

```python
class RuleBasedSystem:
    """규칙 기반 추론 엔진"""

    def __init__(self):
        self.facts = set()
        self.rules = []

    def add_fact(self, fact: str):
        self.facts.add(fact)

    def add_rule(self, conditions: list, conclusion: str):
        self.rules.append({
            "if": conditions,
            "then": conclusion
        })

    def forward_chain(self):
        """전방향 연쇄 추론"""
        changed = True
        while changed:
            changed = False
            for rule in self.rules:
                if all(c in self.facts for c in rule["if"]):
                    if rule["then"] not in self.facts:
                        self.facts.add(rule["then"])
                        changed = True
        return self.facts

# 예시
engine = RuleBasedSystem()
engine.add_fact("has_feathers")
engine.add_fact("can_fly")
engine.add_rule(["has_feathers", "can_fly"], "is_bird")
engine.add_rule(["is_bird", "small"], "is_sparrow")
engine.add_fact("small")

result = engine.forward_chain()
# {'has_feathers', 'can_fly', 'is_bird', 'small', 'is_sparrow'}
```

## 지식 그래프

### 개념

!!! info "지식 그래프란?"
    실세계의 개체(entities)와 그들 간의 관계(relationships)를 그래프 구조로 표현한 지식 베이스입니다.

### 대표적인 지식 그래프

| 지식 그래프 | 개발 | 트리플 수 | 특징 |
|------------|------|----------|------|
| Google Knowledge Graph | Google | 수십억 | 검색 향상 |
| Wikidata | Wikimedia | 10억+ | 오픈 데이터 |
| DBpedia | Community | 5억+ | Wikipedia 기반 |
| YAGO | Max Planck | 1억+ | 학술 연구 |

### 지식 그래프 구축

```python
import networkx as nx

class KnowledgeGraph:
    """간단한 지식 그래프 구현"""

    def __init__(self):
        self.graph = nx.DiGraph()

    def add_entity(self, entity: str, entity_type: str):
        self.graph.add_node(entity, type=entity_type)

    def add_relation(self, subject: str, predicate: str, obj: str):
        self.graph.add_edge(subject, obj, relation=predicate)

    def query(self, subject: str = None, predicate: str = None, obj: str = None):
        """트리플 패턴 매칭"""
        results = []
        for u, v, data in self.graph.edges(data=True):
            if (subject is None or u == subject) and \
               (predicate is None or data['relation'] == predicate) and \
               (obj is None or v == obj):
                results.append((u, data['relation'], v))
        return results

# 사용 예시
kg = KnowledgeGraph()
kg.add_entity("Einstein", "Person")
kg.add_entity("Physics", "Field")
kg.add_entity("Germany", "Country")

kg.add_relation("Einstein", "field", "Physics")
kg.add_relation("Einstein", "birthPlace", "Germany")
kg.add_relation("Einstein", "wonAward", "Nobel Prize")
```

## 임베딩과 분산 표현

### 지식 그래프 임베딩

```python
# TransE 개념적 설명
class TransE:
    """
    TransE: h + r ≈ t

    h: 주어 임베딩
    r: 관계 임베딩
    t: 목적어 임베딩

    관계를 번역(translation)으로 모델링
    """

    def __init__(self, embedding_dim: int):
        self.dim = embedding_dim
        self.entity_embeddings = {}
        self.relation_embeddings = {}

    def score(self, h, r, t):
        """트리플의 점수 계산 (낮을수록 좋음)"""
        h_emb = self.entity_embeddings[h]
        r_emb = self.relation_embeddings[r]
        t_emb = self.entity_embeddings[t]

        return np.linalg.norm(h_emb + r_emb - t_emb)
```

### 다양한 임베딩 방법

| 방법 | 수식 | 특징 |
|------|------|------|
| TransE | h + r ≈ t | 단순, 효과적 |
| TransR | h·Mr + r ≈ t·Mr | 관계별 공간 |
| DistMult | <h, r, t> | 대칭 관계 |
| ComplEx | Re(<h, r, t̄>) | 비대칭 관계 |
| RotatE | h ∘ r ≈ t | 회전 변환 |

## 추론 시스템

### 연역적 추론

```
전제 1: 모든 사람은 죽는다. ∀x(Human(x) → Mortal(x))
전제 2: 소크라테스는 사람이다. Human(Socrates)
결론: 소크라테스는 죽는다. Mortal(Socrates)
```

### 귀납적 추론

```
관찰 1: 참새는 날 수 있다.
관찰 2: 독수리는 날 수 있다.
관찰 3: 비둘기는 날 수 있다.
귀납적 결론: 새는 날 수 있다.
(예외: 펭귄, 타조 등)
```

### 귀추적 추론 (Abduction)

```
규칙: 비가 오면 땅이 젖는다.
관찰: 땅이 젖어 있다.
가설: 비가 왔을 것이다. (최선의 설명)
```

## 상식 추론

### 문제점

!!! warning "상식의 어려움"
    - 상식 지식은 양이 방대함
    - 암묵적이고 명시화하기 어려움
    - 예외가 많음 (비단조 추론)

### 프로젝트들

| 프로젝트 | 목표 |
|----------|------|
| CYC | 상식 지식 수동 입력 |
| ConceptNet | 크라우드소싱 상식 |
| ATOMIC | 인과/의도 지식 |
| COMET | 신경망 상식 생성 |

## 응용 분야

### 질의응답 (QA)

```
사용자: "아인슈타인의 출생지는?"

지식 그래프 쿼리:
SELECT ?place WHERE {
  :Einstein :birthPlace ?place .
}

답변: "독일 울름"
```

### 추천 시스템

```python
def recommend_by_kg(user, kg):
    """지식 그래프 기반 추천"""
    # 사용자가 좋아한 항목들
    liked = kg.query(subject=user, predicate="likes")

    # 유사한 항목 찾기 (같은 카테고리, 같은 제작자 등)
    recommendations = []
    for item in liked:
        similar = kg.query(subject=item, predicate="similarTo")
        recommendations.extend(similar)

    return recommendations
```

### 대화 시스템

지식 기반 대화를 통한 일관되고 정확한 응답 생성

## 미래 방향

!!! tip "발전 방향"
    1. **신경-기호 통합**: 딥러닝 + 기호적 추론
    2. **자동 지식 획득**: 텍스트에서 자동 추출
    3. **동적 지식 갱신**: 실시간 지식 업데이트
    4. **설명 가능성**: 추론 과정 설명

---

이것으로 존재론 가이드의 현대적 응용 섹션을 마칩니다.

[홈으로 돌아가기](../index.md)
