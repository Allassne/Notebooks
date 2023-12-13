Pour combiner les configurations de deux clusters Kubernetes dans un seul fichier `kubeconfig`, il est essentiel de s'assurer que chaque cluster, utilisateur et contexte a un nom unique. Je vais vous montrer comment fusionner les deux configurations que vous avez fournies en ajustant les noms pour éviter les conflits.

### Étape 1 : Renommer les Clusters, Utilisateurs et Contextes

Prenons les deux configurations et donnons des noms uniques à chaque cluster, utilisateur et contexte. Par exemple, nous pouvons nommer le premier cluster et ses entités associées comme `lazok` et le second comme `albam`.

### Configuration Fusionnée

```
apiVersion: v1
kind: Config
clusters:
- name: lazok
  cluster:
    certificate-authority-data: <votre-certificat-lazok>
    server: https://lazok-k3s-conpln-1.dev01.smile.lan:6443
- name: albam
  cluster:
    certificate-authority-data: <votre-certificat-albam>
    server: https://albam-Master1.dev01.smile.lan:6443

users:
- name: user-lazok
  user:
    client-certificate-data: <votre-client-certificat-lazok>
    client-key-data: <votre-client-cle-lazok>
- name: user-albam
  user:
    client-certificate-data: <votre-client-certificat-albam>
    client-key-data: <votre-client-cle-albam>

contexts:
- name: context-lazok
  context:
    cluster: lazok
    user: user-lazok
- name: context-albam
  context:
    cluster: albam
    user: user-albam

current-context: context-lazok
preferences: {}
```

Dans cette configuration :

- J'ai renommé le premier cluster en `lazok` et le second en `albam`.
- Les utilisateurs sont nommés `user-lazok` et `user-albam` respectivement.
- Les contextes sont désormais `context-lazok` et `context-albam`.
- `current-context` est défini sur `context-lazok`, mais vous pouvez le changer selon votre besoin.

### Notes Importantes :

- Remplacez `<votre-certificat-lazok>`, `<votre-certificat-albam>`, `<votre-client-certificat-lazok>`, `<votre-client-certificat-albam>`, `<votre-client-cle-lazok>`, et `<votre-client-cle-albam>` par les valeurs réelles de vos certificats et clés.
- Assurez-vous que les données de certificat et les clés sont correctement formatées en base64 si elles ne le sont pas déjà.
- Si les données de certificat ou les clés sont stockées dans des fichiers séparés plutôt qu'incluses directement dans le fichier `kubeconfig`, vous devrez référencer ces fichiers à la place.

Cette configuration vous permet de basculer entre les deux clusters en changeant le `current-context` avec la commande `kubectl config use-context`. Par exemple :

- Pour basculer sur le cluster `lazok` :
  ```
  kubectl config use-context context-lazok
  ```
- Pour basculer sur le cluster `albam` :
  ```
  kubectl config use-context context-albam
  ```

Gardez à l'esprit que le partage de données sensibles telles que des certificats ou des clés doit être géré avec prudence pour éviter tout risque de sécurité.



---


https://www.youtube.com/watch?v=El7zRS5w0dc

# Ajouter un cluster Kubernetes externe à la configuration d'Argo CD

### Configuration Fusionnée

Combiner les configurations de plusieurs clusters (2 dans notre cas)

Pour combiner les configurations de deux clusters Kubernetes dans le fichier ".kube/config"
Nous allons renommer les Clusters, Utilisateurs et Contextes.
Il est essentiel de s'assurer que chaque cluster, utilisateur et contexte a un nom unique.

Par exemple, nous allons nommer le premier cluster et ses entités associées comme `lazok` et le second comme `albam`.

```
apiVersion: v1
kind: Config
clusters:
- name: lazok
  cluster:
	certificate-authority-data: <votre-certificat-lazok>
	server: https://lazok-k3s-conpln-1.dev01.smile.lan:6443
- name: albam
  cluster:
	certificate-authority-data: <votre-certificat-albam>
	server: https://albam-Master1.dev01.smile.lan:6443

users:
- name: user-lazok
  user:
	client-certificate-data: <votre-client-certificat-lazok>
	client-key-data: <votre-client-cle-lazok>
- name: user-albam
  user:
	client-certificate-data: <votre-client-certificat-albam>
	client-key-data: <votre-client-cle-albam>

contexts:
- name: context-lazok
  context:
	cluster: lazok
	user: user-lazok
- name: context-albam
  context:
	cluster: albam
	user: user-albam

current-context: context-lazok
preferences: {}
```
Remplacez `<votre-certificat-lazok>`, `<votre-certificat-albam>`, `<votre-client-certificat-lazok>`, `<votre-client-certificat-albam>`, `<votre-client-cle-lazok>`, et `<votre-client-cle-albam>` par les valeurs réelles de vos certificats et clés.
- Si les données de certificat ou les clés sont stockées dans des fichiers séparés plutôt qu'incluses directement dans le fichier `.kube/config`, vous devrez référencer ces fichiers à la place.

Dans cette configuration :
- Nous avons nommé le premier cluster en `lazok` et le second en `albam`.
- Les utilisateurs sont nommés `user-lazok` et `user-albam` respectivement.
- Les contextes sont désormais `context-lazok` et `context-albam`.
- `current-context` est défini sur `context-lazok`, mais vous pouvez le changer selon votre besoin.


Cette configuration vous permet de basculer entre les deux clusters en changeant le `current-context` avec la commande `kubectl config use-context`, par exemple :

- Pour basculer sur le cluster `lazok` :
  ```
  kubectl config use-context context-lazok
  ```
- Pour basculer sur le cluster `albam` :
  ```
  kubectl config use-context context-albam
  ```



### Argo CD

Installation de l'outil en ligne de commande Argo CD

```
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/
```

Vérifiez la Version de l'Exécutable Argo CD :
```
argocd version
```


### Connexion au serveur Argo CD

Changer le type de service "ClusterIp" par "NodePort" pour que Argo CD soit joiniable par adresse IP 
Avant de vous connecter au serveur Argo CD en invite de commande il est préferable de basculer sur le cluster que vous aimerais ajouter à Argo CD

pour se connecter à un serveur Argo CD en invite de commande :


```
argocd login <IP-serveur>:<Port> --username admin
```
Exemple :
```
argocd login 192.168.0.128:30443 --username admin
```

sortie de la commande :
```
WARNING: server is not configured with TLS. Proceed (y/n)? yes
Password: 
'admin:login' logged in successfully
Context '192.168.0.128:30443' updated
```

lister les applications deployer sur Argo CD : 
```
argocd app list
```

lister les clusters sur Argo CD : 
```
argocd cluster list
```

```
argocd cluster add <context-name> --name <nom>
```

<context-name> : Le nom du contexte dans votre fichier .kube/config local.
<nom> : Nom utilisé pour identifier le cluster dans l'interface d'Argo CD.

Exemple :
```
argocd cluster add context-lazok --name cluster-lazok
```
sortie de la commande :
```
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `context-lazok` with full cluster level privileges. Do you want to continue [y/N]? y
INFO[0006] ServiceAccount "argocd-manager" already exists in namespace "kube-system" 
INFO[0006] ClusterRole "argocd-manager-role" updated    
INFO[0006] ClusterRoleBinding "argocd-manager-role-binding" updated 
Cluster 'https://192.168.0.186:6443' added
```






