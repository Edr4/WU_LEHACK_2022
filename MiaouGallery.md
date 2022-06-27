## Writeup 🚩 MiaouGallery - LEHACK 2022

## Description

AAAAAAAAAAAAje ne l'ai pas


## Flag

```lh2022_{MyC41Is5OcuT3d0n'1UThINk?}```


## Exploitation 

Miaou Gallery est un site web qui permet d'upload des images
![](https://i.imgur.com/veO1AH0.png)

J'essaye d'upload des fichiers php, null bytes, doubles extensions... mais ça ne fonctionne pas
Après des cherches dans les conditions de sécurité du site web on a les infos de base de données, c'est très certainement une SQLI

![](https://i.imgur.com/LvGFnUS.png)
Mais comment faire une sql injection dans un fichier JPG ?
Dans les EXIFS ! 


Site qui liste tous les Exifs : https://exiv2.org/tags.html
Je tente d'upload plusieurs images avec des EXIFS contenant des guimmets mais ça ne marche pas.

Alors je découvre qu'imageUniqueID, Filename et Copyright sont des exifs 

Ajout des exifs sur l'image
![](https://i.imgur.com/zubFXLZ.png)

ils sont affichés sur l'image
![](https://i.imgur.com/dH6itt2.png)

Si j'ajoute un guimmet sur l'exif ImageUniqueID et je récupère une erreur sql
![](https://i.imgur.com/RaA2EOQ.png)

Je modifie la requête pour rajouter un select de VerySecretThing dans la SQLI et ça fonctionne
![](https://i.imgur.com/BeYQ0je.png)

![](https://i.imgur.com/s964KkO.png)

Comment exfilter les données ?

On a les erreurs en retour donc on peut faire une SQL Injection Error Based

J'essaye plein de payload ça ne marche pas, je tombe sur une doc magique de exploit db https://www.exploit-db.com/docs/33253
![](https://i.imgur.com/9TOZY50.png)

je test avec le payload suivant
```sql=
or updatexml(1,concat(0x7e,(version())),0) or
```
![](https://i.imgur.com/rVpsXZ6.png)
Et je récupère bien la version de mysql

![](https://i.imgur.com/H7rlQY6.png)

Maintenant j'ai juste à récupérer le contenu de la table VerySecretThing
J'ajoute un limit à la fin pour éviter de récupérer plusieurs lignes

![](https://i.imgur.com/FEiBd11.png)
J'ai bien le début du flag (:

![](https://i.imgur.com/ZxPeM4T.png)
Je rajoute un substr() pour avoir la fin du flag

![](https://i.imgur.com/p4vYYxm.png)

![](https://i.imgur.com/ZkwvRWh.png)

Payload
```sql=
or updatexml(1,concat(0x7e,(SELECT SUBSTR(VerySecretThing, 20, 50) FROM image limit 0,1)),0) or 

Voilà flag 🙂
