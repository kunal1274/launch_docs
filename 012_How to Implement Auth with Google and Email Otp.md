# 1. Implementation of the Gmail authentication

1. Install passport

```bash
npm install passport passport-google-oauth20
```

![alt text](image-173.png)

2. Create a folder named config in the project directory

- before this create the user google model

```javascript
// models/User.js
import mongoose from "mongoose";
const { Schema, model } = mongoose;

const UserGoogleSchema = new Schema(
  {
    googleId: {
      type: String,
      unique: true,
    },
    displayName: {
      type: String,
      required: true,
    },
    firstName: String,
    lastName: String,
    email: {
      type: String,
      required: true,
      unique: true,
      lowercase: true,
      trim: true,
    },
    image: String,
  },
  { timestamps: true }
);

export const UserGoogleModel =
  mongoose.models.UserGoogle || model("UserGoogle", UserGoogleSchema);
```

and we are creating the user Global model as well which will store all the users no matter it is created from Gmail or otp based ..

```javascript
// models/UserGlobal.js
import mongoose from "mongoose";
const { Schema, model } = mongoose;

const UserGlobalSchema = new Schema(
  {
    email: {
      type: String,
      required: function () {
        return !this.phoneNumber;
      },
      unique: true,
      lowercase: true,
      trim: true,
    },
    phoneNumber: {
      type: String,
      required: function () {
        return !this.email;
      },
      unique: true,
      trim: true,
      sparse: true,
    },
    name: { type: String, default: "" },
    method: {
      type: String,
      enum: ["phone", "email"],
      required: true,
      default: "email",
    },
    signInMethod: {
      type: String,
      enum: ["otp", "gmail", "both"],
      required: true,
      default: "otp",
    },
  },
  { timestamps: true }
);

export const UserGlobalModel =
  mongoose.models.UserGlobal || model("UserGlobal", UserGlobalSchema);
```

and now create the passport.js in config folder

```javascript
// config/passport.js
import passport from "passport";
import { Strategy as GoogleStrategy } from "passport-google-oauth20";
import dotenv from "dotenv";
import { UserGoogleModel } from "../models/userGoogle.model.js";
import { UserGlobalModel } from "../models/userGlobal.model.js";

dotenv.config();

// Configure the Google OAuth strategy
passport.use(
  new GoogleStrategy(
    {
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL: process.env.GOOGLE_CALLBACK_URL,
    },
    async (accessToken, refreshToken, profile, done) => {
      try {
        // Try to find an existing user by Google ID
        let user = await UserGoogleModel.findOne({ googleId: profile.id });
        let userGlobal = null;
        // let userGlobal = await UserGlobalModel.findOne({
        //   email: profile.emails[0].value,
        // });

        if (!user) {
          // If no user exists, create a new one using data from Google profile
          user = await UserGoogleModel.create({
            googleId: profile.id,
            displayName: profile.displayName,
            firstName: profile.name.givenName,
            lastName: profile.name.familyName,
            email: profile.emails[0].value,
            image: profile.photos[0].value,
          });
        }
        userGlobal = await UserGlobalModel.findOne({
          email: user.email,
        });

        if (!userGlobal) {
          await UserGlobalModel.create({
            email: profile.emails[0].value,
            method: "email",
            signInMethod: "gmail",
          });
        }
        return done(null, user);
      } catch (error) {
        return done(error, null);
      }
    }
  )
);

// Serialize user ID to store in the session
passport.serializeUser((user, done) => {
  done(null, user.id);
});

// Deserialize the user from the session by ID
passport.deserializeUser(async (id, done) => {
  try {
    const user = await UserGoogleModel.findById(id);
    done(null, user);
  } catch (error) {
    done(error, null);
  }
});

export default passport;
```

3. Now we will create the controllers dedicated to google authentication

