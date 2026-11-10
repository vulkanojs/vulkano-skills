## Create + edit → one method/route, not two

Same rule on both sides of a form: don't split create and edit into separate methods/controllers just because one has an `:id` and the other doesn't.

- **API controller**: already the natural shape — `post` (create) and `'put :id'` (update) are two method keys on the _same_ controller calling the _same_ model, shown above. Don't add a separate `CreateController`/`EditController`.
- **View controller** serving a form page (`/products/create`, `/products/edit/:id`) — one method renders the same template for both, branching only on whether `:id` is present to prefill data:

```js
// app/controllers/ProductsController.js
module.exports = {
  'get create': (req, res) => res.render('products/form.html', { product: null }),
  // verb + one path token: 'get edit :id' would register /products/edit and drop :id
  'get edit/:id': (req, res) => {
    Product.getProduct(req.params.id)
      .then((product) => res.render('products/form.html', { product }))
      .catch(() => res.status(404).render('_shared/errors/404.html'));
  }
};
```

Matches the frontend's single `Form.vue` (branches on route `:id` the same way — see vulkano-frontend-router § Create + Edit) — one template/form markup reused for both actions instead of duplicating it.
