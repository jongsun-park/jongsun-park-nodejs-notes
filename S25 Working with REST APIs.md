# S25 Working with REST APIs

- Planning a REST API
- CRUD Operations & Endpoints
  - headers: {'Content-Type': 'application/json' }
  - server: 내보낼 때 // res.json()
  - server: 가져올 때 // const {} = req.body
  - client: 내보낼 때// body: JSON.stringify({obj})
  - client: 가져올 때// res => res.json()
- Validation // express-validation
- Image Upload // multer
- Authentication // jsonwebtoken

## Fetching Lits of Posts

client

```js
fetch("http://localhost:8080/feed/posts")
  .then((res) => {
    if (res.status !== 200) {
      throw new Error("Failed to fetch posts");
    }
    return res.json();
  })
  .then((resData) =>
    this.setData({
      posts: resData.posts,
      totalPosts: resData.totalPosts,
      postsLoading: false,
    })
  )
  .catch(this.catchError);
```

server

```js
// allow any api call from anywhere
app.use((req, res, next) => {
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader(
    "Access-Control-Allow-Methods",
    "GET, POST, PUT, PATCH, DELETE"
  );
  res.setHeader("Access-Control-Allow-Headers", "Content-Type, Authorization");
  next();
});

app.get("/feed/posts", (req, res, next) => {
  res.status(200).json({
    posts: [
      {
        _id: 1,
        title: "First Post",
        description: "This is the first post!",
        imageUrl: "images/duck.jpg",
        creator: { name: "Max" },
        createdAt: new Date(),
      },
    ],
  });
});
```

## Adding a Create Post Endpoint

client

```js
finishEditHandler = (postData) => {
  this.setState({
    editLoading: true,
  });

  let url = "http:localhost:8080/feed/post";
  let method = "POST";

  if (this.state.editPost) {
    url = "URL";
    method = "GET";
  }
  fetch(url, {
    method,
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.Stringify({
      title: postData.title,
      content: postData.content,
    }),
  })
    .then((res) => res.json())
    .then((data) => this.setData(data))
    .catch(this.errorHandle);
};
```

server

```js
app.post("/feed/post", (req, res, next) => {
  const title = req.body.title;
  const content = req.body.content;
  res.status(201).json({
    message: "Post created successfully!",
    post: {
      _id: new Date().toISOString(),
      title: title,
      content: content,
      creater: { name: "Max" },
      createdAt: new Date().toISOString(),
    },
  });
});
```

## Adding Server Side Validation (유효성 검사)

