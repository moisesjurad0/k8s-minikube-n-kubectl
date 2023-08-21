# K8s - Namespaces - 4rd README file

## what is namespace

1. what is namespace?
    1. organise resources in namespaces
        1. you can have multiple namespaces in a cluster
    1. you can think of it like a virtual cluster inside  a cluster
1. what are the use cases?
1. How Namespaces work and how to use it?

## Four OutOfTheBox Namespaces

1. default <--
1. kube-node-lease
    - it's a recent addition to k8s
    - holds information about heartbeats of nodes
    - each node has associated lease object in namespace
1. kube-public
    - publicly accesible data
    - contains a config map wich contains cluster information (is accesible even without authentication)

    ```sh
    kubectl cluster-info                                                                                      pwsh   100  14:44:53 
    ```

    output:

    ```sh
    Kubernetes control plane is running at https://127.0.0.1:59879
    CoreDNS is running at https://127.0.0.1:59879/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
    ```

1. kube-system
    - DO NOT create or modify in kube-system
    - System Processes
    - Master and kubectl processes
1. kubernetes-dashboard
    - specific only with minikube.
    - You will not have this in a standar cluster.

### default
