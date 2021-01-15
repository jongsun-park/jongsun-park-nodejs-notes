# Dynamic Routes & Advanced Models

## Adding the Product ID to the Path

```js
// models/product.js
// product 폼을 입력하고 저장 할 때 새로운 아이디 값을 지정.
// 여기에서는 임의 숫자 + 타이틀
class Product {
  save() {
    this.id =
      Math.floor(Math.random() * 1000).toString() +
      "-" +
      this.title.split(" ").join("-");
    // 생략
  }
}
```

```html
// index.ejs // product-list.ejs
<a href="/products/<%= product.id %>" class="btn"> Details </a>
```

## Extracting Dynamic

`:<name>` 형식으로 경로를 입력을 하면 요청 핸들러의 요청 객체에서 입력된 값을 출력 할 수 있다. (`req.params.name`). 만약 고정된 값이 출력되야 하는 경우, 다이나믹 값을 사용하는 라우터 부터 먼저와야 한다.

```js
// routes/shop.js
router.get("/products/:productId", (req, res, next) => {
  console.log(req.params.productId);
  res.redirect("/");
});
```

```js
// routes/shop.js
router.get("/products/:productId", shopController.getProduct);
```

```js
// controllers/shop.js
exports.getProduct = (req, res, next) => {
  // shopController.getProduct
  console.log(req.params.productId);
  res.redirect("/");
};
```

```js
// static routes vs dynamic routes
router.get("/product/delete", shopController.deleteProduct);
router.get("/product/:productId", shopController.getProduct);
```

## Loading Product Detail Data

`router.get('/shop/:productId', controllers.getProduct )` // 라우터에서 컨트롤러로 ID 전달

`req.params.productId` // 요청 핸들러에서 요청 객체 안의 id 추출

`Product.getById( productId, p => console.log(p))` // Product 클래스 안 전역 메소드를 사용해서 해당 id 조회 후 전달된 콜백 함수 실행

```js
  static getById(id, cb){
    getProductsFromFile(products => {
        cb( products.find( p => p.id === id ))
    })
  }
```

```js
// models/products.
const getProductsFromFile = (cb) => {
  fs.readFile(p, (err, fileContent) => {
    if (err) {
      cb([]);
    } else {
      cb(JSON.parse(fileContent));
    }
  });
};

module.exports = class Product {
  constructor(title, price, imageUrl, description) {}
  // 생략
  static getById(id, cb) {
    getProductsFromFile((products) => {
      const product = products.find((p) => p.id === id);
      cb(product);
    });
  }
};
```

```js
// controllers/shop.js
exports.getProduct = (req, res, next) => {
  const productId = req.params.productId;
  Product.getById(productId, (p) => {
    console.log(p);
  });
  res.redirect("/");
};
```

## Rendering the Product Detail View

```js
// controllers/shop.js
exports.getProduct = (req, res, next) => {
  const productId = req.params.productId;
  Product.getById(productId, (p) => {
    res.render("shop/product-detail", {
      pageTitle: p.title,
      path: "/products",
      product: p,
    });
  });
};
```

```js
// views/shop/product-detail.ejs
<%- include('../includes/head.ejs') %>
    <link rel="stylesheet" href="/css/product.css">
    </head>

    <body>
        <%- include('../includes/navigation.ejs') %>
            <main class="centered">
                <h1><%= product.title %></h1>
                <img src="<%= product.imageUrl %>" alt="<%= product.title %>">
                <h2><%= product.description %></h2>
                <p><%= product.price %></p>
                <form action="/cart" method="post">
                    <button class="btn" type="submit">Cart</button>
                </form>
            </main>
            <%- include('../includes/end.ejs') %>

```

## Passing Data with POST Requests

cart form 에 hidden input을 사용해서 product id를 전달할 수 있다.

해당 form 은 여러 템플릿에서 사용되니, 따로 파일 만들어서 한번에 관리한다.

for loop 안에 있는 경우 두번째 인자를 사용해서 루프 안에서 받은 객체와 템플릿에 전달할 객체를 명시 할 수 있다.

