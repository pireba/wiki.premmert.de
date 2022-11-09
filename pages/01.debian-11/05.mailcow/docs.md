---
title: Mailcow
taxonomy:
    category:
        - docs
media_order: 'Screenshot 2022-06-17 at 15-24-19 mailcow UI.png,Screenshot 2022-06-17 at 15-52-37 mailcow UI.png,Screenshot 2022-06-17 at 15-57-58 mailcow UI.png,Screenshot 2022-06-17 at 16-01-19 mailcow UI.png,Screenshot 2022-06-17 at 16-05-06 mailcow UI.png'
---

[TOC]

## Docker

### Docker installieren

```bash
curl -sSL https://get.docker.com/ | CHANNEL=stable sh
```

### Dienst starten

```bash
systemctl enable --now docker
```

## Docker-Compose

### Docker-Compose installieren

```bash
curl -L https://github.com/docker/compose/releases/download/$(curl -Ls https://www.servercow.de/docker-compose/latest.php)/docker-compose-$(uname -s)-$(uname -m) > /usr/local/bin/docker-compose
```

### Docker-Compose Berechtigungen

```bash
chmod +x /usr/local/bin/docker-compose
```

## Mailcow

<https://mailcow.github.io/mailcow-dockerized-docs/i_u_m/i_u_m_install/>

### Umask prüfen

```bash
umask
```

<pre>
0022
</pre>

### Verzeichnis wechseln

```bash
cd /opt/
```

### Von Github klonen

```bash
git clone https://github.com/mailcow/mailcow-dockerized
```

### Verzeichnis wechseln

```bash
cd mailcow-dockerized
```

### Konfiguration erstellen

```bash
./generate_config.sh
```

<pre>
Press enter to confirm the detected value '[value]' where applicable or enter a custom value.
Mail server hostname (FQDN) - this is not your mail domain, but your mail servers hostname: mail.premmert.de
Timezone [Europe/Berlin]: 
Generating snake-oil certificate...
Generating a RSA private key
...................................++++
............................................................++++
writing new private key to 'data/assets/ssl-example/key.pem'
-----
Copying snake-oil certificate...
</pre>

### Konfigurationsdatei sichern

```bash
cp mailcow.conf mailcow.conf_backup
```

### Konfigurationsdatei anpassen

```bash
nano mailcow.conf
```

<pre>
...
HTTP_PORT=8080
HTTP_BIND=127.0.0.1

HTTPS_PORT=8443
HTTPS_BIND=127.0.0.1
...
SKIP_LETS_ENCRYPT=y
...
SKIP_SOLR=y
...
</pre>

### Container herunterladen

```bash
docker-compose pull
```

### Container starten

```bash
docker-compose up -d
```

### Container stoppen

Bei Bedarf können die Container wie folgt gestoppt werden:

```bash
docker-compose down
```

## Apache

### Module aktivieren

```bash
a2enmod rewrite proxy proxy_http headers ssl
```

### VHost anlegen

```bash
nano /etc/apache2/sites-available/mail.premmert.de.conf
```

<pre>
DEFINE server_name mail.premmert.de
DEFINE proxy_url   http://127.0.0.1:8080/

<VirtualHost *:80>
	ServerName  ${server_name}

	Redirect permanent / https://${server_name}/
</VirtualHost>

<VirtualHost *:443>
	ServerName  ${server_name}

	ServerAlias autodiscover.*
	ServerAlias autoconfig.*

	ProxyPass        / ${proxy_url}
	ProxyPassReverse / ${proxy_url}
	ProxyPreserveHost On
	ProxyAddHeaders On
	RequestHeader set X-Forwarded-Proto "https"

	SSLEngine on
	SSLCertificateFile    /etc/acme.sh/certificates/premmert.de.fullchain.pem
	SSLCertificateKeyFile /etc/acme.sh/certificates/premmert.de.key.pem

	LogLevel info
	ErrorLog ${APACHE_LOG_DIR}/${server_name}_error.log
	CustomLog ${APACHE_LOG_DIR}/${server_name}_access.log combined
</VirtualHost>
</pre>

### VHost aktivieren

```bash
a2ensite mail.premmert.de.conf
```

### Dienst neu laden

```bash
systemctl reload apache2
```

## Shorewall

<https://shorewall.org/Docker.html>

<https://gist.github.com/lukasnellen/20761a20286f32efc396e207d986295d>

### Interfaces

```bash
nano /etc/shorewall/interfaces
nano /etc/shorewall6/interfaces
```

