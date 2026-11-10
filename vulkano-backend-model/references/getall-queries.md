## Ad-hoc filters in `getAll` (e.g. `?category=<id>`)

Not part of the populate mechanism — a separate, equally common need: filtering the list by a plain field value passed in the query string. Read it off `props` and merge it into `defaultProps.filter` before calling `Paginate.serializeQuery`:

```js
getAll(props) {
  const defaultProps = { sort: 'createdAt|DESC', searchBy: ['name'], filter: { active: true } };
  if (props.category) {
    defaultProps.filter.category = props.category; // exact-match filter, e.g. ?category=<id>
  }
  // UNSAFE — never: lets ?filter[active]=false override the soft-delete scope
  // defaultProps.filter = { ...defaultProps.filter, ...props.filter };
  const populate = this._buildPopulate(props);
  const query = Paginate.serializeQuery(defaultProps, props);
  return Paginate.get(Example, query, populate);
}
```

`props` is `req.query` — untrusted. Destructure only the keys you expect; never spread `props` or `props.filter` into `filter`. Validate/cast each value (e.g. confirm it's a valid ObjectId) before using it in the filter.

## Field selection in `getAll` (`?fields=`)

Source: `Paginate.get` in `node_modules/@vulkano/core/libs/Paginate.js`. `getAll`'s `props.fields` — a comma-separated string (`?fields=name,price`) or an array (`['name', 'price']`) — becomes a Mongoose `.select()` (`opts.select = fields.join(' ')`). `getAll({ fields: 'name' })` returns documents with only `name` (plus `_id`, Mongoose always includes it) instead of the full document. Applies to both pagination shapes — the normal page object and the `?page=all` bare array (§ above) — since both go through the same `Paginate.get`. Request fields pass through the model's `_sanitizeFields` (inherited from the scaffold): any `+field` and any `select: false` path is dropped, so `?fields=+password` never returns the hash — mark secrets `select: false` and read them only with `.select('+field')` in model code. Does NOT apply to `get<ModelName>(id, props)` — that's a plain `findOne`, unrelated to `Paginate.get`, so it always returns the full document regardless of `props.fields`.

No extra code needed in a model's `getAll` for this to work — `props`/`req.query` flowing straight into `Paginate.serializeQuery` is enough. Only add a guard if the endpoint must NOT allow field selection (e.g. it always needs a specific field for business logic downstream) — otherwise a request with `?fields=` that omits that field is a real bug: code assuming a field exists on every `getAll` result breaks silently.

## Populate — `autopopulate: true` + built-in `_buildPopulate`

Every model gets `_buildPopulate(props, extra)`, `_parsePopulateEntries(props)`, and `_getSanitizedPopulate(props)` mixed in automatically (`database/scaffold.js`, merged into every model's statics — no per-model wiring needed). Nothing else to require or import.

- **Security gate**: a `ref` attribute is only ever populatable if it declares `autopopulate: true`, or the model explicitly allowlists it per-call via the `extra` arg (`this._buildPopulate(props, ['someRef'])`). A `ref` field without either is never populated, no matter what the caller asks for.
- **Query syntax** — comma-separated relations, optional `|`-separated field allowlist per relation after a colon:
  - `?populate=category` → full related doc
  - `?populate=category:name` → only `{ _id, name }`
  - `?populate=category:name|slug,brand` → `category` limited to name+slug, plus full `brand`
- **Usage in a model method**: call `this._buildPopulate(props)` to get the populate array, then either pass it as `Paginate.get(Model, query, populate)`'s 3rd arg (list endpoints) or `.forEach((p) => query.populate(p))` on a `findOne` query (single-record endpoints).
- Field names are matched case-insensitively against the schema's real paths — the caller doesn't need to know exact casing.
- **Sensitive fields**: mark them `select: false` on the target model. `_buildPopulate` runs the caller's `field1|field2` list through the ref model's `_sanitizeFields` (core 2.1.6+), so `?populate=user:password` and `user:+password` never return the hash (`references/AGENTS/BACKEND.md` § Models).