```javascript
// controllers/userGoogle.controller.js

import jwt from "jsonwebtoken";
import dotenv from "dotenv";

dotenv.config();

export const googleAuthCallback = (req, res) => {
  // `req.user` is the user object from passport deserialize
  if (!req.user) {
    return res.redirect(
      `${process.env.FRONTEND_URL}/auth/google/callback?error=NoUser`
    );
  }
  // Generate JWT
  const payload = {
    googleId: req.user.googleId,
    email: req.user.email,
  };
  //const token = jwt.sign(payload, process.env.JWT_SECRET, { expiresIn: "1h" });
  const token = jwt.sign(payload, process.env.JWT_SECRET, { expiresIn: "5m" });

  // Redirect to frontend with token as a query param
  const redirectUrl = `${
    process.env.FRONTEND_URL
  }/auth/google/callback?token=${token}&email=${encodeURIComponent(
    req.user.email
  )}`;
  res.redirect(redirectUrl);
};

export const googleAuthCallback1 = (req, res) => {
  // Successful authentication – you can redirect to your dashboard or send a JSON response.
  console.log("Successful google login is done for user", req.user);
  res.redirect(`${process.env.FRONTEND_URL}/auth/google/callback`); // Change this route as needed.
};

export const googleAuthFailure = (req, res) => {
  res.status(401).json({ message: "Google authentication failed" });
};
```

4. Now the routes is required

```javascript
// routes/google-auth.routes.js
import express from "express";
import passport from "../config/passport.js";
import {
  googleAuthCallback,
  googleAuthFailure,
} from "../controllers/userGoogle.controller.js";

const googleAuthRouter = express.Router();

// Route to start Google authentication. Request profile and email scopes.
googleAuthRouter.get(
  "/google",
  passport.authenticate("google", { scope: ["profile", "email"] })
);

// Callback route that Google will redirect to after authentication.
googleAuthRouter.get(
  "/google/callback",
  passport.authenticate("google", {
    failureRedirect: "/auth/google/failure",
  }),
  googleAuthCallback
);

// Failure route
googleAuthRouter.get("/google/failure", googleAuthFailure);

export default googleAuthRouter;
```

and the alternative google auth to find remember me or auto login

```javascript
// src/routes/apiAuthRoutes.js
import express from "express";

const googleAlternativeApiAuthRouter = express.Router();

// This endpoint returns the authenticated user details.
// It relies on Passport to populate req.user if the user is logged in.
googleAlternativeApiAuthRouter.get("/me", (req, res) => {
  if (req.isAuthenticated && req.isAuthenticated()) {
    // Return the user data (you might want to sanitize it before sending)
    res.status(200).json({ user: req.user });
  } else {
    res.status(401).json({ message: "Unauthorized" });
  }
});

export default googleAlternativeApiAuthRouter;
```

5. Now the change in index.js

