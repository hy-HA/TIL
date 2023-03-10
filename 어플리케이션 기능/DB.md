# postgreSQL 연동
[참고](https://rypro.tistory.com/221)
1. PostgreSQL 설치
2. 서비스 시작
    - brew services start postsql
3. postgresql 콘솔로 접속
    - \du
4. 사용자 생성
5. 사용자 권한 부여
6. 사용자에게 데이터베이스 권한 부여
7. 데이터베이스 연결
    - \connenct "[데이터베이스명]" as user "[사용자명]"
8. 데이터베이스 확인
    - \list
9. 생성한 사용자로 접속
    - psql [데이터베이스명] -U postgres
10. DB TOOL 설치
11. 종료
    - \q
- application.properties파일
    - spring.profiles.active=local
- application-dev.properties파일
    ```
    spring.jpa.hibernate.ddl-auto=update
    spring.datasource.url=jdbc:postgresql://localhost:5432/testdb
    spring.datasource.username=testuser
    spring.datasource.password=testpass
    ```
- edit-configuration에서 active profiles에서 dev설정
    - db가 postgreSQL로 연결됨

