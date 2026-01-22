# 2026-01-22 학습 기록: API 에이전트 및 보안 설정

## 1. 오늘 학습 요약
오늘은 단순한 API 호출을 넘어, 보안 수칙을 준수하며 LLM이 필요할 때 직접 외부 데이터를 가져와 사용자에게 답변하는 '대화형 에이전트'의 전체 구조를 학습했다. VS Code 환경 에러부터 API 권한 문제까지 실무에서 겪을 수 있는 다양한 트러블슈팅을 경험했다.

## 2. 배운 개념 정리
- **환경 변수(.env)와 보안**: API 키를 코드에 직접 적지 않고 외부 파일에 분리하여 관리한다. 변수명은 대문자로 작성하는 것이 관습이며, `os.getenv()`를 통해 안전하게 불러온다.
- **HTTP 헤더와 쿼리 스트링**: 네이버 API는 `X-Naver-Client-Id`와 같은 특정 헤더 명칭을 요구한다. 검색어 등은 URL 뒤에 붙는 쿼리 스트링으로 전달하며, `requests` 라이브러리의 `params` 인자를 통해 자동 인코딩 처리가 가능하다.
- **Function Calling**: 모델이 스스로 어떤 함수가 필요한지 판단하고, 함수의 인자(Arguments)를 추출해주는 기술이다. `required` 설정을 통해 필수 데이터 누락을 방지할 수 있다.
- **대화 컨텍스트 유지**: `input_list`라는 리스트에 사용자의 질문, 모델의 의도, 함수의 결과값을 순서대로 누적시켜야 AI가 이전 대화를 기억하고 답변할 수 있다.

## 3. 오늘 작성한 코드리뷰
```python
def get_naver_news(query):
    # 네이버 API가 요구하는 정확한 헤더 이름표 사용
    headers = {
        'X-Naver-Client-Id': os.getenv('NAVER_CLIENT_ID'),
        'X-Naver-Client-Secret': os.getenv('NAVER_CLIENT_SECRET')
    }
    # 검색어(query)와 출력 개수를 딕셔너리로 설정하여 전달
    params = {'query': query, 'display': 10}
    
    try:
        # GET 방식으로 요청을 보내고 에러 여부 확인
        response = requests.get(NAVER_URL, headers=headers, params=params)
        response.raise_for_status()
        # 모델이 요약하기 좋게 실제 뉴스 리스트(items)만 반환
        return response.json().get('items', [])
    except Exception as e:
        return f"에러 발생: {e}"

# while 루프를 통해 '그만'이라고 할 때까지 대화 유지
while True:
    user_query = input("\n나: ")
    if user_query.lower() in ['그만', 'q']: break
    
    # 1. 사용자 질문 저장
    input_list.append({"role": "user", "content": user_query})
    
    # 2. 1차 호출 (모델의 도구 사용 판단)
    # 3. 함수 실행 및 결과 input_list에 append
    # 4. 2차 호출 (최종 요약 답변 생성)
```

## 4. 실수/에러와 해결 과정
- **VS Code [NO CONTENT FOUND] 에러**: Webview 렌더링 문제로 인해 발생했다. 에디터를 완전히 종료 후 재시작하거나 파일을 다시 여는 방식으로 해결했다.
- **TMDB 401 Unauthorized**: API 키를 헤더가 아닌 쿼리 스트링에 잘못 넣거나 권한 설정이 미비하여 발생했다. API 문서의 인증 방식을 다시 확인하여 해결했다.
- **Naver API 서비스 URL 설정**: 비로그인 오픈 API 환경 설정 시 `http://localhost`를 서비스 URL에 등록해야 로컬 환경에서 정상적으로 호출이 가능하다는 점을 배웠다.

## 5. 실무 관점의 참견 적용해보기
- **비용 관리**: `display`를 100으로 설정하면 네이버는 많은 데이터를 주지만, 이는 LLM의 토큰 비용 상승으로 이어진다. 요약 목적이라면 상위 5~10개만 가져오는 것이 훨씬 경제적이고 답변 속도도 빠르다.
- **데이터 신뢰성**: 뉴스 API 결과에는 HTML 태그(`<b>` 등)가 포함되어 있다. 이를 정규표현식이나 `replace`로 제거하는 전처리 로직을 추가하면 AI의 요약 품질이 훨씬 깔끔해진다.
- **에러 핸들링**: `response.raise_for_status()`를 사용하여 네트워크 에러를 즉시 감지하고 사용자에게 알리는 것이 견고한 서비스의 기본이다.