# Python Day 11 학습 정리
## 리스트 / 딕셔너리 중첩 구조 & 함수 개념

---

## 오늘 배운 핵심 요약

- 리스트(list), 딕셔너리(dict)가 중첩된 구조에서 데이터 꺼내는 방법
- for문을 몇 번 써야 하는지 판단하는 기준
- 이중 for문이 필요한 경우와 필요 없는 경우 구분
- 조건문과 반복문을 활용한 실전 문제 풀이
- 함수(function)의 기본 개념과 사용하는 이유
- 실무에서는 코드 길이보다 가독성과 구조 이해가 더 중요함

---

## 1. 리스트 / 딕셔너리 기본 순회

### 리스트 순회
```python
numbers = [1, 2, 3]

for n in numbers:
    print(n)
```

---

### 딕셔너리 순회
```python
user = {"name": "Kim", "age": 30}

for key in user:
    print(key)
    print(user[key])
```

- 딕셔너리를 for문으로 돌리면 key가 나온다

---

## 2. 중첩 자료구조 이해

### 구조 예시
```text
딕셔너리
 └ 리스트
    └ 딕셔너리
```

- 리스트 안에 리스트
- 딕셔너리 안에 리스트
- 위 구조에서는 이중 for문이 필요하다

---

## 3. 이중 for문 판단 기준

### 이중 for문이 필요한 경우
- for로 꺼낸 값 안에 또 여러 개의 같은 성격 데이터가 있을 때
- 카테고리 -> 상품
- 주문 -> 주문 상품

### 이중 for문이 필요 없는 경우
- for로 꺼낸 값이 단일 딕셔너리일 때
- 딕셔너리 안에 또 하나의 딕셔너리만 있을 때

---

## 4. 실전 문제 1: 권한 체크

```python
users = [
    {'name': 'Kim', 'info': {'role': 'admin', 'score': 50}},
    {'name': 'Lee', 'info': {'role': 'user', 'score': 85}},
    {'name': 'Park', 'info': {'role': 'user', 'score': 70}},
    {'name': 'Choi', 'info': {'role': 'guest', 'score': 100}}
]

access_results = []

for user in users:
    info = user['info']

    if info['role'] == 'admin':
        access_result = True
    elif info['role'] == 'user' and info['score'] >= 80:
        access_result = True
    else:
        access_result = False

    access_results.append(access_result)

print(access_results)
```

- 반복 대상은 users 하나뿐
- info는 단일 딕셔너리이므로 추가 for문이 필요 없다

---

## 5. 실전 문제 2: 쇼핑몰 상품 필터링

```python
result = []

for category in store_data:
    for product in store_data[category]:
        if product['price'] >= 50000:
            result.append(product['name'])

print(result)
```

- 카테고리 -> 상품 리스트 -> 상품
- 리스트 안에 리스트 구조이므로 이중 for문 사용

---

## 6. 실전 문제 3: 품절 주문 찾기

```python
problematic_orders = []
out_items = []

for order in orders:
    order_id = order['order_id']
    found = False

    for item in order['items']:
        if item['status'] == 'out_of_stock':
            out_items.append(item['name'])

            if not found:
                problematic_orders.append(order_id)
                found = True
```

- 주문 ID는 주문당 한 번만 저장
- 품절 상품 이름은 모두 수집

---

## 7. 함수(function) 기본 개념

```python
def add(a, b):
    return a + b
```

- 반복되는 코드를 하나의 이름으로 묶어서 재사용
- 가독성과 유지보수성 향상

---

## 오늘의 핵심 정리

- 중첩 구조인지보다 반복해야 할 대상 단계가 중요하다
- 이중 for문은 구조에 맞게 사용하는 것이다
- 실무에서는 이해하기 쉬운 코드가 가장 중요하다
