# 브라우저는 어떻게 작동할까?

출처: [https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/]

In the years of IE 90% dominance there was nothing much to do but regard the browser as a "black box", but now, with open source browsers having more than half of the usage share, it's a good time to take a peek under the engine's hood and see what's inside a web browser. Well, what's inside are millions of C++ lines...

IE가 90%이상을 지배하던 시기에는 브라우저가 **블랙박스**처럼 가려져 있었기에, 딱히 주목 할것이 없었으나, 현재는, 오픈소스 브라우저들이 아주 많이 쓰이기 때문에, 엔진이 어떻게 작동하고, 웹브라우저의 내부를 볼만할것 같다. 사실, 브라우저의 내부에는 수백만줄의 C++로 이루어진 코드들이 있다...

As a web developer, learning the internals of browser operations helps you make better decisions and know the justifications behind development best practices. While this is a rather lengthy document, we recommend you spend some time digging in; we guarantee you'll be glad you did. Paul Irish, Chrome Developer Relations.

Web Developer(웹 개발자)로써, 브라우저의 내부 연산에 대해 학습하는 것은 더 나은 결정과 최적의 개발 방식을 정의 하는것에 도움이 될것이다.
이것은 매우 긴 문서이므로, 집중할 시간을 갖는것을 추천한다. 그 시간이 아깝지 않을것이다. Paul Irish, 크롬 개발자 연합.

## Introduction (소개)

Web browsers are the most widely used software. In this primer, I will explain how they work behind the scenes. We will see what happens when you type google.com in the address bar until you see the Google page on the browser screen.

웹 브라우저들은 가장 널리 쓰는 소프트웨어이다. 뒤에서 어떤 작업을 하는지 설명해줄것이다. 당신이 google.com 을 타이핑했을때, 구글 페이지가 나타나기 전까지 어떤일이 일어나는지 볼것이다.

### The browsers we will talk about

There are five major browsers used on desktop today: Chrome, Internet Explorer, Firefox, Safari and Opera. On mobile, the main browsers are Android Browser, iPhone, Opera Mini and Opera Mobile, UC Browser, the Nokia S40/S60 browsers and Chrome–all of which, except for the Opera browsers, are based on WebKit. I will give examples from the open source browsers Firefox and Chrome, and Safari (which is partly open source). According to StatCounter statistics (as of June 2013) Chrome, Firefox and Safari make up around 71% of global desktop browser usage. On mobile, Android Browser, iPhone and Chrome constitute around 54% of usage.

### 우리가 다뤄볼 브라우저들.

요즘에는 5개의 메이저한 브라우저들이 있다: 크롬, IE, 파이어폭스, 사파리 그리고 오페라. 모바일 에서는, 메인 브라우저는 Android 브라우저, 아이폰, 오페라 미니와 오페라 모바일, UCa 브라우저, 노키아 브라우저 그리고 크롬이 있다. 오페라 브라우저를 제외하고는 모두 WebKit에 기반되어있다.
파폭, 크롬, 사파리 (사파리는 부분적 오픈소스)와 같은 오픈소스들의 예제를 보여줄것이다. 2013년 6월의 리서치에 따르면, 파폭과 사파리가 71%정도의 데스크탑 브라우저 사용량을 달하고 있었다고 한다. 모바일에서는, 안드로이드 브라우저, 아이폰 그리고 크롬이 합해서 54프로의 사용량을 차지했다.

### The browser's main functionality

The main function of a browser is to present the web resource you choose, by requesting it from the server and displaying it in the browser window. The resource is usually an HTML document, but may also be a PDF, image, or some other type of content. The location of the resource is specified by the user using a URI (Uniform Resource Identifier).

The way the browser interprets and displays HTML files is specified in the HTML and CSS specifications. These specifications are maintained by the W3C (World Wide Web Consortium) organization, which is the standards organization for the web. For years browsers conformed to only a part of the specifications and developed their own extensions. That caused serious compatibility issues for web authors. Today most of the browsers more or less conform to the specifications.

Browser user interfaces have a lot in common with each other. Among the common user interface elements are:

- Address bar for inserting a URI
- Back and forward buttons
- Bookmarking options
- Refresh and stop buttons for refreshing or stopping the loading of current documents
- Home button that takes you to your home page

### 브라우저가 주되게 하는일

브라우저의 주된 업무는 당신이 선택한 web resource를 보여주는 것이다. (서버에서 요정하여 당신의 브라우저상의 창으로)
그 resource는 대개 HTML이다. PDF도 될수 있고, 이미지나 다른 컨텐트들이 될수도 있다.
URI라는 것을 통해서 유저가 그 리소스들의 위치를 알아낼수 있다.

