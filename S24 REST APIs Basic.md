# S24 REST APIs Basic

## Whats are REST APIs and why do we use Them?

frontend 와 backend 의 분리

- 기존: 서버에서 클라이언트로 html 형태로 전달
- 페이지를 랜더링 하는 대신 데이터만 필요한 경우
  - 모바일 App (built with swift / android studio / ...)
  - SPA
  - Service API
- 페이지를 랜더링 하는 html를 반환하는 대신 데이터만 전달하는 API

REST APIs

- Representational State Transfer
- Transfer Data Instead of User Interfaces
- Only the response (and the request data) changes, NOT the general server-size logic! (middleware, validation, router...)

## Accessing Data with REST APIs

서버에서 전달 하는 데이터를 다양한 형태 (웹/모바일/APP) 에서 사용 가능하다.

데이터를 전달하는 형태

- HTML, PLain Test, XML: 데이터를 사용 가능한 형태로 변환 하는 것이 어렵다. (parsing)
- JSON (JavaScript Objesct Notation)
  - `{"title": "Node.js"}`
  - Data
  - No UI Assumptions
  - Machine-readable and concise; Can easily be converted to JavaScript

## Understanding Reouting & HTTP Methods

API Endpoints

- Client 에서 서버에 요성하는 http methods 와 path 포함
- POST /post
- GET /posts
- GET /posts/:postId

Http Methods (Http Verbs)

- GET / POST / PUT / PATCH / DELETE
- GET: get a resource
- POST: post a resources (create / append)
- PUT: Update entire (create / overwrite)
- PATCH: Update parts of
- DELETE: Remove
- 실제로 로직은 개발자에 의해 작성된다. 작성한 API가 다른 개발자에 의해 사용될 수 있으므로, 위의 패턴을 따르는 것이 좋다.

## REST APIs - The Core Principles

**Uniform Interface**

- 누구나 사용하기 쉽게 http method 와 path를 지정해야한다.

**Stateless Interaction**

- API는 client나 서버의 정보를 가지고 있어서는 안된다.
- 어느 경우에서라도 동일한 요청을 동일한 결과를 가지고 와야한다.
- 모든 요청은 분리 되어야 한다.

Cacheable

- Servers may set caching headers to allow the client to cache responses

Client-Server

- Server and client are separated, client it not concerned with persistent data storage

Layered System

- Server may forward requests to other APIs

Code on Demand

- Execuable code may be transferred from server to client

## Creating our REST API Project & Implementing the Route Setup

production

- express
- body-parser

dev

- nodemon

## Sending Requests & Responses and Working with Postman

json(obj)

- Content-Type: application/json
- 입력한 객체를 json 형태로 변환 시켜 준다.

http status

- 200: success
- 201: success to post resourse

```js
// controllers
exports.getPosts = (req, res, next) => {
  res.status(200).json({
    posts: [
      {
        title: "First post",
        content: "This is the first post",
      },
    ],
  });
};

exports.createPost = (req, res, next) => {
  const { title, content } = req.body;
  res.status(201).json({
    message: "Post created successfully!",
    post: {
      id: new Date().toISOString(),
      title,
      content,
    },
  });
};

// routes

// GET /feed/posts
router.get("/posts", feedController.getPosts);

// POST /feed/post
router.post("/post", feedController.createPost);

// app

const feedRoutes = require("./routes/feed");

// app.use(bodyParser.urlencoded()) // x-www-form-urlencoded
app.use(bodyParser.json()); // application/json

app.use("/feed", feedRoutes);
```

`POST http://localhost:8080/feed/post`

```json
// body raw json
{
  "title": "Second Post",
  "content": "This is a second post"
}
```

```json
{
  "message": "Post created successfully!",
  "post": {
    "id": "2021-01-09T22:39:29.600Z",
    "title": "Second Post",
    "content": "This is a second post"
  }
}
```

## REST APIs, Clients & CORS Errors

```js
// codepen.id
// html
<button id="get">Get Post</button>
<button id="post">Create Post</button>

// js
const getBtn = document.getElementById("get");
const postBtn = document.getElementById("post");

getBtn.addEventListener("click", () => {
  fetch("http://127.0.0.1:8080/feed/posts")
    .then((res) => res.json())
    .then((data) => console.log(data))
    .catch((err) => console.log(err));
});

postBtn.addEventListener("click", () => {
  fetch("http://127.0.0.1:8080/feed/posts", {
    method: "POST",
    body: JSON.stringify({
      title: "A Codepen Post",
      content: "Created via Codepen"
    }),
    headers: {
      'Content-Type':'application/json'
    }
  })
  .then((res) => res.json())
  .then((data) => console.log(data))
  .catch((err) => console.log(err));
})
```

```js
// app.js
app.use((req, res, next) => {
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader(
    "Access-Control-Allow-Methods",
    "GET, POST, PUT, PATCH, DELETE"
  );
  res.setHeader("Access-Control-Allow-Headers", "Content-Type, Authorization");
  next();
});
```

CORS

- Cross Origin Resource Share
- 도메인이 다른 경우 데이터를 통신 할 수 없다
- 이 문제를 해결 하기 위해서는 서버 측에 헤더를 수정 해줘야 한다.
