## request폼이 상황에 따라 달라야 하는 경우
1. payload가 다른 경우
    - 페이로드 : body에 담기는 데이터
        > 페이로드(payload)라는 단어는 운송업에서 비롯하였는데, 지급(pay)해야 하는 적화물(load)을 의미합니다. 예를 들어, 유조선 트럭이 20톤의 기름을 운반한다면 트럭의 총 무게는 차체, 운전자 등의 무게 때문에 그것보다 더 될 것이다. 이 모든 무게를 운송하는데 비용이 들지만, 고객은 오직 기름의 무게만을 지급(pay)하게 된다. 그래서 ‘pay-load’란 말이 나온 것이다.

2. grade를 등록할 때와 수정할 때 보여야 하는 값이 다름
    ```
    1. grade 등록

        private String studentName;
        private String subjectName;
        private Integer score;


    2. grade 수정

        private Integer score;
    ```
    - 해결방안
        1. 클래스를 별도로 관리
            - 단점 : 관리할 클래스가 늘어남
            - 주의 : 새로운 클래스가 생기면 기존 코드가 바뀔 수 있음
                - 기존 : GradeForm
                - 신규 : UpdateGradeForm
                    - GradeForm > CreateGradeForm 으로 바뀌어야
            - 장점 : 클래스가 분리되어 비즈니스 관계를 논리적으로 끊을 수 있음. 
                - 다른 클래스가 변경이 된다고 영향을 받지 않는다. 
        2. innerClass
            - 주의 : static으로 해야(자바 문법)
            - 껍데기 클래스를 만들고 그 안에 각각의 클래스를 만들어 관리