```javascript
import dotenv from "dotenv";
dotenv.config(); // Loads .env into process.env

// In-built Node JS Modules Import
import expressAumMrigah from "express";

// 3rd-Party Node JS Modules Import
import cors from "cors"; // new2
import morgan from "morgan";
import helmet from "helmet";
import xss from "xss-clean";
import rateLimit from "express-rate-limit";
import hpp from "hpp";
import ExpressMongoSanitize from "express-mongo-sanitize";

// Project FMS server related imports
import userGroupRouter from "./routes/userGroupRoutes.js";
import userRouter from "./routes/userRoutes.js";
import connectToDb from "./database/mongoDb.js";
import { companyRouter } from "./routes/company.routes.js";
import { customerRouter } from "./routes/customer.routes.js";
import { itemRouter } from "./routes/item.routes.js";
import { salesOrderRouter } from "./routes/salesorder.routes.js";
import { vendorRouter } from "./routes/vendor.routes.js";
import { purchaseOrderRouter } from "./routes/purchaseorder.routes.js";
import logger from "./utility/logger.util.js";
import { requestTimer } from "./middleware/requestTimer.js";
import googleAuthRouter from "./routes/google-auth.routes.js"; // ADDED
import googleAlternativeApiAuthRouter from "./routes/api-auth.routes.js"; // ADDED

// Environment variables
const PORT = process.env.PORT || 3000;

console.log("This index.js file is working as expected");

// Middleware
const AumMrigahApp = expressAumMrigah();

AumMrigahApp.use(expressAumMrigah.json());

//// Security middleware
AumMrigahApp.use(helmet()); // Secure HTTP headers
AumMrigahApp.use(xss()); // Prevent XSS
AumMrigahApp.disable("x-powered-by"); // Hide the X-Powered-By header
AumMrigahApp.use(ExpressMongoSanitize());
AumMrigahApp.use(hpp());

// Rate Limiter
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: "Too many requests from this IP. Please try again later.",
});
//AumMrigahApp.use("/fms/api", limiter); // specific to router
AumMrigahApp.use(limiter); // to everything

const allowedOrigins = process.env.ALLOWED_ORIGINS
  ? process.env.ALLOWED_ORIGINS.split(",").map((ele) => {
      return ele.trim();
    })
  : [];

console.log("Allowed Origins", process.env.ALLOWED_ORIGINS);

const corsOptions = {
  origin: (origin, callback) => {
    // Allow requests with no origin (for Postman, mobile apps)
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error("Not allowed by CORS"));
    }
  },
  methods: ["GET", "POST", "PUT", "PATCH", "DELETE"],
  credentials: true, // Allow cookies
};

AumMrigahApp.use(cors(corsOptions));

AumMrigahApp.use(requestTimer); // 💥 log for all routes

// we are using after the request processed through json and cors
// Define a stream for morgan to use Winston
const stream = {
  write: (message) => logger.http(message.trim()),
};

AumMrigahApp.use(morgan("combined", { stream }));

// Routes
AumMrigahApp.get("/", (req, res) => {
  res.send(`Hello from Express on Render at Port number ${PORT}!`);
});

AumMrigahApp.use("/fms/api/v0/users", userRouter);
AumMrigahApp.use("/fms/api/v0/userGroups", userGroupRouter);
AumMrigahApp.use("/fms/api/v0/customers", customerRouter);
AumMrigahApp.use("/fms/api/v0/vendors", vendorRouter);
AumMrigahApp.use("/fms/api/v0/items", itemRouter);
AumMrigahApp.use("/fms/api/v0/companies", companyRouter);
AumMrigahApp.use("/fms/api/v0/salesorders", salesOrderRouter);
AumMrigahApp.use("/fms/api/v0/purchaseorders", purchaseOrderRouter);

AumMrigahApp.use("/auth", googleAuthRouter); // added
AumMrigahApp.use("/api/auth", googleAlternativeApiAuthRouter); // added

AumMrigahApp.get("/env", (req, res) => {
  res.json({ allowedOrigins });
});

// Global error handler (optional but recommended)
AumMrigahApp.use((err, req, res, next) => {
  logger.error("Global Error Handler", { error: err });
  res.status(500).send({
    status: "failure",
    message: "An unexpected error occurred from the Backend for launch-app-fms",
  });
});

// final route
AumMrigahApp.use((req, res) => {
  res
    .status(400)
    .send(
      `This is final and invalid path coming from node js backend launch-app-fms`
    );
});

const startServer = async () => {
  try {
    await connectToDb();
    AumMrigahApp.listen(PORT, () => {
      console.log(
        `The Node Launch FMS backend server 1.0.0 has been now running at ${PORT} with the cloud Mongo db`
      );
    });
  } catch (error) {
    console.error(`Server is unable to start due to some error : ${error}`);
    process.exit(1);
  }
};

startServer();
```

# 2. Implementation of the OTP Authentication

### 1. JWT Authentication

a. Install necessary json web token

```bash
npm install jsonwebtoken
```

![alt text](image-174.png)

### 2. OTP Authentication

