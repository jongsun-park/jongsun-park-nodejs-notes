# SQL introduction

## Choosing a database

데이터를 파일에 저장하고 열고 저장하고 하는 것보다 데이터 베이스에 연결 하는게 훨씬 효과적이다. 데이터베이스는 SQL(MySQL)과 NoSQL(MongoDB)로 구분할 수 있다.

### SQL

데이터를 행(Fileds)과 열(Records) 정의된 테이블(Tables)에 정의한다. 모든 데이터는 지정한 형식, 양식에 일치해야만 하고 테이블과 테이블은 서로 연결 될 수 있다. (ono to one, ont to many, many to many)

- Structure

  - Tables
  - Fields / Columns
  - Records (DATA)

- Strong Data Scheme: 모든 데이터는 정의된 형식, 사이즈에 적합해야 한다.
- Data Relations: Tables are conencted
- Query: `SELECT * FROM users WHERE age > 30`
  - SQL keywords / Syntax
  - Parameters / Data

### NoSQL

자바스크립트 객체와 비슷한 형태로 데이터를 저장한다. documents에서 데이터는 SQL 보다 자유로운 형식으로 저장되며, 데이터간 연결되어 있지 않는다. (단점) 연결 되어 있지 않기 때문에 중복된 값이 존재 할 수 있다.

- Structure

  - Database: shop
  - Collections: users, products, orders
  - Documents (DATA): {name: '', age: '' }, {name: '', location: ''}

- No Data Scheme -> No Structure required
- No/Few Data relations

## Setting up MySQL

[Window Installer](https://dev.mysql.com/downloads/installer/)

## Connecting out App to the SQL Database

[npm mysql2](https://www.npmjs.com/package/mysql2)

```js
// /util/database.js
const mysql = require("mysql2");

// pool: 이전 연결을 재사용해서 연결 시간을 단축 시킨다.
const pool = mysql.createPool({
  host: "localhost",
  user: "root",
  database: "node-complete",
  password: "",
});

module.exports = pool.promise();

// const [rows,fields] = await promisePool.query("SELECT 1");
// const [rows, fields] = await db.execute("SELECT * FROM products")

// app.js
const db = require("../util/database.js");

db.execute("<SQL QUERY>");
```

## Basic SQL & Creating a Table

```SQL
CREATE TABLE `node-complete`.`products` (
  `id` INT UNSIGNED ZEROFILL NOT NULL AUTO_INCREMENT,
  `title` VARCHAR(255) NOT NULL,
  `price` DOUBLE NOT NULL,
  `description` TEXT NOT NULL,
  `imageUrl` VARCHAR(255) NULL,
  PRIMARY KEY (`id`),
  UNIQUE INDEX `id_UNIQUE` (`id` ASC) VISIBLE);
```

```SQL
INSERT INTO `node-complete`.`products` (`title`, `price`, `description`, `imageUrl`) VALUES ('A Book', '19.99', 'This is a book', 'https://images.prismic.io/wonderbly/709a459d-43f8-4fd3-adc7-0cf811c93e3c_ABC_carousel_hero_PORTRAIT.png?auto=format%2Ccompress&dpr=&w=1070&h=&fit=&crop=&q=35&gif-q=90');
```

```js
// mysql.createPool({}).promise().execute().catch().then()
db.execute("SELECT * FROM products")
  .then((result) => {
    console.log(result[0], result[1]);
  })
  .catch((err) => console.log(err));
```

promise()

- 자바스크립트 객체로 상태(pending, fullfilled, failed)에 따라 비동기 함수를 실행한다.
- `promise().then( <fullfilled> ).catch( <failed> ).finally( <always> )`

## Fetching Products

```js
// /models/product.js
class Product {
  fetchAll() {
    return db.execute("SELECT * FROM products");
  }
}
// /controllers/shop.js
const getIndex = (req, res, next) => {
  Product.fetchAll()
    .then(([rows, fieldData]) => {
      res.render("shop/index", { products: rows });
    })
    .catch((err) => console.log(err));
};
```

## Inserting Data into the Database

```js
// /models/product.js
class Product {
  static save() {
    return db.execute("INSERT INTO products (title) VALUES (?)", [this.title]);
  }
}

// /controllers/admin.js
const addProduct = (req, res, next) => {
  const [title] = req.body;
  Product.save()
    .then(() => {
      res.redirect("/");
    })
    .catch((err) => console.log(err));
};
```

### [Prepared statement and parameters](https://github.com/sidorares/node-mysql2/blob/master/documentation/Examples.md#prepared-statement-and-parameters)

```js
const mysql = require("mysql2");
const connection = mysql.createConnection({ user: "test", database: "test" });

connection.execute("SELECT 1+? as test1", [10], (err, rows) => {
  //
});
```

## Fetching a Single Product with the "where" Condition

```js
// /models/product.js
class Product {
  static getById(id) {
    return db.execute("SELECT * FROM products WHERE id = ?", [id]);
  }
}

// /controllers/shop.js
exports.getProduct = (req, res, next) => {
  const prodId = req.params.productId;
  Product.findById(prodId)
    .then(([product]) => {
      res.render("shop/product-detail", {
        product: product[0], // product: [{title, price, ...}]
        pageTitle: product[0].title,
        path: "/products",
      });
    })
    .catch((err) => console.log(err));
};
```
