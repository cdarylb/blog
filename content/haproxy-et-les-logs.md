Title: Haproxy et les logs
Date: 2014-05-09 09:19
Author: cyrilb
Category: haproxy
Tags: haproxy, logs
Slug: haproxy-et-les-logs

Utiliser HaProxy sans activer les logs  et savoir les lire, c'est comme
être une fille et ne pas avoir de shampooing. Voici quelques clefs pour
y voir un peu plus clair. Ce billet ne parlera que des logs HTTP, mais
c'est plus ou moins la même chose pour TCP.

**1. Activer les logs**

Ca se fait en deux secondes.

Dans la section 'global' on ajoute :

```bash
log /dev/log local0 info
log /dev/log local0 notice
```

Je pourrai bien sûr les renvoyer directement sur un serveur de logs genre :

```bash
log 10.17.9.5 notice
```

Dans la section 'default' on ajoute (optionnel) :

```bash
option log-health-checks
```


On se rend ensuite dans /etc/rsyslog.d (pour les utilisateurs de Ubuntu)
et on crée le fichier haproxy.conf :

```bash
if ($programname == 'haproxy' and $syslogseverity-text == 'info') then -/var/log/haproxy/haproxy-info.log
& ~
if ($programname == 'haproxy' and $syslogseverity-text == 'notice') then -/var/log/haproxy/haproxy-notice.log
& ~
```

Comme on veut faire les choses bien, on va configurer la rotation des
logs générés par haproxy avec logrotate. Il suffit de se rendre dans
/etc/logrotate.d/ et de créer le fichier haproxy :

```bash
/var/log/haproxy/haproxy-*.log {
    missingok
    notifempty
    sharedscripts
    rotate 14
    daily
    compress
    postrotate
    reload rsyslog >/dev/null 2>&1 || true
    endscript
    }
```

Une fois tout cela effectué, relancez haproxy/syslog/votre chien pour
que toutes les modifications soient prises en compte.

**2. Comment lire les logs**

Voici à quoi ressemble une ligne de logs dans haproxy :

```bash
lb1 haproxy[7112]: 157.56.92.164:41073 [09/May/2014:09:06:59.112] main webback/web3 0/0/0/298/298 200 9041 - - --NI 11/11/0/0/0 0/0 "GET /fr/heros/menage/bulligny-54113/121324 HTTP/1.1"
```

C'est vraiment bluffant de constater à quel niveau de détail on peut
descendre dans cette ligne de logs. Voici à quoi chaque segment
correspond.

**lb1** : Le hostname du serveur qui a écrit le log.  
**haproxy[7112]** : Le nom du process/PID qui a écrit le log.  
**157.56.92.164:41073** : L'adresse IP du client + PORT  
**[09/May/2014:09:06:59.112]** : L'accept date du log  
**main** : Nom du frontend  
**webback/web3** : Backend/serveur qui a servi la requête  
**0/0/0/298/298** : Ca se corse avec ces 5 champs, nous pouvons les
segmenter comme suit  
**Tq** : Le temps passé en ms de la requête http du client  
**Tw** : Le temps passé en ms d'attente des queues si il y en a  
**Tc** : Le temps passé en ms d'attente de connexion au serveur  
**Tr** : Le temps en ms d'envoi de la réponse du serveur vers le
client  
**Tt** : Le temps total de parcours entre l'ouverture de connexion et
la fermeture de la connexion  
**200** : Code retour http  
**9041** : Nombre total de bits transmis au client  
**-** : captured\_request\_cookie  
**-** : captured\_response\_cookie  
**--NI** : Comment la session s'est terminée, "termination\_state", j'y
reviendrai un autre jour, là j'ai piscine.  
**11/11/0/0/0** : On peut segmenter ce tronçon comme suit  
**actonn** : Le nombre total de connexions lorsque la session a été
loggée  
**feconn** : Le nombre total de connexions sur le frontend  
**beconn** : Le nombre total de connexions sur le backend  
**srv\_conn** : Le nombre de connexions actives sur le serveur  
**retries** : Le nombre de connexions qu'il a fallu pour se connecter
au serveur  
**0/0** Découpage comme suit  
**srv\_queue** : Nombre de requêtes qu'il a fallu passer avant de se
connecter au serveur.  
**backend\_queue** : Nombre de requêtes qu'il a fallu passer avant de
se connecter au backend.  
**"GET /fr/heros/menage/bulligny-54113/121324 HTTP/1.1"** : Notre
requête HTTP.