- a. crypto installation for generating otp ( now it's built in module )
- b. nodemailer installation for sending otp to email

![alt text](image-175.png)

the follwoing is the latest package.json after all the above installation

```json
{
  "name": "server",
  "version": "1.0.0",
  "description": "server for fms backend",
  "license": "ISC",
  "author": "Ratxen Solutions Private Limited",
  "type": "module",
  "main": "index.js",
  "keywords": [],
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "nodemon index.js",
    "start": "node index.js"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "dotenv": "^16.4.7",
    "express": "^4.21.2",
    "express-mongo-sanitize": "^2.2.0",
    "express-rate-limit": "^7.5.0",
    "helmet": "^8.1.0",
    "hpp": "^0.2.3",
    "jsonwebtoken": "^9.0.2", // ADDED
    "mongoose": "^8.12.2",
    "morgan": "^1.10.0",
    "nodemailer": "^6.10.0", // ADDED
    "passport": "^0.7.0", // ADDED
    "passport-google-oauth20": "^2.0.0", // ADDED
    "redis": "^4.7.0",
    "stack-trace": "^1.0.0-pre2",
    "winston": "^3.17.0",
    "winston-daily-rotate-file": "^5.0.0",
    "xss-clean": "^0.1.4"
  },
  "devDependencies": {
    "nodemon": "^3.1.9"
  }
}
```

- we are making one helper function generate Otp

```javascript
// utils/generateOtp.js
// const crypto = require("crypto");
import crypto from "crypto";

/**
 * Generates an OTP based on the specified type and length.
 * @param {String} type - Type of OTP: 'numeric', 'alphanumeric', 'alphanumeric_special'
 * @param {Number} length - Length of the OTP
 * @returns {String} - Generated OTP
 */
function generateOtp(type = "numeric", length = 6) {
  let characters = "";

  switch (type) {
    case "numeric":
      characters = "0123456789";
      break;
    case "alphanumeric":
      characters =
        "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
      break;
    case "alphanumeric_special":
      characters =
        "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*()_+[]{}|;:,.<>?";
      break;
    default:
      throw new Error("Invalid OTP type specified.");
  }

  let otp = "";
  const charactersLength = characters.length;
  for (let i = 0; i < length; i++) {
    const randomByte = crypto.randomBytes(1)[0];
    otp += characters.charAt(randomByte % charactersLength);
  }

  return otp;
}

export default generateOtp;
```

- now lets create the otp model

```javascript
// models/Otp.js
import mongoose from "mongoose";
const { Schema, model } = mongoose;

const UserOtpSchema = new mongoose.Schema(
  {
    phoneNumber: {
      type: String,
      required: function () {
        return !this.email;
      },
      trim: true,
    },
    email: {
      type: String,
      required: function () {
        return !this.phoneNumber;
      },
      trim: true,
      lowercase: true,
    },
    otp: {
      type: String,
      required: true,
    },
    method: {
      type: String,
      enum: ["whatsapp", "sms", "email"],
      required: true,
    },
    otpType: {
      type: String,
      enum: ["numeric", "alphanumeric", "alphanumeric_special"],
      required: true,
    },
    expiresAt: {
      type: Date,
      required: true,
      default: () => Date.now() + 5 * 60 * 1000, // otp expires in 5 mins
    },
    createdAt: { type: Date, default: Date.now(), expires: 300 }, // Document expires after 300 seconds eqv to 5 mins
  },
  {
    timestamps: true,

    toJSON: {
      virtuals: true,
      transform: function (doc, ret) {
        // Remove __v if you wish
        //delete ret.__v;
        // Sort keys alphabetically for easier reading
        const sorted = {};
        Object.keys(ret)
          .sort()
          .forEach((key) => {
            sorted[key] = ret[key];
          });
        return sorted;
      },
    },
    toObject: { virtuals: true },
  }
);

UserOtpSchema.index({ email: 1 }, { unique: true });
UserOtpSchema.index({ phoneNumber: 1 }, { unique: true });
// module.exports = mongoose.model("Otp", OtpSchema);
export const UserOtpModel =
  mongoose.models.UserOtp || model("UserOtp", UserOtpSchema);
```

- and now the userotp controller

```javascript
// controllers/authController.js
import nodemailer from "nodemailer";
import { UserOtpModel } from "../models/userOtp.model.js";
import generateOtp from "../utility/generateOtp.utils.js"; // Assumes you have an OTP generator function
import { winstonLogger } from "../utility/logError.utils.js";
import jwt from "jsonwebtoken";
import { UserGlobalModel } from "../models/userGlobal.model.js";

// Nodemailer configuration for sending emails
const transporter = nodemailer.createTransport({
  service: "gmail",
  auth: {
    user: process.env.EMAIL_USER,
    pass: process.env.EMAIL_PASS,
  },
  debug: true,
  logger: true,
});

/**
 * sendOtp - Controller to generate and send an OTP via WhatsApp, SMS, or email.
 * Expects in the request body:
 *  - phoneNumber and/or email,
 *  - method: one of "whatsapp", "sms", or "email",
 *  - otpType (optional, defaults to "numeric"),
 *  - otpLength (optional, defaults to 6).
 */

export const sendOtp = async (req, res) => {
  const { phoneNumber, email, method, otpType, otpLength } = req.body;

  // Validate required inputs
  if (!method || (!phoneNumber && !email)) {
    winstonLogger.error("Missing identifier or method", {
      phoneNumber,
      email,
      method,
    });
    return res.status(400).json({
      msg: "Phone number or email is required, and method must be specified",
    });
  }

  // For email method, ensure the email is not already registered in the User model.
  if (method === "email" && email) {
    const existingGlobalUser = await UserGlobalModel.findOne({
      email: email.toLowerCase().trim(),
    });

    // const existingUser = await UserModel.findOne({
    //   email: email.toLowerCase().trim(),
    // });
    const existingUserEmailInOtpModel = await UserOtpModel.findOne({
      email: email.toLowerCase().trim(),
    });
    if (existingUserEmailInOtpModel) {
      await UserOtpModel.deleteOne({
        email: email.toLowerCase().trim(),
      });
    }

    if (existingGlobalUser) {
      console.log(
        `Found the Global user ${existingGlobalUser}.This email could be already registered. Please use your registered login method.Try with Google Sign in.`
      );
    } else {
      console.log(
        `Not Found the user with email ${email}. We will send otp now and this registration is otp based registration`
      );
    }

    // if (existingUser) {
    //   return res.status(400).json({
    //     msg: "This email is already registered. Please use your registered login method.Try with Google Sign in.",
    //   });
    // }
  }

  if (!["whatsapp", "sms", "email"].includes(method)) {
    return res
      .status(400)
      .json({ msg: "Invalid method. Choose from whatsapp, sms, or email." });
  }
  if (
    otpType &&
    !["numeric", "alphanumeric", "alphanumeric_special"].includes(otpType)
  ) {
    return res.status(400).json({
      msg: "Invalid OTP type. Choose from numeric, alphanumeric, or alphanumeric_special.",
    });
  }

  const finalOtpType = otpType || "numeric";
  const finalOtpLength = otpLength || 6;
  const otp = generateOtp(finalOtpType, finalOtpLength);
  winstonLogger.info("Generated OTP", { otp });

  try {
    // Build query for existing OTP (if any) based on the method
    let query = { otp };
    if ((method === "whatsapp" || method === "sms") && phoneNumber) {
      query.phoneNumber = phoneNumber;
    }
    if (method === "email" && email) {
      query.email = email;
    }

    // Remove any previous OTP matching the query
    await UserOtpModel.findOneAndDelete(query);

    // Save the new OTP record
    const newOtpRecord = await UserOtpModel.create({
      phoneNumber: phoneNumber || null,
      email: email || null,
      otp,
      method,
      otpType: finalOtpType,
    });
    winstonLogger.info("OTP saved to database", { newOtpRecord });

    // Send the OTP based on method
    if (method === "whatsapp" && phoneNumber) {
      await client.messages.create({
        body: `Your OTP is: ${otp}`,
        from: whatsappFrom,
        to: `whatsapp:${phoneNumber}`,
      });
      winstonLogger.info("OTP sent via WhatsApp", { phoneNumber });
      return res
        .status(200)
        .json({ msg: "OTP sent via WhatsApp successfully" });
    } else if (method === "sms" && phoneNumber) {
      await client.messages.create({
        body: `Your OTP is: ${otp}`,
        from: smsFrom,
        to: phoneNumber,
      });
      winstonLogger.info("OTP sent via SMS", { phoneNumber });
      return res.status(200).json({ msg: "OTP sent via SMS successfully" });
    } else if (method === "email" && email) {
      const mailOptions = {
        from: process.env.EMAIL_USER,
        to: email,
        subject: "Your OTP Code",
        text: `Your OTP is: ${otp}`,
        html: `<b>Your OTP is: ${otp}</b>`,
      };
      await transporter.sendMail(mailOptions);
      winstonLogger.info("OTP sent via Email", { email });
      return res.status(200).json({ msg: "OTP sent via email successfully" });
    } else {
      return res
        .status(400)
        .json({ msg: "Invalid method or missing phone number/email" });
    }
  } catch (err) {
    winstonLogger.error("Error in sendOtp", { error: err });
    return res.status(500).json({ msg: "Server Error" });
  }
};

/**
 * verifyOtp - Controller to verify a provided OTP.
 * Expects in the request body:
 *  - phoneNumber and/or email,
 *  - otp.
 */

export const verifyOtp = async (req, res) => {
  const { phoneNumber, email, otp } = req.body;
  console.log("line 279 verifyOtpController.js ", req.body);
  if ((!phoneNumber && !email) || !otp) {
    return res
      .status(400)
      .json({ msg: "Phone number or email and OTP are required" });
  }
  try {
    let query = { otp };
    if (phoneNumber) query.phoneNumber = phoneNumber;
    if (email) query.email = email;
    const existingGlobalUser = await UserGlobalModel.findOne({
      email: email.toLowerCase().trim(),
    });

    const otpRecord = await UserOtpModel.findOne(query);
    if (!otpRecord) {
      return res.status(400).json({ msg: "Invalid or expired OTP" });
    }

    // Check expiration
    if (otpRecord.expiresAt < Date.now()) {
      await UserOtpModel.deleteOne({ _id: otpRecord._id });
      return res.status(400).json({ msg: "OTP has expired" });
    }

    // Generate JWT token (payload can be customized)
    //const payload = { email, phoneNumber };
    const payload = {
      //userId: user._id,
      email: email,
      phoneNumber: phoneNumber,
    };

    const token = jwt.sign(payload, process.env.JWT_SECRET, {
      expiresIn: "5m", // currently jwt token expires within 5 mins
    });
    console.log("line 354 in user otp controller and token is ", token);

    if (!existingGlobalUser) {
      await UserGlobalModel.create({
        email,
        phoneNumber,
        method: email ? "email" : "phone",
        signInMethod: "otp",
      });
    }

    // OTP is valid; delete it and respond
    await UserOtpModel.deleteOne({ _id: otpRecord._id });

    return res
      .status(200)
      .json({ msg: "OTP verified successfully", token: token });
  } catch (err) {
    winstonLogger.error("Error in verifyOtp", { error: err });
    return res.status(500).json({ msg: "Server Error" });
  }
};
```

- and the routes are -

```javascript
// routes/authRoutes.js
import express from "express";
import { sendOtp, verifyOtp } from "../controllers/userOtp.controller.js";
import { authenticateJWT } from "../middleware/authJwtHandler.js";

const otpAuthRouter = express.Router();

// Route to send OTP
otpAuthRouter.post("/send-otp", sendOtp);

// Route to verify OTP
otpAuthRouter.post("/verify-otp", verifyOtp);

// Validate token route
otpAuthRouter.get("/me", authenticateJWT, (req, res) => {
  // If token is valid, req.user is set by the authenticateJWT middleware
  // Return user info or a success message
  res.status(200).json({ msg: "Token is valid", user: req.user });
});

export default otpAuthRouter;
```

- and the final index.js file is

```javascript
import dotenv from "dotenv";
dotenv.config(); // Loads .env into process.env

// In-built Node JS Modules Import
import expressAumMrigah from "express";

// 3rd-Party Node JS Modules Import
import cors from "cors"; // new2
import morgan from "morgan";
import helmet from "helmet";
import xss from "xss-clean";
import rateLimit from "express-rate-limit";
import hpp from "hpp";
import ExpressMongoSanitize from "express-mongo-sanitize";

// Project FMS server related imports
import userGroupRouter from "./routes/userGroupRoutes.js";
import userRouter from "./routes/userRoutes.js";
import connectToDb from "./database/mongoDb.js";
import { companyRouter } from "./routes/company.routes.js";
import { customerRouter } from "./routes/customer.routes.js";
import { itemRouter } from "./routes/item.routes.js";
import { salesOrderRouter } from "./routes/salesorder.routes.js";
import { vendorRouter } from "./routes/vendor.routes.js";
import { purchaseOrderRouter } from "./routes/purchaseorder.routes.js";
import logger from "./utility/logger.util.js";
import { requestTimer } from "./middleware/requestTimer.js";
import googleAuthRouter from "./routes/google-auth.routes.js";
import otpAuthRouter from "./routes/otp-auth.routes.js"; // ADDED
import googleAlternativeApiAuthRouter from "./routes/api-auth.routes.js";

// Environment variables
const PORT = process.env.PORT || 3000;

console.log("This index.js file is working as expected");

// Middleware
const AumMrigahApp = expressAumMrigah();

AumMrigahApp.use(expressAumMrigah.json());

//// Security middleware
AumMrigahApp.use(helmet()); // Secure HTTP headers
AumMrigahApp.use(xss()); // Prevent XSS
AumMrigahApp.disable("x-powered-by"); // Hide the X-Powered-By header
AumMrigahApp.use(ExpressMongoSanitize());
AumMrigahApp.use(hpp());

// Rate Limiter
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: "Too many requests from this IP. Please try again later.",
});
//AumMrigahApp.use("/fms/api", limiter); // specific to router
AumMrigahApp.use(limiter); // to everything

const allowedOrigins = process.env.ALLOWED_ORIGINS
  ? process.env.ALLOWED_ORIGINS.split(",").map((ele) => {
      return ele.trim();
    })
  : [];

console.log("Allowed Origins", process.env.ALLOWED_ORIGINS);

const corsOptions = {
  origin: (origin, callback) => {
    // Allow requests with no origin (for Postman, mobile apps)
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error("Not allowed by CORS"));
    }
  },
  methods: ["GET", "POST", "PUT", "PATCH", "DELETE"],
  credentials: true, // Allow cookies
};

AumMrigahApp.use(cors(corsOptions));

AumMrigahApp.use(requestTimer); // 💥 log for all routes

// we are using after the request processed through json and cors
// Define a stream for morgan to use Winston
const stream = {
  write: (message) => logger.http(message.trim()),
};

AumMrigahApp.use(morgan("combined", { stream }));

// Routes
AumMrigahApp.get("/", (req, res) => {
  res.send(`Hello from Express on Render at Port number ${PORT}!`);
});

AumMrigahApp.use("/fms/api/v0/users", userRouter);
AumMrigahApp.use("/fms/api/v0/userGroups", userGroupRouter);
AumMrigahApp.use("/fms/api/v0/customers", customerRouter);
AumMrigahApp.use("/fms/api/v0/vendors", vendorRouter);
AumMrigahApp.use("/fms/api/v0/items", itemRouter);
AumMrigahApp.use("/fms/api/v0/companies", companyRouter);
AumMrigahApp.use("/fms/api/v0/salesorders", salesOrderRouter);
AumMrigahApp.use("/fms/api/v0/purchaseorders", purchaseOrderRouter);

AumMrigahApp.use("/auth", googleAuthRouter);
AumMrigahApp.use("/api/auth", googleAlternativeApiAuthRouter);
AumMrigahApp.use("/fms/api/v0/otp-auth", otpAuthRouter); // ADDED

AumMrigahApp.get("/env", (req, res) => {
  res.json({ allowedOrigins });
});

// Global error handler (optional but recommended)
AumMrigahApp.use((err, req, res, next) => {
  logger.error("Global Error Handler", { error: err });
  res.status(500).send({
    status: "failure",
    message: "An unexpected error occurred from the Backend for launch-app-fms",
  });
});

// final route
AumMrigahApp.use((req, res) => {
  res
    .status(400)
    .send(
      `This is final and invalid path coming from node js backend launch-app-fms`
    );
});

const startServer = async () => {
  try {
    await connectToDb();
    AumMrigahApp.listen(PORT, () => {
      console.log(
        `The Node Launch FMS backend server 1.0.0 has been now running at ${PORT} with the cloud Mongo db`
      );
    });
  } catch (error) {
    console.error(`Server is unable to start due to some error : ${error}`);
    process.exit(1);
  }
};

startServer();
```

# 3. environment variable should be also changed

```.env

COMPANY_PREFIX=RX
FRONTEND_URL=http://localhost:5173
EMAIL_USER=adhikariratxen@gmail.com
EMAIL_PASS=fkclmsoibzfhnzsw
JWT_SECRET=launching_namami_secret_with_mUshakaH_approach
SESSION_SECRET=your_session_secret
GOOGLE_CLIENT_ID=577653957083-sjhi667j2l6fp3l9r2j7f0qr3saban59.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=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
GOOGLE_CALLBACK_URL=http://localhost:5050/auth/google/callback
PORT=5050
MONGO_URI=mongodb://localhost:27017/fmsdevdb
DATABASE_USERNAME=devratxen
DATABASE_PASSWORD=zLMCvTgWJh2pcxMK
PROJECT_NAME=scalernodebackend2
DATABASE_NAME=fms-cloud-local-dev-db
APP_NAME=ScalerNodeBackend2
ATLAS_URI = mongodb+srv://devratxen:zLMCvTgWJh2pcxMK@scalernodebackend2.pnctyau.mongodb.net/?retryWrites=true&w=majority&appName=ScalerNodeBackend2
REDIS_HOST=redis-12859.c264.ap-south-1-1.ec2.redns.redis-cloud.com
REDIS_PORT=12859
REDIS_PASSWORD=fpuJQ9eiL3rbWQio0uDZPhJUhF0a8AT6
REDIS_USERNAME=default
REDIS_USE_TLS=true
ALLOWED_ORIGINS=http://localhost:5173,http://localhost:5174,http://localhost:5175,https://namami-fe.vercel.app,https://www.postman.com,https://jiodriversprod1.vercel.app

```
