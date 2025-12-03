# 활용 사례

## 개요

슈퍼마켓 온톨로지 시스템의 **실제 비즈니스 활용 사례**와 **SPARQL 쿼리 예시**를 다룹니다.

## 사례 1: 지능형 상품 추천

### 시나리오

> 고객이 "파스타 면"을 장바구니에 담았을 때, 함께 구매하면 좋은 상품을 자동 추천

### 온톨로지 기반 추론

```
파스타면 ──complementaryTo──► 토마토소스
파스타면 ──complementaryTo──► 올리브오일
파스타면 ──complementaryTo──► 파마산치즈
파스타면 ──isIngredientOf──► 까르보나라
까르보나라 ──hasIngredient──► 베이컨
까르보나라 ──hasIngredient──► 생크림
```

### SPARQL 쿼리

```sparql
PREFIX sm: <http://example.org/supermarket#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

# 직접 보완 상품 추천
SELECT ?recommended ?name ?price WHERE {
    sm:pasta_spaghetti sm:complementaryTo ?recommended .
    ?recommended sm:productName ?name ;
                 sm:price ?price ;
                 sm:stockQuantity ?stock .
    FILTER (?stock > 0)
}
ORDER BY ?price
LIMIT 5
```

```sparql
# 레시피 기반 확장 추천
SELECT DISTINCT ?ingredient ?name WHERE {
    sm:pasta_spaghetti sm:isIngredientOf ?recipe .
    ?recipe sm:hasIngredient ?ingredient .
    ?ingredient sm:productName ?name .
    FILTER (?ingredient != sm:pasta_spaghetti)
}
```

### Python 구현

```python
class RecommendationService:
    """추천 서비스"""

    def __init__(self, ontology_service: OntologyService):
        self.ontology = ontology_service

    def get_complementary_recommendations(
        self,
        product_id: str,
        limit: int = 5
    ) -> List[Product]:
        """보완 상품 추천"""
        query = f"""
        PREFIX sm: <http://example.org/supermarket#>

        SELECT ?product ?name ?price ?stock WHERE {{
            sm:{product_id} sm:complementaryTo ?product .
            ?product sm:productName ?name ;
                     sm:price ?price ;
                     sm:stockQuantity ?stock .
            FILTER (?stock > 0)
        }}
        ORDER BY DESC(?stock)
        LIMIT {limit}
        """
        return self.ontology.execute_sparql(query)

    def get_recipe_based_recommendations(
        self,
        product_id: str
    ) -> List[Product]:
        """레시피 기반 추천"""
        query = f"""
        PREFIX sm: <http://example.org/supermarket#>

        SELECT DISTINCT ?ingredient ?name ?price WHERE {{
            sm:{product_id} sm:isIngredientOf ?recipe .
            ?recipe sm:hasIngredient ?ingredient .
            ?ingredient sm:productName ?name ;
                        sm:price ?price ;
                        sm:stockQuantity ?stock .
            FILTER (?ingredient != sm:{product_id})
            FILTER (?stock > 0)
        }}
        """
        return self.ontology.execute_sparql(query)

    def get_personalized_recommendations(
        self,
        customer_id: str,
        limit: int = 10
    ) -> List[Dict]:
        """개인화 추천 (구매 이력 + 선호도 기반)"""
        query = f"""
        PREFIX sm: <http://example.org/supermarket#>

        SELECT DISTINCT ?product ?name ?score WHERE {{
            # 고객 선호 카테고리의 상품
            {{
                sm:{customer_id} sm:hasPreference ?category .
                ?product sm:belongsToCategory ?category ;
                         sm:productName ?name .
                BIND(2 AS ?score)
            }}
            UNION
            # 과거 구매 상품의 보완 상품
            {{
                sm:{customer_id} sm:hasPurchased ?purchased .
                ?purchased sm:complementaryTo ?product .
                ?product sm:productName ?name .
                FILTER NOT EXISTS {{
                    sm:{customer_id} sm:hasPurchased ?product
                }}
                BIND(1 AS ?score)
            }}
        }}
        ORDER BY DESC(?score)
        LIMIT {limit}
        """
        return self.ontology.execute_sparql(query)
```