클라이언트에서 폼을 보내기 전에 validation 을 하고, 서버에서 한번 더 valdiation 을 한다. 이 프로젝트에서는 [express-validation](https://www.npmjs.com/package/express-validation) 모듈 사용

server

```js
const { body, validationResult } = require("express-validation");

app.post(
  "/feed/post",
  [
    body(title).trim().isLength({ min: 5 }),
    body(content).trim().isLength({ min: 5 }),
  ],
  (req, res, next) => {
    // validation error
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(422).json({
        message: "Validation failed, entered data is incorrect.",
        errors: errors.array(),
      });
    }
    // api logic
    const { title, content } = req.body;
    res.status(201).json({
      message: "Post created successfully!",
    });
  }
);
```

## Setting Up a Post Model

app.js: connect mongoose

```js
const mongoose = require("mongoose");

const url = ""; // mongodb + scretkey + db name
mongoose
  .connect(url)
  .then((result) => {
    app.listen(8080);
  })
  .catch((err) => console.log(err));
```

models/post.js: create scheme & export model

자동으로 생성 되는 데이터

- `_id`
- createdAt
- updatedAt

```js
const { Schema } = require("mongoose");

const postSchema = new Schema(
  {
    name: {
      type: String,
      required: true,
    },
    content: {
      type: String,
      required: true,
    },
    imageUrl: {
      type: String,
      required: true,
    },
    creator: {
      type: Object,
      required: true,
    },
  },
  { timestamps: true }
);

module.exports = mongoose.model("Post", postSchema);
```

## Storing Posts in the Database

Post 모델을 Controllers 에 가져와서 mongoose 에서 제공하는 `save()` 메서드로 db에 저장

```js
// controllers
const Post = require("../models/post");

exports.createPost = (req, res, next) => {
  //   validation error handling
  const { title, content } = req.body;

  const post = new Post({
    title,
    content,
    imageUrl: "dummy.jpg",
    create: { name: "max" },
  });

  post
    .save()
    .then((result) => {
      res.status(201).json({
        message: "created successfully!",
        post: result,
      });
    })
    .catch((err) => console.log(err));
};
```

## Static Images & Error Handling

static images

- -> images 폴더에 있는 이미지를 /images 경로에서 접근 가능 하도록 하기

```js
// app.js
const path = require("path");
app.use("/images", express.static(path.join(__dirname, "images")));
```

errror handling

- -> (err) => console.log(err)
- -> 에러가 발생 했을 때 로그로 실행하는 대신, message 와 statusCode 를 입력하고, 미들웨어로 작성된 에러핸들러 에서 관리하기

```js
// controllers
exports.createPost = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    const error = new Error("Validation failed, entered data is incorrect.");
    error.statusCode = 422;
    throw error; // (동기) go to global error handler
  }

  post
    .save()
    .then()
    .catch((err) => {
      if (!err.statusCode) {
        err.statusCode = 500;
      }
      next(err); // (비동기) go to global error handler
    });
};

// app.js
app.use((error, req, res, next) => {
  console.log(error);
  let status = error.statusCode || 500;
  let message = error.message;

  req.status(status).json({
    message: message,
  });
});
```

## Fetching a Single Post

server

- GET /post/:postId

```js
const Post = require("../models/post");

// GET /posts
// DB 에서 모든 post 가져오기
app.get("/posts", (req, res, next) => {
  Post.find()
    .then((posts) => {
      res.status(200).json({
        message: "Fetched posts successfully!",
        posts: post,
      });
    })
    .catch((err) => {
      if (!error.statusCode) {
        error.statusCode = 500;
      }
      next(error); // 비동기
    });
});

// GET /post/:postId
// DB 에서 해당 id의 post 가져와서 클라이언트로 전달하기
app.get("/post/:postId", (req, res, next) => {
  const postId = req.params.postId;
  Post.findById(postId)
    .then((post) => {
      if (!post) {
        const error = new Error("Could not found post.");
        error.statusCode = 404;
        throw error;
      }
      res.status(200).json({
        message: "fetched sucussfully!",
        post: post,
      });
    })
    .catch((err) => {
      if (!error.statusCode) {
        error.statusCode = 500;
      }
      next(error); // 비동기
    });
});
```

client

- url: this.props.match.params.postId // 클래스 컴포넌트에서 url path
- fetch(url).then().catch()

```js
const postId = this.props.match.params.postId;

fetch(`http://localhost:8080/post/${postId})`)
  .then((res) => {
    if (res.status !== 200) {
      throw new Error("Failed to fetch status");
    }
    return res.json();
  })
  .then((resData) =>
    this.setState({
      title: resData.post.title,
      author: resData.post.creator.name,
      image: "http://localhost:8080/" + resData.post.imageUrl,
      date: new Date(resData.post.createdAt).toLocaleDateString("en-US"),
      content: resData.post.content,
    })
  )
  .catch((err) => console.log(err));
```

## Image Names & Windows

윈도우에서 파일명에 날짜 뮨자열(`new Date().toISOString()`) 사용하는 경우 cors 에러가 발생할 수 있다. 날짜 문자열 대신 uuid 라이브러리를 사용해서 고유값을 넣어줄 수 있다.

uuid

- Universally Unique IDentifier
- 범용 고유 식별자
- 버전 1: 타임 스탬프 기준으로 생성
- 버전 4: 랜덤 생성

```js
const { v4: uuidv4 } = require("uuid");
// uuid.v4.uuidv4

const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, "images");
  },
  filename: function (req, file, cb) {
    cb(null, uuidv4());
  },
});
```

```js
exports.createPost = (req, res, next) => {
    ...
    const imageUrl = req.file.path.replace("\\" ,"/");
    ...
}
```

```js
exports.updatePost = (req, res, next) => {
    ...
    imageUrl = req.file.path.replace("\\","/");
}
```

## Uploading Images

server

- malter
- uuiv.v4...

```js
const multer = require("multer");

const fileStorage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "images");
  },
  filename: (req, file, cb) => {
    cb(null, new Date().toISOString() + "-" + file.originalname);
  },
});

const fileFilter = (req, file, cb) => {
  if (
    file.mimetype === "image/png" ||
    file.mimetype === "image/jpg" ||
    file.mimetype === "image/jpeg"
  ) {
    cb(null, true); // accept
  } else {
    cb(null, false); // reject
  }
};

