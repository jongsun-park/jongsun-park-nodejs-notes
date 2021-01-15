# 17 Advanced Authentication

## Resetting Passwords

```js
// view reset.ejs
<form action="/reset" method="POST">
  <label for="email">Email</label>
  <input type="text" name="email" id="email" />

  <input type="hidden" name="_csrf" value="{%=  csrfToken %}" />
  <button type="submit">Reset Password</button>
</form>
<div class="message">{%=  message %}</div>
```

```js
// routes
router.get("/reset", shopRoutes.getResetPassword);
```

```js
// controllers
const getReset = (req, res, next) => {
  res.render("reset", {
    path: "/reset",
    pageTitle: "Reset Password",
    message: req.flash("error"),
  });
};
```

## Implementing the Token Logic

postReset

1. generate totken ([crypto.randomBytes(size[, callback])](https://nodejs.org/api/crypto.html#crypto_crypto_randombytes_size_callback))
2. update user (token, tokenExpiration)
3. sendEmail

```js
// models/user
const userScheme = new Scheme({
  token: String,
  tokenExpiration: Date,
});
```

```js
const crypto = require("crypto");
const postReset = (req, res, next) => {
  const email = req.body.email;
  crypto.randomBytes(32, (err, buffer) => {
    if (err) {
      console.log(err);
      res.redirect("/reset");
    }
    const token = buffer.toString("hex");

    User.findOne({ email: email }).then((user) => {
      // 해당 하는 email 이 없는 경우
      if (!user) {
        req.flash("error", "Email not found");
        res.redirect("/reset");
      }
      //   유일한 토큰 생성하기
      user.token = token;
      user.tokenExpiration = new Date() + 3600000; // 1H = 60 * 60 * 12
      return user.save();
    });
  });
  then((result) => {
    transporter.sendMail({
      to: req.body.email,
      from: "",
      subject: "Reset password",
      html: `
          <a href="http://localhost:3000/reset/${token}">link</a> to reset password
          `,
    });
  }).catch((err) => console.log(err));
};
```

## Creating the Reset Password Form

```js
// routes
router.get("/reset/:token", shop.getNewPassword);

// controllers
exports.getNewPassword = (req, res, next) => {
  const token = req.params.token;
  //   토큰이 일치하고, 유효 기간이 현재 보다 많은 경우
  User.findOne({ token: token, tokenExpiration: { $gt: Date.now() } })
    .then((user) => {
      req.render("auth/new-password", {
        path: "/new-password",
        pageTitle: "New Password",
        errorMessage: req.flash("error"),
        userId: user._id, // hidden field
        token: token, // hidden field
      });
    })
    .catch((err) => console.log(err));
};
```

## Adding Login to Update the Password

```js
exports.postNewPwwsord = (req, res, next) => {
  const newPassword = req.body.newPassword;
  const userId = req.body.userId;
  const token = req.body.token;
  User.findOne({ _id: userId })
    .then((user) => {
      bcrypt
        .hash(newPassword, 12)
        .then((hashed) => {
          user.password = hashed;
          user.token = undefined;
          user.tokenExpiration = undefined;
          return user.save();
        })
        .then((result) => {
          return res.redirect("/login");
        })
        .catch((err) => console.log(err));
    })
    .catch((err) => console.log(err));
};
```
