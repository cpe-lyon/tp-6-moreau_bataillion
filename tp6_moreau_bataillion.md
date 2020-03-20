BATAILLION Alice  
MOREAU Marianne  
4 ETI 

# TP 6 - Gestion des disques, Boot, Gestion des logs  

## Exercice 1. Disques et partitions  

**1. Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement alloués ; puis démarrez la VM**  
	Dans config, stockage, rajouter un dd sata.  
**2. Vérifiez que ce nouveau disque dur est bien détecté par le système**  
`ls /dev/sd*` pour voir les noms des disques attachés. Le nouveau s’appelle sdb (les disque s’appellent sd puis une lettre dans l’ordre de création)  
**3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83), et une seconde partition de 3 Go en NTFS (n°7)**  
`sudo fdisk /dev/sdb` pour accéder  à la création de partition.  
Ensuite on crée des partitions avec `command : n :`  (1 puis 2), de taille +2G puis +3G  
On change le type de la 2nde partition avec `command : t:` et on choisit la 2nde partition puis on met le code hexa 7 pour passer en ntfs.  
`command : w :` pour enregistrer les modifs

**4. A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers.**  
**A l’aide de la commande mkfs, formatez vos deux partitions (pensez à consulter le manuel !)**  
On formate les partitions dans leur format choisi (ext4 et ntfs):  
`sudo mkfs.ext4 /dev/sdb1`
`sudo mkfs.ntfs /dev/sdb2`  

**5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?**  
**6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des UUID en raison de l’impossibilité d’effectuer des copier-coller)**  
**7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration**  
**8. Montez votre clé USB dans la VM**  
**9. Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer les Additions invité de VirtualBox**  

