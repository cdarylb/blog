Title: Tester les performances de ses disques
Date: 2012-10-23 11:11
Author: cyrilb
Category: linux
Tags: hdparm, iostat
Slug: tester-les-performances-de-ses-disques

Je suis en train de changer d’hébergeur, du coup je me retrouve à tester
des serveurs différents de ceux que j’ai actuellement. Afin de m’assurer
de la performance du nouveau matériel qu’on me propose, je déroule ces
trois petits tests unitaires qui me permettent en moins de 20 minutes de
voir si la puissance I/O fournie est suffisante pour ce que je veux en
faire, de manière certes grossière mais pragmatique.

**1. Tests avec DD (Dédé pour les intimes).**

C’est le premier tests que je lance, et le plus basique. Il me permet de
tester la vitesse d’écriture d’un bloc de 1M (bs=1M) en prenant en
compte la synchronisation des données entre la mémoire et le disque
(conv=fdatasync physically write output file data before finishing).
Simple et efficace, ça me donne tout de suite un premier indicateur. Ne
pas hésiter à jouer avec la taille du bloc bien sûr.

```bash
root@mirkwood:~# dd if=/dev/zero of=test bs=1M count=1024 conv=fdatasync
1024+0 records in
1024+0 records out
1073741824 bytes (1,1 GB) copied, 8,46055 s, 127 MB/s
```

**2. Tests avec hdparm**

Nous allons maintenant tester la vitesse de lecture grâce à hdparm.
Option intéressante, hdparm permet de tester la lecture depuis le cache
du disque (option -t).

```bash
root@mirkwood:~# hdparm -t -T /dev/sda
/dev/sda:
Timing cached reads: 12532 MB in 2.00 seconds = 6270.87 MB/sec
Timing buffered disk reads: 742 MB in 3.00 seconds = 247.13 MB/sec
```

**3. Derniers tests avec IoZone.**

On termine avec la Rolls des tests I/O: iozone. En plus de proposer des
options vraiment sexy comme le re-read ou le re-write, sans parler du
random read et random write, il permet d’exporter les résultats du bench
sous format xls qui permettra d’importer vers excel et d’en faire des
beaux tableaux. Iozone est vraiment bien fait, et fourmille d’options.
pour ma part, je me contente généralement d’un (je vous laisse faire un
iozone -h):

```bash
root@mirkood:~/iozone3_394/src/current# ./iozone -a -i 0 -i 1 -b mirkwood.xls
```

A vous de jouer maintenant, et surtout, n’oubliez pas que le type de FS
est primordial (vous n’obtiendrez pas les mêmes résultats entre du ext4
et du XFS par exemple).
