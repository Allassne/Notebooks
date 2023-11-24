

openssl genrsa -out ca.key 2048
openssl req -new -x509 -days 365 -key ca.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=Acme Root CA" -out ca.crt

openssl req -newkey rsa:2048 -nodes -keyout server.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=*.synelia.lan" -out server.csr
openssl x509 -req -extfile <(printf "subjectAltName=DNS:synelia.lan,DNS:www.gitlab.synelia.lan") -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt



## Mettre en place le HTTPS

  

### Générer un certificat

  

Pour mettre en place le HTTPS vous devez générer le certificat dans le répertoire **/etc/gitlab/ssl/**

  

```

openssl genrsa -out ca.key 2048

openssl req -new -x509 -days 365 -key ca.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=Acme Root CA" -out ca.crt

  

openssl req -newkey rsa:2048 -nodes -keyout server.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=*.synelia.lan" -out server.csr

openssl x509 -req -extfile <(printf "subjectAltName=DNS:synelia.lan,DNS:www.gitlab.synelia.lan") -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt

```

  

Ces commandes créent une autorité de certification auto-signée et utilisent cette **CA** pour signer un certificat pour le domaine "*.synelia.lan" avec des SANs spécifiques.

  

### Modifiez le fichier de configuration

  

Pour Modifier l'URL externe, vous devez accéder au fichier **/etc/gitlab/gitlab.rb** et remplacez **external_url**.

Dans mon cas : external_url "http://gitlab.synelia.lan" devient :

```

external_url "https://gitlab.synelia.lan"

```

Activez HTTPS en définissant :

```

nginx['enable'] = true

nginx['redirect_http_to_https'] = true

nginx['ssl_protocols'] = "TLSv1.2 TLSv1.3"

nginx['ssl_ciphers'] = "ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384"

```

Configurez les paramètres SSL en spécifiant les chemins vers votre certificat et votre clé privée

```

nginx['ssl_certificate'] = "/etc/gitlab/ssl/server.crt"

nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/server.key"

```

  

Enregistrez et fermez le fichier, puis exécutez la commande suivante pour reconfigurer GitLab.

```

gitlab-ctl reconfigure

```

Ensuite redemarer gitlab

```

sudo gitlab-ctl restart

```