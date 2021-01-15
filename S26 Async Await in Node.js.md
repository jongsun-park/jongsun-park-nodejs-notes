# S26 Understanding Async Await in Node.js

## What is Async Await All About?

Asynchronous Request in a Synchronous Way

- Only by the way it looks, NOT by the way it behaves
- 동기 형식으로 작성하는 비동기 요청 코드

비동기 요청

- callback: nested callback -> callback hell -> difficult to maintain
- promise: `something().then().then().catch()`
- async await: `async function(){const data = await somecode();}`

## Transforming "Then Catch" to "Async Await"

```js
exports.getPosts = async (req, res, next) => {
  const currentPage = req.query.page || 1;
  const perPage = 2;
  try {
    let totalItems = await Post.find().countDocuments(); // await 이후 코드가 실행 된 후 다음줄 코드 실행
    const posts = await Post.find()
      .puplate("creator")
      .skip((currentPage - 1) * perPage)
      .limit(perPage);

    res.status(200).json({
      message: "",
      posts: postsm,
      totalItems: totalItems,
    });
  } catch (err) {
    if (!err.statusCode) {
      error.statusCode = 500;
    }
    next(err);
  }
};
```
