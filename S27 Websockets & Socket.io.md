# S27 Websockets & Socket.io

## What Are Websockets & Why Would You Use Them?

HTTP Protocol

- client 1 -> Server -> client 1
- pull
- 클라이언트에서 서버에서 요청을 하고 서버에서 클라이언트로 응답

Websockets

- server -> client
- client 1 -> server -> client 2
- push, 양방향 실시간 통신
- 클라이언트 요청없이 서버에서 클라이언트로 데이터를 전달
- ex. chat app

## [socket.io](https://socket.io/)

### server

`socket.io` 는 http 위에서 동작한다.
`socket.io` 모듈을 가져와서 server 객체를 인자로 넘겨 준후, 이벤트 리스트너를 연결한다.

```js
// socket.io
mongoose
  .connect(url)
  .then((result) => {
    //  start
    const server = app.listen(8080);
    const io = require("socket.io")(server);
    io.on("connection", () => {
      console.log("client connected");
    });
    //  end
  })
  .catch((err) => console.log(err));
```

### client

```js
// socket.io-client
import openSocket from "socket.io-client";

export default () => {
  useEffect(() => {
      openSocket('http://localhost:8080')
  }, []);
  return (
      //
  )
};
```

클라디언트에 접속 하면 서버에서 작성한 connection 이벤트의 리스너가 실행된다. `conosle.log('client connected')`

## Identifying Realtime Potential

client 1: add post -> server -> client 2: update post

server feed.js 에서 websoket connect 접근

client

- useEffect: 랜더링 후 웹소켓 연결
- addPost: pagination, state update

```js
// client / feed.js
this.setState((prevState) => {
  const updatedPosts = [...prevState.posts];
  if (prevState.postPage === 1) {
    if (prevState.posts.length >= 2) {
      updatedPosts.pop();
    }
    updatedPosts.unshift(post);
  }
  return {
    posts: updatedPosts,
    totalPosts: prevState.totalPosts + 1,
  };
});
```

## Sharing the IO Instance Acroos Files

socket 을 다루는 파일을 따로 작성하여, app.js 으로 가져와 초기화 시키고, io 가 필요한 모든 파일에서 가져오기

socket.js

```js
let io;
module.exports = {
  init: (httpServer) => {
    io = require("socket.io")(httpServer);
    return io;
  },
  getIo: () => {
    if (!io) {
      return new Error(`socket.io doesn't initialize`);
    }
    return io;
  },
};
```

app.js // init socket.io

```js
const server = app.listen(8080);
const io = require("./socket")(server);
io.on("connection", (socket) => {
  console.log("client connected");
});
```

## Synchronizing POST Additions

server / create post

```js
const io = require("../socket");
// io.getIO().emit("posts", { action: "create", post: post });
io.getIO().emit("posts", {
  action: "create",
  post: { ...post._doc, creator: { _id: req.userId, name: user.name } },
});
```

client / feed - get posts

```js
// useEffect or componeneDidMount
const socket = openSocket("http://localhost:8080");
socket.on("posts", (data) => {
  if (data.action === "create") {
    this.addPost(data.post);
  }
});
```

io.broacast() // 보내는 클라이언트를 제외한 모든 클라이언트에 전달

io.emit() // 모든 클라이언트에 전달

## Updating Posts Ob All Connected Clients

```js
// server / update post
const result = await post.save();
const io = require("../socket");
io.getIo().emit("posts", { action: "update", post: result });
```

```js
// client
updatePost = (post) => {
  const updatedPosts = [...prevState.psots];
  //
  return {
    posts: updatedPosts,
  };
};

socket.on("posts", (data) => {
  if (data.action === "create") {
    this.addPost(data.post);
  } else if (data.action === "updata") {
    this.updatePost(data.post);
  }
});
```

## Sorting Correctly

가장 최신 post 를 가장 위에 위치하도록 하기

```js
const posts = await Post.find()
  .populate("creator")
  .sort({ createAt: -1 })
  .skip((currentPage - 1) * perPage)
  .limit(perPage);
```

## Deleting Posts Across Clients

server

```js
exports.deletePost = async (req, res, next) => {
  //
  io.getIo().emit("posts", { action: "delete", post: postId });
  res.status(200).json({ message: "Deleted post." });
};
```

client

```js
socket.on("posts", (data) => {
  if (data.action === "create") {
    this.addPost(data.post);
  } else if (data.action === "updata") {
    this.updatePost(data.post);
  } else if (data.action === "delete") {
    this.loadPosts();
  }
});
```
