---
title: 'Let''s Encrypt'
taxonomy:
    category:
        - docs
---

[TOC]

## acme.sh

<https://github.com/acmesh-official/acme.sh>

<https://strugglers.net/~andy/blog/2018/03/19/lets-encrypt-wildcard-certificates-acme-sh-and-automated-dns-verification/>

### Installieren

```bash
wget -O -  https://get.acme.sh | sh
```

### Alias laden

```bash
. /root/.bashrc
```

### Verzeichnisse erstellen

```bash
mkdir /etc/acme.sh
mkdir /etc/acme.sh/certificates
mkdir /etc/acme.sh/keys
```

### Account registrieren

```bash
acme.sh --register-account -m admin@premmert.de
```

## DDNS-Key

### DDNS-Key erstellen

```bash
tsig-keygen -a hmac-sha512 acme | tee -a /etc/bind/named.conf.keys | tee /etc/acme.sh/keys/acme.key
```

<pre>
key "acme" {
	algorithm hmac-sha512;
	secret "*****";
};
</pre>

### DDNS-Key einbinden

```bash
nano /etc/bind/named.conf
```

<pre>
...
include "/etc/bind/named.conf.keys";
...
</pre>

## Subdomain

### Subdomain anlegen

```bash
nano /etc/bind/domains/_acme-challenge.premmert.de
```

<pre>
$ORIGIN _acme-challenge.premmert.de.
$TTL 10M

@	IN	SOA	ns1.premmert.de.	admin.premmert.de. (
	2022061701 ; serial
	8H         ; refresh
	2H         ; retry
	7D         ; expire
	3M         ; minimum
)

@			IN	NS	ns1.premmert.de.
</pre>

### Subdomain einbinden

```bash
nano /etc/bind/named.conf.local
```

<pre>
...
zone "_acme-challenge.premmert.de" IN {
	type master;
	file "/etc/bind/domains/_acme-challenge.premmert.de";
	notify no;
	allow-update {
		key "acme";
	};
};
...
</pre>

### Subdomain in Hauptzone eintragen

```bash
nano /etc/bind/domains/premmert.de
```

<pre>
...
_acme-challenge		IN	NS	ns1.premmert.de.
...
</pre>

### Dienst neu starten

```bash
systemctl restart named.service
```

## Update testen

### TXT-Record hinzufügen

```bash
nsupdate -k /etc/acme.sh/keys/acme.key

> server 127.0.0.1
> zone _acme-challenge.premmert.de.
> update add foo._acme-challenge.premmert.de. 86400 TXT "bar"
> send
> quit
```

### TXT-Record abrufen

```bash
dig foo._acme-challenge.premmert.de txt +short
```

<pre>
"bar"
</pre>

### TXT-Record entfernen

```bash
nsupdate -k /etc/acme.sh/keys/acme.key

> server 127.0.0.1
> zone _acme-challenge.premmert.de.
> update delete foo._acme-challenge.premmert.de. TXT
> send
> quit
```

## Wildcard Zertifikat

### Umgebungsvariablen

```bash
export NSUPDATE_SERVER="127.0.0.1"
export NSUPDATE_KEY="/etc/acme.sh/keys/acme.key"
```

### Zertifikat erstellen

```bash
acme.sh --issue -d premmert.de -d '*.premmert.de' --dns dns_nsupdate
```

### Zertifikat installieren

```bash
acme.sh --install-cert -d "premmert.de" \
--cert-file /etc/acme.sh/certificates/premmert.de.cert.pem \
--key-file /etc/acme.sh/certificates/premmert.de.key.pem \
--fullchain-file /etc/acme.sh/certificates/premmert.de.fullchain.pem \
--reloadcmd "/opt/Shell-Scripts/AcmeReload.sh"
```

<pre>
[Mon 14 Mar 2022 11:41:39 PM CET] Installing cert to: /etc/acme.sh/certificates/premmert.de.cert.pem
[Mon 14 Mar 2022 11:41:39 PM CET] Installing key to: /etc/acme.sh/certificates/premmert.de.key.pem
[Mon 14 Mar 2022 11:41:39 PM CET] Installing full chain to: /etc/acme.sh/certificates/premmert.de.fullchain.pem
[Mon 14 Mar 2022 11:41:39 PM CET] Run reload cmd: /opt/Shell-Scripts/AcmeReload.sh
[Mon 14 Mar 2022 11:41:39 PM CET] Reload success
</pre>

### Crontab überprüfen

```bash
crontab -l | grep acme
```

<pre>
49	0	*	*	*	"/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null
</pre>