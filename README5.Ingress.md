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

## try accessing k8s dashboard with ingress

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

    
