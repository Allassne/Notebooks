

**kubectl get pods
Error from server (ServiceUnavailable): apiserver not ready**


```
systemctl status firewalld.service
```

```
systemctl disable
```

```
systemctl disable --now  firewalld.service
```

```
systemctl status k3s.service
```

#### **Supprimer les pod qui ne demarre pas correctement**


```
kubectl get pods --field-selector=status.phase!=Running --all-namespaces -o json | jq -r '.items[] | select(.status.phase != "Running") | .metadata.name + " " + .metadata.namespace' | while read pod ns; do kubectl delete pod $pod -n $ns; done
```


#### Augmenter la taille du répertoire var

```
pvcreate /dev/vdc
vgextend system /dev/vdc
lvextend -l +100%FREE /dev/system/var
resize2fs /dev/system/var
```


#### ckeck

```
systemctl status k3s.service
```

```
systemctl status firewalld.service
```

```
systemctl start k3s.service
```

```
systemctl restart k3s.service
```

