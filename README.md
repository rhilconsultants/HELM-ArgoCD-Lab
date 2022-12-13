# HELM-ArgoCD-Lab1

## In this lab we will create a NodeJS application and deploy it to the Openshift cluster

### lets build our application

1. create a new folder named src

```
$ mkdir src
```

2. create a new file called app.js

```
$ touch app.js
```

3. open the file in VScode and create a basic web application

```
const express = require('express');
const app = express();
const router = express.Router();

router.get('/', function (req, res) {
  res.send(`Hello World!`);
});

app.use('/', router);
app.listen(8080);

console.log(`Running at Port ${port}`);
```

4. test the application localy to see if it works

```
$ node app.js
```

```
$ curl http://localhost:8080
```

