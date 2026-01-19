# [2026-01-19] 파이썬 알고리즘: API 다중 페이지 데이터 수집 및 정렬

### 1. 오늘 학습 요약
- TMDB API를 통해 1페이지만 가져오던 로직을 확장하여 **여러 페이지(100개 이상의 데이터)**를 한 번에 수집함.
- 수집된 대량의 데이터를 **특정 기준(평점)**으로 정렬하고, 필요한 정보(제목)만 추출하는 효율적인 기법을 익힘.

### 2. 배운 개념 정리
- **다중 페이지 크롤링**: `for`문과 `range()`를 사용해 페이지 번호를 바꿔가며 연속적인 API 요청을 보내는 방법.
- **`list.extend()`**: 여러 개의 리스트를 하나의 큰 리스트로 합칠 때 사용 (`append`와 차이점 인지).
- **람다(lambda) 정렬**: `sorted()` 함수의 `key` 인자에 람다식을 넣어 딕셔너리의 특정 키값을 기준으로 줄 세우기.
- **리스트 컴프리헨션(List Comprehension)**: `[결과물 for 변수 in 바구니]` 형식을 통해 3줄짜리 `for`문을 한 줄로 압축하는 기술.

### 3. 오늘 작성한 코드리뷰
```python
import requests
from pprint import pprint

# API 요청을 위한 기본 설정
URL = "[https://api.themoviedb.org/3/movie/now_playing](https://api.themoviedb.org/3/movie/now_playing)"
API_KEY = "..." # 실제 키는 보안을 위해 별도 관리 권장

params = { 'language' : 'ko-kr' }
headers = { "Authorization" : f"Bearer {API_KEY}" }

try:
    all_movies = [] # 5페이지 분량의 모든 영화 데이터를 담을 바구니
    
    # 1. 1페이지부터 5페이지까지 반복해서 요청 보냄
    for page_num in range(1, 6):
        params['page'] = page_num # 현재 요청할 페이지 번호 갱신
        response = requests.get(URL, headers=headers, params=params)
        response.raise_for_status()
            
        data = response.json()
        # append가 아닌 extend를 써서 리스트 안의 내용물(영화들)만 쏟아부음
        all_movies.extend(data['results']) 

    # 2. 수집된 모든 영화를 '평점(vote_average)' 기준, 내림차순(높은순)으로 정렬
    # lambda el : el['vote_average']는 각 영화(el)에서 평점을 꺼내서 기준으로 삼겠다는 뜻
    sorted_movies = sorted(all_movies, key=lambda el : el['vote_average'], reverse=True)
    
    # 3. 리스트 컴프리헨션을 사용해 정렬된 데이터에서 '제목'만 쏙 뽑아 새로운 리스트 생성
    movie_titles = [movie['title'] for movie in sorted_movies]

    # 결과 출력
    pprint(movie_titles)
    print(f"총 {len(movie_titles)}개의 영화 제목을 가져왔습니다.")

except Exception as e:
    print(f"오류 발생: {e}")
```

### 4. 실수/에러와 해결 과정
- **문제**: `sorted(movie_titles, ...)` 처럼 이미 제목만 뽑힌 리스트를 정렬하려 함.
- **해결**: 정렬은 '평점'이라는 정보가 살아있는 **원본 데이터 뭉치(`all_movies`)**에서 먼저 수행한 뒤, 그 다음에 제목을 추출해야 함을 깨달음.
- **문제**: 람다식 `lambda el : el['vote_average']`에서 앞의 `el`이 왜 필요한지 헷갈림.
- **해결**: 앞의 `el`은 "데이터 뭉치에서 꺼낸 하나하나에 붙이는 임시 이름표"라는 점을 이해함.

### 5. 실무관점의 참견 적용해보기
- **변수 네이밍**: `el`이나 `data` 같은 모호한 이름보다 `all_movies`, `page_data` 처럼 데이터의 범위를 명시하는 이름을 쓰는 것이 협업에 유리함.
- **데이터 가공 역량**: 실무 AI 개발에서 LLM(인공지능)에게 데이터를 넘겨주기 전, 지금처럼 필요한 데이터만 정렬하고 추출하는 전처리 과정은 모델의 성능을 결정짓는 핵심 역량임.