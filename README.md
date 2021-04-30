# Swagger?

스웨거(Swagger)는 개발자가 REST 웹 서비스를 설계, 빌드, 문서화, 소비하는 일을 도와주는 대형 도구 생태걔의 지원을 받는 오픈 소스 스프트웨어 프레임워크입니다. 현재 진행하고 있는 프로젝트에 대해서 유지보수를 진행하거나 API를 만들게 될때 API서버가 어떤 스펙을 가지고 있는지 파악해야 한다. Swagger는 API의 명세와 문서를 대신 작성해 주는 아주 착한 친구 입니다.

![swagger](https://static1.smartbear.co/swagger/media/assets/images/swagger_logo.svg)

[Swagger]: https://swagger.io/	"Swagger"

스웨거의 공식 사이트입니다. 저 자세한 정보르 알고싶다면 위 링크에 접속해서 알아보시면 될거에요.

여러가지 환경에서 Swagger를 사용가능한데 이 글에서는 Spring Boot + Swagger 세팅을 해볼께요.



# Spring Boot + Swagger

SpringBoot 스펙

* JDK 8
* Spring Boot 2.4.5
* Mave

먼저 SpringBoot 앱을 생성해 줍니다

![SpringBootSwagger1](https://user-images.githubusercontent.com/54675591/116714336-a044e380-aa10-11eb-90ac-0f8be68590c3.PNG)

![SpringBootSwagger2](https://user-images.githubusercontent.com/54675591/116714359-a9ce4b80-aa10-11eb-9eaa-e8ad8161cca1.PNG)

![SpringBootSwagger2](https://user-images.githubusercontent.com/54675591/116714359-a9ce4b80-aa10-11eb-9eaa-e8ad8161cca1.PNG)

기본적이 Swagger기능만 사용 하기 때문에 SpringWeb만 <dependency>에 추가 해준후 프로젝트를 생성해줍니다.



pom.xml에서 Dependency를 추가해 줍니다.

```xml
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
	<artifactId>springfox-swagger-ui</artifactId>
	<version>2.9.2</version>
</dependency>
```

* springfox-swagger2는 Swagger를 사용하기 위한 필수 라이브러리입니다.
* springfox-swagger-ui는 API스펙 명세화 기능 사용등 여러가지 기능을 UI로 사용하기 위해서 필요한 라이브러리 입니다.

관련 라이브러리가 다 다운되면 Swagger관련되 설정을 하기 위한 설정 클래스 SwaggerConfig를 만들어줍니다.



```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
	
	@Bean
	public Docket api() {
		return new Docket(DocumentationType.SWAGGER_2)
				.select()
				.apis(RequestHandlerSelectors.any())//Swagger API 문서로 만들기 원하는 basePackage 경로
				.paths(PathSelectors.ant("/*/**"))	//URL 경로를 지정하여 해당 URL에 해당하는 요청만 SWAGGER로 만듦
				.build();
	}
}
```

* apis() 는 Swagger API로 문서로 만들기 원하는 basePackage 경로를 적으면 됩니다. 저는 모든 곳을 문서화 하고싶기 때문에 `RequestHandlerSelector.any()` 로 지정해주었습니다.
* path() 는 그 안의 경로의 하위만 Swagger로 만들겠다는 이야기 입니다.
  * `PathSelectors.ant("/api/**")`로 적게되면 /api 하위의 모든 API를 Swagger에서 볼 수 있도록 설정하겠다는 뜻입니다.

* 추가적인 Swagger 환경설정을 할수 있습니다. 다만 이 글은 진짜 간단하게 연동하기 위함으로 생략하겠습니다.
  * `consumes()` ` produces() ` `apiInfo()`등의 함수를 이용해서 여러가지 설정을 할 수 있습니다.



그 다음 Controller를 작성해 줍니다.

```java
@RestController("/api")
public class APiController {

    @GetMapping("getApi")
    public ResponseEntity<HashMap> getApi(@RequestParam(value="param1")String param1){
        HashMap map = new HashMap();
        map.put("param1",param1);

        return new ResponseEntity<>(map, HttpStatus.OK);
    }
}

```

이렇게 작성후 SpringBoot를 실행해준후 http://localhost:8080/swagger-ui.html 으로 접속하면 아래와 같은 화면이 나오게 됩니다.

![SpringBootSwagger4](https://user-images.githubusercontent.com/54675591/116714393-b3f04a00-aa10-11eb-825a-7088f963e5b8.PNG)

왜 a-pi-controller가 나오는지는 모르겠지만 탭을 펼쳐 보면 제가 작성했던  Get방식의 링크(/getApi)를 테스트 할수 있는 탭이 나오게 됩니다.  

![SpringBootSwagger5](https://user-images.githubusercontent.com/54675591/116714403-b81c6780-aa10-11eb-87eb-543681328283.PNG)

여기서 실제 Http를 보내볼 수도 있는데 Try it out을 눌러 `@RequestParama()`으로 지정해준 파라미터를 입력후 Execute 를 누르면 Get방식으로 Request가 전송되고 Sever response에 결과가 나오게 됩니다.

![SpringBootSwagger6](https://user-images.githubusercontent.com/54675591/116714439-bfdc0c00-aa10-11eb-98c5-e321ad196658.PNG)

좀더 추가적인 Swagger 설정을 해줄수도 있습니다.

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket api(){
        return new Docket(DocumentationType.SWAGGER_2)
                .consumes(getConsumeContentTypes())
                .produces(getProduceContentTypes())
                .apiInfo(getApiInfo())
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.ant("/*/**"))
                .build();
    }
    private Set<String> getConsumeContentTypes(){
        Set<String> consumes = new HashSet<>();
        consumes.add("application/json;charset=UTF-8");
        consumes.add("application/x-www-form-urlencoded");
        return consumes;
    }

    private Set<String> getProduceContentTypes(){
        Set<String> produces = new HashSet<>();
        produces.add("application/json;charset=UTF-8");
        return produces;
    }

    private ApiInfo getApiInfo(){
        return new ApiInfoBuilder()
                .title("API")
                .description("SpringBoot With Swagger")
                .contact(new Contact("SpringBoot with Swagger","https://github.com/go-coding1","sample@email.com"))
                .version("1.0")
                .build();
    }
}
```

* .consume()과 .produces()는 각각 Request Content-Type, Response Content-Type에 대한 설정입니다.
* .apiInfo()는 Swagger API문서에 대한 설명을 표기하는 메소드입니다.

위 와 같이 설정후 다시 SpringBoot 서버를 재시작 한 후 다시 링크로 접속하면

![SpringBootSwagger7](https://user-images.githubusercontent.com/54675591/116714461-c4082980-aa10-11eb-849c-764d3713cd83.PNG)

설정했던 정보들이 Swagger UI에 나타나게 됩니다.



추가적이 정보를 알고 싶다면  [swagger.io](https://swagger.io/) 여기 에 접속 하면 됩니다.

제가 만들 프로젝트 레포지토리는 [여기](https://github.com/go-coding1/SpringBoot_With_Swagger.git) 입니다.
