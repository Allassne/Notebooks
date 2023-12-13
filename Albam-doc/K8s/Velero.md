
Installation de Velero

Installation de Velero Client ( Action à faire sur le kubernetes master )

Le client velero permet d'interagir avec les composants déployés sur le cluster kubernetes.

```
VERSION=1.9.0
wget https://github.com/vmware-tanzu/velero/releases/download/v${VERSION}/velero-v${VERSION}-linux-amd64.tar.gz
```

```
tar -xvf velero-v${VERSION}-linux-amd64.tar.gz
cp velero-v${VERSION}-linux-amd64/velero /usr/local/sbin/
```


Activer l'autocompletion velero (https://velero.io/docs/v1.9/customize-installation/#enabling-shell-autocompletion)


Vous devez maintenant vous assurer que le script de complétion de l'interface CLI de Velero est utilisé dans toutes vos sessions shell. Il y a deux façons de le faire :

```
yum install bash-completion -y
```

1 - Source the completion script in your ~/.bashrc file:

```
echo 'source <(velero completion bash)' >>~/.bashrc
```

2 - Ajoutez le script de complétion au répertoire /etc/bash_completion.d :

```
velero completion bash >/etc/bash_completion.d/velero
```
ou 
```
velero completion bash > /usr/local/etc/bash_completion.d/velero
```


Installation des composants de Velero Serveur

Nous utilisons un stockage S3 déployé par Ceph RGW compatible avec l'API S3 AWS. Les identifiants aws_access_key_id et aws_secret_access_key sont récupérés depuis le dashboard Ceph dans la section "Object Gateway > Users" puis éditez, l'utilisateur ayant accès au bucket à utiliser.
Créer le fichier credentials ( cas de Ceph RGW )

```
tee credentials-velero <<EOF
[default]
aws_access_key_id = SRN1NQW3RMJW4A2VOEZM
aws_secret_access_key = BRndHebHEMBAyF8AvNLc7tmXcxj8NuxgEkgSGfSS
EOF
```


Lancer l'installation de velero ( cas de Ceph RGW )

```
velero install \
	--provider aws \
	--plugins velero/velero-plugin-for-aws:v1.5.0 \
	--bucket velero \
	--use-restic \
	--secret-file ./credentials-velero \
	--use-volume-snapshots=false \
	--backup-location-config region=default,s3ForcePathStyle="true",s3Url=http://192.168.0.113:8080 \
	--wait
```

Vérifiez que le stockage est disponible:

```
velero backup-location get
```


```
kubectl logs deployment/velero -n velero
```

### Sauvegarde Planifier

affiche toutes les planifications de sauvegarde configurées :

```
velero schedule get
```

créer une planification de sauvegarde quotidienne nommée "sauve-01" dans Velero, qui s'exécute tous les jours à 9h00 :

```
velero schedule create sauve-01 --schedule "00 09 * * *"
```

supprimer la planification de sauvegarde :
```
velero schedule delete sauve-01
```





https://www.youtube.com/watch?v=uIKaiZQxqkI&ab_channel=DEVOPSD-DAY

### Backup

 Créer une backup :
 
```
velero backup create first-backup
```

Afficher les détails sur la sauvegarde :

```
velero backup describe first-backup
```

Afficher les journaux de la sauvegarde:

```
velero backup logs first-backup
```

voir les sauvegardes disponible

```
velero get backups  
```

### Restauration 

Restaurer à partir d'une sauvegarde spécifique :

```
velero restore create --from-backup NOM_DE_LA_SAUVEGARDE  
```

lister toutes les restaurations effectuées ou en cours dans le système:

```
velero restore get
```

voir la progression ou obtenir un rapport détaillé sur une opération de restauration spécifique :

```
velero restore describe NOM_DU_RESTORE    
```




### Désinstallation de Velero

```
kubectl delete namespace/velero clusterrolebinding/velero
kubectl delete crds -l component=velero
```


