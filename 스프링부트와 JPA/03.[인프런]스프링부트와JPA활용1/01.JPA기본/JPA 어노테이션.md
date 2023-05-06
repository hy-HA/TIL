# JPA 어노테이션
참고 ] https://cjw-awdsd.tistory.com/46

## @Entity
- 해당 클래스를 DB 테이블과 매핑
- JPA가 관리할 객체임을 확인

## @Table
- 엔티티(클래스)와 매핑할 테이블을 지정
- 속성
    - name : 매핑할 테이블 이름 
        - 예. @Table(name = "orders")
        - 생략 시 엔티티(클래스) 이름 사용
    - schema
    - catalog

## @Column
- 객체 필드를 테이블 컬럼과 매핑
- 필드 속성을 지정할 때 사용
- 속성
    - name : 필드와 매핑할 테이블의 컬럼명 지정
        - 예. @Column(name = "member_id")
        - default는 필드이름으로 대체
    - length : 문자 길이 제약 조건 설정
    - nullable : DDL 생성 시 설정 값에 따라 null 가능여부를 설정
    - unique : DDL 생성 시 간단하게 한 컬럼에 유니크 제약 조건을 설정
    ```
    @Column
    private String name;
    ```

## @PersistenceContext   
1. EntityManager를 빈으로 주입할 때 사용하는 어노테이션
    - 스프링에서는 영속성 관리를 위해 EntityManager가 존재
    - 그래서 스프링 컨테이너가 시작될 때 EntityManager를 만들어서 빈으로 등록하는데,
    - 이 때 스프링이 만들어둔 EntityManager를 주입받을 때 사용
2. @PersistenceContext로 지정된 프로퍼티에 아래 두 가지 중 한 가지로 EntityManager를 주입
    - EntityManagerFactory에서 새로운 EntityManager를 생성하거나
    - Transaction에 의해 기존에 생성된 EntityManager를 반환
3. @PersistenceContext를 사용해야 하는 이유
    - EntityManager를 사용할 때 주의해야 할 점은 여러 쓰레드가 동시에 접근하면 동시성 문제가 발생하여 쓰레드 간에는 EntityManager를 공유해서는 안됨
        - 일반적으로 스프링은 싱글톤 기반으로 동작하기에 빈은 모든 쓰레드가 공유함
        - 그러나 @PersistenceContext으로 EntityManager를 주입받아도 동시성 문제가 발생하지 않음
    - 동시성 문제가 발생하지 않는 이유는
        - 스프링 컨테이너가 초기화되면서 @PersistenceContext으로 주입받은 EntityManager를 Proxy로 감쌈
        - 그리고 EntityManager 호출 시 마다 Proxy를 통해 EntityManager를 생성하여 Thread-Safe를 보장
3. 참고] Autowired 와 PersistenceContext
    - @Autowired는 스프링 빈을 주입
    - @PersistenceContext는 JPA 스펙에서 제공하는 기능인데, 영속성 컨텍스트를 주입하는 표준 애노테이션
참고] https://batory.tistory.com/497


## @Id | @GeneratedValue
- @Id
    - 특정 속성을 기본키로 설정
        ```
        @Id
        private Long id;
        ```
    - @Id 어노테이션만 적게될 경우 기본키값을 직접 부여해줘야
    - 하지만 보통 DB를 설계할 때는 기본키는 직접 부여하지 않고 Mysql AUTO_INCREMENT처럼 자동 부여되게끔 한다.
        ```
        @Id
        @GeneratedValue
        private Long id;
        ```

- @GeneratedValue
    - @Id와 함께 사용되는 경우 기본키를 DB가 생성해주는 값으로 사용하게 됨
    - 기본값을 DB에서 자동으로 생성
        - 전략
            1. @GeneratedValue(startegy = GenerationType.IDENTITY)
                - 기본 키 생성을 DB에 위임 (Mysql)
            2. @GeneratedValue(startegy = GenerationType.SEQUENCE)	
                - DB시퀀스를 사용해서 기본 키 할당 (ORACLE)
            3. @GeneratedValue(startegy = GenerationType.TABLE)
                - 키 생성 테이블 사용 (모든 DB 사용 가능)
            4. @GeneratedValue(startegy = GenerationType.AUTO)
                - 선택된 DB에 따라 JPA가 자동으로 전략 선택


## @Embedded | @Embeddable
- 임베디드 타입은 복합 값 타입으로 불리며 새로운 값 타입을 직접 정의해서 사용하는 JPA의 방법
- 더 객체지향적이고 응집도 있는 코드로 개발가능
- @Embeddable : 값 타입을 정의하는 곳에 표시
    - 임베디드 타입을 적용하려면 새로운 Class를 만들고 해당 클래스에 임베디드 타입으로 묶으려던 Attribute들을 넣어준 뒤 @Embeddable를 붙여줌
- @Embedded : 값 타입을 사용하는 곳에 표시
참고 ] https://velog.io/@seongwon97/Spring-Boot-JPA-Embedded-Embeddable

## @JsonIgnore
- 클래스의 속성(필드, 멤버변수) 수준에서 사용
- 직렬화 역직렬화에 사용되는 논리적 프로퍼티(속성..) 값을 무시할때 사용

## @OneToMany | @ManyToOne
[참고](https://boomrabbit.tistory.com/217)
### @OneToMany
- 일대다, 회원 한 명이 게시글을 여러 개 작성할 수 있으므로 회원(Member) 클래스에서 @OneToMany를 선언
    ```
    - @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    - @OneToMany(mappedBy = "member")
    ```
### @ManyToOne
- 다대일, 한 명의 회원이 여러 게시글을 작성할 수 있으므로 게시글(Article) 클래스에서 @ManyToOne을 선언
- @ManyToOne(fetch = LAZY)
### @JoinColumn(name = "member_id")
- 해당 클래스 엔티티에 존재하는 필드명(컬럼)의 이름을 name을 통해 설정
## 생성자 관련 어노테이션
1. @NoArgsConstructor(access = AccessLevel.PROTECTED)
    - 파라미터가 없는 기본 생성자를 생성
    - 초기값 세팅이 필요한 final 변수가 있을 경우 컴파일 에러가 발생함으로 주의
    - 속성
        - @NoArgsConstructor(force=true) 
            - null, 0 등 기본 값으로 초기화 된다.
        - @NoArgsConstructor(AccessLevel.PROTECTED)
            - 기본 생성자의 접근 제어를 PROTECTED로 설정해놓게 되면 무분별한 객체 생성에 대해 한번 더 체크할 수 있음
2. @RequiredArgsConstructor
    - final 변수, Notnull 표시가 된 변수처럼 필수적인 정보를 세팅하는 생성자를 생성
3. @AllArgsConstructor
    - 전체 변수를 생성하는 생성자를 생성
# 롬복
@Getter @Setter