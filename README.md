# What is twiz_oauth2?
Twiz_oauth2 is a custom OAuth2 authentication and authorization system meticulously designed for Twiz.lol. It enables users to securely authorize third-party applications to access their Twiz.lol data, fostering a seamless and controlled sharing of information. With Twiz_oauth2, users can conveniently authenticate their accounts on external platforms while maintaining control over the data they choose to share. This system prioritizes security and user control, empowering individuals to manage their data privacy across connected applications.
# How to use?
For now it only supports `nodejs expressjs` the py and flask still undergoing develoment
# Using Twiz_oauth2 with Node.js and Express.js

## 1. Install Dependencies
```js
npm install twiz_oauth2
```
## 2. Configuration 
Set up the required configurations in your Express.js application. This might include client IDs, secrets, redirect URIs, etc., provided by the Twiz.lol OAuth2 system.
## 3. Implementation
Once configured, you can integrate the OAuth2 flow within your Express.js routes:

### 1. Initiate OAuth2 Flow && Example
```js
const express = require('express');

const app = express();
const port = 3001;

const session = require('express-session');
app.use(session({
  secret: '1313131412141241412412412', // Replace with your own secret key
  resave: false,
  saveUninitialized: true
}));
const twiz_oauth2 = require('twiz_oauth2');

// Initialize Twiz_oauth2
twiz_oauth2.init({
  clientID: '',
  redirection: '/close_popup',
});
if (twiz_oauth2.url == null) return console.error('ClientId not vaild')
// Redirect users to Twiz.lol authorization page
app.get("/login", (req, res) => {
  res.redirect(`${twiz_oauth2.url()}`)
});

// The after callback function
app.get('/close_popup', (req, res) => {
  res.send("closing...<script>window.close();  window.opener.location.reload();</script>")
})


// logout
app.get('/logout',(req,res)=> {
  req.session.destroy();
  res.redirect("/");
})

// callback
app.get('/reversed_pro', twiz_oauth2.callback);

// For realtime data access
app.get("/", async(req,res)=> {
  // console.log(req.session);
  let user = req.session.user ? req.session.user.twiz.email : null;
  if (!user) {
    res.render("index.ejs", {
      user: user,
      code: twiz_oauth2.user_code(req),
    })
  } else {
  const data = await twiz_oauth2.update_user_data(req, res);
  await console.log(data);
  if (await data == "success") {
  // console.log(req.session.user);
  res.render("index.ejs",  {
    user: user,
    code: twiz_oauth2.user_code(req),
  })
} else {
  // Access may be revoked or data couldn't be updated  
  res.status(404).json("Access revoked");
}
  }
})
// For non realtime data
// app.get("/", async (req, res) => {
//   let user = req.session.user ? req.session.user.twiz.email : null;
//   res.render("index.ejs",  {
//         user: user,
//         code: twiz_oauth2.user_code(req),
//       })
// })

app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});

```
