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

# Validation, Error Handling, Middleware
Back-End Development - part 10/12
- [ ] validate using `express-validator` package
- [ ] Error Handling our routes
- [ ] Middleware

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

# Mongoose Validation (pg.1)

1. Create a simple `User` model with validation rules:
    ```js
    import mongoose from "mongoose";

    const userSchema = new mongoose.Schema({
      username: {
        type: String,
        required: true,
        minlength: 3,
        maxlength: 15,
      },
      email: {
        type: String,
        required: true,
        match: /^[\\w-\\.]+@([\\w-]+\\.)+[\\w-]{2,4}$/,
      },
      age: {
        type: Number,
        min: 13,
        max: 120,
      },
    });

    export const User = mongoose.model("User", userSchema);
    ```

---
transition: slide-left
---

# Mongoose Validation (pg.2)

1. In your Express route, handle validation errors:
    ```js
    app.post("/users", async (req, res) => {
      try {
        const user = new User(req.body);
        await user.save();
        res.status(201).send(user);
      } catch (err) {
        if (err.name === "ValidationError") {
          return res.status(400).send({ error: err.message });
        }
        res.status(500).send({ error: "Something went wrong" });
      }
    });
    ```

---
transition: slide-left
---

# Express Validation (pg.1)

1. Install and import express-validator: `npm install express-validator`
    ```js
    import express from "express";
    import { body, validationResult } from "express-validator";
      ```

1. Example `/register` route:
    ```js
    router.post(
      "/register",
      [
        body("username").notEmpty().withMessage("Username is required"),
        body("email").isEmail().withMessage("Must be a valid email"),
        body("password").isLength({ min: 6 }).withMessage("Password must be at least 6 characters"),
      ],
      (req, res) => {
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
          return res.status(400).json({ errors: errors.array() });
        }
        res.send("User registered successfully!");
      }
    );
    ```

---
transition: slide-left
---

# Exercise: `express-validator` (pg.2)
Experiment the various validator methods

1. Use this base route to experiment with different validators:
```js
router.post("/test-inputs",
  [
    // Add different body() validators below
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    res.send("Validation passed!");
  }
);
```

Go to next slide to view validator methods

---
transition: slide-left
---

# Explore express-validator: Validator methods (pg.3)

- General Validators: 
   - `.notEmpty()`  field must not be empty
   - `.isLength({ min, max })`  Check string length
   - `.equals(value)`	Check if field matches a specific value
- String & Email Validators
   - `.isEmail()`	Validates email address
   - `.isAlpha()`	Letters only
   - `.isAlphanumeric()`	Letters and numbers
   - `.isLowercase() / .isUppercase()`	All lowercase or uppercase
- Number Validators
   - `.isNumeric()`	Only numbers
   - `.isInt({ min, max })`	Integer within a range
   - `.isFloat({ gt, lt })`	Float range (greater than, less than)

Goto next slide for more validators...

---
transition: slide-left
---

# Explore express-validator: Validator methods (pg.4)

- URL / Data Validators
   - `.isURL()`	Checks if it's a valid URL
   - `.isJSON()`	Must be a valid JSON string
   - `.isBoolean()`	Accepts "true"/"false"
- Advanced / Other
   - `.custom(fn)`	Your own logic (throw error or return true)
   - `.optional()`	Skip validation if field not present
   - `.exists()` Checks that a value is present in the request
   - `.isIn(array)` Validates that the value is in the provided array.
      - ex: `body("role").isIn(["admin", "user", "guest"])`
   - `.isStrongPassword([options])` Validates that a password is strong (uppercase, lowercase, number, symbol)
   - `.isDate()` Checks if the value is a valid date string.
   - `.isCreditCard()` Validates credit card number format.

---
transition: slide-left
---

# Synchronous Error Handling (pg.1)
Let's explore how to handle errors in express

1. First let's see how node handles synchronous errors
   - Create new file `index2.js` where in Line 1 you say `throw new Error('oops')`
   - run `node index2.js` - what happens?
   - If you had written a Line 2 (ex: console.log('hi')) -- would that run?
2. Now let's see how Express handles errors.  Try the following.  
   ```js
   app.get('/', (req, res) => {
    throw new Error('hey')
   })
   ```
   - Does an error show in the terminal?
   - Did it show an error in the browser?
   - Did the server crash or is still running?

Goto next slide...

---
transition: slide-left
---

# Synchronous Error Handling (pg.2)

1. Let's create an error handler then place it after your routes
   ```js
   app.use(err, req, res, next) => {
    // Above Looks like regular middleware except Only difference is the first parameter
    console.log(err);
    res.json({ message: 'error occurred' })
   }
   ```
   - Did it show an error in the browser? (compare from before)
   - Does an error show in the terminal?
   - Did the server crash or is still running?
