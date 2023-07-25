# HELM-ArgoCD-Lab3-Part-2

## In this lab we will Automate Deployment from helm Changes

## In this lab we will use "sync-waves" and "sync-hooks"

---

1. Create another copy of the deployment.yaml file under helm/templates and name it deployment_2.yaml
   - Change the name of the deployment.yaml to deployment_1.yaml
   - Add the following section, to deployment_1.yaml

   ```YAML
   metadata:
    name: *****
    annotations:
      argocd.argoproj.io/sync-wave: "5"
   ```

   - Update all the Section that contains {{ .Release.Name }} to {{ .Release.Name }}-1
   - change the Selector label from app to app1: & app2: according to the deployment file.
   - Go to the deployment_2.yaml file under helm/templates
     - Add the following section, to deployment_2.yaml

    ```YAML
    metadata:
      name:
      annotations:
        argocd.argoproj.io/sync-wave: "10"
    ```

   - Update all the Section that contains {{ .Release.Name }} to {{ .Release.Name }}-2
   - Edit the service.yaml file, change the selector section, as follows:

     ```YAML
     ...
       selector:
         app1: "{{ .Release.Name }}-1"
         app2: "{{ .Release.Name }}-2"
     ...
     ```

   - Push to git and See what happens, Open the Helm application in the ArgoCD UI and follow the sync steps, notice the flow of the sync.

   - Go to the ConfigMap.yaml file under helm/templates
     - Add the following section:

    ```YAML
    annotations:
      argocd.argoproj.io/sync-wave: "1"
    ```

   - Update the index.html file again and see what happens.
     - Replace "testing automated rollout" with "ArgoCD SyncWaves"
   Add, Commit and Push to the git.

   make other change to the index.html file ,add another header.

     ```html
     <h3>Testing sync waves</h3>
     ```

   push the new change and watch the flow in the ArgoCD.

2. Let's add a Job that curl the service and check if it works

   - Create the following job.yaml, to the helm/templates

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
              value: {{ .Release.Name }}-service
            - name: PORT
              value: {{ .Values.service.servicePort }}
     backoffLimit: 1
   ```

   - add, commit and push to the git.

   - Update the index.html file again and see what happens.
     - Replace "ArgoCD SyncWaves" with "ArgoCD SyncHooks"
   - change the args to the Following:

   ```YAML
           args: ["curl $SERVICE:$PORT/health/liveliness | grep $TEST || exit 1"]
           ...
             - name: TEST
               value: "{{ .Values.test }}"
           ...
   ```

   - add to the values.yaml the following section:

   ```YAML
   test: Healthy
   ```

   - Update the index.html file again and see what happens.
     - Replace "ArgoCD SyncWaves" with "ArgoCD SyncHooks Test"

   - see that the sync is successful.✅💚

   - Now change the value of the key test so it will fail the sync, Set test: Null

   - sync the application and let the sync run.

   - Set the key, test: Healthy, in the values.yaml file

   - Now let's add a syncFail job that will create a new issue in our GitHub repo, create a new file named notification.yaml, edit with the following

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

   - create the following section in the values.yaml file, update the field with your details

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

   - Update the index.html file again, and see the sync process.

       ```html
       <h3>Testing fail hook</h3>
       ```

   - if every thing was successfull the test-url job should completed succeffully.
   - Change the test values in the values.yaml file so the url test will fail.
   - update the index.html file again and refresh the application in the ArgoCD UI.
   - After the Sync finish see if there is a new Issue in the GitHub Portal.

## Hope you enjoyed 🤪
