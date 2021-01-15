# S20 File Uploads & Downloads

## Adding a File Ficker to the Frontend

project add form

- previous: enter image url
- realistic: upload image file
- [<input type="file">](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/file)

```html
<label for="product-image">Image</label>
<input
  type="file"
  id="product-image"
  name="product-image"
  accept="image/png, image/jpeg"
/>
```

## Handling Multipart From Data

```html
<!-- form -->
<!-- default: enctype="application/x-www-form-urlencoded" -->
<form action="/admin/add-product" method="POST" enctype="multipart/form-data">
  <input type="file" name="image" id="image" />
</form>
```

`enctype = multupart/form-data`: 이 폼은 multipart를 포함하고 있다를 알려주는 것

```js
// controllers
exports.getAddProduct = (req, res, next) => {
  const image = req.body.image;
};
```

```js
// app.js
app.use(bodyParser.urlencoded({ extended: false }));
// url encoded data - just 'TEXT' data
// Header/Content-Type/application/x-www-form-urlencoded
```

<hr/>

[body-parser](https://www.npmjs.com/package/body-parser)

- API 요청에서 받은 body 값을 파싱 하는 역할
- client 에서 body에 데이터를 담아 서버에 전달 할 수 있다. (req.body)
- 전달된 incoming request body 에 담긴 데이터를 서버 내에서 해석이 가능한 형태로 변형 해주어야 한다. (body-parser, multer)
  - body-parser: text
  - multer: multipart
- `app.use(bodyParser.urlencoded({ extended: false });`
  - `extended: true` // 기본값, qs 모듈 사용 (설치 필요)
  - `extended: false` // nodejs에 내장 모듈 queryString 사용

[expressjs multer 한글](https://github.com/expressjs/multer/blob/master/doc/README-ko.md)

- 폼에 전달된 파일 데이터를 전달 할 수 있도록 도와준다.
- multer는 `enctype="mulipart/form-data"`가 아닌 폼에서 동작하지 않는다.

[MIME](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types) (Multipurpose Internet Mail Extensions) : HTTP가 웹에서 전송되는 객체 각각에 붙이는 데이터 포맷 라벨

- 개별 타입: text(읽을 수 있는), image, audio, video, application(이진 데이터)
- 멀티 파트 타입: 합성된 문서를 나타내는 방법
- `multipart/form-data`: 브라우저에서 서버로 HTML Form 내용을 전송시 사용

## Handling Multipart Form Data

```js
// app.js
const multer = require("multer");

app.use(bodyParser.urlencoded({ extended: false }));
// app.use(multer().single(image)); // 하나의 파일만 처리 하고 전달된 버퍼를 메모리에 저장
app.use(multer({ dest: "image" }).single(image)); // 지정된 경로에 저장
```

```js
// controllers
const image = req.file;
console.log(image);
```

```js
// req.file
{
  filedname, originalname, encoding, minetype, buffer | filename, size;
}
```

## Configuring Multer to Adjust filename & Filepath

Multer 설정 ([DiskStorage](https://github.com/expressjs/multer#diskstorage))

- 저장 위치
- 파일 이름

```js
// app.js
const fileStorage = multer.diskStorage({
  destination: (req, file, cb) => {
    // error: null
    // path: /images
    cb(null, "images");
  },
  filename: (req, file, cb) => {
    cb(null, new Date().toString() + "-" + file.originalname);
  },
});

app.use(multer({ storage: fileStorage }).single(image));
```

## Filtering Files by Mimetype

[multer filterFilter](https://github.com/expressjs/multer#filefilter)

```js
const fileFilter = (req, file, cb) => {
  if (
    file.minetype === "image/jpg" ||
    file.minetype === "image/jpeg" ||
    file.minetype === "image/png"
  ) {
    cb(null, true); // accept
  } else {
    cb(null, false); // reject
  }
};

app.use(multer({ storage: fileStorage, fileFilter: fileFilter }).single(image));
```

## Storing File Data in Database

전달된 이미지는 이미지 폴더 저장하고, 데이터베이스에는 경로만 저장한다.

```js
// controllers
exports.getAddProduct = (req, res, next) => {
  const image = req.file;
  if (!image) {
    // image 가 없다면,
    return res.status(422).render("admin/edit-product", {
      //
      errorMessage: "Attached file is not an image",
      validationErrors: [],
    });
  }

  const imageUrl = image.filename;

  const product = new Product({
    //
    imageUrl: imageUrl,
  });

  return product.save();
};
```

## Serving Images Statically

imageUrl: '/images/filename.png'

```js
app.use("images", express.static(path.join(__dirname, "images")));
```

- `express.static()`: 디렉토리 안에 있는 모든 파일을 루트 경로에서 접근 가능 하도록 한다.
- images/file.png -> file.png
- 미들웨어에 경로 ('/images')를 하여, 해당 경로에 이미지를 읽을 수 있도록 한다.

```ejs
<img src="/<%= product.url %>" alt="<%= product.title %">
```

- 절대 경로를 써서 루트 경로에서 이미지에 접근 할 수 있다.
- `src="<%= product.url %>"`: localhost:3000/admin/images/filename.png
- `src="/<%= product.url %>"`: localhost:3000/images/filename.png

## Download Files with Authentication

서버에 저장된 pdf 파일 클라이언트로 전송하기

```html
<a href="/orders/<%=order.id %>">Invocies</a>
```

```js
const fs = require("fs"); // file system
const path = require("path"); // path

// shop routers
router.get("/orders/:orderId", isAuth, (req, res, next) => {
  const orderId = req.params.orderId;
  const invoiceName = `invoice-${orderId}.pdf`;
  const invoicePath = path.join("data", "invoices", invoiceName);
  fs.readFile(invoicePath, (err, data) => {
    if (err) {
      return next(err);
    }
    res.send(data);
  });
});
```

## Setting File Type Headers

API 요청을 보냈을 때 파일이 어떻게 열릴지, 어떤 이름으로 열릴지 지정할 수 있다.

```js
setHeader("Content-Type", "application/pdf");
setHeader("Content-Disposition", `attachment; filename="${invoiceName}"`);
```

`application/pdf`

- MIME Type
- 브라우저는 text가 아닌 경우 별도의 행동을 취한다.
- Adobe Portable Document Format (PDF)

`Content-Disposition`

- 파일을 어떻게 보여줄지 결정한다
- `Content-Disposition: inline`: (default) displayed inside the Web page
- `Content-Disposition: attachment`: it should be downloaded
- `Content-Disposition: attachment; filename="filename.jpg"`

## Restricting File Access

접속한 사용자가가 소유한 invoice 만 보여주기

```js
order.user.userId.toString() !== user.userId.toString();
```

```js
exports.getInvoice = (req, res, next) => {
  const orderId = req.params.orderId;

  Order.findById(orderId)
    .then((order) => {
      if (!order) {
        return next(new Error("No order found"));
      }
      if (order.user.userId.toString() !== user.userId.toString()) {
        return next(new Error("Unauthorized"));
      }
    })
    .catch((err) => next(err));

  const invoiceName = `invoice-${orderId}.pdf`;
  const invoicePath = path.join("data", "invoices", invocieName);

  fs.readFile(invoicePath, (err, data) => {
    if (err) {
      return next(err);
    }
    setHeader("Content-Type", "application/pdf");
    setHeader("Content-Disposition", `attachment; filename="${invoiceName}"`);
    res.send(data);
  });
};
```

## Streaming Data vs Preloading Data

Preload: 파일을 읽고 서버 메모리에 저장한 다음 내보낸다

Stream

- res: writable
- 파일을 읽고 데이터를 서버에 저장하지 않고 바로 res 객체로 보낸다.

```js
const file = fs.createReadStream(invoicePath);
setHeader("Content-Type", "application/pdf");
setHeader("Content-Disposition", `attachment; filename="${invoiceName}"`);
file.pipe(res);
```

[How to use fs.createReadStream?](https://nodejs.org/en/knowledge/advanced/streams/how-to-use-fs-create-read-stream/)

```js
// 해당 경로의 파일을 읽을 수 있는 스트림으로 연다
const readStream = fs.createReadStream(filepath);

// 스트림이 유효한 경우, 클라이언트와 연결된 res 객체에 연결한다.
readStream.on("open", () => {
  reaStream.pipe(res);
});

// 스트림 도중 에러가 생겼을 때, 주로 이름이 잘못된 경우
readStream.on("error", () => {
  res.end(error);
});
```

[stream.pipe](https://nodejs.org/en/knowledge/advanced/streams/how-to-use-stream-pipe/)

`readable.pipe(writeable)`: 읽을 수 있는 스트림을 쓸수 있는 스트림으로 연결한다.

## Using PDFKit for .pdf Generation

[https://pdfkit.org/](https://pdfkit.org/)

```js
const PDFDoc = require("pdfkit");

const pdfDoc = new PDFDoc(); // returns readable stream
setHeader("Content-Type", "application/pdf");
setHeader("Content-Disposition", `attachment; filename="${invoiceName}"`);

pdfDoc.pipe(fs.createWriteStream(invoicePath));
pdfDoc.pipe(res); // connect to writable stream

pdfDoc.text("Hello World"); // PDF 안 내용 작성

pdfDoc.end(); // PDF 작성 종료하기
```

## Gernerating .pdf files with Order Data

```js
pdfDoc.fontSize(26).text("Invoice", { underline: true });
pdfDoc.text("---");

let totalPrice = 0;
order.products.forEach((prod) => {
  totalPrice += prod.product.price * prod.quantity;
  pdfDoc.text(
    `${prod.product.title} - ${prod.product.quantity} x $${prod.product.price}`
  );
});

pdfDoc.text("---");
pdfDoc.fontSize(20).text(`Total Price: ${totalPrice}`);

pdfDoc.end();
```

## Deleting Files

이미지를 교체하거나, 제품을 삭제 할 때 연결된 이미지 파일 삭제

```js
// utils/file.js

const fs = require("fs");

const deleteFile = (filepath) => {
  fs.unlink(filepath, (err) => {
    if (err) throw err;
  });
};
```