<pre>
#ZONE		INTERFACE		OPTIONS
...
docker		docker+			routeback=1
docker		br-+			routeback=1
...
</pre>

### Policy

```bash
nano /etc/shorewall/policy
nano /etc/shorewall6/policy
```

<pre>
#SOURCE		DEST		POLICY	LOGLEVEL	RATE	CONNLIMIT
...
docker		all		ACCEPT
fw		docker		ACCEPT
...
</pre>

### Rules

```bash
nano /etc/shorewall/rules
nano /etc/shorewall6/rules
```

<pre>
#ACTION		SOURCE		DEST		PROTO	DPORT	SPORT	ORIGDEST	RATE	USER	MARK	CONNLIMIT	TIME	HEADERS	SWITCH	HELPER
...
IMAP(ACCEPT)	wan		docker
IMAPS(ACCEPT)	wan		docker
MSA(ACCEPT)	wan		docker
POP3(ACCEPT)	wan		docker
POP3S(ACCEPT)	wan		docker
Sieve(ACCEPT)	wan		docker
SMTP(ACCEPT)	wan		docker
SMTPS(ACCEPT)	wan		docker
...
</pre>

### Docker Unterstützung

```bash
nano /etc/shorewall/shorewall.conf
```

<pre>
...
#DOCKER=No
DOCKER=Yes
...
</pre>

### Zones

IPv4:

```bash
nano /etc/shorewall/zones
```

<pre>
#ZONE		TYPE		OPTIONS		IN_OPTIONS	OUT_OPTIONS
...
docker		ipv4
...
</pre>

IPv6:

```bash
nano /etc/shorewall6/zones
```

<pre>
#ZONE		TYPE		OPTIONS		IN_OPTIONS	OUT_OPTIONS
...
docker		ipv6
...
</pre>

## Lokaler Mailversand

<https://mailcow.github.io/mailcow-dockerized-docs/post_installation/firststeps-local_mta/>

### Postfix

#### Paket installieren

```bash
apt install postfix
```

<pre>
General type of mail configuration: No configuration
</pre>

#### Konfigurationsdatei kopieren

```bash
cp /usr/share/postfix/main.cf.debian /etc/postfix/main.cf
```

#### Listener deaktivieren

```bash
nano /etc/postfix/master.cf
```

<pre>
...
#smtp      inet  n       -       y       -       -       smtpd
...
</pre>

#### Relay einrichten

```bash
nano /etc/postfix/main.cf
```

<pre>
...
relayhost = 172.22.1.1
mynetworks = 127.0.0.0/8
inet_interfaces = loopback-only
relay_transport = relay
default_transport = smtp
myhostname = local.premmert.de
</pre>

#### Dienst neu starten

```bash
systemctl restart postfix.service
```

### S-Nail

```bash
apt install s-nail
```

### Mailx

```bash
apt install bsd-mailx
```

## Server-Konfiguration

Konfiguration -> Server-Konfiguration

### Admin Kennwort

Reiter "Zugang" -> Benutzer "admin" auswählen -> Bearbeiten

![Administrator bearbeiten](Screenshot%202022-06-17%20at%2015-24-19%20mailcow%20UI.png "Administrator bearbeiten")

### Fail2Ban

Reiter "Konfiguration -> Fail2ban-Parameter"

![Fail2ban-Parameter](Screenshot%202022-06-17%20at%2015-52-37%20mailcow%20UI.png "Fail2ban-Parameter")

## E-Mail-Setup

Konfiguration -> E-Mail-Setup

### Domain

Reiter "Domains" -> + Domain hinzufügen

![Domain hinzufügen](Screenshot%202022-06-17%20at%2015-57-58%20mailcow%20UI.png "Domain hinzufügen")

### Mailbox

Reiter "Mailboxen" -> + Mailbox hinzufügen

![Mailbox hinzufügen](Screenshot%202022-06-17%20at%2016-01-19%20mailcow%20UI.png "Mailbox hinzufügen")

### Aliasse

Reiter "Aliasse" -> + Alias hinzufügen

![Alias hinzufügen](Screenshot%202022-06-17%20at%2016-05-06%20mailcow%20UI.png "Alias hinzufügen")

## DNS Einstellungen

<https://mailcow.github.io/mailcow-dockerized-docs/prerequisite/prerequisite-dns/>

Alle folgenden Konfigurationen müssen in der entsprechenden Zonendatei vorgenommen werden.