---

## 사례 2: 재고 관리 자동화

### 시나리오

> 유통기한 임박 상품 자동 식별, 재고 부족 알림, 자동 발주 트리거

### SPARQL 쿼리

```sparql
PREFIX sm: <http://example.org/supermarket#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

# 유통기한 3일 이내 상품 조회
SELECT ?product ?name ?expiryDate ?stock ?daysTillExpiry WHERE {
    ?product a sm:Product ;
             sm:productName ?name ;
             sm:expiryDate ?expiryDate ;
             sm:stockQuantity ?stock .

    BIND(xsd:integer((?expiryDate - NOW()) / 86400) AS ?daysTillExpiry)
    FILTER (?daysTillExpiry <= 3 && ?daysTillExpiry >= 0)
}
ORDER BY ?daysTillExpiry
```

```sparql
# 재고 부족 상품 (최소 재고 미만)
SELECT ?product ?name ?currentStock ?minStock ?shortage WHERE {
    ?product a sm:Product ;
             sm:productName ?name ;
             sm:stockQuantity ?currentStock ;
             sm:minimumStock ?minStock .

    FILTER (?currentStock < ?minStock)
    BIND(?minStock - ?currentStock AS ?shortage)
}
ORDER BY DESC(?shortage)
```

```sparql
# 자동 발주 대상 (재고 부족 + 공급자 정보)
SELECT ?product ?name ?supplier ?supplierEmail ?orderQuantity WHERE {
    ?product a sm:Product ;
             sm:productName ?name ;
             sm:stockQuantity ?currentStock ;
             sm:minimumStock ?minStock ;
             sm:reorderQuantity ?orderQuantity ;
             sm:suppliedBy ?supplier .

    ?supplier sm:email ?supplierEmail .

    FILTER (?currentStock < ?minStock)
}
```

### Python 구현

```python
from datetime import datetime, timedelta
from dataclasses import dataclass
from typing import List
import asyncio

@dataclass
class StockAlert:
    """재고 알림"""
    product_id: str
    product_name: str
    alert_type: str  # 'expiring', 'low_stock', 'out_of_stock'
    severity: str    # 'warning', 'critical'
    details: dict

class InventoryService:
    """재고 관리 서비스"""

    def __init__(self, ontology_service: OntologyService):
        self.ontology = ontology_service

    def get_expiring_products(self, days: int = 3) -> List[StockAlert]:
        """유통기한 임박 상품 조회"""
        query = f"""
        PREFIX sm: <http://example.org/supermarket#>
        PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

        SELECT ?product ?name ?expiryDate ?stock WHERE {{
            ?product a sm:Product ;
                     sm:productName ?name ;
                     sm:expiryDate ?expiryDate ;
                     sm:stockQuantity ?stock .

            FILTER (?expiryDate <= "{(datetime.now() + timedelta(days=days)).date()}"^^xsd:date)
            FILTER (?stock > 0)
        }}
        """
        results = self.ontology.execute_sparql(query)

        alerts = []
        for row in results:
            expiry = datetime.fromisoformat(row['expiryDate']).date()
            days_left = (expiry - datetime.now().date()).days

            alerts.append(StockAlert(
                product_id=row['product'].split('#')[-1],
                product_name=row['name'],
                alert_type='expiring',
                severity='critical' if days_left <= 1 else 'warning',
                details={
                    'expiry_date': str(expiry),
                    'days_left': days_left,
                    'current_stock': int(row['stock'])
                }
            ))
        return alerts

    def get_low_stock_products(self) -> List[StockAlert]:
        """재고 부족 상품 조회"""
        query = """
        PREFIX sm: <http://example.org/supermarket#>

        SELECT ?product ?name ?currentStock ?minStock WHERE {
            ?product a sm:Product ;
                     sm:productName ?name ;
                     sm:stockQuantity ?currentStock ;
                     sm:minimumStock ?minStock .
            FILTER (?currentStock < ?minStock)
        }
        """
        results = self.ontology.execute_sparql(query)

        alerts = []
        for row in results:
            current = int(row['currentStock'])
            minimum = int(row['minStock'])

            alerts.append(StockAlert(
                product_id=row['product'].split('#')[-1],
                product_name=row['name'],
                alert_type='out_of_stock' if current == 0 else 'low_stock',
                severity='critical' if current == 0 else 'warning',
                details={
                    'current_stock': current,
                    'minimum_stock': minimum,
                    'shortage': minimum - current
                }
            ))
        return alerts

    async def auto_reorder(self, product_id: str) -> dict:
        """자동 발주 실행"""
        query = f"""
        PREFIX sm: <http://example.org/supermarket#>

        SELECT ?supplier ?email ?reorderQty ?unitPrice WHERE {{
            sm:{product_id} sm:suppliedBy ?supplier ;
                           sm:reorderQuantity ?reorderQty ;
                           sm:unitPrice ?unitPrice .
            ?supplier sm:email ?email .
        }}
        """
        result = self.ontology.execute_sparql(query)

        if result:
            supplier_info = result[0]
            # 발주 로직 (이메일 발송, ERP 연동 등)
            return {
                "status": "ordered",
                "supplier": supplier_info['supplier'],
                "quantity": supplier_info['reorderQty'],
                "estimated_cost": float(supplier_info['reorderQty']) * float(supplier_info['unitPrice'])
            }
        return {"status": "failed", "reason": "No supplier found"}
```

