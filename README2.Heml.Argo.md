# K8s | 2nd README file

## Install Scoop for windows

<https://scoop.sh/>

```PowerShell
# Open a PowerShell terminal (version 5.1 or later) and run:

Set-ExecutionPolicy RemoteSigned -Scope CurrentUser # Optional: Needed to run a remote script the first time
irm get.scoop.sh | iex
```

output:

```PowerShell
Initializing...
Downloading ...
Creating shim...
Adding ~\scoop\shims to your path.
Scoop was installed successfully!
Type 'scoop help' for instructions.
```

## Install Helm for windows using Scoop

<https://helm.sh/docs/intro/install/>

```PowerShell
scoop install helm
```

output:

```PowerShell
Installing 'helm' (3.12.3) [64bit] from main bucket
helm-v3.12.3-windows-amd64.zip (15.4 MB) [======================================================================================================================] 100%
Checking hash of helm-v3.12.3-windows-amd64.zip ... ok.
Extracting helm-v3.12.3-windows-amd64.zip ... done.
Linking ~\scoop\apps\helm\current => ~\scoop\apps\helm\3.12.3
Creating shim for 'helm'.
'helm' (3.12.3) was installed successfully!
```

## Install ArgoCD

1. Add the ArgoCD Helm repository to Helm: `helm repo add argo https://argoproj.github.io/argo-helm`

    output:

    ```PowerShell
    "argo" has been added to your repositories
    ```

1. Update Helm Repositories: `helm repo update`

    output:

    ```PowerShell
    Hang tight while we grab the latest from your chart repositories...
    ...Successfully got an update from the "argo" chart repository
    Update Complete. ⎈Happy Helming!⎈
    ```

1. Install ArgoCD:

    ```PowerShell
    # using helm withOut a namespace
    helm install argocd argo/argo-cd
    # OR
    # using helm with a namespace and a release name "argocd-release" <-- use this one
    kubectl create namespace argocd
    helm install argocd-release argo/argo-cd --namespace argocd
    # OR Install ArgoCD using KubeCTL
    # https://argo-cd.readthedocs.io/en/stable/getting_started
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```

    output:

    ```PowerShell
    NAME: argocd
    LAST DEPLOYED: Tue Aug 15 11:57:36 2023
    NAMESPACE: default
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    In order to access the server UI you have the following options:

    1. kubectl port-forward service/argocd-server -n default 8080:443

        and then open the browser on http://localhost:8080 and accept the certificate

    2. enable ingress in the values file `server.ingress.enabled` and either
        - Add the annotation for ssl passthrough: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-1-ssl-passthrough
        - Set the `configs.params."server.insecure"` in the values file and terminate SSL at your ingress: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-2-multiple-ingress-objects-and-hosts


    After reaching the UI the first time you can login with username: admin and the random password generated during the installation. You can find the password by running:

    kubectl -n default get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

    (You should delete the initial secret afterwards as suggested by the Getting Started Guide: https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli)
    ```

## Install ArgoCD | verify installation

1. `kubectl get all` (before installation)

    output:

    ```PowerShell
    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   105m
    ```

1. `kubectl get all` (after installation)

    output:

    ```PowerShell
    NAME                                                    READY   STATUS              RESTARTS   AGE
    pod/argocd-application-controller-0                     0/1     ContainerCreating   0          13s
    pod/argocd-applicationset-controller-6c8cd6f9b5-948tj   0/1     ContainerCreating   0          13s
    pod/argocd-dex-server-5f4447b6bc-jfvtb                  0/1     Init:0/1            0          13s
    pod/argocd-notifications-controller-6d95c769fd-5q2ss    0/1     ContainerCreating   0          13s
    pod/argocd-redis-64f5b489bd-6c8fb                       0/1     ContainerCreating   0          13s
    pod/argocd-repo-server-6b5dcfbc46-znj7w                 0/1     Init:0/1            0          13s
    pod/argocd-server-54fdc47648-jnsht                      0/1     ContainerCreating   0          13s

    NAME                                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
    service/argocd-applicationset-controller   ClusterIP   10.109.96.25     <none>        7000/TCP            15s
    service/argocd-dex-server                  ClusterIP   10.106.133.194   <none>        5556/TCP,5557/TCP   15s
    service/argocd-redis                       ClusterIP   10.97.150.156    <none>        6379/TCP            15s
    service/argocd-repo-server                 ClusterIP   10.100.99.187    <none>        8081/TCP            15s
    service/argocd-server                      ClusterIP   10.108.229.17    <none>        80/TCP,443/TCP      15s
    service/kubernetes                         ClusterIP   10.96.0.1        <none>        443/TCP             105m

    NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/argocd-applicationset-controller   0/1     1            0           15s
    deployment.apps/argocd-dex-server                  0/1     1            0           15s
    deployment.apps/argocd-notifications-controller    0/1     1            0           15s
    deployment.apps/argocd-redis                       0/1     1            0           15s
    deployment.apps/argocd-repo-server                 0/1     1            0           15s
    deployment.apps/argocd-server                      0/1     1            0           15s

    NAME                                                          DESIRED   CURRENT   READY   AGE
    replicaset.apps/argocd-applicationset-controller-6c8cd6f9b5   1         1         0       15s
    replicaset.apps/argocd-dex-server-5f4447b6bc                  1         1         0       15s
    replicaset.apps/argocd-notifications-controller-6d95c769fd    1         1         0       15s
    replicaset.apps/argocd-redis-64f5b489bd                       1         1         0       15s
    replicaset.apps/argocd-repo-server-6b5dcfbc46                 1         1         0       15s
    replicaset.apps/argocd-server-54fdc47648                      1         1         0       14s

    NAME                                             READY   AGE
    statefulset.apps/argocd-application-controller   0/1     15s
    ```

