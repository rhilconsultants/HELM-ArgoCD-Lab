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
       argocd.argoproj.io/hook-delete-policy: HookSucceeded
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
          args: ["curl $SERVICE:$PORT || exit 1"]
          env:
            - name: SERVICE
              value: {{ .Release.Name }}-service
            - name: PORT
              value: {{ .Values.service.servicePort }}
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
   test: ArgoCD SyncHooks Test
   ```

   - Update the index.html file again and see what happens.
     - Replace "ArgoCD SyncWaves" with "ArgoCD SyncHooks Test"
