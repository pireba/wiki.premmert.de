---
title: 'Tandoor Recipes'
---

[TOC]

## Installation

<https://docs.tandoor.dev/install/docker/#docker-compose>

### Verzeichnis erstellen

```bash
mkdir /opt/tandoor_recipes
```

### Verzeichnis wechseln

```bash
cd /opt/tandoor_recipes/
```

### docker-compose.yml herunterladen

```bash
wget https://raw.githubusercontent.com/vabene1111/recipes/develop/docs/install/docker/plain/docker-compose.yml
```

### .env herunterladen

```bash
wget https://raw.githubusercontent.com/vabene1111/recipes/develop/.env.template -O .env
```

### docker-compose.yml bearbeiten

```bash
nano docker-compose.yml
```

<pre>
...
services:
  ...
  web_recipes:
    ...
    restart: always
  ...
  nginx_recipes:
    ...
    ports:
      - "127.0.0.1:8081:80"
...
</pre>

### .env bearbeiten

```bash
nano .env
```

<pre>
...
SECRET_KEY=******
...
POSTGRES_PASSWORD=******
...
EMAIL_HOST=172.22.1.1
EMAIL_PORT=25
...
DEFAULT_FROM_EMAIL= tandoor@premmert.de
...
ENABLE_PDF_EXPORT=1
...
</pre>

### Container starten

```bash
docker-compose up -d
```

### Container stoppen

Bei Bedarf können die Container wie folgt gestoppt werden:

```bash
docker-compose down
```

## Apache2

<https://docs.tandoor.dev/install/docker/#apache>

### VHost anlegen

```bash
nano /etc/apache2/sites-available/tandoor.premmert.de.conf
```

<pre>
DEFINE server_name tandoor.premmert.de
DEFINE proxy_url   http://localhost:8081/

&lt;VirtualHost *:80&gt;
	ServerName ${server_name}

	Redirect permanent / https://${server_name}/
&lt;/VirtualHost&gt;

&lt;VirtualHost *:443&gt;
	ServerName ${server_name}

	ProxyPass        / ${proxy_url}
	ProxyPassReverse / ${proxy_url}
	ProxyPreserveHost On
	ProxyRequests Off

	RequestHeader set X-Forwarded-Proto "https"
	Header always set Access-Control-Allow-Origin "*"

	SSLEngine on
	SSLCertificateFile    /etc/acme.sh/certificates/premmert.de.fullchain.pem
	SSLCertificateKeyFile /etc/acme.sh/certificates/premmert.de.key.pem

	LogLevel info
	ErrorLog ${APACHE_LOG_DIR}/${server_name}_error.log
	CustomLog ${APACHE_LOG_DIR}/${server_name}_access.log combined
&lt;/VirtualHost&gt;
</pre>

### VHost aktivieren

```bash
a2ensite tandoor.premmert.de.conf
```

### Dienst neu laden

```bash
systemctl reload apache2.service
```