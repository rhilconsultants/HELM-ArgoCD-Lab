# HELM-ArgoCD-Lab1-Part-2

## In this lab we will create a NodeJS application and deploy it to the Openshift cluster

### Part 2 lets Deploy our Application using YAML Files and see if it works

#### 1. create a new folder named yaml in the repo root folder

```Bash
mkdir yaml
```

#### 2. create a new files called deployment.yaml route.yaml service.yaml

```Bash
cd yaml
touch deployment.yaml route.yaml service.yaml
```

#### 3. open the deployment.yaml file in VScode(Codespaces) and create a deployment menifast, Dont forget to edit with your Lab UserName (From the table), and the image name and tag from quay

```YAML
kind: Deployment
apiVersion: apps/v1
metadata:
# set your lab user name
  name: <userName>-hello-world
spec:
  replicas: 3
  selector:
    matchLabels:
# set your lab user name
      app: <userName>-hello-world
  template:
    metadata:
      labels:
# set your lab user name
        app: <userName>-hello-world
    spec:
      containers:
        - resources: {}
          terminationMessagePath: /dev/termination-log
          name: hello-world
          ports:
            - containerPort: 8080
              protocol: TCP
          imagePullPolicy: IfNotPresent
# update with the image you build in part 1
          image: 'quay.io/<quay-userName>/<imageName>:<Tag>'
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
```

#### 4. open the service.yaml file in VScode and create a route menifast, Dont forget to edit with your lab UserName

```YAML
kind: Service
apiVersion: v1
metadata:
# set your user name
  name: <userName>-hello-world
spec:
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  selector:
# set your user name
    app: <userName>-hello-world
```

#### 5. open the route.yaml file in VScode and create a route menifast, Dont forget to edit with your lab UserName

```YAML
kind: Route
apiVersion: route.openshift.io/v1
metadata:
# set your user name
  name:  <userName>-hello-world
spec:
  to:
    kind: Service
# set your user name
    name:  <userName>-hello-world
    weight: 100
  port:
    targetPort: 8080
  wildcardPolicy: None
```

add ,commit and push the new files to the git repo

```bash
git add.
git commit -m "added k8s menifasts"
git push
```

#### 6. Lets Deploy our application to the Cluster with ArgoCD

i. Open your ArgoCD instance via the link found in the Dashboard

> 1. Create a new Application.
> 2. enter your Application name as the Following "{lab-userName}-hello-world".
> 3. Enter your GitHub repo Url.
> 4. select the branch that you are working on (i.e "main").
> 5. select the "Application/yaml/" folder.
> 6. selcet the "in-cluster / 'https://kubernetes.default.svc'"
> 7. enter your application namespace - "user{number}-application".
> 8. under sync policy, leave it manual for now.
> 9. Click on create.

and wait for the applicaion will show in the UI

ii. click on the application and and then click the "Sync" button ,and wait for the application to deploy to the cluster

iii. try to find the URL for the application and access it from your broswer.(some times due to UI issue in ArgoCD the Route will dissapear, so go to the Openshift Console ,Developer, Topology and click the Arrow near the Application.)

- go to the route box in the application diagram in the argocd UI.
- Click on it and look at the Live menifast

it should look like that:

```YAML
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  annotations:
    openshift.io/host.generated: 'true'
  labels:
    app.kubernetes.io/instance: user*-hello-world
  name: user*-hello-world
  namespace: user1-application
spec:
  host: >-
    user*-hello-world-user*-application.apps.cluster-dpzqs.dpzqs.sandbox2119.opentlc.com
  port:
    targetPort: 8080
  to:
    kind: Service
    name: user*-hello-world
    weight: 100
  wildcardPolicy: None
```

- find the 'host:' key and copy the url to the browser, with "http://" before the url.

