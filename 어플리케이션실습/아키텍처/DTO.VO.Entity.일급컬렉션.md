# DTO & VO & Entity
- 레이어끼리 통신할 때 쓰는 용어
- Request도 v1, v2…로 나누어질 수 있음

## DTO ( Data Transfer Object )
- 데이터를 전달하기 위한 객체
- 주로 View와 컨트롤러 사이에 데이터를 주고 받을 때 활용

## VO (Value Object )
- 예시 : 음료수
- 값 자체를 표현하는 객체
- 하나의 필드를 특정한 값 객체로 
- 객체 비교
    - 객체들의 주소가 달라도 값이 같으면 동일한 것으로 여김
    - 값 비교를 위해 equals()와 hashCode()메서드를 오버라이딩해야
- 목적
    - 컨버터를 하기 위해
- 예시)
    ```
    public class Score { 

        private final int value;

        public Score(int value) { 
            if (value < 0 || value > 100) {
                throw new IllegalArgumentException("존재 할 수 없는 점수입니다."); 
            } 
            this.value = value; 
        } 

        public Integer getValue() { return value; } 

        public String toString() { return String.format("%s점", value); } }
    ```
## Entity
- 예시 : 엘리스
- 실제 DB테이블과 매핑되는 핵심 클래스
    - Entity를 기준으로 테이블이 생성되고 스키마가 변경됨
    - 따라서 Entity를 요청이나 응답값을 전달하는 클래스로 사용하면 안됨
- 고유한 식별자가 존재
- 객체 비교
    - id를 통해 객체 비교
    - 식별자를 기준으로 equals와 hashcode()를 재정의

## VO와 Entity
1. 수명
    - VO 
        - 수명이 없음
        - 쉽게 생성 및 삭제 가능 
        - 수명이 없으므로 언제든지 값 객체를 생성하여 대체 가능
        - 독립적으로 존재할 수 없다
            - 항상 Entity에 속해서 존재
    - Entity
        - 수명이 있음. 
        - 객체가 어떻게 변화했는지 어떤 상태인지에 대한 기록들을 가짐

2. 불변성
    - VO 
        - 불변이어야 함
        - 값을 변경해야할 때는 새로운 값 객체로 갈아 끼워주면 됨
        - VO를 변형시킨다면 라이프사이클을 가지게 되는 것
            - VO가 라이프사이클을 가지면, 고유의 식별자를 가져야하고, 이는 VO가 아니게 된다.
            - VO는 단지 어느 상태의 순간(값)임
        - 어떤 객체를 불변으로 만들지 못하면 그 객체는 VO로 사용될 수 없음
    - Entity 
        - 대부분 가변
        - 특정 VO를 필드로 가지는 Entity의 경우 해당 Entity의 라이프사이클을 따라서 VO들은 새로운 VO로 대체됨. 
