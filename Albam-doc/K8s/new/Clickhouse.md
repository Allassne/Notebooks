## Présentation

**Clickhouse** comme base de données analytique pour collecter toutes les données transitant par nos services avec un moteur d'indexation et de recherche comme **Meilisearch**.



Quel est le problème que Clickhouse résout ?
 


Comment Clickhouse fonctionne ?



Quels sont les avantages et les inconvénients de Clickhouse ?



Comment Clickhouse peut être utilisée ?



À qui s'adresse Clickhouse ?





Quel est le problème que Clickhouse résout ?
Comment Clickhouse fonctionne ?
Quels sont les avantages et les inconvénients de Clickhouse ?
Comment Clickhouse peut être utilisée ?
À qui s'adresse Clickhouse ?



quel est Utilité de cette technos dans un cluster Kubernetes ?


```
helm install my-release oci://registry-1.docker.io/bitnamicharts/clickhouse
```






git clone https://github.com/Altinity/clickhouse-operator

kubectl apply -f clickhouse-operator-install.yaml






https://docs.altinity.com/altinitykubernetesoperator/kubernetesquickstartguide/quickstartinstallation/

```
kubectl apply -f https://github.com/Altinity/clickhouse-operator/raw/0.18.3/deploy/operator/clickhouse-operator-install-bundle.yaml
```

```
kubectl get pods --namespace kube-system
```


Désinstaller 

```
kubectl delete --namespace "kube-system" -f https://github.com/Altinity/clickhouse-operator/raw/0.18.3/deploy/operator/clickhouse-operator-install-bundle.yaml
```


```
kubectl delete -f https://github.com/Altinity/clickhouse-operator/raw/0.18.3/deploy/operator/clickhouse-operator-install-bundle.yaml
```





```
helm repo add liwenhe https://liwenhe1993.github.io/charts/
helm repo update
helm install --name clickhouse liwenhe/clickhouse
```


helm show values clickhouse/clickhouse > loki.yaml
helm show values liwenhe/clickhouse > clickhouse.yaml

helm install clickhouse liwenhe/clickhouse -f clickhouse.yaml


```
helm delete --purge clickhouse
```




NAME: clickhouse
LAST DEPLOYED: Wed Dec 13 20:28:32 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

1. Get the Clickhouse URL by running:

  echo URL            : http://127.0.0.1:8080/
  echo Management URL : http://127.0.0.1:8080/manager
  kubectl port-forward --namespace default svc/clickhouse 8123:8123
  kubectl port-forward --namespace default svc/clickhouse 9000:9000
  kubectl port-forward --namespace default svc/clickhouse 9009:9009

2. Get the Tabix URL by running:

  kubectl port-forward --namespace default svc/clickhouse-tabix 80
