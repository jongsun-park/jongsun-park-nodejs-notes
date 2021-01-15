# 13 Working with Mongoose

## Mongoose

- 객체를 문서 형태로 변환해주는 라이브러리

  - ODM( A Oject-Document Mapping Library)
  - (squelize: ORM / Object Relations Library)

- Core Concepts

  - Schemas & Models: `User, Product`
  - Instances: `const user = new User()`
  - Queries: `User.find()`

- mongoDB: `db.collection('users').insertOne({name: 'Max'})`
- mongoose: `const user = User.create({name: 'Max`})

## Connecting to the MongoDB Server with Mongoose

```js
// app.js
const url = `mongodb+srv://park:${password}@cluster0.7kdkz.mongodb.net/${dbname}?retryWrites=true&w=majority`;
mongoose
  .connect(url, { useNewUrlParser: true, useUnifiedTopology: true })
  .then((result) => {
    app.listen(3000, () => {
      console.log("Server is running at http://127.0.0.1:3000");
    });
  })
  .catch((err) => {
    console.log(err);
  });
```

## Creating the Product Scheme

1. Define Scheme:

```js
const blogSchema = new Schema({
  title: String, // String is shorthand for {type: String}
});
```

2. Creating a model

```js
const Blog = mongoose.model("Blog", blogSchema);
```

3. Instance methods

```js
const newBlog = new Blog({ title: "blog title" });
newBlog.save();
```

```js
// models/product
const { Schema, model } = require("mongoose");
const productSchema = new Schema({
  title: { type: String, required: true },
  price: { type: Number, required: true },
  imageUrl: { type: String, required: true },
  description: { type: String, required: true },
});
```

## Saving Data Through Mongoose

`const instace = new Model({})`: 스케마 모델을 통해 새로운 인스턴스 생성

`Instance.save()`: 인스턴스를 연결된 데이터 베이스에 연결

```js
// models/product
module.export = mongoose.model("Product", productSchema);

// controllers/product.js
const postCreateProduct = (req, res, next) => {
  const { title, price, imageUrl, description } = req.body;
  const product = new Product({ title, price, imageUrl, description });

  product
    .save()
    .then((result) => console.log(result))
    .catch((err) => console.log(err));
};
```

## Fetching All Products & Single Product

[Model.find()](https://mongoosejs.com/docs/api.html#model_Model.find)

- Model.find({}) // find all documents
- Model.find({name: "John", age: {$gte: 18}}).exec() // 이름이 John 나이가 18 이상인
- Model.findById(id).exec()

exec()

- mongoose 는 Promise를 리턴하지 않는다.
- query가 전달되는 경우 exec() 메서드를 사용하거나 callback 함수를 넘겨주어야 한다.
- then() 메서드를 사용 할 수 있다.
- `User.findOne({name: "John"}, function(err, user){...})`
- `User.findOne({name: "John"}).exec(function(err, user){...})`
- `User.findOne({name: "John"}).then()`

```js
Product.find()
  .then((products) => console.log(products))
  .catch();
```

## Update Product

1. findById(id)
2. product.title = req.body.title
3. product.save()

```js
exports.postEditProduct = (req, res, next) => {
  const { productId } = req.body;
  const { title, price, imageUrl, description } = req.body;
  Product.findById(productId)
    .then((product) => {
      product.title = title;
      product.price = price;
      product.imageUrl = imageUrl;
      product.description = description;
      return product.save();
    })
    .then((result) => {
      console.log("UPDATED PRODUCT");
      res.redirect("/products");
    })
    .catch((err) => console.log(err));
};
```

## Deleting Products

- `Model.findByIdAndRemove(id)`
- `Model.findByIdAndRemove({_id: id})`
- `Model.findOneAndRemove(id)`
- `Model.findByIdAndDelete(id)`

```js
exports.postDeleteProduct = (req, res, next) => {
  const prodId = req.body.productId;
  Product.findByIdAndRemove(prodId)
    .then(() => {
      console.log("DESTROYED PRODUCT");
      res.redirect("/admin/products");
    })
    .catch((err) => console.log(err));
};
```

## Adding and Using a User Model

```js
app.use((req, res, next) => {
  User.findOne()
    .then((user) => {
      req.user = user;
      next();
    })
    .catch((err) => console.log(err));
  // next();
});
```

```js
User.findOne()
  .then((user) => {
    if (!user) {
      const user = new User({
        name: "Max",
        email: "test@example.com",
        cart: { items: [] },
      });
      user.save();
    }
  })
  .catch((err) => console.log(err));

app.listen(3000, () => {});
```

## Using Relations in Mongoose

Product.userId == User.cart.items.productId

```js
const ProductSchema = {
  userId: {
    type: SchemaTypes.ObjectId,
    ref: "User",
    required: true,
  },
};

