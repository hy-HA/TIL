# create되는 번거로움을 피하기 위해 h2 에서 mysql로 변경

### mysql 의존성 추가
- 버전을 지우니 오류가 사라짐    
```    
implementation 'com.mysql:mysql-connector-j'
```

### application.properties 설정
- 주석처리한 부분을 지우니 오류가 사라짐
```
#spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/test6
spring.datasource.username=root
spring.datasource.password=root

#spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

spring.jpa.hibernate.ddl-auto=create
```