app.use(bodyParser.uelencoded({}));
app.use(multer({ storage: fileStorage, fileFilter }).single("image"));
```

```js
// contollers createPost
if (!req.file) {
  const error = new Error("No image provided");
  error.statusCode = 422;
  throw error;
}

const imageUrl = req.file.path;
```

client

- new FormData()
- FormData.append(key, value)
- body: FormData

```js
// feed.js
const formData = new FormData();
formData.append("title", postData.title);
formData.append("content", postData.content);
formData.append("image", postData.image);

fetch(url, {
  method: method, // 'POST'
  body: formData,
})
  .then((res) => res.json())
  .then((resData) => this.setData(resData))
  .catch((err) => console.log(err));
```

[FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData)

- Web APIS
- 키와 값 쌍으로 이루어진 form 데이터를 쉽게 만들어 준다.
- 주로 XMLHttpRequest.send() 메서드에 사용된다.
- 인코팅 타입이 "multipart/form-data" 인 경우와 구조가 같다.

## Updating Posts

server

- router
- controller
- validation

```js
router.put(
  "/post/:postId",
  [
    body("title").trim().isLength({ min: 5 }),
    body("content").trim().isLength({ min: 5 }),
  ],
  (req, res, next) => {
    const postId = req.params.postId;
    let { title, content, image } = req.body;
    // old image (text)

    if (req.file) {
      imageUrl = req.file.path; // new image (file)
    }

    if (!imageUrl) {
      const error = new Error("No file picked");
      error.statusCode = 422;
      throw error;
    }

    Post.findById(postId)
      .then((post) => {
        if (!post) {
          const error = new Error("Could not find post.");
          error.statusCode = 404;
          throw error;
        }
        if (imageUrl !== post.imageUrl) {
          clearImage(post.imageUrl);
        }

        post.title = title;
        post.imageUrl = imageUrl;
        post.content = content;
        return post.save();
      })
      .then((result) =>
        res.status(200).json({ message: "Post updated", post: result })
      )
      .catch((err) => {
        if (!err.statusCode) {
          err.statusCode = 500;
        }
        next(err);
      });
  }
);

const clearImage = (filePath) => {
  filePath = path.join(__dirname, "..", filePath);
  fs.unlink(filePath, (err) => console.log(err));
};
```

client

```js
// feed.js
loadPosts = (direction) => {
  //
  fetch("http://localhost:8080/feed/posts")
    .then((res) => {
      if (res.status !== 200) {
        throw new Error("Failed to fetch posts.");
      }
      res.json();
    })
    .then((resData) => {
      this.setState({
        posts: resData.posts.map((post) => {
          return {
            ...post,
            imagePath: post.imageUrl, // string, without domain
          };
        }),
        totalPosts: resData.totalItems,
        postsLoading: false,
      });
    })
    .catch((err) => console.log(err));
};

finishEditHandler = (postData) => {
  this.setState({
    editLoading: true,
  });

  const formData = new FormData();
  formData.append("title", postData.title);
  formData.append("content", postData.content);
  formData.append("imageUrl", postData.imageUrl);

  let url = "http://localhost:8080/feed/post";
  let method = "POST";

  if (this.state.editPost) {
    url = `http://localhost:8080/feed/post/${this.state.editPost._id}`;
    method = "PUT";
  }

  fetch(url, {
    method: method,
    body: formData,
  })
    .then()
    .catch();
};
```

## Delete Posts

server

```js
router.delete("/post/:postId", (req, res, next) => {
  const postId = req.params.postId;
  Post.findById(postId)
    .then((post) => {
      if (!post) {
        // error handler
      }
      clearImage(post.imageUrl); // 연결된 이미지 삭제
      return Post.findByIdAndRemove(postId);
    })
    .then((result) => {
      res.status(200).json({ message: "Remove complete" });
    })
    .catch((err) => {
      // error handler
    });
});
```

client

```js
deletePostHandler = (postId) => {
  this.setState({ postLoading: true });
  fetch(`http://localhost:8080/post/${postId}`, { method: "DELETE" })
    .then((res) => {
      res.json();
    })
    .then((resData) => {
      console.log(resData.message);
    })
    .catch();
};
```

## Adding Pagination

client

- posts?page=1

```js
let page = this.state.postPage;
fetch(`http://localhost:8080/feed/posts?page=${page}`).then().catch();
```

server

- page = req.query.page || 1
- skip(page-1)
- limit()

```js
// controller
exports.getPosts = (req, res, next) => {
  const page = req.query.page || 1;
  let perPage = 2;
  let totalItems;
  Post.find()
    .countDocuments()
    .then((count) => {
      totalItems = count;
      return Post.find()
        .skip((page - 1) * perPage)
        .limit(perPage);
    })
    .then((posts) => {
      res.status(200).json({ message: "fetched post", posts: posts });
    })
    .catch((err) => {
      if (!err.statusCode) {
        err.statusCode = 500;
      }
      next(err);
    });
};
```

## ADding a User Model

user

- email: string, required
- name: string, required
- password: string, required
- posts: array, objectId, ref

```js
// models/auth.js
const mongoose,
  { Schema } = require("mongoose");
