# 도메인 모델링

## 개요

슈퍼마켓 온톨로지의 **클래스(Classes)**, **속성(Properties)**, **관계(Relations)**를 체계적으로 설계합니다.

## 클래스 계층 구조

### 상위 온톨로지 연결

BFO(Basic Formal Ontology)를 상위 온톨로지로 사용:

```
owl:Thing
└── bfo:Entity
    ├── bfo:Continuant (지속체)
    │   ├── bfo:IndependentContinuant
    │   │   ├── bfo:MaterialEntity
    │   │   │   ├── sm:Product          # 상품
    │   │   │   ├── sm:Store            # 매장
    │   │   │   ├── sm:Equipment        # 장비
    │   │   │   └── sm:Packaging        # 포장재
    │   │   └── bfo:Object
    │   │       ├── sm:Person
    │   │       │   ├── sm:Customer     # 고객
    │   │       │   ├── sm:Employee     # 직원
    │   │       │   └── sm:Supplier     # 공급자
    │   │       └── sm:Organization     # 조직
    │   │
    │   └── bfo:DependentContinuant
    │       ├── sm:Price               # 가격
    │       ├── sm:Quality             # 품질
    │       └── sm:Quantity            # 수량
    │
    └── bfo:Occurrent (발생체)
        ├── sm:Transaction             # 거래
        ├── sm:Promotion               # 프로모션
        └── sm:InventoryEvent          # 재고 이벤트

* sm = supermarket namespace
```

## 핵심 클래스 정의

### 1. Product (상품)

```turtle
@prefix sm: <http://example.org/supermarket#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

sm:Product a owl:Class ;
    rdfs:label "상품"@ko, "Product"@en ;
    rdfs:comment "슈퍼마켓에서 판매하는 모든 상품"@ko .

# 상품 하위 분류
sm:FreshProduct a owl:Class ;
    rdfs:subClassOf sm:Product ;
    rdfs:label "신선식품"@ko .

sm:ProcessedFood a owl:Class ;
    rdfs:subClassOf sm:Product ;
    rdfs:label "가공식품"@ko .

sm:Beverage a owl:Class ;
    rdfs:subClassOf sm:Product ;
    rdfs:label "음료"@ko .

sm:HouseholdItem a owl:Class ;
    rdfs:subClassOf sm:Product ;
    rdfs:label "생활용품"@ko .

sm:PersonalCare a owl:Class ;
    rdfs:subClassOf sm:Product ;
    rdfs:label "개인위생용품"@ko .
```

### 상품 분류 체계 (Taxonomy)

```
sm:Product
├── sm:FreshProduct (신선식품)
│   ├── sm:Vegetable (채소)
│   │   ├── sm:LeafyVegetable (엽채류)
│   │   ├── sm:RootVegetable (근채류)
│   │   └── sm:FruitVegetable (과채류)
│   ├── sm:Fruit (과일)
│   │   ├── sm:DomesticFruit (국산과일)
│   │   └── sm:ImportedFruit (수입과일)
│   ├── sm:Meat (육류)
│   │   ├── sm:Beef (소고기)
│   │   ├── sm:Pork (돼지고기)
│   │   └── sm:Chicken (닭고기)
│   ├── sm:Seafood (수산물)
│   └── sm:Dairy (유제품)
│
├── sm:ProcessedFood (가공식품)
│   ├── sm:CannedFood (통조림)
│   ├── sm:FrozenFood (냉동식품)
│   ├── sm:Snack (스낵)
│   ├── sm:Noodle (면류)
│   └── sm:Sauce (소스/양념)
│
├── sm:Beverage (음료)
│   ├── sm:Water (생수)
│   ├── sm:Juice (주스)
│   ├── sm:SoftDrink (탄산음료)
│   ├── sm:Coffee (커피)
│   └── sm:AlcoholicBeverage (주류)
│
└── sm:NonFood (비식품)
    ├── sm:HouseholdItem (생활용품)
    └── sm:PersonalCare (개인위생용품)
```

### 2. Customer (고객)

```turtle
sm:Customer a owl:Class ;
    rdfs:subClassOf sm:Person ;
    rdfs:label "고객"@ko .

# 고객 세그먼트
sm:PremiumCustomer a owl:Class ;
    rdfs:subClassOf sm:Customer ;
    owl:equivalentClass [
        a owl:Restriction ;
        owl:onProperty sm:totalPurchaseAmount ;
        owl:minInclusive "1000000"^^xsd:integer
    ] ;
    rdfs:label "프리미엄 고객"@ko ;
    rdfs:comment "연간 구매액 100만원 이상 고객"@ko .

sm:RegularCustomer a owl:Class ;
    rdfs:subClassOf sm:Customer ;
    rdfs:label "일반 고객"@ko .

sm:NewCustomer a owl:Class ;
    rdfs:subClassOf sm:Customer ;
    rdfs:label "신규 고객"@ko .
```

