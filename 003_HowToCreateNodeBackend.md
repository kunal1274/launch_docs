# Here we will create the node js backend

1. Go to the project folder

```bash
npm init
```

or you can also use to automatically populate the details

```bash
npm init -y
```

![alt text](image-44.png)

![alt text](image-46.png)

- the package.json is created

![alt text](image-45.png)
like the below -

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
    "test": "echo \"Error: no test specified\" && exit 1"
  }
}
```

2. trying to run the server

```bash
npm start
```

![alt text](image-47.png)

- Need to install express and also configure something on index.js

```bash
npm install express
```

or

```bash
npm i express
```

![alt text](image-48.png)

![alt text](image-50.png)

3. Create and modify the index.js file to see if the node js server is running fine or not .

![alt text](image-51.png)

- index.js file content

```javascript
// In-built Node JS Modules Import
import expressAumMrigah from "express";

// 3rd-Party Node JS Modules Import

// Project FMS server related imports

console.log("This index.js file is working as expected");
const AumMrigahApp = expressAumMrigah();

const PORT = 3000;

AumMrigahApp.listen(PORT, () => {
  console.log(`The Node fms-server.1.0.0 is running at port ${PORT}`);
});
```

- run the server in the terminal

```bash
npm start
```

![alt text](image-49.png)
