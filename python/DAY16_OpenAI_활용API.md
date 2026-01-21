#### [학습 기록] OpenAI API 진화: Chat Completions vs Responses API 비교 및 실습

### 1. 오늘 학습 요약
* 기존의 Stateless 방식인 **Chat Completions API**와 서버 측 상태 유지가 가능한 **Responses API**의 구조적 차이 이해.
* 각 API의 REST 방식 호출 및 Python SDK를 활용한 실습 진행.
* 시스템 프롬프트(Instructions), 스트리밍, 비동기 요청 등 실무 필수 기능 구현.

### 2. 배운 개념 정리
#### ① 상태 유지 방식 (State Management)
* **Chat Completions (Stateless)**: 모델은 이전 대화를 기억하지 못하므로, 매번 `history` 리스트에 대화 전체를 담아 보내야 함.
* **Responses API (Stateful)**: 서버가 대화 맥락을 보관함. `previous_response_id`를 전달하면 이전 대화 내용을 자동으로 참조함.

#### ② 주요 파라미터 및 구조 변화
| 구분 | Chat Completions | Responses API |
| :--- | :--- | :--- |
| **입력 필드** | `messages` (배열) | `input` (문자열, 객체, 배열 등) |
| **시스템 지침** | `role: system` 메시지 사용 | `instructions` 독립 파라미터 사용 |
| **응답 구조** | `choices[0].message.content` | `output[0].content` 또는 `output_text` |
| **형식 지정** | `response_format` | `text: {"format": "json_object"}` |

### 3. 주요 실습 코드 리뷰

#### [Chat Completions] 대화 맥락 유지 (Memory)
```python
# history 리스트를 직접 관리하며 누적된 대화 내역을 매번 전송함
history = [{"role": "system", "content": "비서입니다."}]
def chat_with_memory(user_input):
    history.append({"role": "user", "content": user_input})
    response = client.chat.completions.create(model=model, messages=history)
    answer = response.choices[0].message.content
    history.append({"role": "assistant", "content": answer})
    return answer
```

#### [Responses API] 서버 측 ID를 활용한 맥락 유지
```python
# 이전 응답의 ID(res1.id)만 넘겨주면 서버가 이전 맥락을 알아서 연결함
res1 = client.responses.create(model=model, input="내 이름은 Jun이야.")
response_id = res1.id

res2 = client.responses.create(
    model=model, 
    input="내 이름이 뭐라고?", 
    previous_response_id=response_id
)
```

### 4. 실수/에러와 해결 과정
* **현상**: Responses API 결과 추출 시 기존처럼 `.choices`를 사용하여 `AttributeError` 발생.
* **해결**: Responses API는 `output` 배열을 반환하거나, SDK에서는 `response.output_text`라는 전용 속성을 사용해야 함을 확인.
* **현상**: `streamlit run` 명령어 실행 시 파일 미존재 에러 발생.
* **해결**: 터미널의 현재 위치(Working Directory)가 파일이 있는 폴더인지 `cd` 명령어로 확인 후 실행.

### 5. 실무관점의 참견
* **비용 최적화**: 대화가 길어질수록 `Chat Completions` 방식은 입력 토큰(Input Tokens)량이 기하급수적으로 늘어남. 긴 대화 서비스라면 `Responses API`의 `previous_response_id`나 `compact`(대화 압축) 기능을 활용해 비용과 토큰 제한을 관리하는 것이 유리함.
* **구조화된 출력**: 웹 백엔드와 연동할 때는 반드시 `JSON object` 모드를 활성화하여 AI의 답변을 데이터베이스나 프론트엔드에서 즉시 파싱 가능한 상태로 받아야 함.