### 3. Transaction (거래)

```turtle
sm:Transaction a owl:Class ;
    rdfs:label "거래"@ko ;
    rdfs:comment "구매 거래 기록"@ko .

sm:Purchase a owl:Class ;
    rdfs:subClassOf sm:Transaction ;
    rdfs:label "구매"@ko .

sm:Return a owl:Class ;
    rdfs:subClassOf sm:Transaction ;
    rdfs:label "반품"@ko .

sm:Exchange a owl:Class ;
    rdfs:subClassOf sm:Transaction ;
    rdfs:label "교환"@ko .
```

## 속성 정의

### 데이터 속성 (Datatype Properties)

```turtle
# === 상품 속성 ===
sm:productName a owl:DatatypeProperty ;
    rdfs:domain sm:Product ;
    rdfs:range xsd:string ;
    rdfs:label "상품명"@ko .

sm:barcode a owl:DatatypeProperty ;
    rdfs:domain sm:Product ;
    rdfs:range xsd:string ;
    rdfs:label "바코드"@ko .

sm:price a owl:DatatypeProperty ;
    rdfs:domain sm:Product ;
    rdfs:range xsd:decimal ;
    rdfs:label "가격"@ko .

sm:expiryDate a owl:DatatypeProperty ;
    rdfs:domain sm:Product ;
    rdfs:range xsd:date ;
    rdfs:label "유통기한"@ko .

sm:stockQuantity a owl:DatatypeProperty ;
    rdfs:domain sm:Product ;
    rdfs:range xsd:integer ;
    rdfs:label "재고수량"@ko .

sm:calories a owl:DatatypeProperty ;
    rdfs:domain sm:Product ;
    rdfs:range xsd:integer ;
    rdfs:label "칼로리"@ko .

sm:allergenInfo a owl:DatatypeProperty ;
    rdfs:domain sm:Product ;
    rdfs:range xsd:string ;
    rdfs:label "알레르기 정보"@ko .

# === 고객 속성 ===
sm:customerName a owl:DatatypeProperty ;
    rdfs:domain sm:Customer ;
    rdfs:range xsd:string ;
    rdfs:label "고객명"@ko .

sm:membershipLevel a owl:DatatypeProperty ;
    rdfs:domain sm:Customer ;
    rdfs:range xsd:string ;
    rdfs:label "멤버십 등급"@ko .

sm:totalPurchaseAmount a owl:DatatypeProperty ;
    rdfs:domain sm:Customer ;
    rdfs:range xsd:decimal ;
    rdfs:label "총 구매금액"@ko .

sm:registrationDate a owl:DatatypeProperty ;
    rdfs:domain sm:Customer ;
    rdfs:range xsd:date ;
    rdfs:label "가입일"@ko .

# === 거래 속성 ===
sm:transactionDate a owl:DatatypeProperty ;
    rdfs:domain sm:Transaction ;
    rdfs:range xsd:dateTime ;
    rdfs:label "거래일시"@ko .

sm:totalAmount a owl:DatatypeProperty ;
    rdfs:domain sm:Transaction ;
    rdfs:range xsd:decimal ;
    rdfs:label "총 금액"@ko .

sm:paymentMethod a owl:DatatypeProperty ;
    rdfs:domain sm:Transaction ;
    rdfs:range xsd:string ;
    rdfs:label "결제수단"@ko .
```

### 객체 속성 (Object Properties)

```turtle
# === 상품 관계 ===
sm:belongsToCategory a owl:ObjectProperty ;
    rdfs:domain sm:Product ;
    rdfs:range sm:Category ;
    rdfs:label "카테고리 소속"@ko .

sm:suppliedBy a owl:ObjectProperty ;
    rdfs:domain sm:Product ;
    rdfs:range sm:Supplier ;
    rdfs:label "공급자"@ko .

sm:locatedIn a owl:ObjectProperty ;
    rdfs:domain sm:Product ;
    rdfs:range sm:ShelfLocation ;
    rdfs:label "진열 위치"@ko .

sm:hasSubstitute a owl:ObjectProperty, owl:SymmetricProperty ;
    rdfs:domain sm:Product ;
    rdfs:range sm:Product ;
    rdfs:label "대체 상품"@ko .

sm:complementaryTo a owl:ObjectProperty, owl:SymmetricProperty ;
    rdfs:domain sm:Product ;
    rdfs:range sm:Product ;
    rdfs:label "보완 상품"@ko .

sm:isIngredientOf a owl:ObjectProperty ;
    rdfs:domain sm:Product ;
    rdfs:range sm:Recipe ;
    rdfs:label "레시피 재료"@ko .

# === 고객 관계 ===
sm:hasPurchased a owl:ObjectProperty ;
    rdfs:domain sm:Customer ;
    rdfs:range sm:Product ;
    rdfs:label "구매한 상품"@ko .

sm:hasPreference a owl:ObjectProperty ;
    rdfs:domain sm:Customer ;
    rdfs:range sm:Category ;
    rdfs:label "선호 카테고리"@ko .

sm:hasAllergy a owl:ObjectProperty ;
    rdfs:domain sm:Customer ;
    rdfs:range sm:Allergen ;
    rdfs:label "알레르기"@ko .

sm:madeTransaction a owl:ObjectProperty ;
    rdfs:domain sm:Customer ;
    rdfs:range sm:Transaction ;
    owl:inverseOf sm:transactionBy ;
    rdfs:label "거래 내역"@ko .

# === 거래 관계 ===
sm:transactionBy a owl:ObjectProperty ;
    rdfs:domain sm:Transaction ;
    rdfs:range sm:Customer ;
    rdfs:label "거래 고객"@ko .

sm:containsItem a owl:ObjectProperty ;
    rdfs:domain sm:Transaction ;
    rdfs:range sm:TransactionItem ;
    rdfs:label "거래 품목"@ko .

sm:processedBy a owl:ObjectProperty ;
    rdfs:domain sm:Transaction ;
    rdfs:range sm:Employee ;
    rdfs:label "처리 직원"@ko .

sm:atStore a owl:ObjectProperty ;
    rdfs:domain sm:Transaction ;
    rdfs:range sm:Store ;
    rdfs:label "거래 매장"@ko .
```

