# 구현 가이드

## 개요

Python과 RDFLib를 사용하여 슈퍼마켓 온톨로지 시스템을 **단계별로 구현**합니다.

## 프로젝트 구조

```
supermarket-ontology/
├── ontology/
│   ├── supermarket.owl          # 메인 온톨로지 파일
│   ├── shapes.ttl               # SHACL 검증 규칙
│   └── instances/
│       ├── products.ttl         # 상품 데이터
│       └── customers.ttl        # 고객 데이터
├── src/
│   ├── __init__.py
│   ├── config.py                # 설정
│   ├── models/
│   │   ├── __init__.py
│   │   ├── product.py           # 상품 모델
│   │   ├── customer.py          # 고객 모델
│   │   └── transaction.py       # 거래 모델
│   ├── services/
│   │   ├── __init__.py
│   │   ├── ontology_service.py  # 온톨로지 서비스
│   │   ├── reasoning_service.py # 추론 서비스
│   │   └── query_service.py     # 쿼리 서비스
│   ├── api/
│   │   ├── __init__.py
│   │   ├── routes.py            # API 라우트
│   │   └── schemas.py           # Pydantic 스키마
│   └── utils/
│       ├── __init__.py
│       └── rdf_helpers.py       # RDF 헬퍼 함수
├── tests/
├── docker-compose.yml
├── requirements.txt
└── main.py
```

## Step 1: 온톨로지 파일 생성

