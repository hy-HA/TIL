## DTO는 request와 response 두개로 관리
- 목적
    - 프론트에서 요구사항은 계속 바뀜
    - 도메인에서 필요한 부분만 받아오기위해
- 예시)
    - ID : request는 ID가 없지만, response는 ID가 있음
    - validation : 생성, 수정의 validation이 다를 수 있음
- 장점	
    - 변경사항을 최소화 
        - 같은 객체를 쓰면 다 같이 변경이 됨
        - 각각의 요구사항을 명확하게 표현 가능
    - 의존성을 끊어주는 것이 중요. 
        - 변경사항이 생겼을 때 같은 객체를 쓰면 다같이 변경이 됨
        - 하나로 만들면 의존성을 끊기 어려워서 유지보수 힘들어짐
    - DDD 개념이 그래서 나오게 된 것. 
- 단점
    - dto 객체가 많아지면, 레이어가 많아지면, 
    - 변환로직 컨버터 비용이 늘어난다. 
        - Request > Entity > Response 등 컨트롤러에서 서비스로 넘어갈때
        - A객체에서 B객체로 변환하는 로직(mapstruct)

## 참고 : DTO & VO & Entity
- 레이어끼리 통신할 때 쓰는 용어
- Request도 v1, v2…로 나누어질 수 있음
1. DTO ( Data Transfer Object )
    - 데이터를 전달하기 위한 객체
    - 주로 View와 컨트롤러 사이에 데이터를 주고 받을 때 활용
2. VO (Value Object )
    - 값 자체를 표현하는 객체
    - 하나의 필드를 특정한 값 객체로 표현
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
 
3. Entity
- 실제 DB테이블과 매핑되는 핵심 클래스
- 이를 기준으로 테이블이 생성되고 스키마가 변경됨
- 따라서 Entity를 요청이나 응답값을 전달하는 클래스로 사용하면 안됨
