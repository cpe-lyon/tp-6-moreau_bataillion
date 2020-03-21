BATAILLION Alice  
MOREAU Marianne  
4 ETI 

# TP 6 - Gestion des disques, Boot, Gestion des logs  

## Exercice 1. Disques et partitions  

**1. Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement alloués ; puis démarrez la VM**  
Dans config, stockage, rajouter un disque dur sata.  
	
**2. Vérifiez que ce nouveau disque dur est bien détecté par le système**  
`ls /dev/sd*` pour voir les noms des disques attachés. Le nouveau s’appelle sdb (les disque s’appellent sd puis une lettre dans l’ordre de création)  

**3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83), et une seconde partition de 3 Go en NTFS (n°7)**  
`sudo fdisk /dev/sdb` pour accéder  à la création de partition.  
Ensuite on crée des partitions avec `command : n `:  (1 puis 2), de taille +2G puis +3G  
On change le type de la 2nde partition avec `command : t`: et on choisit la 2nde partition puis on met le code hexa 7 pour passer en ntfs.  
`command : w `: pour enregistrer les modifs

**4. A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers.**  
**A l’aide de la commande mkfs, formatez vos deux partitions (pensez à consulter le manuel !)**  
On formate les partitions dans leur format choisi (ext4 et ntfs):  
`sudo mkfs.ext4 /dev/sdb1`  
`sudo mkfs.ntfs /dev/sdb2`  

**5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, 
ne fonctionne-telle pas sur notre disque ?**  
`df -T` n’affiche pas sdb1 ni sdb2.  
Hypothèse : les partitions ne seront pas prises en compte jusqu’à ce qu’elles soient montées.  

**6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des UUID en raison de l’impossibilité d’effectuer des copier-coller)**  
Aller dans `/etc/fstab` et rajouter une ligne par partition :  
1ère colonne: nom de la partition à monter  
2ème colonne: nom du point de montage  
3ème colonne: système de fichier utilisé (ext4, ntfs...)  
Attention: il faut créer /win et /data.  

**7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration**  
Pour forcer la prise en compte des modification de la fstab : `mount -a`.  

**8. Montez votre clé USB dans la VM**  
Dans config, usb, on rajoute un filtre usb vide, ce qui va permettre de lire tous les périphériques usb branchés.  
Avec `sudo fdisk -l` on constate qu’il s’est bien rajouté une clef (sdc1) lorsqu’on branche une clef à la machine hôte.  
On va créer un point de montage pour la clef. On crée le dossier `/media/usb` comme dossier de montage et on monte la clef grâce à `mount /dev/sdc1 /media/usb`.


**9. Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer les Additions invité de VirtualBox**  



## Exercice 2. Personnalisation de GRUB  

**GRUB est considérablement paramétrable : résolution, langue, fond d’écran, thème, disposition du clavier...  
GRUB se configure via un fichier de paramètres (/etc/default/grub), mais aussi par des scripts situés dans /etc/grub.d; ces scripts commencent tous par un numéro et sont traités dans l’ordre.  
Evidemment, seuls les scripts exécutables sont pris en compte.  
Sous Ubuntu Server, GRUB prend aussi en compte les fichiers d’extension .cfg présents dans /etc/default/grub.d. En particulier, sur les versions récentes, le fichier de configuration 50-curtin-settings.cfg donne à la variable GRUB_TERMINAL la valeur console, ce qui désactive tous les paramètres liés aux fonds d’écran, thèmes, certaines résolutions, etc.**  

**1. Commencez par changer l’extension du fichier /etc/default/grub.d/50-curtin-settings.cfg s’il est présent dans votre environnement (vous pouvez aussi commenter son contenu).**  
On n'a pas le fichier /etc/default/grub.d/50-curtin-settings.cfg, donc on ne le modifie pas. Ce fichier est utilisé au démarrage par GRUB pour interdire tout contenu graphique dans la console. Cela empêcherait par exemple de mettre en place un fond d'écran. Pour que le fichier ne soit pas pris en considération par GRUB il suffit de changer son extension : il n'est alors pas exécuté au démarrage.

**2. Modifiez le fichier /etc/default/grub pour que le menu de GRUB s’aﬀiche pendant 10 secondes; passé ce délai, le premier OS du menu doit être lancé automatiquement.**  
Dans /etc/default/grub : passer GRUB_TIMEOUT_STYLE=menu et GRUB_TIMEOUT=10. Le premier indique que l'on doit afficher le menu de GRUB (choix de lancement d'une distribution), le second indique le temps alloué à l'utilisateur pour faire son choix avant que l'OS par défaut ne soit lancé.

**3. Lancez la commande `update-grub`.  
Cette commande fait appel au script grub-mkconfig qui construit le fichier de configuration ”final” de GRUB (/boot/grub/grub.cfg) à partir du fichier de paramètres et des scripts.**  

**4. Redémarrez votre VM pour valider que les changements ont bien été pris en compte.  
Pensez à lancer la commande `update-grub` après chaque modification de la configuration de GRUB!**  
Après redémarrage de la VM, on a bien accès au menu de choix d'OS pendant 10s.

**5. On va augmenter la résolution de GRUB et de notre VM. Cherchez sur Internet le ou les paramètres à rajouter au fichier grub.**  
Au démarrage de la VM, appuyer sur c lors de l'affichage du menu GRUB pour accéder à la ligne de commande grub. Faire `vbeinfo` pour découvrir la plus grande résolution possible pour notre machine (il s'agit de la dernière ligne: 1152x864x32). Faire echap pour quitter cet invite de commande.  
Dans /etc/default/grub : décommenter la ligne GRUB_GFXMODE (résolution du menu grub) et mettre la résolution précédemment trouvée. 
Puis, ajouter la ligne GRUB_GFXPAYLOAD_LINUX (résolution VM linux) avec la même résolution.  
Faire `update-grub` pour prendre en compte les modifs.  
Redémarrer VM : la fenêtre a changé de taille !  


**6. On va à présent ajouter un fond d’écran. Il existe un paquet en proposant quelques uns: grub2-splashimages (après installation, celles-ci sont disponibles dans /usr/share/images/grub).**  






