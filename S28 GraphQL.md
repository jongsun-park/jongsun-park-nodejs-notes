# S28 GraphQL

[graphql docs](https://graphql.org/)

[kakao Tech GraphQl 개념 잡기](https://tech.kakao.com/2019/08/01/graphql-basic/)

GraphQL이란?

- sql 목적: 데이터베이스 시스템에 저장된 데이터를 효율적으로 가져오는 것 (backend)
- gql 목적: 웹 클라이언트가 데이터를 서버로 부터 효율적으로 데이터를 가져오는 것 (fontend)

gql 파이프라인

- GraphQL Query
- Query Language Processor
- GraphQl Resolver
  - RDB / NoSQL / memory DB / REST/Soap
- Output (JSON)

REST API와 비교

- REST API
  - 다양한 endpoint (method, url)
  - endpoint 마다 DB SQL 쿼리가 달라짐
  - backend: db library, express
  - client: axios or fetch, react
- GraphQL
  - 단 하나의 endpoint (`/graphql`)
  - gql 스키마의 타입에 따라 SQL 쿼리가 달라짐
  - 한번의 네트워크 호출로 모든 요청을 처리 할 수 있다.
  - backend: db library, GrapgQL server module, express
  - client: GrapgQL client module, react

GraphQL 구조

- query/mutation
  - query (R: 조회)
  - mutation (CUD: 변조)

일반 쿼리

```
{
  human((id: "1000")){
      name
      height
  }
}
```

오페레이션 네임 쿼리

- 쿼리용 함수
- 데이터베이스의 프로시져 개념과 유사
- 클라이언트 프로그래머가 작성하고 관리

```
query HeroNameAndFriends($episode: Episode){
    hero(episode: $episode){
        name
        friends{
            name
        }
    }
}
```

스키마/타입(schema/type)

- 오브젝트 타입: Character
- 핗드: name, appearsIn
- 스칼라 타입: String, ID, Int
- 느낌표: non-nullable
- 대괄호: array

```
type Character{
    name: String!,
    appearsIn: [Episode!]!
}
```

리졸버(resolver)

- gql에서 데이터를 가져오는 과정(리졸버)는 직접 구현해야 한다.
- 이를 통해, 데이터 source의 종류에 상관 없이 구현이 가능해진다.
- 각각의 필드마다 함수가 하나씩 존재 한다. (이 함수가 리졸버)
- 만약 필드가 스칼라 값 (원시 자료형)인 경우 실행이 종료 되고, 사용자가 정의한 타입이라면 해당 타입의 리졸버가 호출된다.

## Understanding the Setup & Writing out First Query

remove

- socket.io
- routes => graphql endpoint

intall

- graphql // find schema from qsl server
- express-graphql // creating server & parsing
- (advance) apollo-server

```js
// graphql/schema.js
const { buildSchema } = require("graphql");

module.exports = buildSchema(`
    type TestData {
        text: String!
        views: Int!
    }

    type RootQuery {
        hello: String!
    }

    schema {
        query: RootQuery
    }
`);
```

```js
// graphql/resolvers.js
module.exports = {
  hello: () => {
    return { text: "Hello World!", views: 1245 };
  },
};
```

```js
// app.js
const express = require("express");
const { graphqlHTTP } = require("express-graphql");

const app = express();

// 미들웨어 마지막에 작성
app.use(
  "/graphql",
  graphqlHTTP({
    schema: require("../graphgl/schema"),
    rootValue: require("../graphgl/resolvers"),
    // graphiql: true,
  })
);

app.listen(4000);
```

POSTMAN Text

- method: POST
- url: `http://localhost:8080/graphql`
- body: `{ query: "{ hello { text views } }" }`
- response: `{"data": { "hello": { "text": "Hello World!", "views": 1245 } } }`

어떤 데이터를 가져 올지는 클라이언트 사이드에서 결정하고, 서버 측에서는 주어진 데이터를 가공해서 응답한다.

## Defining a Mutation Schema

graphql/schema

```js
const { buildSchema } = require("graphql");

module.exports = buildSchema(`
    type Post {
        _id: ID!
        title: String!
        content: String!
        imageUrl: String!
        creator: User!
        createdAt: String!
        updatedAt: String!
    }

    type User {
        id: ID!
        name: String!
        email: String!
        password: String!
        status: String!
        posts: [Post!]!
    }

    input UserInputData {
        email: String!
        name: String!
        password: String!
    }

    type RootMutation {
        createUser(userInput: UserInputData): User!
    }

    schema {
        mutaion: RootMutation
    }
`);
```

input types

- 스칼라 값을 전달 하지 않고, 객체 형태로 전달 할 수 있다.
- 전달된 객체를 정의 할 때는 type 대신 input 키워드를 사용한다.

```
input ReviewInput {
  stars: Int!
  commentary: String
}
```

## Adding Mutation Resolvers & GraphQL

resolvers

- create user
- connect with db models

app.js

- `graphiql: true`
- graphql playground with docs

```js
// graphql/resolvers
const User = require("../models/user");

module.exports = {
  hello: () => {
    return "Hello World";
  },

  createUser: async ({ userInput }, req) => {
    // const { email, name, password } = userInput;

    const existingUser = await User.findOne({ email: userInput.email });
    if (existingUser) {
      const error = new Error("exist user");
      throw error;
    }

    const hashedPw = await bcrypt.hash(userInput.password, 12);
    const user = new User({
      email: userInput.email,
      name: userInput.name,
      password: hashedPw,
    });

    const createUser = await user.save();

    return {
      ...createUser._doc,
      _id: createdUser._id.toString(),
    };
  },
};
```

mutations

- (args, req) => {}
- 함수 내부에서 return 을 하지 않은 경우 비동기 코드는 실행 되지 않는다.
- then 내부에 return 키워드 사용!
- async await 구문 사용 하기

`user._doc`: print all fields exclude db generated one

## 423 Adding input Validation

REST API

- use `express-validation`
- `router.post("/create-user", [body("email").isEmail() .withMessage("")], controller)`
- 미들웨어로 유효성 검사를 하고 메시지를 전달

graphQL

- 하나의 라우트만 존재 하기 때문에 미들웨어를 전달 할 수 있다.
- resolvers 내부 내에서 유효성 검사를 대신 한다.
- lib: [validator](https://www.npmjs.com/package/validator)

```js
// resolvers
const validator = require("validator");

module.exports = {
  createUser: async ({ userInput }, req) => {
    const { email, name, password } = userInput;

    const error = [];
    if (validator.isEmail(email)) {
      error.push({ message: "Email is invalid" });
    }
    if (
      validator.isEmpty(password) ||
      !validator.isLength(password, { mmin: 5 })
    ) {
      error.push({ message: "Password too short!" });
    }

    if (error.length > 0) {
      const error = new Error("invalid user input");
      throw error;
    }
  },
};
```

## Handling Errors

resolvers

```js
if (errors.lenght > 0) {
  const error = new Error("Invalida input.");
  error.data = errors;
  error.code = 422;
  throw error;
}
```

app.js

```js
app.use(
  "/graphql",
  graphqlHttp({
    schema: graphqlSchema,
    rootValue: graphqlResolver,
    graphiql: true,
    formatError(err) {
      if (!err.originalError) {
        return err; // 사용자 속성이 없는 경우: default
      }
      const data = err.originalError.data;
      const message = err.message || "An error occurred.";
      const code = err.originalError.code || 500;
      return { message, status: code, data };
    },
  })
);
```

## Connecting the Frontend to the GraphQL API

```js
const gqlQuery = {
  query: `
  mutation: {
      createUser(userInput: {email: "${authData.signupForm.email.value}", name:"${authData.signupForm.name.value}", password: "${authData.signupForm.password.value}"}){
          _id
          name
      }
  }
  `,
};

fetch("http://localhost:8080/graghql", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify(gqlQuery),
})
  .then((res) => res.json())
  .then((resData) => {
    if (resData.errors && resData.errors[0].status === 422) {
      throw new Error(
        "Validation failed. Make sure the email address isn't used yet!"
      );
    }

    if (resData.errors) {
      throw new Error("User creation failed!");
    }
    //
  })
  .catch();
```

graphql

- res.method === 'OPTIONS' => error
- graphql: post / get 이 아닌 요청을 모두 거절 한다.
- OPTIONS 일 때, graphql 을 실행 하지 않고 응답을 보내도록 작성한다.

```js
app.use((req, res, next) => {
  if (req.method === "OPTIONS") {
    return res.sendStatus(200);
  }
  next();
});
```

## Adding a Login Query & a Resolver

schema

```js
module.exports = buildSchema({`
    type AuthData{
        token: String!
        userId: String!
    }

    type RootQuery{
        login(email: String!, password: String): AuthData!
    }
`})
```

resolvers

```js
const jwt = require("jsonwebtoken");
const bcrypt = require("bcrypt");

const User = require("../models/user");

module.exports = {
  login: async ({ email, password }) => {
    const user = await User.findOne({ email: email });
    if (!user) {
      const error = new Error("User not found.");
      error.code = 401;
      throw error;
    }

    const isEqual = await bcrypt.compare(password, user.password);
    if (!isEqual) {
      const error = new Error("Password is incorrect");
      error.code = 401;
      throw error;
    }

    const token = await jwt.sign(
      { userId: user._id.toString(), email: user.email },
      "superscretkey",
      { expiresIn: "1h" }
    );

    return {
      token: token,
      userId: user._id.toString(),
    };
  },
};
```

## Adding Login Functionality

client

```js
const query = {
  query: `{
  login(email: "${authData.email}, password: "${authData.password}"){
    token
    userId
  }
}`,
};

fetch("http://localhost:8080/graphql", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify(query),
})
  .then((res) => res.json())
  .then((resData) => {
    const { userId, token } = resData;
    if (resData.errors && resData.errors[0].status === 422) {
      throw new Error("Validation failed");
    }
    if (resData.errors) {
      throw new Error("User login failed");
    }
    this.setState({});
  });
```

## Adding a Create Post Mutation

server / schema

```
type Post{
    id: ID!
    title: String!
    content: String!
    imageUrl: String!
    creator: User!
    createdAt: String!
    updatedAt: String!
}

input PostInputData{
    title: String!
    content: String!
    imageUrl: String!
}

type RootMutation{
    createPost(postInput: PostInputData!): Post!
}
```

server / resolvers

```js
module.exports = {
  createPost: async ({ postInput }, req) => {
    const { title, content, imasgeUrl } = postInput;

    const errors = [];
    if (validator.isEmpty(title) || !validator.isLength(title, { min: 5 })) {
      errors.push({ message: "Title is invalide" });
    }

    if (
      validator.isEmpty(content) ||
      !validator.isLength(content, { min: 5 })
    ) {
      errors.push({ message: "Content is invalide" });
    }

    const post = new Post({ title, content, imageUrl });
    const createdPost = await post.save();

    return {
        ...createdPost._doc,
      _id: createdPost._id.toString(),
      createdAt: createdPost.createdAt.toISOString()
      updatedAt: createdPost.updatedAt.toISOString()
    };
  },
};
```

## Extracting User Data from the Auth Token

client: token을 헤더에 담아서 서버에 요청하기

```js
fetch(url, {
  method: method,
  body: formData,
  headers: {
    Authorization: "Bearer " + this.props.token,
  },
});
```

server auth.js

- token 을 가져와서
- decoded 한 다음
- req.user에 userId 담기

```js
const jwt = require("jsonwebtoken");

module.exports = (req, res, next) => {
  const authHeader = req.get("Authorization");
  if (!authHeader) {
    req.isAuth = false;
    return next();
  }

  const token = authHeader.split(" ")[1];
  let decodedToken;
  try {
    decodedToken = jwt.verify(token, "superscretkey");
  } catch (err) {
    req.isAuth = false;
    return next();
  }

  if (!decodedToken) {
    req.isAuth = false;
    return next();
  }

  req.userId = decodedToken.userId;
  req.isAuth = true;
  next();
};
```

app.js

- graphql 라우트를 설정 하기 전에 auth 확인
- token 이 있는 경우 `req.isAuth = true` / 아니면 `false`

```js
app.use(auth);
app.use("/graphql", graphqlHttp({}));
```

resolvers.js

- authorization 이 필요한 요청의 경우 req.isAuth 를 사용해서 에러 던지기

```js
if (!req.isAuth) {
  const error = new Error("Not authenticated!");
  error.code = 401;
  throw error;
}

const user = await User.findById(req.userId);

if (!user) {
  const error = new Error("Invaled user!");
  error.code = 401;
  throw error;
}

const post = new Post({
  title: postInput.title,
  content: postInput.content,
  imageUrl: postInput.imageUrl,
  creator: user,
});

const createdPost = await post.save();

user.posts.push(createdPost);
await user.save();

return {};
```

## Adding a "Get Post" Query & Resolvers

schemas

```js
type PostData{
  posts: [Post!]!
  totalPosts: Int!
}

rootQuery{
  posts: PostData
}
```

resolvers

```js
module.exports = {
  posts: async () => {
    const totalPosts = await Product.find().countsDocuments();
    const posts = await Product.find()
      .sort({ createdAt: -1 })
      .populate("creater");
    return {
      posts: posts.map((p) => {
        return {
          ...p._doc,
          id: p._id.toString(),
          createdAt: p.createdAt.toISOString(),
          updatedAt: p.updatedAt.toISOString(),
        };
      }),
      totalPosts: totalPosts,
    };
  },
};
```

## Pagination

schema

```js
type PostData {}

posts(page: Int): PostData!
```

resolvers

```js
posts: async ({ post }, req) => {
  const perPage = 2;
  const products = await Product.find()
    .sort({ creatredAt: -1 })
    .skip(page - 1)
    .limit(perPage)
    .populate("creator");
  return {};
};
```

client

```js
const gqlQuery = {
  query: `{
    posts(page: ${page}){
        posts{
          _id
          title
          content
          imageUrl
          creator{
            name
          }
          createdAt
        }
        totalPosts
  }
`,
};
```

## Uploading Images

graphql

- only json
- solution:

  - REST Api 로 이미지를 서버에 저장하고
  - 이미지의 경로를 다시 graphql에 전달

- image -> app server (multer) -> graphql: imageUrl

app.js

```js
const fileStorage = () => {};
const fileFilter = () => {};

app.user(
  multer({
    storage: fileStorage,
    fileFilter: fileFilter,
  }).single("image")
);

app.use(auth);

app.put("/post-image", (req, res, next) => {
  if (!req.isAuth) {
    throw new Error("Not authenticated!!");
  }

  // post 를 수정 할 때, 기존 이미지를 그대로 사용하는 경우 새로운 파일이 저장되지 않는다.
  if (!req.file) {
    return res.status(200).json({ message: "No file provided!" });
  }
  if (req.body.oldPath) {
    clearImage(req.body.oldPath);
  }
  return res
    .status(201)
    .json({ message: "File stored.", filePath: req.file.path });
});

// 넘겨준 파일 경로에 있는 파일을 삭제
const clearImage = (filePath) => {
  filePath = path.join(__dirname, "..", filePath);
  fs.unlink(filePath, (err) => console.log(err));
};
```

client

```js
finishEditHandler = (postData) => {
  const formData = new FormData();
  formData.append("image", postData.image);
  // 포스트를 수정 하는 경우 oldPaht에 기존 이미지 경로를 저장
  if (this.state.editPost) {
    formData.append("oldPath", this.state.editPost.imagePath);
  }

  fetch("http://localhost:8080/post-image", {
    method: "PUT",
    headers: {
      Authorization: `Bearer ${this.props.token}`,
    },
    body: formData,
  })
    .then((res) => res.json())
    .then((fileResData) => {
      const imageUrl = fileResData.filePath;
      const query = {
        query: `
          mutation: {
            createPost(postInput: {title: "${postData.title}", content: "${postData.content}", imgUrl: "${imageUrl}"}){
              _id
              title
              content
              imageUrl
              creator{
                name
              }
              createdAt
            }
          }
        `,
      };
      return fetch("http://localhost:8080/graphql", {
        method: "POST",
        body: JSON.stringify(query),
        headers: {
          Authorization: `Bearer ${this.props.token}`,
          "Content-Type": "application/json",
        },
      });
    })
    .then((res) => res.json())
    .then((resData) => {});
};
```

## Viewing a Single Post

schema

```js
type RootQuery{
  post(id: ID!): POST!
}
```

resolvers

```js
post: async ({ id }, req) => {
  if (!req.isAuth) {
    const error = new Error("Not authenticated");
    error.code = 401;
    throw error;
  }

  const post = await Post.findById({ _id: id }).populate("creator");

  if (!psot) {
    const error = new Error("No post found!");
    error.code = 404;
    throw error;
  }

  return {
    ...post._doc,
    _id: post._id.toString(),
    createdAt: post.createdAt.toISOString(),
    updatedAt: post.updatedAt.toISOString(),
  };
};
```

client

```js
const postId = this.props.match.params.postId;
const query = {
  query: `{
    post(id: "${postId}"){
      title
      content
      imageUrl
      creator{
        name
      }
      createdAt
    }
  }`,
};

fetch("http://localhost:8080/grapgql", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${this.state.token}`,
  },
  body: JSON.stringify(query),
})
  .then((res) => res.json())
  .then((resData) => {
    if (resData.errors) {
      throw new Error("");
    }
    this.setState({
      title: resData.data.post.title,
      author: resData.data.post.creator.name,
      image: "http://localhost:8080/" + resData.data.post.imageUrl,
      data: new Date(resData.data.post.createdAt).toLocaleDateString("en-US"),
      content: resData.data.post.content,
    });
  })
  .catch();
```

## Updating Posts

schema

```js

RootMutation{
  updatePost(id: ID!, postInput: PostInputData!): Post!
}

```

resolvers

```js
module.exports = {
  updatedPost: async ({ id, postInput }, req) => {
    if (!req.isAuth) {
      // error handler
      // 401, Not authenticated! (not logined)
    }
    const post = await Post.findById(id).populate("creator");
    if (!post) {
      // error hadler
      // 404, Not post found!
    }
    if (post.creator._id.toString() !== req.userId.toString()) {
      // error handler
      // 403, Not authoized! (not allowed to edit)
    }

    // validation for inputs
    post.title = postInput.title;
    post.content = postInput.content;
    if (postInput.imageUrl !== "undefined") {
      post.imageUrl = postInput.imageUrl;
    }
    const updatedPost = await post.save();
    return {
      ...updatedPost._doc,
      _id: updatedPost._id.toString(),
      createdAt: updatedPost.createdAt.toISOString(),
      updatedAt: updatedPost.updatedAt.toISOString(),
    };
  },
};
```

client

```js
let qraphqlQuery = "";

if (this.state.editPost) {
  qraphqlQuery = {
    query: `{
      mutation { updatedPost(id: "${}", postInputs: {title: "${postData.title}", content: "${postData.content}", imageUrl: "${postData.imageUrl}"}){
        //
        }
      }
  }`,
};
}

let resDataField = "createPost";
if (this.state.editPost) {
  resDataField = "updatePost";
}
const post = {
  _id: resData.data[resDataField]._id,
  title: resData.data[resDataField].title,
  content: resData.data[resDataField].content,
  creator: resData.data[resDataField].creator,
  createdAt: resData.data[resDataField].createdAt,
  imageUrl: resData.data[resDataField].imageUrl,
};
```

## Deleting Posts

schemas

```js
RootMutation{
  deletePost(id: ID!): Boolean!
}
```

resolvers

```js
const clearImage = require("./utils/file");

module.exports = {
  deletePost: async ({ id }, req) => {
    if (!req.isAuth) {
      // 401, Not authenticated
    }
    const post = await Post.findById(id);
    if (!post) {
      // 404, Not found
    }
    if (post.creator.toString() !== req.userId.toString()) {
      // 403, Not authorized
    }
    clearImage(post.imageUrl);
    await Post.findByIdAndRemove(id);
    const user = await User.findById(req.userId);
    user.posts.pull(id);
    await user.save();
    return true;
  },
};
```

client

```js
const query = {
  query: `
    mutation {
      deletePost(id: "${postId}")
    }
  `,
};

fetch("http://localhost:8080/grapgql", {
  method: "POST",
  headers: {
    Athorization: `Bearer ${this.props.token}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify(query),
})
  .then((res) => {
    res.json();
  })
  .then((resData) => {
    if (resData.errors) {
      //
    }
  });
```

## Using Variables

query 문 내부에서 동적 변수를 사용하지 않고, 네임 쿼리를 사용해서 쿼리에 이름을 주고 괄호를 사용하여 쿼리문 내부에서 사용한 변수를 할당 할 수 있다. 해당 변수는 query 의 variables 값에 객체로 넣어 준다.

before

```js
const graphqlQuery = {
  query: `
    {
      posts(page: ${page}){
        posts: {
          _id
          title
          content
          //
        }
      }
    }
  `,
};
```

after : 네임 오퍼레이션 쿼리
변수의 타입은 schema 와 일치 시켜줘야 한다. (`String!, Int!...`)

```js
const graphqlQuery = {
  query: `
  query GetFetches($page: Int!)
    {
      posts(page: $page){
        posts: {
          _id
          title
          content
          //
        }
      }
    }
  `,
  variables{
    page: this.state.page
  }
};
```

## Module Summary

### GraphQL Core Concepts

- Stateless, client-independent API
- Higher flexibility than REST APIs offer due to custom query language that is exposed to the client
- Queries (GET), Mutation (POST, PUT, PATCH, DELETE) and Subscriptions can be used to exchange and manage data
- ALL GraphQL requests are directed to ONE endpoint (POST /graphql)
- The server parses the incoming query expression (typically done by third-party packages) and calls the appropriate resolvers
- GraphQL is Not limited to React.js applications!

### GraphQL vs REST

- REST APIs are great for static data requirements (e.g file upload, scenarios where you need the same data all the time)
- GraphQL gives you higher flexibility by exposing a full query language to the client
- Both REST and GraphQL APIs can be implemented with ANY framework and actually even with ANY server-side language