---

## 사례 3: 동적 할인 정책

### 시나리오

> 유통기한, 재고 상황, 고객 등급에 따른 동적 할인 적용

### 온톨로지 규칙

```turtle
# 할인 규칙 온톨로지
sm:DiscountRule a owl:Class .

sm:ExpiryDiscountRule a owl:Class ;
    rdfs:subClassOf sm:DiscountRule .

sm:CustomerDiscountRule a owl:Class ;
    rdfs:subClassOf sm:DiscountRule .

sm:hasDiscountRate a owl:DatatypeProperty ;
    rdfs:domain sm:DiscountRule ;
    rdfs:range xsd:decimal .

# 유통기한 3일 이내: 30% 할인
sm:expiry_3day_rule a sm:ExpiryDiscountRule ;
    sm:daysBeforeExpiry 3 ;
    sm:hasDiscountRate 0.30 .

# 유통기한 7일 이내: 20% 할인
sm:expiry_7day_rule a sm:ExpiryDiscountRule ;
    sm:daysBeforeExpiry 7 ;
    sm:hasDiscountRate 0.20 .

# 프리미엄 고객: 추가 10% 할인
sm:premium_customer_rule a sm:CustomerDiscountRule ;
    sm:appliesTo sm:PremiumCustomer ;
    sm:hasDiscountRate 0.10 .
```

### SPARQL 쿼리

```sparql
PREFIX sm: <http://example.org/supermarket#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

# 상품별 적용 가능한 할인 계산
SELECT ?product ?name ?originalPrice ?discountRate ?finalPrice WHERE {
    ?product a sm:Product ;
             sm:productName ?name ;
             sm:price ?originalPrice ;
             sm:expiryDate ?expiryDate .

    # 유통기한 기반 할인 규칙 적용
    ?rule a sm:ExpiryDiscountRule ;
          sm:daysBeforeExpiry ?days ;
          sm:hasDiscountRate ?discountRate .

    BIND(xsd:integer((?expiryDate - NOW()) / 86400) AS ?daysTillExpiry)
    FILTER (?daysTillExpiry <= ?days && ?daysTillExpiry >= 0)

    BIND(?originalPrice * (1 - ?discountRate) AS ?finalPrice)
}
ORDER BY ?daysTillExpiry
```

