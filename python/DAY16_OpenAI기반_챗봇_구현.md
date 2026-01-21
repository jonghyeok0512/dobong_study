# [학습 기록] OpenAI SDK를 활용한 컨텍스트 유지형 챗봇 구현

### 1. 오늘 학습 요약
* OpenAI Python SDK의 `client.chat.completions.create` 사용법 숙달
* 리스트(`history`)를 활용한 대화 문맥(Context) 유지 원리 이해
* `while` 루프와 조건문을 이용한 터미널 기반 대화 인터페이스 구축
* 세션 종료 시 대화 기록을 초기화하여 보안 및 메모리 관리 적용

---

### 2. 배운 개념 정리
* **Context Window**: 모델이 한 번에 기억할 수 있는 정보의 양. 대화가 길어질수록 `history` 리스트 전체를 다시 보내야 하므로 비용과 토큰 관리가 중요함.
* **Role 구분**: `system`(지침), `user`(질문), `assistant`(답변)로 역할을 나누어 메시지 구조를 설계해야 모델이 대화 흐름을 정확히 파악함.
* **State Management**: `history = [...]`와 같이 변수를 재할당하여 프로그램 실행 중 상태를 초기화하는 방법.

---

### 3. 오늘 작성한 코드리뷰

```python
import os
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()

# API 클라이언트 초기화 및 모델 설정
OPENAI_API_KEY = os.getenv('OPENAI_API_KEY')
client = OpenAI(api_key=OPENAI_API_KEY)
model = "gpt-4o-mini"

# 대화 기록 초기화: 시스템 메시지는 챗봇의 정체성을 결정하므로 항상 유지
history = [
    {"role": "system", "content": "당신은 사용자의 이름을 기억하는 비서입니다."}
]

def chat_with_memory(user_input):
    """
    사용자의 입력을 받아 이전 대화 기록과 함께 모델에 전달하고, 
    응답을 다시 기록에 저장하는 함수
    """
    # 1. 사용자 질문 기록 (질문을 던지기 전에 리스트에 추가)
    history.append({"role": "user", "content": user_input})
    
    # 2. 전체 대화 기록 전송 (기록이 누적될수록 토큰 소모량이 늘어남에 주의)
    response = client.chat.completions.create(
        model=model,
        messages=history
    )
    
    # 3. 모델의 답변 추출 및 기록 (assistant 역할로 저장해야 다음 대화에서 참조 가능)
    answer = response.choices[0].message.content
    history.append({"role": "assistant", "content": answer})
    
    return answer

print("대화종료를 원하시면 '그만'이라고 입력하세요.")

while True:
    user_content = input("나: ")
    
    # 종료 및 초기화 로직
    if user_content == '그만':
        print("대화를 종료하고 히스토리를 초기화합니다.")
        # history 리스트를 초기 상태로 재할당 (메모리 상의 데이터 초기화)
        history = [
            {"role": "system", "content": "당신은 사용자의 이름을 기억하는 비서입니다."}
        ]
        break
    
    # 함수 호출을 통한 대화 실행
    bot_answer = chat_with_memory(user_content)
    print(f"AI 비서: {bot_answer}")
```

---

### 4. 실수/에러와 해결 과정
* **문제**: 처음에는 `requests` 라이브러리와 `OpenAI SDK` 방식을 혼용하여 코드가 복잡해지고 `payload` 변수 미정의 에러가 발생함.
* **해결**: SDK 방식이 가독성과 유지보수 면에서 유리하므로 `client.chat.completions.create`로 통일함.
* **문제**: 대화가 끝난 뒤에도 이전 데이터가 남아있어 보안상 우려됨.
* **해결**: '그만' 입력 시 `history` 변수를 초기 시스템 메시지로 재할당하여 이전 대화 맥락을 완전히 삭제함.

---

### 5. 실무관점의 참견 적용해보기
* **비용 절감 전략**: 현재는 모든 대화 기록을 계속 보냄. 실무에서는 토큰 제한(`max_tokens`)을 걸거나, 대화가 너무 길어지면 `history = history[-5:]` 처럼 최근 대화만 슬라이싱하여 보내는 방식(Sliding Window)을 고려해야 함.
* **에러 디버깅**: API 키가 환경 변수에서 제대로 로드되지 않을 경우를 대비해 `if not OPENAI_API_KEY: raise ValueError(...)` 구문을 추가하면 장애 대응이 빨라짐.
```

