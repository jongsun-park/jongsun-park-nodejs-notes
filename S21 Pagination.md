# S21 Adding Pagination

## Pagination Links

```html
<div class="pagination">
  <a href="/?page=1">1</a>
  <a href="/?page=2">2</a>
</div>
```

## Retrieving a Chunk of Data

쿼리를 전달 할 때 어디에서 부터 (skip) 얼마나 만은 (limit) 문서를 출력 할지 전달 할 수 있다.

```js
const ITEMS_PER_PAGE = 2;

// controllers
exports.getIndex = (req, res, next) => {
  // url /?page=1
  const page = req.query.page;
  Product.find()
    .skip((page - 1) * ITEMS_PER_PAGE)
    .limit(ITEMS_PER_PAGE)
    .then((products) => {
      res.render("shop/index", {
        prods: products,
        path: "/",
      });
    })
    .catch((err) => next(err));
};
```

## Preparing Pagination Data on the Server

문서의 갯 수를 파악한후, ejs 템플릿에 pagination 관련 데이터 전달 하기

```js
const ITEMS_PER_PAGE = 2;

exports.getIndex = (req, res, next) => {
  // url /?page=1
  const page = +req.query.page;
  let totalItems = "";

  Product.find()
    .countDocuments((numProducts) => {
      totalItems = numProducts;
      return Product.find()
        .skip((page - 1) * ITEMS_PER_PAGE)
        .limit(ITEMS_PER_PAGE);
    })
    .then((products) => {
      res.render("shop/index", {
        prods: products,
        pageTitle: "Shop",
        path: "/",
        currentPage: page,
        hasNextPage: ITEMS_PER_PAGE * page < totalItems,
        hasPreviousPage: page > 1,
        nextPage: page + 1,
        previousPage: page - 1,
        lastPage: Math.ceil(totalItems / ITEMS_PER_PAGE),
      });
    })
    .catch((err) => next(err));
};
```

## Dynamic Pagination Buttons

```html
<section class="pagination">
  <% if(currentPage !== 1 && previousPage !== 1){ %>
    <a href="?page=1">
  <% } %>
  <% if(hasPreviousPage){ %>
    <a href="?page=<%= previousPage %>"><%= previousPage %></a>
  <% } %>
  <a href="?page=<%= currentPage %>" class="active"><%= currentPage %></a>
  <% if(hasNextPage){ %>
    <a href="?page=<%= nextPage %>"><%= nextPage %></a>
  <% } %>
  <% if(lastPage !== currentPage && nextPage !== lastPage){ %>
    <a href="?page=<%= lastPage %>" class="active"><%= lastPage %></
  <% } %>
</section>
```
