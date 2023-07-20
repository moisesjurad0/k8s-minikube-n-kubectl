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
