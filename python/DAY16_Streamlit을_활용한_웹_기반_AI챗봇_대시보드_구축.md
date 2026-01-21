#### [학습 기록] Streamlit을 활용한 웹 기반 AI 챗봇 대시보드 구축

### 1. 오늘 학습 요약
* Streamlit의 `st.session_state`를 이용한 상태 유지(State Management) 기법 습득
* `st.chat_message`와 `st.chat_input`을 활용한 채팅 UI 구현
* OpenAI SDK의 스트리밍 응답을 웹 화면에 실시간으로 렌더링하는 방법 학습

### 2. 배운 개념 정리
* **st.session_state**: 페이지가 새로고침되어도 변수 값을 유지시키는 저장소. 대화형 인터페이스 구축 시 필수적임.
* **st.write_stream**: OpenAI의 `stream=True` 응답(Generator)을 받아 화면에 타이핑 효과와 함께 출력해주는 편리한 함수.
* **st.rerun()**: 특정 조건(초기화 등)에서 페이지 로직을 즉시 재실행하여 변경 사항을 반영함.

### 3. 오늘 작성한 코드리뷰
```python
import streamlit as st
from openai import OpenAI
import os
from dotenv import load_dotenv

load_dotenv()

# 1. 초기 설정
st.set_page_config(page_title="AI 비서 챗봇", page_icon="🤖")
st.title("🤖 나만의 AI 비서")

OPENAI_API_KEY = os.getenv('OPENAI_API_KEY')
client = OpenAI(api_key=OPENAI_API_KEY)
model = "gpt-4o-mini"

# 2. 세션 상태(Session State)를 이용한 대화 기록 관리
# Streamlit은 버튼을 누를 때마다 코드가 처음부터 다시 실행되므로, 
# 대화 내용을 유지하려면 st.session_state를 사용해야 함.
if "messages" not in st.session_state:
    st.session_state.messages = [
        {"role": "system", "content": "당신은 사용자의 이름을 기억하는 친절한 비서입니다."}
    ]

# 3. 사이드바에 초기화 버튼 배치
if st.sidebar.button("대화 내용 초기화"):
    st.session_state.messages = [
        {"role": "system", "content": "당신은 사용자의 이름을 기억하는 친절한 비서입니다."}
    ]
    st.rerun() # 화면을 다시 그려서 초기화 반영

# 4. 이전 대화 기록 화면에 출력 (시스템 메시지 제외)
for message in st.session_state.messages:
    if message["role"] != "system":
        with st.chat_message(message["role"]):
            st.markdown(message["content"])

# 5. 사용자 입력 처리
if prompt := st.chat_input("메시지를 입력하세요..."):
    # (1) 사용자 메시지 표시 및 저장
    st.chat_message("user").markdown(prompt)
    st.session_state.messages.append({"role": "user", "content": prompt})

    # (2) Assistant 응답 생성 및 표시
    with st.chat_message("assistant"):
        # 실무적인 UX를 위해 스트리밍 효과 적용
        stream = client.chat.completions.create(
            model=model,
            messages=st.session_state.messages,
            stream=True
        )
        response = st.write_stream(stream) # 스트리밍 출력을 도와주는 Streamlit 함수
    
    # (3) 응답 저장
    st.session_state.messages.append({"role": "assistant", "content": response})
```
* 주석: 각 라인은 세션 상태 확인 -> 이전 메시지 렌더링 -> 유저 입력 처리 -> AI 응답 생성 및 저장의 흐름을 따름.

### 4. 실수/에러와 해결 과정
* **문제**: Streamlit 페이지를 조작할 때마다 `history` 리스트가 빈 값으로 초기화되어 대화 흐름이 끊김.
* **원인**: Streamlit은 상호작용 발생 시 스크립트 전체가 재실행되는 특성이 있음.
* **해결**: 단순 리스트 변수 대신 `st.session_state`에 메시지를 저장하여 재실행 후에도 데이터가 보존되도록 수정함.

### 5. 실무관점의 참견 적용해보기
* **UX 개선**: `st.write_stream`을 사용하여 사용자가 AI의 답변이 생성되는 과정을 실시간으로 볼 수 있게 함으로써 체감 대기 시간을 줄임.
* **자원 최적화**: 사이드바에 리셋 버튼을 배치하여, 불필요하게 길어진 컨텍스트(토큰)를 사용자가 직접 정리할 수 있는 제어권을 부여함.