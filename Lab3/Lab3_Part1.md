# HELM-ArgoCD-Lab3-Part-1

## In this lab we will Automate Deployment from helm Changes

## this lab will be diffrent, in this lab no code samples will be given and you will need to find the current solution by yourself

---

1. Update the HELM chart values.yaml with the following.
   - Set a new Parent Section named - "probes"
     - create a sub section named - "readiness"
       - create a key names - "path", with the value of the current path
       - create a key names - "initialDelaySeconds", with the value of the current initialDelaySeconds
       - create a key names - "timeoutSeconds", with the value of the current timeoutSeconds
       - create a key names - "periodSeconds", with the value of the current periodSeconds
       - create a key names - "successThreshold", with the value of the current successThreshold
       - create a key names - "failureThreshold", with the value of the current failureThreshold
     - create a sub section names - "liveness"
       - create a key names - "path", with the value of the current path
       - create a key names - "initialDelaySeconds", with the value of the current initialDelaySeconds
       - create a key names - "timeoutSeconds", with the value of the current timeoutSeconds
       - create a key names - "periodSeconds", with the value of the current periodSeconds
       - create a key names - "successThreshold", with the value of the current successThreshold
       - create a key names - "failureThreshold", with the value of the current failureThreshold
   - Set a new Parent Section named - "volume"
     - create a sub section named - "mount"
       - create a key names - "path", with the value of the current mountPath
   - Add, commit and push to git to see if everything is working.

2. Add annotations to the deployment that after each new commit ArgoCD will rollout.

3. Change the annotations so it will rollout after the Configmap has been updated.

4. Update the index.html and see the deployment rollout the new change.
   - Change the "Testing Probes buttons" to "testing automated rollout"

> TIP: [HELM Tips and tricks](https://helm.sh/docs/howto/charts_tips_and_tricks/#automatically-roll-deployments)

## Go to Part 2