### supermarket.owl

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
         xmlns:rdfs="http://www.w3.org/2000/01/rdf-schema#"
         xmlns:owl="http://www.w3.org/2002/07/owl#"
         xmlns:xsd="http://www.w3.org/2001/XMLSchema#"
         xmlns:sm="http://example.org/supermarket#">

  <!-- Ontology Declaration -->
  <owl:Ontology rdf:about="http://example.org/supermarket">
    <rdfs:label>Supermarket Ontology</rdfs:label>
    <rdfs:comment>슈퍼마켓 운영을 위한 온톨로지</rdfs:comment>
    <owl:versionInfo>1.0.0</owl:versionInfo>
  </owl:Ontology>

  <!-- ==================== Classes ==================== -->

  <!-- Product Classes -->
  <owl:Class rdf:about="http://example.org/supermarket#Product">
    <rdfs:label xml:lang="ko">상품</rdfs:label>
    <rdfs:label xml:lang="en">Product</rdfs:label>
  </owl:Class>

  <owl:Class rdf:about="http://example.org/supermarket#FreshProduct">
    <rdfs:subClassOf rdf:resource="http://example.org/supermarket#Product"/>
    <rdfs:label xml:lang="ko">신선식품</rdfs:label>
  </owl:Class>

  <owl:Class rdf:about="http://example.org/supermarket#Vegetable">
    <rdfs:subClassOf rdf:resource="http://example.org/supermarket#FreshProduct"/>
    <rdfs:label xml:lang="ko">채소</rdfs:label>
  </owl:Class>

  <owl:Class rdf:about="http://example.org/supermarket#Fruit">
    <rdfs:subClassOf rdf:resource="http://example.org/supermarket#FreshProduct"/>
    <rdfs:label xml:lang="ko">과일</rdfs:label>
  </owl:Class>

  <owl:Class rdf:about="http://example.org/supermarket#Meat">
    <rdfs:subClassOf rdf:resource="http://example.org/supermarket#FreshProduct"/>
    <rdfs:label xml:lang="ko">육류</rdfs:label>
  </owl:Class>

  <owl:Class rdf:about="http://example.org/supermarket#ProcessedFood">
    <rdfs:subClassOf rdf:resource="http://example.org/supermarket#Product"/>
    <rdfs:label xml:lang="ko">가공식품</rdfs:label>
  </owl:Class>

  <owl:Class rdf:about="http://example.org/supermarket#Beverage">
    <rdfs:subClassOf rdf:resource="http://example.org/supermarket#Product"/>
    <rdfs:label xml:lang="ko">음료</rdfs:label>
  </owl:Class>

  <!-- Person Classes -->
  <owl:Class rdf:about="http://example.org/supermarket#Person">
    <rdfs:label xml:lang="ko">사람</rdfs:label>
  </owl:Class>

  <owl:Class rdf:about="http://example.org/supermarket#Customer">
    <rdfs:subClassOf rdf:resource="http://example.org/supermarket#Person"/>
    <rdfs:label xml:lang="ko">고객</rdfs:label>
  </owl:Class>

  <owl:Class rdf:about="http://example.org/supermarket#PremiumCustomer">
    <rdfs:subClassOf rdf:resource="http://example.org/supermarket#Customer"/>
    <rdfs:label xml:lang="ko">프리미엄 고객</rdfs:label>
  </owl:Class>

  <!-- Transaction Classes -->
  <owl:Class rdf:about="http://example.org/supermarket#Transaction">
    <rdfs:label xml:lang="ko">거래</rdfs:label>
  </owl:Class>

  <!-- Category Class -->
  <owl:Class rdf:about="http://example.org/supermarket#Category">
    <rdfs:label xml:lang="ko">카테고리</rdfs:label>
  </owl:Class>

  <!-- ==================== Properties ==================== -->

  <!-- Datatype Properties -->
  <owl:DatatypeProperty rdf:about="http://example.org/supermarket#productName">
    <rdfs:domain rdf:resource="http://example.org/supermarket#Product"/>
    <rdfs:range rdf:resource="http://www.w3.org/2001/XMLSchema#string"/>
    <rdfs:label xml:lang="ko">상품명</rdfs:label>
  </owl:DatatypeProperty>

  <owl:DatatypeProperty rdf:about="http://example.org/supermarket#barcode">
    <rdfs:domain rdf:resource="http://example.org/supermarket#Product"/>
    <rdfs:range rdf:resource="http://www.w3.org/2001/XMLSchema#string"/>
    <rdfs:label xml:lang="ko">바코드</rdfs:label>
  </owl:DatatypeProperty>

  <owl:DatatypeProperty rdf:about="http://example.org/supermarket#price">
    <rdfs:domain rdf:resource="http://example.org/supermarket#Product"/>
    <rdfs:range rdf:resource="http://www.w3.org/2001/XMLSchema#decimal"/>
    <rdfs:label xml:lang="ko">가격</rdfs:label>
  </owl:DatatypeProperty>

  <owl:DatatypeProperty rdf:about="http://example.org/supermarket#stockQuantity">
    <rdfs:domain rdf:resource="http://example.org/supermarket#Product"/>
    <rdfs:range rdf:resource="http://www.w3.org/2001/XMLSchema#integer"/>
    <rdfs:label xml:lang="ko">재고수량</rdfs:label>
  </owl:DatatypeProperty>

  <owl:DatatypeProperty rdf:about="http://example.org/supermarket#expiryDate">
    <rdfs:domain rdf:resource="http://example.org/supermarket#Product"/>
    <rdfs:range rdf:resource="http://www.w3.org/2001/XMLSchema#date"/>
    <rdfs:label xml:lang="ko">유통기한</rdfs:label>
  </owl:DatatypeProperty>

  <owl:DatatypeProperty rdf:about="http://example.org/supermarket#customerName">
    <rdfs:domain rdf:resource="http://example.org/supermarket#Customer"/>
    <rdfs:range rdf:resource="http://www.w3.org/2001/XMLSchema#string"/>
    <rdfs:label xml:lang="ko">고객명</rdfs:label>
  </owl:DatatypeProperty>

  <owl:DatatypeProperty rdf:about="http://example.org/supermarket#totalPurchaseAmount">
    <rdfs:domain rdf:resource="http://example.org/supermarket#Customer"/>
    <rdfs:range rdf:resource="http://www.w3.org/2001/XMLSchema#decimal"/>
    <rdfs:label xml:lang="ko">총구매금액</rdfs:label>
  </owl:DatatypeProperty>

  <!-- Object Properties -->
  <owl:ObjectProperty rdf:about="http://example.org/supermarket#belongsToCategory">
    <rdfs:domain rdf:resource="http://example.org/supermarket#Product"/>
    <rdfs:range rdf:resource="http://example.org/supermarket#Category"/>
    <rdfs:label xml:lang="ko">카테고리 소속</rdfs:label>
  </owl:ObjectProperty>

  <owl:ObjectProperty rdf:about="http://example.org/supermarket#hasPurchased">
    <rdfs:domain rdf:resource="http://example.org/supermarket#Customer"/>
    <rdfs:range rdf:resource="http://example.org/supermarket#Product"/>
    <rdfs:label xml:lang="ko">구매함</rdfs:label>
  </owl:ObjectProperty>

  <owl:ObjectProperty rdf:about="http://example.org/supermarket#complementaryTo">
    <rdf:type rdf:resource="http://www.w3.org/2002/07/owl#SymmetricProperty"/>
    <rdfs:domain rdf:resource="http://example.org/supermarket#Product"/>
    <rdfs:range rdf:resource="http://example.org/supermarket#Product"/>
    <rdfs:label xml:lang="ko">보완상품</rdfs:label>
  </owl:ObjectProperty>

  <owl:ObjectProperty rdf:about="http://example.org/supermarket#hasSubstitute">
    <rdf:type rdf:resource="http://www.w3.org/2002/07/owl#SymmetricProperty"/>
    <rdfs:domain rdf:resource="http://example.org/supermarket#Product"/>
    <rdfs:range rdf:resource="http://example.org/supermarket#Product"/>
    <rdfs:label xml:lang="ko">대체상품</rdfs:label>
  </owl:ObjectProperty>

