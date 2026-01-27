# DVD 렌탈 데이터베이스 SQL 실습 및 데이터 정합성 이해

## 1. 오늘 학습 요약
- **JOIN의 심화**: 5개 이상의 테이블을 연결하여 복잡한 데이터(고객별 대여 내역, 배우별 장르 등)를 추출함.
- **GROUP BY의 원칙**: 집계 함수를 제외한 컬럼은 모두 `GROUP BY`에 포함해야 함(표준 SQL 준수).
- **데이터 검증**: 쿼리 결과의 신뢰성을 높이기 위해 중복 가능성을 예측하고 `DISTINCT`나 `COUNT`로 검증하는 사고 과정을 학습함.
- **Git 초기화**: 프로젝트 꼬임 방지를 위해 `.git` 폴더를 삭제하고 재설정하는 물리적 초기화 방법 숙지.

---

## 2. 배운 개념 정리
### GROUP BY와 SELECT의 관계
- **정의**: 특정 컬럼을 기준으로 데이터를 그룹화하여 요약된 정보를 보여주는 구문.
- **왜 중요한지**: 단순히 목록을 보는 것과 '통계'를 내는 것을 구분하기 위함.
- **주의점**: 
    - `SELECT`에 명시된 컬럼 중 집계 함수(`COUNT`, `SUM` 등)가 아닌 것은 반드시 `GROUP BY` 절에 포함해야 에러가 없음(특히 Oracle/SQLD 환경).
    - 하지만 목록만 보고 싶을 때는 `GROUP BY`를 쓰지 않는 것이 성능상 이득임.

### 데이터 정합성 및 중복 검증
- **중복 발생 이유**: N:M(다대다) 관계(예: 한 배우가 여러 영화에 출연)를 JOIN 하면 결과 행이 뻥튀기될 수 있음.
- **검증 방법**:
    - `COUNT(*)` vs `COUNT(DISTINCT 컬럼)` 비교.
    - 특정 샘플 데이터를 추적하여 비즈니스 로직(도메인 지식)과 맞는지 대조.

---

## 3. 오늘 작성한 코드리뷰

### [배우-장르 JOIN 및 중복 제거]
```sql
/* Action 카테고리에 출연한 배우 목록 조회 
   중복 제거(DISTINCT)를 통해 배우 이름이 한 번씩만 나오게 처리
*/
SELECT DISTINCT 
    ca.name AS category_name, 
    a.first_name, 
    a.last_name
FROM actor a 
JOIN film_actor fa ON a.actor_id = fa.actor_id -- 배우와 영화 연결
JOIN film f ON fa.film_id = f.film_id         -- 영화 상세 정보
JOIN film_category fc ON f.film_id = fc.film_id -- 영화와 카테고리 연결
JOIN category ca ON fc.category_id = ca.category_id -- 카테고리 이름
WHERE ca.name = 'Action';
```
- **주의사항**: `DISTINCT`는 성능에 영향을 줄 수 있으므로, 진짜 중복이 발생하는 구조인지 먼저 파악하고 사용할 것.

### [고객 대여 내역 상세 조회]
```sql
/* 고객별 대여 영화 및 결제 정보 조회 
   집계가 필요 없는 목록 조희이므로 GROUP BY 대신 ORDER BY 사용
*/
SELECT 
    c.customer_id,
    c.first_name, 
    c.last_name, 
    f.title, 
    pay.amount,
    pay.payment_date, 
    r.rental_date 
FROM customer c
JOIN rental r ON c.customer_id = r.customer_id
JOIN payment pay ON r.rental_id = pay.rental_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
ORDER BY c.customer_id, r.rental_date DESC; -- 최신 대여순 정렬
```

---

## 4. 실수/에러와 해결 과정
- **문제**: 목록 조회 쿼리에 습관적으로 `GROUP BY`를 사용함.
- **원인**: 그룹화의 목적(통계/요약)과 단순 리스트 조회의 차이를 명확히 인지하지 못함.
- **해결**: 집계 함수가 없을 때는 `GROUP BY`를 제거하고, 데이터 구분을 위해 `ORDER BY`와 `DISTINCT`를 적절히 활용하기로 함.

---

## 5. 실무 관점의 참견
- **성능(Efficiency)**: JOIN이 많아질수록 데이터베이스 부하가 커짐. 실무에서는 `EXPLAIN` 같은 명령어로 쿼리 실행 계획을 확인하는 습관이 중요함.
- **데이터 마스킹**: 고객 정보를 조회할 때 `last_name`의 일부를 `*` 처리하는 등 보안 관점의 쿼리 작성법도 고민해 볼 필요가 있음.
- **변수명**: `ca.name` 같은 모호한 이름보다는 `category_name`처럼 직관적인 별칭(Alias)을 부여해야 협업 시 혼선이 없음.