# k8s-minikube-n-kubectl

This workshop will include steps for using minikube & kubectl on Windows 11

links reference:

- <https://minikube.sigs.k8s.io/docs/start/>
- <https://minikube.sigs.k8s.io/docs/drivers/>
- <https://minikube.sigs.k8s.io/docs/drivers/docker/>
- <https://youtu.be/s_o8dwzRlu4>
- <https://gitlab.com/nanuchi/k8s-in-1-hour>
- <https://hub.docker.com/r/nanajanashia/k8s-demo-app>

## requirements

1. install docker for Windows

## configuration

1. run in terminal: `winget install minikube`
1. run in terminal: `minikube start --driver=docker`
    - error output:

        ```txt
        minikube: The term 'minikube' is not recognized as a name of a cmdlet, function, script file, or executable program.
        Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
        ```

    - if you have this error try to restart the windows terminal or follow these link(s):
        - <https://minikube.sigs.k8s.io/docs/drivers/>
            - <https://minikube.sigs.k8s.io/docs/drivers/docker/>
                - <https://minikube.sigs.k8s.io/docs/tutorials/wsl_docker_driver/>
    - success output:

        ![Image01](<https://github.com/moisesjurad0/k8s-minikube-n-kubectl/raw/main/images/Screenshot 2023-07-19 211348.png>)

        ![Image02](<https://github.com/moisesjurad0/k8s-minikube-n-kubectl/raw/main/images/Screenshot 2023-07-19 205259.png>)

1. run in terminal: `minikube status`
    - output:

        ```shell
        minikube
        type: Control Plane
        host: Running
        kubelet: Running
        apiserver: Running
        kubeconfig: Configured
        ```

1. we don't need to install kubectl because it is a dependency within the minikube installation.
1. run in terminal: `kubectl get node`
    - output:

        ```shell
        NAME       STATUS   ROLES           AGE   VERSION
        minikube   Ready    control-plane   37m   v1.26.3
        ```

## test deploying on minikube

1. download these files from <https://gitlab.com/nanuchi/k8s-in-1-hour>:
    1. mongo-config.yaml
    1. mongo-secret.yaml
    1. mongo.yaml
    1. webapp.yaml
1. run in terminal: `kubectl get pod`
    - output:

        ```shell
        No resources found in default namespace.
        ```

    - it means minikube is running but no component are deployed yet
1. run line by line in this order:

     ```shell
    kubectl apply -f mongo-config.yaml
    kubectl apply -f mongo-secret.yaml
    kubectl apply -f mongo.yaml
    kubectl apply -f webapp.yaml
    ```

1. run:`kubectl get all` to verify success creating components
    - output:

        ```shell
        NAME                                    READY   STATUS              RESTARTS   AGE
        pod/mongo-deployment-855b879886-4vcvk   0/1     ContainerCreating   0          12s
        pod/webapp-deployment-9fccd564d-2xc2r   0/1     ContainerCreating   0          5s

        NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
        service/kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP          5h20m
        service/mongo-service    ClusterIP   10.102.132.76   <none>        27017/TCP        12s
        service/webapp-service   NodePort    10.103.35.95    <none>        3000:30100/TCP   5s

        NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/mongo-deployment    0/1     1            0           12s
        deployment.apps/webapp-deployment   0/1     1            0           5s

        NAME                                          DESIRED   CURRENT   READY   AGE
        replicaset.apps/mongo-deployment-855b879886   1         1         0       12s
        replicaset.apps/webapp-deployment-9fccd564d   1         1         0       5s
        ```

1. run: `kubectl get configmap`
    - output:

        ```shell
        NAME               DATA   AGE
        kube-root-ca.crt   1      5h25m
        mongo-config       1      5m59s
        ```

1. run: `kubectl get secret`

    - output:

        ```shell
        NAME           TYPE     DATA   AGE
        mongo-secret   Opaque   2      6m13s
        ```

1. get the external ip address
    1. run: `minikube ip`
        - output:

            ```shell
            192.168.49.2
            ```

    1. run: `kubectl get node -o wide`
    - output:

        ```shell
        NAME       STATUS   ROLES           AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION                       CONTAINER-RUNTIME
        minikube   Ready    control-plane   5h31m   v1.26.3   192.168.49.2   <none>        Ubuntu 20.04.5 LTS   5.10.102.1-microsoft-standard-WSL2   docker://23.0.2
        ```

1. access app in 192.168.49.2:30100
