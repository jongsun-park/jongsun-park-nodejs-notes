# 6 Working with Dynamic content & Adding templating Engines

## Sharing Data Across Requests & Users

서버에서 데이터를 변수에 넣어 뿌리는 방식, 이 디자인 패턴은 연습용으로만 사용된다. 실제 어플리케이션에서는 사용자, 브라우저에 따라 요청에 대해 응답 값을 가져와야 한다.

```js
// routes/admin.js
const products = [];

router.post("/add-product", (req, res, next) => {
  products.push({ title: req.body.title });
  res.redirect("/");
});

exports.products = products;
```

```js
// routes/shop.js
router.get("/", (req, res, next) => {
  console.log("shop.js", adminData.products);
  res.sendFile(path.join(rootDir, "views", "shop.html"));
});
```

## Templating Engines

- HTMLish Template -> HTML file
- Node / Express content + Templating Engine
- Replaces placeholders or snippets with HTML Content

템플릿 언어로 작성된 파일은 / 서버에 저장된 콘턴츠 (데이터베이스, 변수 ...)와 템플릿 엔진의 도움을 받아 / 브라우저에서 읽을 수 있는 HTML로 전환된다.

템플릿 문서와, 서버에 저장된 콘텐츠는 클라이언트(브라우저)에서 접근할 수 없다. 즉, 보호 된다.

