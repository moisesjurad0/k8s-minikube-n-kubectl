# K8s - Ingress - 5th README file

redirects traffic from ing to internal svc

## Activate Ingress in minikube

```sh
minikube addons enable ingress
```

ouput

```sh
W0822 16:29:20.154044   11580 main.go:291] Unable to resolve the current Docker CLI context "default": context "default" does not exist
💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
💡  After the addon is enabled, please run "minikube tunnel" and your ingress resources would be available at "127.0.0.1"
    ▪ Using image registry.k8s.io/ingress-nginx/controller:v1.7.0
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230312-helm-chart-4.5.2-28-g66a760794
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230312-helm-chart-4.5.2-28-g66a760794
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```

---

verify components in kube-system

```sh
kubectl get pod -n kube-system
```

ouput

```sh
NAME                               READY   STATUS    RESTARTS        AGE
coredns-787d4945fb-zpsf5           1/1     Running   0               3d21h
etcd-minikube                      1/1     Running   0               3d21h
kube-apiserver-minikube            1/1     Running   0               3d21h
kube-controller-manager-minikube   1/1     Running   0               3d21h
kube-proxy-wvmmt                   1/1     Running   0               3d21h
kube-scheduler-minikube            1/1     Running   0               3d21h
storage-provisioner                1/1     Running   1 (3d21h ago)   3d21h
```

---

verify components in freshly created ingress-nginx namespace

```sh
kubectl get pod -n ingress-nginx
```

ouput

```sh
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-m6fr8        0/1     Completed   0          11m
ingress-nginx-admission-patch-wc6sz         0/1     Completed   1          11m
ingress-nginx-controller-6cc5ccb977-wvvkc   1/1     Running     0          11m
```

## try accessing k8s dashboard with ingress (in Minikube)

1. activate k8s dashboard (in minikube it can be done using this)

    ```sh
    minikube dashboard
    ```

1. verify minikube dashboard ns

    ```sh
    kubectl get namespaces
    ```

    output

    ```sh
    NAME                   STATUS   AGE
    apph3                  Active   3d9h
    argocd                 Active   3d21h
    default                Active   3d21h
    ingress-nginx          Active   22m
    kube-node-lease        Active   3d21h
    kube-public            Active   3d21h
    kube-system            Active   3d21h
    kubernetes-dashboard   Active   90s
    my-namespace           Active   25h
    your-namespace-name    Active   24h
    ```

1. verify componentes inside kubernetes-dashboard ns

    ```sh
    kubectl get all -n kubernetes-dashboard
    ```

    output

    ```sh
    NAME                                            READY   STATUS    RESTARTS   AGE
    pod/dashboard-metrics-scraper-5c6664855-2bj8r   1/1     Running   0          107s
    pod/kubernetes-dashboard-55c4cbbc7c-cr79q       1/1     Running   0          107s

    NAME                                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
    service/dashboard-metrics-scraper   ClusterIP   10.97.182.96   <none>        8000/TCP   107s
    service/kubernetes-dashboard        ClusterIP   10.96.41.75    <none>        80/TCP     107s

    NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/dashboard-metrics-scraper   1/1     1            1           108s
    deployment.apps/kubernetes-dashboard        1/1     1            1           108s

    NAME                                                  DESIRED   CURRENT   READY   AGE
    replicaset.apps/dashboard-metrics-scraper-5c6664855   1         1         1       108s
    replicaset.apps/kubernetes-dashboard-55c4cbbc7c       1         1         1       108s
    ```

1. write the yaml definition file to create the ingress component in order to access the internal svc "service/kubernetes-dashboard" (listed in the previous step)

    version from utub

    ```yaml
    apiVersion: networking.k8s.io/v1beta1
    kind: Ingress
    metadata:
      name: dashboard-ingress
      namespace: kubernetes-dashboard
    spec:
      rules:
        - host: dashboard.com
          http:
            paths:
              - backend:
                  serviceName: kubernetes-dashboard
                  servicePort: 80

    ```

    apply configuration

    ```yaml
    kubectl apply -f .\dashboard-ingress.yaml
    ```

    output

    ```yaml
    error: resource mapping not found for name: "dashboard-ingress" namespace: "kubernetes-dashboard" from ".\\dashboard-ingress.yaml": no matches for kind "Ingress" in version "networking.k8s.io/v1beta1"
    ensure CRDs are installed first
    ```

    ---

    version from repo

    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: dashboard-ingress
      namespace: kubernetes-dashboard
      annotations:
        kubernetes.io/ingress.class: "nginx"
    spec:
      rules:
        - host: dashboard.com
          http:
            paths:
              - path: /
                pathType: Exact
                backend:
                  service:
                    name: kubernetes-dashboard
                    port:
                      number: 80
    ```

    apply configuration

    ```yaml
    kubectl apply -f .\dashboard-ingress.yaml
    ```

    output

    ```yaml
    ingress.networking.k8s.io/dashboard-ingress created
    ```

    <https://youtu.be/X48VuDVv0do> 2:16:13 | <https://youtu.be/X48VuDVv0do?t=8173>

    verify

    ```yaml
    kubectl get ingress -n kubernetes-dashboard
    ```

    output

    ```yaml
    NAME                CLASS    HOSTS           ADDRESS        PORTS   AGE
    dashboard-ingress   <none>   dashboard.com   192.168.49.2   80      8m19s
    ```

    can use --watch to wait

  %SystemRoot%\System32\drivers\etc\hosts

## Try Ingress-Nginx Controller from Windows11 docker-desktop k8s built-in environment

using this guide : <https://kubernetes.github.io/ingress-nginx/deploy/>

```sh
# use helm to install Ingress-Nginx Controller
helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace
# or kubectl 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml


