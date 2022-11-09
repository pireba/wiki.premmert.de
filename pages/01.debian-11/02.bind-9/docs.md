---
title: 'Bind 9'
media_order: 'Screenshot 2022-06-17 at 17-09-45 - Free free set them free - 1984 - Agnes hefur gætur á þér!.png,Screenshot 2022-06-17 at 17-16-28 - FreeDNS - 1984 - Agnes hefur gætur á þér!.png'
taxonomy:
    category:
        - docs
---

[TOC]

## Paket installieren

```bash
apt install bind9
```

## Konfiguration

### Default Zones deaktivieren

```bash
nano /etc/bind/named.conf
```

<pre>
...
//include "/etc/bind/named.conf.default-zones";
...
</pre>

### Optionen

```bash
nano /etc/bind/named.conf.options
```

<pre>
options {
	directory "/var/cache/bind";

	dnssec-validation auto;

	listen-on {
		127.0.0.1;
		*IPv4 Adresse*;
	};

	listen-on-v6 {
		::1;
		*IPv6 Adresse*;
	};

	allow-recursion {
		127.0.0.1;
	};

	allow-transfer {
		none;
	};
};
</pre>

## Zonen

### Verzeichnis

#### Anlegen

```bash
mkdir /etc/bind/domains
```

#### Berechtigungen setzen

```bash
chmod g+w /etc/bind/domains/
```

### Anlegen

```bash
nano /etc/bind/domains/premmert.de
```

<pre>
TODO
</pre>

### Einbinden

```bash
nano /etc/bind/named.conf.local
```

<pre>
...
zone "premmert.de" IN {
        type master;
        file "/etc/bind/domains/premmert.de";
        notify yes;
        allow-transfer {
                93.95.224.6;
        };
};
...
</pre>

## Logging

<https://stackoverflow.com/a/12114139/11975109>

### Konfiguration erstellen

```bash
nano /etc/bind/named.conf.logging
```

<pre>
logging {
	channel default_file {
		file "/var/log/bind/default.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel general_file {
		file "/var/log/bind/general.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel database_file {
		file "/var/log/bind/database.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel security_file {
		file "/var/log/bind/security.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel config_file {
		file "/var/log/bind/config.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel resolver_file {
		file "/var/log/bind/resolver.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel xfer-in_file {
		file "/var/log/bind/xfer-in.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel xfer-out_file {
		file "/var/log/bind/xfer-out.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel notify_file {
		file "/var/log/bind/notify.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel client_file {
		file "/var/log/bind/client.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel unmatched_file {
		file "/var/log/bind/unmatched.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel queries_file {
		file "/var/log/bind/queries.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel network_file {
		file "/var/log/bind/network.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel update_file {
		file "/var/log/bind/update.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel dispatch_file {
		file "/var/log/bind/dispatch.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel dnssec_file {
		file "/var/log/bind/dnssec.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	channel lame-servers_file {
		file "/var/log/bind/lame-servers.log" versions 3 size 5m;
		severity dynamic;
		print-time yes;
	};

	category default { default_file; };
	category general { general_file; };
	category database { database_file; };
	category security { security_file; };
	category config { config_file; };
	category resolver { resolver_file; };
	category xfer-in { xfer-in_file; };
	category xfer-out { xfer-out_file; };
	category notify { notify_file; };
	category client { client_file; };
	category unmatched { unmatched_file; };
	category queries { queries_file; };
	category network { network_file; };
	category update { update_file; };
	category dispatch { dispatch_file; };
	category dnssec { dnssec_file; };
	category lame-servers { lame-servers_file; };
};
</pre>

### Konfiguration einbinden

```bash
nano /etc/bind/named.conf
```

<pre>
...
include "/etc/bind/named.conf.logging";
...
</pre>

### Verzeichnis

#### Anlegen

```bash
mkdir /var/log/bind
```

#### Eigentümer

```bash
chown bind:bind /var/log/bind
```

## Dienst

### Dienst starten

```bash
systemctl restart bind9.service
```

### Autostart

```bash
systemctl enable bind9.service
```

## Tests

### Auflösung muss funktionieren

```bash
dig @ns1.premmert.de www.premmert.de
```

### Auflösung darf nicht funktionieren

```bash
dig @ns1.premmert.de www.google.de
```

## Sekundärer DNS-Server

<https://1984.hosting/>

### Sekundäre Zone anlegen

