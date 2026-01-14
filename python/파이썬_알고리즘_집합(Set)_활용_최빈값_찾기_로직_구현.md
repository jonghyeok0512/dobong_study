# 파이썬 알고리즘: 집합(Set)을 활용한 최빈값 찾기

## 1. 문제
- 주어진 숫자 리스트에서 가장 많이 등장하는 숫자(최빈값)를 찾아 반환하기.
- 만약 최빈값이 여러 개라면, 그중 가장 작은 숫자를 반환해야 함.

## 2. 학습 포인트와 문제 풀이 핵심
- `set()`을 활용해 리스트의 중복을 제거하면, 전체 리스트를 반복해서 뒤질 필요 없이 고유한 숫자들에 대해서만 개수를 셀 수 있음.
- 최댓값(`max_count`)을 저장할 변수를 두고, 루프를 돌며 더 큰 빈도수가 나올 때마다 정답을 갱신하는 것이 핵심.

## 3. 풀이 과정
```python
def solution(numbers):
    answer = 0
    # 전체 리스트를 다 도는 건 비효율적이라 set으로 중복 제거한 보따리를 만듦
    unique_numbers = set(numbers)
    max_count = 0 
    
    for num in unique_numbers:
        # 원본 리스트에서 현재 숫자(num)가 몇 번 나오는지 측정
        current_count = numbers.count(num)
        
        # 생각의 흐름:
        # 1. 지금 센 개수가 여태까지 본 최대 개수보다 많으면? 
        # 2. 바로 무조건 정답을 이 숫자로 갈아치운다.
        if max_count < current_count:
            max_count = current_count
            answer = num
            
        # 만약 최대 개수가 똑같은 숫자가 또 나왔다면?
        # 문제 조건대로 더 작은 숫자를 정답으로 선택함
        elif max_count == current_count and answer > num:
            answer = num
            
    return answer
```
## 4. 내가 실수했던거나 어려웠던 부분
- 디버깅 기록: unique_numbersa라고 오타를 내서 NameError 발생.

- 원인 분석: 에러 메시지의 is not defined를 확인하고 변수 선언부와 대조함. 변수명이 길어질수록 오타 확률이 높으니 오타 검수를 꼼꼼히 하거나 자동 완성 기능을 더 적극적으로 써야겠음.

## 5. 이번학습으로 내가 얻은내용이나 알고가면 좋은것들
- 성능 리뷰: 이 방식의 시간 복잡도는 O(U * N)임. (U는 고유 숫자의 개수, N은 리스트 전체 길이)

- 실무/RAG 관점: 최빈값 로직은 사용자 추천 검색어 순위를 매기거나, RAG 시스템에서 여러 문서 중 공통적으로 가장 많이 언급된 핵심 키워드를 추출할 때 응용할 수 있음.

- 팁: 데이터 종류가 수만 개 이상으로 많아질 때는 파이썬 기본 라이브러리인 collections.Counter를 사용하는 게 성능상 훨씬 유리함.