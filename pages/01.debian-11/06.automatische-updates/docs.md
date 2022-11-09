---
title: 'Automatische Updates'
taxonomy:
    category:
        - docs
---

[TOC]

## Pakete installieren

```bash
apt install unattended-upgrades apt-listchanges
```

## Unattended Upgrades

### Konfiguration

```bash
nano /etc/apt/apt.conf.d/51unattended-upgrades-overwrite
```

<pre>
// https://wiki.debian.org/UnattendedUpgrades
// https://github.com/mvo5/unattended-upgrades

Unattended-Upgrade::Origins-Pattern {
	"o=Debian,	n=${distro_codename},		l=Debian";
	"o=Debian,	n=${distro_codename}-updates,	l=Debian";
	"o=Debian,	n=${distro_codename},		l=Debian-Security";
	"o=Debian,	n=${distro_codename}-security,	l=Debian-Security";
	"o=MEGA,	n=Debian_11,			l=DEB";
	"o=Docker,					l=Docker CE";
};

Unattended-Upgrade::Mail "root";
Unattended-Upgrade::Sender "updates@premmert.de";
//Unattended-Upgrade::MailReport "always";
//Unattended-Upgrade::MailReport "only-on-error";
Unattended-Upgrade::MailReport "on-change";

Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-New-Unused-Dependencies "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";

Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-WithUsers "true";

Unattended-Upgrade::SyslogEnable "true";
Unattended-Upgrade::Verbose "true";
Unattended-Upgrade::MinimalSteps "false";
</pre>

### Aktivieren

```bash
dpkg-reconfigure -plow unattended-upgrades
```

<pre>
Automatically download and install stable updates? &lt;Yes&gt;
Creating config file /etc/apt/apt.conf.d/20auto-upgrades with new version
</pre>

### Testen

```bash
unattended-upgrade --dry-run
```

## apt-listchanges

```bash
nano /etc/apt/listchanges.conf
```

<pre>
[apt]
frontend=mail
email_address=root
confirm=false
save_seen=/var/lib/apt/listchanges.db
which=both
headers=true
</pre>

## Timer

### Download-Timer erstellen

```bash
systemctl edit --full apt-daily.timer
```

<pre>
[Unit]
Description=Daily apt download activities

[Timer]
OnCalendar=*-*-* 04:00
RandomizedDelaySec=0
Persistent=true

[Install]
WantedBy=timers.target
</pre>

### Upgrade-Timer erstellen

```bash
systemctl edit --full apt-daily-upgrade.timer
```

<pre>
[Unit]
Description=Daily apt upgrade and clean activities
After=apt-daily.timer

[Timer]
OnCalendar=*-*-* 04:05
RandomizedDelaySec=0
Persistent=true

[Install]
WantedBy=timers.target
</pre>

### Timer neu starten

```bash
systemctl daemon-reload 
systemctl restart apt-daily.timer 
systemctl restart apt-daily-upgrade.timer
```

### Download-Timer anzeigen lassen

```bash
systemctl status apt-daily.timer
```

<pre>
● apt-daily.timer - Daily apt download activities
     Loaded: loaded (/etc/systemd/system/apt-daily.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Sun 2022-06-12 04:07:47 CEST; 1 weeks 1 days ago
    <b>Trigger: Tue 2022-06-21 04:00:00 CEST; 13h left</b>
   Triggers: ● apt-daily.service

Jun 12 04:07:47 premmert.de systemd[1]: Started Daily apt download activities.
</pre>

### Upgrade-Timer anzeigen lassen

```bash
systemctl status apt-daily-upgrade.timer
```

<pre>
● apt-daily-upgrade.timer - Daily apt upgrade and clean activities
     Loaded: loaded (/etc/systemd/system/apt-daily-upgrade.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Sun 2022-06-12 04:07:47 CEST; 1 weeks 1 days ago
    <b>Trigger: Tue 2022-06-21 04:05:00 CEST; 13h left</b>
   Triggers: ● apt-daily-upgrade.service

Jun 12 04:07:47 premmert.de systemd[1]: Started Daily apt upgrade and clean activities.
</pre>