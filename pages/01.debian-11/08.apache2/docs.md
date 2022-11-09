---
title: Apache2
taxonomy:
    category:
        - docs
---

[TOC]

## Paket installieren

```bash
apt install apache2
```

## Einstellungen

### Verzeichnisse sperren

```bash
nano /etc/apache2/apache2.conf
```

<pre>
...
&lt;Directory /usr/share&gt;
	AllowOverride None
	<b>Require all denied</b>
&lt;/Directory&gt;

&lt;Directory /var/www/&gt;
	Options Indexes FollowSymLinks
	AllowOverride None
	<b>Require all denied</b>
&lt;/Directory&gt;
...
</pre>

### Fußzeile deaktivieren

<https://httpd.apache.org/docs/2.4/de/mod/core.html#serversignature>

```bash
nano /etc/apache2/apache2.conf
```

<pre>
...
ServerSignature Off
ServerTokens Prod
...
</pre>

### cgi-bin deaktivieren

```bash
a2disconf serve-cgi-bin
```

## SSL - Let's Encrypt

Um in den folgenden Schritten SSL-Zertifikate angeben zu können, müssen welche generiert werden.

Das geht am Besten über Let's Encrypt.

[Let's Encrypt](../lets-encrypt)

## Virtual Hosts

### Standard VHost bearbeiten

```bash
nano /etc/apache2/sites-available/000-default.conf
```

<pre>
DEFINE server_name localhost

&lt;VirtualHost *:80&gt;
	ServerName ${server_name}

	Redirect 403 /
	UseCanonicalName Off

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
&lt;/VirtualHost&gt;

&lt;VirtualHost *:443&gt;
	ServerName ${server_name}

	Redirect 403 /
	UseCanonicalName Off

	SSLEngine on
	SSLCertificateFile    /etc/acme.sh/certificates/premmert.de.fullchain.pem
	SSLCertificateKeyFile /etc/acme.sh/certificates/premmert.de.key.pem

	LogLevel info
	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
&lt;/VirtualHost&gt;
</pre>

### Standard SSL VHost löschen

```bash
rm /etc/apache2/sites-available/default-ssl.conf
```

## Dienst

### Dienst neu starten

```bash
systemctl restart apache2.service
```

### Autostart

```bash
systemctl enable apache2.service
```

## Fail2ban

[Apache2 Jails](../fail2ban#apache2-jails)