we should see our web site
![hello-world](https://github.com/rhilconsultants/Application-Deployment-Workshop/blob/main/Class%20artifacts/lab1-part-1-web.png)

### 7. now that we have our application running on our Openshift Cluster, now we will create a HTML web and serve it from our Deployment

1. return to our src folder and create a new folder named html inside it.

      ```Bash
      cd src
      mkdir html
      cd html
      touch index.html
      ```

2. Open the index.html and create the following html web site.

  ```html
  <html>
  <head>
    <title>K8S Application</title>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
    <link rel="stylesheet"   href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.1/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.1/css/bootstrap-theme.min.css">
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.1/js/bootstrap.min.js"></script>
  </head>
  <body>
  </div>
    <div style="margin:100px;">
     
  <class="navbar navbar-inverse navbar-static-top">
    <div class="container">
      <a class="navbar-brand" href="/">K8S Application</a>
      <ul class="nav navbar-nav">
        <li class="active">
          <a href="/">Home</a>
        </li>
        <li>
          <a href="/health/liveliness">Check liveliness</a>
        </li>
        <li>
          <a href="/health/readiness">Check readiness</a>
        </li>
      </ul>
    </div>
  </nav>
        <div class="jumbotron"  style="padding:40px;">
          <img   src="https://developers.redhat.com/sites/default/files/styles/article_feature/public/blog/2018/05/openshift-featured.png?itok=g0Ee8H1H" alt="OpenShift">
          <h1>Hello, world!</h1>
          <h2>This is a simple hello World Web Page, this message will be modifed.</h2>
      </div>
    </body>
    </html>
  ```

- add ,commit and push our new file.

    ```Bash
    git add .
    git commit -m "a html file"
    git push
    ```

- update our NodeJS with this Application, Open app.js File and replace with this new code block

  ```js
  const express = require('express');
  const app = express();
  const path = require('path');
  const router = express.Router();
  var port = process.env.PORT || 8080;
  
  
  // Default values for init status values
  var liveliness_new = 200
  var readiness_new = 200
  
  
  // Route application to index.html file.
  router.get('/',function(req,res){
    res.sendFile(path.join(__dirname+'/html/index.html'));
    //__dirname : It will resolve to your project folder.
  });
  
  // Health Probe - Application Liveliness
  router.get('/health/liveliness',function(req,res){
      console.log(`code ----> ${liveliness_new}`)
      res.status(parseInt(liveliness_new))
        if (liveliness_new > 399){
          res.send('Not Healthy')
        }
        else{ res.send('Healthy')}
    });
  
  // Health Probe - Application Readiness
  router.get('/health/readiness',function(req,res){
    console.log(`code ----> ${readiness_new}`)
    res.status(parseInt(readiness_new))
      if (readiness_new > 399){
        res.send('Not Ready')
      }
      else{ res.send('Ready')}
      
    });  
  // Change Health probe status
  router.get('/liveliness/:statuse', function(req,res){
    var l_statuse = req.params['statuse'];
    console.log(l_statuse)
    liveliness_new = l_statuse;
    console.log(`New status code set ${l_statuse}`)
    res.redirect('/')
  });
  // Change Readiness probe status
  router.get('/readiness/:statuse', function(req,res){
    var r_statuse = req.params['statuse'];
    console.log(r_statuse)
    readiness_new = r_statuse;
    console.log(`New status code set ${r_statuse}`)
    res.redirect('/')
  });
  
  //add the router
  app.use('/', router);
  app.listen(port);
  
  console.log(`Running at Port ${port}`);
  ```

- add ,commit and push our new file.

  ```Bash
  git add .
  git commit -m "updated the app.js application"
  git push
  ```

### 8. Build a new image and push it to the registry

1. navigate to the Dockerfile location.

    1. now we will add a .dockerignore file so the image will be slimmer

        ```Bash
        touch .dockerignore
        echo "*/node_modules" >> .dockerignore
        ```

    2. And build our new image

        ```Bash
        docker build . -t quay.io/<quay-userName>/<imageName>:v2
        ```

2. push the new image to the quay registry:

      ```Bash
      docker push quay.io/<quay-userName>/<imageName>:v2
      ```

3. Update the Deployment.yaml file with the new image tag and wait for ArgoCD to update the Deployment, (you can refresh the application manualy):

```YAML
    spec:
      containers:
          name: hello-world
          image: 'quay.io/<quay-userName>/<imageName>:v2'
```

add ,commit and push our new file.

```Bash
git add .
git commit -m "updated the iamge tag in Deployment"
git push
```

- Click on Sync to Update the Openshift Cluster.
- wait for the pods to rollout and the new ver. had been deployed
- test the URL again to see the new Web Page.

![Web application v2](https://github.com/rhilconsultants/Application-Deployment-Workshop/blob/main/Class%20artifacts/lab1-part-2-web.png)

## Great Jog You have Finished Part 2

### Now you can start part 3 [Here](https://github.com/rhilconsultants/HELM-ArgoCD-Lab/blob/main/Lab1/Lab1_part_3.md)