- `<%- includes('../shop/postCart', {product: product} )`
- `{template_product: loop_product}`

```html
<!-- views/includes/product-list.ejs -->
<div class="card__actions">
  <a href="/products/<%= product.id %>" class="btn">Details</a>
  <%- include('./add-to-cart', {product: product}) %>
</div>
```

```html
<!-- views/includes/shop/add-to-cart.ejs -->
<form action="/cart" method="post">
  <button class="btn" type="submit">Cart</button>
  <input type="hidden" name="productId" value="<%= product.id %>" />
</form>
```

```js
// routes/shop.js
router.post("/cart", shopController.postCart);
```

```js
// contollers/shop.js
exports.postCart = (req, res, next) => {
  console.log(req.body.productId);
  res.redirect("/products");
};
```

## Adding a Cart Model

```js
// models/cart.js
const p = path.join(
  path.dirname(process.mainModule.filename),
  "data",
  "cart.json"
);
module.exports = class Cart {
  static addProduct(id, price) {
    // 기존 카트 여부 확인
    fs.readFile(p, (err, fileContent) => {
      // 카트 초기화
      const cart = { products: [], price: 0 };
      if (!err) {
        // cart.json 이 존재 한다면, 파일을 읽어서 cart 변수에 담기
        cart = JSON.parse(fileContent);
      }

      // 기존 카트에 제품이 있는지 확인
      const updatedProductIndex = cart.products.findIndex((p) => p.id === id);
      const updatedProduct = cart.products[updatedProductIndex];

      let updatedProduct;
      // 존재 하지 경우
      if (updatedProductIndex) {
        updatedProduct = { ...existingProduct };
        updatedProduct.qty = updatedProduct.qty + 1;
        cart.products = [...cart.products];
        cart.products[existingProductIndex] = updatedProduct;
        // 존재 하지 않는 경우
      } else {
        updatedProduct = { id: id, qyt: 1 };
        cart.products = [...cart.products, updatedProduct];
      }

      // 전체 금액 업데이트
      // typeof +문자열 : 숫자
      cart.totalPrice = cart.totalPrice + +productPrice;

      // 카트 업데이트
      fs.wrtieFile(p, JSON.stringify(cart), (err) => console.log(err));
    });
  }
};
```

```js
// routes
router.post("/cart", (req, res, next) => {
  const productId = req.body.productId;
  Product.getById(productId, (product, price) => {
    Cart.addProduct(productId, product.price);
  });
  res.redirect("/cart");
});
```

## Using Query Params

`?key=value&key=value`를 사용해서 url 파라미터 외 옵션 값을 경로를 통해 전달 할 수 있다.

매개변수: `/:key` -> `req.params.key`

쿼리: `?key=value` -> `req.query.key`

```js
// localhost:3000/admin/edit-product/<product-id>?edit=true
router.get("/admin/edit-product/:productId", (req, res, next) => {
  const productId = req.params.productId; // <product-id>
  const editmode = req.query.edit; // "true
  if (!editmode) {
    return res.redirect("/");
  }
  res.render("admin/edit-product", {
    pageTitle: "Edit Product",
    path: "/edit-product",
    editmode: editmode,
  });
});
```

## Pre-Populating the Edit Product Page with Data

```js
// /admin/edit-mode/:productId?edit=true
const productId = req.params.productId;
const editmode = req.query.edit;
Product.getById(productId, (product) => {
  res.render("edit-product", { product: product });
});
```

```html
<input
  type="text"
  id="title"
  name="title"
  value="<% if(editmode) { %><%= product.title %><% } %>"
/>
<textarea name="description" id="deacription" row="5">
<% if(editmode) { %><%= product.description %><% } %></textarea
>
<button type="submit">
  <% if(editmode) { %>Edit Product<% }else{ %>Add Product<% } %>
</button>
```

## Linking to the Edit Page

```html
<!-- views/admin/shop.ejs -->
<a href="/admin/edit-product/<%= product.id %>?edit=true" class="btn">Edit</a>
```

## Editing the Product Data

