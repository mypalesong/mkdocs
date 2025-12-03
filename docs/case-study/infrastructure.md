# 인프라 구축

## 개요

온톨로지 기반 슈퍼마켓 시스템을 구축하기 위한 **필수 인프라**와 **기술 스택**을 설명합니다.

## 시스템 아키텍처

### 전체 구성도

```
┌─────────────────────────────────────────────────────────────────┐
│                        클라이언트 계층                            │
├─────────────────────────────────────────────────────────────────┤
│  웹 대시보드  │  모바일 앱  │  POS 시스템  │  관리자 콘솔         │
└───────┬─────────────┬─────────────┬─────────────┬───────────────┘
        │             │             │             │
        └─────────────┴──────┬──────┴─────────────┘
                             │
┌────────────────────────────┴────────────────────────────────────┐
│                        API Gateway                               │
│                    (Kong / AWS API Gateway)                      │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────┴────────────────────────────────────┐
│                      애플리케이션 계층                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  Product    │  │  Customer   │  │ Transaction │              │
│  │  Service    │  │  Service    │  │  Service    │              │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
│  ┌──────┴────────────────┴────────────────┴──────┐              │
│  │           Ontology Service Layer              │              │
│  │  ┌─────────────────────────────────────────┐  │              │
│  │  │  • SPARQL Query Engine                  │  │              │
│  │  │  • Reasoning Engine                     │  │              │
│  │  │  • Ontology Management API              │  │              │
│  │  └─────────────────────────────────────────┘  │              │
│  └───────────────────────┬───────────────────────┘              │
│                          │                                       │
└──────────────────────────┼──────────────────────────────────────┘
                           │
┌──────────────────────────┴──────────────────────────────────────┐
│                        데이터 계층                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │   Triple Store  │  │   PostgreSQL    │  │   Redis Cache   │  │
│  │   (GraphDB /    │  │   (Operational  │  │   (Query Cache) │  │
│  │    Apache Jena) │  │    Data)        │  │                 │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐                       │
│  │  Elasticsearch  │  │   Message Queue │                       │
│  │  (Full-text     │  │   (Kafka/       │                       │
│  │   Search)       │  │    RabbitMQ)    │                       │
│  └─────────────────┘  └─────────────────┘                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 핵심 컴포넌트

### 1. Triple Store (트리플 저장소)

!!! info "Triple Store란?"
    RDF 트리플(주어-서술어-목적어)을 저장하고 SPARQL 쿼리를 처리하는 데이터베이스입니다.

#### 주요 옵션 비교

| 솔루션 | 라이선스 | 추론 | 확장성 | 특징 |
|--------|---------|------|--------|------|
| **GraphDB** | 상용/무료 | ✓ 강력 | ✓ | 기업용, 우수한 추론 |
| **Apache Jena Fuseki** | 오픈소스 | ✓ | 중간 | 무료, Java 생태계 |
| **Blazegraph** | 오픈소스 | ✓ | ✓ | 고성능, Wikidata 사용 |
| **Stardog** | 상용 | ✓ 강력 | ✓ | 가상 그래프, ML 통합 |
| **Amazon Neptune** | 클라우드 | 제한적 | ✓ | AWS 통합, 관리형 |
| **Virtuoso** | 오픈소스/상용 | ✓ | ✓ | 하이브리드 DB |

#### 권장 구성: GraphDB

```yaml
# docker-compose.yml - GraphDB 설정
version: '3.8'
services:
  graphdb:
    image: ontotext/graphdb:10.4.0
    container_name: supermarket-graphdb
    ports:
      - "7200:7200"
    volumes:
      - graphdb-data:/opt/graphdb/home
      - ./ontology:/opt/graphdb/import
    environment:
      - GDB_HEAP_SIZE=4g
      - GDB_MIN_MEM=2g
      - GDB_MAX_MEM=4g
    restart: unless-stopped

volumes:
  graphdb-data:
