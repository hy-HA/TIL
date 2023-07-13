### Subject도메인
- SubjectType enum클래스로 선언
- SubjectType 필드는 Unique entity로 설정
```java
public class Subject{

    @Enumerated(value = EnumType.STRING)
    @Column(name = "subject_type", unique = true)
    private SubjectType subjectType;
}
```