# 데이터베이스 기초 및 SQL 활용 실습 정리

## 1. 오늘 학습 요약
* 데이터베이스의 기본 정의와 관계형(RDBMS) vs 비관계형(NoSQL)의 차이점 이해.
* SQL의 가장 기본이 되는 SELECT 문과 다양한 필터링 조건(WHERE) 활용법 습득.
* 데이터 정렬(ORDER BY) 및 특수 조건(NULL, BETWEEN, IN, LIKE) 처리 능력 함양.

---

## 2. 배운 개념 정리
### 데이터베이스 기초와 설계
* **RDBMS**: 테이블 간의 관계를 중요시하며 데이터의 무결성과 일관성을 보장함. 금융, 회원 관리 등 정확도가 중요한 서비스에 적합.
* **NoSQL**: 고정된 스키마가 없어 확장이 빠르고 유연함. 대량의 로그 데이터나 실시간 피드 처리에 유리.
* **정규화**: 데이터 중복을 최소화하여 이상 현상을 방지하는 설계 과정.
* **키(Key)**: 각 행을 식별하는 기본키(PK)와 다른 테이블을 참조하는 외래키(FK)의 역할 이해.

### SQL 주요 문법
* **WHERE 절**: 특정 조건에 맞는 데이터만 필터링. 문자열은 반드시 홑따옴표(' ') 사용.
* **LIKE/ILIKE**: 패턴 매칭. %는 0개 이상의 문자를 의미함. ILIKE는 대소문자를 구분하지 않음.
* **IN 연산자**: 여러 개의 값 중 하나라도 일치하는지 확인할 때 사용 (OR의 효율적인 대체제).
* **IS NULL**: 데이터가 비어 있는 상태를 찾을 때 사용 (= NULL로는 찾을 수 없음).
* **ORDER BY**: 데이터를 정렬하며, DESC를 붙이면 내림차순 정렬됨.

---

## 3. 오늘 작성한 코드리뷰
```sql
-- 1. 인구가 800만 이상인 도시 조회
SELECT c.name, c.population FROM city c
WHERE population >= 8000000;
-- 주의: 숫자 비교 시에는 따옴표를 쓰지 않음.

-- 2. 한국(KOR) 도시 조회
SELECT c.name, c.countrycode FROM city c
WHERE c.countrycode = 'KOR'; 
-- c라는 별칭(Alias)을 주었으므로 c.컬럼명 형태로 호출하는 습관이 좋음.

-- 3. 특정 패턴으로 시작하는 도시 조회
SELECT c.name FROM city c 
WHERE name ILIKE 'san%';
-- 'san'으로 시작하는 모든 도시를 대소문자 구분 없이 검색.

-- 4. 범위 및 다중 조건 검색 (BETWEEN, IN)
SELECT name FROM city
WHERE population BETWEEN 1000000 AND 2000000 
  AND countrycode = 'KOR';
-- BETWEEN은 이상/이하 범위를 모두 포함함.

SELECT name, countrycode, population FROM city
WHERE population >= 5000000 
  AND countrycode IN ('KOR', 'JPN', 'CHN');
-- 여러 국가를 찾을 때는 OR보다 IN이 가독성과 성능 면에서 우수함.

-- 5. 복합 정렬 (대륙별 정렬 후 GNP 내림차순)
SELECT name, continent, gnp FROM country
ORDER BY continent, gnp DESC;
-- 먼저 continent로 오름차순 정렬 후, 같은 대륙 내에서만 gnp로 내림차순 정렬함.
```

---

## 4. 실수/에러와 해결 과정
* **문자열 따옴표 누락**: `WHERE continent = Europe`이라고 작성하여 에러 발생. Europe을 컬럼명으로 인식했기 때문임. `'Europe'`으로 감싸서 문자열임을 명시하여 해결.
* **논리 연산자(OR) 오용**: `countrycode = 'KOR' OR 'JPN'` 형태는 잘못된 논리임. SQL은 각 조건마다 컬럼명을 명시해야 하므로 `countrycode = 'KOR' OR countrycode = 'JPN'`으로 쓰거나 `IN` 연산자를 사용해야 함.
* **패턴 매칭 시 컬럼명 누락**: `WHERE name LIKE 'A%' AND '%a'`처럼 작성하면 AND 뒤의 대상을 찾지 못함. `name LIKE 'A%a'`로 합치거나 `name LIKE '%a'`를 다시 써주어야 함.

---

## 5. 실무관점의 참견 적용해보기
* **데이터 타입 선택**: GNP나 수명 데이터에 쓰인 `real` 타입은 부동 소수점 오차가 발생할 수 있음. 정확한 계산이 필요한 실무 데이터에서는 `decimal` 타입을 고려하는 것이 안전함.
* **인덱스 활용**: `WHERE name LIKE '%san'`처럼 앞에 와일드카드가 붙으면 인덱스를 타지 못해 전체 데이터를 다 읽어야 함(Full Scan). 성능을 생각한다면 가급적 `'san%'`처럼 접두어 검색을 활용하는 것이 유리함.
* **NULL 처리**: `lifeexpectancy IS NULL` 조건은 실무에서 매우 중요함. 통계 데이터를 뽑을 때 NULL 값이 포함되면 평균값이 왜곡될 수 있으므로, 항상 데이터가 비어 있는 경우를 어떻게 처리할지 기획적으로 정의해야 함.