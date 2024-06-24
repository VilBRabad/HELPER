# TypeScript Express Server With MongoDB Helper
Edited at: 25 June 2024

## Project Setup

### Directory Structure:
```
project-root/
    ├── build
    ├── node_modules
    ├── src/
    │   ├── controllers
    │   ├── db
    │   ├── middlewares
    │   ├── models
    │   ├── routes
    │   ├── utils
    │   ├── app.ts
    │   └── index.ts
    ├── .env
    ├── .gitignore
    ├── tsconfig.json
    └── package.json
```


## Start:
### Init
```bash
yarn add -D typescript tsc-watch @types/express @types/express @types/cors

# init typescript
tsc --init
```

```json
//Some scripts
"scripts": {
    "start": "node build/index",
    "build": "tsc -p .",
    "dev": "tsc-watch --onSuccess \"npm run start\""
},
```

### Edit tsconfig.json file
1. rootDir: `"rootDir": "./src"`
2. outDir: `"outDir": "./build"`


### Other common required packages:
```bash
yarn add express body-parser cors dotenv cookie-parser
yarn add jsonwebtoken bcrypt

yarn add mongoose
```


### app.ts
```ts
import express from "express";
import cors from "cors"
import bodyParser from "body-parser";

const app = express();

app.use(cors({
    origin: process.env.CORS_ORIGIN,
    credentials: true
}))

app.use(bodyParser.urlencoded());
app.use(bodyParser.json());
app.use(express.urlencoded({extended: true, limit: "16kb"}));
app.use(express.json({limit: "16kb"}));


export {app};
```


### index.ts
```ts
import dotenv from "dotenv";
import connectDb from "./db";
import {app} from "./app"

dotenv.config();

connectDb()
.then(()=>{
    app.listen(process.env.PORT || 8000, ()=>{
        console.log("Server running...");
    })
})
.catch((error)=>{
    console.log("Database connection error: ", error);
})

```
