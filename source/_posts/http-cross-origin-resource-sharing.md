---
title: CORS(Cross Origin Resource Sharing, 교차 출처 리소스 공유)
description: 서로 다른 도메인간의 리소스를 공유할 수 있게 하는 기법인 CORS에 대해서 알아봅니다.
---

## 소개
브라우저는 동일 출처 정책(Single Origin Policy, SOP)에 의해 스크립트 단에서 다른 도메인의 서버로 HTTP 요청을 하는 것이 제한된다. 예를 들면 브라우저를 통해 "naver.com"이라는 URL의 웹 페이지에서 `XMLHttpRequest` API를 통해 "open-api.com"의 API 서버에 요청하는 것은 차단된다.

 > *대신에 `XMLHttpRequest`가 아니라 `<script>`, 혹은 `<link>`와 같이 태그 형태로 외부의 리소스를 받아오는 것에 대해서는 동일 출처 정책이 적용되지 않는다. 이를 이용한 기법으로는 JSONP가 있다.*

 점차 웹 어플리케이션의 역할이 커지면서 개발자들은 이 제한을 우회할 수 있는 방법들을 요구하기 시작했다. 그래서 브라우저 벤더사들은 CORS(Cross Origin Resource Sharing, 교차 출처 리소스 공유)라는 메커니즘을 브라우저에 적용했으며, CORS는 W3C의 권고안이 되어 완전한 웹 표준이 되었다. 

 하지만 동일 출처 정책이 없어진 것은 아니며, 추가적인 작업을 해줘야만 서로 다른 도메인 간의 리소스를 공유 할 수 있다. CORS 요청을 가능하게 하기 위해서는 다른 도메인을 가진 서버 쪽에서 **특별한 응답 헤더를 설정** 해야 한다. 앞서 살펴본 경우를 예로 들자면, "naver.com" 서버가 아니라 "open-api.com" 서버 쪽에서 설정해야만 한다.

 ## 매커니즘
 만약 스크립트 단에서 다른 도메인의 서버로 HTTP 요청을 하게 되면, 브라우저는 실제 요청을 보내기 전에 **‘사전 요청(preflight)’**을 보낸다. 그리고 **‘사전 요청’**에 대한 응답을 받은 후 요청이 가능하다고 판단되면, 브라우저는 원래대로 실제 HTTP 요청을 보낸다. 과정을 요약한다면 다음과 같다.

1. (JavaScript) `XMLHttpRequest`로 HTTP 요청 
2. (Browser) 다른 도메인 서버로 가는 HTTP 요청임을 감지
3. (Browser) 사전 요청 전송 ( HTTP OPTIONS METHOD )
4. (Server) 사전 요청에 대한 응답 전송 (  알맞는 헤더가 설정된 응답 )
5. (Browser) CORS 요청가 가능하다고 판단
6. (Browser) 원래의 HTTP 요청을 계속 함



 <u>‘1’번</u>을 제외한 나머지 과정은 브라우저와 서버가 알아서 해주는 과정이다. 서버는 이 과정이 시작되기 전에 응답에 알맞는 헤더가 설정되도록 작업하기만 하면 된다. 여기에서 <u>‘3’번</u>의 사전 요청 전송 과정은 생략되는 경우도 있다. 이것은 클라이언트에서 어떤 헤더나 메서드로 요청을 하는가에 따라서 달라진다. 

## 간단한 요청

 사전 요청을 하지 않아도 된다고 판단되는 **간단한 CORS 요청**들은 다음의 세가지 조건을 **<U>모두 만족</U>**해야 한다.

* 메서드는 다음 중 하나여야 한다 : `GET`, `HEAD`, `POST `
* 스크립트에서 수동 설정이 허용되는 헤더는 다음 중 하나여야 한다 : `Accept`, `Accept-Language`, `Content-Language`, `Content-Type` 
* `Content-Type` 헤더 값은 다음 중 하나여야 한다 : `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain`

그리고 당연한 이야기지만, 이 모든 조건이 만족해도 서버에서 알맞은 헤더가 사전에 설정되어 있지 않으면 브라우저에 의해 CORS 요청은 거부된다.

아래는 **"foo.example"** 페이지를 보고 있는 브라우저와 **"bar.other"** 서버가 주고받은 HTTP 요청, 응답의 내용이다.

```text
GET /resources/public-data/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1. 9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
Referer: http://foo.example/examples/access-control/simpleXSInvocation.html
Origin: http://foo.example


HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2.0.61 
Access-Control-Allow-Origin: *
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml

[XML Data]
```

첫 번째 블록은 HTTP GET 요청의 헤더인데, `Host`와 `Origin` 헤더의 주소가 다르므로 이 요청은 CORS 요청에 해당된다. 그런데 HTTP 메서드가 GET이고, 스크립트에서 수동 설정되는 헤더는 없기 때문에 브라우저는 사전 요청을 보내지 않는다. 

두 번째 블록은 HTTP GET 요청에 대한 응답의 내용이다. 여기에서 CORS 요청을 가능하게 끔 하는 가장 중요한 헤더는 `Access-Control-Allow-Origin`이다. 이 헤더가 설정되지 않았거나, `*` 혹은 `http://foo.example`이 포함되지 않았다면 요청은 거부되었을 것이다. 이 헤더는 앞에서 말한 것처럼 서버 단에서 별도로 설정해줘야 한다.

예를 들어서 NodeJS 서버에서 `Access-Control-Allow-Origin` 헤더를 설정한다면 다음과 같을 것이다.

```javascript
// …
var express = require('express');
var app = express();

app.use((req, res, next) => {
    res.header('Access-Control-Allow-Origin', '*');
    next();
});
// …
```

