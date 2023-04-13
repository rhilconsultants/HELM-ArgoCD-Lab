# HELM-ArgoCD-Lab1-Bonus

## In this lab we will create a Canary Route or our 2 Applications

1. Create a new route.yaml

   - create a new folder named  "blue-green" in the root of the repo.

     ```Bash
     mkdir blue-green
     ```

   - Create a new route file named "blue-green.yaml"

     ```Bash
     touch blue-green/blue-green.yaml
     ```

   - Edit the blue-green.yaml with the Following context

     ```YAML
     kind: Route
     apiVersion: route.openshift.io/v1
     metadata:
       name: blue-green-route
       annotations:
         haproxy.router.openshift.io/disable_cookies:'true'
         haproxy.router.openshift.io/balance:'roundrobin'
     spec:
       to:
         kind: Service
         name: user{n}-hello-chart-service
         weight: 90
       alternateBackends:
       - kind: Service
         name: user{n}-hello-world
         weight: 10
     ```

   add ,commit and push the file to the git repo

     ```bash
     git add.
     git commit -m "create a blue-green route"
     git push
     ```

2. Create a new argoCD application for the Blue-Green Route

   - In the ArgoCD UI click on "+ New APP" :
   fill the form with this parameters

      - Application Name = blue-green
      - project name = default
      - Sync Policy = Automatic (check the prune & Auto Heal)
      - Repository URL = Your Git Clone URL
      - Revision = main
      - ClusterUrl= 'https://kubernetes.default.svc'
      - Namespace = user{n}-application

   And click create!

   wait for the sync to complete and then open the new route URL in the browser

   refresh several times and see if the web page changes.

3. change route wights for services.

   - edit the blue-green.yaml

   change the wight for the chart service to 0 and for the hello-world service to 100.

   add ,commit and push the file to the git repo

     ```bash
     git add.
     git commit -m "updated a blue-green route"
     git push
     ```

   wait for argo to refresh and sync or just sync it manualy.

   and try now to refresh the page. or even open it in incognito or try diffrent browser.

   play with the service wights and see if you trigger diffrent pages

   TIP, try to scale on deployment from 3 to 1 and see what is going on.

## Great Job You have Finished the Bonus Lab
