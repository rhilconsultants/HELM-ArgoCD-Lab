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
   - Update all the labels from "app" to "app-1"

   - Go to the deployment_2.yaml file under helm/templates
     - Add the following section, to deployment_2.yaml

    ```YAML
    annotations:
      argocd.argoproj.io/sync-wave: "10"
    ```

   - Update all the Section that contains {{ .Release.Name }} to {{ .Release.Name }}-2
   - Update all the labels from "app" to "app-1"
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