const UserSchema = {
  cart: {
    items: [
      {
        productId: {
          type: SchemaTypes.ObjectId,
          ref: "Product",
          required: true,
        },
        quantity: {},
      },
    ],
  },
};
```

```js
const product = new Product({
  title,
  price,
  imageUrl,
  description,
  proeuctId: req.user, // req.user._id
});
```

## One Important Thing About Fetching Relations

```js
Product.find()
.select('title price -_id')
.populate('userId')
.then(products=>{
  ...
}
.catch(err=>console.log(err)))
```

[`select()`](https://mongoosejs.com/docs/api.html#query_Query-select)

- `Query.prototype.select()`
- 출력하는 데이터를 선택할 수 있다.
- `.select('title price -_id')`: title, price 출력 / \_id 미출력

[`populate()`](https://mongoosejs.com/docs/populate.html)

- ref 로 연결된 테이블을 가져올 수 있다.
- `.populate('userId')`: userId로 연결된 모델 User의 데이터를 가져옴
- `.populate('userId', 'name')`: 모델 User의 userId와 name만 출력

## Working on the Shopping Cart

```js
// models/user.js
userSchema.methods.addToCart = function () {
  // 생략
  this.cart = updatedCart;
  return this.save();
};
// controllers/shop.js
exports.postCart = (req, res, next) => {
  const prodId = req.body.productId;
  Product.findById(prodId)
    .then((product) => {
      return req.user.addToCart(product);
    })
    .then((result) => {
      console.log(result);
      res.redirect("/cart");
    });
};
```

## Loading the Cart

[.execPopulate()](https://mongoosejs.com/docs/api.html#document_Document-execPopulate)

- populate() 된 문서를 Promise 형태로 리턴한다.

```js
exports.getCart = (req, res, next) => {
  req.user
    .populate("cart.items.productId")
    .execPopulate() // return promise
    .then((user) => {
      const products = user.cart.items;
      // console.log(products);
      res.render("shop/cart", {
        path: "/cart",
        pageTitle: "Your Cart",
        products: products,
      });
    })
    .catch((err) => console.log(err));
};
```

```js
[
  {
    _id: 5fea1fa3786d7a2a74b0ff99,
    productId: {
      _id: 5fea1ace87f44313e46ee673,
      title: 'Book',
      price: 123,
      description: '123',
      imageUrl: 'https://picsum.photos/200/300',
      userId: 5fea11fe1bf4d347c8f00eec,
      __v: 0
    },
    quantity: 2
  }
]
```

## Deleting Cart Items

```js
// models/user.js
userSchema.methods.removeFromCart = function (productId) {
  const updatedCartItems = this.cart.items.filter((item) => {
    return item.productId.toString() !== productId.toString();
  });
  this.cart.items = updatedCartItems;
  return this.save();
};
```

## Creating & Getting Orders

```js
// models/order.js
const orderSchema = new Schema({
  products: [
    {
      product: { type: Object, required: true },
      quantity: { type: Number, required: true },
    },
  ],
  user: {
    name: {
      type: String,
      required: true,
    },
    userId: { type: SchemaTypes.ObjectId, required: true, ref: "User" },
  },
});
```

```js
// controllers/shop
exports.postOrder = (req, res, next) => {
  req.user
    .populate("cart.items.productId")
    .execPopulate()
    .then((user) => {
      const products = user.cart.items.map((item) => ({
        product: { ...item.productId._doc },
        quantity: item.quantity,
      }));

      const order = new Order({
        user: {
          name: req.user.name,
          userId: req.user,
        },
        products: products,
      });

      return order.save();
    })
    .then((result) => {
      res.redirect("/orders");
    })
    .catch((err) => console.log(err));
};
```

```js
const products = user.cart.items.map((item) => ({
  product: { ...item.productId._doc },
  quantity: item.quantity,
}));
```

`_doc`

- `{ ...data[i]._doc }` // data
- 해당 속성을 가지고 있는 객체를 반환한다.

## Clear the Cart After Storing an Order

```js
userSchema.methods.clearCart = function () {
  this.cart = { items: [] };
  return this.save();
};
```

```js
exports.postOrder = (req, res, next) => {
  req.user
    .populate("cart.items.productId")
    .execPopulate()
    .then((user) => {
      // 생략
      return order.save();
    })
    .then(() => {
      req.user.clearCart();
    })
    .then(() => {
      res.redirect("/orders");
    })
    .catch((err) => console.log(err));
};

exports.getOrders = (req, res, next) => {};
```

## Getting & Displaying the Orders

```js
exports.getOrders = (req, res, next) => {
  Order.find({ "user.userId": req.user._id })
    .then((orders) => {
      res.render("shop/orders", {
        path: "/orders",
        pageTitle: "Your Order",
        orders: orders,
      });
    })
    .catch((err) => console.log(err));
};
```
