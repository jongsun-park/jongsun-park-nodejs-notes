# 11 Understanding Sequelize

## Sequilize

- An Object Relation Mapping Library
- without Sequelize
  - js: `const User = {name, age, password}`
  - SQL: `INSERT INTO users VALUES (1, 'Max', 28, 'dsds123'),`
  - DB: users
    | id | name | age | password |
    | -- | ---- | --- | -------- |
    | 1 | 'Max' | 28 | 'dsds123' |
    | 2 | 'John' | 32 | 'popo123' |
- with Sequelize
  - `const user = User.create({name: 'Max', age: 28, password: 'dsds213'})`
  - Models: User, Product
  - Instances: `const user = User.build()`
  - Queries: `User.findAll()`
  - Associations: `User.hadMany(Product)`
  - SQL를 직접 작성해서 데이터베이스를 관리하는 대신, 객체 (클래스, 메소드)를 사용해서 보다 간편하게 관리한다.

## Connecting to the database

```js
// /utils/database.js
const Sequelize = require("sequelize");

// database, username, [password], options
const sequelize = new Sequelize("node-complete", "root", "", {
  dialect: "mysql",
  host: "localhost",
});

module.exports = sequelize;
```

## Defining a Model

```js
const { DataTypes } = require("sequelize");
const sequelize = require("../util/sequelize");

// sequelize.define(modelName, attributes, options)
const Product = sequelize.define("products", {
  id: {
    type: DataTypes.INTEGER,
    allowNull: false,
    primaryKey: true,
    unique: true,
    autoIncrement: true,
  },
  title: {
    type: DataTypes.STRING,
  },
});

module.exports = Product;
```

## Syncing JS Definitions to the Database

`squelize.synce()`

- `public async sync(options: object): Promise`
- 정의된 모든 모델(`sequelize.define()`)을 DB에 연동
- 프로미스 객체 반환

```js
// app.js
const squelize = require("./util/databse.js");

squelize
  .sync()
  .then((result) => {
    console.log(result);
    app.lisnten(3000);
  })
  .catch((err) => console.log(err));
```

```SQL
// 자동으로 생성 되는 query
Executing (default): CREATE TABLE IF NOT EXISTS `products` (`id` INTEGER NOT NULL auto_increment UNIQUE , `title` VARCHAR(255), `price` DOUBLE PRECISION
NOT NULL, `description` VARCHAR(255) NOT NULL, `imageUrl` VARCHAR(255) NOT NULL, `createdAt` DATETIME NOT NULL, `updatedAt` DATETIME NOT NULL, PRIMARY KEY (`id`)) ENGINE=InnoDB;
Executing (default): SHOW INDEX FROM `products`
```

## Inserting Data & Creating a Product

```js
exports.postAddProduct = (req, res, next) => {
  const title = req.body.title;
  const imageUrl = req.body.imageUrl;
  const price = req.body.price;
  const description = req.body.description;

  Product.create({
    title: title,
    imageUrl: imageUrl,
    price: price,
    description: description,
  })
    .then((result) => console.log(result))
    .catch();
};
```

`Product.create( obj ).then().catch()`

- `public static async create(values: object, options: object): Promise<Model>`

## Versiotn updated: findByPk()

Product.findByPk()

- `public static async findByPk(param: number | string | Buffer, options: object): Promise<Model>`

## Retrieving Data & Finding Products

```js
exports.getIndex = (req, res, next) => {
  Product.findAll()
    .then((products) => {
      res.render("shop/index", {
        prods: products,
        pageTitle: "Shop",
        path: "/",
      });
    })
    .catch((err) => console.log(err));
};
```

`Product.findAll().then().catch()`

- `public static async findAll(options: object): Promise<Array<Model>>`

## Getting a Single Produt with the 'where' Condition

```js
Product.findByPk(productId)
  .then((product) => {
    res.render("views", { product: product });
  })
  .catch((err) => console.log(err));
```

```js
Product.findAll({ where: { id: productId } })
  .then((product) => res.render("views", { product: product[0] }))
  .catch((err) => console.log(err));
```

## Updating Products

Model.save()

- `public async save(options: object): Promise<Model>`
- 유효성 검사를 하고, 업데이트 내용이 있는 경우 데이터베이스에 데이터를 업데이트 한다.

