# 카테고리 기준 상품 리스트 그룹핑 학습 로그

## 1. 문제 정의

여러 개의 상품 데이터가 리스트 안에 딕셔너리 형태로 주어졌을 때,
각 상품의 `category` 값을 기준으로 상품 이름을 묶어
딕셔너리 형태로 그룹핑하는 것이 목표였다.

입력 데이터 구조는 다음과 같다.

- products: 리스트
- 리스트 내부 요소: 딕셔너리
  - category: 상품 분류 기준
  - name: 상품 이름

최종 목표 구조는 아래와 같다.

```text
{
  "전자제품": ["키보드", "마우스", "노트북"],
  "의류": ["티셔츠", "청바지"],
  "식품": ["사과", "배"]
}
```

---

## 2. 최종 완성 코드

```python
products = [
    {"category": "전자제품", "name": "키보드"},
    {"category": "의류", "name": "티셔츠"},
    {"category": "전자제품", "name": "마우스"},
    {"category": "전자제품", "name": "노트북"},
    {"category": "식품", "name": "사과"},
    {"category": "식품", "name": "배"},
    {"category": "의류", "name": "청바지"}
]

grouped_data = {}

for product in products:
    category = product['category']
    name = product['name']

    if category not in grouped_data:
        grouped_data[category] = []

    grouped_data[category].append(name)
```

---

## 3. 핵심 로직 설명

- products는 리스트이므로 바로 key 접근이 불가능하다.
- for문을 사용해 리스트에서 하나씩 딕셔너리를 꺼내야 한다.
- 그룹핑 기준이 되는 category는 딕셔너리의 key로 사용한다.
- category가 처음 등장하면 빈 리스트를 생성한다.
- 상품 이름(name)은 항상 해당 category 리스트에 추가한다.

핵심 포인트는
**초기화(if)와 누적(append)을 분리하는 것**이다.

---

## 4. 실수했던 부분과 헷갈렸던 점

- products는 리스트인데, products['category']처럼 접근하려고 했던 실수
- else 블록에서 리스트를 덮어써 버려 기존 값이 사라졌던 문제
- 첫 데이터가 리스트에 추가되지 않는 조건문 구조 실수
- `=` 연산자와 `append()`의 역할 차이를 혼동했던 점

---

## 5. 헷갈리기 쉬운 개념 정리

- products vs product
  - products: 전체 리스트
  - product: 리스트에서 하나씩 꺼낸 딕셔너리

- `=` vs `append()`
  - `=` : 값 자체를 새로 할당 (덮어쓰기)
  - `append()` : 기존 리스트에 값 추가

- 조건문의 역할
  - if: key가 없을 때 초기화
  - append: 항상 실행되어야 하는 누적 동작

---

## 6. 느낀 점

단순히 데이터를 묶는 문제였지만,
리스트와 딕셔너리의 구조를 정확히 이해하지 못하면
접근 자체가 틀어질 수 있다는 점을 느꼈다.

특히,
조건문 안에서 무엇을 처리하고,
조건문 밖에서 무엇을 처리해야 하는지에 따라
결과가 완전히 달라진다는 점이 인상 깊었다.

---

## 7. 한 줄 핵심 정리

딕셔너리 그룹핑 문제는
key 초기화와 값 누적을 분리해서 생각하면
가장 안전하고 읽기 쉬운 코드가 된다.
