# 15 Adding Authentication

[bcrypt](https://github.com/dcodeIO/bcrypt.js)

[CSRF Attacks](https://www.acunetix.com/websitesecurity/csrf-attacks/)

## What is Authentication

shop - open for all visitor

create / edit product - open for only logged in user

모든 페이지를 모든 사용자에게 공개하는 것이 아니라, 데이터의 생성, 가공이 일어나는 페이지, 즉 제품을 추가하거나 수정하는 페이지는 로그인한 유저에게만 보이도록 한다.

## How is Authentication implemented?

- 브라우저 -> 서버: 요청
- 서버: 세션 아이디 발급
- 서버 -> 데이터베이스: 세션 아이디 저장 (+ 추가 사용자 정보)
- 서버 -> 브라우저: 쿠키를 통해 세션 아이디 발급
- 브라우저 -> 서버: 쿠키(세션정보)와 함께 요청
- 서버 -> 브라우저: 세션을 통해 사용자 정보 접근

## Implementing an Authentication Flow

```js
const User = require("../models/user");

exports.postSingup = (req, res, next) => {
  const { email, password, passwordConfirm } = req.body;
  //   if user is existed
  User.findOne({ email: email })
    .then((user) => {
      if (user) {
        res.redirect("/signup");
      }
      return new User({ email, password, cart: { items: [] } }).save();
    })
    .then((result) => {
      console.log(result);
      res.redirect("/");
    })
    .catch((err) => console.log(err));
};
```

## Encryping Passwords

사용자가 입력한 비밀번호는 데이터베이스에 그대로 저장 되어서는 안된다. 비밀번호를 그대로 저장하지 않고 해쉬화 (암호화) 하여 전달한다.

`bcript.hash(string, 12)`

```js
// Auto-gen a salt and hash
// salt: 암호화를 도와주는 문자열
bcrypt.hash("bacon", 8, function (err, hash) {});
```

```js
const bcrypt = require("bcryptjs");

exports.postSingup = (req, res, next) => {
  const { email, password, passwordConfirm } = req.body;
  //   if user is existed
  User.findOne({ email: email })
    .then((user) => {
      if (user) {
        res.redirect("/signup");
      }
      return bcrypt
        .hash(password, 12)
        .then((hashedpassword) => {
          return new User({
            email,
            password: hashedpassword,
            cart: { items: [] },
          }).save();
        })
        .then((result) => {
          console.log(result);
          res.redirect("/");
        });
    })
    .catch((err) => console.log(err));
};
```

## Adding the Signin Functionality

```js
// login 버튼을 눌렀을 때
exports.postlogin = (req, res, next) => {
  const { email, password } = req.body;
  User.findOne({ email: email })
    .then((user) => {
      // not found user(email)
      if (!user) {
        res.redirect("/login");
      }
      // compare(사용자 입력한 비밀번호, DB에 저장된 해쉬화된 비밀번호)
      bcrypt.compare(password, user.password).then((match) => {
        //   로그인 성공
        if (match) {
          // 사용자 정보를 세션에 저장
          req.session.user = user;
          req.session.isLogged = true;
          //   세션을 저장한 후 콜백 실행 (redirect)
          return req.sesstion.save((err) => {
            console.log(err);
            res.redirect("/");
          });
        } else {
          // 로그인 실패
          res.redirect("/login");
        }
      });
    })
    .catch((err) => console.log(err));
};
```

## Product Routes / Middlewares

```js
// middlewares/is-auth.js
const isAuth = (req, res, next) => {
  if (!req.session.isLoggedIn) {
    return res.redirect("/");
  }
  next;
};
```

```js
// routes/shop
const isAuth = require("../middleware/is-auth.js");

router.get("/", isAuth, shopController.getIndex);
```

## CSRF Attacks (Cross-Site Request Forgery)

웹 페이지에 대한 요청을 위조함으로써 정당한 인증 완료 이용자로 위장 혹은 둔갑하여 처리를 실행하는 공격 수법

인증을 한 사용자가 자신이 의도하지 않은 리퀘스트가 서버에 전송되어 공격자가 의도한 결과가 실행된다.

## Using a CSRF Token

csruf

- cookie-parser, session 을 설정한 후 사용한다.
- get을 제외한 모든 요청에 token을 발급하고 전달해야 한다.
- token 값: `req.csrfToken()`
- form name: `_csrf`

```js
// app.js
const csrf = require("csurf");
const csrfProtection = csrf({ cookie: true }

app.get("/form", csrfProtection, (req, res, next)=>{
    res.render('send', {csrfToken: req.csrfToken()})
})
```

```html
<form type="POST" action="">
  <input type="hidden" name="_csrf" value="{{csrfToken}}" />
  <button type="submit">Summit</button>
</form>
```

## res.locals

[`res.locals`](https://expressjs.com/en/api.html#res.locals): 모든 뷰에 해당 로컬 변수를 선언 할 수 있다.

```js
// app.js
app.use((req, res, next) => {
  res.locals.isAuthenticated = req.session.isLoggedIn;
  req.locals.csrfToken = req.csrfToken();
  next();
});

// routes
```

```html
<input type="hidden" name="_csrf" value="{{csrfToken}}" />
```

## Providing User Feedback / connect-flash

connect-flash

- message 를 세션에 전달하고, 전달을 완료하면 세션에서 삭제된다.

```js
// app
const flash = require("connect-flash");

app.use(session());
app.use(flash()); // 미들웨어 연결

// controllers/user.js
const postLogin = (req, res, next) => {
  // 생략
  if (!user) {
    req.flash("error", "Invalid email or password");
    res.redirect("/login");
  }
};

const getLogin = (req, res, next) => {
  res.render("login", { errorMessage: req.flash("error") });
};
```

## Summary

Authentication

- 인증은 임의의 사용자가 모든 웹페이지를 보지 못하게 하는 것이다.
- 인증은 서버 쪽에서 실행되어야하고 세션에 저장되어야 한다.
- controller 미들웨어 앞에 세션 인증을 하여 일부 라우터를 차단할 수 있다.

Security & UX

- 비밀번호는 hash 되어 데이터베이스에 저장되어야 한다. (bcrypt)
- CSRF (Cross-Site Request Forgery): 웹페이지 요청 위조 공격에 대비 해야한다. (csuf)
  - get 이외의 요청에 \_csrf 값을 요구 해야 한다.
  - `_csrf` 값은 req.csrfToken() 으로 생성할 수 있다.
  - `app.use(csrf())`
- redirect() 시 사용자에게 메시지를 전달 할 수 있다. (connect-flash)
  - app.js // `app.use(flash())` // 초기화
  - postRequest // `req.flash('msg', 'Some message')`
  - getRequest // `res.render('view', {msg: req.flash('msg')})`
  - View // `<p>{%= msg %}</p>`
