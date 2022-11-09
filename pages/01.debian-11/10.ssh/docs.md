---
title: SSH
taxonomy:
    category:
        - docs
---

[TOC]

## Passwort deaktivieren

## Zweite SSH-Server Instanz mit Passwort

### SystemD-Service kopieren

```bash
cp /etc/systemd/system/sshd.service /etc/systemd/system/sshd-1234.service
```

### SystemD-Service anpassen

```bash
nano /etc/systemd/system/sshd-1234.service
```

<pre>
[Unit]
Description=OpenBSD Secure Shell server
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target auditd.service
ConditionPathExists=!/etc/ssh/sshd_not_to_be_run

[Service]
EnvironmentFile=-/etc/default/ssh
ExecStartPre=/usr/sbin/sshd -t
ExecStart=/usr/sbin/sshd -D $SSHD_OPTS <b>-o PermitRootLogin=yes -o PasswordAuthentication=yes -p 1234</b>
ExecReload=/usr/sbin/sshd -t
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartPreventExitStatus=255
Type=notify
RuntimeDirectory=sshd
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
</pre>

### SystemD Daemon neu laden

```bash
systemctl daemon-reload
```

### Dienst starten

```bash
systemctl start sshd-1234.service
```

### Autostart

```bash
systemctl enable sshd-1234.service
```

### Kontrolle

```bash
netstat -tulpen | grep 1234
```

<pre>
tcp        0      0 0.0.0.0:1234            0.0.0.0:*               LISTEN      0          32858903   2999863/sshd: /usr/ 
tcp6       0      0 :::1234                 :::*                    LISTEN      0          32858905   2999863/sshd: /usr/
</pre>

## SSH-Portknocking

[SSH-Portknocking](../shorewall#ssh-portknocking)