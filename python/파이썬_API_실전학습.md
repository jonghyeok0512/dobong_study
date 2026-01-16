# [2026-01-16] 파이썬 API 실전 및 CRUD 개념 심화 학습

### 1. 오늘 학습 요약 (핵심 정리)
오늘은 API를 통해 단순히 데이터를 가져오는 것을 넘어, 가져온 데이터를 가공(정렬)하고 그 결과를 다시 다른 API의 입력값으로 사용하는 '실무형 데이터 흐름'을 학습했다. 특히 HTTP 메서드(GET, POST, PATCH, DELETE)를 이용한 자원 관리의 기초를 다졌다.

### 2. 배운 개념 정리
#### (1) API 체이닝 (API Chaining)
- 정의: 한 API의 응답 데이터(예: 영화 ID)를 다음 API 호출의 주소나 파라미터로 사용하는 방식.
- 왜 중요한가: 실무에서는 한 번의 호출로 모든 정보를 주지 않기 때문에, 필요한 정보를 얻기 위해 여러 번 전화를 거는 과정이 필수적이다.

#### (2) 데이터 정렬과 가공 (Sorted & Lambda)
- 복잡한 딕셔너리 리스트에서 특정 키(`vote_average`)를 기준으로 순서를 바꾸는 법을 익혔다.
- `reverse=True`를 사용해 내림차순(큰 수부터) 정렬하는 것이 핵심이다.

#### (3) REST API 자원 관리 (CRUD)
- GET: 조회 (게시글 읽기)
- POST: 생성 (새 게시글 쓰기)
- PATCH: 일부 수정 (제목만 바꾸기)
- DELETE: 삭제 (글 지우기)

### 3. 오늘 작성한 코드리뷰
#### [TMDB 실전 문제 1~5번 통합]
```python
import requests
from pprint import pprint

# 공통 설정
BASE_URL = "[https://api.themoviedb.org/3](https://api.themoviedb.org/3)"
API_KEY = "내_API_키"
headers = {"Authorization": f"Bearer {API_KEY}"}
params = {"language": "ko-KR"}

try:
    # [과정 1] 현재 상영 영화 목록 가져오기
    list_url = f"{BASE_URL}/movie/now_playing"
    response = requests.get(list_url, headers=headers, params=params)
    response.raise_for_status() # 통신 실패 시 에러 발생시켜 중단
    
    data = response.json()
    movies = data['results'] # 영화 정보가 담긴 '리스트' 추출
    
    # [과정 2] 평점순 내림차순 정렬
    # x는 리스트 안의 영화 한 편(딕셔너리)을 의미함
    sorted_movies = sorted(movies, key=lambda x: x['vote_average'], reverse=True)
    
    # [과정 3] 최고 평점 영화 ID 추출
    top_movie_id = sorted_movies[0]['id']
    
    # [과정 4] 상세 정보 조회를 위한 두 번째 API 호출
    # 여기서 URL 변수명을 다르게 하여 이전 주소와 섞이지 않게 주의
    detail_url = f"{BASE_URL}/movie/{top_movie_id}"
    
    # 상세 정보 호출 시에는 기존 목록용 params가 방해될 수 있어 headers만 사용하거나 
    # 상세용 params를 새로 정의함
    detail_response = requests.get(detail_url, headers=headers)
    detail_data = detail_response.json()
    
    # [과정 5] 최종 데이터 추출
    revenue = detail_data.get('revenue')
    title = detail_data.get('title')
    
    print(f"평점 1위 영화: {title} / 수익: ${revenue:,}")

except Exception as e:
    print(f"오류 발생: {e}")
```

### 4. 실수/에러와 해결 과정
#### (1) "목록 데이터가 자꾸 반복되는 문제"
- **현상**: 두 번째 상세 정보를 요청했는데 자꾸 첫 번째 영화 목록(`results`)이 출력됨.
- **원인**: `URL` 변수를 덮어쓰거나, 두 번째 `requests.get()`을 보낼 때 첫 번째에서 쓰던 `params`를 그대로 넣어서 서버가 목록 요청으로 오해함.
- **해결**: 상세 정보용 URL 변수명을 `detail_url`로 명확히 나누고, 두 번째 요청 시 `params`를 빼고 호출하여 정확한 상세 정보를 받아냄.

#### (2) "데이터 구조(List vs Dict) 접근 실수"
- **현상**: `data['title']`을 호출했는데 에러가 남.
- **원인**: `data`는 전체 응답 뭉치이고, 실제 제목은 `data['results']`라는 '리스트' 안에 들어있었음.
- **해결**: `pprint`로 구조를 먼저 찍어본 뒤, `data['results'][0]['title']`처럼 단계별로 접근해야 함을 인지함.

### 5. 실무관점의 참견 적용해보기
- **변수 네이밍**: `data1`, `data2`보다는 `movie_list`, `movie_detail`처럼 데이터의 정체를 알 수 있는 이름을 써야 코드가 길어져도 헷갈리지 않는다.
- **방어적 코드**: API가 언제나 완벽한 데이터를 주는 것은 아니다. 수익(`revenue`)이 없는 경우 `0`으로 올 수 있으므로, `if not revenue: print("수익 정보 없음")` 같은 처리를 해주는 것이 사용자 경험에 좋다.
- **상태 코드 확인**: `response.status_code`가 200인지 확인하는 습관은 서버 에러인지 내 코드 에러인지 구분하는 가장 빠른 길이다.