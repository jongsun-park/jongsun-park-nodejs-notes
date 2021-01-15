# S23 Adding Payments

## How Payments Work

Payment Process

- Collect Payment Method // Stripe
- Verify Payment Method // Stripe
- Charge Payment Method // Stripe
- Manage Payments // Stripe
- Process Order in App

Stripe

- (Client) Collect Payment detail
- (Client) Send to stripe servers
- (Client) Return payment token
- (Client) Send to server with token & payment details
- (Server) Create Payment Data to stripe servers

## Using Stripe in Your App

```html
<!-- client -->
<!-- chekcout.ejs -->
<!-- use stripe CDN -->
<div class="centered"><button id="order-button" class="btn">ORDER</div>
<script src="https://js.stripe.com/v3/"> // stripe CDN
<script>
var stripe = Stripe('Publishable Key'); // local testing key
var orderBtn = document.getElementById("order-button")
orderBtn.addEventListener('click', function(){
    stripe.redirectToCheckout({
        sessionId: <%= sessionId %>
    })
    // sessionId - 서버에서 제공되어야 하는 세션
})
</script>

```

```js
// Server
// use stripe library

const stripe = require('stripe')('scret key')

exports.getChechout = (req, res, next) => {
  let products;
  let total;

  req.user
    .populate("cart.items.productId")
    .execPoluate()
    .then((user) => {
      products = user.cart.items;
      products.forEach((p) => {
        total += p.quantity * p.productId.price;
      });

      return stripe.checkout.sessions.create({
        payment_method_types: ["card"], // accepts credit cart
        line_items: products.map((p) => {  // what products will checkout
          return {
            name: p.productId.title, // from populate from product id
            description: p.productId.description,
            amount: p.productId.price * 100, // cent to dollar
            currency: "usd",
            quantity: p.quantity,
          };
        }),
        success_url:
          req.protocol + "://" + req.get("host") + "/checkout/success", // http://localhost:3000/checkout/success
      }),
        cancel_url:
          req.protocol + "://" + req.get("host") + "/checkout/cancel",
      });
    })
    .then((session) => {
      res.render("shop/checkout", {
        path: "/checkout",
        pageTitle: "Checkout",
        products: products,
        totalSum: total,
        sessionId: session.id,
      });
    });
};
```

```js
// router
router.get("/checkout", isAuth, shopController.getCheckout);

router.get("/checkout/success", shopController.getCheckoutSuccess);
router.get("/checkout/cancel", shopController.getCheckout);
```