```bash
nano /etc/bind/domains/premmert.de
```

### Grundeinstellungen

<pre>
mail			IN	A	*IPv4 Adresse*
mail			IN	AAAA	*IPv6 Adresse*

@			IN	MX	10 mail.premmert.de.

autoconfig		IN	CNAME	mail.premmert.de.
autodiscover		IN	CNAME	mail.premmert.de.
</pre>

### DKIM-Record

Der DKIM-Key kann auf folgender Seite in der Mailcow Admin Oberfläche eingesehen werden:

Konfiguration -> Server-Konfiguration -> Reiter "ARC/DKIM-Keys"

Der Key ist für Bind zu lang, wesshalb er in 4 Parts aufgeteilt werden muss. Sie werden in der Zonendatei mit runden Klammern zusammengefasst.

<pre>
dkim._domainkey		IN	TXT	(
						"v=DKIM1;k=rsa;t=s;s=email;p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAzUgZ+Z72WbuowEI9KvgLnOXNq52BxiVQ69XK8i0c4TGNzYPUZDlJxJh"
						"+a6XC6TOYxJ+WlL1pSipRCQkQBFUhTddS1gYKpNziPbbUfOwoJFwP+9FBVYE0wybNqrqjc/kBChboDpjehTiSQGupMUxEweAcvUDGUkId/qwaW0QDAJEFWVIACo2wmX"
						"Fxj8QXyKH5z6U1q4X8xvVoADUK8ivDx/lCIrGV2XvWVRW0D2Scx2gEFnirNa88pRKRkTDyb/o+bRWrpeNQ5/wPLjB7/Myy9kfErV68q8/zYOVQSA6O7mg5rBj6yOpWr"
						"Y9NfnG1sFOSaW/rVldXGFkexVtCaAjikwIDAQAB"
					)
</pre>

### DMARC-Record

<pre>
_dmarc			IN	TXT	"v=DMARC1; p=quarantine; rua=mailto:admin@premmert.de; ruf=mailto:admin@premmert.de; fo=1; pct=10"
</pre>

### SPF-Record

<pre>
@			IN	TXT	"v=spf1 mx -all"
</pre>

### SRV-Record

<pre>
_autodiscover._tcp	IN	SRV	0	1	443	mail.premmert.de.
_caldavs._tcp		IN	SRV	0	1	443	mail.premmert.de.
_carddavs._tcp		IN	SRV	0	1	443	mail.premmert.de.
_imap._tcp		IN	SRV	0	1	143	mail.premmert.de.
_imaps._tcp		IN	SRV	0	1	993	mail.premmert.de.
_pop3._tcp		IN	SRV	0	1	110	mail.premmert.de.
_pop3s._tcp		IN	SRV	0	1	995	mail.premmert.de.
_sieve._tcp		IN	SRV	0	1	4190	mail.premmert.de.
_smtps._tcp		IN	SRV	0	1	465	mail.premmert.de.
_submission._tcp	IN	SRV	0	1	587	mail.premmert.de.
_caldavs._tcp		IN	TXT	"path=/SOGo/dav/"
_carddavs._tcp		IN	TXT	"path=/SOGo/dav/"
</pre>

## SSL-Zertifikat

<https://mailcow.github.io/mailcow-dockerized-docs/post_installation/firststeps-ssl/#how-to-use-your-own-certificate>

### Mailcows Let's Encrypt deaktivieren

<https://mailcow.github.io/mailcow-dockerized-docs/post_installation/firststeps-ssl/#disable-lets-encrypt>

#### Verzeichnis wechseln

```bash
cd /opt/mailcow-dockerized/
```

#### Konfigurationsdatei anpassen

```bash
nano mailcow.conf
```

<pre>
...
SKIP_LETS_ENCRYPT=y
...
</pre>

#### Container stoppen

```bash
docker-compose down
```

#### Container starten

```bash
docker-compose up -d
```

### Eigenes Zertifikat einbinden

#### Zertifikate kopieren

```bash
cp /etc/acme.sh/certificates/premmert.de.fullchain.pem /opt/mailcow-dockerized/data/assets/ssl/cert.pem
cp /etc/acme.sh/certificates/premmert.de.key.pem /opt/mailcow-dockerized/data/assets/ssl/key.pem
```

#### Container neu starten

```bash
docker restart $(docker ps -qaf name=postfix-mailcow)
docker restart $(docker ps -qaf name=nginx-mailcow)
docker restart $(docker ps -qaf name=dovecot-mailcow)
```