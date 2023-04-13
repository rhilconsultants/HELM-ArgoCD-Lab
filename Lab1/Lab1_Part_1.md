# HELM-ArgoCD-Lab1-Part-1

## In this lab we will create a NodeJS application and deploy it to the Openshift cluster

### Part 1 lets build our application

#### 1. create a new folder named Application (this will be our root application folder) in this folder create another folder named "src"

```Bash
mkdir Application
cd Application
mkdir src
```

#### 2. create a new file called app.js

```Bash
cd src
touch app.js
npm init
```

Fill the fields:

- package-name = hello-world
- version = 1.0.0
- description = application for workshop
- entrypoint = app.js
- all the rest leave empty

#### 3. open the file in VScode(codespaces) and create a basic web application

```js
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

```Bash
npm install express router
node app.js
```

the codespaces enviourment will notify you with a link to the Application access or you can:

```Bash
curl http://localhost:8080
```

add a .gitignore to the root folder to not sync the Node Modules folder.

```Bash
cd ..
touch .gitignore
echo "src/node_modules" >> .gitignore
```

Commit and push the new file to the Repo

```Bash
git add .
git commit -m "hello-world app"
git push
```

#### 5. now let build a Contianer for our app and push it to our quay.io image registry

i. create a Dockerfile in our Home folder

```Bash
touch Dockerfile
```

ii. Open the Dockerfile with VScode and create as the following Dockerfile

```Dockerfile
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

```Bash
docker build . -t quay.io/<quay-userName>/<imageName>:v1
```

and wait for it to finish

navigate to www.quay.io, and login with your qauy username and password

- Click on "+ Create New Repository".
- enter the image name you enter in the docker build step.
- Select public
- Select (Empty repository)
- Click "Create Public Repositoy"

iiii. push the image to quay.io registry

```Bash
$ docker login -u <userName> -p <Password> quay.io

'Login Successful'

$ docker push quay.io/<userName>/<imageName>:v1
...

pushed successfuly!
```

Add ,commit and push our changes to the Git

```Bash
git add .
git commit -m "added Dockerfile"
git push
```

## Great Jog You have Finished Part 1

### Now you can start part 2 [Here](https://github.com/rhilconsultants/HELM-ArgoCD-Lab/blob/main/Lab1/Lab1_Part_2.md)