</rdf:RDF>
```

## Step 2: Python 모델 구현

### config.py

```python
"""설정 모듈"""
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    """애플리케이션 설정"""

    # GraphDB 설정
    GRAPHDB_URL: str = "http://localhost:7200"
    GRAPHDB_REPOSITORY: str = "supermarket"

    # 네임스페이스
    SM_NAMESPACE: str = "http://example.org/supermarket#"

    # Redis 설정
    REDIS_URL: str = "redis://localhost:6379"

    # 캐시 TTL (초)
    CACHE_TTL: int = 300

    class Config:
        env_file = ".env"

settings = Settings()
```

### models/product.py

```python
"""상품 모델"""
from dataclasses import dataclass
from datetime import date
from decimal import Decimal
from typing import Optional, List
from rdflib import Graph, Namespace, Literal, URIRef
from rdflib.namespace import RDF, RDFS, XSD

SM = Namespace("http://example.org/supermarket#")

@dataclass
class Product:
    """상품 엔터티"""
    id: str
    name: str
    barcode: str
    price: Decimal
    stock_quantity: int
    category: str
    expiry_date: Optional[date] = None
    complementary_products: List[str] = None
    substitute_products: List[str] = None

    def __post_init__(self):
        if self.complementary_products is None:
            self.complementary_products = []
        if self.substitute_products is None:
            self.substitute_products = []

    def to_rdf(self, graph: Graph) -> URIRef:
        """RDF 트리플로 변환"""
        product_uri = SM[self.id]

        # 타입 선언
        graph.add((product_uri, RDF.type, SM.Product))

        # 데이터 속성
        graph.add((product_uri, SM.productName, Literal(self.name, lang="ko")))
        graph.add((product_uri, SM.barcode, Literal(self.barcode)))
        graph.add((product_uri, SM.price, Literal(self.price, datatype=XSD.decimal)))
        graph.add((product_uri, SM.stockQuantity, Literal(self.stock_quantity, datatype=XSD.integer)))

        if self.expiry_date:
            graph.add((product_uri, SM.expiryDate, Literal(self.expiry_date, datatype=XSD.date)))

        # 카테고리 관계
        graph.add((product_uri, SM.belongsToCategory, SM[self.category]))

        # 보완 상품 관계
        for comp_id in self.complementary_products:
            graph.add((product_uri, SM.complementaryTo, SM[comp_id]))

        # 대체 상품 관계
        for sub_id in self.substitute_products:
            graph.add((product_uri, SM.hasSubstitute, SM[sub_id]))

        return product_uri

    @classmethod
    def from_rdf(cls, graph: Graph, uri: URIRef) -> 'Product':
        """RDF에서 Product 객체 생성"""
        id = str(uri).split('#')[-1]

        name = str(graph.value(uri, SM.productName))
        barcode = str(graph.value(uri, SM.barcode))
        price = Decimal(str(graph.value(uri, SM.price)))
        stock = int(graph.value(uri, SM.stockQuantity))

        category_uri = graph.value(uri, SM.belongsToCategory)
        category = str(category_uri).split('#')[-1] if category_uri else ""

        expiry = graph.value(uri, SM.expiryDate)
        expiry_date = date.fromisoformat(str(expiry)) if expiry else None

        # 보완 상품
        complementary = [
            str(p).split('#')[-1]
            for p in graph.objects(uri, SM.complementaryTo)
        ]

        # 대체 상품
        substitutes = [
            str(p).split('#')[-1]
            for p in graph.objects(uri, SM.hasSubstitute)
        ]

        return cls(
            id=id,
            name=name,
            barcode=barcode,
            price=price,
            stock_quantity=stock,
            category=category,
            expiry_date=expiry_date,
            complementary_products=complementary,
            substitute_products=substitutes
        )
