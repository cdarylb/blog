Title: Combien de workers pour nginx
Date: 2012-08-10 11:16
Author: cyrilb
Category: linux, nginx
Tags: nginx, process, workers
Slug: combien-de-workers-pour-nginx

La question du « combien je mets de worker\_processes et
worker\_connections dans mon nginx.conf » revient souvent. Voici
quelques lignes afin de savoir à quoi servent ces workers et de combien
vous en avez besoin, et surtout, de savoir combien de requêtes clients
vous allez pouvoir servir avec votre configuration.

**worker\_processes** : Vient du Latin workus processus (ok c’est pas
drôle). C’est tout simplement le nombre de « single-threaded process »
qui vont spawner, un peu à l’instar des child process de Apache. Ca sert
à plein de choses (diminuer la latence lorsque des workers sont bloqués
par trop d’I/O sur le disque, limiter le nombre de connexions par
process, etc). Dans la pratique, ça donne ça (remarquez le master
process qui fait démarrer les workers process):

```bash
root@elendil:~# ps auxfw | grep -i nginx | grep -v grep
root 6718 0.0 0.0 35496 1180 ? Ss Jul03 0:00 nginx: master process /usr/local/nginx/sbin/nginx
nobolife 6719 0.0 0.0 37272 4092 ? S Jul03 22:01 \_ nginx: worker process
nobolife 6721 0.0 0.0 36980 3868 ? S Jul03 21:02 \_ nginx: worker process
nobolife 6722 0.0 0.0 37056 4192 ? S Jul03 21:39 \_ nginx: worker process
nobolife 6723 0.0 0.0 37004 3980 ? S Jul03 21:40 \_ nginx: worker process
```

Alors bien sûr, vous allez me demander pourquoi j’ai mis ma directive
worker\_processes à 4. Une fois de plus avec nginx c’est très simple. Le
nombre de workers dépend tout simplement du nombre de cores que vous
avez. Attention, là on est sur une configuration de type frontal web qui
utilise gzip (setté à 9 chez moi) et SSL, avec un fort traffic, bref,
une utilisation plus ou moins classique. Par contre, si vous utilisiez
nginx pour servir des images statiques et que la taille totale de tous
ces fichiers est supérieure au total de votre mémoire vive, les
intégristes de nginx préconisent de baisser le nombre de workers process
afin d’utiliser pleinement la bande passante de votre disque dur.

Il y a plein d’options que je vous encourage à tester avec la directive
workers\_processes: worker\_priority qui permet de donner une priorité à
tel ou tel processeur, worker\_cpu\_affinity qui permet de binder un
worker à un cpu dédié, worker\_rlimit\_nofile…

**worker\_connections** : C’est le nombre de connections client que peut
recevoir un process. Avec 1024 par défaut dans le nginx.conf, je vous
suggère de vous baser sur cette simple équation pour connaître votre
valeur de max\_clients :

> max\_clients = worker\_processes \* worker\_connections

Il vous suffit ensuite de tester l’équation en pratique sur les pages de
votre site avec [Apache
bench](http://httpd.apache.org/docs/2.2/programs/ab.html "Apache bench") ou
encore
mieux, [httperf](http://linux.die.net/man/1/httperf "httperf") afin de
tester l’utilisation mémoire et vos temps de réponse. A vous de jouer !

A noter que vous retrouverez toutes ces informations sur le [wiki de
nginx](http://wiki.nginx.org/Main "Wiki Nginx").