```sparql
# 고객별 최종 가격 계산 (고객 할인 + 상품 할인 중첩)
SELECT ?product ?name ?originalPrice ?productDiscount ?customerDiscount ?finalPrice WHERE {
    ?product a sm:Product ;
             sm:productName ?name ;
             sm:price ?originalPrice .

    # 고객 등급 확인
    sm:customer_123 a ?customerType .

    # 상품 할인 (옵션)
    OPTIONAL {
        ?product sm:expiryDate ?expiryDate .
        ?productRule a sm:ExpiryDiscountRule ;
                     sm:daysBeforeExpiry ?days ;
                     sm:hasDiscountRate ?productDiscount .
        FILTER (xsd:integer((?expiryDate - NOW()) / 86400) <= ?days)
    }

    # 고객 할인 (옵션)
    OPTIONAL {
        ?customerRule a sm:CustomerDiscountRule ;
                      sm:appliesTo ?customerType ;
                      sm:hasDiscountRate ?customerDiscount .
    }

    BIND(COALESCE(?productDiscount, 0) AS ?pDisc)
    BIND(COALESCE(?customerDiscount, 0) AS ?cDisc)
    BIND(?originalPrice * (1 - ?pDisc) * (1 - ?cDisc) AS ?finalPrice)
}
```

### Python 구현

```python
from decimal import Decimal
from typing import Optional

class PricingService:
    """가격 정책 서비스"""

    def __init__(self, ontology_service: OntologyService):
        self.ontology = ontology_service

    def calculate_price(
        self,
        product_id: str,
        customer_id: Optional[str] = None
    ) -> dict:
        """최종 가격 계산"""

        # 기본 상품 정보 조회
        product_query = f"""
        PREFIX sm: <http://example.org/supermarket#>

        SELECT ?price ?expiryDate WHERE {{
            sm:{product_id} sm:price ?price .
            OPTIONAL {{ sm:{product_id} sm:expiryDate ?expiryDate }}
        }}
        """
        product_result = self.ontology.execute_sparql(product_query)

        if not product_result:
            raise ValueError(f"Product {product_id} not found")

        original_price = Decimal(product_result[0]['price'])
        discounts = []

        # 유통기한 할인 확인
        if 'expiryDate' in product_result[0]:
            expiry_discount = self._get_expiry_discount(product_id)
            if expiry_discount:
                discounts.append({
                    'type': 'expiry',
                    'rate': expiry_discount,
                    'description': '유통기한 임박 할인'
                })

        # 고객 할인 확인
        if customer_id:
            customer_discount = self._get_customer_discount(customer_id)
            if customer_discount:
                discounts.append({
                    'type': 'customer',
                    'rate': customer_discount,
                    'description': '회원 등급 할인'
                })

        # 최종 가격 계산
        final_price = original_price
        for discount in discounts:
            final_price = final_price * (1 - Decimal(str(discount['rate'])))

        return {
            'original_price': float(original_price),
            'final_price': float(final_price.quantize(Decimal('0.01'))),
            'total_discount': float(original_price - final_price),
            'discount_rate': float((original_price - final_price) / original_price),
            'applied_discounts': discounts
        }

    def _get_expiry_discount(self, product_id: str) -> Optional[float]:
        """유통기한 할인율 조회"""
        query = f"""
        PREFIX sm: <http://example.org/supermarket#>
        PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

        SELECT ?discountRate WHERE {{
            sm:{product_id} sm:expiryDate ?expiryDate .
            ?rule a sm:ExpiryDiscountRule ;
                  sm:daysBeforeExpiry ?days ;
                  sm:hasDiscountRate ?discountRate .

            BIND(xsd:integer((?expiryDate - NOW()) / 86400) AS ?daysTillExpiry)
            FILTER (?daysTillExpiry <= ?days && ?daysTillExpiry >= 0)
        }}
        ORDER BY DESC(?discountRate)
        LIMIT 1
        """
        result = self.ontology.execute_sparql(query)
        return float(result[0]['discountRate']) if result else None

    def _get_customer_discount(self, customer_id: str) -> Optional[float]:
        """고객 등급 할인율 조회"""
        query = f"""
        PREFIX sm: <http://example.org/supermarket#>

        SELECT ?discountRate WHERE {{
            sm:{customer_id} a ?customerType .
            ?rule a sm:CustomerDiscountRule ;
                  sm:appliesTo ?customerType ;
                  sm:hasDiscountRate ?discountRate .
        }}
        LIMIT 1
        """
        result = self.ontology.execute_sparql(query)
        return float(result[0]['discountRate']) if result else None
```

