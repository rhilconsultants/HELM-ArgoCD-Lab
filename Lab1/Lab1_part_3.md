# HELM-ArgoCD-Lab1-Part-3

## In this lab we will create a NodeJS application and deploy it to the Openshift cluster

### Part 3 lets create a HELM chart and deploy it to the cluster with argoCD

#### 1. create a Helm Chart for our application

1. create folder in the root folder of our application.

   ```Bash
   mkdir helm
   ```

2. enter the "helm" folder and create the following.

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
   cp ../yaml/* ./templates/
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

   - the value of "metadata: -> name:" change to {{ .Release.Name }}
   - the value of "spec: -> replica:" cchange to {{ .Values.ReplicaNumber }}
   - the value of "spec: -> selector: -> matchLabels: change to {{ .Release.Name }}
   - the value of "spec: -> emplate: -> metadata: -> labels: -> app:" change to {{ .Release.Name }}
   - the value of "spec: -> contianer: -> name:" change to {{ .Release.Name }}
   - the value of "spec: -> contianer: -> ports: -> containerPort:" change to {{ .Values.containers.containerPort }}
   - the value of "spec: -> contianer: -> image:" change to {{ .Values.containers.image }}:{{ .Values.containers.tag }}

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
     ```

   add ,commit and push the file to the git repo

     ```bash
     git add.
     git commit -m "edited the HELM template for Deployment"
     git push
     ```

6. Edit the "templates/service.yaml" file as Following:

   - The value of "metadata: -> name" change to {{ .Release.Name }}-service
   - The value of "spec: -> selector: -> app" change to {{ .Release.Name }}
   - The value of "spec: -> ports: -> port" change it to {{ .Values.service.servicePort }}
   - The value of "spec: -> ports: -> targetPort" change it to {{ .Values.containers.containerPort }}

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

   - The value of "metadata: -> name" change it to {{ .Release.Name }}-route
   - The value of "spec: -> to: -> name" change it to {{ .Release.Name }}-service
   - The value of "spec: -> port: -> targetPort" change it to {{ .Values.service.servicePort }}

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
       image: 'quay.io/<quay-userName>/<imageName>:v2'
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

9. Now we test if our Chart is working without any issues.

   ```Bash
   helm template user{n}-hello-world .
   ```

   in the out put we will get all our template Yaml rendered with our values from the values.yaml

   For example:

   ```YAML
   ---
   # Source: hello-world/templates/service.yaml
   kind: Service
   apiVersion: v1
   metadata:
     name: user1-hello-world-service
   spec:
     ports:
       - protocol: TCP
         port: 8080
         targetPort: 8080
     selector:
       app: user1-hello-world
   ---
   # Source: hello-world/templates/deployment.yaml
   kind: Deployment
   apiVersion: apps/v1
   metadata:
     name: user1-hello-world
   spec:
     replicas: 3
     selector:
     matchLabels:
       app: user1-hello-world
     template:
       metadata:
         labels:
           app: user1-hello-world
       spec:
         containers:
           - resources: {}
             terminationMessagePath: /dev/termination-log
             name: user1-hello-world
             ports:
               - containerPort: 8080
                 protocol: TCP
             imagePullPolicy: Always
             image: quay.io/argo-helm-workshop/workshop-app:v7
         restartPolicy: Always
         terminationGracePeriodSeconds: 30
     strategy:
       type: RollingUpdate
       rollingUpdate:
         maxUnavailable: 25%
         maxSurge: 25%
     revisionHistoryLimit: 10
   ---
   # Source: hello-world/templates/route.yaml
   kind: Route
   apiVersion: route.openshift.io/v1
   metadata:
     name: user1-hello-world-route
   spec:
     to:
       kind: Service
       name:  user1-hello-world-service
       weight: 100
     port:
       targetPort: 8080
     wildcardPolicy: None
   ```

10. Create an ArgoCD application for the HELM Chart.

    - in the ArgoCD UI Click "+ NEW APP"
    - fill the field as following:

      - Application Name: user{n}-hello-chart
      - Project Name: default
      - Sync Policy: Automatic and check the prune and auto-heal check boxxes
      - Repository URL, copy your git clone URL.
      - revision: main
      - Path: Application/helm/
      - Destination: 'https://kubernetes.default.svc'
      - Namespace: user{n}-application

    and click create.

    - wait a few seconds for the deployment to finish.
    - find the url for the new application

11. Build a New image for our Helm Chart Application.

    - Navigate to our HTML folder under the src folder
    - Replace "K8S Application" With "Helm Chart Application"
    - Replace "this message will be modifed" With "Deployed with HELM and ArgoCD"
    - Add under < img src="https://developers.redhat.com/sites/default/files/styles/article_feature/public/blog/2018/05/openshift-featured.png?itok=g0Ee8H1H" alt="OpenShift" > 
      a new image: 
      "< img src="https://redhat-scholars.github.io/argocd-tutorial/argocd-tutorial/_images/argocd-sync-flow.png" >"
    - Don't forget to clean the white spaces after "<" and before ">" of the  new image.

add,commit, and push the file to the git repo

```bash
git add.
git commit -m "edited the html file"
git push
```

- Now build a new image:

```Bash
docker build . -t quay.io/(quay-userName)/(imageName):chart_v1
```

- Now lets push our image to the registry

```Bash
docker push quay.io/(quay-userName)/(imageName):chart_v1
```

and wait for it to finish

- Open our values.yaml file and update the container.tag value to "chart_v1"

```YAML
containers:
  containerPort: 8080
  image: 'quay.io/(quay-userName)/(imageName)'
  tag: 'chart_v1'
```

add ,commit and push the file to the git repo

```bash
git add.
git commit -m "edited the HELM values.yaml file with new image tag version"
git push
```

- wait for ArgoCD to refresh by itself(240 sec) or refresh it manually, but this time it will sync by itself.
- wait for the deployment rollout to be completed and refresh the URL.

![Helm-Chart-web](https://github.com/rhilconsultants/Application-Deployment-Workshop/blob/main/Class%20artifacts/lab1-part-3-web.png)

## Great Jog You have Finished Part 3

### If you have finished go do the Bouns Lab [Here](https://github.com/rhilconsultants/HELM-ArgoCD-Lab/blob/acfa93eab54740d9c0e3a252fbe580fb67c62964/Lab1/Bonus_Lab.md)
