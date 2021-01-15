# 12 Working with NoSQL & Yusing Mongo DB

## [Node.js MongoDB Driver API](https://mongodb.github.io/node-mongodb-native/3.6/api/)

## Connect MongoDB

`npm install mongodb dotenv`

- mongodb: 데이터베이스 연결 mongodb.MongoClient.connect()
- dotenv: 데이터베이스 비밀번호를 환경 변수로 분리해서 사용하기

[node.js quickstarter](https://docs.mongodb.com/drivers/node/quick-start)

```js
// .env
MONGO_DB_PASSWORD = "";
MONGO_DB_DBNAME = "";
```

```js
// util/database.js
require("dotenv").config();

const { MongoClient } = require("mongodb");

const password = process.env.MONGO_DB_PASSWORD;
const dbname = process.env.MONGO_DB_DBNAME;

const url = `mongodb+srv://park:${password}@cluster0.7kdkz.mongodb.net/${dbname}?retryWrites=true&w=majority`;

const client = new MongoClient(url, { useUnifiedTopology: true });

async function run(cb) {
  try {
    await client.connect();
    console.log(client);
    cb();
  } catch {
    (err) => console.log(err);
  }
}

module.exports = run;
```

```js
const mongoConnect = require("./util/database.js");

mongoConnect(() => {
  // MongoClient.connect(url).then((client)=>{
  // console.log(client)
  app.liste(3000);
  // }).catch()
});
```

## Creating the Data Connection

데이터베이스 모듈(파일)에 DB를 객체에 담고 내보내는 메서드 작성

```js
// util/database.js
let _db;

exports.mongoConnect = async (cb) => {
  try {
    await client.connect();
    await _db = client.db(dbname);
    cb();
  } catch {
    (err) => throw new Error(err);
  }
};
exports.getDB = () => {
  if (_db) {
    return _db;
  } else {
    throw "No Database";
  }
};
```

```js
const { getDB } = require("../util/database.js");
class Product {
  constructor() {}
  // Create & Update
  save() {}
  // Read
  static findById() {}
  // Delete
}
```

## Creating Products

- db 가져오기: `getDB()`
- collection 가져오기: `.collection('products')`
- document 추가하기: `.insertOne(this)`

```js
// models/product.js
class Product(){
    save(){
        const db = getDB();
        return db.collection('products')
            .insertOne(this)
            .then((result) => console.log(result))
            .catch((err) => console.log(err));
    }
}
```

```js
// controllers/admin.js
exports.postEditProduct = (req, res, next) => {
  const { title, price, description, imageUrl } = req.body;
  const product = new Product(title, price, description, imageUrl);
  product
    .save()
    .then((result) => {
      console.log(result);
      res.redirect("/admin/products");
    })
    .catch((err) => console.log(err));
};
```

## Fetching All Products

[.find(query, options)](https://mongodb.github.io/node-mongodb-native/3.6/api/Collection.html#find): DB로 부터 반복할 수 있는 문서 집합인 커서를 리턴한다.

[.toArray()](https://mongodb.github.io/node-mongodb-native/3.6/api/AggregationCursor.html#toArray): 문서 배열을 리턴한다.

```js
// models
class Product {
  static fetchAll() {
    const db = getDB();
    return db.collection("products").find().toArray();
  }
}
```

```js
// controllers
const getProducts = (req, res, next) => {
  Product.fetchAll()
    .then((products) => {
      res.render("/", { products: products });
    })
    .catch((err) => console.log(err));
};
```

## Fetching a Single Product

.next()

- 커서 안에서 가능한 문서를 추출한다. (단일)
- 문서가 없는 경우 null 을 리턴한다.

```js
// models
class Product {
  static findbyId(productId) {
    const db = getDB();
    db.collection("products")
      .find({ _id: new ObjectID(productId) })
      .next()
      .then((product) => {
        console.log(product);
        return product;
      })
      .catch((err) => console.log(err));
  }
}
```

```js
// controllers
// http://127.0.0.1:3000/products/5fdb51826857da1af8b77502
const getProduct = (req, res, next) => {
  const { productId } = req.body.params;
  Product.findById(productId)
    .then((product) => {
      render("product-detail", { product: product, path: "products" });
    })
    .catch((err) => console.log(err));
};
```

## Edit & Delete

save()

- 아이디가 존재하는 경우 업데이트 하고, 아이다가 없는 경우 새로운 문서를 입력
- 입력 받은 아이디는 문자열 형태
- 문서를 새로 저장하기 위해서는 ObjectID 인스턴스로 생성 해줘야 한다.

`.updateOne({_id: new ObjectID()}, {$set: this})`

- 아이디가 ObjectId 인스턴스인 객체에 this 객체를 엎은 객체를 기존 문서에 덮어 쓴다.

`.insertOne(this)`

- product 인스턴스를 하나 추가한다. (id가 없는 경우 자동 생성)

```js
// models
class Product {
  constructor(title, price, description, imageUrl, id) {
    this.title = title;
    this.price = price;
    this.description = description;
    this.imageUrl = imageUrl;
    this._id = id ? new ObjectId(id) : null;
  }
  save() {
    const db = getDB();
    let dbOp;
    if (_id) {
      // update
      dbOp = db.collection("products").updateOne({ _id: id }, { $set: this });
    } else {
      // create
      dbOp = db.collection("products").insertOne(this);
    }
    return dbOp
      .then((result) => console.log(result))
      .catch((err) => console.log(err));
  }
}
```

```js
// controllers
const postEditProduct = (req, res, next) => {
  const productId = req.params.productId;
  const { title, price, description, imageUrl } = req.body;
  const product = new Product(title, price, description, imageUrl, productId);
  product
    .save()
    .then((result) => {
      console.log(result);
      res.redirect("/products");
    })
    .catch((err) => console.log(err));
};
```

## Deleting Products

```js
class Product {
  static deleteById(productId) {
    const db = getDB();
    return db
      .collection("products")
      .deleteOne({ _id: ObjectId(productId) })
      .then((result) => console.log(result))
      .catch((err) => console.log(err));
  }
}
```

```js
// controllers
const postDelteProduct = (req, res, next) => {
  const productId = req.body;
  Product.deleteById(productId)
    .then((result) => {
      res.redirect("/products");
    })
    .catch((err) => console.log(err));
};
```

## Creating New Users

```js
// models/user
const { ObjectId } = require("mongodb");
const getDB = require("../util/database");

class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
  static findById(id) {
    const db = getDB();
    return db
      .collection("users")
      .findOne({ _id: ObjectId(id) })
      .then((user) => {
        console.log(user);
        return user;
      })
      .catch((err) => console.log(err));
  }
}
```

```js
// app.js req.user = user
app.use((req, res, next) => {
  const userId = "5fe767bff959f849676b6e7b";
  User.findById(userId)
    .then((user) => {
      req.user = user;
      next();
    })
    .catch((err) => console.log(err));
});
```

## Storing the User in our Database

```js
// models/product.js
class Product {
  constructor(title, price, desciprion, imageUrl, id, userId) {
    // 생략
    this.userId = userId;
  }
}
```

```js
// create product
const product = new Product(
  title,
  price,
  description,
  imageUrl,
  null,
  ObjectId(req.user._id)
);
```

## Working on Cart Items & Orders

`.updateOne(query, options)`

- query: `{_id: new ObjectId(id)}`
- options:
  - `{$set: {cart: updateCart}}` // 업데이트할 필드 지정
  - `{$set: this}` // this 객체 덮어 쓰기

```js
// models/user.js
class User {
  constructor(name, email, cart, id) {
    this.name = name;
    this.email = email;
    this.cart = cart;
    this._id = id;
  }

