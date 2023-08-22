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
    1. As in the example in "Case 2 | Many teams, same application"
        1. you can give each team access only to theri namespaces this way you isolate each team by their respective namespace
        1. each team has own isolated environment
        1. you can also limit the resources (CPU, RAM, Storage) in each team namespace
            - you can define resource quotas per namespace

### Summary

1. **Structure** your components
1. **Avoid conflicts** between teams
1. **Share services** between different environments
1. **Access and Resource Limits** on Namespaces Level

## Characteristics of Namespaces

1. You can't access most resources from another namespace
    1. example
        1. Project A Namespace
            - ConfigMap with a reference to "DB ns Service"
        1. Project A Namespace
            - Can't use the "Project A ns" ConfiMap
            - Each NS must define own ConfigMap
        1. DB Namespace
            - Service
    1. the same applies to secrets
1. Access Service in another Namespace
    1. but you can access services from another ns

        example: you have to use this notation in the db_url `service.namespace`. This way you can reference services in another ns.

        ```yaml
        apiVersion: v1
        kind: ConfigMap
        metadata:
            name: mysql-configmap
        data:
            db_url: mysql-service.DB
        ```

1. Components, wich can't be created within a Namespace
    - they just live globally in the cluster and you can't isolate them or put them in a certain ns
    - examples:
        - volume or persistent volume
        - node
    - you can list resources that are not in a ns using:

        ```sh
        kubectl api-resources --namespaced=false
        ```

        ```sh
        NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
        componentstatuses                 cs           v1                                     false        ComponentStatus
        namespaces                        ns           v1                                     false        Namespace
        nodes                             no           v1                                     false        Node
        persistentvolumes                 pv           v1                                     false        PersistentVolume
        mutatingwebhookconfigurations                  admissionregistration.k8s.io/v1        false        MutatingWebhookConfiguration
        validatingwebhookconfigurations                admissionregistration.k8s.io/v1        false        ValidatingWebhookConfiguration
        customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1                false        CustomResourceDefinition
        apiservices                                    apiregistration.k8s.io/v1              false        APIService
        tokenreviews                                   authentication.k8s.io/v1               false        TokenReview
        selfsubjectaccessreviews                       authorization.k8s.io/v1                false        SelfSubjectAccessReview
        selfsubjectrulesreviews                        authorization.k8s.io/v1                false        SelfSubjectRulesReview
        subjectaccessreviews                           authorization.k8s.io/v1                false        SubjectAccessReview
        certificatesigningrequests        csr          certificates.k8s.io/v1                 false        CertificateSigningRequest
        flowschemas                                    flowcontrol.apiserver.k8s.io/v1beta3   false        FlowSchema
        prioritylevelconfigurations                    flowcontrol.apiserver.k8s.io/v1beta3   false        PriorityLevelConfiguration
        ingressclasses                                 networking.k8s.io/v1                   false        IngressClass
        runtimeclasses                                 node.k8s.io/v1                         false        RuntimeClass
        clusterrolebindings                            rbac.authorization.k8s.io/v1           false        ClusterRoleBinding
        clusterroles                                   rbac.authorization.k8s.io/v1           false        ClusterRole
        priorityclasses                   pc           scheduling.k8s.io/v1                   false        PriorityClass
        csidrivers                                     storage.k8s.io/v1                      false        CSIDriver
        csinodes                                       storage.k8s.io/v1                      false        CSINode
        storageclasses                    sc           storage.k8s.io/v1                      false        StorageClass
        volumeattachments                              storage.k8s.io/v1                      false        VolumeAttachment
        ```

    - also you can do the oposite:

        ```sh
        kubectl api-resources --namespaced=true
        ```

        ```sh
        NAME                        SHORTNAMES         APIVERSION                       NAMESPACED   KIND
        bindings                                       v1                               true         Binding
        configmaps                  cm                 v1                               true         ConfigMap
        endpoints                   ep                 v1                               true         Endpoints
        events                      ev                 v1                               true         Event
        limitranges                 limits             v1                               true         LimitRange
        persistentvolumeclaims      pvc                v1                               true         PersistentVolumeClaim
        pods                        po                 v1                               true         Pod
        podtemplates                                   v1                               true         PodTemplate
        replicationcontrollers      rc                 v1                               true         ReplicationController
        resourcequotas              quota              v1                               true         ResourceQuota
        secrets                                        v1                               true         Secret
        serviceaccounts             sa                 v1                               true         ServiceAccount
        services                    svc                v1                               true         Service
        controllerrevisions                            apps/v1                          true         ControllerRevision
        daemonsets                  ds                 apps/v1                          true         DaemonSet
        deployments                 deploy             apps/v1                          true         Deployment
        replicasets                 rs                 apps/v1                          true         ReplicaSet
        statefulsets                sts                apps/v1                          true         StatefulSet
        applications                app,apps           argoproj.io/v1alpha1             true         Application
        applicationsets             appset,appsets     argoproj.io/v1alpha1             true         ApplicationSet
        appprojects                 appproj,appprojs   argoproj.io/v1alpha1             true         AppProject
        localsubjectaccessreviews                      authorization.k8s.io/v1          true         LocalSubjectAccessReview
        horizontalpodautoscalers    hpa                autoscaling/v2                   true         HorizontalPodAutoscaler
        cronjobs                    cj                 batch/v1                         true         CronJob
        jobs                                           batch/v1                         true         Job
        leases                                         coordination.k8s.io/v1           true         Lease
        endpointslices                                 discovery.k8s.io/v1              true         EndpointSlice
        events                      ev                 events.k8s.io/v1                 true         Event
        alertmanagerconfigs         amcfg              monitoring.coreos.com/v1alpha1   true         AlertmanagerConfig
        alertmanagers               am                 monitoring.coreos.com/v1         true         Alertmanager
        podmonitors                 pmon               monitoring.coreos.com/v1         true         PodMonitor
        probes                      prb                monitoring.coreos.com/v1         true         Probe
        prometheusagents            promagent          monitoring.coreos.com/v1alpha1   true         PrometheusAgent
        prometheuses                prom               monitoring.coreos.com/v1         true         Prometheus
        prometheusrules             promrule           monitoring.coreos.com/v1         true         PrometheusRule
        scrapeconfigs               scfg               monitoring.coreos.com/v1alpha1   true         ScrapeConfig
        servicemonitors             smon               monitoring.coreos.com/v1         true         ServiceMonitor
        thanosrulers                ruler              monitoring.coreos.com/v1         true         ThanosRuler
        ingresses                   ing                networking.k8s.io/v1             true         Ingress
        networkpolicies             netpol             networking.k8s.io/v1             true         NetworkPolicy
        poddisruptionbudgets        pdb                policy/v1                        true         PodDisruptionBudget
        rolebindings                                   rbac.authorization.k8s.io/v1     true         RoleBinding
        roles                                          rbac.authorization.k8s.io/v1     true         Role
        csistoragecapacities                           storage.k8s.io/v1                true         CSIStorageCapacity
        ```