```

### 2. 추론 엔진 (Reasoning Engine)

추론 엔진은 명시적 지식에서 암묵적 지식을 도출합니다.

```
명시적 지식                      추론된 지식
─────────────                   ─────────────
Banana → subClassOf → Fruit     Banana → subClassOf → Product
Fruit → subClassOf → Product    (추론됨)
```

#### 추론 수준

| 수준 | 설명 | 성능 | 사용 사례 |
|------|------|------|----------|
| **RDFS** | 기본 클래스 계층 | 빠름 | 단순 분류 |
| **OWL-RL** | 규칙 기반 OWL | 중간 | 대부분의 비즈니스 규칙 |
| **OWL-DL** | 완전한 기술 논리 | 느림 | 복잡한 제약조건 |

```python
# 추론 설정 예시 (rdflib + owlrl)
from rdflib import Graph
import owlrl

def apply_reasoning(graph: Graph, reasoning_type: str = "RDFS"):
    """온톨로지에 추론 적용"""
    if reasoning_type == "RDFS":
        owlrl.DeductiveClosure(owlrl.RDFS_Semantics).expand(graph)
    elif reasoning_type == "OWL-RL":
        owlrl.DeductiveClosure(owlrl.OWLRL_Semantics).expand(graph)
    return graph
```

### 3. SPARQL 엔드포인트

SPARQL 쿼리를 처리하는 HTTP 엔드포인트:

```python
# FastAPI SPARQL 엔드포인트 예시
from fastapi import FastAPI, HTTPException
from SPARQLWrapper import SPARQLWrapper, JSON

app = FastAPI()

SPARQL_ENDPOINT = "http://graphdb:7200/repositories/supermarket"

@app.post("/sparql")
async def execute_sparql(query: str):
    """SPARQL 쿼리 실행"""
    sparql = SPARQLWrapper(SPARQL_ENDPOINT)
    sparql.setQuery(query)
    sparql.setReturnFormat(JSON)

    try:
        results = sparql.query().convert()
        return results
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))
```

### 4. 캐시 계층 (Redis)

자주 사용되는 쿼리 결과 캐싱:

```python
import redis
import hashlib
import json
from functools import wraps

redis_client = redis.Redis(host='localhost', port=6379, db=0)

def cache_sparql_result(ttl: int = 300):
    """SPARQL 결과 캐싱 데코레이터"""
    def decorator(func):
        @wraps(func)
        def wrapper(query: str, *args, **kwargs):
            # 쿼리 해시로 캐시 키 생성
            cache_key = f"sparql:{hashlib.md5(query.encode()).hexdigest()}"

            # 캐시 확인
            cached = redis_client.get(cache_key)
            if cached:
                return json.loads(cached)

            # 쿼리 실행
            result = func(query, *args, **kwargs)

            # 결과 캐싱
            redis_client.setex(cache_key, ttl, json.dumps(result))
            return result
        return wrapper
    return decorator
```

## 기술 스택 상세

### 백엔드

```yaml
# 권장 기술 스택
Backend:
  Language: Python 3.11+
  Framework: FastAPI
  ORM: SQLAlchemy (관계형), RDFLib (트리플)

RDF/OWL Libraries:
  - rdflib: RDF 그래프 조작
  - owlrl: OWL 추론
  - SPARQLWrapper: SPARQL 클라이언트
  - pyshacl: SHACL 검증

Data Storage:
  - GraphDB: 트리플 스토어 (주)
  - PostgreSQL: 운영 데이터
  - Redis: 캐시
  - Elasticsearch: 전문 검색

Message Queue:
  - Apache Kafka: 이벤트 스트리밍
  - Celery: 비동기 작업
```

### Python 의존성

```txt
# requirements.txt
fastapi>=0.104.0
uvicorn>=0.24.0
rdflib>=7.0.0
owlrl>=6.0.2
SPARQLWrapper>=2.0.0
pyshacl>=0.24.0
redis>=5.0.0
sqlalchemy>=2.0.0
pydantic>=2.5.0
celery>=5.3.0
kafka-python>=2.0.0
elasticsearch>=8.0.0
```

### Docker Compose 전체 구성

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Triple Store
  graphdb:
    image: ontotext/graphdb:10.4.0
    ports:
      - "7200:7200"
    volumes:
      - graphdb-data:/opt/graphdb/home
    environment:
      - GDB_HEAP_SIZE=4g

  # 관계형 DB
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: supermarket
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  # 캐시
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data

  # 검색 엔진
  elasticsearch:
    image: elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    volumes:
      - es-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"

  # 메시지 큐
  kafka:
    image: confluentinc/cp-kafka:7.5.0
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"

  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  # 애플리케이션
  api:
    build: ./api
    ports:
      - "8000:8000"
    depends_on:
      - graphdb
      - postgres
      - redis
    environment:
      - GRAPHDB_URL=http://graphdb:7200
      - DATABASE_URL=postgresql://admin:${DB_PASSWORD}@postgres/supermarket
      - REDIS_URL=redis://redis:6379

volumes:
  graphdb-data:
  postgres-data:
  redis-data:
  es-data:
```

