---
title: 'Mega CMD'
---

[TOC]

{$USER$} == Benutzer  
{$PASSWORD$} == Passwort

## Installation

### Abhängigkeiten installieren

```bash
apt install libc-ares2 libpcrecpp0v5
```

### Paket herunterladen

```bash
wget https://mega.nz/linux/repo/Debian_11/amd64/megacmd-Debian_11_amd64.deb
```

### Paket installieren

```bash
dpkg -i megacmd-Debian_11_amd64.deb
```

## Bedienung

### Einloggen

```bash
mega-login {$USER$} '{$PASSWORD$}'
```

Beispiel:

```bash
mega-login user@mail.com 'G3h31m3sPassw0rt'
```

### Synchronisation

#### Externe Ordner erstellen

```bash
mega-mkdir /backup
mega-mkdir /nextcloud
```

#### Synchronisationen einrichten

```bash
mega-sync /backup/ /backup
mega-sync /nextcloud /nextcloud
```

#### Status anzeigen lassen

```bash
mega-sync 
```

<pre>
ID          LOCALPATH  REMOTEPATH ACTIVE  STATUS  ERROR SIZE    FILES DIRS
yS82BIdL2WM /backup    /backup    Enabled Ignored NO    0.00  B 0     0   
18YfTvbwlRY /nextcloud /nextcloud Enabled Ignored NO    0.00  B 0     0
</pre>

## Troubleshooting

### Speicherplatzgrenze erreicht

#### Ausloggen

```bash
mega-quit
```

#### Einloggen

```bash
mega-login {$USER$} '{$PASSWORD$}'
```

Beispiel:

```bash
mega-login user@mail.com 'G3h31m3sPassw0rt'
```

#### Sync reaktivieren

```bash
mega-sync --enable /backup
```

#### Status anzeigen

```bash
mega-sync
```