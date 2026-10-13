---
name: vulkano-backend-auth
description: Use when adding or editing login/logout/session-check authentication in this Vulkano framework project — Auth model + User model with bcrypt password hashing, AuthController, app/config/express/jwt.js, the req.auth middleware, protected /api routes, JWT in an httpOnly cookie.
---

# Backend Auth

Rules: `references/AGENTS/BACKEND.md` § Auth. This skill is the wiring, in order. Reference implementation in the core: `node_modules/@vulkano/core/examples/` (`services/Auth.js`, `controllers/api/AuthController.js`, `config/middlewares/auth.js`, `config/express/jwt.js`) — here `Auth` is a **model** (it owns the authentication business logic), not a service.

## When to use

- Adding login/logout/current-user to a project
- Protecting `/api` routes, reading the logged user (`req.auth`) in a controller
- Hashing/verifying passwords

Not for the frontend guard — see vulkano-frontend-router § Auth guard.

## 1. Setup

```bash
pnpm add bcryptjs   # core uses it too, but under pnpm the app can't require a core dep
```

- `.env`: non-empty `JWT_SECRET_KEY` and `SALT_KEY` (already listed in `.env.example`). Missing → ask the user or generate one (`node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`) and write it to `.env`. Never commit it.
- `app/config/settings.js`: add `salt: process.env.SALT_KEY || ''` (read as `app.config.settings.salt`).
- Copy `examples/config/express/jwt.js` → `app/config/express/jwt.js`, set `enabled: true`. Its default `ignore` leaves `/api/` and every `/api/auth/*` route public except `/api/auth/current` — add each other public endpoint there. `enabled: true` + empty key → the server does not boot.
- Copy `examples/config/middlewares/auth.js` → `app/config/middlewares/auth.js` unchanged. express-jwt leaves the encrypted payload on `req.auth`; this middleware decrypts it, so controllers read `req.auth._id`. Role/ACL checks go in its commented block.
- Token expiration: `Jwt.encode` payloads need `expiration` (ms timestamp) — `Jwt.getToken` drops expired/missing ones, so the request gets 401. Only `expiration: false` in `jwt.js` disables that (avoid).

## 2. User model — password hashing

```js
// app/models/User.js
const bcrypt = require('bcryptjs');

const hash = (plain) => bcrypt.hashSync(`${app.config.settings.salt}-${String(plain).trim()}`, 10);

module.exports = {
  attributes: {
    email: { type: String, required: true, lowercase: true, trim: true },
    password: { type: String, select: false } // never returned unless .select('+password')
  },

  beforeSave(next) {
    if (this.isModified('password')) {
      this.password = hash(this.password); // only when changed — re-saving must not re-hash
    }
    next();
  },

  beforeFindOneAndUpdate(next) {
    if (this._update.password && String(this._update.password).trim()) {
      this._update.password = hash(this._update.password);
    } else {
      delete this._update.password; // empty password on edit = keep current one
    }
    next();
  },

  hash
};
```

## 3. Auth model

```js
// app/models/Auth.js — no attributes: plain global, not a collection
const bcrypt = require('bcryptjs');

const SESSION_MS = 1000 * 60 * 60 * 24 * 7; // 1 week — keep cookie maxAge equal

module.exports = {
  SESSION_MS,

  verifyPassword(plain, hashed) {
    return bcrypt.compareSync(
      `${app.config.settings.salt}-${String(plain || '').trim()}`,
      hashed || ''
    );
  },

  login({ email, password } = {}) {
    const conditions = {
      email: String(email || '')
        .toLowerCase()
        .trim(),
      active: true
    };

    return User.findOne(conditions)
      .select('+password')
      .then((u) => {
        if (!u || !Auth.verifyPassword(password, u.password)) {
          return VSError.reject('Invalid email or password', 401); // reject, never `return new VSError()`
        }
        return Auth.setToken(u);
      });
  },

  setToken(u) {
    const user = { _id: u._id, email: u.email, role: u.role || '' };
    return { user, token: Jwt.encode({ ...user, expiration: Date.now() + SESSION_MS }) };
  },

  getCurrent(auth) {
    const { _id } = auth || {};
    if (!_id) {
      return VSError.reject('Invalid token', 401);
    }
    return User.getUser(_id).then((u) =>
      u && u.active ? u : VSError.reject('User not found', 401)
    );
  }
};
```

Errors: always `return VSError.reject(...)` or `throw` inside `.then` — `return new VSError(...)` resolves the promise with an error object and login "succeeds".

## 4. AuthController

```js
// app/controllers/api/AuthController.js
module.exports = {
  'post login': (req, res) => {
    res.vsr(
      Auth.login(req.body).then(({ user, token }) => {
        res.cookie(Jwt.getConfig().cookieName || 'token', token, {
          httpOnly: true,
          secure: app.PRODUCTION,
          sameSite: 'lax',
          maxAge: Auth.SESSION_MS
        });
        return user; // token travels only in the cookie
      })
    );
  },

  'post logout': (req, res) => {
    res.clearCookie(Jwt.getConfig().cookieName || 'token');
    res.vsr(Promise.resolve({}));
  },

  'get current': (req, res) => {
    res.vsr(Auth.getCurrent(req.auth));
  }
};
```

`res.vsr(promise)` turns a rejection into the error response — no `.catch` needed. A rejected `VSError` keeps its status (401); a plain rejection becomes 400.

## After writing

- [ ] Test `test/app/controllers/Auth.http.test.js`: login ok (sets `token` cookie, body has no token/password), wrong password → 401, logout clears cookie, `current` with cookie → user, without → 401. Model test `test/app/models/Auth.test.js` for `verifyPassword`/`login`.
- [ ] `JWT_SECRET_KEY` and `SALT_KEY` set in `.env`; server boots with `jwt.js` enabled.
- [ ] Every non-auth public endpoint listed in `jwt.js` `ignore`; everything else under `/api/` requires the cookie.