```

### models/customer.py

```python
"""고객 모델"""
from dataclasses import dataclass
from decimal import Decimal
from typing import List, Optional
from datetime import date
from rdflib import Graph, Namespace, Literal, URIRef
from rdflib.namespace import RDF, XSD

SM = Namespace("http://example.org/supermarket#")

@dataclass
class Customer:
    """고객 엔터티"""
    id: str
    name: str
    email: str
    membership_level: str
    total_purchase_amount: Decimal
    registration_date: date
    purchased_products: List[str] = None
    preferences: List[str] = None

    def __post_init__(self):
        if self.purchased_products is None:
            self.purchased_products = []
        if self.preferences is None:
            self.preferences = []

    @property
    def is_premium(self) -> bool:
        """프리미엄 고객 여부"""
        return self.total_purchase_amount >= Decimal('1000000')

    def to_rdf(self, graph: Graph) -> URIRef:
        """RDF 트리플로 변환"""
        customer_uri = SM[self.id]

        # 타입 선언 (프리미엄 여부에 따라)
        if self.is_premium:
            graph.add((customer_uri, RDF.type, SM.PremiumCustomer))
        else:
            graph.add((customer_uri, RDF.type, SM.Customer))

        # 데이터 속성
        graph.add((customer_uri, SM.customerName, Literal(self.name, lang="ko")))
        graph.add((customer_uri, SM.email, Literal(self.email)))
        graph.add((customer_uri, SM.membershipLevel, Literal(self.membership_level)))
        graph.add((customer_uri, SM.totalPurchaseAmount,
                   Literal(self.total_purchase_amount, datatype=XSD.decimal)))
        graph.add((customer_uri, SM.registrationDate,
                   Literal(self.registration_date, datatype=XSD.date)))

        # 구매 상품 관계
        for product_id in self.purchased_products:
            graph.add((customer_uri, SM.hasPurchased, SM[product_id]))

        # 선호 카테고리 관계
        for pref in self.preferences:
            graph.add((customer_uri, SM.hasPreference, SM[pref]))

        return customer_uri
```

## Step 3: 온톨로지 서비스 구현

### services/ontology_service.py

```python
"""온톨로지 서비스"""
from typing import List, Optional, Dict, Any
from rdflib import Graph, Namespace, URIRef
from rdflib.namespace import RDF, RDFS, OWL
import owlrl
from SPARQLWrapper import SPARQLWrapper, JSON, POST
import logging

from src.config import settings
from src.models.product import Product
from src.models.customer import Customer

logger = logging.getLogger(__name__)

SM = Namespace(settings.SM_NAMESPACE)

