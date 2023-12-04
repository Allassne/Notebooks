



### Gestion des utilisateurs

Créer un utilisateur :

```
radosgw-admin user create --uid=nextcloud --display-name "Nextcloud service account" --email "admin@nextcloud.ci"
```

Lister les utilisateurs :

```
radosgw-admin user list
```

Afficher les infos d'un utilisateur :

```
radosgw-admin user info --uid=rgwadm
```

Supprimer un utilisateur et toutes ses données :

```
radosgw-admin user rm --uid=nextcloud --purge-data
```

### Gestion des buckets

#### Avec Radosgw-admin

Lister les buckets depuis la CLI radosgw :

```
radosgw-admin buckets list
```

Lister les objets présent dans un bucket:

```
radosgw-admin bucket list --bucket=mybucket
```

Supprimer un bucket et tous les objects qu'il contient :

```
radosgw-admin bucket rm --bucket=k3s --purge-objects
```

Supprimer un objet d'un bucket:

```
radosgw-admin object rm --bucket=<NOM_DU_BUCKET> --object=<NOM_DE_L'OBJET>
```













### [](#ceph-rbd)Ceph RBD

Installer le repo de la chart du connecteur ceph RBD

```
helm repo add ceph-csi https://ceph.github.io/csi-charts
```

Récupérer les valeurs par défaut

```
helm show  values ceph-csi/ceph-csi-rbd > ceph-csi-rbd.yml
```

Ci dessous les valeurs à changer

```
csiConfig:
   # Le retour de la commande *ceph mon dump*
   - clusterID: "XXXXXXXXXX-XXXXXXXXX"
     monitors:
       - "xxxx:6789"

provisioner:
  # Nombre de nodes
  replicaCount: 1

storageClass:
  create: true
  # ceph fsid
  clusterID: "XXXXXXXXXX-XXXXXXXXX"
  # Creer le pool avec le tag rbd au niveau
  pool: default
  controllerExpandSecret: csi-rbd-secret
  nodeStageSecret: csi-rbd-secret

secret:
  create: true
  userID: admin
  # le retour de la commande *ceph auth get-key client.admin*
  userKey: XXXXXXXXXXXx==
```

- Déployé la chart du connecteur Ceph RBD

```
helm upgrade -i rbd ceph-csi/ceph-csi-rbd -f ceph-csi-rbd.yml
```

Vérifier que vos ressources sont bien crées:

```
kubectl get secrets
kubectl get sc
kubectl get pods
```

- Tester avec la creation d'un PVC

```
cat > pvc.yaml << EOF
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rbd-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: csi-rbd-sc
EOF

kubectl apply -f pvc.yaml
```

### Ceph CEPHS

- Installer la chart du connecteur ceph CEPHS

```
helm repo add ceph-csi https://ceph.github.io/csi-charts
```

Récupérer les valeurs par defaut

```
helm show  values ceph-csi/ceph-csi-cephfs > ceph-csi-cephfs.yaml
```

Ci dessous les valeurs à ajouter dans le volet csiConfig decommenter

```
csiConfig:
  # Le retour de la commande *ceph mon dump*
  - clusterID: "xxxxxxxxxxxxx"
    monitors:
      - ip:6789
      - ip:6789
      - ip:6789

provisioner:
  # Nombre de nodes
  replicaCount: 1

storageClass:
  create: true
  # ceph fsid
  clusterID: XXXXXXXXXX-XXXXXXXXX # 
  # Vous devez creer le filesystem au niveau de ceph
  fsName: myfs
  controllerExpandSecret: csi-cephfs-secret
  nodeStageSecret: csi-cephfs-secret


secret:
  create: true
  adminID: admin
  # le retour de la commande ceph auth get-key client.admin
  adminKey: XXXXXXXXXXXx==
```

- Deployé le chart

Installation:

```
helm upgrade -n kube-system -i cephfs ceph-csi/ceph-csi-cephfs -f ceph-csi-cephfs.yaml
```

Vérifier que vos ressources sont bien crées:

```
kubectl get secrets
kubectl get sc
kubectl get pods
```

- Tester avec la creation d'un PVC

```
cat > pvc.yaml << EOF
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cephfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: csi-cephfs-sc
EOF

kubectl apply -f pvc.yaml
```