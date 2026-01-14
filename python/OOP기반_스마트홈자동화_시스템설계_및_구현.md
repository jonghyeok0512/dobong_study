## 목표
스마트 홈 환경을 **객체지향(OOP)** 으로 설계하고,  
허브가 다양한 기기(Device)를 제어하되 **통신 방식(Protocol)이 호환될 때만 등록**되도록 구현한다.

핵심은
- **합성(Composition)**: 허브가 여러 기기 객체를 리스트로 보유
- **추상화/인터페이스(ABC)**: Device, Protocol의 공통 규격 강제
- **다형성(Polymorphism)**: 허브는 기기 종류를 몰라도 `turn_on()/turn_off()`로 제어
## 클래스 구조

#### [오늘 배운 내용 정리]

#### 1. 오늘 학습 요약
- 객체지향 프로그래밍(OOP)의 핵심인 **추상 클래스(ABC)**와 **다형성**을 활용한 스마트 홈 시스템 구축
- 기기(Device)와 연결 방식(Protocol)을 분리하여 설계하는 **합성(Composition)** 구조 학습
- 허브(Smarthub)를 통한 기기 등록 시 **호환성 검사(타입 비교)** 로직 구현 및 시나리오 테스트

#### 2. 배운 개념 정리
- **추상 클래스(Abstract Class)**: `Device`, `Protocol`처럼 실제 물건을 만들기 전 '규격'을 정하는 설계도. `ABC`를 상속받아 만듦.
- **추상 메서드(@abstractmethod)**: 자식 클래스에서 반드시 구현해야 할 '기능의 이름'을 정의함. 하나라도 구현 안 하면 객체를 만들 수 없음.
- **다형성(Polymorphism)**: `activate_all`에서 보듯, 기기가 전등이든 에어컨이든 상관없이 동일한 이름의 메서드(`turn_on`)로 제어하는 능력.
- **타입 비교(`type()`)**: `type(A) == type(B)`를 통해 두 객체의 클래스 원형이 같은지 확인하여 시스템 호환성을 검증함.
- **f-string**: `f'{변수}'` 형태를 사용해 문자열 내부에 객체의 속성값(`self.brand` 등)을 효율적으로 삽입함.

#### 3. 오늘 작성한 코드
```python
from abc import ABC, abstractmethod

# [1. 통신 방식 설계] 모든 통신 프로토콜의 표준 규격
class Protocol(ABC):
    @abstractmethod
    def start_connection(self):
        """통신 방식에 따라 연결 메시지를 출력하도록 자식에게 강제함"""
        pass

# [2. 통신 방식 구현] 각 연결 방식별 구체적인 동작
class WiFiProtocol(Protocol):
    def start_connection(self):
        print('WiFi로 연결을 시도합니다.')

class ZigbeeProtocol(Protocol):
    def start_connection(self):
        print('Zigbee망을 구성합니다.')

# [3. 기기 설계] 스마트 홈에 등록될 모든 기기의 공통 부모
class Device(ABC):
    def __init__(self, name, brand, protocol):
        self.name = name          # 기기 이름 (예: 거실 전등)
        self.brand = brand        # 브랜드 (예: 필립스)
        self.protocol = protocol  # 이 기기가 사용하는 통신 객체 (합성 활용)

    @abstractmethod
    def turn_on(self):
        pass # 자식 클래스에서 실제 작동 로그를 구현해야 함

    @abstractmethod
    def turn_off(self):
        pass

    @abstractmethod
    def get_status(self):
        pass

# [4. 기기 구현] 전등 및 에어컨
class SmartLight(Device):
    def turn_on(self):
        # 띄어쓰기를 추가하여 가독성을 높인 출력
        print(f'{self.brand} {self.name} 전등을 켭니다.')

    def turn_off(self):
        print(f'{self.brand} {self.name} 전등을 끕니다.')

    def get_status(self):
        print('현재 전등의 밝기는 100%입니다.')

class AirConditioner(Device):
    def turn_on(self):
        print(f'{self.brand} {self.name} 에어컨을 가동합니다.')

    def turn_off(self):
        print(f'{self.brand} {self.name} 에어컨을 정지합니다.')

    def get_status(self):
        print('현재 설정 온도는 24도 입니다.')

# [5. 스마트 허브] 기기 관리 및 전체 제어 로직의 핵심
class Smarthub:
    def __init__(self, name, protocol):
        self.name = name
        self.protocol = protocol  # 허브가 지원하는 프로토콜
        self.devices = []         # 호환성 검사를 통과한 기기 저장소

    def register_device(self, device):
        # [의도] 허브와 기기의 통신 방식 클래스 타입이 같은지 비교
        if type(device.protocol) == type(self.protocol):
            self.devices.append(device)
            print(f'[{device.name}] 등록 성공')
        else:
            # 일치하지 않으면 등록을 거부하여 시스템 안정성 확보
            print(f'[{device.name}] 호환되지 않는 연결 방식입니다.')

    def activate_all(self):
        # [의도] 전체 기기를 순회하며 '통신 연결 -> 작동' 시퀀스 수행
        print(f'\n--- {self.name} 전체 가동 ---')
        for device in self.devices:
            # 기기 내부의 프로토콜 객체 메서드를 먼저 호출
            device.protocol.start_connection()
            # 이후 기기 가동 메서드 호출 (다형성)
            device.turn_on()

    def deactivate_all(self):
        for device in self.devices:
            device.turn_off()
```

#### 4. 실수/에러와 해결 과정 (헷갈렸던 점)
- **오타 사건**: `self.procotol`처럼 스펠링을 틀리면 파이썬은 완전히 다른 변수로 인식함. `protocol`로 정확히 맞추는 연습을 함.
- **추상 메서드 호출**: 부모의 추상 메서드가 `pass`인 경우 `super()`를 호출해도 아무 동작이 없으므로, 자식에서 직접 `print` 로직을 짜야 함을 깨달음.
- **메서드 vs 속성**: `device.protocol()`처럼 괄호를 붙여서 에러가 났던 경험을 통해, '객체 데이터'와 '실행 함수'의 차이를 명확히 구분함.
- **타입 비교의 필요성**: 객체 자체(`==`)가 아니라 클래스의 종류(`type()`)를 비교해야 "동일한 통신 방식"인지를 정확히 걸러낼 수 있음을 배움.

#### 5. 실무/RAG 관점의 참견 적용
- **확장성**: 오늘 짠 구조는 새로운 통신 방식(Bluetooth 등)이나 새로운 기기(로봇청소기 등)가 추가되어도 기존 `Smarthub` 코드를 수정할 필요가 없는 '개방-폐쇄 원칙(OCP)'을 잘 지킨 구조임.
- **RAG 연결**: 나중에 각 기기의 `get_status` 정보를 모아서 LLM에게 전달하면, "지금 우리 집 전등 켜져 있어?" 같은 질문에 답하는 AI 비서를 만들 수 있음.
```