---

## 사례 4: 고객 세그먼트 분석

### 시나리오

> 구매 패턴 기반 고객 자동 분류 및 마케팅 타겟팅

### SPARQL 쿼리

```sparql
PREFIX sm: <http://example.org/supermarket#>

# 카테고리별 구매 비율로 고객 세그먼트 식별
SELECT ?customer ?customerName ?topCategory (COUNT(?purchase) AS ?purchaseCount) WHERE {
    ?customer a sm:Customer ;
              sm:customerName ?customerName ;
              sm:hasPurchased ?product .

    ?product sm:belongsToCategory ?category .

    {
        SELECT ?customer (MAX(?cnt) AS ?maxCnt) WHERE {
            SELECT ?customer ?cat (COUNT(?p) AS ?cnt) WHERE {
                ?customer sm:hasPurchased ?p .
                ?p sm:belongsToCategory ?cat .
            }
            GROUP BY ?customer ?cat
        }
        GROUP BY ?customer
    }

    {
        SELECT ?customer ?topCategory (COUNT(?prod) AS ?catCount) WHERE {
            ?customer sm:hasPurchased ?prod .
            ?prod sm:belongsToCategory ?topCategory .
        }
        GROUP BY ?customer ?topCategory
    }
    FILTER (?catCount = ?maxCnt)
}
GROUP BY ?customer ?customerName ?topCategory
ORDER BY DESC(?purchaseCount)
```

```sparql
# 프리미엄 고객 후보 식별 (구매액 기준)
SELECT ?customer ?name ?totalAmount WHERE {
    ?customer a sm:Customer ;
              sm:customerName ?name ;
              sm:totalPurchaseAmount ?totalAmount .

    FILTER (?totalAmount >= 800000 && ?totalAmount < 1000000)
}
ORDER BY DESC(?totalAmount)
```

```sparql
# 이탈 위험 고객 식별 (최근 구매 없음)
SELECT ?customer ?name ?lastPurchase ?daysSinceLastPurchase WHERE {
    ?customer a sm:Customer ;
              sm:customerName ?name ;
              sm:lastPurchaseDate ?lastPurchase .

    BIND(xsd:integer((NOW() - ?lastPurchase) / 86400) AS ?daysSinceLastPurchase)
    FILTER (?daysSinceLastPurchase > 30)
}
ORDER BY DESC(?daysSinceLastPurchase)
```

### Python 구현

