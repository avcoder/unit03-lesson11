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

---
transition: slide-left
---

# JWT (pg.2)
Need to create a bodyguard function that will protect our routes based on whether the request has the token

1. In same auth.js file
  ```js
  export const protect = (req, res) => {
    const bearer = req.headers.authorization;

    if (!bearer) {
      res.status(401);
      res.json({ message: 'not authorized' });
      return;
    }
  }
  ```
1. For now, import our protect function in `server.js` then change our existing `app.use('/api'...)`
  ```js
  import { protect } from './modules/auth.js'

  // Now, a request won't even make it to router if it can't get past protect
  app.use('/api', protect, router)
  ```
1. Let's test what we have so far. Did you get back the `401` "not authorized" message?

---
transition: slide-left
---

# JWT (pg.3)

1. Q: why did it reject again?
   - A: request was missing a Bearer token (which usually is contained in the Header)
1. So let's play hacker and put in a fake bearer token in the request
   - In Postman, underneath the url, click "Auth" tab > in dropdown choose "Bearer Token" > enter any string then test again
   - Did we pass the protect guard function?
1. Let's inspect the Bearer token in Headers on server?
  - Ask ChatGPT: `in express.js, how can i view the request's headers.  in particular, the bearer token.  I just want to console.log it`

---
transition: slide-left
---

# JWT (pg.4) 

1. We need to now check if the token is real and not fake.
1. Continuing after `if (!bearer) { }` block of code
  ```js
  const [, token] = bearer.split(" ");

  if (!token) {
    res.status(401);
    res.json({ message: 'not a valid token' });
    return;
  }
  ```

---
transition: slide-left
---



# JWT (pg.5) 


---
transition: slide-left
---

# Explore 

---
transition: slide-left
---

# asdf

---
transition: slide-left
---

# asdf

---
transition: slide-left
---

# asdf

---
transition: slide-left
---

# asdf

---
transition: slide-left
---

# Aasdf

---
transition: slide-left
---

# Exercise: 

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

# asdf


---
transition: slide-left
---

# fdsa

---
transition: slide-left
---

# asdf

---
transition: slide-left
---

# asdf

---
transition: slide-left
---

# asdf

<!--
-->

---
transition: slide-left
---

# Homework

- Make a To-Do List App with MongoDB [see instructions in LMS](https://courses.circuitstream.com/d2l/le/lessons/9514/topics/49825)