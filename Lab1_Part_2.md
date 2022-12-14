# HELM-ArgoCD-Lab1-Part-2

## In this lab we will create a NodeJS application and deploy it to the Openshift cluster

### Part 2 lets Deploy our Application using YAML Files and see if it works

#### 1. create a new folder named yaml

```
cd ~
mkdir yaml
```

#### 2. create a new files called deployment.yaml route.yaml service.yaml

```
cd yaml
touch deployment.yaml route.yaml service.yaml
```

#### 3. open the deployment.yaml file in VScode and create a deployment menifast, Dont forget to edit with your UseName

```
kind: Deployment
apiVersion: apps/v1
metadata:
# set your user name
  name: <userName>-hello-world
spec:
  replicas: 3
  selector:
    matchLabels:
# set your user name
      app: <userName>-hello-worldhello-world
  template:
    metadata:
      labels:
# set your user name
        app: <userName>-hello-worldhello-world
    spec:
      containers:
        - resources: {}
          readinessProbe: {}
          terminationMessagePath: /dev/termination-log
          name: hello-world
          livenessProbe: {}
          ports:
            - containerPort: 8080
              protocol: TCP
          imagePullPolicy: IfNotPresent
# update with the iamge you build in part 1
          image: 'quay.io/<usrName>/<imageName>:<Tag>'
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
```

#### 4. open the service.yaml file in VScode and create a route menifast, Dont forget to edit with your UseName

```
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

#### 4. open the route.yaml file in VScode and create a route menifast, Dont forget to edit with your UseName

```
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

#### 6. Lets Deploy our application to the Cluster with ArgoCD

i. Open your ArgoCD instance via the link it the Dashboard

> Create a new Application
> enter your Application name as the Following <userName>-hello-world
> Enter your GitHub repo Url
> select the branch that you are working on (e.i "main")
> select the "yaml/" folder
> selcet the "in-cluster"
> enter your deployment desired Namespace.
> Don't check "auto-sync" yet
> Click on save

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
podman build . -t quay.io/<userName>/<imageName>:<Tag>
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