수정 페이지에서 새로운 Product 인스턴스를 만들고, 컨트롤러에서 인스턴스가 아이디를 기존 데이터에 존재하면 교체하고, 없는 경우 데이터에 추가한다.

```html
<!-- edit-product.ejs -->
<input type="hidden" name="productId" value="<%= product.id %>" />
```

```js
// controllers/admin.js
module.exports = (req, res, next) => {
  const { productId, title, price, imageUrl, description } = req.body;
  const updated = new Product(productId, title, imageUrl, description, price); // order matters
  update.save();
  res.redirect("/products");
};
```

```js
// models/product.js
const getProductsFromFile = (cb) => {
  fs.readFile(p, (err, fileContent) => {
    if (err) {
      cb([]);
    } else {
      cb(JSON.parse(fileContent));
    }
  });
};

module.exports = class Product {
  constructor(id, title, imageUrl, description, price) {
    this.id = id;
    this.title = title;
    this.imageUrl = imageUrl;
    this.description = description;
    this.price = price;
  }
  save() {
    if (this.id) {
      // existing product
      getProductsFromFile((products) => {
        const index = products.findIndex((p) => p.id === this.id);
        const updatedProducts = [...products];
        updatedProducts[index] = this;
        fs.writeFile(p, JSON.stringify(updatedProducts), (err) =>
          console.log(err)
        );
      });
    } else {
      // new product
      this.id = randomId(this.title);
      products.push(this);
      fs.writeFile(p, JSON.stringify(products), (err) => console.log(err));
    }
  }
};
```

## Adding the Product-Delete Functionality

```html
<!-- edit-produt.ejs -->
<form action="/admin/delete-product" method="POST">
  <input type="hidden" name="productId" value="<%= product.id %>" />
  <button class="btn" type="submit">Delete</button>
</form>
```

```js
// controller/admin.js
exports.postDeleteProduct = (req, res, next) => {
  const productId = req.body.productId;
  Product.deleteById(productId);
  res.redirect("/admin/products");
};
```

```js
// models/product.js
static deleteById(id) {
    getProductsFromFile((products) => {
      const updatedProducts = products.filter((p) => p.id !== id);
      fs.writeFile(p, JSON.stringify(updatedProducts), (err) => {
        if (!err) {
          // code for remove deleted item in cart
        }
        if (err) console.log(err);
      });
    });
  }
```

## Deleting Cart Items

```js
// controllers/product.js
static deleteById(id){
  getProductsFromFile((products) => {
    const product = products.find(p => p.id === id)
    Cart.deleteById(id, product.price)
  })
}
```

```js
// models/cart.js
module.exports = class Cart {
  static deleteProduct(id, productPrice) {
    fs.readFile(p, (err, fileContent) => {
      if (err) return;
      const updatedCart = { ...JSON.parse(fileContent) }; // str -> obj // copy
      const product = updatedCart.products.find((p) => p.id === id);
      const productQty = product.qty;
      updatedCart.products = updatedCart.products.filter((p) => p.id !== id);
      updatedCart.totalPrice =
        updatedCart.totalPrice - productPrice * productQty;
      fs.writeFile(p, JSON.stringify(updatedCart), (err) => console.log(err));
    });
  }
};
```

## Displaying Cart items on the Cart Page

```js
// controllers/shop.js
exports.getCart = (req, res, next) => {
  Cart.getCart((cartData) => {
    console.log(cartData);
    res.render("shop/cart", {
      path: "/cart",
      pageTitle: "Your Cart",
      cart: cartData,
    });
  });
};
```

```js
// models/cart.js
static getCart(cb) {
  fs.readFile(p, (err, fileContent) => {
    if (err) return null;
    const cartData = JSON.parse(fileContent);
    cb(cartData);
  });
  }
```

```js
// view/shop/cart.ejs
<% if(cart.products){ %>
  <% for(productData of cart.products){ %>
      <li><%= productData.id %>(<%= productData.qty %>)</li>
  <% } %>
<% }else{ %>
  <p>Cart is empty</p>
<% } %>
```

## Deleting Cart items
