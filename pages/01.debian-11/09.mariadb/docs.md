---
title: MariaDB
---

[TOC]

{$DATABASE$} == Datenbankname  
{$PASSWORD$} == Passwort  
{$USER$} == Benutzer

## Pakete installieren

```bash
apt install mariadb-server mariadb-client
```

## Secure Installation

```bash
mysql_secure_installation
```

<pre>
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] n
 ... skipping.

You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] n
 ... skipping.

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
</pre>

## An Localhost binden

```bash
nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

<pre>
...
[mysqld]
...
bind-address            = 127.0.0.1
...
</pre>

## Dienst

### Dienst neu starten

```bash
systemctl restart mariadb.service
```

### Autostart

```bash
systemctl enable mariadb.service
```

## Datenbank und Benutzer anlegen

### Datenbankumgebung öffnen

```bash
mariadb
```

### Datenbank erstellen

```sql
MariaDB [(none)]> CREATE DATABASE {$DATABASE$};
```

Beispiel:

```sql
MariaDB [(none)]> CREATE DATABASE nextcloud;
```

<pre>
Query OK, 1 row affected (0.000 sec)
</pre>

### Benutzer erstellen

```sql
MariaDB [(none)]> CREATE USER '{$USER$}'@'localhost' IDENTIFIED BY '{$PASSWORD$}';
```

Beispiel:

```sql
MariaDB [(none)]> CREATE USER 'nextcloudUser'@'localhost' IDENTIFIED BY 'G3h31m3sPassw0rt';
```

<pre>
Query OK, 0 rows affected (0.000 sec)
</pre>

### Berechtigungen setzen

```sql
MariaDB [(none)]> GRANT ALL PRIVILEGES ON {$DATABASE$}.* TO '{$USER$}'@'localhost';
```

```sql
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloudUser'@'localhost';
```

<pre>
Query OK, 0 rows affected (0.000 sec)
</pre>

### Verbindungen vom Localhost zulassen

```sql
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO '{$USER$}'@'localhost' IDENTIFIED BY '{$PASSWORD$}' WITH GRANT OPTION;
```

Beispiel:

```sql
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'nextcloudUser'@'localhost' IDENTIFIED BY 'G3h31m3sPassw0rt' WITH GRANT OPTION;
```

<pre>
Query OK, 0 rows affected (0.000 sec)
</pre>

### Änderungen übernehmen

```sql
MariaDB [(none)]> FLUSH PRIVILEGES;
```

<pre>
Query OK, 0 rows affected (0.000 sec)
</pre>

### Datenbankumgebung verlassen

```sql
MariaDB [(none)]> QUIT
```

<pre>
Bye
</pre>