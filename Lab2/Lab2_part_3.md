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
    helm push <<Chart-Name>>.tgz oci://ghcr.io/<<Git-Hub User Name>>/helm/Repository-Name
    ```

    If everything is succesful we will get:
    > Pushed: localhost:5000/helm-charts/mychart:0.1.0
    > Digest: sha256:ec5f08ee7be8b557cd1fc5ae1a0ac985e8538da7c93f51a51eff4b277509a723