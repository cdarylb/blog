Title: Mutualiser ses logs
Date: 2013-12-10 19:20
Author: cyrilb
Category: linux
Tags: kibana, linux, logs, logstash, rsyslog, syslog
Slug: mutualiser-ses-logs

Lorqu’on a plusieurs serveurs (webs, mysql, etc), il devient compliqué
d’aller d’une machine à l’autre pour s’assurer que tout va bien ou pour
déceler un problème qui affecte l’ensemble de votre infrastructure.

C’est pour cela, qu’en plus de stocker les logs individuellement sur
chaque serveur, il est fortement recommandé de les exporter sur un
serveur dédié et de les mutualiser afin de pouvoir effectuer des
traitement groupés sur l’ensemble des logs à notre disposition.

Et comme tout bon admin, qui dit traitement mutualisé dit traitements
industrialisés.

Nous allons voir ici deux façons de mutualiser les logs nginx de ses
frontaux webs pour l’exemple, sans trop rentrer dans le détail, la
première avec Rsyslog, la seconde avec logstash.

**Mutualiser ses logs nginx avec Rsyslog:**

[Rsyslog](http://www.rsyslog.com/ "RSyslog") est propulsé de base avec
Ubuntu, rien à configurer donc, ça tourne déjà, on va donc se contenter
de configurer rsyslog afin de parser les logs nginx et de les envoyer au
serveur qui va mutualiser les logs.

Sur les serveurs webs, rajouter le fichier nginx.conf dans
/etc/rsyslog.d/

```bash
$ModLoad imfile
 
$InputFileName          /usr/local/nginx/logs/access.log
$InputFileTag           nginx_access_log:
$InputFileStateFile     nginx_access_log
$InputFileSeverity      info
$InputFileFacility      user
$InputRunFileMonitor
 
$InputFileName          /usr/local/nginx/logs/error.log
$InputFileTag           nginx_error_log:
$InputFileStateFile     nginx_error_log
$InputFileSeverity      info
$InputFileFacility      user
$InputRunFileMonitor
 
# lit le fichier de log toutes les deux secondes
$InputFilePollingInterval 2
 
#envoie les logs en tcp a 192.168.1.15 port 1025
if $syslogtag == 'nginx_access_log:' then @@(z9)192.168.1.15:1025
if $syslogtag == 'nginx_error_log:' then @@(z9)192.168.1.15:1025
 
#ne log pas ce qui est taggé nginx (pour ne pas logger en double)
:syslogtag, contains, "nginx" ~
```

Sur le serveur qui recevra les logs, on s’assurera d’avoir au minimum
ces directives dans /etc/rsyslog.conf (à personnaliser suivant votre
configuration)

```bash
$FileOwner logz
$FileGroup log-prod

$CreateDirs on
$FileCreateMode 0640
$DirCreateMode 0755
$Umask 0022

$ModLoad imtcp
$InputTCPServerRun 1025
$AllowedSender TCP, 127.0.0.1, 192.168.1.0/24
```

Puis on crée un fichier nginx.conf sous /etc/rsyslog.d/ :

```bash
$template nginx_access_log,"/home/logz/remote/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/nginx.access.log"
$template nginx_error_log,"/home/logz/remote/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/nginx.error.log"
 
if $syslogtag == 'nginx_access_log:'         then ?nginx_access_log
if $syslogtag == 'nginx_access_log:'         then ~
 
if $syslogtag == 'nginx_error_log:'                   then ?nginx_error_log
if $syslogtag == 'nginx_error_log:'                   then ~
```

Il suffit maintenant de reloader rsyslog sur les serveurs nginx et sur
le « master » et magie, les logs de vos frontaux webs apparaitront dans
le répertoire /home/logs/remote/.

On peut aller encore beaucoup plus loin, comme envoyer les logs
directement dans une base mysql, ou encore faire transiter les logs via
un fichier fifo pour ne pas avoir à les écrire sur nos serveurs webs et
économiser autant d’ios, ça se fait tout aussi facilement à partir de
rsyslog sans trop d’efforts et avec un minimum d’efforts.

**Mutualiser ses logs avec logstash, redis et Elasticsearch**

La solution des hipsters, des vrais. Le gros avantage de jouer avec
logstash c’est qu’on s’absout des contraintes liées au local\*, on log
donc ce qu’on veut.

Dans l’exemple ci-dessous, on va parser les logs
avec [logstash](http://logstash.net/ "logstash"), les envoyer sur une
instance [redis](http://redis.io/ "redis"), et les manipuler
avec [Elasticsearch](http://www.elasticsearch.org/ "Elasticsearch") avec
une couche de [Kibana](http://three.kibana.org/ "Kibana3") comme web
front end à Elasticsearch.

Sur tous vos frontaux webs (après avoir installé l’openjdk, java
oblige):

```bash
wget http://logstash.objects.dreamhost.com/release/logstash-1.1.13-flatjar.jar
mkdir -p /usr/local/bin/logstash
mv logstash-1.1.13-flatjar.jar /usr/local/bin/logstash/logstash.jar
```

```bash
vi /etc/logstash.conf

input {
 file {
 type =&gt; "nginx_access"
 path =&gt; ["/usr/local/nginx/logs/access.log"]
 discover_interval =&gt; 10
 }
}

input {
 file {
 type =&gt; "nginx_error"
 path =&gt; ["/usr/local/nginx/logs/error.log"]
 discover_interval =&gt; 10
 }
}

filter {
 grok {
 type =&gt; nginx_access
 pattern =&gt; "%{COMBINEDAPACHELOG}"
 }
}

filter {
 grok {
 type =&gt; nginx_error
 pattern =&gt; "%{COMBINEDAPACHELOG}"
 }
}
 
output {
 redis { host =&gt; "10.17.9.12" data_type =&gt; "list" key =&gt; "logstash" }
}
```

Puis hop, on lance logstash:

```bash
/usr/bin/java -jar /usr/local/bin/logstash/logstash.jar agent -f /etc/logstash.conf -w 1
```

Je vous conseille d’écrire un script d’init ou d’en récupérer un.

Rendez-vous ensuite sur le serveur qui accueillera vos logs, et on
installe [elasticsearch](http://www.elasticsearch.org/ "elasticsearch"):

```bash
wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-0.90.1.zip
unzip elasticsearch-0.90.1.zip
rm -rf elasticsearch-0.90.1.zip
mv elasticsearch-0.90.1 elasticsearch
sudo mv elasticsearch /usr/local/share
cd /usr/local/share
sudo chmod 755 elasticsearch
cd $HOME
curl -L http://github.com/elasticsearch/elasticsearch-servicewrapper/tarball/master | tar -xz
mv *servicewrapper*/service /usr/local/share/elasticsearch/bin/
rm -Rf *servicewrapper*
/usr/local/share/elasticsearch/bin/service/elasticsearch install
```

On installe ensuite Redis (un simple apt-get install redis-server
suffit).

Puis installation de logstash et on renseigne le fichier de conf:

```bash
input {
 redis {
 host =&gt; "127.0.0.1"
 port =&gt; 6379
 type =&gt; "redis-input"
 data_type =&gt; "list"
 key =&gt; "logstash"
 format =&gt; "json_event"
 }
}
output {
 elasticsearch {
 host =&gt; "10.17.9.12"
 }
}
```

Et comme on est trop des hipsters, on
installe [Kibana3](http://three.kibana.org/intro.html "Kibana3") qui va
nous permettre de taper notre ES comme des swags.

Si vous avez bien tout suivi, vous aurez accès à un joli dashboard très
pratique.
