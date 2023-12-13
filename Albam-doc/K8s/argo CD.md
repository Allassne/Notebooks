# Déploiement avec ArgoCD

## [](#installation-argocd)Installation ArgoCD

1- Ajoutez le repo

```
helm repo add argo https://argoproj.github.io/argo-helm
```

2- Modifier les valeurs de la charte comme suit :

```
helm show values argo/argo-cd > argocd.yml
```

```
  ingress:
    # -- Enable an ingress resource for the Argo CD server
    enabled: true
    # -- Additional ingress annotations
    annotations: {}
    # -- Additional ingress labels
    labels: {}
    # -- Defines which ingress controller will implement the resource
    ingressClassName: "traefik"

    # -- List of ingress hosts
    ## Argo Ingress.
    ## Hostnames must be provided if Ingress is enabled.
    ## Secrets must be manually created in the namespace
    hosts:
      - argocd.dev01.ovh.smile.lan


    server.insecure: true
    # -- Value for base href in index.html. Used if Argo CD is running behind reverse proxy under subpath different from /
    server.basehref: /
    # -- Used if Argo CD is running behind reverse proxy under subpath different from /
    server.rootpath: ''
    # -- Directory path that contains additional static assets
    server.staticassets: /shared/app
    # -- Disable Argo CD RBAC for user authentication
    server.disable.auth: false
    # -- Enable GZIP compression
    server.enable.gzip: true
    # -- Set X-Frame-Options header in HTTP responses to value. To disable, set to "".
    #server.x.frame.options: sameorigin
    server.x.frame.options: ""
```

3- Déploiement de argoCD

```
helm upgrade -i argocd argo/argo-cd -f argocd.yml
```

4- Une fois le déploiement terminé, il vous est possible de voir les pods comme suit :

```
root@rodjeMaster1:~/argocd# kubectl get pods | grep argo
argocd-application-controller-0                       1/1     Running   1 (7m43s ago)    12h
argocd-applicationset-controller-54dcc45d9f-z5rnz     1/1     Running   2 (6m44s ago)    12h
argocd-dex-server-5b5cbc74c-jjj8g                     1/1     Running   1 (7m44s ago)    12h
argocd-notifications-controller-89876c7ff-lwls6       1/1     Running   1 (7m41s ago)    12h
argocd-redis-75cbc4b785-45ld9                         1/1     Running   1 (7m42s ago)    12h
argocd-repo-server-85d5cc6d55-sjfnw                   1/1     Running   1 (7m43s ago)    12h
argocd-server-67944c8d-86blj                          1/1     Running   2 (6m35s ago)    12h
```

Le mot de passe par défaut de l'utilisateur admin peut être trouvé comme suit :

```
kubectl -n default get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```










In order to access the server UI you have the following options:

1. kubectl port-forward service/argocd-server -n default 8080:443

    and then open the browser on http://localhost:8080 and accept the certificate

2. enable ingress in the values file `server.ingress.enabled` and either
      - Add the annotation for ssl passthrough: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-1-ssl-passthrough
      - Set the `configs.params."server.insecure"` in the values file and terminate SSL at your ingress: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-2-multiple-ingress-objects-and-hosts


After reaching the UI the first time you can login with username: admin and the random password generated during the installation. You can find the password by running:

kubectl -n default get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

(You should delete the initial secret afterwards as suggested by the Getting Started Guide: https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli)



## Créer un répertoire helm 

Settings/Repositories/connect repo

```
https://charts.bitnami.com/bitnami
```