Hey, le format des logs ne vous convient pas ? Pas de problèmes, vous
pouvez en changer aisément. Mettons que par exemple vous ne désiriez
logger que l'adresse IP du client et la requête HTTP, la directive
log-format permet de le faire :

```bash
log-format %ci %r
```

Toutes ces variables sont bien sûres accessibles dans la documentation
de haproxy.

**3. Analyser les logs de haproxy grâce à halog**

C'est là que ça devient intéressant grâce à halog. Pour l'instant il
faut l'installer en se rendant dans les sources de
haproxy/contrib/halog.

```bash
root@corniaud:/usr/local/src/haproxy-1.5-dev22/contrib/halog# make
gcc -O3  -o halog -I../../include -I../../ebtree ../../ebtree/ebtree.c ../../ebtree/eb32tree.c ../../ebtree/eb64tree.c ../../ebtree/ebmbtree.c ../../ebtree/ebsttree.c ../../ebtree/ebistree.c ../../ebtree/ebimtree.c halog.c fgets2.c
```

Une fois halog installé (vous pouvez en faire un lien symbolique vers
/usr/bin) voici quelques exemples de ce que halog permet de faire :

Voir les derniers messages d'erreur tirés des logs de haproxy :

```bash
cat /var/log/haproxy/haproxy-info.log | halog -e
May  6 11:25:04 lb2 haproxy[7112]: 82.236.50.216:53111 [06/May/2014:11:25:03.191] main~ webback/web2 64/0/0/-1/1373 -1 0 - - CHVN 31/31/0/0/0 0/0 "POST /up/account/pic/upload HTTP/1.1"
May  6 11:29:19 lb2 haproxy[7112]: 80.12.100.198:7300 [06/May/2014:11:29:18.555] main~ webback/web3 1399/0/-1/-1/1399 503 212 - - CCVN 38/38/0/0/0 0/0 "GET /f/pics/user_3263079/big_9cid38215a7f2177ed95106e78754d69b088d06b1.jpeg HTTP/1.1"
```

En tirer carrément des stats sur le nombre de hits, de temps de réponse,
de codes retours http, etc...

```bash
root@lb2:~# cat /var/log/haproxy/haproxy-info.log | halog -srv -H -q | awk 'NR==1; NR>1 { print $0 | "sort -k12rn,12"}' | column -t
#srv_name               1xx  2xx    3xx   4xx   5xx  other  tot_req  req_ok  pct_ok  avg_ct  avg_rt
fuel1/monitor02          0    2023   37    0     0    0      2060     2060    100.0   0       131
webback/web3            0    16286  1940  37    12   3      18278    18265   99.9    1       70
webback/web2            0    15574  1992  33    4    3      17606    17602   100.0   1       63
dagobah/lb2              0    447    0     0     0    0      447      447     100.0   0       53
webback/web1            0    17000  1831  36    5    1      18873    18867   100.0   1       47
badcountry/maintenance  0    2      0     12    0    0      14       14      100.0   0       0
main/<NOSRV>            0    0      0     3650  0    0      3650     0       0.0     0       0
```

On peut vraiment tout faire, je vous laisse découvrir la suite, mais
c'est l'outil idéal pour tracker les erreurs 50x par exemple ou trouver
un mouton noir dans votre cluster de serveurs.

**4. Aller (un peu) plus loin avec les logs de haproxy.**

Vous vous en êtes rendu compte, même si de base, les logs sont
pleinement exploitables, on peut ajouter d'autres infos, par exemple (à
mettre dans votre frontend par exemple) :

```bash
capture request header Referer               len 64
capture request header User-Agent            len 128
capture request header Host                  len 64
capture request header X-Forwarded-For       len 64
capture request header Accept-Encoding       len 64
```

C'est assez parlant non ? On peut aller encore plus loin (l'exemple
parle de lui même une fois de plus) :

```bash
capture response header X-WebApp-Id     len 5
capture response header X-Req-My-Uid       len 36
rspidel ^(X-WebApp-Id|Server|X-Req-My-Uid):
```

Le prochain billet sera certainement consacré aux "termination\_states"
qui sont indispensables.
