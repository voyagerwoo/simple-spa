# SPA와 SPA 라우팅 원리
이번 포스팅에서 웹서비스가 Single Page Application(이하 SPA)까지 발전하게 된 경위와 SPA의 라우팅의 원리에 대해서 알아보려고 합니다. 스펙보다는 개인적인 경험과 지식 위주로 정리했기 때문에 부족한 부분이 있을 수 있습니다. 피드백 주세요!

### 목차
- 기존의 웹서비스
- Ajax로 부분만 새로 그리는 웹서비스
- Single Page Application
- SPA의 라우팅 원리
- 정리

### 실습
- serve를 설치한다. `npm install -g serve` (https://www.npmjs.com/package/serve)
- git clone https://github.com/voyagerwoo/simple-spa
- 기존 웹서비스 예제 서버 실행 : `serve -p 3001 01_old`
- Ajax 웹서비스 예제 서버 실행 : `serve -p 3002 02_ajax`
- SPA 웹서비스 예제 서버 실행 : `serve -p 3003 03_spa`
- http://localhost:3001, http://localhost:3002, http://localhost:3003 들어가서 확인해본다.

## 01 기존의 웹서비스
기존의 웹서비스는 링크(앵커 `<a href="#">`)를 클릭하면 해당 페이지로 이동하게 된다. 정확히 이야기하면 앵커에 명시되어 있는 자원(일반적으로 html)을 서버에 요청하고 응답으로 받은 내용을 브라우저에 표현하게 된다. 이런 식으로 매 페이지마다 서버에 문서(html)을 요청하고 응답받아서 표현한다.

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>MAIN</title>
  </head>
  <body>
    <nav>
      <ul>
        <li><a href="/">main</a></li>
        <li><a href="sub1.html">sub1</a></li>
        <li><a href="sub2.html">sub2</a></li>
      </ul>
    </nav>
    <section>
      <h1>MAIN</h1>
      This is main page.
    </section>
  </body>
</html>
```


```html
<!-- sub1.html -->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>SUB1</title>
  </head>
  <body>
    <nav>
      <ul>
        <li><a href="/">main</a></li>
        <li><a href="sub1.html">sub1</a></li>
        <li><a href="sub2.html">sub2</a></li>
      </ul>
    </nav>
    <section>
      <h1>SUB1</h1>
      This is sub 1 page.
    </section>
  </body>
</html>
```
이런 전통적인 방식에서는 서버에서 html을 완성해서 보내준다. 이걸 서버사이드 렌더링이라고 한다.
##### 장점
첫째, 브라우저가 응답을 받자마자 렌더링을 할 수 있어서 빠르다는 점이 있다.

둘째, 자바스크립트 코드가 없어서 훨씬 쉽게 작성할 수 있다.
##### 단점
첫째, 중복되는 데이터가 계속 네트워크를 타고 넘어온다는 것이다. 예를 들어서 위 코드의 네비게이션 영역은 모든 페이지에서 동일하게 보여지는 부분인데 이 코드가 매번 서버로 요청할 때 마다 응답으로 넘어간다. 낭비일 수 있다.

## 02 Ajax로 특정 부분만 새로 그리는 웹서비스
기존 웹서비스의 단점을 보완하기 위해서 서버에서 첫 화면을 그리고 다음부터는 ajax 방식으로 데이터만 가져온 뒤에 클라이언트에서 html을 렌더링하는 작업을 많이 했다. 즉 필요한 부분만 다시 그리도록 자바스크립트 코드를 작성했다.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>MAIN</title>
  </head>
  <body>
    <nav>
      <ul>
        <li><a href="/">main</a></li>
        <li><a id="navSub1" href="">sub1</a></li>
        <li><a id="navSub2" href="">sub2</a></li>
      </ul>
    </nav>
    <section>
      <h1>MAIN</h1>
      This is main page.
    </section>
    <script>
    (function(){
      document.getElementById('navSub1').addEventListener('click', function(e) {
        e.preventDefault();
        drawSection('/sub1.json');
      });

      document.getElementById('navSub2').addEventListener('click', function(e) {
        e.preventDefault();
        drawSection('/sub2.json');
      });

      function drawSection(url) {
        ajaxGet(url, function(response) {
          var data = JSON.parse(response);
          document.querySelector('section').innerHTML =
          '<h1>' + data.title +  '</h1>'
          +data.content
        });
      }

      function ajaxGet(url, callback) {
        url = url || '';
        callback = callback || function () { ; };
        var xhr = new XMLHttpRequest();
        xhr.open("get", url, true);
        xhr.setRequestHeader('X-Requested-With', 'XMLHttpRequest');
        var that = this;
        xhr.onload = function () {
            callback.apply(that, [xhr.response]);
        };
        xhr.send(null);
      }
    })();
    </script>
  </body>
</html>
```

코드가 굉장히 복잡해졌다. 위에서부터 차근차근 보게 되면 각 앵커에 이벤트 리스너(핸들러)를 붙였다. 각 앵커를 클릭하게 되면 데이터를 서버에 요청해서 받은 응답으로 html을 새로 만들고 section의 innerHTML로 붙이는 코드이다.

##### 장점
첫째, 필요한 부분만 새로 그리기 때문에 낭비가 없다.

둘째, 이 방법으로 기존의 링크를 타고 다니던 웹서비스보다 편한 사용자 경험을 줄 수 있다.
##### 단점
첫째, 히스토리 관리가 안된다. 뒤로가거나 앞으로 가거나 하면 브라우저는 sub1, sub2 앵커를 눌렀을 때의 상태는 전혀 없다는 듯이 행동한다.

둘째, sub1 앵커를 눌렀을 때 어떤 액션을 하는지 추적하기가 어렵다. 예전에는 href에 가야할 주소가 있었는데 지금은 그냥 빈 문자열만 있다. 저 먼곳에 이벤트 리스너에 어떤 액션을 하는지 적어두었다.

셋째, 자바스크립트 코드가 너무 많아졌다.

이 정도만 해도 page가 하나이기 때문에 SPA라고 할 수 있지만 현재의 SPA는 한걸음 더 나아간다.

## 03 Single Page Application
위키피디아에서 SPA의 정의를 찾아보면 아래와 같은 내용이 있다.

>The page does not reload at any point in the process, nor does control transfer to another page, although the location hash or the HTML5 History API can be used to provide the perception and navigability of separate logical pages in the application.

location.hash와 HTML5의 history API를 통해서 예전 웹 페이지처럼 논리적으로 페이지를 분리하고 그 분리된 페이지를 이동하는 것이 가능하다는 뜻이다. hash는 URI에서 #으로 시작하는 문자열을 의미하는데 정확히는 Fragment Identifier라고 한다.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>MAIN</title>
  </head>
  <body>
    <nav>
      <ul>
        <li><a href="#">main</a></li>
        <li><a href="#sub1">sub1</a></li>
        <li><a href="#sub2">sub2</a></li>
      </ul>
    </nav>
    <section>
      <h1>MAIN</h1>
      This is main page.
    </section>
    <script>
    (function(){
      var sectionEl = document.querySelector('section');
      var mainHtml = sectionEl.innerHTML;
      var routerMap = {
        '' : function() {
          sectionEl.innerHTML = mainHtml;
        },
        'sub1' : function() {
          drawSection('sub1.json')
        },
        'sub2' : function() {
          drawSection('sub2.json')
        }
      }

      function otherwise() {
        sectionEl.innerHTML =
          'Not Found';
      }

      function router() {
        var hashValue = location.hash.replace('#', '');
        (routerMap[hashValue] || otherwise)();
      }


      function drawSection(url) {
        //02와 동일하므로 생략
      }

      function ajaxGet(url, callback) {
        //02와 동일하므로 생략
      }

      window.addEventListener('DOMContentLoaded', router);
      window.addEventListener('hashchange', router);
    })();
    </script>
  </body>
</html>
```
휠씬 더 스크립트가 많아졌다. 중요한 차이는 해시값이 변경되는 시점(`on hashchange`)에 해시값에 매핑된 함수를 실행하도록 해서 라우팅이 되도록 한것이다. 그리고 앵커에도 해시값을 넣어서 명시적으로 확인할 수 있도록 했다. 이렇게 코드를 작성함으로써 논리적으로 페이지가 분리되고 히스토리도 관리가 된다. 왜 히스토리 관리가 되는지는 SPA 라우팅의 원리에서 자세히 알아보자.

##### 장점
첫째, 히스토리 관리가 된다.

둘째, 논리적으로 페이지가 분리되어서 이동할 수 있다.

##### 단점
js 코드가 더 많아졌다. 그러나 역할별로 분리한다면 크게 관리가 어렵지 않다.

##### 참고 - DOMContentLoaded 이벤트가 발생하는 시점에도 router 함수를 실행시키게 한 이유
해시값을 가지고 있는 URL을 브라우저에서 요청할 경우에도 라우팅이 되도록 하기 위함이다. 예를 들어 `http://localhost:3000/#sub1`이라고 브라우저 주소창에 입력하고 enter를 치거나, 새로고침을 누르게 되면 서버로 `http://localhost:3000` 페이지를 요청하게 된다. 그리고 페이지의 dom content loaded 시점에 라우팅 로직을 실행하여 해당 해시값에 맞는 페이지를 또 클라이언트 사이드에서 렌더링 하게 된다. 기능의 일관성을 지키기 위함이다.

## SPA의 라우팅 원리
SPA 작성을 돕는 여러 프레임워크도 보통은 이런방식의 routing을 한다. 그 원리를 한번 알아보자.

#### URI 구성
http: //reimaginer:password@www.tistory.com:8011/search?q=xper#n10
- URI Scheme : http
- 사용자 정보 : reimaginer:password
- hostname : www.tistory.com
- port : 8000
- path : /search
- query parameter (=쿼리 문자열): q=test&debug=true
- fragment identifier : n10  --> # 앞의 문자열로 표현한 URI가 가리키는 리소스 내부에서 더 세세한 부분을 특정할 때 이용.

#### hash는 변경해도 서버에 페이지가 갱신되지 않는다
브라우저에서 URI의 호스트명, 패쓰, 쿼리 파라미터를 변경하게 되면 해당 주소로 서버에 요청하고 응답을 받아서 화면이 갱신된다. 그러나 fragment identifier 즉 hash는 변경되어도 화면은 갱신되지 않는다. 왜냐하면 hash는 문서에서 부차적인 자원에 대한 참조를 가리키기 위해서 만들었기 때문이다. 예를 들어 `https://tools.ietf.org/html/rfc3986#section-3.5`에서 #section-3.5는 rfc3986이라는 문서에서 section-3.5를 가리킨다. 문서 내의 참조이기 때문에 화면이 갱신될 필요가 없는 것이다.

이 규칙을 이용해서 SPA에서는 hash를 이용하여 라우팅을 한다.


#### hash를 변경하면 history에 쌓인다
hash를 변경하여도 서버에 새로 요청을 보내서 페이지를 갱신하거나 하지는 않지만 history는 쌓인다. (URI가 변경되면 히스토리가 쌓인다.) 쉽게 설명하면 뒤로가기를 누르게 되면 해시를 변경하지 않았던 상태로 이동하고 다시 앞으로 가기를 누르게 되면 해시를 변경했던 상태로 이동할 수 있게 되는 것이다.

이를 이용하여 SPA에서 히스토리를 관리할 수 있게 된다.

#### hashchange & popstate event
브라우저에서 hash가 변경될 때마다 hashchange & popstate 이벤트가 발생한다. hashchange는 말 그대로 hash가 변경되었다는 의미의 이벤트이고 popstate는 history entry가 변경되었을 경우에 발생하는 이벤트이다.
이번 코드에서 hashchange 이벤트를 사용한 이유는 popstate는 HTML5 스펙이기 때문에 하위 브라우저에서 동작하지 않기 때문이다. hashchange 이벤트는 ie8 이상 브라우저에서 지원한다.

```js
var routerMap = {
  '' : function() {
    sectionEl.innerHTML = mainHtml;
  },
  'sub1' : function() {
    drawSection('sub1.json')
  },
  'sub2' : function() {
    drawSection('sub2.json')
  }
}

function otherwise() {
  sectionEl.innerHTML =
    'Not Found';
}

function router() {
  var hashValue = location.hash.replace('#', '');
  (routerMap[hashValue] || otherwise)();
}


window.addEventListener('hashchange', router);
```
위 코드처럼 hashchange 이벤트에 라우팅하는 이벤트 핸들러를 붙이면 된다. 지금은 정말 간단한 웹페이지라 코드가 단순하지만 나중에는 drawSection하는 부분에 템플릿을 만들고 통신하고 데이터를 바인딩하고 이벤트도 바인딩하는 복잡한 코드가 추가될 것이다. 그리고 지금 나온 angular, react, vue 등을 사용하면 편리하게 복잡한 코드 없이 SPA를 작성할 수 있다.

##### 참고 - if-else가 아닌 map이라는 자료구조를 사용한 이유
코드 퀄리티 관한 책을 읽다보면 자연스레 if-else를 싫어하게 된다. 그 이유는 복잡도가 높아지고 자유도 또한 높아져 점점 관리가 안되는 코드를 생산하기 때문이다. 예를 들어 지금 코드를 if-else로 바꾼다고 하면 아래처럼 작성할 것이다.

```js
function router() {
  if (location.hash == 'sub1') {
    drawSection('sub1.json');
  } else if (location.hash == 'sub2') {
    drawSection('sub2.json');
  } else {
    otherwise();
  }
}
```
그런데 페이지가 점점 많아지면서 if-else 블록이 많아지게 된다면 정말 보기 싫은 코드가 된다.

그런데 그냥 map이란 자료구조를 사용하여 hash값과 해야할 일(함수)을 매핑해두고 그냥 맵에 있는 값을 가져와서 실행하는 식으로 작성한다면 코드가 좀더 간단해진다. 만약 메뉴가 많이 늘어나더라도 맵에만 hash값과 함수만 매핑해주면 된다. 이게 편한 이유는 실행하는 영역(`router function`)과 페이지를 매핑하는 영역(`routerMap`)을 분리했기 때문이다. if-else로 작성하면 그 두 영역이 같이 있기 때문에 더 복잡했던 것이다.

## 정리
기존의 웹서비스, ajax, SPA 발전상황에 따라 다른 방식이라고 설명했지만, 사실 실제 서비스에서는 이 세가지를 상황에 맞게 섞어서 웹서비스를 만든다. 그래도 이 세가지의 발전 흐름을 아는 것은 초보자가 프론트앤드 개발을 하는데 있어서 도움이 될것이라 생각한다.

또한 해시를 이용한 SPA에서 라우팅을 이해하는 것은 SPA라는 것의 작동원리를 이해하는 데도 도움이 되며, 간단한 기능의 경우에는 스스로 구현해볼 수 있는 기회를 줄 것이다.

### 관련 링크
- Fragment Identifier : https://en.wikipedia.org/wiki/Fragment_identifier
- rfc3986 : https://tools.ietf.org/html/rfc3986#section-3.5
- HTML5 PushState()를 활용한 PJAX 구현 :  http://mohwaproject.tistory.com/entry/HTML5-PushState%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-PJAX-%EA%B5%AC%ED%98%84
- Ajax를 사용할 때 웹브라우저 "뒤로 가기"의 구현 : https://blog.outsider.ne.kr/1276