|                                                                |               |              |                          |
| -------------------------------------------------------------- | ------------- | ------------ | ------------------------ |
| ejs                                                            | `<?= name ?>` | normal HTML  | plain javascript         |
| pug (jade)                                                     | `p #{name}`   | minimal HTML | custom template language |
| [handlebars](https://www.npmjs.com/package/express-handlebars) | `{{ name }}`  | normal HTML  | custom template language |

## Installing & Implementing [Pug](https://pugjs.org/api/getting-started.html)

[app.set(name, value)](https://expressjs.com/en/api.html#app.set)

- 서버 전체에 적용할 값 지정 할 수 있다.
- views: 템플릿 파일에 있는 위치
- view engine: 템플릿 파일에 적용 시킬 템플릿 엔진
- `app.get(name); // value`
- pug 템플릿 안에 public 파일을 사용 하기 때문에, bodyParser, static 미들웨어 뒤, 라우트 앞에 와야 한다.

```js
// app.js
app.set("view engine", "pug"); // 사용할 템플릿 엔진
app.set("views", "views"); // 템플릿이 저장된 위치 (process.pwd()/views)

// routs/shop.js
res.render("shop"); // 기본값 템플릿 엔진, 폴더 사용
```

```pug
// views/shop.pug
<!DOCTYPE html>
html(lang="en")
    head
        meta(charset="UTF-8")
        meta(name="viewport", content="width=device-width, initial-scale=1.0")
        title Add Product
        link(rel="stylesheet", href="/css/main.css")
        link(rel="stylesheet", href="/css/product.css")
    body
        header.main-header
            nav.main-header__nav
                ul.main-header__item-list
                    li.main-header__item
                        a.active(href="/") Shop
                    li.main-header__item
                        a(href="/admin/add-product") Add Product

        main
            h1 My Products
            p List of all the products...

```

## Outputting Dynamic Content

`res.render('shop', {title: "Shop", products: adminData.products})`

```pug
// pug.js
h1 #{title}

if product.length > 0 // conditions
    each product in products // looping
        h1 #{product.title}
else
    h1 No Products
```

## Adding a Layout

[pug template inheritance](https://pugjs.org/language/inheritance.html)

레이아웃 템플릿을 만들어 중복되는 html 코드를 한 곳에서 관리 할 수 있다. 레이아웃 템플릿에서 block 키워드를 사용하여 삽입될 태그의 위치를 지정하고, 각 페이지에서 해당 block 에 들어갈 요소를 넣을 수 있다.

```
// 레이아웃 템플렛
block <name>
```

```pug
// 페이지 템플렛
extends layout/main-layout.pug

block <name>
    code
```

```pug
// layout/main-layout.pug
<!DOCTYPE html>
html(lang="en")
    head
        meta(charset="UTF-8")
        meta(name="viewport", content="width=device-width, initial-scale=1.0")
        title Page Not Found
        link(rel="stylesheet", href="/css/main.css")
        block styles
    body
        header.main-header
            nav.main-header__nav
                ul.main-header__item-list
                    li.main-header__item
                        a(href="/") Shop
                    li.main-header__item
                        a(href="/admin/add-product") Add Product

        main
            block content
```

```pug
//shop.pug
extends layout/main-layout.pug

block styles
    link(rel="stylesheet", href="/css/product.css")

block content
    if prods.length > 0
        each product in prods
            .grid
                article.card.product-item
                    header.card__header
                        h1.product__title #{product.title}
                        // 생략
    else
        h1 No Products
```

페이지 마다 다른 title 지정 하기

- 렌더링 할 때 변수로 전달하고, layout 템플릿에서 변수로 렌더링 지정
- layout.pug: `title #{pageTitle}`
- routes/shop.js: `res.render('shop', {pageTitle: "shop"})`

경로에 따라 클래스 넣기

- layout.pug:
  - `a(href="/admin/add-product", class=(path === '/admin/add-product' && "active")) Add Product`
  - `class=(javascript expression)`
- routes/shop.js: `res.render('add-product', {pageTitle: "shop", path: '/add-product'})`

## Working with Handlebars

express에 등록되지 않은 엔진의 경우, 모듈에서 인스턴스를 가져온 후 app에 엔진 등록을 해주어야 한다. 엔진을 등록한 경우 등록한 이름의 확장자를 사용할 수 있다.

```js
// app.js
const handlebars = require("express-handlebars");
// 생략
app.engine("hbs", handlebars());
// 함수를 실행하여 초기화 된 인스턴스 두번째 인수로 넣는다
app.set("view engine", "hbs");
app.set("views", "views");
```

```js
res.render("404", { pageTitle: "404" });
```

```html
// views/404.hbs
<h1>{{ pageTitle }}</h1>
```

## Converting Handlebars: [handlebarsjs guide](https://handlebarsjs.com/guide/)

pug 와 가장 큰 차이점은 템플렛 안에서 자바스크립트 표현식을 사용할 수 없다. 로직은 노드 서버 안에서 작성되어 템플릿으로 전달해야 한다.

렌더링:

```js
const products = [{ title: "PRODUCT NAME" }];
res.render("shop", {
  pageTitle: "shop",
  products: products, // array or null
  hasProducts: products.length > 0, // boolean
});
```

데이터 출력: `{{ pageTitle }}`

조건문:

```
{{#if hasProducts}}
    <h1>Has Product</h1>
{{ else }}
    <h1>Don't have Product</h1>
{{/if}}
```

Loop

```
{{#each products}}
    <h1>{{ this.name }}</h1>
{{/each}}
```

## Adding the Layout to Handlebars

handlebars

- 기본으로 사용할 레이아웃 정보를 인스턴스에 적용해야 한다.
- 템플릿 파일에서는 `{{{ body }}}` 안에 들어갈 html 요소만 입력 할 수 있다.
- 템플릿 마다 따로 들어 가거나, 로직이 필요한 경우 템플릿을 랜더링 할 때 변수 값(boolean) 을 넣어 `{{#if}} condition {{/if}}`로 출력한다.

```js
//app.js

app.engine(
  "hbs",
  handlebars({
        layoutDir: "views/layout", // 레이아웃 디렉토리
        name: "main-layout", // 파일 명
        extname: "hbs", // 확장자 명
        // layout: false // 기본 레이아웃을 사용하지 않는 경우
    })
);

//views/main-layout.hbs
{{#if condition }}
// CODE (alternative to 'block style' or 'block script')
{{/if}}

{{{ body }}}

//404.hbs
<h1>Page Not Found!</h1>
```

```js
// routes/admin.js
router.get("/add-product", (req, res, next) => {
  res.render("add-product", {
    pageTitle: "Add Product",
    path: "/admin/add-product",
    formsCSS: true,
    productCSS: false,
    shopActive: false,
    productActive: true,
  });
});
```

## Working with EJS

```js
//app.js

app.set("view engine", "ejs");
app.set("views", "views");
```

```ejs
//값 출력
<title><%= pageTitle %></title>

//조건문
<% if( products.length ){ %>
    <h1>Product List</h1>
<% }else{ %>
    <h1>No Product Found</h1>
<% } %>

//루프
<% for(product of products){ %>
    <h1><%= product.title %></h1>
<% } %>
```

## Working on the Layout with Partials

메인 레인아웃을 존재 하지 않지만, 공통된 코드를 includes 폴더 안에 저장해서 템플릿 파일 안에 가져 올 수 있다.

- `views/includes/head.ejs` (`<html> ~ main css`)
- `views/includes/navigation.ejs` (`<header>~</header>`)
- `views/includes/end.ejs` (`</body></html>`)

`<%- include('includes/head') %>`

```ejs
// 404.ejs
<%- include('includes/head.ejs') -%>
  </head>
  <body>
    <%- include('includes/navigation.ejs') -%>
    <h1>Page Not Found!</h1>
<%- include('includes/end.ejs') -%>
```

```ejs
// navigation.ejs
// 404.ejs 에서 해당 partials 을 사용하는 경우, 반드시 path 를 렌더링 할 때 전달 해줘야 한다.
 <ul class="main-header__item-list">
    <li class="main-header__item"><a class="<%= path === '/' ? 'active' : '' %>" href="/">Shop</a></li>
    <li class="main-header__item">
    <a class="<%= path === '/admin/add-product' ? 'active' : '' %>" href="/admin/add-product">Add Product</a>
    </li>
</ul>
```
