| [참고](https://ckddn9496.tistory.com/69)

![](https://i.imgur.com/MgNjYnT.png)
- (id=1,challengaName=영어)를 JSON형태로 바꿔줌
- jackson을 이용한 json직렬화
# 직렬화와 역직렬화
- 자바에서 직렬화,역직렬화는 객체를 파일로 저장하거나 네트워크를 통해 전송하기 위해 제공되는 기능

- 직렬화
    - 객체를 전송가능한 형태로 말아주는 것
    - 저장된 데이터를 I/O 스트림에 출력하기 위해 연속적인 데이터로 변환하는 것
    - Object > Json
- 역직렬화
    - 데이터들을 자바 객체로 변환해주는 것
    - I/O 스트림에서 데이터를 읽어서 객체를 만드는 것
    - Json > Object

# Jackson Serialization Annotations

## @JsonAnyGetter
- Map 필드에 대해서 일반 properties로 처리한다
```java
public class ExtendableBean {

	private String name;
	private Map<String, String> properties;

	@JsonAnyGetter
	public Map<String, String> getProperties() {
		return properties;
	}
}
```
```java
/** @JsonAnyGetter 적용 전 */
{
  "name" : "My bean",
  "properties" : {
    "attr2" : "val2",
    "attr1" : "val1"
  }
}

/** @JsonAnyGetter 적용 후 */
{
  "name" : "My bean",
  "attr2" : "val2",
  "attr1" : "val1"
}
```
## @JsonGetter
- 메소드의 value에서 지정한 필드의 getter를 지정
- getTheName() 메서드는 name 필드의 getter로 지정
    - 즉, serialization 과정에서 key=name의 value=getTheName()

```java
public class MyBean {

    public int id;
    private String name;
 
    @JsonGetter("name")
    public String getTheName() {
        return name;
    }
}
```

## @JsonPropertyOrder
- serialization 순서를 정의
```java
@JsonPropertyOrder({ "name", "id" })
public class MyBean {
    public int id;
    public String name;
}
```
```java
/** @JsonPropertyOrder 적용 전 */
{
  "id" : 1,
  "name" : "My bean"
}

/** @JsonPropertyOrder 적용 후 */
{
  "name" : "My bean",
  "id" : 1
}
```

## @JsonRawValue
- 메서드나 필드에 적용되며, 그 필드를 그대로 직렬화하여 JSON 값으로 여긴다.
```java
public class RawBean {

    public String name;
 
    @JsonRawValue
    public String json;
}
```
```java
@Test
public void testJsonRawValue() throws JsonProcessingException {

    RawBean rawBean = new RawBean("My bean", "{\"key\":\"value\",\"isTrue\":false}");

    System.out.println(new ObjectMapper()
        .writerWithDefaultPrettyPrinter()
        .writeValueAsString(rawBean));
}

/** @JsonRawValue 적용 전 */
{
  "name" : "My bean",
  "json" : "{\"key\":\"value\",\"isTrue\":false}"
}

/** @JsonRawValue 적용 후 */
{
  "name" : "My bean",
  "json" : {"key":"value","isTrue":false}
}
```

## @JsonValue
- 하나의 인스턴스를 직렬화하는것에 있어 하나의 메서드를 지정하도록 한다. 
- 아래의 예시는 Enum타입 TypeEnumWithValue을 직렬화 시 value로 getName() 메서드를 사용하도록 지정한 것
```java
public enum TypeEnumWithValue {

	TYPE1(1, "Type A"), TYPE2(2, "Type 2");

	private Integer id;
	private String name;

	TypeEnumWithValue(Integer id, String name) {
		this.id = id;
		this.name = name;
	}

	@JsonValue
	public String getName() {
		return name;
	}
}
```
```java
@Test
public void testJsonValue() throws JsonProcessingException {
    System.out.println(new ObjectMapper().writeValueAsString(TypeEnumWithValue.TYPE1));
    System.out.println(new ObjectMapper().writeValueAsString(TypeEnumWithValue.TYPE2));
}

/** 적용 전 */
"TYPE1"
"TYPE2"

/** @JsonValue 적용 후*/
"Type A"
"Type 2"
``` 

## @JsonRootName
- Class, Enum, Interface위에 선언되어 해당 클래스가 wrapping 될 수 있다면, value가 root wrapper로 처리되도록 한다. 
- 이 애노테이션은 ObjectMapper에서도 함께 enable.(SerializationFeature.WRAP_ROOT_VALUE)를 처리해주어야 동작한다. 
- 주의할 점은 ObjectMapper에서만 처리하고 @JsonRootName 애노테이션을 붙여주지 않는다면 기본으로 root wrapper는 클래스의 이름으로 처리가 된다 (이러한 처리는 깔끔하지 못하다). 
```java
@JsonRootName(value = "user")
public class UserWithRoot {
    public int id;
    public String name;
}
```
```java
@Test
public void testJsonRootName() throws JsonProcessingException {

    UserWithRoot userWithRoot = new UserWithRoot(1, "changwoo.son");

    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.enable(SerializationFeature.WRAP_ROOT_VALUE); // enable wrapping

    System.out.println(objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(userWithRoot));
}

/** @JsonRootName, SerializationFeature.WRAP_ROOT_VALUE 적용 전 */
{
  "id" : 1,
  "name" : "changwoo.son"
}

/** @JsonRootName 적용 전, SerializationFeature.WRAP_ROOT_VALUE 적용 후 */
{
  "UserWithRoot" : {
    "id" : 1,
    "name" : "changwoo.son"
  }
}

/** @JsonRootName, SerializationFeature.WRAP_ROOT_VALUE 적용 후 */
{
  "user" : {
    "id" : 1,
    "name" : "changwoo.son"
  }
}
```

## @JsonSerialize
- marshalling 과정에 custom serializer를 사용하도록 지정
- marshalling과 serialization은 차이가 있다. 
    - serialization은 객체를 byte stream으로 변환하는 것 만을 의미
    - 이에 반해 marshalling은 객체를 저장이나 전송을 위해 적당한 자료 형태로 변형하는 것을 의미
- Jackson 라이브러리의 StdSerializer를 상속하여 custom serializer를 만들 수 있다.
- 아래의 예시에선 Date 객체를 CustomDateSerializer를 이용하여 직렬화하도록 지정해주었다.  
```java
public class EventWithSerializer {

    public String name;
 
    @JsonSerialize(using = CustomDateSerializer.class)
    public Date eventDate;
}
```
```java
public class CustomDateSerializer extends StdSerializer<Date> {

	private static SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

	public CustomDateSerializer() {
		this(null);
	}

	public CustomDateSerializer(Class<Date> t) {
		super(t);
	}

	@Override
	public void serialize(Date value, JsonGenerator gen, SerializerProvider provider) throws IOException {
		gen.writeString(formatter.format(value));
	}
}
```
```java
@Test
public void testJsonSerialize() throws JsonProcessingException, ParseException {

	SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

	String toParse = "2020-09-10 01:23:45";
	Date date = format.parse(toParse);
	EventWithSerializer event = new EventWithSerializer("party", date);

	System.out.println(new ObjectMapper().writeValueAsString(event));
}
```
```java
/** @JsonSerialize 적용 전 */
{
  "name" : "party",
  "eventDate" : 1599668625000
}

/** @JsonSerialize 적용 후 */
{
  "name" : "party",
  "eventDate" : "2020-09-10 01:23:45"
}
```