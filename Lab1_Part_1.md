# HELM-ArgoCD-Lab1-Part-1

## In this lab we will create a NodeJS application and deploy it to the Openshift cluster

### Part 1 lets build our application

#### 1. create a new folder named src

```
$ mkdir src
```

#### 2. create a new file called app.js

```
$ touch app.js
$ node init
```

#### 3. open the file in VScode and create a basic web application

```
const express = require('express');
const app = express();
const router = express.Router();
var port = process.env.PORT || 8080;

router.get('/', function (req, res) {
  res.send(`Hello World!`);
});

app.use('/', router);
app.listen(port);

console.log(`Running at Port ${port}`);
```

#### 4. test the application localy to see if it works

```
$ npm install express router
$ node app.js
```

```
$ curl http://localhost:8080
```

#### 5. now let build a Contianer for our app and push it to our quay.io image registry

i. create a Dockerfile in our Home folder
```
$ cd ..

$ touch Dockerfile

```
ii. Open the Dockerfile with VScode and create as the following Dockerfile
```
FROM registry.access.redhat.com/ubi8/nodejs-16

# Create app directory
WORKDIR /tmp

USER root
# copy packge.json to the workdir
COPY src/package*.json ./


# install npm dependencies
RUN npm install && npm audit fix --force

# Copy application file to the 
COPY src .

# create local env varibel for the PORT
ENV PORT 8080

USER 1001

EXPOSE 8080
CMD [ "node", "app.js" ]
```
iii. build the continer image
```
$ podman build . -t quay.io/<userName>/<imageName>:<Tag>
```
and wait for it to finish

iiii. push the image to quay.io registry
```
$ podman login -u <userName> -p <Password> quay.io

'Login Successful'

$ podman push quay.io/<userName>/<imageName>:<Tag>
...

pushed successfuly!
```


