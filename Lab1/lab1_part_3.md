# HELM-ArgoCD-Lab1-Part-3

## In this lab we will create a NodeJS application and deploy it to the Openshift cluster

### Part 3 lets create a HELM chart and deploy it to the cluster with argoCD

#### 1. create a Helm Chart for our application

1. create folder in the root folder of our application.

```Bash
mkdir helm
```

2. enter the "helm" folder and create the following:
- Chart.yaml
- values.yaml
- a folder named "templates"

```Bash
cd helm
touch Chart.yaml values.yaml
mkdir templates
```

3. Copy all 3 YAML files from our yaml folder to our helm/templates

```Bash
cp ../yaml/* .
```

4. Open the Chart.yaml file and Copy the Following Context:

```YAML
apiVersion: v2
name: hello-world
description: A Helm chart for Kubernetes
type: application
version: 1.0.0
appVersion: "1.0.0"
```

5. Edit the "templates/Deployment.yaml" file as Following:

- the value of "metadata.name:" change to {{ .Release.Name }}
- the value of "spec.replica:" cchange to {{ .Values.ReplicaNumber }}
- the value of "spec.selector.matchLabels: change to {{ .Release.Name }}
- the value of "spec.template.metadata.labels.app:" change to {{ .Release.Name }}
- the value of "spec.contianer.name:" change to {{ .Release.Name }}
- the value of "spec.contianer.ports.containerPort:" change to {{ .Values.containers.containerPort }}
- the value of "spec.contianer.iamge:" change to {{ .Values.containers.image }}:{{ .Values.containers.tag }}

the edited file should look like this:

```YAML
kind: Deployment
apiVersion: apps/v1
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.ReplicaNumber }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
        - resources: {}
          terminationMessagePath: /dev/termination-log
          name: {{ .Release.Name }}
          ports:
            - containerPort: {{ .Values.containers.containerPort }}
              protocol: TCP
          imagePullPolicy: Always
          terminationMessagePolicy: File
          image: {{ .Values.containers.image }}:{{ .Values.containers.tag }}
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```

add ,commit and push the file to the git repo

```bash
git add.
git commit -m "edited the HELM template for Deployment"
git push
```

6. Edit the "templates/service.yaml" file as Following:

- The value of "metadata.name" change to {{ .Release.Name }}-service
- The value of "spec.selector.app" change to {{ .Release.Name }}
- The value of "spec.ports.port" change it to {{ .Values.service.servicePort }}
- The value of "spec.ports.targetPort" change it to {{ .Values.containers.containerPort }}

the edited file should look like this:

```YAML
kind: Service
apiVersion: v1
metadata:
  name: {{ .Release.Name }}-service
spec:
  ports:
    - protocol: TCP
      port: {{ .Values.service.servicePort }}
      targetPort: {{ .Values.containers.containerPort }}
  selector:
    app: {{ .Release.Name }}
```

add ,commit and push the file to the git repo

```bash
git add.
git commit -m "edited the HELM template for Service"
git push
```

7. Edit the "templates/route.yaml" file as Following:

- The value of "metadata.name" change it to {{ .Release.Name }}-route
- The value of "spec.to.name" change it to {{ .Release.Name }}-service
- The value of "spec.port.targetPort" change it to {{ .Values.service.servicePort }}

the edited file should look like this:

```YAML
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: {{ .Release.Name }}-route
spec:
  to:
    kind: Service
    name: {{ .Release.Name }}-service
    weight: 100
  port:
    targetPort: {{ .Values.service.servicePort }}
  wildcardPolicy: None
```

add ,commit and push the file to the git repo

```bash
git add.
git commit -m "edited the HELM template for Route"
git push
```

8. Edit the values.yaml file accourding to our template values.

- every section that we edited that have the {{ .Values }} in it need to be in the values file or enter manualy with --set option.
- the {{ .Release.Name }} value is created from the release of the chart creation.
- Open the values.yaml file and create the following Context:

```YAML
ReplicaNumber: 3

containers:
  containerPort: 8080
  image: 'quay.io/argo-helm-workshop/hello-world:'
  tag: 'base'

service:
  servicePort: 8080
```

add ,commit and push the file to the git repo

```bash
git add.
git commit -m "edited the HELM values.yaml file"
git push
```