const userSchema = new Schema({
  email: {
    type: String,
    required: true,
  },
  name: {
    type: String,
    required: true,
  },
  password: {
    type: String,
    required: true,
  },
  posts: [
    {
      type: Schema.Types.ObjectId,
      ref: "Post",
    },
  ],
});
module.exports = mongoose.model("User", userSchema);
```

```js
// app.js
const authRoutes = require("./routes/auth");

app.use("/auth", authRoutes);
```

```js
// routes
const router = require("express").Router();

router.put("/signup", () => {});

module.exports = router;
```

## Signup Validation

email

- email 형식인지
- 새로운 값인지
  password
- min length: 5

```js
const { body, validationResult } = require("express-validation/check");

// router & controller
// path, validation, controller
router.get(
  "/signup",
  [
    body("email")
      .isEmail()
      .withMessage("Please enter a valid email.")
      .custom((value, { req }) => {
        return User.findOne({ email: value }).then((userDoc) => {
          if (userDoc) {
            return Promise.reject("E-mail address already exists!");
          }
        });
      })
      .nomalizeEmail(),
    body("password").trim().islength({ min: 5 }),
    body("name").trim().not().isEmpty(),
  ],
  (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      const error = new Error("Validation failed.");
      error.statusCode = 422;
      erro.data = errors.array();
      throw error;
    }

    const { email, name, password } = req.body;
    // 생략
  }
);
```

```js
// app.js error handling
app.use((error, req, res, next) => {
  console.log(error);
  const status = error.statusCode || 500;
  const message = error.message;
  const data = error.data;
  res.status(status).json({ message, data });
});
```

## Signing Users up

set default value to user status

```js
// models/user
status: {
  type: String,
  default: "new user"
}
```

client

- default state: `{login: false}`
- send signup form data

```js
signupHandler = (event, authData) => {
  event.preventDefault();
  this.setState({ authLoading: true });
  const url = "http://localhost:8080/auth/signup";
  fetch(url, {
    method: "PUT"
    header: {"Content-Type":"application/json" },
    body: JSON.stringify({
      email: authData.signupForm.email.value,
      password: authData.signupForm.password.value,
      name: authData.signupForm.name.value,
    }),
  })
    .then((res) => res.json())
    .then((resData) => {
      this.setState(login, true);
    })
    .catch((err) => console.log(err));
};
```

server

- get email, name, password from client
- passwrod: bcrypt

```js
// controllers
const bcrypt = require("bcryptjs");

const User = require("../models/user");

exports.signup = (req, res, next) => {
  // error handler
  const { email, name, password } = req.body;
  bcrypt
    .hash(password, 12)
    .then((hashedPw) => {
      const user = new User({ email, name, password: hashedPw });
      return user.save();
    })
    .then((result) => {
      res.status(201).json({ message: "User created!!", userId: result._id });
    })
    .catch
    // error handler
    ();
};
```

## How Does Authentication Work?

1. Client -> Server: Login form data (Auth Data)
2. Server: Create Token
3. Server -> Client: Send authentication token
4. Client: Save token to its storage
5. Client -> Server: Request with token
6. Server: Verify token and response

Token

- JSON + Signiture via scret key
- JSON Web Token(JWT)

## Starting with User Login

1. find User with user email
2. compare password with user password

```js
exports.login = (req, res, next) => {
  const { email, password } = req.body;
  let loadedUser;
  User.findOne({ email: email })
    .then((user) => {
      if (!user) {
        const error = new Error("A user with this email could not be found.");
        error.statusCode = 401; // not authenticated
        throw error;
      }
      loadedUser = user;
      return bcrypt.compare(password, user.password); // password, hashed password
    })
    .then((isEqual) => {
      if (!isEqual) {
        const error = new Error("wrong password");
        error.statusCode = 401; // not authenticated
        throw error;
      }

      // send jwt
      res.status(200).json({
        userId: loadedUser._id.toString(),
      });
    })
    .catch();
};
```

## Logging in & Creating JSON Web Tokens

create json web token(jwt)

server

- install jsonwebtoken
- jwt.sign({email, password, exp: "1h"}, "secret key")
- send to client

[jwt](https://jwt.io/)

- 서버에서 시크릿 키를 통해 입력된 이메일과 비밀번호를 암호화 한다.
- 암호화 된 토큰은 클라이언트와 서버가 통신 하는데 사용된다.

```js
const jwp = require("jsonwebtoken");

