---
title: Shorewall
taxonomy:
    category:
        - docs
process:
    markdown: true
    twig: true
---

[TOC]

## Pakete installieren

```bash
apt install shorewall shorewall6
```

## IPv4

### Vorlagen kopieren

```bash
cd /usr/share/shorewall/configfiles/
cp interfaces policy rules shorewall.conf zones /etc/shorewall/
```

### Interfaces

```bash
nano /etc/shorewall/interfaces
```

<pre>
#
# Shorewall -- /etc/shorewall/interfaces
#
# For information about entries in this file, type "man shorewall-interfaces"
#
# The manpage is also online at
# http://www.shorewall.net/manpages/shorewall-interfaces.html
#
?FORMAT 2
###############################################################################
#ZONE		INTERFACE		OPTIONS

wan		eth0

docker		docker+			routeback=1
docker		br-+			routeback=1
</pre>

### Policy

```bash
nano /etc/shorewall/policy
```

<pre>
#
# Shorewall -- /etc/shorewall/policy
#
# For information about entries in this file, type "man shorewall-policy"
#
# The manpage is also online at
# http://www.shorewall.net/manpages/shorewall-policy.html
#
###############################################################################
#SOURCE		DEST		POLICY	LOGLEVEL	RATE	CONNLIMIT

docker		all		ACCEPT
fw		docker		ACCEPT

all		all		DROP	info
</pre>

### Rules

```bash
nano /etc/shorewall/rules
```

<pre>
...
</pre>

### Shorewall.conf

```bash
nano /etc/shorewall/shorewall.conf
```

<pre>
...
</pre>

### Zones

```bash
nano /etc/shorewall/zones
```

<pre>
#
# Shorewall -- /etc/shorewall/zones
#
# For information about this file, type "man shorewall-zones"
#
# The manpage is also online at
# http://www.shorewall.net/manpages/shorewall-zones.html
#
###############################################################################
#ZONE		TYPE		OPTIONS		IN_OPTIONS	OUT_OPTIONS

fw		firewall
wan		ip
docker		ip
</pre>

### Shorewall starten

```bash
shorewall check && shorewall restart
```

### Shorewall Autostart

```bash
systemctl enable shorewall.service
```

## IPv6

### Vorlagen kopieren

```bash
cd /usr/share/shorewall6/configfiles/
cp params shorewall6.conf /etc/shorewall6/
```

### Shorewall6.conf

```bash
nano /etc/shorewall6/shorewall6.conf
```

<pre>
...
#CONFIG_PATH=":${CONFDIR}/shorewall6:/usr/share/shorewall6:${SHAREDIR}/shorewall"
CONFIG_PATH=":${CONFDIR}/shorewall6:${CONFDIR}/shorewall:/usr/share/shorewall6:${SHAREDIR}/shorewall"
...
</pre>

### Shorewall6 starten

```bash
shorewall6 check && shorewall6 restart
```

### Shorewall6 Autostart

```bash
systemctl enable shorewall6.service
```

## SSH-Portknocking

### Vorlagen kopieren

```bash
cp /usr/share/shorewall/configfiles/actions /etc/shorewall/
cp /usr/share/shorewall/action.template /etc/shorewall/action.SSHKnock
```

### Action anlegen

```bash
nano /etc/shorewall/actions
```

<pre>
#ACTION		OPTIONS		COMMENT

SSHKnock			# SSH Port Knocking
</pre>

### Action definieren

```bash
nano /etc/shorewall/action.SSHKnock
```

<pre>
#ACTION					SOURCE		DEST		PROTO	DPORT	SPORT		ORIGDEST	RATE		USER	MARCONNLIMIT	TIME         HEADERS         SWITCH        HELPER

IfEvent(SSH,ACCEPT:info,60,1,src,reset)	-		-		tcp	1234
SetEvent(SSH,DROP:info)			-		-		tcp	9999
ResetEvent(SSH,DROP:info)
</pre>

### Action einbinden

```bash
nano /etc/shorewall/rules
```

<pre>
...
SSHKnock	wan		fw		tcp	1234,9998-10000
...
</pre>

### Shorewall neu starten

```bash
shorewall check && shorewall restart
shorewall6 check && shorewall6 restart
```