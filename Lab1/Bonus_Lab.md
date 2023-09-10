# HELM-ArgoCD-Lab1-Bonus

## In this lab we will create a Canary Route For a Blue green Deployments using the range operator in helm.

1. Create a new Application Deployment files:

   - create a new folder named  "blue-green" in the root of the repo.

     ```Bash
     mkdir blue-green
     ```

   - Create a Helm Chart in the folder"

     ```Bash
     touch blue-green/Chart.yaml
     ```

   - Edit the Chart.yaml with the Following context

   ```YAML
   apiVersion: v2
   name: blue-green
   description: A Helm chart for Kubernetes
   type: application
   version: 1.0.0
   appVersion: "1.0.0"
   ```

   create a values.yaml file:

   ```YAML
   deployment:
     - name: blue
       replicas: 1
       color: blue
       weight: 5
   ```

   - Create a new templates folder in the blue-green folder:

     ```Bash
     mkdir blue-green/templates
     ```

   - under it create the following files:

    deployment.yaml

     ```YAML
    {{- range .Values.deployment }}
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: {{ .name }}
      name: {{ .name }}
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: {{ .name }}
      strategy: {}
      template:
        metadata:
          labels:
            app: {{ .name }}
        spec:
          containers:
          - image: quay.io/rhdevelopers/bgd:1.0.0
            name: {{ .name }}
            env:
            - name: COLOR
              value: {{ .color }}
            resources: {}
            securityContext:
              allowPrivilegeEscalation: false
              capabilities:
                drop:
                - ALL
          securityContext:
            runAsNonRoot: true
            seccompProfile:
              type: RuntimeDefault
    {{- end }}
     ```

    service.yaml file

    ```YAML
    {{- range .Values.deployment }}
    ---
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        app: {{ .name }}
      name: {{ .name }}
    spec:
      ports:
      - port: 8080
        protocol: TCP
        targetPort: 8080
      selector:
        app: {{ .name }}
    {{- end }}
    ```

    route.yaml

    ```YAML
    apiVersion: route.openshift.io/v1
    kind: Route
    metadata:
      labels:
        app: blue-green
      name: blue-green
      annotations:
        haproxy.router.openshift.io/disable_cookies: 'true'
        haproxy.router.openshift.io/balance: 'roundrobin'
    spec:
      port:
        targetPort: 8080
      to:
        kind: Service
        name: blue-green
        weight: 0
      alternateBackends:
    {{- range .Values.deployment }}
      - kind: Service
        name: {{ .name }}
        weight: {{ .weight }}
    {{- end }}
    ```

    > Note that the blue green service have 0 weight so no traffic will be transport to it
    > The route object support up to 3 Backends services!!

   add ,commit and push the file to the git repo

     ```bash
     git add.
     git commit -m "create a blue-green chart"
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
      - ClusterUrl= '<https://kubernetes.default.svc>'
      - Namespace = user{n}-application

   And click create!

   wait for the sync to complete and then open the new route URL in the browser

   refresh several times and see if the web page changes.

3. change route wights for services.

   - edit the values.yaml

   add a new array under deploymnet

   ```YAML
     - name: green
       replicas: 1
       color: green
       weight: 5
   ```

     ```bash
     git add.
     git commit -m "updated a blue-green route"
     git push
     ```

   wait for argo to refresh and sync or just sync it manualy.

   now wait for the bobbles you should have blue and green bubbles.

   add a 3rd color (i.e "yellow) and see what happens.

   play with the service wights and see what happens to the bubbles in the page.

## Great Job You have Finished the Bonus Lab
