# 16 Sending Emails

[nodemailer](https://nodemailer.com/about/)

[sendgrid](https://sendgrid.com/docs/)

```js
// app.js
```

## install

`npm install --save nodemailer nodemailer-sendgrid-transporter`

[npm nodemailer](https://www.npmjs.com/package/nodemailer)
[npm nodemailer-sendgrid-transport](https://www.npmjs.com/package/nodemailer-sendgrid-transport)

## Use Nodemailer to Send an Email

```js
// controllers/auth.js

const nodemailer = require("nodemailer");
const sendgridTransport = require("nodemailer-sendgrid-transport");

const transporter = nodemailer.createTransporter(
  sendgridTransport({
    auth: {
      api_key: "",
    },
  })
);

exports.postSingup = (req, res, next) => {
  // 생략
  res.redirect("/");
  return transporter
    .sendMail({
      to: ["joe@foo.com", "mike@bar.com"],
      from: "roger@tacos.com",
      subject: "Hi there",
      text: "Awesome sauce",
      html: "<b>Awesome sauce</b>",
    })
    .catch((err) => console.log(err));
};
```
