# HELM-ArgoCD-Lab3-Part-2

## In this lab we will Automate Deployment from helm Changes

## In this lab we will use "sync-waves" and "sync-hooks"

---

1. Create another copy of deployment.yaml file under helm/templates and name it deployment_2.yaml
   - Change the name of deployment.yaml to deployment_1.yaml
   - Add the following section, to deployment_1.yaml

   ```YAML
    annotations:
      argocd.argoproj.io/sync-wave: "5"
   ```

   - Update all the Section that contains {{ .Release.Name }} to {{ .Release.Name }}-1
   - Go to the deployment_2.yaml file under helm/templates
     - Add the following section, to deployment_2.yaml

    ```YAML
    annotations:
      argocd.argoproj.io/sync-wave: "10"
    ```

   - Update all the Section that contains {{ .Release.Name }} to {{ .Release.Name }}-2
   - Edit the service.yaml file, change the selecto, as following:

     ```YAML
     ...
       selector:
         app: "{{ .Release.Name }}-1"
         app: "{{ .Release.Name }}-2"
     ...
     ```

   - Push to git and See what happens

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
          args: ["curl ${SERVICE}:${PORT}/health/liveliness | grep ${TEST} || exit 1"]
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
           args: ["curl $SERVICE:$PORT | grep $TEST || exit 1"]
           ...
             - name: TEST
             value: "{{ .Values.test }}"
           ...
   ```

   - add to the values.yaml the following section:

   ```YAML
   test: Healty
   ```

   - Update the index.html file again and see what happens.
     - Replace "ArgoCD SyncWaves" with "ArgoCD SyncHooks Test"

   - see that the sync is successfull.✅💚

   - Now change the value of key test so it will fail the sync.

   - sync the application and let the sync run.

   - Now let add a syncFail job that will create a new issue in our github repo, create a new file named notification.yaml, edit with the following

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
           - configMapRef:
               name: issue-message
           command: ["/bin/bash", "-c"]
           args: ["curl -u $USERNAME:$TOKEN -X POST -d '{\"title\":\"Deployment sync Done\",\"body\":\"The web site not Healthy.\"}' $URL" ]
         restartPolicy: Never
     backoffLimit: 1
   ```

   - Replace in the args, {UserName} with your GitHub User Name
   - create a new file name gh-details.yaml

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

   - create the following section in the values.yaml file

   ```YAML
   ...
   github:
     user: # Your GitHub User name
     token: empty-pass
     url: <https://api.github.com/repos/{Git-userName}/{Repository-Name}/issues>
   ...
   ```

   - After the first sync Open the application Details in the argoCD UI and change to the Parameters Tab and enter you token in the token field.
   ![ArgoCD App params](https://raw.githubusercontent.com/rhilconsultants/Application-Deployment-Workshop/main/Class%20artifacts/lab3-part2-ui.png)

   - Update the index.html file again, and see the sync process.
   - if every thing was successfull the test-url job should completed succeffully.
   - Change the test values in the values.yaml file so the url test will fail.
   - update the index.html file again and refresh the application in the ArgoCD UI.
   - After the Sync finish see if there is a new Issue in the GitHub Portal.

## Hope you enjoyed 🤪
