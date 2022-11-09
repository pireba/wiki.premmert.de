---
title: Fail2ban
taxonomy:
    category:
        - docs
---

[TOC]

## Paket installieren

```bash
apt install fail2ban
```

## Konfiguration

### Standard Einstellungen

```bash
nano /etc/fail2ban/jail.local
```

<pre>
[DEFAULT]
ignoreip  = 127.0.0.1/8 ::1
bantime   = 1h
findtime  = 10m
maxretry  = 3
banaction = shorewall
action    = %(action_mwl)s
destemail = root
sender    = fail2ban@premmert.de
mta       = sendmail
</pre>

### Apache2 Jails

```bash
nano /etc/fail2ban/jail.local
```

<pre>
...
# Apache: Catches authorization errors.
[apache-auth]
enabled  = true

# Apache: Catches known spambots and software alike.
[apache-badbots]
enabled  = true

# Apache: Catches certain URLs that do not exist.
[apache-botsearch]
enabled  = true

# Apache: Catches fake Googlebot User Agents.
[apache-fakegooglebot]
enabled  = true

# Apache: Catches mod_security failures.
[apache-modsecurity]
enabled  = false

# Apache: Catches errors when locating a home directory.
[apache-nohome]
enabled  = true

# Apache: Catches requests for non-existent scripts.
[apache-noscript]
enabled  = true

# Apache: Catches long or suspicious requests.
[apache-overflows]
enabled  = true

# Apache: Catches queries with custom headers that attempt to exploit the Shellshock bug.
[apache-shellshock]
enabled  = true

# Apache: Catches queries with a URL as a script parameter, which may be an indication of a fopen url php injection.
[php-url-fopen]
enabled = true
...
</pre>

### Bind 9 Jails

```bash
nano /etc/fail2ban/jail.local
```

<pre>
...
# Bind: Catches TCP attacks against bind9.
[named-refused-tcp]
enabled  = true
filter   = named-refused
port     = domain,953
protocol = tcp
logpath  = /var/log/bind/security.log

# Bind: Catches UDP attacks against bind9.
[named-refused-udp]
enabled  = true
filter   = named-refused
port     = domain,953
protocol = udp
logpath  = /var/log/bind/security.log
...
</pre>

### SSH Jails

```bash
nano /etc/fail2ban/jail.local
```

<pre>
...
# SSH: Catches authorization errors.
[sshd]
enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log
...
</pre>

## Shorewall Blocktype

```bash
nano /etc/fail2ban/action.d/shorewall.conf
```

<pre>
#blocktype = reject
blocktype = drop
</pre>

## Dienst

### Dienst neu starten

```bash
systemctl restart fail2ban.service
```

### Autostart

```bash
systemctl enable fail2ban.service
```

## Bedienung

### Über Fail2ban

#### Gesperrte Adressen anzeigen

```bash
fail2ban-client banned
```

<pre>
[{'sshd': ['<b>1.2.3.4</b>', '<b>5.6.7.8</b>']}, {'apache-auth': []}, {'apache-badbots': ['<b>2001:0db8:85a3:0000:0000:8a2e:0370:7334</b>']}, {'apache-noscript': []}, {'apache-overflows': []}, {'apache-nohome': []}, {'apache-botsearch': []}, {'apache-fakegooglebot': []}, {'apache-shellshock': []}, {'php-url-fopen': []}, {'named-refused-tcp': []}, {'named-refused-udp': []}]
</pre>

#### Gesperrte Adressen freigeben

```bash
fail2ban-client set sshd unbanip 1.2.3.4
```

<pre>
1
</pre>

### Über Shorewall

#### Gesperrte Adressen anzeigen

```bash
shorewall show dynamic
```

<pre>
Shorewall 5.2.3.4 Chain dynamic at vmd89862.contaboserver.net - Fri 11 Mar 2022 09:49:44 PM CET

Counters reset Fri 11 Mar 2022 09:08:02 PM CET

# Warning: iptables-legacy tables present, use iptables-legacy to see them
Chain dynamic (4 references)
 pkts bytes target     prot opt in     out     source               destination         
<b>   29  1740 DROP       all  --  *      *       218.92.0.205         0.0.0.0/0</b>
<b>    5   300 DROP       all  --  *      *       62.157.225.138       0.0.0.0/0</b>
</pre>

#### Gesperrte Adressen freigeben

```bash
shorewall allow 62.157.225.138
```

<pre>
62.157.225.138 Allowed
</pre>