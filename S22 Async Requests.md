# S22 Understanding Async Requests

## Acync Requests

- 기존: 클라이언트(브라우저)에서 요청을 하면 서버에서 새로운 html 페이지를 만들어서 렌더링 한다.
- 비동기 요청: JSON 형태로 클라이언트와 서버는 데이터를 주고 받으며, 페이지를 새롭게 랜더링 하지 않고 변화를 준다.

## Adding Client Side JS Code

- form 태그를 사용해서 http 요청을 하면 페이지가 리렌더링 된다.
- form 을 submit 하지 않고, 버튼에 이벤트 핸들러를 연결 하여 원하는 데이터를 추출 할 수 있다. (productId, csrf token)

```html
<input type="hidden" name="productId" value="<%= product._id %" /> />
<input type="hidden" name="_csrf" value="<%= csrfToken %" /> />
<button class="btn" type="button" onclick="deleteProduct(this)">DELETE</button>

<script src="/js/admin.js" />
```

```js
const deleteProduct = (button) => {
  const productId = button.parentNode.querySelector(`[name="productId"]`).value;
  const csrf = button.parentNode.querySelector(`[name="_csrf"]`).value;
};
```

## Sending & Handling Background Requests

```js
const deleteProduct = (button) => {
  const productId = button.parentNode.querySelector(`[name="productId"]`).value;
  const csrf = button.parentNode.querySelector(`[name="_csrf"]`).value;
};

fetch(`/admin/product/${productId}`, {
  method: "DELETE",
  headers: {
    "csrf-token": csrf,
  },
})
  .then((result) => console.log(result.json()))
  .catch((err) => console.log(err));
```

```js
// router
router.delete("/product/:productId", isAuth, (req, res, next) => {
  // admin/product/delete/:productId
  const productId = req.params.productId;

  Product.findById(productId)
  .then(product=>{
      if(!product) => {
          return next(new Error('Product not found.'))
      }
      fileHelper.deleteFile(product.imageUrl);
      return Product.deleteOne({_id: productId, userId: req.user._id})
  })
    .then(() => {
      res.status(200).json({message: "Success"});
    })
    .catch((err) => {
      res.status(500).json({message: "Deleting product failed"});
    })
};)
```

## Manipulating the DOM

```js
const productElement = btn.closest("article"); // product wrapper

fetch(`/admin/product/${productId}`, {
  method: "DELETE",
  headers: {
    "csrf-token": csrf,
  },
})
  .then((result) => {
    console.log(result.json());
  })
  .then((data) => {
    console.log(data);
    // productElement.remove(); // modern browser
    productElement.parentNode.removeChild(productElement); // for old browser
  })
  .catch((err) => console.log(err));
```
