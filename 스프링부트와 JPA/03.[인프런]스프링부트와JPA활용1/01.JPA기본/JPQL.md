# JPQL
- 가장 단순한 조회방법
- 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리
- SQL을 추상화해서 특정 DB SQL에 의존하지 않음
- 객체지향SQL
    - JPA를 사용하면 엔티티 객체를 중심으로 개발
    - 문제는 검색 쿼리
    - 모든 DB데이터를 객체로 변환해서 검색하는 것은 불가능
    - 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요

# JPQL 과 SQL
- JPQL : 엔티티 객체를 대상으로 쿼리
- SQL : 데이터베이스 테이블을 대상으로 쿼리

```
          //m : jpql은 테이블(member)이 아니라 객체가 대상
List<Member> result = em.createQuery("select m from Member as m", Member.class)
    .getResultList();
for (Member member : result) {
    sout(member.getName());
}
```