const token = jwp.sign(
  { email: loadedUser.email, password: loadedUser.password },
  "secretkey",
  { expiresIn: "1h" }
);

res.status(200).json({
  token: token,
  userId: loadedUser._id.toString(),
});
```

client

- POST http://localhost:8080/auth/login
- body {email, password}

```js
fetch("http://localhost:8080/auth/login", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    email: authData.email,
    password: authData.password,
  }),
})
  .then((res) => res.json())
  .then((resData) => {})
  .catch();
```

## Using & Validating the Token

client

- send token via header
- url을 심플하게 유지하고
- 메소드에 상관없이 토큰을 보낼 수 있다.

```js
fetch(url, {
  headers: {
    Authentication: `Bearer ${this.props.token}`,
  },
})
  .then()
  .catch();
```

```js
const jwt = require("jsonwebtoken");

// utils/is-Auth.js
exports.isAuth = (req, res, next) => {
  const authHeader = req.get("Authentication");
  if (!authHeader) {
    const error = new Error("Not authenticated");
    error.statusCode = 401;
    throw error;
  }
  const token = authHeader.split(" ")[1];
  let decodedToken;

  try {
    decodedToken = jwt.verify(token, "secretkey");
  } catch (err) {
    err.statusCode = 500;
    throw err;
  }

  if (!decodedToken) {
    const error = new Error("Not authenticated");
    error.statusCode = 401;
    throw error;
  }

  req.userId = decodedToken.userId;

  next();
};
```

## Adding Auth Middleware to All Routes

```js
// app.js
const isAuth = require("./utils/is-Auth");

app.use(router, isAuth, contollers);
```

## Connecting Posts & Users

models

- post{author: {ref: "USER"}}
- user{posts: [{ref: "POST"}]}

```js
// models/post
const postSchema = new Schema({
  creator: {
    type: Schema.Types.ObjectId
    ref: "User",
    required: true,
  },
});
// models/user
const userSchema = new Schema({
  posts: [{
    type: Schema.Types.ObjectId,
    ref: "Post",
  }]
})
```

```js
//isAuth
req.user = decodedToken.userId;

// createPost
exports.createPost = (req, res, next) => {
  const { title, content, imageUrl } = req.body;

  const post = new Post({
    title,
    content,
    imageUrl,
    creator: req.userId,
  });

  post
    .save()
    .then((result) => {
      return User.findById(req.userId);
    })
    .then((user) => {
      creator = user;
      user.posts.push(post);
      return user.save();
    })
    .then((result) => {
      res.status(201).json({
        message: "Post creatred successfully!",
        post: post,
        creator: { _id: creator._id, name: creator.name },
      });
    });
};
```

## Adding Authorization Checks

Authentication

- 인증
- 클라이언트가 자신이 주장하는 사용자와 같은 사용자 인지를 확인하는 과정

Authorization

- 인가(권한부여)
- 클라이언트가 하고자 하는 작업이 해당 클라이언트에게 허가된 작업인지를 확인

Authentication -> Autorization: 인증을 거친 후 인증된 사용자에 대한 특정한 권한을 부여

server

```js
if (req.userId !== post.creatoer.toString()) {
  const error = new Error("Not authorized");
  error.statusCode = 403;
  throw error;
}
```

## Clearing Post-User Relations

post 를 삭제 했을 때, 연결된 user의 posts 배열의 post id 삭제 하기

```js
exports.deletePost = (req, res, next) => {
  const postId = req.params.postId;

  Post.findById(postId)
    .then((post) => {
      if (!post) {
        // error handler "post not found"
      }
      if (post.creator.toSting() !== req.userId) {
        // error handler "fail to authorize"
      }

      clearImages(post.imageUrl);
      return Post.findByIdAndRemove(postId);
    })
    .then((result) => {
      return User.findById(req.userId);
    })
    .then((user) => {
      user.posts.pull(postId);
      return user.save();
    })
    .then((result) => {
      res.status.json({ message: "Deleted post." });
    })
    .catch((err) => console.log(err));
};
```