```python
@dataclass
class CustomerSegment:
    """고객 세그먼트"""
    segment_id: str
    name: str
    description: str
    criteria: dict
    customers: List[str]

class CustomerAnalyticsService:
    """고객 분석 서비스"""

    def __init__(self, ontology_service: OntologyService):
        self.ontology = ontology_service

    def segment_customers(self) -> List[CustomerSegment]:
        """고객 세그먼트 분류"""
        segments = []

        # VIP 고객
        vip_query = """
        PREFIX sm: <http://example.org/supermarket#>

        SELECT ?customer WHERE {
            ?customer a sm:PremiumCustomer .
        }
        """
        vip_customers = [r['customer'].split('#')[-1]
                        for r in self.ontology.execute_sparql(vip_query)]

        segments.append(CustomerSegment(
            segment_id="vip",
            name="VIP 고객",
            description="연간 구매액 100만원 이상",
            criteria={"min_purchase": 1000000},
            customers=vip_customers
        ))

        # 신선식품 선호 고객
        fresh_query = """
        PREFIX sm: <http://example.org/supermarket#>

        SELECT ?customer WHERE {
            ?customer sm:hasPreference sm:FreshProduct .
        }
        """
        fresh_customers = [r['customer'].split('#')[-1]
                         for r in self.ontology.execute_sparql(fresh_query)]

        segments.append(CustomerSegment(
            segment_id="fresh_lover",
            name="신선식품 선호 고객",
            description="신선식품 카테고리 선호",
            criteria={"preference": "FreshProduct"},
            customers=fresh_customers
        ))

        # 이탈 위험 고객
        churn_query = """
        PREFIX sm: <http://example.org/supermarket#>
        PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

        SELECT ?customer WHERE {
            ?customer a sm:Customer ;
                      sm:lastPurchaseDate ?lastPurchase .
            FILTER ((NOW() - ?lastPurchase) / 86400 > 30)
        }
        """
        churn_customers = [r['customer'].split('#')[-1]
                         for r in self.ontology.execute_sparql(churn_query)]

        segments.append(CustomerSegment(
            segment_id="churn_risk",
            name="이탈 위험 고객",
            description="30일 이상 구매 없음",
            criteria={"days_since_purchase": 30},
            customers=churn_customers
        ))

        return segments

    def get_marketing_targets(
        self,
        campaign_type: str
    ) -> List[dict]:
        """마케팅 타겟 추출"""

        if campaign_type == "upsell":
            # 프리미엄 등급 직전 고객
            query = """
            PREFIX sm: <http://example.org/supermarket#>

            SELECT ?customer ?name ?totalAmount ?needed WHERE {
                ?customer a sm:Customer ;
                          sm:customerName ?name ;
                          sm:totalPurchaseAmount ?totalAmount .
                FILTER (?totalAmount >= 800000 && ?totalAmount < 1000000)
                BIND(1000000 - ?totalAmount AS ?needed)
            }
            ORDER BY ?needed
            """
        elif campaign_type == "retention":
            # 이탈 위험 고객
            query = """
            PREFIX sm: <http://example.org/supermarket#>

            SELECT ?customer ?name ?lastPurchase WHERE {
                ?customer a sm:Customer ;
                          sm:customerName ?name ;
                          sm:lastPurchaseDate ?lastPurchase .
                FILTER ((NOW() - ?lastPurchase) / 86400 > 30)
            }
            """
        else:
            return []

        return self.ontology.execute_sparql(query)
```

---

## 사례 5: 지식 그래프 시각화

### 시나리오

> 상품 간 관계, 고객 구매 패턴을 시각적으로 탐색

### SPARQL CONSTRUCT 쿼리

```sparql
PREFIX sm: <http://example.org/supermarket#>

# 상품 관계 그래프 생성
CONSTRUCT {
    ?product1 sm:relatedTo ?product2 .
    ?product1 sm:productName ?name1 .
    ?product2 sm:productName ?name2 .
}
WHERE {
    {
        ?product1 sm:complementaryTo ?product2 .
    } UNION {
        ?product1 sm:hasSubstitute ?product2 .
    }
    ?product1 sm:productName ?name1 .
    ?product2 sm:productName ?name2 .
}
```

### D3.js 시각화

