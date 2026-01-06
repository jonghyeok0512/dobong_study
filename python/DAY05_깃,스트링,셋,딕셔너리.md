## Day05

### 1. 문자열 (String)
- 문자열은 따옴표(' ', " ")로 표현
- 인덱싱과 슬라이싱 가능
- 문자열 메서드로 가공 가능

```python
text = "Hello Python"

print(text[0])            # H
print(text[0:5])          # Hello
print(text.lower())       # hello
print(text.upper())       # HELLO
print(text.replace("Python", "World"))
```

---

### 2. 집합 (Set)
- 중복을 허용하지 않는 자료형
- 순서가 없음
- 중복 제거에 유용

```python
numbers = {1, 2, 3, 3, 3}

print(numbers)  # {1, 2, 3}

numbers.add(4)
numbers.remove(2)
print(numbers)
```

---

### 3. 딕셔너리 (Dictionary)
- key-value 구조
- 의미 있는 데이터 관리에 적합

```python
person = {
    "name": "jun",
    "age": 18,
    "job": "student"
}

print(person["name"])
print(person.get("age"))
```

---

### 4. 함수 (Function)
- 반복되는 코드를 하나로 묶어 재사용
- def 키워드 사용
- return으로 결과값 반환

```python
def add(a, b):
    return a + b

result = add(3, 5)
print(result)
```

---

### 5. 모듈 (Module)
- 함수나 변수를 다른 파일에서 불러와 사용
- import 문 사용

```python
import math

print(math.sqrt(16))
```

---

### 헷갈렸던 점
- 문자열 인덱스는 0부터 시작
- set은 순서가 없어 인덱싱 불가
- 딕셔너리에서 없는 key 접근 시 에러 발생

---

### 내일 다시 볼 키워드
- 문자열 메서드
- set vs list 차이
- 함수 return 흐름
- import / from import 차이