class OntologyService:
    """온톨로지 관리 서비스"""

    def __init__(self):
        self.graph = Graph()
        self.graph.bind("sm", SM)
        self.graph.bind("owl", OWL)
        self.graph.bind("rdfs", RDFS)

        self.sparql_endpoint = f"{settings.GRAPHDB_URL}/repositories/{settings.GRAPHDB_REPOSITORY}"

    def load_ontology(self, file_path: str, format: str = "xml") -> None:
        """온톨로지 파일 로드"""
        self.graph.parse(file_path, format=format)
        logger.info(f"Loaded ontology from {file_path}: {len(self.graph)} triples")

    def apply_reasoning(self, reasoning_type: str = "RDFS") -> None:
        """추론 적용"""
        initial_size = len(self.graph)

        if reasoning_type == "RDFS":
            owlrl.DeductiveClosure(owlrl.RDFS_Semantics).expand(self.graph)
        elif reasoning_type == "OWL-RL":
            owlrl.DeductiveClosure(owlrl.OWLRL_Semantics).expand(self.graph)

        inferred = len(self.graph) - initial_size
        logger.info(f"Applied {reasoning_type} reasoning: {inferred} new triples inferred")

    def add_product(self, product: Product) -> URIRef:
        """상품 추가"""
        uri = product.to_rdf(self.graph)
        logger.info(f"Added product: {product.name}")
        return uri

    def add_customer(self, customer: Customer) -> URIRef:
        """고객 추가"""
        uri = customer.to_rdf(self.graph)
        logger.info(f"Added customer: {customer.name}")
        return uri

    def get_product(self, product_id: str) -> Optional[Product]:
        """상품 조회"""
        uri = SM[product_id]
        if (uri, RDF.type, SM.Product) in self.graph:
            return Product.from_rdf(self.graph, uri)
        return None

    def get_products_by_category(self, category: str) -> List[Product]:
        """카테고리별 상품 조회"""
        products = []
        category_uri = SM[category]

        for product_uri in self.graph.subjects(SM.belongsToCategory, category_uri):
            products.append(Product.from_rdf(self.graph, product_uri))

        return products

    def get_complementary_products(self, product_id: str) -> List[Product]:
        """보완 상품 조회"""
        uri = SM[product_id]
        complementary = []

        for comp_uri in self.graph.objects(uri, SM.complementaryTo):
            complementary.append(Product.from_rdf(self.graph, comp_uri))

        return complementary

    def get_class_hierarchy(self, root_class: str = "Product") -> Dict[str, Any]:
        """클래스 계층 구조 조회"""
        def get_subclasses(class_uri):
            result = {"name": str(class_uri).split('#')[-1], "children": []}
            for subclass in self.graph.subjects(RDFS.subClassOf, class_uri):
                result["children"].append(get_subclasses(subclass))
            return result

        return get_subclasses(SM[root_class])

    def execute_sparql(self, query: str) -> List[Dict]:
        """SPARQL 쿼리 실행 (로컬 그래프)"""
        results = self.graph.query(query)
        return [
            {str(var): str(binding[var]) for var in results.vars}
            for binding in results
        ]

    def execute_remote_sparql(self, query: str) -> Dict:
        """원격 SPARQL 엔드포인트 쿼리"""
        sparql = SPARQLWrapper(self.sparql_endpoint)
        sparql.setQuery(query)
        sparql.setReturnFormat(JSON)
        return sparql.query().convert()

    def save_to_file(self, file_path: str, format: str = "turtle") -> None:
        """온톨로지를 파일로 저장"""
        self.graph.serialize(destination=file_path, format=format)
        logger.info(f"Saved ontology to {file_path}")

    def upload_to_graphdb(self) -> None:
        """GraphDB에 온톨로지 업로드"""
        sparql = SPARQLWrapper(f"{self.sparql_endpoint}/statements")
        sparql.setMethod(POST)
        sparql.setRequestMethod("postdirectly")

        turtle_data = self.graph.serialize(format="turtle")
        sparql.setQuery(turtle_data)

        sparql.query()
        logger.info("Uploaded ontology to GraphDB")

    def get_statistics(self) -> Dict[str, int]:
        """온톨로지 통계"""
        return {
            "total_triples": len(self.graph),
            "classes": len(list(self.graph.subjects(RDF.type, OWL.Class))),
            "object_properties": len(list(self.graph.subjects(RDF.type, OWL.ObjectProperty))),
            "datatype_properties": len(list(self.graph.subjects(RDF.type, OWL.DatatypeProperty))),
            "products": len(list(self.graph.subjects(RDF.type, SM.Product))),
            "customers": len(list(self.graph.subjects(RDF.type, SM.Customer))),
        }
```

## Step 4: API 구현

### api/schemas.py

```python
"""Pydantic 스키마"""
from pydantic import BaseModel, Field
from decimal import Decimal
from datetime import date
from typing import List, Optional

class ProductCreate(BaseModel):
    """상품 생성 스키마"""
    id: str = Field(..., description="상품 ID")
    name: str = Field(..., description="상품명")
    barcode: str = Field(..., pattern=r"^\d{13}$", description="바코드 (13자리)")
    price: Decimal = Field(..., gt=0, description="가격")
    stock_quantity: int = Field(..., ge=0, description="재고수량")
    category: str = Field(..., description="카테고리")
    expiry_date: Optional[date] = Field(None, description="유통기한")
    complementary_products: List[str] = Field(default=[], description="보완 상품 ID 목록")

class ProductResponse(BaseModel):
    """상품 응답 스키마"""
    id: str
    name: str
    barcode: str
    price: Decimal
    stock_quantity: int
    category: str
    expiry_date: Optional[date]
    complementary_products: List[str]
    substitute_products: List[str]

class CustomerCreate(BaseModel):
    """고객 생성 스키마"""
    id: str
    name: str
    email: str
    membership_level: str = "basic"
    total_purchase_amount: Decimal = Decimal("0")
    registration_date: date

class SPARQLQuery(BaseModel):
    """SPARQL 쿼리 스키마"""
    query: str = Field(..., description="SPARQL 쿼리문")

class RecommendationRequest(BaseModel):
    """추천 요청 스키마"""
    customer_id: str
    product_id: Optional[str] = None
    limit: int = Field(default=5, le=20)
```

### api/routes.py

```python
"""API 라우트"""
from fastapi import APIRouter, HTTPException, Depends
from typing import List

