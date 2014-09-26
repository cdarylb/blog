Title: Modifier les servers headers sur nginx
Date: 2012-07-18 11:20
Author: cyrilb
Category: nginx
Tags: headers, nginx
Slug: modifier-les-servers-headers-sur-nginx

Nginx est à la fois un serveur HTTP et un reverse proxy. Pour ceux qui
ne connaissent pas, je leur conseille de l’essayer sans tarder tant il
s’avère robuste et souple (les fichiers de configuration sont un vrai
régal pour qui vient d’Apache et sa relative lourdeur). C’est bien
simple, à requêtes égales, nginx consomme beaucoup moins de processeur
et de mémoire. L’essayer c’est l’adopter.  
Nous allons aujourd’hui aborder une petite astuce très simples à mettre
en place sur Nginx en moins de trente secondes.

Modifier les servers header de nginx, afin qu’un curl par exemple nous
retourne un « Server : Plop » plutôt qu’un « Server : Nginx ». A quoi ça
sert? Surtout à éviter les scripts-kiddies qui se basent sur les headers
afin d’exploiter des failles ou autre, et de rester discret si vous le
souhaitez sur la technologie utilisée pour propulser votre serveur web.
Et puis c’est toujours classe de renvoyer des headers bien
personnalisés.

Je me base sur la version actuelle stable de nginx pour ces exemples,
soit la release 1.2.2.

​1. Téléchargez les sources sur le site officiel.

​2. Décompressez où vous voulez (suivant votre sens plus ou moins
développé du FHS).

​3. Editez avec votre éditeur favori le fichier
src/http/ngx\_http\_header\_filter\_module.c

​4. Modifiez la réponse des lignes 49 et 50 :

```bash
static char ngx\_http\_server\_string[] = "Server: nginx" CRLF;  
static char ngx\_http\_server\_full\_string[] = "Server: " NGINX\_VER CRLF;
```

​5. Sauvegardez, puis compilez avec les options de votre choix, lancez
nginx, puis vérifiez vos headers avec un curl -I http://www.monsite.com/
par exemple. Enjoy, cela vous a pris 30 secondes.
