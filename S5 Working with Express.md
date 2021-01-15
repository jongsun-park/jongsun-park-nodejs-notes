# Working with Express.js

## Installing express.js

`npm init`
`npm install --save express `
`npm install --save-dev nodemon`

```js
// core library
const http = require("http");

// 1st party library
const express = require("express");

const app = express();

const server = http.createServer(app);

server.listen(3000);
```

## Adding middleware

express의 가장 큰 장점은 미들웨어를 간편하게 만들 수 있다. Incoming Message 를 최종 응답을 보내기 전까지 여러 로직에 통과 시킬 수 있다.

`app.use((req, res, next)=>{})` 를 사용해서 middleware 을 함수 형태로 전달되고, 세번째 인자로 전달된 next()를 사용하여 다음 미들웨어를 실행 시킨다.

`Application.use( ReqeustHandler )`

`(Request, Response, NextFunc) => {...}`

```js
app.use((req, res, next) => {
  console.log("In the middleware");
  next(); // 다음 미들웨어로 전달
});

app.use((req, res, next) => {
  console.log("In another middleware");
});
```

## How middleware works

헤더를 설정 할 필요 없이 바로 [`res.send()`](https://expressjs.com/en/5x/api.html#res.send)를 통해 응답을 보낼 수 있다. 응답을 보낸 후에는 다른 미들웨어에 연결 할 수 없다.

send() 메서드는 기본적으로 간단한 응답에 대한 기본 옵션을 제공한다. (Content-Type, Content-Length, HEAD, HTTP cache )

```js
app.use((req, res, next) => {
  console.log("In another middleware");
  res.send("Hello From express");
});
```

## Express.js - Looking behind the scenes

express.js 는 오픈 프로젝트로 [깃허브](https://github.com/expressjs/express)에서 소스코드를 찾아 볼 수 있다. http 모듈을 기반으로 입력된 값에 따라 content-type를 자동으로 지정해거나, listn() 메서드의 경우 http 모듈을 가져와 서버를 만든 다음 입력된 포트로 서버를 연다.

```js
// const http = require("http");
// const server = http.createServer(app);
// server.listen(3000);

// 생략

app.listen(3000);
```

```js
// express/lib/application.js
app.listen = function listen() {
  var server = http.createServer(this);
  return server.listen.apply(server, arguments);
};
```

## Routes

`app.use([path,] callback [, callback...])`

`app.use('path', (req, res, next) => {res.send()})`

- path를 포함하는 경로에 전달 받은 미들웨어 실행
- path 문자열로 시작하는 경로
  - '/': 늘 실행되는 경우 가장 먼저 작성하고, 기본 값의 경우 가장 나중에 작성한다.
- 미들웨어 함수: 응답을 보낼 경우 `res.send()` 이후 에는 `next()`를 사용해서 응답을 다시 보내지 않아야 한다.

```js
app.use("/", (req, res, next) => {
  console.log("This always runs");
  next();
});

app.use("/add-product", (req, res, next) => {
  res.send('<h1>The "Add Product" page</h1>');
});

app.use("/", (req, res, next) => {
  res.send("<h1>Hello From express</h1>");
});
```

## Parsing Incoming requests

- `req.body`의 데이터를 사용하기 위해서는 'body-parser' 외부 모듈이 필요하다.
- `app.use(bodyParser.urlencoded({ extended: false }))`
  - 모든 미들웨어에서 실행 되기 위해, 라우터 역할을 하는 미들웨어 앞에 위치해야 한다.
  - form 와 같이 데이터가 입력되는 경우, 요청 몸통 (request body)에 실어 미들웨어에서 사용 가능하도록 한다.
  - 결국 `(req, res, next)=>{ //some logic + next() }` 형식으로 구성된다.
- `res.redirect("/")`: 해당 경로('/')로 리다이렉트 한다.
  - = `res.setHeader("Location", "/");`

```js
app.use(bodyParser.urlencoded({ extended: false }));

app.use("/users", (req, res, next) => {
  res.send(
    `<form action="/create-user" method="POST"><input type="text" name="name"></input><button type="submit">Send</button></form>`
  );
});

app.use("/create-user", (req, res, next) => {
  console.log(req.body);
  res.redirect("/");
});
```

## Limiting Middleware Execution to POST

- `app.use()`: 모든 method 에서 실행
- `app.get()`: get 만
- `app.post()`: post 만
- put / delete / patch: http 요청에서 사용할 수 없음

## Express Router

Router

- `express` 내장 모듈
- route 관련 미들웨어를 따로 분리하여 관리하고, 엔트리 파일에서 미들웨어로 연결 할 수 있다.

```js
const router = require("express").Router();
router.get(path, (req, res, next) => {
  // CODE;
});
module.exports = router;
```

```js
const userRoutes = require("./routes/users");
// userRoutes 자체가 미들웨어 기능을 수행한다.
app.use(userRoutes);
//
```

`app.use('/', ()=>{})`: `/` 로 시작하는 모든 경로

`app.get('/', ()=>{})`: 정확히 `/` 인 경로 (exact match), 라우터 중에서 가장 먼저 나와도 경로가 정확히 일치 해야 연결된 미들웨어가 실행된다.

## 404 page

미들웨어는 위에서 아래로 순서대로 실행된다. 모든 라우트에서 원하는 경로를 찾을 수 없는 경우, 마지막 미들웨어에 404 페이지를 출력하는 미들웨어를 전달 할 수 있다.

`res.status(404).send()`

- = `res.writeHead(404, {"Content-Type":"text/html;utf-8"})`
- `res.setHeader('Content-Type', 'text/html');`
- `res.statusCode = 404;`
- 여러 함수를 바로 이어 사용 가능 하다.
- send() 는 가장 마지막에 와야한다.

```js
app.use(userRoutes);
app.use(shopRoutes);

app.use((req, res, next) => {
  res.status(404).send("<h1>Page not found</h1>");
});
```

## Filtering Paths

같은 paths 에 있는 라우터는 라우터를 가져 올 때, 공통된 부분을 경로로 첫번째 인자에 넣어 라우트 파일 안에서 공통된 부분을 생략 할 수 있다. get / post 는 같은 경로라도 다르게 구분한다. 경로의 마지막 슬래쉬는 자동으로 무시된다.

`app.use('/users', usersRoutes)`

가져온 userRoutes 에서 url에서 첫번째 인자로 받은 경로를 제외하고 나머지 부분을 비교하여 미들웨어를 실행한다.

```js
router.get("/", () => {}); // /users/
router.post("/", () => {}); // /users/
router.post("/create", () => {}); // /users/create
```

## Creating HTML Pages

`res.send( string )`: 문자열, html, json ... 을 전달 할 수 있다.

`res.sendFile( absolute path )`: 절대 경로를 입력해서 html 파일을 그대로 전달한다.

`path.join(__dirname, "../", "views", "shop.html")`: 현재 폴더 까지의 절대 경로를 `__dirname` 변수로 가져오고, view 폴더로 가기 위해 한 단계 상위로 올라 간다음, 원하는 html 파일을 가져온다.

```js
const path = require("path");
// 생략
app.get("/", (req, res, next) => {
  res.sendFile(path.join(__dirname, "../", "views", "shop.html"));
});
```

```js
// app.js - serve 404 page
app.use((req, res, next) => {
  res.status(404).sendFile(path.join(__dirname, "views", "404.html"));
});
```

## rootDir

root directory 를 출력하는 유틸 함수

`path.dirname( string )`: 입력 받은 문자열의 경로를 반환

`process.mainModule`: package.json / main / app.js

`path.dirname(process.mainModule.filename)`: `C:\Users\jongs\OneDrive\바탕 화면\maximilian_nodejs\S5\express_sample`

// console.log(Object.keys(process.mainModule));
// ["id","path","exports","parent","filename","loaded","children","paths"];

```js
const path = require("path");

module.exports = path.dirname(process.mainModule.filename);
```

## Serving File Statically

app.js // `app.use(express.static(path.join(__dirname, "public")));`

- public 폴더 안에 모든 내용을 root 경로로 옮겨, 클라이언트에서 바로 읽을 수 있도록 한다.
- `__dirname/public/css/main.css`
- views/shop.html // `<link rel="stylesheet" href="/css/main.css" />`
