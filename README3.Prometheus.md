# K8s - Prometheus - 3rd README file

## Command Summary

having alredy installed helm chart for kube-prometheus-stack (also known as the good old "prometheus operator")

can use argocd to install it with a chart. See <https://github.com/moisesjurad0/k8s-argoCD-app-of-apps-definition/tree/main/apph3>

1. kubectl logs pr-op-grafana-7b4cb67f8f-cbhbk -n argocd
1. kubectl port-forward deployment.apps/pr-op-grafana -n argocd 3000 <-- access grafana UI
1. kubectl port-forward service/arcd-argocd-server -n argocd 8080:443 <-- access argocd UI
1. argocd admin initial-password -n argocd
1. kubectl port-forward prometheus-pr-op-kube-prometheus-stac-prometheus-0 -n argocd 9090 <-- access prometheus UI
