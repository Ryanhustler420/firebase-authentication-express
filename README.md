Creating a simple Firebase authentication project with Express involves setting up a Node.js application that uses the Firebase Authentication service for user registration, login, and session management. Here are the steps to create such a project:

1.  Create a Firebase Project:

    - Go to the [Firebase Console](https://console.firebase.google.com/).
    - Create a new project or select an existing one.

2.  Enable Firebase Authentication:

    - In the Firebase Console, navigate to "Authentication" and enable the authentication methods you want to use (e.g., email/password, Google, or others).

3.  Get Firebase Configuration:

    - Go to "Project settings" > "Service accounts" > "Generate new private key". It will download the .json file which will have all the secret data. (Keep this safe)

4.  Set Up an Express Project:

    - Create a new directory for your Express project.
    - Initialize a Node.js project using npm init and install Express with npm install express.
    - Create a new JavaScript file (e.g., app.js) for your Express application.

5.  Install Firebase Admin SDK:

    - Install the Firebase Admin SDK to manage authentication on the server-side.

      ```bash
      npm install express express-session firebase-admin
      ```

### Source Code

```javascript
const express = require("express");
const session = require("express-session");
const admin = require("firebase-admin");
const serviceAccount = require("./keys.json"); // Replace with your key

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
});

const app = express();
const port = process.env.PORT || 3000;

app.use(express.json());
app.use(
  session({
    secret: "super-secret-key",
    resave: true,
    saveUninitialized: true,
  })
);

// Middleware to check if the user is authenticated
const checkAuth = (req, res, next) => {
  if (req.session.user) {
    next();
  } else {
    res.status(401).send("Unauthorized");
  }
};

app.get("/", (req, res) => {
  res.send(
    "Welcome to the Firebase Authentication and Post Management Example!"
  );
});

// Register new user
app.post("/register", (req, res) => {
  const { email, password } = req.body;
  admin
    .auth()
    .createUser({
      email,
      password,
    })
    .then((userRecord) => {
      console.log("Successfully created user:", userRecord.uid);
      res.send("User registered successfully");
    })
    .catch((error) => {
      console.error("Error creating user:", error);
      res.status(500).send("User registration failed");
    });
});

// Log in a user
app.post("/login", (req, res) => {
  const { email, password } = req.body;
  admin
    .auth()
    .getUserByEmail(email)
    .then((userRecord) => {
      // Check the provided password against the user's stored password hash
      // This part may vary depending on your application
      if (password === "admin123") {
        req.session.user = userRecord.uid;
        res.send("Login successful");
      } else {
        res.status(401).send("Login failed");
      }
    })
    .catch((error) => {
      console.error("Error getting user:", error);
      res.status(401).send("Login failed");
    });
});

// Log out a user
app.get("/logout", (req, res) => {
  req.session.user = null;
  res.send("Logged out");
});

// Create a post (requires authentication)
app.post("/posts", checkAuth, (req, res) => {
  const { title, content } = req.body;
  // Add logic to save the post, e.g., in a database
  res.send("Post created");
});

// Get all posts (requires authentication)
app.get("/posts", checkAuth, (req, res) => {
  // Add logic to retrieve all posts, e.g., from a database
  const posts = [
    { title: "Post 1", content: "Content of Post 1" },
    { title: "Post 2", content: "Content of Post 2" },
  ];
  res.json(posts);
});

app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});
```

#### Run the server

```javascript
const port = process.env.PORT || 3000;

app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});
```
