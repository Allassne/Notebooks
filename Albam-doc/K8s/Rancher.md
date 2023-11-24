
# Installe rancher pour gérer le cluster

### Installation sur centos

sudo docker run --privileged -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher

### Installation avec helm 

```
helm repo add jetstack https://charts.jetstack.io
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm repo update
```

```
helm install cert-manager jetstack/cert-manager  --namespace cert-manager  --create-namespace  --version v1.7.1 --set installCRDs=true
kubectl get pods --namespace cert-manager
```

```
helm upgrade -i rancher rancher-stable/rancher --create-namespace  --namespace cattle-system --version 2.7   --set bootstrapPassword=admin  --set hostname=rancher.synelia.ci
```

REMARQUE : L'initialisation complète de Rancher peut prendre plusieurs minutes. Veuillez rester en attente pendant que les certificats sont émis, que les conteneurs sont démarrés et que la règle Ingress apparaît.

Consultez nos documents sur https://rancher.com/docs/

Si vous avez fourni votre propre mot de passe d'amorçage lors de l'installation, accédez à https://rancher.synelia.ci pour commencer.

Si c'est la première fois que vous installez Rancher, commencez par exécuter cette commande et en cliquant sur l'URL qu'elle génère :


```
echo https://rancher.synelia.ci/dashboard/?setup=$(kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}')
```

Pour obtenir uniquement le mot de passe d'amorçage, exécutez :

```
kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'
```


Happy Containering!


