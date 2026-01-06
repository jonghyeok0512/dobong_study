## Pre-study Summary (Day01~Day04)

### 1. 변수 (Variables)

- 변수(variable): 데이터의 값을 저장하는 상자  
- 할당: 변수에 데이터를 집어넣는 것  
- 할당 연산자 `=` 사용  
- `print()` : 출력하라는 뜻  

```python
name = "kyle"   # name 변수에 문자열 "kyle" 할당
age = 20        # age 변수에 정수형 20 할당
is_male = True  # is_male 변수에 불린형 True 할당

print(name)     # "kyle" 출력
print(age)      # 20 출력
print(is_male)
```
### 2. 리스트 (List)

- 여러 값을 하나로 묶어서 저장하는 자료형
- 순서가 있으며 인덱스는 0부터 시작
- 대괄호 [] 사용
```python
names = ['jun', 'alex', 'ken']

print(names[0])  # jun
print(names[1])  # alex
print(names[2])  # ken
```

### 3. 딕셔너리 (Dictionary)

- key-value 형태의 자료형
- 의미 있는 데이터 묶을 때 사용
- 중괄호 {} 사용
```python
person = {
    'name': 'jun',
    'age': 18,
    'gender': 'M'
}

print(person['name'])
print(person['age'])
print(person['gender'])
```

### 4. 식별자와 리터럴
- 식별자(identifier): 변수의 이름
- 리터럴(literal): 변수에 저장되는 실제 값

age = 20
age -> 상자
20 -> 리터럴

변수는 상자,
식별자는 상자 이름,
리터럴은 상자 안의 값

### 5. 재할당 (Reassignment)
- 한 번 할당한 변수에 다시 값을 할당 가능
- 변수는 항상 마지막 값을 기억함
```python
number = 100
number = 200
number = 300

print(number)  # 300
```

### 6. 동시 할당 (Multiple Assignment)
- 여러 변수에 값을 한 줄로 할당 가능
```python
# 같은 값 할당
x = y = 10
print(x)
print(y)

# 서로 다른 값 할당
x, y = 10, 20
print(x)
print(y)
```

### 7. 자료형 (Data Types)
- 숫자형 
int : 정수
float : 실수

- 나눗셈 연산자
/ 나누기
// 몫
% 나머지 

### 8. 조건문 (if)
- 조건이 True일때 코드 실행
- 들여쓰기로 실행 범위 구분
```python
score = 85

if score >= 80 and score <= 100:
    print("합격")
else:
    print("불합격")
```

