# 쿠키 & 세션

[modizlla.org - http 쿠키](https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies)

쿠키의 세가지 목적

- 세션 관리(Session management): 서버에 저장해야 정보 관리(로그인, 장바구니, 게임스코어)
- 개인화(Personalization): 사용자 선호, 테마
- 트래킹(Tracking): 사용자 행동 기록 분석

모든 요청마다 쿠키와 함께 사용자 정보가 전달 되어 성능이 떨어지는 것을 방지 하지 위해 **modern stoage APIs** (웹 스토리지 API (localStorage, sessionStorage), IndexedDB)를 사용할 수 있다.

쿠키 생성

- Response: `Set-Cookie: <cookie-name>=<cookie-value>`
- Request: Cookie 헤더

쿠키의 라이프타임

- 세션 쿠키: 브라우저 종료시 (세션 복원 가능)
- 영속적인 쿠키: Expires 속성, Max-Age 속성

설정

- Secure: https 에서만 전송
- HttpOnly: 자바스크립트의 Document.cookie API에 접근 할 수 없다. 서버에서 전송하기만 한다. (ex. 세션쿠키)
- Scope: Domain, Path 디렉티브
- SameSite: cross-site 요청과 함께 전송되지 않았음을 요구

<hr/>

[tistory - 쿠키, 세션 차이점](https://hahahoho5915.tistory.com/32)

쿠키와 세션이 필요한 이유?

- http request 에서 state을 관리 하기 위해

  - http request: connectless, 요청을 하고 응답을 하면 연결이 종료된다.
  - 클라이언트의 상태를 담기 위해 상태를 로컬PC(쿠키)나 서버(세션)에 저장한다.
  - 요청을 하면 쿠키를 설정하는 응답을 보내고, 브라우저에서 쿠키를 저장하여 쿠키 정보가 담긴 요청을 한다.
  - 쿠키는 브라우저가 종료 되면 없어 지는 세션 쿠키와 일정 시간 까지 계속 존재하는 영속 쿠키로 분류된다.

http 프로토콜 특징

- connectionless 프로토콜 (비연결 지향): 클라이언트의 요청에 서버에서 응답을 한 후 연결은 종료된다.
- stateless 프로토콜 (상태정보 유지 안함): 이전 통신에서 주고 받은 데이터를 서버에서 유지하지않는다.

쿠키 요약

- 웹사이트가 사용하고 있는 서버에서 사용자의 컴퓨터에 저장하는 작은 기록 정보
- ex: 아이디, 비밀번호 자동 입력, 팝업 창 닫기

세션 요약

- 일정 시간 동안 칸은 사용자(브라우저)로부터 들어 오는 일련의 요구를 하나의 상태로 보고, 그 상태를 일정하게 유지시키는 기술 (웹 서버 접속 ~ 웹 브라우저 종료)
- 소멸: 브라우저 종료, 웹 서버에서 삭제
- 각 클라이언트에 고유 Session ID 부여
- 쿠키에 담긴 세션 아이디를 통해 세션 아이디를 발급 하거나 전달 (JSESSIONID)
- ex: 브라우저를 닫기 전까지 로그인 상태 유지

## http 모듈 쿠키 생성 & 쿠키 추출

```js
var http = require("http");
http
  .createServer(function (request, response) {
    var date = new Date();
    date.setDate(date.getDate() + 7);

    //   쿠키 입력
    response.writeHead(200, {
      "Content-Type": "text/html",
      "Set-Cookie": [
        "breakfast = toast;Expires = " + date.toUTCString(),
        "dinner = chicken",
      ],
    });

    // 쿠키 출력
    response.end("<h1>" + request.headers.cookie + "</h1>");
  })
  .listen(3000, function () {});
```

## express-session 미들웨어

세션

- 서버에 정보를 저장하는 기술
- 클라이언트에 세션 식별자 쿠키를 부여하고, 해당 위치에 데이터 저장
- 수명: (기본값) 웹 브라우저가 서버에서 접속 하는 동안
- express-session 미들웨어를 사용해서 서버에 세션 속성을 지정

```js
const session = require("express-session");

app.use(
  session({
    secret: "secret key",
    resave: false, // 세션이 변경되지 않았어도 세션 저장소에 반영할지
    saveUninitialized: true, // 초기화되지 않은 세션을 세션 저장소에 저장할지
    // cookie: {
    //   maxAge: 60 * 1000,
    // },
  })
);

app.use((req, res) => {
  req.session.now = new Date().toUTCString();

  res.send(req.session);
});
```

```js
// req.session
{
    "cookie": {
        "originalMaxAge": null,
        "expires": null,
        "httpOnly": true,
        "path": "/
    },
    "now": "Mon, 21 Mar 2016 08:10:09 GMT"
}
```

메서드

- session.regenerate()
- session.destror()
- session.reload()
- session.save()

<hr/>

## What's a Cookie?

- 브라우저 -> 서버: 요청
- 서버 -> 브라우저: 응답 + SetCookie (in Header)
- 브라우저 -> 서버: 요청 + Cookie

## Adding the Request Driven Login Solution

```js
exports.postLogin = (req, res, next) => {
  req.isLoggedIn = true;
  res.redirect("/");
};
```

- req.isLoggedIn 는 요청이 끝나면 없어 진다.
- 다른 페이지에서는 해당 데이터가 존재하지 않는다.
- 서버는 여러 곳에서 요청을 받을 수 있고, 똑같은 요청에 똑같은 응답을 해야 한다.
- 요청 끼리는 독립적으로 분리되어야 한다.

## Setting a Cookie

```js
// controllers/auth.js
// 로그인 페이지 요쳥
exports.getLogin = (req, res, next) => {
  const isLoggedIn =
    req.get("Cookie").split(";")[0].trim().split("=")[1] === "true";
  res.render("auth/login", {
    isAuthenticated: isLoggedIn,
  });
};

// 로그인 버튼 클릭
exports.postLogin = (req, res, next) => {
  res.setHeader("setCookie", "loggedIn=true");
  res.redirect("/login");
};
```

## What is a Session?

- 브라우저 -> 서버: 요청
- 서버 <-> 메모리 or 데이터베이스 (세션 스토리지)
- 서버 -> 브라우저: 세션ID 발급 + 세션정보출력(개인정보)

## express-session

```js
// app.js
const session = require("express-session");

// app.use(bodyParser.urlencoded({extended: false}))
// app.use(express.static(path.join(__dirname, 'public')))
app.use(
  session({
    secret: "secret key", // for hash
    resave: false, // 요청시 마다 저장할 지
    saveUnintialize: false,
    cookie: {
      //
    },
  })
);

// routes
app.use((req, res, next) => {});
```

## Using the Session Middleware

- express-session 미들웨어를 사용하면, req.session 을 통해 세션에 접근할 수 있다.
- 브라우저에서 서버에 접속할 경우 서버로 부터 세션 id를 발급받아 쿠키에 저장한다.
- 세션 아이디를 가진채 다시 요청을 할 경우 req.session 객체를 통해 저장된 데이터를 출력하거나 가공할 수 있다.

```js
exports.getLogin = (req, res, next) => {
  console.log(req.session.isLoggedIn);
};

exports.postLogin = (req, res, next) => {
  req.session.isLoggedIn = true;
  res.redirect("/");
};
```

## Using MongoDB to Store Sessions

- 세션 스토어가 없는 경우 메모리에 저장한다.
- production의 경우 서버에 과부하가 생길 수 있다.
- 데이터베이스를 세션 스토리지로 사용할 수 있다.
- [connect-mongodb-session](https://www.npmjs.com/package/connect-mongodb-session)

```js
// app.js
const session  = require("express-sesssion");
const MongoDBStore  = require("connect-mongodb-session")(session);

const MONGODB_CONNECT_URL = ""; // ?writeAndRead, 쿼리 부분 지우기
const store = new MongoDBStore({
    url: MONGODB_CONNECT_URL,
    collection: 'session' // db collection 에 사용할 이름
});

app.use(session({
    scret: "",
    cookie: {},
    store: store,
    resave: false,
    saveUninitialized: false
}))

const mongoose.connect(url, {store: store})
```

## Assignment 5

유저 정보를 세션에 담기

```js
// controllers/auth.js
exports.getLogin = (req, res, next) => {
  const isAuthenticated = !!req.session.user;
  res.render("auth/login", {
    pageTitle: "login",
    isAuth: isAuthenticated,
    path: "/login",
  });
};

exports.postLogin = (req, res, next) => {
  const id = "";
  User.findById(id)
    .then((user) => {
      req.session.user = user;
      res.redirect("/");
    })
    .catch((err) => console.log(err));
};

// other pages need to user infomation
exports.getUser = (req, res, next) => {
  const user = req.session.user;
  res.render("user", { user: user });
};
```

## Delete Cookie

`req.session.desctory(callback)`: 해당 세션 삭제하고 callback 실행

```js
exports.postLogout = (req, res, next) => {
  req.session.destroy(() => {
    res.redirect("/");
  });
};
```

## Fix Error / req.session.user hasn't mongoose util methods

Session 에서 제공하는 객체는 일반 객체이다. mongoose에서 제공하는 메서드를 사용하기 위해서는 세션에서 id를 가져와서 mongoose 모델로 다시 변경해준다.

```js
app.use((req, res, next) => {
  if (!req.session.user) {
    return next(); // 아래 코드 실행 X
  }
  User.findById(req.session.user._id)
    .then((user) => {
      req.user = user;
      next();
    })
    .catch((err) => console.log(err));
});
```

## session.save(callback)

세션을 업데이트하고 전달한 callback 함수를 실행한다.

세션이 없데이트 되는 동안 다음 코드가 실행되어 원치 않은 결과를 얻을 수 있다. save() 메서드를 사용해서 코드의 순서대로 실행 시킨다.

```js
req.session.save((err) => {
  console.log(err);
  req.redirect("/");
});
```