브라우저가 해석하고 보여주는 HTML 파일들은 HTML 언어와 CSS 스펙으로 단정지을수 있다. 이러한 스펙들은 W3C에서 관리된다. (웹의 표준을 규정하는 기구). 수년동안, 브라우저는 스펙들의 단부분으로 이루어져있었고 그들의 확장을 계속 개발해 나갔다. 그것이 웹 저자들에게 심각한 적합성 이슈들을 야기했다. 오늘날 대부분의 브라우저들은 이러한 규정들에 따르고 있다.

브라우저 사용자 인터페이스들은 (Browser UI) 다른것들과 많이 공통점이 있다. 공통된 유저 인터페이스들은 다음과 같다:

- 주소를 칠수 있는 주소창
- 앞으로 뒤로가기 버튼
- 북마크(즐겨찾기) 옵션
- 새로고침과 정지 버튼
- 홈페이지로 가게하는 홈버튼

Strangely enough, the browser's user interface is not specified in any formal specification, it just comes from good practices shaped over years of experience and by browsers imitating each other. The HTML5 specification doesn't define UI elements a browser must have, but lists some common elements. Among those are the address bar, status bar and tool bar. There are, of course, features unique to a specific browser like Firefox's downloads manager.

이상하게도, 브라우저들의 유저 인터페이스는 어떠한 정해진 방법대로 규정되있지 않다, 단지 그냥 더 좋으면 그것을 적용하고 나머지들이 따라가는 방식이다. HTML5 규정은 브라우저가 반드시 갖춰야할 UI 엘리먼트들을 정의하지 않는다, 다만 그냥 공통된 요소들을 나열할 뿐이다. 주소창, 상태창, 툴바와 같은 것들을. 물론, 파이어폭스의 다운로드 매니저와 같은 특이한 기능도 있다.

### The browser's high level structure

The browser's main components are (1.1):

1 .The user interface: this includes the address bar, back/forward button, bookmarking menu, etc. Every part of the browser display except the window where you see the requested page.

2. The browser engine: marshals actions between the UI and the rendering engine.
3. The rendering engine : responsible for displaying requested content. For example if the requested content is HTML, the rendering engine parses HTML and CSS, and displays the parsed content on the screen.
4. Networking: for network calls such as HTTP requests, using different implementations for different platform behind a platform-independent interface.
5. UI backend: used for drawing basic widgets like combo boxes and windows. This backend exposes a generic interface that is not platform specific. Underneath it uses operating system user interface methods.
6. JavaScript interpreter. Used to parse and execute JavaScript code.
7. Data storage. This is a persistence layer. The browser may need to save all sorts of data locally, such as cookies. Browsers also support storage mechanisms such as localStorage, IndexedDB, WebSQL and FileSystem.

### 브라우저의 고수준 구조

브라우저의 주된 컴포넌트들은 다음과 같다 :

1. UI: 주소창, 뒤로가기 앞으로가기 버튼, 북마크 메뉴, 등등 을 포함한다. 당신이 보고있는 창에서 나타나는 모든 파트들이다.
2. 브라우저 엔진: UI와 렌더링 엔진 사이에서 액션들을 규제한다.
3. 렌더링 엔진: 요청한 컨텐트들을 보여주는 역할을 한다. 예를들면 요청된 컨텐트가 HTML 이라면, 렌더링 엔진이 HTML과 CSS를 파싱한다, 그리고 파싱된 컨텐트들을 스크린에 보여준다.
4. 네트워킹: HTTP 요청과 같은 네트워크 요청을 위해서, 다른 플랫폼에 의존하는 인터페이스들을 위해 다른 실행을 한다.
5. UI 백엔드: 기초적인 위젯(콤보박스와 창 같은)들을 그리는데에 사용된다. 이 백엔드는 플랫폼에 따라 다르지 않은 공통적인 인터페이스들을 보여준다.
6. 자바스크립트 인터프리터: 자바스크립트 코드를 파싱하고 실행하는데 쓰인다.
7. 데이터 스토리지: 영구적인 층이다. 브라우저가 어떤 일련의 데이터들을 지역적으로 저장하는것이 필요할수도있다.(예를들자면 쿠키같은것들). 브라우저는 또한 localStorage, IndexedDB, WebSQL 그리고 FileSystem과 같은 저장 메커니즘을 지원한다.

![High Level Structure of Browser Diagram](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/layers.png)

It is important to note that browsers such as Chrome run multiple instances of the rendering engine: one for each tab. Each tab runs in a separate process.

크롬과 같은 브라우저들이 렌더링 엔진의 여러개의 인스턴스들을 실행 하는것은 주목해볼만 하다: 하나의 탭에서 하나하나씩. 각 탭들은 나누어진 프로세스로 실행된다.
