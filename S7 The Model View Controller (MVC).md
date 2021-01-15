# The Model View Controller (MVC)

## What is MVC

어플리케이션을 구성하는 코드를 역할을 맡게 나눠서 작성하고 관리하는 것. 유지보수성을 높이고 불필요한 중복 코드를 제거할 수 있다. M/V/C간 영향은 배제되어야만 한다.

1. Model: 어플리케이션의 모든 정보, 그 정보를 가공하는 컴포넌트
2. View: 데이터 기반 사용자가 볼 수 있는 화면
3. Controller: 데이터와 사용자 인터페이스 요소를 연결, 이벤트를 처리하는 부분

- view -> user -> controller: sees & uses
- controller -> model: manipulates
- model -> view: updates

## Adding Controllers

Reqeust Hanlders(`(req, res, next) => {}`)를 `/controllers/product.js` 로 분리한 후, 라우터 파일에 가져오기. 컨트롤러 파일은 라우터와 매칭 해서 작성 할 수도 있지만, 사용하는 데이터에 따라 분리 할 수도 있다.

```js
// /controllers/product.js
const products = [];
exports.getAddProduct = (req, res, next) => {
  res.render("add-product", {
    pageTitle: "Add Product",
    path: "/admin/add-product",
    formsCSS: true,
    productCSS: true,
    activeAddProduct: true,
  });
};

// /routes/admin.js
// /admin/add-product => GET
router.get("/add-product", productController.getAddProduct);
```

## Adding a Product Model

```js
// models/product.js
// it will be DB, later
const products = [];

module.exports = class Product {
  // const product = new Product(t);
  constructor(t) {
    this.title = t;
  }

  // 인스턴스에 사용할 메서드: product.save()
  save() {
    products.push(this);
  }

  // 클래스에 사용할 메서드: Product.fetchAll() // products array
  static fetchAll() {
    return products;
  }
};
```

```js
// controllers/product.js
const Product = require("../models/product.js");

// const products = [];
Product.fetchAll();

// products.push(res.body.title)
const product = new Product(res.body.title);
product.save();
```

## Storing Data in Files Via the Model

Syntax

- Root Directory: `const rootDir = path.dirname(process.mainModule.filename)`
- product.json: `const p = path.join(rootDir, "data", "product.jsos")`
- 파일 읽기: `fs.readFile(p, (err, fileContent) => {...})`
- 파일 쓰기: `fs.writeFile(p, fileContent, err => {...})`

```js
// models/product.js save()
save() {
    fs.readFile(p, (err, fileContent) => {
      let products = [];
      if (!err) {
        products = JSON.parse(fileContent);
      }
      products.push(this);

      fs.writeFile(p, JSON.stringify(products), (err) => console.log(err));
    });
  }
```

```js
// models/product.js fetchAll()
// 콜백 함수를 전달해서, 요청이 들어 왔을 때 이벤트 리스너가 등록이 되는게 아니라, 실행되어 값을 사용 할 수 있도록 한다.
static fetchAll(cb) {
    fs.readFile(p, (err, fileContent) => {
      if (err) cb([]);
      cb(JSON.parse(fileContent));
    });
  }
```

```js
// controllers/product.js
// products
// - err: []
// - success: [title: <product name>]
exports.getProduct = (req, res, next) => {
  Product.fetchAll((products) => {
    res.render("shop", {
      prods: products,
      pageTitle: "Shop",
      path: "/",
      hasProducts: products.length > 0,
      activeShop: true,
      productCSS: true,
    });
  });
};
```

## Refactoring

```js
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
  constructor(t) {
    this.title = t;
  }

  save() {
    getProductsFromFile((products) => {
      products.push(this);
      fs.writeFile(p, JSON.stringify(products), (err) => console.log(err));
    });
  }

  static fetchAll(cb) {
    getProductsFromFile(cb);
  }
};
```
