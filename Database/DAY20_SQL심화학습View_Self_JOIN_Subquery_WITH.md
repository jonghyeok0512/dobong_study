# SQL 심화 학습: 가상 테이블과 복잡한 관계의 해결 (View, Self JOIN, Subquery, WITH)

## 1. 오늘 학습 요약
- **View**: 자주 사용하는 복잡한 쿼리를 '가짜 테이블'로 만들어 편의성과 보안을 높임.
- **Self JOIN**: 하나의 테이블을 두 개처럼 사용하여 댓글-대댓글 같은 계층 구조를 해결함.
- **Subquery**: 쿼리 안에 또 다른 쿼리를 넣어 데이터를 단계적으로 필터링하거나 계산함.
- **WITH (CTE)**: 복잡한 서브쿼리에 이름을 붙여 코드의 가독성을 높이고 재사용 가능하게 함.

---

## 2. 배운 개념 정리

### (1) View (가상 테이블)
- **정의**: 실제 데이터를 저장하지는 않지만, 실행 결과만 보여주는 '저장된 쿼리'임.
- **왜 중요한지**: 
    - 보안: 사용자에게 주민번호 같은 민감한 정보는 빼고 이름과 아이디만 들어있는 View만 보여줄 수 있음.
    - 편의성: 오늘 짠 5개 테이블 JOIN 쿼리 같은 걸 View로 만들어두면, 다음부턴 그냥 `SELECT * FROM 뷰이름`으로 끝낼 수 있음.
- **주의점**: 뷰를 통해 데이터를 수정할 때는 제약이 많으므로 주로 '조회용'으로 사용함.

### (2) Self JOIN (셀프 조인)
- **정의**: 똑같은 테이블을 별칭(Alias)만 다르게 해서 두 번 JOIN 하는 것.
- **핵심 원리**: '댓글' 테이블에서 `댓글ID`와 `부모댓글ID`를 연결함.
    - 왼쪽(a)은 '부모 댓글' 역할로 쓰고, 오른쪽(b)은 '자식 대댓글' 역할로 쓰는 식임.
- **주의점**: 테이블 별칭을 주지 않으면 컴퓨터가 어떤 테이블의 컬럼인지 몰라서 에러를 뱉음.

### (3) Subquery (서브쿼리)
- **정의**: SQL 문장 안에 삽입된 또 다른 `SELECT` 문.
- **위치별 명칭**:
    - SELECT 절에 있으면: 스칼라 서브쿼리
    - FROM 절에 있으면: 인라인 뷰
    - WHERE 절에 있으면: 중첩 서브쿼리
- **왜 중요한지**: "평균 대여료보다 비싼 영화들만 보여줘"처럼 '평균값'을 먼저 구해야 다음 단계를 할 수 있는 상황에서 필수임.

### (4) WITH (CTE - Common Table Expression)
- **정의**: 서브쿼리를 미리 정의해서 임시로 이름을 붙여두는 것.
- **왜 중요한지**: 서브쿼리가 너무 많아지면 괄호가 겹쳐서 가독성이 떨어지는데, WITH를 쓰면 "먼저 이 임시 테이블을 만들고, 그다음에 본 쿼리를 실행해"라는 순서가 생겨서 이해하기 쉬움.

---

## 3. 오늘 작성한 코드리뷰

### [Self JOIN을 활용한 댓글-대댓글 관계 조회]
```sql
/* comments 테이블에서 댓글(p)과 그에 달린 대댓글(c)을 연결함.
   부모가 없는 댓글(최상위 댓글)도 보여주기 위해 LEFT JOIN 사용.
*/
SELECT 
    p.content AS parent_comment, -- 부모 댓글 내용
    c.content AS reply_content   -- 대댓글 내용
FROM comments p
LEFT JOIN comments c ON p.comment_id = c.parent_id
WHERE p.parent_id IS NULL; -- 최상위 댓글을 기준으로 그 밑에 달린 것들을 찾음
```
- **주의사항**: `ON p.comment_id = c.parent_id` 부분이 핵심임. 부모의 ID와 자식이 가리키는 부모ID를 매칭하는 것임.

### [WITH와 Subquery를 활용한 실무형 쿼리]
```sql
/* 카테고리별 매출 합계를 먼저 구하고(WITH), 
   그중에서 전체 평균보다 높은 카테고리만 조회함.
*/
WITH category_revenue AS (
    SELECT 
        c.name AS category_name,
        SUM(p.amount) AS total_sales
    FROM category c
    JOIN film_category fc ON c.category_id = fc.category_id
    JOIN inventory i ON fc.film_id = i.film_id
    JOIN rental r ON i.inventory_id = r.inventory_id
    JOIN payment p ON r.rental_id = p.rental_id
    GROUP BY c.name
)
SELECT * FROM category_revenue
WHERE total_sales > (SELECT AVG(total_sales) FROM category_revenue); -- 평균보다 높은 것만!
```
- **코드 설명**: `WITH`로 매출표를 먼저 만들어서 `category_revenue`라는 이름을 붙였기 때문에, 밑에서 일반 테이블처럼 편하게 갖다 쓸 수 있음.

---

## 4. 실수/에러와 해결 과정
- **문제**: Self JOIN 시 컬럼이 모호하다는(Ambiguous) 에러 발생.
- **원인**: 같은 테이블을 두 번 쓰면서 별칭(p, c 등)을 컬럼 앞에 붙여주지 않아 컴퓨터가 헷갈려 함.
- **해결**: 모든 컬럼 앞에 반드시 `p.content`, `c.content`처럼 별칭을 붙여서 출처를 명확히 함.

---

## 5. 실무 관점의 참견 적용해보기
- **RAG 구조와의 연관성**: 나중에 AI Agent를 개발할 때, 문서의 계층 구조(장-절-단락)를 DB에 저장하고 Self JOIN으로 불러오는 경우가 많음. 이 구조를 잘 이해해야 AI가 문맥을 정확히 파악하도록 데이터를 전달할 수 있음.
- **SQLD 자격증 팁**: 
    - 서브쿼리는 SQLD의 단골 문제임. 특히 `IN`, `ANY`, `ALL` 같은 연산자와 함께 쓰이는 중첩 서브쿼리를 잘 봐둬야 함.
    - WITH절은 가독성을 위해 실무에서 강력 추천하지만, 성능(Index 활용 등) 면에서는 상황에 따라 서브쿼리가 더 빠를 수도 있다는 점을 인지해야 함.
- **변수명(영어)**: 
    - 댓글: `comment` / 대댓글: `reply`
    - 부모: `parent` / 자식: `child`
    - 가상 테이블 이름: `temp_table`보다는 `category_summary`처럼 목적이 드러나게 짓는 것이 좋음.