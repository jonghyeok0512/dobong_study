# 파이썬 예외 처리 및 형식 힌트 실전 복습

1. 오늘 학습 요약
- 프로그램 실행 중 발생하는 논리적 오류인 예외를 관리하여 비정상 종료를 방지함
- 코드의 데이터 타입을 명시하는 형식 힌트와 설명서 역할을 하는 독스트링 사용법을 익힘
- 에러가 발생했을 때 프로그램이 멈추지 않고 적절한 안내를 하도록 흐름을 제어함

2. 배운 개념 정리
- 예외 처리(Exception Handling): try 문 내부에 예외 발생 가능 코드를 두고 except 문에서 처리함
- else/finally: 예외가 없을 때 실행되는 else 문과 성공 여부와 무관하게 항상 실행되는 finally 문을 활용함
- 형식 힌트(Type Hinting): 콜론(:)과 화살표(->)를 사용하여 변수와 함수의 데이터 형식을 선언함
- 독스트링(Docstring): 큰따옴표 3개를 사용하여 코드의 의도와 매개변수 정보를 기록하며 __doc__으로 접근 가능함

3. 오늘 작성한 코드리뷰
```python
# [실습] 예외 처리와 형식 힌트가 결합된 함수 설계
def process_data(input_val: str | int) -> None:
    """
    입력값을 정수로 변환하여 계산 후 결과를 출력함
    Args:
        input_val (str | int): 연산에 사용할 입력 데이터
    """
    try:
        # 데이터 형변환 시도 (ValueError 발생 가능 지점)
        number: int = int(input_val)
        result: float = 100 / number # 0일 경우 ZeroDivisionError 발생
    except ValueError as e:
        # 유효하지 않은 숫자 형식 입력 시 처리
        print(f"오류 발생: {e}")
    except ZeroDivisionError:
        # 0으로 나눌 때 예외 처리
        print("0으로 나눌 수 없습니다.")
    else:
        # 예외 없이 성공적으로 연산된 경우 실행
        print(f"연산 결과: {result}")
    finally:
        # 성공/실패 여부와 관계없이 항상 마지막에 실행
        print("데이터 처리를 완료했습니다.")
```

4. 실수/에러와 해결 과정
- 문제: AttributeError: 'NoneType' object has no attribute 'damage' 발생
- 원인: 캐릭터 객체의 weapon 속성이 None인 상태에서 damage 속성을 호출함
- 해결 과정: 
  - 에러 메시지의 마지막 줄을 통해 호출 주체가 None임을 확인
  - 공격(attack) 전 무기 장착(pick_up_weapon) 함수가 선행되도록 코드 순서 조정
  - if self.weapon: 조건문을 추가하여 객체가 존재할 때만 속성에 접근하도록 방어 코드 작성

5. 실무/RAG 관점
- 실무 관점: 실무 코드는 단순히 기능을 구현하는 것보다 예외가 발생했을 때 시스템이 죽지 않게 만드는 것이 더 중요함. raise를 활용해 서비스의 정책(예: 레벨 제한)에 맞는 사용자 정의 예외를 만드는 습관이 필요함
- RAG 관점: AI 에이전트가 외부 데이터를 조회할 때 데이터가 비어있을 확률이 매우 높음. 이때 형식 힌트로 데이터의 형태를 명시하고 try-except로 데이터 누락 상황을 처리해야 AI 서비스의 신뢰도를 높일 수 있음
```