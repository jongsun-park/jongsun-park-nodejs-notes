# Error Handling

## Error Types & Error Handling

### Error Types

- Technical/Network Error
  - ex: DB Server is down
  - solution: Error page
- Expected Error
  - ex: File can't be read, database operation fails
  - solution: inform user, possibly retry
- Bugs / Logic

  - ex: read user is not exist
  - solution:fix during development

### Working with Errors

- Error Page (500 page)
- Intended Page / Response with error information
- Redirect

## Errors - Some Theory

에러가 발생 하면 (`thow new Error(message)`), 앱은 종료되고 이후 코드는 실행되지 않는다. 동기(`try catch`), 비동기(`then catch`)에 따라 에러를 관리 할 수 있다.

```js
const sum = (a, b) => {
  return a + b;
};
```

```js
const sum = (a, b) => {
  if (a && b) {
    return a + b;
  }
  throw new Error("Invalid Arguments");
};

try {
  console.log(sum(1));
} catch (err) {
  console.log("Error Occured!");
  //   console.log(err)
}
console.log("This works");
```

## Throwing Errors in Code

```js
app.use((req, res, next) => {
  if (!req.session.user) {
    return next();
  }
  User.findById(req.sesstion.user._id)
    .then((user) => {
      if (!user) {
        next(); // avoid undefined user in req
      }
      req.user = user;
      next();
    })
    .catch((err) => {
      // console.log(err)
      throw new Error(err);
    });
});
```

## Returning Error Pages

에러가 발생 했을 때 입력된 값 및 메시지와 함께 기존 페이지로 리다이렉트 할 수 있고(validation error), 에러 페이지 (500)로 이동 할 수 있다.(technical error)

```js
product
  .save()
  .then((result) => {
    res.redirect("/admin/products"); // success
  })
  .catch((err) => {
    return res.status(500).render("admin/edit-project", {
      //   ...
    });
    // res.redirect("/500"); // fail
  });
```

```js
app.get("/500", (req, res, next) => {
  res.render("/500", {
    // ...
  });
});
```

### [HTTP response status error](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

- 300 - redirects
- 400 - client errors (404: Not Found)
- 500 - server errors (502: Bad Gateway)

## Express.js Error Handling Middleware

에러 객체를 next() 메서드에 전달하면, 뒤의 모든 미들웨어는 실행 되지 않고, 에러 핸들링 미들웨어에서 실행된다.

```js
catch(err){
    const error = new Error(err)
    error.httpStatusCode = 500;
    next(error)
}
```

```js
app.use((error, req, res, next) => {
  // res.status(error.httpStatusCode).render()
  res.redirect("/500");
});
```

## Using the Error Handling Middleware Correctly

```js
app.use((req, res, next) => {
  throw new Error(""); // 에러 핸들링 미들웨어에 전달 된다.
  promise
    .then((result) => res.render("", {}))
    .catch((err) => next(new Error(err))); // next() 메서드에 전달해야 아래 코드를 실행 할 수 있다.
});
```
