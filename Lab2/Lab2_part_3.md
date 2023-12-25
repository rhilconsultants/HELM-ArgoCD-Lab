# HELM-ArgoCD-Lab2-Part-3

## In this lab we will Create a HELM Package and push it to our GitHub Packages

for this step you need to make your github account token, [How to create github token](https://docs.github.com/en/enterprise-server@3.4/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token), Make sure that you have selected:
![Token-package](https://github.com/rhilconsultants/Application-Deployment-Workshop/blob/main/Class%20artifacts/Github-Token-for-package.png)

1. Navigate to your Helm folder, and run the following command:

    ```Bash
    helm package .
    ```

    This will create a new tar.gz file under the helm folder with the name of the chart.

2. Login to the GitHub package service (ghcr.io)

    ```Bash
    helm registry login -u <<Git-Hub User name>> ghcr.io
    ```

    after it copy and paste the token we have created in the beggining.
    if everything is fine you should get "Login Succeeded" massage.

3. Push the helm package to the ghcr.io server.

    ```Bash
    helm push <<Chart-Name>>.tgz oci://ghcr.io/<<Git-Hub User Name>>/helm
    ```

    If everything is succesful we will get:
    > Pushed: localhost:5000/helm-charts/mychart:0.1.0
    > Digest: sha256:ec5f08ee7be8b557cd1fc5ae1a0ac985e8538da7c93f51a51eff4b277509a723

4. Now lets create a new folder in our Repository root, named "sub_chart"

    ```Bash
    mkdir sub_chart
    ```

    in the folder we will create the follwoing files:

    Chart.yaml

    ```YAML
    apiVersion: v2
    name: workshop-test
    description: A Helm chart for Kubernetes
    type: application
    dependencies:
      - name: <<Name-of-Your-Chart>>
        repository: oci://ghcr.io/<<Git-Hub-User>>/helm/
        version: "1.0.0"
        alias: deploy1
    version: 1.0.0
    appVersion: "1.0.0"
    ```

    values.yaml

    ```YAML
    deploy1: 
      ReplicaNumber: 1
  
      containers:
          containerPort: 8080
          image: 'quay.io/argo-helm-workshop/workshop-app'
          tag: 'chart_v2'
  
      service:
          servicePort: 8080
    ```

    and a 2 folders named "charts" and "templates", in each one of them create an empty file named ".gitkeep",
    the folder and files should look something like this:

    ![subchart-folder](https://github.com/rhilconsultants/Application-Deployment-Workshop/blob/main/Class%20artifacts/sub-chart-folder-n-files.png)

    - *Dont forget to commit and push the new changes before continuing to the next step.*

5. Now we will create a new ArgoCD application, for the sub-chart.

    Click on the "+ NEW APP" button on the argoCD UI and fill the form as follows:

    Under General:

    - Application Name: user{n}-sub-chart
    - Project Name: default
    - SYNC Policy: automatic, and check the Prune and self-heal check boxs

    Under Source:

    - Repository Url: your Git Hub Repository Url
    - Revision: main
    - Path: sub_chart

    Destination:

    - Cluster URL: Select "<https://kubernetes.default.svc>"
    - Namespace: user{n}-application

    Then click create, a new argo application will be created and will began syncing, wait for it to be in healthy and Synced state.

6. Now we will add a 2nd sub chart based on the same Chart package that we created.

    - edit the Chart.yaml file, add a 2nd ependencies:

    ```YAML
    - name: workshop-test
      repository: "oci://ghcr.io/tal-hason/helm"
      version: 1.0.1
      alias: deploy2
    ```

    - edit the values.yaml file, add a the deploy2 section:

    ```YAML
    deploy2: 
      ReplicaNumber: 1

      containers:
        containerPort: 8080
        image: 'quay.io/argo-helm-workshop/workshop-app'
        tag: 'chart_v2'

      service:
        servicePort: 8080
    ```

    After this sync or refresh our sub-chart argo application, it will create a 2nd deployment service route and config-map.

    it should look like this:

    ![Sub-chart-deployed](https://github.com/rhilconsultants/Application-Deployment-Workshop/blob/main/Class%20artifacts/sub-chart-deployed.png)

## Great Jog You have Finished Part 3
