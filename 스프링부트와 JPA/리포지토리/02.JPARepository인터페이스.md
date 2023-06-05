#### 참고 JPQL, Optional

# 공통 인터페이스 기능

# 순수 JPA 기반 리포지토리 만들기
- 순수한 JPA 기반 리포지토리를 만들자 
    - 기본 CRUD
        - 저장
        - 변경 변경감지 사용 삭제
        - 전체 조회
        - 단건 조회
        - 카운트
- 참고: **변경감지(dirty checking)**
    - JPA에서 수정은 변경감지 기능을 사용하면 된다.
    - 트랜잭션 안에서 엔티티를 조회한 다음에 데이터를 변경하면, 트랜잭션 종료 시점에 변경감지 기능이 작동해서 변경된 엔티티를 감지하고 UPDATE SQL을 실행한다.
    - 따라서 update메소드가 없어도 자동으로 처리된다
        - (자바 컬렉션과 동일하게 사용가능해짐)

# 순수 JPA 기반 리포지토리의 crud 코드

### **저장**

**public Member save(Member member)**

    em.persist(member);
    return member;
**public Team save(Team team)** 

    em.persist(team);
    return team;
### **삭제**
**public void delete(Member member)** 

    em.remove(member);
**public void delete(Team team)** 

    em.remove(team);
### **전체조회**
- 전체를 조회하거나 특정 조건으로 필터링해서 조회할 때는 JPA가 제공하는 jpql 기술을 사용하여 조회
- jpql
    - 테이블 대상이 아니라 객체를 대상으로 하는 쿼리
    - 모양은 sql과 동일하게 생김

**public List<Member> findAll()** 

    return em.createQuery("select m from Member m", Member.class)
        .getResultList();
**public List<Team> findAll()** 

    return em.createQuery("select t from Team t”, Team.class)
        .getResultList();
### **단건 조회**
**public Member find(Long id)** 

    return em.find(Member.class, id);

### **Optional로 조회**
**public Optional<Member> findById(Long id)** 

    Member member = em.find(Member.class, id);
    return Optional.ofNullable(member);

**public Optional<Team> findById(Long id)**  

    Team team = em.find(Team.class, id);
    return Optional.ofNullable(team);

### **카운트**
**public long count()** 

    return em.createQuery("select count(m) from Member m",  Long.class)
        .getSingleResult();

**public long count()** 

    return em.createQuery("select count(t) from Team t”, Long.class)
            .getSingleResult();

## 공통 인터페이스 설정
-  JavaConfig 설정
    - **스프링 부트 사용시 생략 가능**
    - 스프링부트 사용 시 JPA는 @SpringBootApplication이 클래스의 패키지부터 하위 패키지까지 자동으로 인식함
    - 만약 위치가 달라지면 @EnableJpaRepositories 필요
```
@Configuration
@EnableJpaRepositories(basePackages = "jpabook.jpashop.repository")
public class AppConfig {}
```
# 리포지토리 구현클래스의 생성 원리
- 스프링 데이터 JPA가 리포지토리 인터페이스의 구현클래스를 대신 생성
![](https://i.imgur.com/VjI44aP.png)

**JpaRepository 상속하기**

```
public interface MemberRepository extends JpaRepository<Member, Long> {}
```
- org.springframework.data.repository.Repository 를 구현한 클래스는 스캔 대상   
    - 실제 출력해보기(Proxy)
        - memberRepository.getClass() => class com.sun.proxy.$ProxyXXX

**@Repository 애노테이션 생략 가능**

- 컴포넌트 스캔을 스프링 데이터 JPA가 자동으로 처리 
- JPA 예외를 스프링 예외로 변환하는 과정도 자동으로 처리

**JpaRepository 구성**

- 공통 CRUD 제공
- 제네릭 타입
    - 제네릭은 <엔티티 타입, 식별자 타입> 설정
    1. T : 엔티티
    2. ID : 엔티티의 식별자 타입 
    3. S : 엔티티와 그 자식 타입
- 주요 메서드
    1. save(S) 
        - 새로운 엔티티는 저장하고 이미 있는 엔티티는 병합한다.
    2. delete(T) 
        - 엔티티 하나를 삭제한다. 
        - 내부에서 EntityManager.remove() 호출
    3. findById(ID) 
        - 엔티티 하나를 조회한다. 
        - 내부에서 EntityManager.find() 호출
    4. getOne(ID) 
        - 엔티티를 프록시로 조회한다. 
        - 내부에서 EntityManager.getReference() 호출 
    5. findAll(...) 
        - 모든 엔티티를 조회한다. 
        - 정렬( Sort )이나 페이징( Pageable ) 조건을 파라미터로 제공할 수 있다.
- 주의 : 버전변경에 따라 아래처럼 변경됨
    - T findOne(ID) => Optional<T> findById(ID) 변경
    - boolean exists(ID) => boolean existsById(ID) 변경

**스프링 데이터 공통 인터페이스 구성**

- Crud리포지토리, PagingAndSorting리포지토리
    - 몽고db 등도 같이 쓰는 기능들
- JpaRepository
    - JPA에 특화된 기능들만 제공

![](https://i.imgur.com/TQgVbWF.png)