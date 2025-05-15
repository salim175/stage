# User CRUD:

```js
const UserModel = require("../models/user");

const bcrypt = require("bcrypt")
const saltRounds = 10

//Get All users method
exports.getAllUsers = async (req, res) => {
    if (req.user && req.user.role === "admin") {
        try {
            const users = await UserModel.find({}, { password: 0 });
            // console.log('this is the req: ', req);
            res.status(200).send(users);
        } catch (error) {
            res.status(500).send("Server Error");
            //Just in devMode
            console.log(error)
            throw new Error('Failed to get classifications');
        }
    } else {
        res.status(403).send("Priviliege Error");
    }

};


//Create new user method 
exports.createUser = async (req, res) => {


    if (req.user && req.user.role === "admin") {
        const { firstname, lastname, email, pseudo, role, profileimg, password, blocked } = req.body

        try {
            const userExist = await UserModel.findOne({ email: email });

            if (!userExist) {
                const pseudoExist = await UserModel.findOne({ pseudo: pseudo });
                if (!pseudoExist) {
                    let hashedPwd = await bcrypt.hash(password, saltRounds)
                    const user = await UserModel.create({
                        firstname: firstname,
                        lastname: lastname,
                        pseudo: pseudo,
                        role: role,
                        blocked: blocked,
                        email: email,
                        profileimg: "default.png",
                        password: hashedPwd,
                    });
                    if (user) {

                        return res.status(200).send(user);

                    } else {
                        return res.status(400).send("info are not correct");
                    }
                } else {
                    return res.status(403).send("pseudo used!");
                }
            } else {
                return res.status(409).send("email used!");
            }

        } catch (error) {
            return res.status(500).send(error);
        }
    } else {
        return res.status(403).send("Privilige error");
    }

};

// Update user method
exports.updateUser = async (req, res) => {
    //user profile update case
    if (req.params.id == -1) {
        req.params.id = req.user._id;
    }

    if ((req.user && req.user.role === "admin") || (req.user && req.user._id == req.params.id)) {
        const { firstname, lastname, email, pseudo, role, oldPassword, password, blocked } = req.body

        try {
            const userExist = await UserModel.findById(req.params.id)
            let userToUpdate = userExist


            if (userExist) {
                //email change logic
                if (isNotBlank(email) && email !== userExist.email) {
                    const checkEmailExist = await UserModel.findOne({ email: email.toLowerCase() })
                    if (!checkEmailExist) {
                        userToUpdate.email = email.toLowerCase();
                    } else {
                        return res.status(409).send("email already in use");
                    }

                }
                //pseudo change logic
                if (isNotBlank(pseudo) && pseudo !== userExist.pseudo) {
                    const checkPseudoExist = await UserModel.findOne({ pseudo: pseudo })
                    if (!checkPseudoExist) {
                        userToUpdate.pseudo = pseudo;
                    } else {
                        return res.status(403).send("pseudo already in use");
                    }

                }
                // first and last name change logic
                if (isNotBlank(firstname)) {
                    userToUpdate.firstname = firstname;
                }
                if (isNotBlank(lastname)) {
                    userToUpdate.lastname = lastname;
                }
                //Password change logic
                if (isNotBlank(password)) {
                    let hashedPwd = null;
                    if (req.user.role !== "admin") {
                        //    console.log("pass change or user reached")
                        let pwdConfirm = await bcrypt.compare(oldPassword, userExist.password);
                        if (pwdConfirm) {
                            hashedPwd = await bcrypt.hash(password, saltRounds)
                        } else {
                            return res.status(401).send("old password is not correct");
                        }
                    } else {
                        hashedPwd = await bcrypt.hash(password, saltRounds)
                    }
                    if (hashedPwd != null) {
                        userToUpdate.password = hashedPwd
                    }
                }
                //Role change logic
                // console.log("role here :no" + role)
                if (isNotBlank(role) && (role === 'user' || role === 'observer' || role === 'admin') && req.user.role === "admin") {

                    userToUpdate.role = role;
                }
                if (req.user.role === "admin") {
                    userToUpdate.blocked = blocked;
                }

                const updated = await userToUpdate.save();
                if (updated) {

                    return res.status(200).send(updated);

                } else {
                    return res.status(400).send("info are not correct");
                }
            } else {
                return res.status(404).send("user not found!");
            }

        } catch (error) {
            console.log('internal: ', error)
            return res.status(500).send(error);
        }
    } else {
        return res.status(403).send("Privilige error");
    }

};

exports.deleteUser = async (req, res) => {
    if ((req.user && req.user.role === "admin")) {
        try {
            const userToDelete = await UserModel.deleteOne({ _id: req.params.id })
            if (userToDelete) {
                return res.status(201).send();
            }

        } catch (error) {
            console.log('internal: ', error)
            return res.status(500).send(error);
        }
    } else {
        return res.status(403).send("Privilige error");
    }

};

function isNotBlank(str) {
    return str != null && str.trim() !== '';
}
```

# User Routes:
```js
const router = require('express').Router();
const { getAllUsers, createUser, updateUser, deleteUser } = require('../controllers/user_controller')
const { requireSignIn, verifyUser } = require('../middlewares/auth-guard')

router.get("/all", requireSignIn, verifyUser, getAllUsers)
router.post("/add", requireSignIn, verifyUser, createUser)
router.put("/:id", requireSignIn, verifyUser, updateUser)
router.delete("/:id", requireSignIn, verifyUser, deleteUser)

module.exports = router;
```