from src.services.ontology_service import OntologyService
from src.models.product import Product
from src.models.customer import Customer
from src.api.schemas import (
    ProductCreate, ProductResponse,
    CustomerCreate, SPARQLQuery, RecommendationRequest
)

router = APIRouter()

# 의존성 주입
def get_ontology_service() -> OntologyService:
    service = OntologyService()
    service.load_ontology("ontology/supermarket.owl")
    return service

@router.post("/products", response_model=ProductResponse)
async def create_product(
    product: ProductCreate,
    service: OntologyService = Depends(get_ontology_service)
):
    """상품 생성"""
    p = Product(
        id=product.id,
        name=product.name,
        barcode=product.barcode,
        price=product.price,
        stock_quantity=product.stock_quantity,
        category=product.category,
        expiry_date=product.expiry_date,
        complementary_products=product.complementary_products
    )
    service.add_product(p)
    return p

@router.get("/products/{product_id}", response_model=ProductResponse)
async def get_product(
    product_id: str,
    service: OntologyService = Depends(get_ontology_service)
):
    """상품 조회"""
    product = service.get_product(product_id)
    if not product:
        raise HTTPException(status_code=404, detail="Product not found")
    return product

@router.get("/products/category/{category}", response_model=List[ProductResponse])
async def get_products_by_category(
    category: str,
    service: OntologyService = Depends(get_ontology_service)
):
    """카테고리별 상품 조회"""
    return service.get_products_by_category(category)

@router.get("/products/{product_id}/complementary", response_model=List[ProductResponse])
async def get_complementary_products(
    product_id: str,
    service: OntologyService = Depends(get_ontology_service)
):
    """보완 상품 조회"""
    return service.get_complementary_products(product_id)

@router.post("/sparql")
async def execute_sparql(
    query: SPARQLQuery,
    service: OntologyService = Depends(get_ontology_service)
):
    """SPARQL 쿼리 실행"""
    try:
        results = service.execute_sparql(query.query)
        return {"results": results}
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))

@router.post("/recommendations")
async def get_recommendations(
    request: RecommendationRequest,
    service: OntologyService = Depends(get_ontology_service)
):
    """상품 추천"""
    # SPARQL로 추천 상품 조회
    query = f"""
    PREFIX sm: <http://example.org/supermarket#>

    SELECT DISTINCT ?product ?name WHERE {{
        sm:{request.customer_id} sm:hasPurchased ?purchased .
        ?purchased sm:complementaryTo ?product .
        ?product sm:productName ?name .
    }}
    LIMIT {request.limit}
    """
    results = service.execute_sparql(query)
    return {"recommendations": results}

@router.get("/statistics")
async def get_statistics(
    service: OntologyService = Depends(get_ontology_service)
):
    """온톨로지 통계"""
    return service.get_statistics()

@router.get("/hierarchy/{class_name}")
async def get_class_hierarchy(
    class_name: str,
    service: OntologyService = Depends(get_ontology_service)
):
    """클래스 계층 구조"""
    return service.get_class_hierarchy(class_name)
```

## Step 5: 메인 애플리케이션

### main.py

```python
"""메인 애플리케이션"""
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
import logging

from src.api.routes import router
from src.services.ontology_service import OntologyService

# 로깅 설정
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# FastAPI 앱 생성
app = FastAPI(
    title="Supermarket Ontology API",
    description="슈퍼마켓 온톨로지 기반 API",
    version="1.0.0"
)

# CORS 설정
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 라우터 등록
app.include_router(router, prefix="/api/v1", tags=["ontology"])

@app.on_event("startup")
async def startup_event():
    """앱 시작 시 온톨로지 로드"""
    logger.info("Starting Supermarket Ontology API...")

    # 온톨로지 초기화 및 추론 적용
    service = OntologyService()
    service.load_ontology("ontology/supermarket.owl")
    service.apply_reasoning("RDFS")

    stats = service.get_statistics()
    logger.info(f"Ontology statistics: {stats}")

@app.get("/health")
async def health_check():
    """헬스 체크"""
    return {"status": "healthy"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## 실행 방법

```bash
# 1. 의존성 설치
pip install -r requirements.txt

# 2. GraphDB 실행 (Docker)
docker-compose up -d graphdb

# 3. 애플리케이션 실행
python main.py

# 4. API 테스트
curl http://localhost:8000/api/v1/statistics
```

---

다음: [활용 사례](use-cases.md)에서 실제 비즈니스 시나리오와 SPARQL 쿼리 예시를 살펴봅니다.
