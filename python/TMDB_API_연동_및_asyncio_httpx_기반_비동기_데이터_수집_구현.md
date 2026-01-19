# 오늘 학습 요약 (실전 복습용)

- **핵심 목표**: TMDB API에서 현재 상영 영화 목록을 가져온 뒤, 각 영화의 '수익(revenue)' 정보를 포함한 [제목, 평점, 수익] 리스트 만들기.
- **성과**: 동기 방식보다 수십 배 빠른 비동기 처리(asyncio/httpx)를 통해 실무형 데이터 파이프라인 구조를 완성함.

---

## 1. 오늘 학습 핵심 개념

### ① API 활용 및 환경변수 (.env)
- **TMDB API**: 영화 데이터를 제공하는 외부 서비스와 통신하는 법을 익힘.
- **보안(Environment Variables)**: API 키처럼 중요한 정보를 코드에 직접 적지 않고 `.env` 파일에 저장하여 `os.getenv()`로 불러오는 실무 보안 원칙을 적용함.

### ② 비동기 프로그래밍 (asyncio / httpx)
- **개념**: 여러 개의 주소에 배달을 보낼 때, 한 명의 배달원이 다 돌아올 때까지 기다리는 게 아니라(동기), 여러 명의 배달원을 한꺼번에 출발시키는 방식(비동기).
- **효율성**: 네트워크 응답을 기다리는 '대기 시간'을 활용해 다른 작업을 처리함으로써 전체 실행 시간을 획기적으로 단축함.

---

## 2. 오늘 작성한 최종 코드 리뷰

```python
import requests
import httpx
import asyncio
import os
from dotenv import load_dotenv
from pprint import pprint

load_dotenv() # 환경변수 로드

# 1. 동기 방식으로 영화 목록(ID 리스트) 가져오기
URL = "[https://api.themoviedb.org/3/movie/now_playing](https://api.themoviedb.org/3/movie/now_playing)"
API_KEY = os.getenv('TMDB_API_KEY')
headers = {"Authorization" : f"Bearer {API_KEY}"}
params = {'language' : 'ko-kr'}

response = requests.get(URL, headers=headers, params=params)
# .get()을 사용하여 데이터가 없을 경우에도 에러 없이 빈 리스트를 반환하게 함 (안정성)
data = response.json().get('results', [])

# 2. 비동기 요청을 위한 상세 페이지 URL 리스트 생성 (리스트 컴프리헨션)
url_lst = [f"[https://api.themoviedb.org/3/movie/](https://api.themoviedb.org/3/movie/){m['id']}" for m in data]

# 3. 비동기 상세 정보 수집 함수
async def fetch_detail(client, url):
    # 비동기 클라이언트를 이용해 상세 데이터 요청 (헤더 포함 필수!)
    resp = await client.get(url, headers=headers)
    detail = resp.json()
    
    # 필요한 정보만 딕셔너리로 추출하여 반환
    return {
        "title": detail.get('title'),
        "vote_average": detail.get('vote_average'),
        "revenue": detail.get('revenue', 0) # 수익 정보가 없으면 0으로 처리
    }

# 4. 전체 비동기 흐름 제어
async def main():
    # 세션을 하나만 열어 모든 요청에서 재사용 (연결 비용 절감)
    async with httpx.AsyncClient() as client:
        # 모든 URL에 대한 실행 예약(Task) 리스트 생성
        tasks = [fetch_detail(client, url) for url in url_lst]
        # 예약된 모든 작업을 동시에 실행하고 결과를 한꺼번에 수집
        results = await asyncio.gather(*tasks)
        pprint(results)

# 5. 실행부 (Jupyter 환경에서는 await main() 사용)
await main()
```

---

## 3. 실수와 배움 (사고 과정의 흔적)

| 구분 | 내용 | 해결 및 배운 점 |
| :--- | :--- | :--- |
| **코드 오타** | `[data for i in data['id']]` | 리스트 컴프리헨션의 순서(`[표현식 for 변수 in 리스트]`)를 다시 익힘. |
| **환경 에러** | `asyncio.run()` 실행 시 RuntimeError 발생 | 주피터는 이미 엔진이 돌아가고 있으므로 `await`로 바로 실행해야 함을 이해함. |
| **인증 에러** | 비동기 함수 내에서 `headers` 누락 | API 서버는 매 요청마다 '열쇠(인증)'를 확인하므로 모든 요청에 헤더를 넣어야 함. |
| **라이브러리** | `httpx` 대신 `https` 임포트 시도 | `https`는 약속(규약)이고, `httpx`는 그 약속을 지키며 일하는 도구(라이브러리)임을 구분함. |
| **논리 오류** | `now_playing`에 `revenue`가 없음을 발견 | 하나의 API가 모든 정보를 주지 않을 때, ID를 뽑아 상세 API를 다시 호출하는 '데이터 보완' 방식을 학습함. |

---

## 4. 실무 관점의 참견 (나의 역량 성장)

1. **안정성 최우선**: `data['results']` 대신 `.get('results', [])`를 쓰는 습관은 실무에서 프로그램이 갑자기 죽는 사고를 막아주는 아주 훌륭한 습관이다.
2. **비동기 활용**: AI 서비스 개발자로서 대량의 데이터를 크롤링하거나 여러 모델의 결과를 합칠 때 오늘 배운 `asyncio.gather`는 필수 무기가 될 것이다.
3. **가독성**: 리스트 컴프리헨션(`[... for ... in ...]`)을 통해 코드를 간결하게 짜는 것은 협업 효율을 높여주는 기술이다.