## 사전 요청

간단한 요청이 되는 경우를 제외하고, 다른 출처간의 리소스가 공유되기 전에 브라우저에서 받아들여도 되는 리소스인지 확인하기 위해서는 **사전요청**이 전달된다. 만약 다음의 조건 중 **<U>하나라도 해당된다면</U>** 실제 HTTP 요청 이전에 **사전 요청**이 전달된다. 

* GET, POST, HEAD 이외의 메서드인 요청
* 요청이 `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain` 이외의 값을 가진`Content-Type`으로 전송될 경우
* 커스텀 헤더를 설정하는 경우

다음은 사전 요청이 전달될 수 있는 자바스크립트의 코드이다.

```javascript
var invocation = new XMLHttpRequest();
var url = 'http://bar.other/resources/post-here/';
var body = '<?xml version="1.0"?><person><name>Arun</name></person>';
    
function callOtherDomain(){
  if(invocation)
    {
      invocation.open('POST', url, true);
      invocation.setRequestHeader('X-PINGOTHER', 'pingpong');
      invocation.setRequestHeader('Content-Type', 'application/xml');
      invocation.onreadystatechange = handler;
      invocation.send(body); 
    }
}
// ......
```

그리고 다음은 위의 스크립트가 실행되면서 브라우저와 서버 간 요청과 응답을 주고받는 내용이다.

```text
OPTIONS /resources/post-here/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
Origin: http://foo.example
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER


HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER
Access-Control-Max-Age: 1728000
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain

POST /resources/post-here/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
X-PINGOTHER: pingpong
Content-Type: text/xml; charset=UTF-8
Referer: http://foo.example/examples/preflightInvocation.html
Content-Length: 55
Origin: http://foo.example
Pragma: no-cache
Cache-Control: no-cache

<?xml version="1.0"?><person><name>Arun</name></person>


HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:40 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://foo.example
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 235
Keep-Alive: timeout=2, max=99
Connection: Keep-Alive
Content-Type: text/plain

[Some GZIP'd payload]
```

먼저 살펴볼 부분은 첫 번째 요청인 HTTP OPTIONS 메서드의 사전 요청이다. 

```text
OPTIONS /resources/post-here/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
Origin: http://foo.example
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER
```

사전 요청이 일어난 이유는 스크립트를 살펴보면 알 듯이 커스텀 헤더인 `X-PINGOTHER`이 설정되었기 때문이다. 여기서 `Access-Control-Request-Method`로는 실제 요청이 어떤 메서드로 이뤄지는지 서버에게 알려준다. `Access-Control-Request-Headers`는 실제 요청에 어떤 헤더가 세팅될 것인지 알려주는데, 여기에 `Content-Type`과 같은 기본적인 헤더들은 리스트업되지 않는다. 이 두 개의 헤더는 프로그래머가 직접 설정해주는 것이 아니며, 브라우저에 의해 사전 요청에 자동 삽입되는 헤더이다.

다음으로 살펴볼 부분은 첫 번째 HTTP OPTIONS 요청에 대한 응답이다.

```text
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER
Access-Control-Max-Age: 1728000
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```

OPTIONS 요청에 대한 응답을 살펴보면 `Access-Control-Allow-Origin` 헤더가 웹 페이지의 출처인 `http://foo.example`을 가르키고 있다. 그리고 `Access-Control-Allow-Methods`는 허용되는 HTTP 메서드들을 말하는데, 스크립트 상에서 요청되는 메서드인 POST가 포함되어 있다. 그리고 `Access-Control-Allow-Headers`는 클라이언트의 요청에 설정됐던 `X-PINGOTHER` 헤더를 포함하고 있다. 이 OPTIONS 요청에 대한 응답으로 인해서 CORS 요청이 승인되고, 다음으로 실질적인 HTTP POST 요청, 응답이 이뤄진다.

추가적으로, `Access-Control-Max-Age` 헤더는 사전 전달 요청이 얼마동안 캐시되는지 알려주는 초 단위의 값이다. 사전 전달 요청이 캐시되면 이 헤더에 설정된 시간 동안은 사전전달 요청이 전달되지 않는다.

## 인증을 이용한 요청

서버와 통신하는 부분의 코드를 살펴보다보면 흔하게 봤었던 것이 `withCrendentials` 플래그이다. 

```javascript
var invocation = new XMLHttpRequest();
var url = 'http://bar.other/resources/credentialed-content/';
    
function callOtherDomain(){
  if(invocation) {
    invocation.open('GET', url, true);
    invocation.withCredentials = true;
    invocation.onreadystatechange = handler;
    invocation.send(); 
  }
}
```

`withCredentials` 플래그의 역할은 CORS 요청을 하면서 인증 정보(ex. 쿠키, authorization 헤더, 또는 TLS 클라이언트 인증서)를 서버로 보낼 수 있게 해준다. 그러면 서버 쪽 응답에서는 `Access-Control-Allow-Credentials` 헤더가 `true`로 설정되어 있어야한다. 스크립트에서 `withCredentials` 플래그가 설정됐는데도 `Access-Control-Allow-Credentials` 헤더가 `true`가 아니라면, 아래와 같이 처리된다.

* 사전 요청이 없는 경우에는 브라우저가 요청에 대한 응답을 무시한다.
* 사전 요청이 전달되는 경우에는 실질적인 Cross Origin 요청에 인증 정보를 사용될 수 없게 된다.

## 참고자료

* https://developer.mozilla.org/ko/docs/Web/HTTP/Access_control_CORS
* https://homoefficio.github.io/2015/07/21/Cross-Origin-Resource-Sharing/
* https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials