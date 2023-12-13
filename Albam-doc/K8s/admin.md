# [](#administration)Administration

L'administration journaliere de kubernetes se fait soit en ligne de commande avec kubectl ou via l'interface web.

## [](#d%C3%A9ploiement-avec-kubernetes)Déploiement avec kubernetes

Le déploiement est lancé depuis le noeud master. Mais avant, il est préférable de vérifier le status des différents noeuds. Ceux-ci doivent être prêt (Ready)

```
kubectl get nodes
```

- On crée un déploiement en utilisant une image nginx

```
kubectl create deployment nginx --image=nginx
```

- On peut également voir le status du déploiement

```
kubectl get deployments
```

## [](#services)Services

On peut lister les services présent avec la commande suivante:

```
kubectl get service
```

### [](#exposer-le-service)Exposer le service

Le déploiement effectué, pour pouvoir accéder au service nginx depuis l'extérieur, il faut l'exposer

```
kubectl create service nodeport nginx --tcp=80:80
```

Pour voir les ports et les services exposés

```
kubectl get svc
```

Une fois le service exposé, il est possible d'y acceder à partir de l'IP de l'un des workers associé au port affecté

## [](#scaling)Scaling

Le scaling dans Kubernetes se réfère à la capacité de modifier dynamiquement le nombre d'instances de pods en cours d'execution pour s'adapter à la demande ou aux conditions de charge.

Le scaling peut s'effectuer sur un déploiement comme suit

```
kubectl scale deployement nginx --replicas=2
```

NB: --replicas : indique le nombre de réplique que l'on veut

Il est possible aussi de faire de **l'auto scaling** pour un déploiement comme suit:

```
kubectl autoscale deployment nginx --min=2 --max=10
```

NB: --min et --max indiquent le nombre minimal et maximal de replica qu'on doit avoir pour notre deploiement.

## [](#gestion-des-pods)Gestion des Pods

```
kubectl get pods                                    # show pods of default namespace
kubectl get pods -A                     # show pods from all namespace
kubectl logs my-pod                                 # dump pod logs (stdout)
kubectl logs -l name=myLabel                        # dump pod logs, with label name=myLabel (stdout)
kubectl logs my-pod --previous                      # dump pod logs (stdout) for a previous instantiation of a container
kubectl logs my-pod -c my-container                 # dump pod container logs (stdout, multi-container case)
kubectl logs -l name=myLabel -c my-container        # dump pod logs, with label name=myLabel (stdout)
kubectl logs my-pod -c my-container --previous      # dump pod container logs (stdout, multi-container case) for a previous instantiation of a container
kubectl logs -f my-pod                              # stream pod logs (stdout)
kubectl logs -f my-pod -c my-container              # stream pod container logs (stdout, multi-container case)
kubectl get pods   -o wide --all-namespaces         # Avoir la liste de tous les pods
kubectl logs -f pod [container]                     # Regarder les logs d'un pod
kubectl exec  -it pod [-c container_name] -- bash   # Se connecter dans un container
```

## [](#les-volumes)Les volumes

- Afficher les Persistent Volumes (PV)

```
kubectl get pv
```

- Afficher les Persistent Volumes Claim (PVC)

```
kubectl get pvc
```

## [](#helm)Helm

### [](#introduction-%C3%A0-helm)Introduction à helm

TODO

### [](#maitris%C3%A9-helm)Maitrisé helm

- Créer un package helm

Pour creer un package helm utiliser la commande

```
helm create my-chart
```

Faite vos modifications pour adapter votre chart

- Faite un tar

```
helm package .
```

- Stocker le package helm dans gitlab

Vous devez d'aborg generer un access token avec "api" comme permission

Avec curl

```
curl --request POST \
     --form 'chart=@mychart-0.1.0.tgz' \
     --user <username>:<access_token> \
     https://gitlab.example.com/api/v4/projects/<project_id>/packages/helm/api/<channel>/charts
```

- _username_ : Gitlab Username
- _access_token_ : Gitlab access tocken
- _project_id_ : Projet ID
- _channel_ : Le nom de la channel, tel que "stable"
- Ajouter le repo

```
helm repo add --username <username> --password <access_token> project-1 https://gitlab.example.com/api/v4/projects/<project_id>/packages/helm/<channel>
```

- Installer le chart

```
helm install my-release project-1/mychart
```

## [](#monitoring-du-cluster-k8s-avec-prometheus-centreon)Monitoring du cluster k8s avec prometheus + centreon

- Mise en place de prometheus

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack
```

- Mise en place du plugin k8s pour centreon

**NB:** A faire sur le poller !

```
yum install centreon-plugin-Cloud-Prometheus-Kubernetes-Api -y
yum install centreon-plugin-Cloud-Kubernetes-Api -y
```


## Supprimer les pods qui ne démarre pas correctement

```
kubectl get pods --field-selector=status.phase!=Running --all-namespaces -o json | jq -r '.items[] | select(.status.phase != "Running") | .metadata.name + " " + .metadata.namespace' | while read pod ns; do kubectl delete pod $pod -n $ns; done
```

## Taintez le nœud master :
Pour marquer votre nœud master de manière à ce qu'il n'accepte aucun pod de déploiement, suivez ces étapes:

Pour ajouter un taint au nœud master, exécutez la commande suivante sur votre machine où kubectl est configuré pour interagir avec votre cluster k3s :

```
kubectl taint nodes <nom-du-master> key=value:NoSchedule
```

Remplacez "**nom-du-master**" par le nom de votre nœud master. La clé et la valeur peuvent être ce que vous voulez, par exemple node-role.kubernetes.io/master="". NoSchedule signifie que aucun nouveau pod ne sera programmé sur le nœud, sauf s'ils ont une tolérance correspondante.

Exemple :

```
kubectl taint nodes albam-maasster-1.dev01.smile.lan key=allassaneBamba:NoSchedule
```

Vérifiez le taint:

Pour confirmer que le taint a été appliqué, exécutez :

```
kubectl get nodes -o json | jq '.items[].spec.taints'
```

Gérez les tolerations pour les pods spéciaux :

Si vous avez des pods qui doivent s'exécuter sur le nœud master, vous devrez ajouter une tolération à la spécification de ce pod. Voici un exemple de tolération dans un manifeste de pod :

```
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```

Cette tolération doit correspondre au taint que vous avez mis sur le master.

N'oubliez pas que lorsque vous taintez le nœud master, les pods qui y sont déjà en cours d'exécution ne seront pas affectés; seuls les nouveaux déploiements seront repoussés. Si vous voulez également évacuer les pods déjà en cours d'exécution, vous devrez le faire manuellement avec la commande kubectl drain, en prenant soin de ne pas interrompre les services importants.

pour retirer un taint

```
kubectl taint nodes albam-maasster-1.dev01.smile.lan key=allassaneBamba:NoSchedule-
```