## 온톨로지 관리 워크플로우

### CI/CD 파이프라인

```yaml
# .github/workflows/ontology-ci.yml
name: Ontology CI/CD

on:
  push:
    paths:
      - 'ontology/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install rdflib pyshacl

      - name: Validate OWL syntax
        run: |
          python -c "
          from rdflib import Graph
          g = Graph()
          g.parse('ontology/supermarket.owl', format='xml')
          print(f'Loaded {len(g)} triples')
          "

      - name: Run SHACL validation
        run: |
          python -c "
          from pyshacl import validate
          from rdflib import Graph

          data = Graph().parse('ontology/supermarket.owl')
          shapes = Graph().parse('ontology/shapes.ttl')

          conforms, _, report = validate(data, shacl_graph=shapes)
          print(report)
          assert conforms, 'SHACL validation failed'
          "

  deploy:
    needs: validate
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to GraphDB
        run: |
          curl -X POST \
            -H "Content-Type: application/x-turtle" \
            --data-binary @ontology/supermarket.owl \
            ${{ secrets.GRAPHDB_URL }}/repositories/supermarket/statements
```

## 모니터링 및 관찰성

### 메트릭 수집

```python
# Prometheus 메트릭 예시
from prometheus_client import Counter, Histogram, start_http_server

# SPARQL 쿼리 메트릭
sparql_queries = Counter(
    'sparql_queries_total',
    'Total SPARQL queries',
    ['query_type', 'status']
)

sparql_duration = Histogram(
    'sparql_query_duration_seconds',
    'SPARQL query duration',
    ['query_type']
)

# 온톨로지 메트릭
ontology_triples = Gauge(
    'ontology_triples_total',
    'Total triples in ontology'
)
```

### Grafana 대시보드

```json
{
  "dashboard": {
    "title": "Supermarket Ontology Metrics",
    "panels": [
      {
        "title": "SPARQL Query Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(sparql_queries_total[5m])"
          }
        ]
      },
      {
        "title": "Query Latency (p95)",
        "type": "gauge",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sparql_query_duration_seconds)"
          }
        ]
      }
    ]
  }
}
```

## 보안 고려사항

### 인증/인가

```python
# JWT 기반 SPARQL 접근 제어
from fastapi import Depends, HTTPException
from fastapi.security import HTTPBearer

security = HTTPBearer()

class SPARQLPermission:
    """SPARQL 쿼리 권한 검사"""

    ALLOWED_OPERATIONS = {
        "viewer": ["SELECT", "ASK", "DESCRIBE"],
        "editor": ["SELECT", "ASK", "DESCRIBE", "INSERT", "DELETE"],
        "admin": ["*"]
    }

    @classmethod
    def check(cls, query: str, role: str) -> bool:
        if role == "admin":
            return True

        allowed = cls.ALLOWED_OPERATIONS.get(role, [])
        operation = query.strip().split()[0].upper()

        return operation in allowed
```

### 쿼리 검증

```python
def sanitize_sparql(query: str) -> str:
    """위험한 SPARQL 패턴 차단"""
    dangerous_patterns = [
        "DROP",
        "CLEAR",
        "DELETE WHERE",
        "SERVICE",  # 외부 엔드포인트 차단
    ]

    query_upper = query.upper()
    for pattern in dangerous_patterns:
        if pattern in query_upper:
            raise ValueError(f"Forbidden operation: {pattern}")

    return query
```

---

다음: [구현 가이드](implementation.md)에서 실제 코드로 시스템을 구현합니다.
