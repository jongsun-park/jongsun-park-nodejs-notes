# Introduction

## What is Node.js

JavaScript: Browser -> DOM -> User Interaction

Node.js: JavaScript on Server, (incl. Browser, local encironment)

Chrome V8 Engine: Complier, JavaScript to Machine Code

C++: Node.js written in C++

C++ -> V8 engine -> Machine Code

## Frist App

```js
// first-app.js
console.log("hello node.js");
const fs = require("fs");
fs.writeFileSync("hello.txt", "hello node.js");
```

`node first-app.js` -> log & wirte file

## the Role & Usage of Node.js

1. Run Server: Create Server & Listen to Incoming Requests
2. Businese logic: Hanle Requests, Validation Input, Connect to Database
3. Responses: Return Responses (Rendered HTML, JSON, ... )

웹 브라우저는 HTML, CSS, JavaScript + a 로 구성되어 있다.
클라이언트 에서 서버로 요청을 하면 서버측에서 데이터베이스와 연동하여 새로운 자원을 클라이언트에 전송한다.

Node.js 는 서버 측에서 서버를 실행하고, 데이터베이스 연동, 비즈니스 로직, 인증 시스템(Authenication, Input Validation) 등을 구현하여 사용자에게 필요한 정보를 제공 한다.

Node.js 는 자바스크립트 런타임이다. 브라우저 뿐만 아니라, 다양한 환경 (서버, utility scripts, build tools) 에서 사용할 수 있다.

Node.js Alternative: Python, Ruby, PHP, ASP.NET ...

# Node.js Basics

## How The Web Works

Request: User/Client (Browser) -> (domain) -> Server (Code: Node.js)

Response: Server -> User/Client

HTTP/HTTPS: HyperText Transfer Protocol (Secure), 서버와 클라이언트에서 정보를 주고 받을 수 있도록 하는 규칙

HTTPS = HTTP + S (secure; Data Encryption)

## Creating Node Server

Core Modules (1 party)

- http: lunch a server, send requests
- https: launch a SSL server
- fs
- path
- os

```js
// app.js
const http = require("http");

const server = http.createServer((req, res) => {
  console.log(req);
});

server.listen(3000);
```

http 내장 모듈을 가져와서 변수에 담고, createServer 메서드를 사용해서 서버 인스턴스를 만든다. 인스턴스의 listen 메서드 서버를 실행 시킨다.