## Create Component in a ns

1. using parammeter `--namespace` with kubectl or `-n`

    ```sh
    kubectl apply -f file.yaml --namespace=my-namespace
    # replace file.yaml by the yaml definition of your component.
    # repplace my-namespace by the ns your want to use.
    ```

1. using a namespace tag inside a yaml file

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
        name: mysql-configmap
        namespace: my-namespace
    data:
        db_url: mysql-service.DB
    ```

    - this way is a better option because is well documented

## Change active ns

- example: a team that only have permissions in its namespace "my-namespace", i'd be annoying add `-n my-namespace` to all commands
- Change active ns with **kubens**. k8s nor kubectl have an option for this feature.

```sh
# kubens lives inside the kubectx repo in github. Both lives there.

# https://github.com/ahmetb/kubectx
# kubectx is a tool to switch between contexts (clusters) on kubectl faster.
# kubens is a tool to switch between Kubernetes namespaces (and configure them for kubectl) easily.

# note that both are a combination of "kube" and  "ctx" / "ns" that stands for Context and Namespace

# with Chocolatey (Windows)
choco install kubens kubectx

# Windows Installation (using Scoop)
scoop bucket add main
scoop install main/kubens main/kubectx

# with winget (Windows)
winget install --id ahmetb.kubectx
winget install --id ahmetb.kubens
```

output using winget kubectx

```PowerShell
Found kubectx [ahmetb.kubectx] Version 0.9.5
This application is licensed to you by its owner.
Microsoft is not responsible for, nor does it grant any licenses to, third-party packages.
Downloading https://github.com/ahmetb/kubectx/releases/download/v0.9.5/kubectx_v0.9.5_windows_x86_64.zip
  ██████████████████████████████  1.05 MB / 1.05 MB
Successfully verified installer hash
Extracting archive...
Successfully extracted archive
Starting package install...
Path environment variable modified; restart your shell to use the new value.
Command line alias added: "kubectx"
Successfully installed
```

output using winget kubens

```PowerShell
Found kubens [ahmetb.kubens] Version 0.9.5
This application is licensed to you by its owner.
Microsoft is not responsible for, nor does it grant any licenses to, third-party packages.
Downloading https://github.com/ahmetb/kubectx/releases/download/v0.9.5/kubens_v0.9.5_windows_x86_64.zip
  ██████████████████████████████  9.58 MB / 9.58 MB
Successfully verified installer hash
Extracting archive...
Successfully extracted archive
Starting package install...
Path environment variable modified; restart your shell to use the new value.
Command line alias added: "kubens"
Successfully installed
```

### kubens and kubectx mini demos

kubens
![Image01](<https://github.com/ahmetb/kubectx/raw/master/img/kubens-demo.gif>)
(extracted from <https://github.com/ahmetb/kubectx>)

kubectx
![Image02](<https://github.com/ahmetb/kubectx/raw/master/img/kubectx-demo.gif>)
(extracted from <https://github.com/ahmetb/kubectx>)