## 관계 다이어그램

```
                    ┌─────────────┐
                    │  Supplier   │
                    └──────┬──────┘
                           │ suppliedBy
                           ▼
┌──────────┐       ┌─────────────┐       ┌──────────┐
│ Category │◄──────│   Product   │──────►│ Location │
└──────────┘       └──────┬──────┘       └──────────┘
  belongsTo               │
                          │ containsItem
                          ▼
                   ┌─────────────┐
                   │ Transaction │
                   └──────┬──────┘
                          │ transactionBy
                          ▼
                   ┌─────────────┐       ┌──────────┐
                   │  Customer   │──────►│Preference│
                   └─────────────┘       └──────────┘
                     hasPreference
```

## 추론 규칙 (SWRL Rules)

### 1. 프리미엄 고객 자동 분류

```
Customer(?c) ∧ totalPurchaseAmount(?c, ?amount) ∧
swrlb:greaterThan(?amount, 1000000)
→ PremiumCustomer(?c)
```

### 2. 유통기한 임박 상품 식별

```
Product(?p) ∧ expiryDate(?p, ?date) ∧
swrlb:lessThan(?date, currentDate + 3days)
→ ExpiringProduct(?p)
```

### 3. 재고 부족 알림

```
Product(?p) ∧ stockQuantity(?p, ?qty) ∧
minimumStock(?p, ?min) ∧ swrlb:lessThan(?qty, ?min)
→ LowStockProduct(?p)
```

### 4. 보완 상품 추천

```python
# 규칙: 파스타를 사면 토마토 소스 추천
"""
Product(?p1) ∧ hasCategory(?p1, Pasta) ∧
Product(?p2) ∧ hasCategory(?p2, PastaSauce) ∧
complementaryTo(?p1, ?p2)
→ recommend(?p1, ?p2)
"""
```

## 제약 조건 (Constraints)

### OWL 제약 조건

```turtle
# 상품은 반드시 하나의 바코드를 가짐
sm:Product a owl:Class ;
    rdfs:subClassOf [
        a owl:Restriction ;
        owl:onProperty sm:barcode ;
        owl:cardinality 1
    ] .

# 거래는 반드시 한 명의 고객에 의해 수행됨
sm:Transaction a owl:Class ;
    rdfs:subClassOf [
        a owl:Restriction ;
        owl:onProperty sm:transactionBy ;
        owl:cardinality 1
    ] .

# 가격은 0보다 커야 함
sm:price a owl:DatatypeProperty ;
    rdfs:range [
        a rdfs:Datatype ;
        owl:onDatatype xsd:decimal ;
        owl:withRestrictions ([xsd:minExclusive 0])
    ] .
```

### SHACL 제약 조건

```turtle
@prefix sh: <http://www.w3.org/ns/shacl#> .

sm:ProductShape a sh:NodeShape ;
    sh:targetClass sm:Product ;
    sh:property [
        sh:path sm:productName ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:datatype xsd:string ;
        sh:minLength 1 ;
        sh:message "상품명은 필수입니다"@ko
    ] ;
    sh:property [
        sh:path sm:price ;
        sh:minCount 1 ;
        sh:datatype xsd:decimal ;
        sh:minExclusive 0 ;
        sh:message "가격은 0보다 커야 합니다"@ko
    ] ;
    sh:property [
        sh:path sm:barcode ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:pattern "^[0-9]{13}$" ;
        sh:message "바코드는 13자리 숫자여야 합니다"@ko
    ] .
```

---

다음: [인프라 구축](infrastructure.md)에서 온톨로지 시스템을 위한 기술 스택을 알아봅니다.