1. again after some minutes

    ```PowerShell
    NAME                                                    READY   STATUS    RESTARTS   AGE
    pod/argocd-application-controller-0                     1/1     Running   0          34m
    pod/argocd-applicationset-controller-6c8cd6f9b5-948tj   1/1     Running   0          34m
    pod/argocd-dex-server-5f4447b6bc-jfvtb                  1/1     Running   0          34m
    pod/argocd-notifications-controller-6d95c769fd-5q2ss    1/1     Running   0          34m
    pod/argocd-redis-64f5b489bd-6c8fb                       1/1     Running   0          34m
    pod/argocd-repo-server-6b5dcfbc46-znj7w                 1/1     Running   0          34m
    pod/argocd-server-54fdc47648-jnsht                      1/1     Running   0          34m

    NAME                                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
    service/argocd-applicationset-controller   ClusterIP   10.109.96.25     <none>        7000/TCP            34m
    service/argocd-dex-server                  ClusterIP   10.106.133.194   <none>        5556/TCP,5557/TCP   34m
    service/argocd-redis                       ClusterIP   10.97.150.156    <none>        6379/TCP            34m
    service/argocd-repo-server                 ClusterIP   10.100.99.187    <none>        8081/TCP            34m
    service/argocd-server                      ClusterIP   10.108.229.17    <none>        80/TCP,443/TCP      34m
    service/kubernetes                         ClusterIP   10.96.0.1        <none>        443/TCP             139m

    NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/argocd-applicationset-controller   1/1     1            1           34m
    deployment.apps/argocd-dex-server                  1/1     1            1           34m
    deployment.apps/argocd-notifications-controller    1/1     1            1           34m
    deployment.apps/argocd-redis                       1/1     1            1           34m
    deployment.apps/argocd-repo-server                 1/1     1            1           34m
    deployment.apps/argocd-server                      1/1     1            1           34m

    NAME                                                          DESIRED   CURRENT   READY   AGE
    replicaset.apps/argocd-applicationset-controller-6c8cd6f9b5   1         1         1       34m
    replicaset.apps/argocd-dex-server-5f4447b6bc                  1         1         1       34m
    replicaset.apps/argocd-notifications-controller-6d95c769fd    1         1         1       34m
    replicaset.apps/argocd-redis-64f5b489bd                       1         1         1       34m
    replicaset.apps/argocd-repo-server-6b5dcfbc46                 1         1         1       34m
    replicaset.apps/argocd-server-54fdc47648                      1         1         1       34m

    NAME                                             READY   AGE
    statefulset.apps/argocd-application-controller   1/1     34m
    ```

## Exposing ArgoCD Server

```PowerShell
kubectl port-forward service/argocd-server -n default 8080:443
# OR
kubectl port-forward svc/argocd-server -n argocd 8080:443
# OR
kubectl port-forward -n argocd svc argocd-server 8080:443
```

output:

```PowerShell
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
.
.
Handling connection for 8080
.
.
```

### Getting Password to login to ArgoCD Server

Keep in mind the User is **"Admin"**

```PowerShell
kubectl -n default get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
# OR
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
echo <"paste the password string from the previous command"> | base64 --decode
# OR
kubectl get secret argocd-initial-admin-secret -n default -o yaml
echo <"paste the password string from the previous command"> | base64 --decode
# OR go to
# Getting Password with ArgoCD CLI
```

output:

```PowerShell
# its a secret
# OR 
apiVersion: v1
data:
  password: [its a secret]
kind: Secret
metadata:
  creationTimestamp: "2023-08-15T16:58:19Z"
  name: argocd-initial-admin-secret
  namespace: default
  resourceVersion: "5652"
  uid: 8c638f8d-8fcf-4d7b-a058-09480c344ab5
type: Opaque
```

OR

#### Install ArgoCD CLI using PowerShell

<https://argo-cd.readthedocs.io/en/stable/cli_installation/>

Download With PowerShell: Invoke-WebRequest¶
You can view the latest version of Argo CD at the link above or run the following command to grab the version:

```PowerShell
$version = (Invoke-RestMethod https://api.github.com/repos/argoproj/argo-cd/releases/latest).tag_name
```

Replace `$version` in the command below with the version of Argo CD you would like to download:

```PowerShell
$url = "https://github.com/argoproj/argo-cd/releases/download/" + $version + "/argocd-windows-amd64.exe"
$output = "argocd.exe"

Invoke-WebRequest -Uri $url -OutFile $output
```

Also please note you will probably need to move the file into your PATH. Use following command to add Argo CD into environment variables PATH

```PowerShell
[Environment]::SetEnvironmentVariable("Path", "$env:Path;C:\Path\To\ArgoCD-CLI", "User")
```

After finishing the instructions above, you should now be able to run `argocd` commands.

#### Getting Password with ArgoCD CLI

The initial password for the admin account is auto-generated and stored as clear text in the field password in a secret named argocd-initial-admin-secret in your Argo CD installation namespace. You can simply retrieve this password using the argocd CLI:

```PowerShell
argocd admin initial-password -n argocd
```

output:

```PowerShell
<its a secret>

 This password must be only used for first time login. We strongly recommend you update the password using `argocd account update-password`.
```
