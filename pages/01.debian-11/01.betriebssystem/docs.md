---
title: Betriebssystem
taxonomy:
    category:
        - docs
---

[TOC]

## Root-Passwort ändern

```bash
passwd
```

## APT

### Paketquellen

```bash
nano /etc/apt/sources.list
```

<pre>
deb http://ftp.de.debian.org/debian/ bullseye main contrib non-free
deb http://ftp.de.debian.org/debian/ bullseye-updates main contrib non-free
deb http://ftp.de.debian.org/debian/ bullseye-backports main contrib non-free
deb http://security.debian.org/debian-security bullseye-security/updates main contrib non-free
</pre>

### Paketquellen aktualisieren

```bash
apt update
```

### Pakete aktualisieren

```bash
apt upgrade
```

### Nützliche Pakete installieren

```bash
apt install bash-completion bzip2 dnsutils conntrack curl git htop iftop ipcalc iptraf less man-db nano net-tools nmap rsync screen ssh strace tcpdump tcpflow telnet unzip wget
```

## Resolvconf

### Paket deinstallieren

```bash
apt purge resolvconf
```

### DNS-Server eintragen

```bash
nano /etc/resolv.conf
```

<pre>
nameserver ***IP_ADRESSE***
</pre>

## NTP

### Konfiguration

```bash
nano /etc/systemd/timesyncd.conf
```

<pre>
...
[Time]
NTP=0.de.pool.ntp.org 1.de.pool.ntp.org 2.de.pool.ntp.org 3.de.pool.ntp.org
...
</pre>

### Dienst neu starten

```bash
systemctl restart systemd-timesyncd.service
```

### Autostart

```bash
systemctl enable systemd-timesyncd.service
```

### Kontrolle

```bash
systemctl status systemd-timesyncd.service
```

<pre>
● systemd-timesyncd.service - Network Time Synchronization
     Loaded: loaded (/lib/systemd/system/systemd-timesyncd.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2022-06-12 04:07:47 CEST; 1 weeks 1 days ago
       Docs: man:systemd-timesyncd.service(8)
   Main PID: 450 (systemd-timesyn)
     Status: "Initial synchronization to time server [2a02:c207:3004:9819::1]:123 (2.de.pool.ntp.org)."
      Tasks: 2 (limit: 9510)
     Memory: 1.0M
        CPU: 15.482s
     CGroup: /system.slice/systemd-timesyncd.service
             └─450 /lib/systemd/systemd-timesyncd

Jun 12 04:07:47 premmert.de systemd[1]: Starting Network Time Synchronization...
Jun 12 04:07:47 premmert.de systemd[1]: Started Network Time Synchronization.
<b>Jun 12 04:08:18 premmert.de systemd-timesyncd[450]: Initial synchronization to time server [2a02:c207:3004:9819::1]:123 (2.de.pool.ntp.org).</b>
</pre>

## AppArmor

### Dienst beenden

```bash
systemctl stop apparmor.service
```

### Dienst deaktivieren

```bash
systemctl disable apparmor.service
```

### Paket entfernen

```bash
apt purge apparmor
```

### Im Grub deaktivieren

```bash
nano /etc/default/grub.d/apparmor.cfg
```

<pre>
GRUB_CMDLINE_LINUX_DEFAULT="$GRUB_CMDLINE_LINUX_DEFAULT apparmor=0"
</pre>

### Grub neu erstellen

```bash
update-grub
```

### Paket halten

```bash
apt-mark hold apparmor
```

## Mails

### Mails an Root weiterleiten

```bash
nano /etc/aliases
```

<pre>
...
root:admin@premmert.de
...
</pre>

### Aliases neu laden

```bash
newaliases
```

## Spracheinstellungen

### Uhrzeit

```bash
localectl set-locale LC_TIME=de_DE.UTF-8
```

## Message of the Day (MOTD)

```bash
nano /etc/motd
```

<pre>

</pre>

## SSH

### SFTP

```bash
nano /etc/ssh/sshd_config
```

<pre>
...
#Subsystem      sftp    /usr/lib/openssh/sftp-server
Subsystem       sftp    internal-sftp
...
</pre>

### Dienst neu starten

```bash
systemctl restart sshd.service
```

## Neustart

```bash
reboot
```