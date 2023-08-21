# K8s - Namespaces - 4rd README file

## what is namespace

1. what is namespace?
    1. organise resources in namespaces
        1. you can have multiple namespaces in a cluster
    1. you can think of it like a virtual cluster inside  a cluster
1. what are the use cases?
1. How Namespaces work and how to use it?

## Four OutOfTheBox Namespaces

1. default
1. kube-node-lease
1. kube-public
1. kube-system
1. kubernetes-dashboard <-- You will not have this in a standar cluster

## Explanation of Namespaces created by default

1. default
    - resources you create are located here (by default if no haven't create another namespace and specify it on the resource creation)
1. kube-node-lease <-- (it's a recent addition to k8s)
    - holds information about heartbeats of nodes
    - each node has associated lease object in namespace
    - determines the availability of a node
1. kube-public
    - publicly accesible data
    - contains a config map wich contains cluster information (is accesible even without authentication)

    ```sh
    kubectl cluster-info
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

## Create a Namespace

```sh
# command to create namespace:
kubectl create namespace <my-namespace>
```

```sh
# output:
namespace/my-namespace created
```

## List namespaces

```sh
# command:
kubectl get namespaces
```

```sh
# output:
NAME              STATUS   AGE
apph3             Active   2d7h
argocd            Active   2d20h
default           Active   2d20h
kube-node-lease   Active   2d20h
kube-public       Active   2d20h
kube-system       Active   2d20h
my-namespace      Active   71s
```

## create namespace with configuration file

use a k8s yaml definition file containing the namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: your-namespace-name
```

```sh
# then apply the k8s file
kubectl apply -f namespace.yaml
```

you can't use namespaces that don't exists in configuration files

## Why to use Namespaces | Use Cases

1. Case 1 | Everything in one Namespace
    - it's gonna be really difficult to have an overview of what's in there specially if you have multiple users
1. Case 1 | Resources Grouped in Namespaces
    1. Example: you could create
        - "Database" namespace and
        - "Monitoring" namespace and put prometheus, grafana and all related. you could also have
        - "Elastic Stack" namespace and
        - "Nginx-Ingress" namespace
    1. according to officially documentation you shouldn't use namespaces for smaller projects / up to then users.
        - but nana belive it's always good idea to group the resources in namespaces.
1. Case 2 | Many teams, same application
    1. Example: you have 2 teams that use the same cluster.
        - team 1, deploys my-app deployment
        - team 2, deploys my-app deployment (second team is gonna overwrite the first deployment)
            - if the're using some CI/CD tool (jenkins, argoCD) they aren't gonna know that they have overwritten the other team deployment.
    1. solution: Each team could work in their own namespace.
1. Case 3 | Resource Sharing: Staging and Development
    1. Example: you have 1 cluster and you want to host staging and development environment
        - Staging Namespace
        - Development Namespace
        - Nginx-Ingress Controller Namespace <-- As it is in the same cluster as both environment ns you can access'em
        - Elastic Stack Namespace <-- As it is in the same cluster as both environment ns you can access'em
    1. this way you don't have to deploy this Nginx-Ingress Controller and Elastic Stack twice in the same cluster.
1. Case 3 | Resource Sharing: Blue/Green Development
    1. Example: in the same cluster you need to have 2 versions of production.
        - Production Blue Namespace <-- the one that is active
        - Production Green Namespace <-- the other one that is gonna be the next production version.
    1. these namespaces (both) might need to use the same resources like:
        - Nginx-Ingress Controller Namespace <-- As it is in the same cluster as both environment ns you can access'em
        - Elastic Stack Namespace <-- As it is in the same cluster as both environment ns you can access'em
1. Case 4 | Access and Resource Limits on Namespaces
    1.  