  static addToCart(product) {
    // new item
    const updateCart = { items: [{ productId: product._id, quantity: 1 }] }; // {items: []}
    const db = getDB();
    return db
      .collection("users")
      .updateOne({ _id: new ObjectId(id) }, { $set: { cart: updateCart } });

    // exist item
  }
}
```

## 'Add to Cart' Button

`req.user = user`

- 객체 user 를 req 객체의 속성으로 가져옴
- user 메서드는 전달 되지 않는다.
- `req.user = new User(user.name, user.email, user.cart, user._id)`
- 새로운 user 인스턴스를 만들어서 메서드도 함께 전달 한다.

```js
// app.js
app.use((req, res, next) => {
  const user = User.findById("")
    .then((user) => {
      // req.user = user
      req.user = new User(user.name, user.email, user.cart, user._id);
      next();
    })
    .catch((err) => console.log(err));
});
```

```js
// controllers/shop.js
const postToCart = (req, res, next) => {
  const { productId } = req.body;
  Product.findById(productId)
    .then((product) => {
      return req.user.addToCart(porduct);
    })
    .then((result) => {
      console.log(result);
      res.redirect("/cart");
    })
    .catch((err) => console.log(err));
};
```

```js
// models/user.js
class User {
  constructor(name, email, cart, id) {
    this.name = name;
    this.email = email;
    this.cart = cart;
    this._id = id;
  }

  // req.user.addToCart()
  // 인스턴스에 적용될 메서드
  addToCart(product) {
    const db = getDB();
    const updatedCart = { productId: product._id, quantity: 1 };
    return db
      .collection("users")
      .updateOne(
        { _id: new ObjectId(this._id) },
        { $set: { cart: updatedCart } }
      )
      .then((result) => console.log(result))
      .catch((err) => console.log(err));
  }
}
```

## Storing Multiple Products in the Cart

```js
// models/user.js
class User {
  addToCart(product) {
    const productIndex = cartItems.findIndex((item) => {
      item.productId.toString() === product._id.toString();
    });
    const cartItems = this.cart.items;
    const updatedItems = [...cartItems];

    if (productIndex >= 0) {
      // exist Item
      updatedItems[productIndex].quantity += 1;
    } else {
      // new Item
      updatedItems.push({ productId: product._id, quantity: 1 });
    }
    return req.user.updateOne(
      { _id: this._id },
      { $set: { cart: { items: updatedItems } } }
    );
  }
}
```
