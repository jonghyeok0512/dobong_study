# 2026-01-26 SQL 학습 기록: 집계 함수와 그룹화

## 1. 오늘 학습 요약
- SQL 키워드 `DESC`의 이중적 의미 이해
- `GROUP BY`와 집계 함수(`COUNT`, `SUM`, `AVG`)의 기본 사용법
- `WHERE`와 `GROUP BY`의 결합 및 실행 순서 파악
- `ROUND` 함수를 이용한 수치 데이터 정형화

## 2. 배운 개념 정리
- **DESC의 두 가지 얼굴**:
    - `DESC [테이블명]`: Describe의 약자. 테이블 구조(스키마)를 확인할 때 사용.
    - `ORDER BY [컬럼] DESC`: Descending의 약자. 내림차순 정렬 시 사용.
- **GROUP BY**: 특정 컬럼을 기준으로 데이터를 그룹핑하는 기능. 반드시 집계 함수와 함께 사용하여 그룹별 통계를 냄.
- **ROUND(값, 자릿수)**: 소수점 반올림 함수. 두 번째 인자가 0이면 정수로 반올림됨.

## 3. 오늘 작성한 코드리뷰
```sql
-- 1. 인구 50만~100만 사이 도시의 District별 도시 수
SELECT district, count(*) 
FROM city
WHERE population BETWEEN 500000 AND 1000000 -- 그룹화 전 범위 필터링
GROUP BY district; 

-- 2. 대륙별 평균 인구 및 GNP (반올림 처리)
SELECT 
    continent, 
    ROUND(AVG(population), 0) AS 인구평균, -- 소수점 첫째자리에서 반올림하여 정수화
    ROUND(AVG(gnp), 0) AS GNP평균
FROM country
GROUP BY continent;

-- 3. 아시아 대륙 내 지역(Region)별 총 GNP
SELECT region, sum(gnp) as 총GNP
FROM country
WHERE continent = 'Asia' -- 특정 대륙만 먼저 필터링
GROUP BY region;
```

## 4. 실수/에러와 해결 과정
- **문제**: 평균값을 구할 때 소수점이 너무 길게 나와 가독성이 떨어짐.
- **해결**: `ROUND()` 함수를 사용하여 해결. `ROUND(데이터, 0)`을 통해 깔끔한 정수 형태로 출력하는 법을 익힘.
- **주의**: `GROUP BY`를 쓸 때 `SELECT` 절에는 그룹화 기준이 된 컬럼이나 집계 함수만 들어갈 수 있다는 점을 명심해야 함.

## 5. 실무관점의 참견
- **데이터 정형화**: 보고서용 쿼리를 작성할 때는 `AS`를 사용해 한글이나 명확한 별칭을 주는 것이 중요함.
- **성능 최적화**: `HAVING`은 그룹화가 완료된 후 전체 결과 세트에서 필터링하므로, 가능하다면 `WHERE`에서 먼저 데이터를 쳐내는 것이 성능상 유리함.