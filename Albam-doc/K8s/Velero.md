
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
aws_access_key_id = S2ZZJW616TKPUH6A22CW
aws_secret_access_key = IHdcKsuBn7eLv2KW65WLoSaPNh3FkNDazqA9kHxD
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
	--backup-location-config region=default,s3ForcePathStyle="true",s3Url=http://192.168.0.112:8080 \
	--wait
```

Vérifiez que le stockage est disponible:

```
velero backup-location get
```


```
velero schedule create sauve-01 --schedule "42 20 * * *"
```

```
velero schedule get
```

```
velero schedule delete sauve-01
```





https://www.youtube.com/watch?v=uIKaiZQxqkI&ab_channel=DEVOPSD-DAY

```
velero backup create first-backup
```

```
velero backup describe first-backup
velero backup logs first-backup
```

```
kubectl logs deployment/velero -n velero
```

```
velero get backups  # voir les sauvegardes disponible
```

```
velero restore create --from-backup NOM_DE_LA_SAUVEGARDE  # Restauration 
```

```
velero restore
```

```
velero restore describe NOM_DU_RESTORE    # voir la progression 
```

```
velero restore create --from-backup sauve-20231120084817
```