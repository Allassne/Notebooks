
# Installation de Helm

Ajoutez le référentiel stable à votre installation Helm:
```
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

```
helm repo update
```

Ensuite, nous allons créer un custom namespace sur notre cluster K8s pour gérer toute la pile de surveillance:

```
kubectl create ns monitoring
```

## Installer Prometheus

Nous pouvons maintenant installer le chart Prometheus dans namespace monitoring nouvellement créé

```
helm install prometheus stable/prometheus --namespace monitoring
```