then 메서드 안에 프로미스를 한번더 써야 되는 경우, 프로미스 객체를 반환하고, then 메서드를 연달아 사용할 수 있다. 마지막에 나오는 catch 메서드는 1번, 2번 프로미스안에서 에러가 발생 했을 경우 모두 실행 된다.

```js
promise
  .then(() => {
    promise.then().catch();
  })
  .catch();
```

```js
promise
  .then(() => return promise)
  .then()
  .catch();
```

```js
const postUpdateProduct = (req, res, next) => {
  const productId = req.body.productId;
  const { title, price, imageUrl, desciption } = req.body;
  Product.getByPk(productId)
    .then((product) => {
      product.title = title;
      product.price = price;
      product.imageUrl = imageUrl;
      product.desciption = description;
      return product.save();
    })
    .then((result) => {
      console.log(result);
      res.redirect("/admin/products");
    })
    .catch((err) => console.log(err));
};
```

## Deleting Products

```js
exports.postDeleteProduct = (req, res, next) => {
  const prodId = req.body.productId;

  Product.findByPk(prodId)
    .then((product) => {
      return product.destroy();
    })
    .then((result) => {
      console.log("DESTROYED");
      res.redirect("/admin/products");
    })
    .catch((err) => console.log(err));
};
```

```js
Product.destroy({ where: { id: prodId } })
  .then((result) => {
    console.log(result);
    res.redirect("/admin/products");
  })
  .catch((err) => console.log(err));
```

## Creating a User Model

```js
// /models/user.js
const { DataType } = require("sequelize");
const sequelize = require("../util/sequelize");

const User = sequelize.define("user", {
  id: {
    type: DataTypes.INTEGER,
    allowNull: false,
    primaryKey: true,
    autoIncrement: true,
  },
  // ...
});

module.exports = User;
```

## Adding a One-To-Many Relationship (Associations)

