Go to folder where you would like to install the node js backend

```bash
npm init -y
```

and package.json file is created..

```json
{
  "name": "server",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "module",
  "dependencies": {}
}
```

and now install the required package

```bash
npm i dotenv express mongoose cors
```

or

```bash
npm install nodemon dotenv express mongoose cors multer
```

```json
{
  "name": "server",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "nodemon index.js", // on the cmd : npm run dev will do the hot realoading for any changes server will auto start and apply the changes
    "start": "node index.js" // on the cmd : npm start will run like prod kind of running but in this case no ho reloading and for any change you need to restart the server
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "module",
  "dependencies": {
    "cors": "^2.8.5", // for cross origin interaction between server and client
    "dotenv": "^16.5.0", // for creating the environment variable
    "express": "^5.1.0", // for http server and making web api
    "mongoose": "^8.14.0", // for the mongo db schema creation and mongo db connection
    "multer": "^1.4.5-lts.2", // for image or file upload feature
    "nodemon": "^3.1.10" // for hot reloading of the node application on any changes without restarting the server
  }
}
```
