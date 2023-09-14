# 인텔리J 단축키
- 단축키 설정
    - 설정 > Editor > Live Templates > custom > tdd 설정
- auto Import
    - settings > Editor > General > Auto Import > Add unambiguous imports on the fly
# 간편하게 작성
- 예외처리
    - try catch할 구문 드래그 > code > surround wiith(Ctrl + Alt + T) > try/catch

- 패키지나 폴더가 없는데 사용됨
    - more actions > move to package 'com.fastcampus.ch4.controller'(없는데 사용된 폴더) > 클릭 시 자동으로 폴더 생성됨
    
- 클래스가 없는데 사용됨
    - create class 'UserDao'(없는데 사용된 클래스) > 클릭 후 클래스가 위치할 패키지 설정 > 클래스 생성됨

- 메소드로 뽑아내기
    - ctrl + t > extract method
    - 혹은 cmd + option + m

- 데이터베이스에서 셀렉트문 간편하게 작성하기
    - 테이블명 우클릭 > sql scripts > select all rows from a table
    - * 앞을 클릭 > 전구모양 클릭 > expand column list
    
- 코드 생성
    - 새로 만들기 
        - cmd+N
    - 클래스 생성
        - ctrl + 스페이스
        - ex) Foo foo = new Foo()

- 퀵 픽스
    - 직접 생성하는것보다 있다치고 사용한뒤 퀵픽스로

# 추적
- 인터페이스에서 구현체 찾기
    - cmd+ opt + b
- 코드 추적
    - cmd + b
- 클래스 찾기 
    - shift 두번클릭 > 2번째 탭
- 히스토리 보기
    - cmd + e
- 파라미터 정보 확인
    - cmd + p
- 소스 코드와 테스트 코드 이동(또는 생성)
    - 클래스 이동
        - cmd + 마우스로 클릭
    - 테스트코드 생성
        - cmd + shift + t
- url로 찾기
    - cmd + \
- 파일 찾기
    - cmd + shift + o 

# 리팩토링
- 변수 추출하기
    - cmd + opt + v
- 리팩토링
    - 변수,메소드 빼내기
        - 블럭처리 후 변수(variable)로 빼기
        - option + cmd + v
        - 중복으로 사용하는 코드들을 변수로 사용하기 편함
    - 리네임 리팩토링(클래스 이름 변경)
        - shift + f6

- 프로젝트 내 단어 전체 검색
    - ctrl + shift + f  
- 프로젝트 내 단어 바꾸기
    - ctrl + shift + r 

# 테스트 단축키
- 테스트코드 실행
    - 커서 올리고 ctrl+shift+R
        - 현재 문맥에 있는 것을 실행
    - 만약 커서가 다른 곳에 있는데 이전에 실행했던 테스트코드 다시 실행하려면
        - ctrl+R
- 테스트 코드 생성
    - cmd+shift+t

# 어떤 파라미터를 써야하는지 확인
- cmd + p

# 서버 재시작
- ctrl + d