[Associations](https://sequelize.org/master/manual/assocs.html)

```js
Product.belongsTo(User, { contraints: true, onDelete: "CASCADE" });
// User.hasMany(Product);

sequelize
  .sync({ force: true })
  // 서버를 실행 할 때 데이터베이스 테이블을 덮어 쓰기
  .then((result) => {
    app.listen(3000, () => {
      console.log("Server is running at http://127.0.0.1:3000");
    });
  })
  .catch((err) => console.log(err));
```

association methods (source model & target model)

- HasOne
  - `User.hasOne(Cart)`
  - One-To-One relationship
  - foreign key in Target Model(B)
- BelongsTo:
  - `Product.belongsTo(User)`
  - One-To-One relationship
  - foreign key in Source Model(A)
- HasMany:
  - `User.hasMany(Product)`
  - One-To-Many relationship
  - foreign key in Target Model(B)
- BelongsToMany:

  - `A.belongsToMany(B, { through: 'C' })`
  - Many-to-Many relationship
  - C: junction table, has foreign key(aId and bId)

Options

- `{onUpdate: '', onDelete: ''}`
- RESTRICT, CASCADE, NO ACTION, SET DEFAULT and SET NULL
- `Product.belongsTo(User, { onDelete: "CASCADE" })`
  - User 가 삭제되면 Product 도 삭제

## Creating & Managing a Dummy User

1. 미들웨어는 서버를 실행 했을 때 event request 에 등록 된다. (실행X, 등록O)
2. sequelize.sync() 가 성공적으로 실행되면, app 에 연결되 미들웨어가 차례로 실행된다.
3. `sequelize.sync().then()`: user가 하나도 없으면 더비 유저를 하나 등록 한다.
4. `app.use()`: 1번 유저를 조회해서 req 객체에 담는다. (`req.user=user`), 앱 어느 곳에서나 user를 조회할 수 있다.

```js
// 미들웨어, squelize.sync() 이후에 실행
app.use((req, res, next) => {
  User.findByPk(1)
    .then((user) => {
      req.user = user;
      next();
    })
    .catch((err) => console.log(err));
});

// user 가 없는 경우 더미 유저 생성
sequelize
  .sync()
  .then(() => {
    User.findByPk(1).then((user) => {
      if (!user) {
        return User.create({ name: "MAX", email: "test@test.com" });
      } else {
        return user;
      }
    });
  })
  .then(() => {
    app.listen(3000, () => {
      console.log("Server is running at http://127.0.0.1:3000");
    });
  })
  .catch((err) => console.log(err));
```

## Using Magic Association Methods

```js
// controllers/admin.js
Product.create({
  title: title,
  imageUrl: imageUrl,
  price: price,
  description: description,
  userId: req.user.id,
});
```

```js
req.use.createProduct({
  title: title,
  imageUrl: imageUrl,
  price: price,
  description: description,
});
```

[Special methods/mixins added to instances](https://sequelize.org/master/manual/assocs.html#special-methods-mixins-added-to-instances)

- 두 모델의 관계에 따라 특정 메소드와 믹스인을 제공한다.
- `Foo.belongsTo(Bar)`
- `fooInstance.createBar()`: Bar 모델 인스턴스를 만들고 fooId 를 자동으로 입력한다.

## One-To-Many & Many-To-Many Relations

Model

- Cart: {id}
- CartItem: {id, quantity}

```js
// app.js
User.hasOne(Cart);
Cart.belongsTo(User);
Cart.belongsToMany(Product, { through: CartItem });
Product.belongsToMany(Cart, { through: CartItem });
```

## Creating & Fetching a Cart

```js
//app.js
sequelize
  .sync({ force: true })
  // .sync()
  .then((result) => {
    return User.findByPk(1);
    // console.log(result);
  })
  .then((user) => {
    if (!user) {
      return User.create({ name: "Max", email: "test@test.com" });
    }
    return user;
  })
  .then((user) => {
    // console.log(user);
    return user.createCart();
  })
  .then((cart) => {
    app.listen(3000, () => {
      console.log("Server is running at http://127.0.0:3000");
    });
  })
  .catch((err) => {
    console.log(err);
  });
```

```js
// contollers/shop.js
exports.getCart = (req, res, next) => {
  req.user
    .getCart()
    .then((cart) => {
      return cart
        .getProducts()
        .then((products) => {
          res.render("shop/cart", {
            path: "/cart",
            pageTitle: "Your Cart",
            products: products,
          });
        })
        .catch((err) => console.log(err));
    })
    .catch((err) => console.log(err));
};
```

## Adding New Products to the Cart

```js
// controller/shop.js
exports.postCart = (req, res, next) => {
  // 제품 목록에서 가져온 제품 번호
  const productId = req.body.productId;
  let fetchedCart;
  let product;
  let newQuantity;
  req.user
    .getCart() // user id 와 일치하는 카트 가져오기
    .then((cart) => {
      fetchedCart = cart;
      return cart.getProducts({ where: { id: productId } });
      // cart 에 있는 해당 제품 가져오기
    })
    .then((products) => {
      if (products.length > 0) {
        // 카드에 해당 제품이 있으면
        // ...
        product = products[0];
      }
      if (product) {
        // 해당 제품의 수량 증가 시키기
      }

      // 제품 모델에서 해당 id의 제품을 가져와서
      // 카트에 제품을 추가 한다.
      // 카트에 바로 추가 하는 것이 아니라
      // 카트와 제품을 연결 하는 카트 아이템 모델에 추가한다.
      // 카트 아이템 모델에는 카트 id 와 제품 id, 그리고 수량 정보를 담고 있다.
      return Product.findByPk(productId)
        .then((product) =>
          fetchedCart.addProduct(product, {
            through: { quantity: newQuantity },
          })
        )
        .catch((err) => console.log(err));
    })
    .then(() => {
      res.redirect("/cart");
    })
    .catch((err) => console.log(err));
};
```

## Adding Existing Products & Retrieving Cart Items

`Cart.belongsToMany(Product, { through: CartItem });`: product -> cartItem -> quantiry

```html
<% products.forEach(p => { %>
<li class="cart__item">
  <h1><%= p.title %></h1>
  <h2>Quantity: <%= p.cartItem.quantity %></h2>
  <form action="/cart-delete-item" method="POST">
    <input type="hidden" value="<%= p.id %>" nameq="productId" />
    <button class="btn danger" type="submit">Delete</button>
  </form>
</li>
<% }) %>
```

```js
// controllers/shop.js
exports.postCart = (req, res, next) => {
  const productId = req.body.productId;
  const fetchedCart;
  let newQantity = 1;
  req.user
    .getCart()
    .then((cart) => {
      fetchedCart = cart;
      return cart.getProducts({ where: { id: prodId } });
    })
    .then((products) => {
      return products.getFindByPk(productId);
    })
    .then((products) => {
      const product;
      // 기존 카트에 제품이 있는 경우
      if (products.length > 0) product = products[0];
      if (product) {
        const oldQuantity = product.cartItem.quantity;
        newQuantity = oldQuantity + 1;
        return product;
      }
      // 기존 카드에 제품이 없는 경우
      return Product.findByPk(productId);
    })
    .then((product) => {
      // 카트에 제품 추가
      return fetchedCart.addProduct({ through: { quantity: newQuantity } });
    })
    // 카트로 리다이렉트
    .then(() => res.redirect("/cart"))
    .catch((err) => console.log(err));
};
```

## Deleting Related Items & Deleting Cart Products

```js
exports.postCartDeleteProduct = (req, res, next) => {
  const productId = req.body.productId;
  req.user
    .getCart()
    .then((cart) => {
      return cart.getProducts({ where: { id: productId } });
    })
    .then((products) => {
      const product = products[0];
      product.cartItem.destroy();
    })
    .then(() => res.redirect("/cart"))
    .catch((err) => console.log(err));
};
```

## Adding an Order Model

```js
Order.belongsTo(User); // 오더는 유저에 연결 된다.
User.hasMany(Order); // 유저는 여러 오더를 가질 수 있다.
Order.belongsToMany(Product, { through: OrderItem }); // 제품은 오더 아이템을 거져 오더와 연결 된다.
```

## Storing Cartitems as Orderitems

Order button

```html
<!-- views/cart.ejs -->
<form action="/create-order" method="POST">
  <button type="submit">Order Now</button>
</form>
```

```js
// routes/shop.js
router.post("/create-order", shopController.postOrder);
```

```js
//  controllers/shop.js
exports.postOrder = (req, res, next) => {
  req.user
    .getCart()
    .then((cart) => {
      return cart.getProducts();
    })
    .then((products) => {
      return req.user
        .createOrder()
        .then((order) => {
          order.addProducts(
            products.map((product) => {
              product.orderItem = { quantity: product.cartItem.quantity };
              return product;
            })
          );
        })
        .catch((err) => console.log(err));
    })
    .then(() => {
      res.redirect("/order");
    })
    .catch((err) => console.log(err));
};
```

## Resetting the Cart & Fetching and Outputting Orders

```js
req.user
  .getCart()
  .then((cart) => cart.setProducts(null);
  .catch((err) => console.log(err));
```

```js
req.user
  // each order has product array
  .getOrder({ include: ["products"] })
  .then((orders) => {
    res.render("/orders", { orders: orders });
  })
  .catch((err) => console.log(err));
```

```html
<!-- orders.ejs -->
<% orders.forEach( order => { %>
<li>
  <h1># <%= order.id %></h1>
  <ul>
    <% orders.products.forEach( product => { %>
    <li><%= product.title %>(<%= product.quantity %>)</li>
    <% }); %>
  </ul>
</li>
<% }); %>
```

[Eager Loading](https://sequelize.org/master/manual/eager-loading.html)

- `A.findAll({include: 'B'})`: B 와 연관된 모든 A 를 가져온다.
- A 모델의 메소드를 사용해서 A와 연결되어 있는 B 모델을 반환 값으로 가져 올 수 있다.

```js
req.user
  .getOrder({ include: ["products"] })
  .then((orders) => {
    // orders.products: [{title, quantity}, {title, quantity}]
  })
  .catch((err) => console.log(err));
```

```js
const tasks = await Task.findAll({ include: User });
```

```js
// console.log(JSON.stringify(tasks, null, 2));
[
  {
    name: "A Task",
    id: 1,
    userId: 1,
    user: {
      name: "John Doe",
      id: 1,
    },
  },
];
```

```js
tasks[0].user instanceof User; // true
```
