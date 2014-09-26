Title: Aria2c, une alternative à Wget
Date: 2013-03-12 15:59
Author: cyrilb
Category: linux
Tags: aria2c, curl, linux, wget
Slug: aria2c-une-alternative-a-wget

Je me sers pratiquement tous les jours de wget ou de curl, que ce soit
pour downloader un fichier quelconque, pour rapatrier et analyser mes
headers ou tester les performances de certaines parties de mon site.

Je ne parle même pas du nombre de mes scripts qui se servent de curl ou
de wget, bref, impossible de m’en passer, d’autant plus que ces outils
ont l’avantage d’être installé de base sur la plupart des distributions
Linux actuelles.

J’ai néanmoins découvert récemment Aria2, un outil suffisamment sympa
qui gère le download multi-protocoles (http, ftp, bittorent, etc)… et
qui fourmille d’options que je vais vous faire découvrir ici, du moins
celles qui m’ont servies cette semaine:

Le download d’un fichier bittorrent qu’on met en mémoire:

```bash
cyril@hoth:~$ aria2c --follow-torrent=mem http://slackware.com/torrents/slackware-14.0-install-dvd.torrent
```

Voici l’output tronqué pour les curieux:

```bash
\#2 SIZE:177.4MiB/2,345.4MiB(7%) CN:55 SEED:49 SPD:7.5MiBs
UP:66.4KiBs(1.9MiB) ETA:4m48s]
```

Le download de base, avec un fichier de sortie pour les erreurs:

```bash
cyril@hoth:~$ aria2c --save-session=output.log http://mirrors.slackware.com/slackware-iso/slackware64-14.0-iso/slackware64-14.0-install-dvd.iso
```

Le download avec plusieurs sources, le download est donc plus rapide:

```bash
cyril@hoth:~$ aria2c -x 2 http://mirrors.slackware.com/slackware-iso/slackware64-14.0-iso/slackware64-14.0-install-dvd.iso
```

Le download de la même image depuis plusieurs sources (ftp et http):

```bash
cyril@hoth:~$ aria2c -x 2 http://mirrors.slackware.com/slackware-iso/slackware64-14.0-iso/slackware64-14.0-install-dvd.iso ftp://mirrors.slackware.com/slackware-iso/slackware64-14.0-iso/slackware64-14.0-install-dvd.iso
```

Le download de fichiers contenus dans un fichier quelconque, avec une
concurrence de 10 et avec une sortie:

```bash
cyril@lb2:~$ aria2c -istuff.txt -j10 --save-session=output.log
```

Qui nous permettra un joli resume si ça plante grâce à un:

```bash
cyril@hoth:~$ aria2c -ioutput.log
```

Bien sûr la séquence de download sera restaurée automatiquement après un
CTRL-C et aria2 gère même IPV6, que demander de plus.
