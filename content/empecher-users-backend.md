Title: Empêcher un utilisateur d'accéder à vos backends avec haproxy
Date: 2014-03-14 13:30
Author: cyrilb
Category: haproxy
Tags: acl, haproxy
Slug: Block-user

Empêcher un utilisateur d'accéder à votre site internet qui est derrière
[haproxy](http://haproxy.1wt.eu/ "Haproxy homepage") est très simple, et
autant le bloquer directement depuis haproxy.

Une fois encore, la souplesse des acl de haproxy permet de le faire en
30 secondes.

Exemple pour bannir une adresse ip :

```bash
acl bad_ip src www.xxx.yyy.zzz
tcp-request connection reject if bad_ip
```

On peut bien sûr remplacer l'adresse ip par une liste d'ip stockées dans
un fichier plat :

```bash
acl bad_ip src -f /opt/blacklist.txt
tcp-request connection reject if bad_ip
```

