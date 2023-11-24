Un cluster HA K3s avec etcd intégré est composé de :

Trois nœuds de serveur ou plus qui serviront l'API Kubernetes et exécuteront d'autres services de plan de contrôle, ainsi qu'hébergeront le magasin de données etcd intégré.

Facultatif : zéro ou plusieurs nœuds d'agent désignés pour exécuter vos applications et services

## [](#installation-des-controller-nodes)Installation des controller nodes

Pour commencer, lancez d'abord un nœud de serveur avec l'itindicateur --cluster-init pour activer le clustering et un jeton qui sera utilisé comme secret partagé

pour joindre des serveurs supplémentaires au cluster.

```
export INSTALL_K3S_VERSION=v1.23.16+k3s1
curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server   --cluster-init


mettre l'option : -no-deploy=traefik   si vous vouler desactiver traefik
```

Après avoir lancé le premier serveur, joignez les deuxième et troisième serveurs au cluster à l'aide du secret partagé.

le token qui va nous permettre de joint les worker au control panel ce trouve dans le fichier token du master creer plus haut.

```
sudo cat /var/lib/rancher/k3s/server/node-token
```

NB: Desactiver le firewalld sur tous les nodes

```
systemctl disable firewalld.service --now
```

```
export INSTALL_K3S_VERSION=v1.23.16+k3s1
curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server  --no-deploy=traefik  --server https://<premier_server>:6443

export INSTALL_K3S_VERSION=v1.23.16+k3s1
curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server  --no-deploy=traefik  --server https://<premier_server>:6443

```

Vérifiez que les deuxième et troisième serveurs font maintenant partie du cluster :

```
$ kubectl get nodes
NAME        STATUS   ROLES                       AGE   VERSION
server1     Ready    control-plane,etcd,master   28m   v1.23.16+k3s1
server2     Ready    control-plane,etcd,master   13m   v1.23.16+k3s1
server3     Ready    control-plane,etcd,master   10m   v1.23.16+k3s1
```

Enfin, nous devrons mettre à jour le premier fichier d'unité de nœud pour supprimer l'indicateur --cluster-init . Nous pouvons le faire comme ceci :

```
sed -e '/server \\/,$d' -e 's@ExecStart=.*@ExecStart=/usr/local/bin/k3s server@' -i /etc/systemd/system/k3s.service
systemctl daemon-reload
```

Vous disposez maintenant d'un plan de contrôle hautement disponible. Tous les serveurs mis en cluster avec succès peuvent être utilisés dans l'argument --server pour

joindre des nœuds de serveur et d'agent supplémentaires. Joindre des nœuds d'agent supplémentaires au cluster suit la même procédure que les serveurs

## [](#exemple-dajout-de-worker)Exemple d'ajout de Worker

```

export INSTALL_K3S_VERSION=v1.23.16+k3s1

export K3S_TOKEN=xxxxxxxxxxx

curl -sfL https://get.k3s.io |  sh -s - agent --server https://<ip or hostname of server1>:6443
```

Un cluster HA K3s avec etcd intégré est composé de :

Trois nœuds de serveur ou plus qui serviront l'API Kubernetes et exécuteront d'autres services de plan de contrôle, ainsi qu'hébergeront le magasin de données etcd intégré.

Facultatif : zéro ou plusieurs nœuds d'agent désignés pour exécuter vos applications et services

## [](#installation-des-controller-nodes-1)Installation des controller nodes

Pour commencer, lancez d'abord un nœud de serveur avec l'itindicateur --cluster-init pour activer le clustering et un jeton qui sera utilisé comme secret partagé

pour joindre des serveurs supplémentaires au cluster.

```
export INSTALL_K3S_VERSION=v1.23.16+k3s1
curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server   --cluster-init


mettre l'option : -no-deploy=traefik   si vous vouler desactiver traefik
```

Après avoir lancé le premier serveur, joignez les deuxième et troisième serveurs au cluster à l'aide du secret partagé.

le token qui va nous permettre de joint les worker au control panel ce trouve dans le fichier token du master creer plus haut.

```
sudo cat /var/lib/rancher/k3s/server/node-token
```

NB: Desactiver le firewalld sur tous les nodes

```
systemctl disable firewalld.service --now
```

```
export INSTALL_K3S_VERSION=v1.23.16+k3s1
curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server  --no-deploy=traefik  --server https://<premier_server>:6443

export INSTALL_K3S_VERSION=v1.23.16+k3s1
curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server  --no-deploy=traefik  --server https://<premier_server>:6443

```

Vérifiez que les deuxième et troisième serveurs font maintenant partie du cluster :

```
$ kubectl get nodes
NAME        STATUS   ROLES                       AGE   VERSION
server1     Ready    control-plane,etcd,master   28m   v1.23.16+k3s1
server2     Ready    control-plane,etcd,master   13m   v1.23.16+k3s1
server3     Ready    control-plane,etcd,master   10m   v1.23.16+k3s1
```

Enfin, nous devrons mettre à jour le premier fichier d'unité de nœud pour supprimer l'indicateur --cluster-init . Nous pouvons le faire comme ceci :

```
sed -e '/server \\/,$d' -e 's@ExecStart=.*@ExecStart=/usr/local/bin/k3s server@' -i /etc/systemd/system/k3s.service
systemctl daemon-reload
```

Vous disposez maintenant d'un plan de contrôle hautement disponible. Tous les serveurs mis en cluster avec succès peuvent être utilisés dans l'argument --server pour

joindre des nœuds de serveur et d'agent supplémentaires. Joindre des nœuds d'agent supplémentaires au cluster suit la même procédure que les serveurs

## [](#exemple-dajout-de-worker-1)Exemple d'ajout de Worker

```
export INSTALL_K3S_VERSION=v1.23.16+k3s1

export K3S_TOKEN=xxxxxxxxxxx

curl -sfL https://get.k3s.io |  sh -s - agent --server https://<ip or hostname of server1>:6443
```