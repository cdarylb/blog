Title: Changer sa locale par défaut
Date: 2012-12-26 14:38
Author: cyrilb
Category: linux
Tags: linux, locale
Slug: changer-sa-locale-par-defaut

Pas trop le temps de poster ces temps-ci, je me contenterai juste d’une
petite note qui peut faire gagner pas mal de temps: changer les locales
de son environnement. Je me base sur une distribution Ubuntu Server
12.10 pour les exemples ci-dessous.

De base, mon système était installé avec les locales françaises:

```bash
root@isengard:~# printenv | grep LANG LANG=fr_FR.UTF-8
```

Je désirais passer mon environnement sous locale EN pour de multiples
raisons. Les opérations à effectuer sont très simples:

​1. Rajouter la locale désirée dans le fichier
/var/lib/locales/supported.d/local avec votre éditeur préféré (vi
forcément).

​2. Régénérer la liste des locales supportées:

```bash
root@isengard:~# dpkg-reconfigure locales
    Generating locales...
    en_AG.UTF-8... up-to-date
    en_AU.UTF-8... up-to-date
    en_BW.UTF-8... up-to-date
    en_CA.UTF-8... up-to-date
    en_DK.UTF-8... up-to-date
    en_GB.UTF-8... up-to-date
    en_HK.UTF-8... up-to-date
    en_IE.UTF-8... up-to-date
    en_IN.UTF-8... up-to-date
    en_NG.UTF-8... up-to-date
    en_NZ.UTF-8... up-to-date
    en_PH.UTF-8... up-to-date
    en_SG.UTF-8... up-to-date
    en_US.UTF-8... up-to-date
    en_ZA.UTF-8... up-to-date
    en_ZM.UTF-8... up-to-date
    en_ZW.UTF-8... up-to-date
    fr_BE.UTF-8... up-to-date
    fr_CA.UTF-8... up-to-date
    fr_CH.UTF-8... up-to-date
    fr_FR.UTF-8... up-to-date
    fr_LU.UTF-8... up-to-date
    Generation complete.
```

​3. Editer le fichier /etc/default/locale et rajoutez la locale de votre
choix (dans mon cas j’ai ajouté LANG= »en\_GB.UTF-8″).

​4. Redémarrez votre serveur.

​5. Enjoy !

 