```javascript
// 지식 그래프 시각화 (D3.js)
async function visualizeKnowledgeGraph() {
    // SPARQL 쿼리로 데이터 가져오기
    const response = await fetch('/api/v1/sparql', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            query: `
                PREFIX sm: <http://example.org/supermarket#>
                SELECT ?source ?target ?relation WHERE {
                    {
                        ?source sm:complementaryTo ?target .
                        BIND("complementary" AS ?relation)
                    } UNION {
                        ?source sm:hasSubstitute ?target .
                        BIND("substitute" AS ?relation)
                    }
                }
                LIMIT 100
            `
        })
    });

    const data = await response.json();

    // 노드와 링크 구성
    const nodes = new Map();
    const links = [];

    data.results.forEach(row => {
        const sourceId = row.source.split('#')[1];
        const targetId = row.target.split('#')[1];

        if (!nodes.has(sourceId)) {
            nodes.set(sourceId, { id: sourceId });
        }
        if (!nodes.has(targetId)) {
            nodes.set(targetId, { id: targetId });
        }

        links.push({
            source: sourceId,
            target: targetId,
            type: row.relation
        });
    });

    // D3 Force 시뮬레이션
    const svg = d3.select("#graph");
    const width = 800;
    const height = 600;

    const simulation = d3.forceSimulation(Array.from(nodes.values()))
        .force("link", d3.forceLink(links).id(d => d.id))
        .force("charge", d3.forceManyBody().strength(-100))
        .force("center", d3.forceCenter(width / 2, height / 2));

    // 링크 그리기
    const link = svg.selectAll(".link")
        .data(links)
        .enter().append("line")
        .attr("class", d => `link ${d.type}`)
        .attr("stroke", d => d.type === "complementary" ? "#4CAF50" : "#FF9800");

    // 노드 그리기
    const node = svg.selectAll(".node")
        .data(Array.from(nodes.values()))
        .enter().append("circle")
        .attr("class", "node")
        .attr("r", 10)
        .attr("fill", "#2196F3")
        .call(d3.drag()
            .on("start", dragstarted)
            .on("drag", dragged)
            .on("end", dragended));

    // 라벨
    const labels = svg.selectAll(".label")
        .data(Array.from(nodes.values()))
        .enter().append("text")
        .text(d => d.id)
        .attr("font-size", 10);

    simulation.on("tick", () => {
        link
            .attr("x1", d => d.source.x)
            .attr("y1", d => d.source.y)
            .attr("x2", d => d.target.x)
            .attr("y2", d => d.target.y);

        node
            .attr("cx", d => d.x)
            .attr("cy", d => d.y);

        labels
            .attr("x", d => d.x + 12)
            .attr("y", d => d.y + 4);
    });
}
```

---

## 성능 최적화 팁

### 1. 인덱스 활용

```sparql
# GraphDB 인덱스 힌트 사용
PREFIX hint: <http://www.ontotext.com/owlim/hint#>

SELECT ?product ?name WHERE {
    hint:Query hint:optimizer "Alternative" .
    ?product a sm:Product ;
             sm:productName ?name .
}
```

### 2. 쿼리 최적화

```sparql
# BAD: 전체 스캔
SELECT * WHERE {
    ?s ?p ?o .
    FILTER (str(?o) = "사과")
}

# GOOD: 인덱스 활용
SELECT * WHERE {
    ?s sm:productName "사과"@ko .
}
```

### 3. 페이지네이션

```sparql
# OFFSET/LIMIT으로 페이지네이션
SELECT ?product ?name WHERE {
    ?product a sm:Product ;
             sm:productName ?name .
}
ORDER BY ?name
LIMIT 20
OFFSET 40
```

---

## 요약

| 활용 사례 | 핵심 기술 | 비즈니스 가치 |
|----------|----------|--------------|
| 상품 추천 | 관계 추론, 그래프 탐색 | 매출 증가, 고객 만족 |
| 재고 관리 | 규칙 기반 추론, 알림 | 폐기 감소, 품절 방지 |
| 동적 가격 | 온톨로지 규칙, 추론 | 마진 최적화 |
| 고객 분석 | 패턴 매칭, 집계 쿼리 | 타겟 마케팅 |
| 시각화 | CONSTRUCT, 그래프 | 인사이트 도출 |

---

이것으로 슈퍼마켓 온톨로지 시스템 가이드를 마칩니다.

[처음으로 돌아가기](index.md)
