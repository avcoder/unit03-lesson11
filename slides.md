---
# You can also start simply with 'default'
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: /assets/intro.jpg
# some information about your slides (markdown enabled)
title: Software Development | Foundations
info: |
  ## Software Development | Foundations
# apply unocss classes to the current slide
class: text-left
drawings:
  persist: false
transition: slide-left
mdc: true
---

# Authentication via JWT
Back-End Development - part 11/12
- [ ] JSON Web Tokens (JWT)
- [ ] Hashing Passwords
- [ ] Authenticating a User

<div class="abs-br m-6 text-xl">
  <a href="https://github.com/slidevjs/slidev" target="_blank" class="slidev-icon-btn">
    <carbon:logo-github />
  </a>
</div>

<!--
-->

---
transition: slide-left
---

# Recap

- [My Github Repo](https://github.com/avcoder/unit03-backend) of backend up to this point
- if Zoom crashes goto: https://meet.google.com/xzx-rria-pvs

---
transition: slide-left
---

# JWT (pg.1)

1. `npm i jsonwebtoken`
1. in `.env` create a new entry `JWT_SECRET=whatever` 
1. create a new file `/src/modules/auth.js`:
  ```js
  import jwt from 'jsonwebtoken'

  export const createJWT = ({id, username}) => {
    const token = jwt.sign({ id, username }, process.env.JWT_SECRET); 
    return token; // token is a string
  }
  ```
1. Let's test this so far and see what we get
1. in index.js:
  ```js
  import { createJWT } from "./modules/auth.js";

  console.log(createJWT({ id: "2", username: "al" }));
  ```

---
transition: slide-left
---

# JWT (pg.2)

1. Start creating a guard function that will protect our routes based on whether the request has the token
1. In same auth.js file:
  ```js
  export const protect = (req, res, next) => {
    // console.log(req.headers)
    const bearer = req.headers.authorization; 

    console.log("in protect, req.headers.authorization: ", bearer);
    next();
  };
  ```
1. import our protect function in `server.js` then change our existing `app.use('/api'...)`
  ```js
  import { protect } from './modules/auth.js'

  // Now, a request won't even make it to router if it can't get past protect
  app.use('/api', protect, router)
  ```
1. Let's test this in Postman by NOT sending any token to see if we get "not authorized"
1. Ask chatGPT: `using javascript's fetch method, how do you pass an authorization token in the header?`

---
transition: slide-left
---

# JWT (pg.3)
Continue modifying our protect function:

1. In same auth.js file
  ```js
  export const protect = (req, res, next) => {
    const bearer = req.headers.authorization;

    if (!bearer) {
      res.status(401);
      res.json({ message: 'not authorized' });
      return;
    }
  }
  ```
1. Let's test what we have so far. Did you get back the `401` "not authorized" message?

---
transition: slide-left
---

# JWT (pg.4)
Continue modifying our protect function:

1. Q: why did it reject again?
   - A: request was missing a Bearer token (which usually is contained in the Header)
1. So let's play hacker and put in a fake bearer token in the request
   - In Postman, underneath the url, click "Auth" tab > in dropdown choose "Bearer Token" > enter any string then test again
   - Did we pass the protect guard function?
1. Thus, we need to now check if the token is real and not fake.

---
transition: slide-left
---

# JWT (pg.5) 

1. After `if (!bearer) { }` block of code
  ```js
  const [, token] = bearer.split(" ");

  if (!token) {
    res.status(401);
    res.json({ message: 'not a valid token' });
    return;
  }

  ```
1. Test it in Postman; try passing a space character for Bearer token. Do you see "not a valid token"
- 👮‍♀️ From a security point of view, is it wise to send these detailed error messages?
- 👮‍♀️ If you were making a login form, what error messages should you display if username fails? if password fails?

---
transition: slide-left
---

# JWT (pg.6)

1. After `if(!token) { }` block of code:

  ```js
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET);
    req.user = payload;
    console.log(payload);
    next();
  } catch (e) {
    // console.error(e);
    res.status(401);
    res.json({ message: "token malformed" });
  }
  ```
1. Test it: pass any string as Bearer token. Do you see "malformed token"?

---
transition: slide-left
---


# JWT (pg. 7) 
How does the client/browser get a token in the first place?

- IF a user registers on our website, we give them a JSON web token -- that's like their identification card
- Then if they wish to login, they have to show their identification card (JWT)

BUT FIRST, we need to talk about how to store passwords:
- 👮‍♀️ Do you think it's wise to store your user's passwords in plain text, in your database?



---
layout: image
transition: slide-left
image: /assets/hashing.png
backgroundSize: 500px 400px
---

# JWT: Hashing (pg. 8)

---
layout: image
transition: slide-left
image: /assets/hacker.png
backgroundSize: 500px 400px
---

# JWT: Hashing (pg. 9)

---
transition: slide-left
---

# JWT (pg.10)

1. `npm i bcrypt`
1. In same auth.js, Let's create functions that hash our passwords:
  ```js
  import bcrypt from "bcrypt";

  export const hashPassword = (password) => {
    const SALT = 10;
    const hashedPassword = bcrypt.hash(password, SALT);
    // console.log(hashedPassword); fyi - may need to async/await to see this
    return hashedPassword;
  };
  ```
1. Test it. in server.js
   - `import { protect, hashPassword } from "./modules/auth.js";`
   - then `hashPassword("kiwi")`



---
transition: slide-left
---

# JWT (pg. 11)

1. In same auth.js, after our `hashPassword` function:
  ```js
  export const comparePasswords = (password, hash) => {
    // is plain text password the same as hashed password in db?
    const isMatch = bcrypt.compare(password, hash);
    console.log(isMatch); // may need to async/await to see this
    return isMatch;
  };
  ```
1. Test it.
   - Copy from terminal one of the hashed passwords
   - In server.js, Paste hashed password below
   ```js
    const result = comparePasswords(
      "kiwi",
      "$2b$10$NDP9ipimMkH27hGpdoHbCumZGwTmpfVmuJz82sRwR.zg2zFktWoSS"
    );
    console.log("result: ", result);
   ```
1. Did 'kiwi' match to the hashed password?
1. What happens if you just remove one character from hashed password?

---
transition: slide-left
---

# Exercise: JWT (pg.12)
Before we use our auth.js, create a new /user route

1. Create all folders/files necessary to create a new user route
1. Ensure you have completed the following:
   - `router.post("/user", userController.createUser);`
   - `/src/controllers/userController.js`
   - `/src/handlers/user.js`
   - `/src/models/user.js` 
      - should have only 2 fields: username `{ String, required, unique}`
      - and password `{ String, required }`
   - Compass should now show new `users` collection within `foodtruck` database
   - Test via Postman: see if POSTing username/password creates a new user in mongoDB (may have to remove our earlier `protect` function to test this)
1. Next - incorporate the hashing functionality...Goto next slide


---
transition: slide-left
---

# Exercise continued: JWT (pg.13)

1. In `/src/handlers/user.js`:
  ```js
  import User from "../models/user.js"; // will handle DB interactions
  import { hashPassword, comparePasswords } from "../modules/auth.js";

  const createUser = async ({ username, password }) => {
    const user = await User.create({
      username,
      password: await hashPassword(password),
    });

    return user;
  };

  export default {
    createUser,
  };
  ```
1. Test via POSTMAN: did it create a new user entry in mongoDB with the password being hashed?
1. Did the server return `201` and also a token? (click "Body" tab to the left of `201` to check) 
1. Try registering again using existing username in database?  Gracefully [handle the error](https://unit03-lesson10.netlify.app/presenter/13).

---
layout: image-right
transition: slide-left
image: /assets/bos.png
backgroundSize: 500px 300px
class: text-left
---

# 10 minute break

🍦 Cool Tips, Trends and Resources:
- 🕵️ [JWT Decoder](https://jwt.io/)
- ✌️ [Authentication Overview](https://www.loom.com/share/f8f5769ea88c4e6cb2170beee656f334)
- ✌️ [Authentication in more Detail](https://www.loom.com/share/75e6cb8654a649e3b4bee5d7f4af1ec7)

<br>
<hr>
<br>

- 🧪 [Enter anonymous lab questions](https://docs.google.com/forms/d/e/1FAIpQLSevvGARdHQikso-uLqFCO481MABKE5HofuSrlzEPMNQ2ZLykw/viewform?usp=dialog)
- ℹ️ [Course feedback survey](https://circuitstream.typeform.com/to/ZoyYk7px#course_id=SoftwareAN&instructor=9514)

<!-- 
- take attendance
-->

---
transition: slide-left
---

# JWT: Login (pg.1)
Let's create sign-in functionality

1. When users login, thereby sending username/password, how will you authenticate? (what exactly will you be comparing?)
1. Let's create a `/login` route
1. In `server.js`
  ```js
  import userController from "./controllers/userController.js";

  app.post("/user", userController.createUser);
  app.post("/login", userController.loginUser);
  ```

---
transition: slide-left
---

# JWT: Login (pg.2)

1. in `/src/controlllers/userController.js`
  ```js
  const loginUser = async (req, res) => {
    const { username, password } = req.body;
    const user = await userHandler.loginUser({ username, password });

    if (!user) {
      res.status(401);
      res.json({ message: "invalid password" });
      return;
    }

    const token = createJWT(user);
    res.status(200).json({ token });
  };
  ```

---
transition: slide-left
---

# JWT: Login (pg.3)

1. in `/src/handlers/user.js`
  ```js
  const loginUser = async ({ username, password }) => {
    const user = await User.findOne({ username });
    const isValid = await comparePasswords(password, user.password);

    return isValid ? user : null;
  };

  export default {
    createUser,
    loginUser,
  };
  ```
- Test via Postman: see if POSTing username/password via `/login` signs in a user; IF successful, did you see a token?

---
transition: slide-left
---

# JWT: Login (pg.4)

1. Let's put back our `protect` guard function. In `server.js`
  ```js
  import { protect } from "./modules/auth.js";

  app.use("/api", protect, router);
  ```
1. Test it
  - create a new user
  - login using user credentials but remember to also pass in authorization token
  - Try GETting `/api/order` -- Does it work?
  - Try GETting `/api/order` without the token?  Does it fail?



---
transition: slide-left
---

# Exercise: 
Refactor Login

Scenario: Senior leadership has decided that as their app grows, they made a mistake and now want the `/user` and `/login` route inside the `/api` route

Refactor the necessary files:
1. Ensure `/api/user` route and all related functionality works as before
1. Ensure `/api/login` route and all functionality still works as before



---
transition: slide-left
---

# EJS templates 

1. `npm i ejs`
1. Set view engine.  In `server.js` after your `app = express()` statement:
  ```js
  app.set('view engine', 'ejs');
  app.set('views', __dirname + '/views')
  ```
1. For now, create a new folder/file in `/src/views/login.ejs` and put create a basic HTML page using the following ejs variables:
  ```html
  <!DOCTYPE html>
  <html>
  <head><!-- note: we are using variables in HTML (see below). Remind you of anything we've done in the past? -->
      <title><%= title %></title> 
  </head>
  <body>
      <h1>Welcome, <%= user %>!</h1>
  </body>
  </html>
  ```
1. Pass variables from route, then start your server
  ```js
  app.get('/', (req, res) => {
      res.render('index', { title: 'Home Page', user: 'Al' });
  });
  ```

---
transition: slide-left
---

# Exercise

1. Apply what you've just learned and using our foodtruck app (or your personal app), create a login/signin page, and a register/signup page

---
transition: slide-left
---

# Homework

- Make a To-Do List App with MongoDB [see instructions in LMS](https://courses.circuitstream.com/d2l/le/lessons/9514/topics/49825)