# HELM-ArgoCD-Lab2-Part-1

## In this lab we will Add More Features to our HELM Chart

1. Add a ConfigMap with the HTML file and inject it to the Application Deployment.

   Lets create a new File named ConfigMap.yaml in our Helm chart templates folder

   ```Bash
   cd helm/templates
   touch ConfigMap.yaml
   ```

   Open the File with VScode and add the New Context.

   ```YAML
   kind: ConfigMap
   apiVersion: v1
   metadata:
     name: index.html
   immutable: false
   data:
     index.html: |-
       <html>
       <head>
         <title>HELM Application</title>
         <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
         <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.1/css/bootstrap.min.css">
         <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.1/css/bootstrap-theme.min.css">
         <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.1/js/bootstrap.min.js"></script>
       </head>
       <body>
         <div style="margin:100px;">
          
       <class="navbar navbar-inverse navbar-static-top">
         <div class="container">
           <a class="navbar-brand" href="/">HELM Application</a>
           <ul class="nav navbar-nav">
             <li class="active">
               <a href="/">Home</a>
             </li>
             <li>
               <a href="/health/liveliness">Check liveliness</a>
             </li>
             <li>
               <a href="/health/readiness">Check readiness</a>
             </li>
           </ul>
         </div>
       </nav>
           <div class="jumbotron"  style="padding:40px;">
             <h1>Hello, world!</h1>
             <p>This is a simple hello World Web Page, this message will be modiifed.</p>
             <p><a class="btn btn-primary btn-lg" href="/liveliness/400" role="button">Not Healty</a></p>
             <p><a class="btn btn-primary btn-lg" href="/health/liveliness" role="button">Check liveliness</a></p>
             <p><a class="btn btn-primary btn-lg" href="/health/readiness" role="button">Check readiness</a></p>
             <p><a class="btn btn-primary btn-lg" href="/readiness/400" role="button">Not Ready</a></p>
           </div>
           <div class="jumbotron"  style="padding:40px;">
             <img src="https://developers.redhat.com/sites/default/files/styles/article_feature/public/blog/2018/05/openshift-featured.png?   itok=g0Ee8H1H" alt="OpenShift">
             <img src="https://raw.githubusercontent.com/rhilconsultants/Application-Deployment-Workshop/main/Class%20artifacts/helm-icon-color.png" itok=g0Ee8H1H" alt="HELM">
           </div>
         </div>
       </body>
       </html>
   ```

   Now Open the Deployment.yaml file in the templates folder and add the following section

   ```YAML
   ...  #this will Create Volume From our ConfigMap
   spec:
   ...  
     template:
     ...     
       spec:
         volumes:
           - name: index-html
             configMap:
               name: index.html
               defaultMode: 420
   ...  # This will mount our File to the Pod File system
         containers:
         ...
             volumeMounts:
             - name: index-html
               mountPath: /tmp/html # this will mount the html.index file to it application location
         ...
   ...
   ```

   Add, Commit and Push the new Update to the Git Repository

   ```Bash
   git add .
   git commit -m "added configmap"
   git push
   ```

2. go to ArgoCD ui and see if the change has been updated.

   Navigate to the Application URL and see the new Page.
   ![Web1](https://github.com/rhilconsultants/Application-Deployment-Workshop/blob/main/Class%20artifacts/lab2-part1-web1.png?raw=true)

3. Now we will Generate the Config map Data directly from the html it self.

   - Navigate to the ConfigMap.yaml in the helm/template folder.
   - Change the ConfigMap as following.

   ```YAML
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: index.html
   immutable: false
   data:
   {{ (.Files.Glob "html/*").AsConfig | indent 2 }}
   ```

   We set the source of the file to a html folder in side the HELM chart.

   - Do to HELM Limitation we need to copy our HTML file into our HELM chart

   ```Bash
   cp -r src/html helm/
   ```

   Add, Commit and Push the new Update to the Git Repository

   ```Bash
   git add .
   git commit -m "Generate configmap from file"
   git push
   ```

4. Go to the ArgoCD ui and refresh and check if the ConfigMap has been updated.

   - See that we now have a ConfigMap item in the argoCD.
   - Refresh or access the URL and see if anything changed

5. Because that helm dont update the deployment for the ConfigMap updates, Delete the pod from the argoCD ui, and wait for it to start again.

   - after the new pod has started
   - Access the url again.
   ![web2](https://github.com/rhilconsultants/Application-Deployment-Workshop/blob/main/Class%20artifacts/lab2-part1-web2.png?raw=true)

## Congertes You finshed Part 1 of Lab 2

## Now Move over to Part 2