Auf der Adminseite von 1984 kann unter "FreeDNS -> Add slave zone" eine neue sekundäre Zone angelegt werden:

<https://1984.hosting/domains/newslave/>

![Add slave zone](Screenshot%202022-06-17%20at%2017-09-45%20-%20Free%20free%20set%20them%20free%20-%201984%20-%20Agnes%20hefur%20g%C3%A6tur%20%C3%A1%20%C3%BE%C3%A9r!.png "Add slave zone")

### Sekundäre DNS-Server in der Zone eintragen

```bash
/etc/bind/domains/premmert.de
```

<pre>
...
@			IN	NS	ns0.1984.is.
@			IN	NS	ns1.1984.is.
@			IN	NS	ns2.1984.is.
...
</pre>

### Zonentransfer einrichten

```bash
nano /etc/bind/named.conf.local
```

<pre>
...
zone "premmert.de" IN {
	type master;
	file "/etc/bind/domains/premmert.de";
	notify yes;
	allow-transfer {
		93.95.224.6;
	};
};
...
</pre>

### Dienst neu starten

```bash
systemctl restart named.service
```

### Kontrolle

#### Status bei 1984

Auf der Adminseite von 1984 muss unter "FreeDNS -> Manage zones -> premmert.de -> Status" die aktuelle Serial der Zone zu sehen sein:

![Status](Screenshot%202022-06-17%20at%2017-16-28%20-%20FreeDNS%20-%201984%20-%20Agnes%20hefur%20g%C3%A6tur%20%C3%A1%20%C3%BE%C3%A9r!.png "Status")

#### Kommandozeile

<pre><code>
dig @ns0.1984.is premmert.de soa +short
dig @ns1.1984.is premmert.de soa +short
dig @ns2.1984.is premmert.de soa +short
</code></pre>

<pre>
ns1.premmert.de. admin.premmert.de. <b>2022061707</b> 28800 7200 604800 180
</pre>

## Update Skript

Dieses Skript hat die folgenden Aufgaben:

1. Es prüft die Syntax in jeder Zonendatei und bricht bei Fehlern die weitere Verarbeitung ab.
2. Es synchronisiert die offenen Änderungen aus einer Journaldatei in die Zonendatei.
3. Es setzt die Seriennummer in allen Zonendateien auf den nächst größeren Tageswert (YYYYMMDD**XX**) und gibt die Änderung aus.
4. Es starten den Dienst neu.
5. Es lässt bis zum Beenden des Skriptes ein "tail -f" auf die Log-Datei laufen. Das Skript wird erst nach 10 Sekunden oder durch Drücken von "Ctrl + C" beendet.

<pre>
#!/bin/bash

while read file
do
	named-checkzone "$( basename "$file" )" "$file"
	if [ $? -ne 0 ]
	then
		exit 1
	fi
done &gt; &gt;( ls -1 /etc/bind/domains/*.de )

rndc sync -clean

oldID=$( grep -Eo "[0-9]{10}.*[Ss]erial" /etc/bind/domains/premmert.de | cut -d";" -f1 | xargs )
oldDate="${oldID: 0: 8}"
oldRunningNumber="${oldID: 8: 9}"

newDate=$( date +%Y%m%d )
if [ "$newDate" == "$oldDate" ]
then
	newRunningNumber=$( expr $oldRunningNumber + 1 )
else
	newRunningNumber="01"
fi
if [ ${#newRunningNumber} -eq 1 ]
then
	newRunningNumber=0${newRunningNumber}
fi
newID=${newDate}${newRunningNumber}

echo
echo "--------------------------------------------------"
echo
echo "Alte ID: ${oldID}"
echo "Neue ID: ${newID}"
echo

if sed -i -r 's/[0-9]{10}(.*[Ss]erial)/'${newID}'\1/' /etc/bind/domains/*.de
then
	echo "[x] Die IDs wurden angepasst."
else
	echo "[ ] Die IDs konnten nicht angepasst werden."
	exit 1
fi
echo ""

grep -Hi "serial" /etc/bind/domains/*.de | tr -s '\ \t' | cut -d";" -f1
echo ""

for i in {1..3}
do
	echo "[${i}/3] Der Dienst wird gleich neu gestartet..."
	sleep 1s
done

tail -f /var/log/bind/default.log &amp;
TAIL_PID=$!
trap "kill ${TAIL_PID}" EXIT
service named restart
sleep 10s
</pre>

## Fail2ban

[Bind 9 Jails](../fail2ban#bind-9-jails)