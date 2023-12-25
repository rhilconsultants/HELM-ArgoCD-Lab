# HELM-ArgoCD-Lab3-Part-2

## In this lab we will Automate Deployment from helm Changes

## In this lab we will use "sync-waves" and "sync-hooks"

---

*We will edit some parts in the helm folder and also edit the sub chart*

1. In the Helm folder in our source helm package add the following annotation, and add the value of it to the value file.

   ```YAML
   metadata:
    name: '*****'
    annotations:
      argocd.argoproj.io/sync-wave: '{{ .Values.argocd.syncwave.deployment }}'
   ```

   ```YAML
   argocd:
     syncwave:
        deployment: 5
   ```

   - Update the Chart.yaml Version to 1.0.2 package the chart and push it to the ghcr registry.

   - add this value also to the value file in the sub_chart as following:
   - deploy1, with value 5
   - deploy2, with value 10
   - update the Chart.yaml in the sub_chart folder with the new chart version 1.0.2

   - Push to git and See what happens, Open the Helm application in the ArgoCD UI and follow the sync steps, notice the flow of the sync.

   - Go to the ConfigMap.yaml file under helm/templates
     - Add the following section:

    ```YAML
    annotations:
      '{{ .Values.argocd.syncwave.configmap }}'
    ```

    add this new values to the value file, in both helm and sub_chart folder like this:

    ```YAML
    argocd:
      syncwave: 
        deployment: 5
        configmap: 1
    ```
  
   - Change the chart version under helm to 1.0.3, package it and push to ghcr registry.
   - Update the Chart.yaml under sub_chart with the new dependencies version that we created.

   - in the helm folder update the index.html file
      - Update the index.html file again and see what happens.
        - Replace "testing automated rollout" with "ArgoCD SyncWaves"
        - update the Chart.yaml file with a new version 1.0.4
        - package and push the new chart to ghcr.
        - update the chart.yaml in the sub_chart folder with the new version 1.0.4
   Add, Commit and Push to the git.

   make other change to the index.html file ,add another header.

     ```html
     <h3>Testing sync waves</h3>
     ```

   - Update the chart version in the helm folder with a new version 1.0.5
   - Package the new version and push it to the ghcr registery
   - update the Chart.yaml file under sub_chart folder, edit the version only to deploy1 dependency with the new chart 1.0.5
   - push the new change and watch the flow in the ArgoCD.
   - access both deploy1 and deploy2 route to see if they have diffrent HTML file.

2. Let's add a Job that curl the service and check if it works, we will add this job in our sub_chart template

   - Create the following job.yaml, in the sub_chart/templates folder.

   ```YAML
    apiVersion: batch/v1
    kind: Job
    metadata:
      annotations:
        argocd.argoproj.io/hook: PostSync
        argocd.argoproj.io/hook-delete-policy: BeforeHookCreation
      name: test-url
    spec:
      template:
        metadata:
          name: test-url
        spec:
        restartPolicy: Never
        containers:
        - name: test-url
          image: registry.access.redhat.com/ubi8-minimal:8.7-1031
          workingDir: /workspace/output
          command: ["/bin/bash", "-c"]
          args: ["curl ${SERVICE}:${PORT}/health/liveliness || exit 1"]
          env:
            - name: SERVICE
              value: "deploy-service"
            - name: PORT
              value: "{{ .Values.deploy.service.servicePort }}"
      backoffLimit: 1
   ```

   - add, commit and push to the git.

   - Sync the sub-cahrt application, and check the test-url job pod log, it should print "healty"
   - change the args and add a new env variable to the job.

   ```YAML
           args: ["curl $SERVICE:$PORT/health/liveliness | grep $TEST || exit 1"]
           ...
             - name: TEST
               value: "{{ .Values.test }}"
           ...
   ```

   - add to the values.yaml the following section, on the top part of the file:

   ```YAML
   test: Healty
   ---
   ```

   - Sync the Sub-chart and follow the test-url Job

   - see that the sync is successful.✅💚

   - Now change the value of the key test so it will fail the sync, Set test: Null

   - sync the application and let the sync run.

   - Set the key, test: Healty, in the values.yaml file

   - Now let's add a syncFail job that will create a new issue in our GitHub repo, create a new file named notification.yaml in the sub_chart/templates folder edit with the following

   ```YAML
   apiVersion: batch/v1
   kind: Job
   metadata:
     name: github-issue-job
     annotations:
       argocd.argoproj.io/hook: SyncFail
       argocd.argoproj.io/hook-delete-policy: BeforeHookCreation
   spec:
     template:
       spec:
         containers:
         - name: github-issue
           image: registry.access.redhat.com/ubi8-minimal:8.7-1031
           envFrom:
           - secretRef:
               name: gh-details
           command: ["/bin/bash", "-c"]
           args: ["curl -u $USERNAME:$TOKEN -X POST -d '{\"title\":\"Deployment sync Done\",\"body\":\"The web site not Healthy.\"}' $URL" ]
         restartPolicy: Never
     backoffLimit: 1
   ```

   - create a new file name gh-details.yaml, for this step you need to make your github account token, [How to create github token](https://docs.github.com/en/enterprise-server@3.4/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

   ```YAML
   apiVersion: v1
   kind: Secret
   metadata:
     name: gh-details
   type: Opaque
   data:
     USERNAME: {{ .Values.github.user | b64enc }}
     TOKEN: {{ .Values.github.token | b64enc }}
     URL: {{ .Values.github.url | b64enc }} 
   ```

   - create the following section in the top part of the values.yaml file, update the field with your details

   ```YAML
   ...
   github:
     user: # Your GitHub User name
     token: empty-pass
     url: <https://api.github.com/repos/{Git-userName}/{Repository-Name}/issues>
   ...
   ```

   - if we store the GitHub token in our Repository GitHub will block the token, so we need to enter it directly in the ArgoCD UI.
   - After the first sync, Open the application Details in the argoCD UI, change to the Parameters Tab, and enter your token in the token field.
   ![ArgoCD App params](https://raw.githubusercontent.com/rhilconsultants/Application-Deployment-Workshop/main/Class%20artifacts/lab3-part2-ui.png)

   - sync the application again...
      - if every thing was successfull the test-url job should completed succeffully.
      - Change the value of test in the values.yaml file to "fail" so the url-test job will fail.
      - Click the refersh and then Sync buttons in the ArgoCD UI
      - After the Sync finish see if there is a new Issue in the GitHub Portal.

## Hope you enjoyed 🤪
