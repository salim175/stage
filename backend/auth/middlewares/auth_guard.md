# 🔐 Middleware: JWT Authentication & User Verification
```js
require('dotenv').config();

const { expressjwt: jwt } = require("express-jwt");
const UserModel = require('../models/user');

exports.requireSignIn = jwt({
    secret: process.env.TOKEN_SECRET,
    algorithms: ["HS256"],
    userProperty: 'auth', //to make req.auth available
    onExpired: async (req, err) => {
        console.log("handling expired", err)
        if (new Date() - err.inner.expiredAt < 5000) { return; }
        throw err;
    },
})

exports.verifyUser = async (req, res, next) => {

    let decodedProperties = req.auth
    // Just on dev-mode ------
    console.log(decodedProperties)
    // -----
    if (!decodedProperties) {
        return res.status(403).json({
            error: "you must signin  !!!!!"
        })
    }
    if (decodedProperties.id) {
        //to complete -------

        const user = await UserModel.findById(decodedProperties.id) // retriving user from db based on decodedProperties.userId
        // ----------
        if (user) {
            user.password = null
        }
        req.user = user
    }
    next()
}
```

🚨 in ``.env`` add ``TOKEN_SECRET=test``

## 1️⃣ ``requireSignIn`` :
### ✅ What it does:

1. Validates the JWT token sent in the ``Authorization: Bearer <token>`` header.

2.  Uses the HS256 algorithm for signature verification.

3. Verifies the token using the ``TOKEN_SECRET`` from the ``.env`` file.

4. If the token is valid, the decoded payload is stored in ``req.auth``.

5. If the token is expired, it throws (unless within a 5-second grace window for retry).

## 2️⃣ ``verifyUser`` :

### ✅ What it does:

1. Requires a valid ``req.auth`` (set by ``requireSignIn``).

2. Fetches the user from the database using the ID from the token payload.

3. If user exists, it attaches the full user document to ``req.user`` (with password removed).

4. Makes ``req.user`` available to all downstream route handlers and middlewares.


## ✅ Combined Usage in Route: 
```js
ex:
router.get("/user",
  requireSignIn, // Verifies token and sets req.auth
  verifyUser,    // Loads user from DB and sets req.user
  getAllUsers // Can now access req.user
)
```

## ✅ What requireSignIn does: 
Your middleware uses:

```js
exports.requireSignIn = jwt({
  secret: process.env.TOKEN_SECRET,
  algorithms: ["HS256"],
  userProperty: 'auth'
})
```
This:
- Verifies the token
- Decodes it
- Attaches the decoded payload to:

```js
req.auth = {
  id: '68249dd9b1225bbf77e46655',
  iat: 1747304222,
  exp: 1747390622
}
```