[Event-driven architecture](https://12bme.tistory.com/540)

- 이벤트 기반(주도) 아키텍처
- 분산 비동기 아키텍처 패턴
- 이벤트를 비동기식으로 수신하고 처리하는 고도로 분리된 단일 용도의 이벤트 처리 구성 요소로 구성된다.

## Node.js Lifecycle & Event Loop

1. node app.js
2. start script
3. parse code & register variables & functions
4. event loop (The Node Application)
5. exit: `process.exit()`

Incoming message: 서버가 실행 되는 브라우저에 접속 했을 때, 이벤트 리스너에 전달 된다. (request)

서버를 실행 하면 코드를 읽고 변수와 이벤트 리스너를 등록한다. 등록한 이벤트 리스너가 헤지되지 않는 동안 서버는 계속 대기 상태로 유지 된다.

`RequestListener: (IncomingMessage, ServerResponse) => {}`

## Response

```js
const server = http.createServer((req, res) => {
  console.log(req.url);
  console.log(req.method);
  console.log(req.headers);
});
```

req.headers

```json
{
  "host": "127.0.0.1:3000",
  "connection": "keep-alive",
  "upgrade-insecure-requests": "1",
  "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.67 Safari/537.36",
  "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
  "sec-fetch-site": "none",
  "sec-fetch-mode": "navigate",
  "sec-fetch-user": "?1",
  "sec-fetch-dest": "document",
  "accept-encoding": "gzip, deflate, br",
  "accept-language": "en,ko;q=0.9,ko-KR;q=0.8"
}
```

### [윈도우 10 포트 충돌 나는 경우](https://mainia.tistory.com/6165)

1. 해당 포트를 사용하는 프로세스의 PID 확인: `netstat -ano | findstr :3306`
2. 작업 관리자에서 해당 번호 프로세스를 찾아서 종료 시키기

## Response

```js
const server = http.createServer((req, res) => {
  res.setHeader("Content-Type", "text/html");
  res.write("<html>");
  res.write("<head><title>My Rist Page</title></head>");
  res.write("<body><h1>Hello from my Node.js Server</h1></body>");
  res.write("</html>");
  res.end();
  // res.wirte() X
});
```

## [HTTP headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)

## Routing Requests

url 경로에 따라 다른 화면을 보여 줄 수 있다. input에 문자열을 입력하고 submit 버튼을 누르면 /message 페이지로 이동한다. 입력된 값은 req 객체 (incoming massage) 에서 추출 가능하다. 리턴 키워드를 사용해요, if 블록 이후의 코드가 실행 되지 않도록 한다.

```js
if (req.url === "/") {
  res.write("<html>");
  res.write("<head><title></title></head>");
  res.write(
    `<body><form action="/message" method="POST"><input type="text" name="message"/><button type="submit">Send</button></body>`
  );
  res.write("</html>");
  return res.end();
}
```

## Redirectiing Requests

```js
if (url === "/message" && method === "POST") {
  fs.writeFileSync("message.txt", "DUMMY");
  res.statusCode = 302;
  res.setHeader("Location", "/");
  return res.end();
}
```

[response.setHeader( name, value )](https://nodejs.org/docs/v0.4.0/api/http.html)

- `res.setHeader("Location", "/");`
- [Location](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Location): 리다이렉트 되는 경로

[fs.writeFileSync(file, data[, options])](https://nodejs.org/api/fs.html#fs_fs_writefilesync_file_data_options)

```js
fs.writeFileSync("message.txt", data});
```

[fs.writeFile(file, data[, options], callback)](https://nodejs.org/api/fs.html#fs_fs_writefile_file_data_options_callback)

```js
fs.writeFile("message.txt", data, (err) => {
  if (err) throw err;
  console.log("The file has been saved!");
});
```

## Parsing Request Bodies

```js
const body = [];
req.on("data", (chunk) => {
  console.log(chunk);
  //  <Buffer 6d 65 73 73 61 67 65 3d 68 65 6c 6c 6f 25 32 31>
  body.push(chunk);
});
req.on("end", () => {
  const parsedBody = Buffer.concat(body).toString();
  console.log(parsedBody);
  //  message=hello%21
  const message = parsedBody.split("=")[1];
  console.log(message);
  //  hello%21
  fs.writeFileSync("message.txt", message);
});
```

### stream / chunk / buffer

Stream:

- 데이터가 흘러 가는 것. (Ongoing process)
- EventEmitter을 상속한다 이를 통해 이벤트를 emit 하거나 on을 통해 이벤트 발생을 인식할 수 있다.
- fs 모듈을 Stream 을 상속 받는다.

Chunk

- 데이터 덩어리,
- 파일을 통째로 한번에 읽는 것 보다, 파일 안의 데이터를 덩어리(chunk)로 잘개 쪼개 데이터의 stream에 얹어서 읽어 들이는 방식이 메모리나 속도 측면에서 더 효율적이다.
- trigger data event: `on.('data', ()=>{})`
- Incoming message is done: `on.('end', ()=>{})`

Buffer:

- 데이터의 Stream에서 프로세스를 위해 필요한 데이터를 임시적으로 모으는 메모리 공간
- Stream으로 흘러들어오는 데이터의 덩어리(chunk)들은 Buffer에서 수집되서 Process 되게 된다.

[Anatomy of an HTTP Transaction](https://nodejs.org/en/docs/guides/anatomy-of-an-http-transaction/)

```js
const http = require("http");

http
  .createServer((request, response) => {
    const { headers, method, url } = request;
    let body = [];
    request
      .on("error", (err) => {
        console.error(err);
      })
      .on("data", (chunk) => {
        body.push(chunk);
      })
      .on("end", () => {
        body = Buffer.concat(body).toString();
        // At this point, we have the headers, method, url and body, and can now
        // do whatever we need to in order to respond to this request.
      });
  })
  .listen(8080); // Activates this server, listening on port 8080.
```

## Event Driven Code Execution

코드는 위에서 아래로 읽으면서 실행 된다. (동기, synchronous)

이벤트 리스너의 경우 해당 리스너는 바로 실행 되는 것이 아니라, 해당 이벤트 이름에 등록만 된다.

맨 아랫 줄 까지 코드가 실행 된 후, 이벤트가 발생 하면 연결된 이벤트가 실행된다.

하위 코드가 실행 되지 않게 하기 위해서는 리턴 키워드를 사용해서, 블록을 벗어나게 만든다.

## Blocking and Non-Blocking Code

동기 / Blocking

- `fs.writeFileSync(file, data)`
- 파일을 전부 읽고 나서 다음 코드를 실행 한다.

비동기 / Non-Blocking

- `fs.writeFile(file, data, (err) => {})`
- 완료 여부와 상관없이 다음 코드를 실행한다. 대신 콜백함수를 전달하여, 파일을 모두 읽었을 때 해당 콜백 함수가 실행된다. 에러 객체를 매개변수로 받아, 에러를 출력하거나, 이후 로직을 작성할 수 있다.

```js
// fs.writeFileSync("message.txt", message);
fs.writeFile("message.txt", message, (err) => {
  res.statusCode = 302;
  res.setHeader("Location", "/");
  return res.end();
});
```

## Single Thread, Event Loop & Blocking Code

(Incoming message) 여러 요청이 한번에 들어 올 수 있다.

자바스트립트는 하나의 스레드 (Thread / Process operating system) 를 가진다. 즉 한번에 하나의 일만 수행한다.

이벤트 리스너를 등록 만 하고, 이벤트가 실행된 경우 연결된 리스너를 실행한다.

파일 시스템의 경우, 자원 소모가 큰 다. 프로세스가 멈추는 것을 방지 하기 위해, Worker Pool (Different Threads)에서 수행 한 후 event loop 로 Trigger Callback을 전달 한다.

Event Loop (Handle Event Callbacks / Fast finishing code)

- Timers (Execute setTimeout / setInterval callbacks)
- Pending callbacks: Execute I/O-realted Callbacks that were deffered. (Disk & Netwrok Operations / Blocking Operations)
- Poll (Retrieve new I/O events, excecte their callbacks)
- Check (Execute setImmediate() callback)
- Close Callbacks (Execute all 'close' event callbacks)
  - process.exit: refs == 0 (연결된 리스너가 모두 해제 된 경우)

## Using the Node Modules System

함수 내 로직을 분리해서 별도의 파일로 만들어 가져와서 사용 가능 하다.

내보내기

- `module.exports = routes` // default 로 내보니기
- `module.exports = obj` // 객체 형태로 내보내기
- `module.exports.key = value` // 객체를 분해해서 내보내기
- `exports.key = value` // 단축 문법

```js
const http = require("http");

const routes = require("./37.routes");

const server = http.createServer(routes);

server.listen(3000, () => {
  console.log("Server is running at http://127.0.0.1:3000");
});
```

```js
const routesHandler = (req, res) => {
  // 생략
};

module.exports = routesHandler;

// module.exports = { hanlder: routesHandler, someText: "Some Text" };

// module.exports.hanlder = routesHandler;
// module.exports.someText = "SomeText";

// exports.hanlder = routesHandler;
// exports.someText = "SomeText";
```
