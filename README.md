# HTTP 서버 구현하기 미션

* [1단계 Pull Request](https://github.com/woowacourse/jwp-dashboard-http/pull/61)
* [2단계 Pull Request](https://github.com/woowacourse/jwp-dashboard-http/pull/103)


## 1단계 HTTP 서버 구현하기
### GET /index.html 응답하기

서버를 실행시켜서 브라우저로 서버(http://localhost:8080/index.html)에 접속하면 index.html 페이지를 보여준다.
브라우저에서 요청한 http request header는 다음과 같다.

```
HTTP Header
GET /index.html HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Accept: */*
```

1. 먼저 첫 번째 라인(Request URI) 을 읽어오자.
2. 나머지 http request header는 어떻게 읽을까?
  * IOStreamTest를 참고해서 http request를 읽어오자.
  * line이 null인 경우에 예외 처리를 해준다. 그렇지 않으면 무한 루프에 빠진다.
    * if (line == null) { return; }
  * 헤더 마지막은 while (!"".equals(line)) {}으로 확인 가능하다.
3. http request의 첫 번째 라인에서 request uri를 추출한다.
  * line.split(" ");을 활용해서 문자열을 분리한다.
4. FileTest를 참고해서 요청 url에 해당되는 파일을 resource 디렉토리에서 읽는다.

### Query String 파싱

1. http request의 첫 번째 라인에서 request uri를 추출한다.
2. login.html 파일에서 태그에 name을 추가해준다.
3. 추출한 request uri에서 접근 경로와 이름=값으로 전달되는 데이터를 추출해서 User 객체를 만든다.
```
String uri = "/login?account=gugu&password=password";
int index = uri.indexOf("?");
String path = uri.substring(0, index);
String queryString = uri.substring(index + 1);
```
4. InMemoryUserRepository를 사용해서 미리 가입되어 있는 회원을 조회해서 로그로 확인해보자.

### HTTP Status Code 302

회원을 조회해서 로그인에 성공하면 ```/index.html```로 리다이렉트한다.

```/login?account=gugu&password=password```로 접근해서 로그인 성공하면 응답 헤더에 **http status code**를 **302**로 반환한다.

로그인에 실패하면 **401.html로 리다이렉트**한다.

### POST 방식으로 회원가입 

http://localhost:8080/register으로 접속하면 회원가입 페이지(register.html)를 보여준다.   
회원가입 페이지를 보여줄 때는 GET을 사용한다.   
회원가입을 버튼을 누르면 HTTP method를 GET이 아닌 POST를 사용한다.   
회원가입을 완료하면 index.html로 리다이렉트한다.   
로그인도 버튼을 눌렀을 때 GET 방식에서 POST 방식으로 전송하도록 변경하자.   

요청 예시
```
POST /register HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Content-Length: 80
Content-Type: application/x-www-form-urlencoded
Accept: */*

account=gugu&password=password&email=hkkang%40woowahan.com
```

1. while 문으로 http request header를 읽고 나서 request body를 읽어온다.
  * request header의 Content-Length가 request body의 length다.
```java
int contentLength = Integer.parseInt(httpRequestHeaders.get("Content-Length"));
char[] buffer = new char[contentLength];
reader.read(buffer, 0, contentLength);
String requestBody = new String(buffer);
```
2. POST로 전달하면 GET과 다르게 http request body에 데이터가 담긴다.
3. login.html도 form 태그를 수정한다.
4. InMemoryUserRepository에서 save 메서드를 사용해서 가입 완료 처리한다.
5. 이상 없으면 리다이렉트

### CSS 지원하기
클라이언트에서 요청하면 CSS 파일도 제공하도록 수정한다.
```
GET /css/styles.css HTTP/1.1
Host: localhost:8080
Accept: text/css,*/*;q=0.1
Connection: keep-alive
```

1. 응답 헤더의 Content-Type을 text/html로 보내면 브라우저는 HTML 파일로 인식하기 때문에 CSS가 정상적으로 동작하지 않는다.
2. CSS인 경우 응답 헤더의 Content-Type을 text/css로 전송한다.
3. Content-Type은 확장자를 통해 구분할 수도 있으며, 요청 헤더의 Accept를 활용할 수도 있다.


## 2단계 리팩터링

* 요구사항
  * HTTP 서버를 구현한 코드의 복잡도가 높아졌다.
  * 적절한 클래스를 추가하고 역할을 맡겨서 코드 복잡도를 낮춰보자

### HttpRequest
HTTP 요청을 처리하는 클래스를 추가한다.   
HTTP 요청은 어떤 형태로 구성되어 있는가?   
클래스로 HTTP 요청을 어떻게 구성하면 좋을까?   

HTTP 요청 이미지를 참고해서 구현해보자

![image](https://user-images.githubusercontent.com/57378410/148748833-beb4efef-49d1-4d9b-a976-e02fba0cfc08.png)

HTTP 요청의 첫 줄은 Request Line이라고 부른다.

### HttpResponse

HTTP 응답을 처리하는 클래스를 추가한다.   
HTTP 응답은 어떤 형태로 구성되어 있는가?   
클라이언트에게 어떤 형태로 HTTP를 응답하면 좋을까?   

![image](https://user-images.githubusercontent.com/57378410/148748916-b6cd500b-c6d0-442b-9356-2065a0679063.png)

### Controller 인터페이스 추가

HTTP 요청, 응답을 다른 객체에게 역할을 맡기고 나니까 uri 경로에 따른 if절 분기 처리가 남는다.   
if절 분기는 어떻게 리팩토링하는게 좋을까?   
컨트롤러 인터페이스를 추가하고 각 분기에 있는 로직마다 AbstractController를 상속한 구현체로 만들어보자.   

```java
public interface Controller {
    void service(HttpRequest request, HttpResponse response) throws Exception;
}
```
```java
public abstract class AbstractController implements Controller {

    @Override
    public void service(HttpRequest request, HttpResponse response) throws Exception {
        // http method 분기문
    }

    protected void doPost(HttpRequest request, HttpResponse response) throws Exception { /* NOOP */ }
    protected void doGet(HttpRequest request, HttpResponse response) throws Exception { /* NOOP */ }
}
```

http request 객체의 요청을 처리할 컨트롤러 객체를 반환한다.
