# 🔐 Authentication Controller: Login & Get Logged-In User:
```js
const UserModel = require('../models/user');
const moment = require('moment-timezone');

const jwt = require("jsonwebtoken");
const bcrypt = require("bcrypt")

exports.login = async (req, res) => {

    const { email, password, rememberMe } = req.body || {}

    if (!email || !password) {
        return res.status(400).json({ error: 'Email and Password are required' });
    }

    //console.log(req.body, rememberMe == true ? "1d" : "60m")
    const foundUser = await UserModel.findOne({ email: email.toLowerCase() });
    if (foundUser && password) {
        if (!foundUser.blocked) {
            const validPwd = await bcrypt.compare(password, foundUser.password)
            if (validPwd) {
                let token = await jwt.sign({ "id": foundUser.id }, process.env.TOKEN_SECRET, { expiresIn: rememberMe == true ? "1d" : "60m" });
                if (foundUser.last_logins.length == 10) {
                    foundUser.last_logins.shift();
                }
                foundUser.last_logins.push({ date: moment.tz(new Date(), "DD MMM YYYY HH:mm", "Europe/Paris").unix() })
                await foundUser.save();
                return res.status(200).send(token);
            } else {
                return res.status(401).send("info are not correct");
            }
        } else {
            return res.status(403).send("user blocked");
        }
    } else {
        return res.status(401).send("info are not correct");
    }

}

exports.getLoggedUser = async (req, res) => {
    try {

        if (req.user) {

            return res.status(200).send(req.user);

        } else {
            return res.status(400).send("info are not correct");
        }
    } catch (error) {
        return res.status(500).send(error);
    }
};
```

## 1️⃣ ``login`` :

```js
let token = await jwt.sign(
  { id: foundUser.id },
  process.env.TOKEN_SECRET,
  { expiresIn: rememberMe == true ? "1d" : "60m" }
)
```
### ✅ What this does:
Creates a JWT with this payload:

```json
{ "id": "68249dd9b1225bbf77e46655" }
```
- Adds auto fields:
    - ``iat``: issued at (Unix timestamp)
    - ``exp``: expiration time (Unix timestamp)

- Signs it using your ``TOKEN_SECRET``

**So the full decoded token looks like:**

```json
{
  "id": "68249dd9b1225bbf77e46655",
  "iat": 1747304222,
  "exp": 1747390622
}
```

### 🔄 So what’s happening?
1. You create a JWT with ``{ id: foundUser.id }``

2. JWT library adds ``iat`` and ``exp``

3. Client sends the token in ``Authorization: Bearer <token>``

4. ``requireSignIn`` decodes it and stores it in ``req.auth``

## 🎯 Routes:
```js
const express = require('express');
const { login, getLoggedUser } = require('../controllers/auth_controller');
const { requireSignIn, verifyUser } = require('../middlewares/auth-guard');


const router = express.Router();


router.post("/login", login)
router.get("/profile", requireSignIn, verifyUser, getLoggedUser)

module.exports = router;
```