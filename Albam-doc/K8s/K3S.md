### Mise à jour

```
sudo dnf update -y
```

## Installer docker (Master & worker)

```
curl https://releases.rancher.com/install-docker/20.10.sh | sh
```

```
sudo systemctl enable docker 
sudo systemctl start docker
sudo systemctl status docker
```

```
sudo usermod -aG docker $user
sudo usermod -aG docker albam
```


## Installation du controller node avec docker (Master)
Pour installer le node master nous allons executer ce script

```
export INSTALL_K3S_VERSION=v1.23.16+k3s1
curl -sfL https://get.k3s.io | sh -s - --docker
```

Le script nous permet d'installer les utilitaire suivant:

kubectl
crictl
k3s-killall.sh
k3s-uninstall.sh

Cette commande va télécharger et sauvegarder le binaire k3s dans le chemin /usr/local/bin. Assurez-vous alors que le chemin /usr/local/bin a été déclaré comme un chemin binaire.

```
echo "export PATH=\$PATH:/usr/local/bin" | sudo tee -a /etc/profile
source /etc/profile
```

Rajouter la config .kube/config

```
mkdir -p /root/.kube
ln -s /etc/rancher/k3s/k3s.yaml /root/.kube/config
```

Autorisez les ports ci-dessous à travers le pare-feu sur le nœud de contrôle :

```
sudo firewall-cmd --zone=trusted --add-interface=cni0 --permanent
sudo firewall-cmd --add-port=8090/tcp --permanent
sudo firewall-cmd --add-port=10250/tcp --permanent
sudo firewall-cmd --add-port=10255/tcp --permanent
sudo firewall-cmd --add-port=8472/udp --permanent
sudo firewall-cmd --add-port=6443/tcp --permanent
sudo firewall-cmd --add-port=9443/tcp --permanent
sudo firewall-cmd --reload
```

Les service k3s seront gerer avec ces commande ci-dessous

```
sudo systemctl start k3s
sudo systemctl enable k3s
sudo systemctl status k3s
```



#### Configuration
la configuration de k3s se fait dans le fichier /etc/rancher/k3s/config.yaml
Par example pour desactiver traefik

```
disable:
  - traefik
#  - helm-controller
#  - cloud-controller
```

## Installation des workers nodes avec docker
le token qui va nous permettre de joint les worker au control panel ce trouve dans le fichier token du master creer plus haut.

```
sudo cat /var/lib/rancher/k3s/server/node-token
```

Ensuite nous allons executer le script suivant pour permettre aux nodes worker de rejoindre le master et creer le cluster.

K3S_URL and K3S_TOKEN son des variable d'environment a remplacer dans la commande.

```
export  K3S_URL=https://192.168.0.108:6443 
export INSTALL_K3S_VERSION=v1.23.16+k3s1
export K3S_TOKEN=K10e7f995ed9c0741dfcc82c832fba0bfc91092a5a2e49db176209f7f0f68d28109::server:a92fbf551ab732169bc871617ff56e15
curl -sfL https://get.k3s.io | sh -s - --docker
```

Autorisez les ports suivants à travers le pare-feu sur le nœud de travail :

```
sudo firewall-cmd --zone=trusted --add-interface=cni0 --permanent
sudo firewall-cmd --add-port=8090/tcp --permanent
sudo firewall-cmd --add-port=10250/tcp --permanent
sudo firewall-cmd --add-port=10255/tcp --permanent
sudo firewall-cmd --add-port=8472/udp --permanent
sudo firewall-cmd --reload
```

## auto-completion

```
sudo dnf install bash-completion
echo "source <(kubectl completion bash)" >> ~/.bashrc
source .bashrc
```

### Autorisation k3s refusée lors de l'utilisation de kubectl

https://0to1.nl/post/k3s-kubectl-permission/

sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config && chown $USER ~/.kube/config && chmod 600 ~/.kube/config && export KUBECONFIG=~/.kube/config

sudo chmod 644 /etc/rancher/k3s/k3s.yaml
