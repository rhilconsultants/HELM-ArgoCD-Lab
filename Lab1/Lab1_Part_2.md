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

#### 3. open the deployment.yaml file in VScode and create a deployment menifast, Dont forget to edit with your UseName

```YAML
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
      app: <userName>-hello-world
  template:
    metadata:
      labels:
# set your user name
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
# update with the iamge you build in part 1
          image: 'quay.io/<userName>/<imageName>:<Tag>'
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

#### 4. open the route.yaml file in VScode and create a route menifast, Dont forget to edit with your UseName

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

add and commit and push the new files to the git repo

```bash
git add.
git commit -m "added k8s menifasts"
git push
```

#### 6. Lets Deploy our application to the Cluster with ArgoCD

i. Open your ArgoCD instance via the link found in the Dashboard

> 1. Create a new Application.
> 2. enter your Application name as the Following "{userName}-hello-world".
> 3. Enter your GitHub repo Url.
> 4. select the branch that you are working on (e.i "main").
> 5. select the "yaml/" folder.
> 6. selcet the "in-cluster / 'https://kubernetes.default.svc'"
> 7. enter your application namespace - "user1-application.
> 8. under sunc policy, leave it manual for now.
> 9. Click on create.

and wait for the applicaion will show in the UI

ii. click on the application and and then click the "Sync" button ,and wait for the application to deploy to the cluster

iii. try to find the URL for the application and access it from your broswer.

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
    name: user1-hello-world
    weight: 100
  wildcardPolicy: None
```

- find the 'host:' key and copy the url to the browser, with http:// before the url.

we should see our web site
![hello-world]()

