# 인텔리J 단축키

- auto Import
    - settings > Editor > General > Auto Import > Add unambiguous imports on the fly

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

- 파라미터 정보 확인
    - cmd + p
    
- 코드 생성
    - 새로 만들기 
        - cmd+N
    - 클래스 생성
        - ctrl + 스페이스
        - ex) Foo foo = new Foo()
- 소스 코드와 테스트 코드 이동(또는 생성)
    - 클래스 이동
        - cmd + 마우스로 클릭
    - 테스트코드 생성
        - cmd + shift + t
- 퀵 픽스
    - 직접 생성하는것보다 있다치고 사용한뒤 퀵픽스로
- 클래스 찾기 
    - shift 두번클릭 > 2번째 탭
- 리팩토링
    - 변수,메소드 빼내기
        - 블럭처리 후 변수(variable)로 빼기
        - option + cmd + v
        - 중복으로 사용하는 코드들을 변수로 사용하기 편함
    - 리네임 리팩토링(클래스 이름 변경)
        - shift + f6
- 테스트코드 실행
    - 커서 올리고 ctrl+shift+R
        - 현재 문맥에 있는 것을 실행
    - 만약 커서가 다른 곳에 있는데 이전에 실행했던 테스트코드 다시 실행하려면
        - ctrl+R

# 리팩토링
- 변수 추출하기
    - cmd + opt + v