1. Error handlers are good place to put in:
   - security and hiding stack traces
   ```js
   app.use((err, req, res, next) => {
    res.status(500).json({ message: 'Internal Server Error' });
   });
   ```
   - error logging
   - Sentry metrics and Monitoring

---
transition: slide-left
---

# Asynchronous Error Handling (pg.1)

1. Now let's see how it asynchronous errors
   ```js
   app.get('/', (req, res) => {
     setTimeout(() => {
        throw new Error('hey')
     }, 1)
   })
   ```
   - Did it show an error in the browser?
   - Does an error show in the terminal?
   - Did the server crash or is still running?

---
transition: slide-left
---

# Asynchronous Error Handling (pg.2)

1. Let's create an error handler then place it after your routes
   ```js
   app.get('/', (req, res, next) => { // only difference here is the last parameter
     setTimeout(() => {
        next(new Error('hey')) 
     }, 1)
   })
   ```
   - if you call next using any value, it'll treat it like an error
   - which will eventually be handled in your original error handler
   - Did it show an error in the browser? (compare from before)
   - Does an error show in the terminal?
   - Did the server crash or is still running?

---
transition: slide-left
---

# Asynchronous Error Handling (pg.3)

1. Practical example: use try/catch and next
   ```js
    router.get("/user/:id", async (req, res, next) => {
      try {
        const user = await User.findById(req.params.id);
        if (!user) {
          return res.status(404).send({ error: "User not found" });
        }
        res.send(user);
      } catch (err) {
        next(err); // pass the error to the middleware
      }
    });
   ```

---
transition: slide-left
---

# Exercise: Implement Error Handling

1. Using either our food truck app or your own personal app, implement error handling for all the places where synchronous/asynchronous errors could occur

---
layout: image-right
transition: slide-left
image: /assets/bos.png
backgroundSize: 500px 300px
class: text-left
---

# 10 minute break

🍦 Cool Tips, Trends and Resources:
- 🎸 [Default h1 styles are changing](https://developer.mozilla.org/en-US/blog/h1-element-styles)
- ⚡ [JSX Over the Wire](https://overreacted.io/jsx-over-the-wire/)
- 🩳 [Pocketbase](https://pocketbase.io/)
- 🔎 [SQL Noir](https://www.sqlnoir.com/)
- 🌤️ [Sunsetting Create React App](https://react.dev/blog/2025/02/14/sunsetting-create-react-app)

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

# Middleware (pg.1)

- Middleware is a list of functions to run right before your handlers do
- Use cases: logging the request in terminal, handle errors, authentication, transform request object 
- 3rd party middleware: `morgan`, `cors` `express.json()` `express.urlencoded()` `body-parser`
- Let's create our own custom middleware:
   ```js
    app.use((req, res, next) => {
       console.log("Custom middleware ran");
       next(); // important to pass control to the next middleware/route
    }
   ```
   - place above middleware before it hits our routes
   - run it. Are you able to see the log?


---
transition: slide-left
---

# Middleware (pg.2)

1. Let's try modifying the request object in our middleware
   ```js
   app.use((req, res, next) => {
      req.whatever = 'hi';
      next()
   })
   ```

1. Now in any route handler, you now should be able to see `req.whatever`
   ```js
    router.get('/order', (req, res) => {
      res.json({ message: req.whatever })
    })
   ```
- Exercise: Does this mean we can also modify the response object?  Try it.


---
transition: slide-left
---

# Middleware (pg.3)

1. Let's create our own simple version of `morgan` middleware
   ```js
    export const logger = (req, res, next) => {
      console.log(`[LOG] ${req.method} - ${req.url}`);
      next();
    };
   ```
   - Try importing your logger and replacing morgan with it 
   ```js
    import { logger } from "./logger.js";
    app.use(logger);
   ```
   - Does it work? Does it log requests in the terminal?

---
transition: slide-left
---

# Exercise: Middleware (pg.4)

1. Let's create a custom middleware that adds user info (or order info) to the request object
   ```js
    export const attachUser = (req, res, next) => {
      req.user = { id: 1, name: "Demo User" };
      next();
    };
   ```
1. Now use it:
   ```js
    app.get("/me", attachUser, (req, res) => {
      res.send(req.user);
    });
   ```
- What happens if you forget to call `next()` in middleware?
- DID YOU KNOW?  you use multiple middleware on a single route:
   ```js
   app.get("/todo/:id", myMiddleware, my2ndMiddleware, handler);
   // or
   app.get("/todo/:id", [myMiddleware, my2ndMiddleware], handler));
   ```  

Goto next slide...

---
transition: slide-left
---

# Exercise: Explore Middleware (pg.5)

1. Try installing, learning and playing with some other common middleware:
   - helmet
   - compression
   - cookie-parser
   - express-session
   - rate-limit
   - serve-static

<!--
-->

---
transition: slide-left
---

# Homework

- Make a To-Do List App with MongoDB [see instructions in LMS](https://courses.circuitstream.com/d2l/le/lessons/9514/topics/49825)