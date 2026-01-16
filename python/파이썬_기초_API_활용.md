# [2026-01-16] 파이썬_기초_API 활용(requests) 핵심 정리

오늘 학습은 `requests` 모듈을 사용하여 서버와 데이터를 주고받는 5가지 핵심 방식(GET, POST, PATCH, DELETE)과 원하는 데이터만 골라내는 필터링 기법을 중심으로 진행됨.

---

#### 1. 오늘 학습 요약 (핵심 정리)
- API 자원 관리의 기본인 **CRUD**(생성, 조회, 수정, 삭제) 개념을 파이썬 코드로 구현함.
- `JSONPlaceholder`라는 가짜(Fake) API 서버를 활용해 실습을 진행함.
- `params`를 이용한 데이터 필터링과 경로를 이용한 중첩 리소스 접근법을 익힘.

#### 2. 배운 개념 정리
- **GET (조회)**: 서버에서 데이터를 가져올 때 사용. 전체 목록이나 특정 ID의 데이터를 요청함.
- **POST (생성)**: 서버에 새로운 데이터를 등록할 때 사용. 데이터는 주로 딕셔너리 형태로 전달함.
- **PATCH (수정)**: 기존 데이터 중 일부만 변경하고 싶을 때 사용.
- **DELETE (삭제)**: 서버의 데이터를 삭제할 때 사용.
- **Status Code (상태 코드)**: 요청이 성공했는지 알려주는 숫표. (200/201: 성공, 404: 찾을 수 없음 등)

#### 3. 오늘 작성한 코드리뷰
```python
import requests
from pprint import pprint

# 1. 자원 조회 (GET) - 특정 게시글 가져오기
url = "[https://jsonplaceholder.typicode.com/posts/10](https://jsonplaceholder.typicode.com/posts/10)"
response = requests.get(url)
data = response.json() # 응답을 파이썬 딕셔너리로 변환
print(data['body']) # 데이터의 본문만 추출

# 2. 자원 생성 (POST) - 새 글 쓰기
new_url = "[https://jsonplaceholder.typicode.com/posts](https://jsonplaceholder.typicode.com/posts)"
new_data = {"title": "하하 호호", "body": "새로운 내용", "userId": 1}
# data 인자에 딕셔너리를 전달하여 서버에 생성 요청
res_post = requests.post(new_url, data=new_data)
print(res_post.status_code) # 201이 나오면 성공적으로 생성됨

# 3. 필터링 (Query Parameter)
query_url = "[https://jsonplaceholder.typicode.com/posts](https://jsonplaceholder.typicode.com/posts)"
# 특정 조건(userId가 2인 것만)을 params로 전달
filter_res = requests.get(query_url, params={"userId": 2})
pprint(filter_res.json()) # 조건에 맞는 게시글들만 출력
```

#### 4. 실수/에러와 해결 과정
- **문제**: 수정(PATCH) 실습 시 전체 데이터를 다 보내야 하는지 헷갈림.
- **해결**: PATCH는 변경하고 싶은 부분(`title` 등)만 `json` 인자에 담아 보내면 나머지는 유지된다는 것을 코드로 확인함.
- **주의**: `requests.post` 시 데이터 전송 방식에 따라 `data=` 또는 `json=` 인자를 구분해서 써야 서버가 정확히 인식함.

#### 5. 실무관점의 참견 적용해보기
- **상태 코드의 중요성**: 실무에서는 단순히 `response.json()`을 바로 찍지 않음. 반드시 `if response.status_code == 200:` 처럼 성공 여부를 먼저 체크하거나 `raise_for_status()`를 써서 프로그램이 비정상적인 데이터를 처리하지 않도록 방어막을 쳐야 함.
- **API 테스트 툴**: 코드를 짜기 전에 `Postman`이나 `Insomnia` 같은 툴로 먼저 API 주소가 잘 작동하는지 확인하면 오늘 겪었던 "왜 목록만 나오지?" 같은 시행착오를 훨씬 줄일 수 있음.