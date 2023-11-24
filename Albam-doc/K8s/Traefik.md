
Utiliser helm en tant que root

### Installer HELM 

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Nous allons utiliser helm pour installer traefik comme ingress controller

```
 helm repo add traefik https://helm.traefik.io/traefik
```

mise à jour du dépôt helm

```
helm repo update
```

## Intaller traefik avec helm

```
kubectl delete ingressclasses traefik 
```

```
helm install traefik traefik/traefik
```

Récuperer les valeurs par defaut

```
helm show values traefik/traefik > traefik.yml
```

Ensuite modifier les valeurs par defaut

```
vim traefik.yml
```

```
service:é
...
# Addresse IP des noeuds qui peuvent accepter des requetes de l'exterieur
  externalIPs:
	- X.X.X.X
globalArguments:
  # Désactive la verification ssl au niveau du backend
  - "--serversTransport.insecureSkipVerify=true"
  - "--entryPoints.web.proxyProtocol.insecure"
  - "--entryPoints.web.forwardedHeaders.insecure"

...
# Décommenteurs les clés Port
ports:
  traefik:
	Port: 9000
	expose: true
  web:
	Port: 80
  websecure:
	Port: 443
logs:
  # Traefik logs concern everything that happens to Traefik itself (startup, configuration, events, shutdown, and so on).
  general:
	level: DEBUG
  access:
	# To enable access logs
	enabled: true

```

Activer le dashboard de traefik

```
ingressRoute:
  dashboard:
	enabled: true
	# Additional ingressRoute annotations (e.g. for kubernetes.io/ingress.class)
	annotations: {}
	# Additional ingressRoute labels (e.g. for filtering IngressRoute by custom labels)
	labels: {}
	# The router match rule used for the dashboard ingressRoute
	matchRule:  Host(`albam-traefik.lan`) &&  PathPrefix(`/dashboard`) || PathPrefix(`/api`)
	# Specify the allowed entrypoints to use for the dashboard ingress route, (e.g. traefik, web, websecure).
	# By default, it's using traefik entrypoint, which is not exposed.
	# /!\ Do not expose your dashboard without any protection over the internet /!\
	entryPoints: ["web"]
	# Additional ingressRoute middlewares (e.g. for authentication)
	middlewares: []
	# TLS options (e.g. secret containing certificate)
	tls: {}
```

Ensuite installer

```
helm upgrade -n kube-system -i traefik traefik/traefik -f traefik.yml
```

si la ressource existe déjà 

```
kubectl delete ingressclasses traefik 
```


http://192.168.0.108:9000/dashboard/

http://albam-traefik.lan/dashboard/


## Configure des ingress pour pouvoir joindre tes pods

Testons L'Ingress

Attaquons notre application via un nom de domaine (myservicename.lan)
Pour tester le controller, creer un webservice de test

```
kubectl create deployment demo --image=httpd --port=80
```

```
kubectl expose deployment demo
```

## ou

```
kubectl create service clusterip demo --tcp=80:80
```

### Ensuite créer une resource ingress
```
kubectl create ingress demo-web --class=traefik\
  --rule=demo.lan/*=demo:80
```

Détails:

```
kubectl create ingress ingress-name --class=ingress_controller --rule=demo.lan/*=myservice-name:service-port
```

NB: --class=nginx si l'ingress controller c'est nginx et --class=traefik si l'ingress controller c'est traefik


## Testons

```
curl -H "Host:demo.lan" http://192.168.0.108/
```

## Attaquons notre application via un endpoint (mydomain.lan/myservicename)

### Créons un deploy:

```
kubectl create deploy webapp --image nginx
kubectl create service clusterip webapp --tcp=8080:80
```

Créons un fichier manifest comme suit:

```
vim myingress.yml
```

```
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: monitor
  namespace: default
spec:
  stripPrefix:
	forceSlash: false
	prefixes:
	  - /webapp
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
	ingress.kubernetes.io/rewrite-target: /
	kubernetes.io/ingress.class: traefik
	traefik.ingress.kubernetes.io/router.middlewares: default-monitor@kubernetescrd
  generation: 3
  name: webapp
  namespace: default
spec:
  rules:
	- host: myapp.lan
	  http:
		paths:
		  - path: /webapp
			pathType: Prefix
			backend:
			  service:
				name: webapp
				port:
				  number: 8080
status:
  loadBalancer: {}
```

```
kubectl apply -f myingress.yml
```

```
curl -H "Host:myapp.lan" http://10.1.4.153/webapp
```

http://myapp.lan/webapp