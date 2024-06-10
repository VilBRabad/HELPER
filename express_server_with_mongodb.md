# Express Server With MongoDB Helper
Edited at: 11 June 2024

## Project Setup

### Directory Structure:
```
project-root/
    ├── node_modules
    ├── public
    ├── src/
    │   ├── controllers
    │   ├── db
    │   ├── middlewares
    │   ├── models
    │   ├── routes
    │   ├── utils
    │   ├── app.js
    │   └── index.js
    ├── .env
    ├── .gitignore
    ├── package-lock.json
    └── package.json
```

```bash
#Initialized Project
npm init -y
# Then as per directory structure as shown above make files
```

In ```package.json``` change type to  ```"type": "module"``` to use module js, By default it was commanjs(i.e. import packages by require()).

__Write Some scripts:__
```json
"scripts": {
    "start": "node index.js",
    "dev": "nodemon -r dotenv/config --experimental-json-modules src/index.js"
}
```

## Basic Packages Required:
```bash
npm i cors express dotenv mongoose nodemon body-parser cookie-parser
```

### app.js

```js
import express from "express";
import cors from "cors";
import cookieParser from "cookie-parser";

const app = express();

app.use(cors({
   origin: process.env.CORS_ORIGIN,
   credentials: true
}))

app.use(cookieParser());
app.use(express.urlencoded({extended: true, limit: "16kb"}));
app.use(express.json({limit: "16kb"}));


// Routes
import userRouter from "./routes/user.routes.js";
import basicRouter from "./routes/basic.routes.js";

app.use("/api/v1/users", userRouter);
app.use("/api/v1/basic", basicRouter);

export {app};
```


### index.js
```js
import dotenv from "dotenv";
import connectDB from "./db/index.js";
import { app } from "./app.js";

dotenv.config({
   path: "./.env"
});

connectDB()
.then(()=>{
   app.listen(process.env.PORT || 8000, ()=>{
      console.log("Server running on PORT: ", process.env.PORT || 8000);
   })
})
.catch((err)=>{
   console.log("DB connection Error, ", err);
})
```


### db/index.js
```js
import mongoose from "mongoose";
import {DB_NAME} from "../constants.js"

const connectDB = async()=>{
   try{
      const connInstance = await mongoose.connect(`${process.env.DATABASE_URI}/${DB_NAME}`);
      console.log("Database Connected !! DB Host: ", connInstance.connection.host);
   }
   catch(error){
      console.log("Error to connect database.!");
      process.exit(1);
   }   
}

export default connectDB;
```



### utils/ApiResponse.js
```js
class ApiResponse{
   constructor(statusCode, data, message="success"){
      this.statusCode = statusCode;
      this.data = data;
      this.message = message;
      this.success = statusCode < 400;
   }
}

export {ApiResponse}
```


### utils/ApiError.js
```js
class ApiError extends Error{
   constructor(
      statusCode,
      message= "Something went wrong",
      errors= [],
      stack= ""
   ){
      super(message);
      this.statusCode = statusCode;
      this.message = message;
      this.data = null;
      this.errors = errors;

      if(stack){
         this.stack = stack;
      }
      else{
         Error.captureStackTrace(this, this.constructor);
      }
   }
}

export {ApiError};
```


### utils/AsyncHandler.js
__Wrap every controller with the asyncHandler__ as best practice.
```js
const asyncHandler = (fn)=>{
    return (req, res, next)=>{
        Promise.resolve(fn(req, res, next))
        .catch((err)=> next(err));
    }
}

export {asyncHandler};
```


### Ex. user.model.js
```js
import mongoose, {Schema} from "mongoose";

const userSchema = new Schema({ 
    email: {
        type: String,
        required: true,
        unique: true,
        lowercase: true,
        trim: true
    },
    password: {
        type: String,
        required: true,
        trim: true
    },
    name: {
        type: String,
        required: true
    }
})

// *Here you can inject some methods related to User model, ex. isPasswordCorrect etc.
userSchema.methods.method_name =  async function(){
    // method logic
}

// Or Do something before inserting or other operation in database

userSchema.pre("save", async function(next){
   if(!this.isModified("password")){
      return next();
   }

//    this.password = await bcrypt.hash(this.password, 10);
    // perform operations
   return next();
})


export const User = mongoose.model("User", userSchema);
```


## CURD Operations Using Mongoose

```js
// Find
const existsUser = await User.findOne({email})
const existsUser = await User.find({})

// Find By Id
const user = await User.findById(user_Id);

// Insert new data 
const user = await User.create({
    email: email, 
    password: password
})

// Find and Update
await User.findByIdAndUpdate(
    req.user._id,
    {
        $set: {
            refreshToken: undefined
        }
    }
)
```


