# 컴퓨터 과학에서의 존재론

## 개요

**형식 존재론(Formal Ontology)**은 컴퓨터 과학, 특히 인공지능과 지식 관리에서 중요한 역할을 합니다.

!!! info "컴퓨터 과학에서의 정의"
    존재론은 특정 도메인에서 **개념들의 명시적 명세화(explicit specification of conceptualization)**입니다.
    — Tom Gruber (1993)

## 온톨로지의 구성 요소

### 기본 요소

```
온톨로지의 구조
├── 클래스 (Classes)
│   └── 개념, 범주
├── 인스턴스 (Instances)
│   └── 개별 객체
├── 속성 (Properties)
│   ├── 데이터 속성
│   └── 객체 속성
├── 관계 (Relations)
│   └── 클래스/인스턴스 간 연결
└── 공리 (Axioms)
    └── 제약 조건, 규칙
```

### 예시: 동물 온톨로지

```python
# 온톨로지 구조 예시
class Ontology:
    def __init__(self):
        self.classes = {
            "Animal": {
                "subclasses": ["Mammal", "Bird", "Fish"],
                "properties": ["hasAge", "hasHabitat"]
            },
            "Mammal": {
                "subclasses": ["Dog", "Cat", "Human"],
                "properties": ["hasHairColor"]
            }
        }

        self.instances = {
            "buddy": {
                "type": "Dog",
                "properties": {
                    "hasAge": 5,
                    "hasHairColor": "golden"
                }
            }
        }

        self.relations = {
            "owns": ("Human", "Animal"),
            "livesIn": ("Animal", "Habitat")
        }
```

## OWL (Web Ontology Language)

### OWL 개요

**OWL**은 시맨틱 웹을 위한 표준 온톨로지 언어입니다.

| OWL 버전 | 특징 | 사용 사례 |
|----------|------|----------|
| OWL Lite | 단순, 분류 계층 | 간단한 분류 |
| OWL DL | 결정 가능, 표현력 | 대부분의 응용 |
| OWL Full | 최대 표현력 | 복잡한 모델링 |

### OWL 예시

```xml
<?xml version="1.0"?>
<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
         xmlns:owl="http://www.w3.org/2002/07/owl#"
         xmlns:rdfs="http://www.w3.org/2000/01/rdf-schema#">

  <!-- 클래스 정의 -->
  <owl:Class rdf:about="#Animal"/>

  <owl:Class rdf:about="#Mammal">
    <rdfs:subClassOf rdf:resource="#Animal"/>
  </owl:Class>

  <!-- 속성 정의 -->
  <owl:DatatypeProperty rdf:about="#hasAge">
    <rdfs:domain rdf:resource="#Animal"/>
    <rdfs:range rdf:resource="http://www.w3.org/2001/XMLSchema#integer"/>
  </owl:DatatypeProperty>

  <!-- 인스턴스 정의 -->
  <Mammal rdf:about="#buddy">
    <hasAge>5</hasAge>
  </Mammal>

</rdf:RDF>
```

## RDF와 트리플

### RDF 트리플

모든 지식은 **주어-서술어-목적어** 트리플로 표현:

```
<subject> <predicate> <object>

예시:
<buddy> <isA> <Dog>
<buddy> <hasAge> "5"^^xsd:integer
<buddy> <ownedBy> <Alice>
```

### RDF 그래프

```
     ┌──────────────┐
     │    Alice     │
     └──────┬───────┘
            │ owns
            ▼
     ┌──────────────┐
     │    buddy     │──── hasAge ───→ 5
     └──────┬───────┘
            │ isA
            ▼
     ┌──────────────┐
     │     Dog      │
     └──────┬───────┘
            │ subClassOf
            ▼
     ┌──────────────┐
     │    Mammal    │
     └──────────────┘
```

## 상위 온톨로지 (Upper Ontology)

### 대표적인 상위 온톨로지

!!! tip "상위 온톨로지란?"
    특정 도메인에 국한되지 않고, 모든 도메인에 공통적으로 적용되는 일반 개념을 정의하는 온톨로지입니다.

| 온톨로지 | 개발 기관 | 특징 |
|----------|-----------|------|
| SUMO | IEEE | 표준화, 대규모 |
| DOLCE | LOA | 인지적 편향 |
| BFO | Barry Smith | 과학적 응용 |
| Cyc | Cycorp | 상식 지식 |

### BFO (Basic Formal Ontology) 구조

```
Entity
├── Continuant (지속체)
│   ├── Independent Continuant
│   │   ├── Material Entity
│   │   └── Immaterial Entity
│   └── Dependent Continuant
│       ├── Quality
│       └── Realizable Entity
│
└── Occurrent (발생체)
    ├── Process
    ├── Process Boundary
    └── Spatiotemporal Region
```

## 도메인 온톨로지

### 의료 분야

**SNOMED CT** 예시:

```
질병 (Disease)
├── 감염성 질병 (Infectious disease)
│   ├── 바이러스성 (Viral)
│   │   └── COVID-19
│   └── 세균성 (Bacterial)
├── 종양성 질병 (Neoplastic disease)
└── 대사 질환 (Metabolic disease)
```

### 생명과학 분야

**Gene Ontology (GO)**:

- **분자 기능 (Molecular Function)**
- **생물학적 과정 (Biological Process)**
- **세포 구성요소 (Cellular Component)**

## 온톨로지 추론

### Description Logic

```python
class DescriptionLogic:
    """기술 논리 기반 추론"""

    def subsumption(self, class_a, class_b) -> bool:
        """클래스 포함 관계 추론"""
        # A ⊑ B: A의 모든 인스턴스가 B의 인스턴스인가?
        pass

    def satisfiability(self, concept) -> bool:
        """개념 만족 가능성 검사"""
        # 이 개념의 인스턴스가 존재할 수 있는가?
        pass

    def instance_checking(self, individual, concept) -> bool:
        """인스턴스 검사"""
        # 이 개체가 이 개념의 인스턴스인가?
        pass
```

### SPARQL 쿼리

```sparql
# 모든 포유류 찾기
SELECT ?animal ?name
WHERE {
  ?animal rdf:type :Mammal .
  ?animal :hasName ?name .
}

# 특정 조건의 동물 찾기
SELECT ?animal
WHERE {
  ?animal rdf:type :Dog .
  ?animal :hasAge ?age .
  FILTER (?age > 3)
}
```

## 온톨로지 도구

### 개발 도구

| 도구 | 용도 | 특징 |
|------|------|------|
| Protégé | 온톨로지 편집 | Stanford, 무료 |
| TopBraid | 기업용 | 상용, 강력한 기능 |
| OntoGraf | 시각화 | Protégé 플러그인 |

### 추론 엔진

| 엔진 | 특징 |
|------|------|
| Pellet | OWL DL 추론 |
| HermiT | 효율적 OWL 추론 |
| FaCT++ | 빠른 분류 |
| RDFox | 고성능 RDF |

## 다음 단계

온톨로지의 지식 표현 응용을 더 알아보려면 [지식 표현](knowledge-representation.md)으로 이동하세요.
