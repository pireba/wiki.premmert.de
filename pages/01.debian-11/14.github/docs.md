---
title: Github
media_order: 'Screenshot 2022-06-27 at 11-06-21 Build software better together.png'
---

[TOC]

{$USER$} == Benutzer  
{$MAIL$} == E-Mail-Adresse  
{$GPGKEY$} == GPG-Key ID

## GPG

### Paket installieren

```bash
apt install gpg
```

### Zertifikat importieren

```bash
gpg --import gpg.asc
```

<pre>
gpg: key G32E1EF75672005B: public key "Max Mustermann (Github) &lt;max@mustermann.de&gt;" imported
gpg: key G32E1EF75672005B: secret key imported
gpg: key G32E1EF75672005B: "Max Mustermann (Github) &lt;max@mustermann.de&gt;" not changed
gpg: Total number processed: 2
gpg:               imported: 1
gpg:              unchanged: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
</pre>

## Github

### Paket installieren

```bash
apt install git
```

### Konfiguration

```bash
git config --global user.name "{$USER$}"
git config --global user.email "{$MAIL$}"
git config --global user.signingkey {$GPGKEY$}
```

Beispiel:

```bash
git config --global user.name "Max Mustermann"
git config --global user.email "max@mustermann.de"
git config --global user.signingkey 47110815A
```

### SSH-Key erstellen

```bash
ssh-keygen -t ed25519 -C "{$MAIL$}"
```

Beispiel:

```bash
ssh-keygen -t ed25519 -C "max@mustermann.de"
```

<pre>
Generating public/private ed25519 key pair.
Enter file in which to save the key (/root/.ssh/id_ed25519): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_ed25519
Your public key has been saved in /root/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:QuPwZbJm2zm6x3YGRHq887Jy/rsM7Rx/4zbI8HYinO0 max@mustermann.de
The key's randomart image is:
+--[ED25519 256]--+
|                 |
|        .        |
|    . ++o        |
|     =.*+        |
|      BoS.       |
|     o ++o.      |
|      ..==o* .   |
|      ..**BoB *  |
|      oBo*B*E*.o |
+----[SHA256]-----+
</pre>

### Öffentlichen Schlüssel anzeigen lassen

```bash
cat /root/.ssh/id_ed25519.pub
```

<pre>
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILsX5hhEdZayjcto9OtN59LiQCdtKOzPNnQ8aU1q7swL max@mustermann.de
</pre>

### Auf Github hinterlegen

<https://github.com/settings/ssh/new>

![SSH keys / Add new](Screenshot%202022-06-27%20at%2011-06-21%20Build%20software%20better%20together.png "SSH keys / Add new")