## Pré Requis

Python 3
Systemd
Podman ou Docker pour exécuter des conteneurs
Synchronisation de l'heure (telle que chrony ou NTP)
LVM2 pour le provisionnement des périphériques de stockage


*Voir la section Compatibilité avec les versions de Podman pour un tableau des versions de Ceph compatibles avec Podman. Toutes les versions de Podman ne sont pas compatibles avec Ceph.*

## Installation

Utilisez curl pour récupérer la version la plus récente du script autonome.

```
cd /tmp
curl --silent --remote-name --location https://github.com/ceph/ceph/raw/quincy/src/cephadm/cephadm
```

Rendre le cephadm script exécutable :

```
chmod +x cephadm
./cephadm add-repo --release quincy
./cephadm install
```

La première étape de la création d'un nouveau cluster Ceph consiste à exécuter la commande sur le premier hôte du cluster Ceph. Le fait d'exécuter la commande sur le premier hôte du cluster Ceph crée le premier « démon de surveillance » du cluster Ceph, et ce démon de surveillance a besoin d'une adresse IP. Vous devez transmettre l'adresse IP du premier hôte du cluster Ceph à la commande, vous aurez donc besoin de connaître l'adresse IP de cet hôte.
Exécutez la commande :

**cephadm bootstrap  --mon-ip 10.1.6.53  --allow-fqdn-hostname**

```
cephadm bootstrap  --mon-ip 192.168.0.113  --allow-fqdn-hostname
```


Cette commande va :

Crée un démon de surveillance et de gestion pour le nouveau cluster sur l'hôte local.
Génère une nouvelle clé SSH pour le cluster Ceph et ajoutez-la au /root/.ssh/authorized_keysfichier de l'utilisateur racine.
Écris une copie de la clé publique dans /etc/ceph/ceph.pub.
Écris un fichier de configuration minimal dans /etc/ceph/ceph.conf. Ce fichier est nécessaire pour communiquer avec le nouveau cluster.
Écris une copie de la client.admin clé secrète administrative (privilégiée !) dans /etc/ceph/ceph.client.admin.keyring.
Ajoute l'étiquette _admin à l'hôte d'amorçage. Par défaut, tout hôte avec cette étiquette obtiendra (également) une copie de /etc/ceph/ceph.conf et /etc/ceph/ceph.client.admin.keyring.

A la fin du script, vous devriez avoir les acces au dashboard, tel que:
```
Ceph Dashboard is now available at:

	     URL: https://albam-C-1.dev01.smile.lan:8443/
	    User: admin
	Password: 6h9efh0a25

Enabling client.admin keyring and conf on hosts with "admin" label
Saving cluster configuration to /var/lib/ceph/31b000b8-903f-11ee-ad1a-525400392747/config directory
Enabling autotune for osd_memory_target
You can access the Ceph CLI as following in case of multi-cluster or non-default config:

	sudo /sbin/cephadm shell --fsid 31b000b8-903f-11ee-ad1a-525400392747 -c /etc/ceph/ceph.conf -k /etc/ceph/ceph.client.admin.keyring

Or, if you are only running a single cluster on this host:

	sudo /sbin/cephadm shell 

Please consider enabling telemetry to help improve Ceph:

	ceph telemetry on

For more information see:

	https://docs.ceph.com/docs/master/mgr/telemetry/

Bootstrap complete.
```

Installer ceph-cli pour utiliser directement les commandes de ceph dans le shell Bash:

```
cephadm add-repo --release quincy
cephadm install ceph-common
```

## Ajouter un hote

Rajouter la clé publique sur le serveur que vous voulez rajouter

```
ssh-copy-id -f -i /etc/ceph/ceph.pub root@192.168.0.114
ssh-copy-id -f -i /etc/ceph/ceph.pub root@192.168.0.115
```


Installer les pre requis plus haute sur le serveur
Lancer la commande suivante


**ceph orch host add HOSTNAME_OR_FQDN IP_ADDRESS --labels _admin**
```
ceph orch host add albam-C-2.dev01.smile.lan 192.168.0.114 --labels _admin
ceph orch host add albam-C-3.dev01.smile.lan 192.168.0.115 --labels _admin
 ```

## Initialisation des disques des OSD

Listons et Vérifions la compatibilité du ou des disques présent sur l'hôte

```
ceph orch device ls --refresh
ceph orch apply osd --all-available-devices --dry-run
```


Mettons en place le(s) disque(s) des OSD de façon automatique

NB: *Tous les disques non formatés seront utilisés et lorsque vous ajouterez de nouveaux disques, ils seront automatiquement utilisés.*

```
ceph orch apply osd --all-available-devices
```


Mettons en place le(s) disque(s) des OSD de façon manuelle

```
ceph orch device ls --wide --refresh
```

```
ceph orch daemon add osd HOSTNAME:_DEVICE_PATH_
```

```
ceph orch ls osd
```