# Pre-flight check
kubectl get pods --namespace=ingress-nginx
# and this
kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=120s


# CREATE a Deployment & Service to test 
kubectl create deployment demo --image=httpd --port=80
kubectl expose deployment demo
# THEN create an Ingress resource
kubectl create ingress demo-localhost --class=nginx --rule="demo.localdev.me/*=demo:80"


# Now, forward a local port to the ingress controller:
kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80


# At this point, you can access your deployment using curl
curl --resolve demo.localdev.me:8080:127.0.0.1 http://demo.localdev.me:8080
```

### k8s references

ingress

- <https://kubernetes.io/docs/concepts/services-networking/ingress/>
- <https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/>
- <https://kubernetes.github.io/ingress-nginx/deploy/>

dashboard

- <https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/>
- <https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md>

minikube

- <https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/>

argocd

- <https://argo-cd.readthedocs.io/en/stable/>
- <https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd_app_create/>

### command summary

```sh
# install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


# delete ArgoCD
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl delete namespace argocd
```

```sh
# install kubernetes-dashboard 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml


# delete kubernetes-dashboard installation
kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

```sh
# kubectl port-forward explanation
kubectl port-forward <pod-name> <local-port>:<remote-port>
# <pod-name>: The name of the pod you want to create the port-forward for.
# <local-port>: The port on your local machine that you want to use to access the forwarded service.
# <remote-port>: The port inside the pod that you want to forward to.


# example
# forward this
kubectl port-forward backend-service 9090:8081
# any requests you make to
http://localhost:9090
# will be forwarded to the service running on port 8081 inside the pod in the k8s cluster.
```

```sh
# PORT(S) Columns

# 1. Container Port: If the resource is a Pod, the PORT(S) column might display the port(s) that the containers within the pod are listening on. For example, if your pod has a container running a web server on port 80, you might see 80/TCP in the PORT(S) column.

# 2. Service Ports: If the resource is a Service, the PORT(S) column could show the ports exposed by that service. A Kubernetes Service allows you to expose a set of pods as a network service. If you have a Service that exposes ports 80 and 443, you might see 80:30800/TCP,443:30443/TCP in the PORT(S) column. The format is externalPort:internalPort/protocol.

# 3. Ingress Ports: If you have an Ingress resource, it could list the ports being routed by the Ingress controller. Ingress is a way to manage external access to the services in your cluster. The ports listed might indicate which external ports are being routed to which internal services.
```

---

```sh
# generate ssl cert from gitbash
openssl genpkey -algorithm RSA -out private.key
openssl req -new -key private.key -out certificate.csr
openssl x509 -req -days 365 -in certificate.csr -signkey private.key -out certificate.crt
base64 -w 0 certificate.crt > tls.crt
base64 -w 0 private.key > tls.key
```

apply here in this secret object

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dashboard-secret-tls
  namespace: your-namespace
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-certificate-data>
  tls.key: <base64-encoded-private-key-data>
```

using

```sh
kubectl apply -f <tls-secret.yaml>
```

---

```sh
kubectl get all -n kubernetes-dashboard
```

output

```sh
NAME                                             READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-5cb4f4bb9c-q4pzk   1/1     Running   0          17h
pod/kubernetes-dashboard-6967859bff-6j68s        1/1     Running   0          17h

NAME                                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   10.101.56.96   <none>        8000/TCP   17h
service/kubernetes-dashboard        ClusterIP   10.106.74.9    <none>        443/TCP    17h

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           17h
deployment.apps/kubernetes-dashboard        1/1     1            1           17h

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-5cb4f4bb9c   1         1         1       17h
replicaset.apps/kubernetes-dashboard-6967859bff        1         1         1       17h
```

to try dashboard

```sh
kubectl port-forward -n kubernetes-dashboard svc/kubernetes-dashboard 4545:443
```

```sh
Forwarding from 127.0.0.1:4545 -> 8443
Forwarding from [::1]:4545 -> 8443
Handling connection for 4545
Handling connection for 4545
Handling connection for 4545
Handling connection for 4545
Handling connection for 4545
Handling connection for 4545
Handling connection for 4545
```
