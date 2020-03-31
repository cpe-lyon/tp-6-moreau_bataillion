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
- Installer les additions invités:  
Ajouter l'image iso des additions invités précédemment téléchargée au disque contrôleur IDE (confi, stockage.  
En root faire:
```
mkdir /mnt/cdrom
mount /dev/cdrom /mnt/cdrom
cd /mnt/cdrom
ls
```
Pour /dev/cdrom, faire tab pour trouver le nom complet du cdrom à disposition (cdrom, cdrom2...). Avec le résultat de ls, trouver le nom du script pour installer les additions invités. Puis, le lancer, toujours en root:
`./VBoxLinuxAdditions.run`
Redémarrer la VM avec reboot.

- Mettre en place le dossier partagé
Sous windows, créer un dossier shared_folder.  
VM éteinte, dans les config de la VM, onglet Dossiers partagés, créer un nouveau dossier partagé dont le chemin est celui de shared_folder. Cocher configuration permanente.  
Allumer la VM, créer un dossier point de montage /shared_folder_VM. Se placer en `sudo su`, puis taper:  
`mount -t vboxsf shared_folder /shared_folder_VM`  
Pour que le dossier soit monté automatiquement à l'allumage de la VM, ajouter dans /etc/fstab la ligne:  
shared_folder   /shared_folder_VM   vboxsf   defaults  0   0  

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
Redémarrer la VM : la fenêtre a changé de taille !  


**6. On va à présent ajouter un fond d’écran. Il existe un paquet en proposant quelques uns: grub2-splashimages (après installation, celles-ci sont disponibles dans /usr/share/images/grub).**  
`sudo apt install grub2-splashimages`pour installer le paquet d'image.
Ensuite, dans le fichier /etc/default/grub, on ajoute la ligne GRUB_BACKGROUND=/usr/share/images/grub/Apollo_17_The_Last_Moon_Shot_Edit1.tga  
Faire `update-grub` pour prendre en compte les modifs.  
Redémarrer la VM : le menu GRUB a un fond d'écran!  

**7. Il est également possible de configurer des thèmes. On en trouve quelques uns dans les dépôts (grub2-themes-*). 
Installez-en un.**  
`apt install grub2-themes-*` pour télécharger les thèmes.  
Dans /etc/default/grub, ajouter: GRUB_THEME=”boot/grub/themes/ubuntu-mate/theme.txt”  

**8. Ajoutez une entrée permettant d’arrêter la machine, et une autre permettant de la redémarrer.**  
On ajoute ces lignes au fichier /etc/grub.d/40_custom :  
```
menuentry 'Arrêt du système' { 
	halt 
} 
menuentry 'Redémarrage du système' { 
	reboot 
}
```
https://doc.ubuntu-fr.org/tutoriel/grub2_parametrage_manuel#menu_arret_du_systeme

**9. Configurer GRUB pour que le clavier soit en français**  
```
sudo mkdir /boot/grub/layouts
sudo grub-kbdcomp -o /boot/grub/layouts/fr.gkb fr
```
Ajouter `GRUB_TERMINAL_INPUT=at_keyboard` au fichier /etc/default/grub
Ajouter à /etc/grub.d/40_custom :
```
# Clavier fr
insmod keylayouts
keymap fr
```
https://doc.ubuntu-fr.org/tutoriel/grub2_parametrage_manuel#clavier_francais

## Exercice 3. Noyau 

**Dans cet exercice, on va créer et installer un module pour le noyau.**  

**1. Commencez par installer le paquet build-essential, qui contient tous les outils nécessaires (compilateurs, bibliothèques) à la compilation de programmes en C (entre autres).**  
`sudo apt install build-essential`  

**2. Créez un fichier hello.c contenant le code suivant :**  
```
1 #include <linux/module.h> 
2 #include <linux/kernel.h> 
3 
4 MODULE_LICENSE("GPL"); 
5 MODULE_AUTHOR("John Doe"); 
6 MODULE_DESCRIPTION("Module hello world"); 
7 MODULE_VERSION("Version 1.00"); 
8 
9 int init_module(void) 
10 { 
11 	printk(KERN_INFO "[Hello world] - La fonction init_module() est appelée.\n"); 
12 	return 0; 
13 } 
14 
15 void cleanup_module(void) 
16 { 
17 	printk(KERN_INFO "[Hello world] - La fonction cleanup_module() est appelée.\n"); 
18 } 
```

**3. Créez également un fichier Makefile :**  
```
1 obj-m += hello.o 
2 
3 all: 
4 make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules 
5 
6 clean: 
7 make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean 
8 
9 install: 
10 cp ./hello.ko /lib/modules/$(shell uname -r)/kernel/drivers/misc
```
**Les lignes 4, 7 et 10 doivent commencer par une tabulation.**  

**4. Compilez le module à l’aide de la commande `make`, puis installez-le à l’aide de la commande `make install`.  
Le module est installé dans le dossier spécifié à la ligne 10.**  
```
make		//compilation de hello.c
make install	//installation de hello.c
```

**5. Chargez le module; vérifiez dans le journal du noyau que le message ”La fonction init_module() est appelée” a bien été inscrit, synonyme que le module a été chargé; confirmez avec la commande lsmod.**
```
modprobe -a hello		//charger le module avec ses dépendances
tail /var/log/syslog		//Regarder le journal du noyau : on voit que init_module() a bien été déclenché
lsmod | grep hello   		//lsmod liste les modules chargés dans la VM, avec grep on ne sélectionne que le module hello. Le  				    module ayant chargé, il est bien présent dans la liste
```

**6. Utilisez la commande `modinfo` pour obtenir des informations sur le module hello.ko; vous devriez notamment voir les informations figurant dans le fichier C.**  
`modinfo hello` donne des infos sur le module : filename, version, description, author, license, srcversion, depends, retpoline, name, vermagic.

**7. Déchargez le module; vérifiez dans le journal du noyau que le message ”La fonction cleanup_module() est appelée” a bien été inscrit, synonyme que le module a été déchargé; confirmez avec la commande lsmod.**  
```
modprobe -r hello		//décharger le module avec ses dépendances
tail /var/log/syslog		//Regarder le journal du noyau : on voit que cleanup_module() a bien été déclenché
lsmod | grep hello   		//lsmod liste les modules chargés dans la VM, avec grep on ne sélectionne que le module hello. Le module ayant été déchargé, il est absent de la liste
```

**8. Pour que le module soit chargé automatiquement au démarrage du système, il faut l’inscrire dans le fichier /etc/modules. Essayez, et vérifiez avec la commande lsmod après redémarrage de la machine.**  
Dans /etc/modules, ajouter : hello. Redémarrer la VM avec `reboot`. Puis vérifier avec `lsmod | grep hello` que le module hello a bien été rechargé au démarrage automatiquement.


## Exercice 4: Exécution de commandes en différé : at et cron  

On utilise at pour un évènement ponctuel et cron pour plusieurs évènements répétitifs (toutes les heures, tous les 5 jours, tous les vendredi, tous les 8 mois, tous les ans...).  

**1. Programmez une tâche qui aﬀiche un rappel pour la réunion qui aura lieu dans 3 minutes. Vérifiez entre temps que la tâche est bien programmée.**  
`echo ‘echo Réunion Rappel’ | at now +3 minutes` : met en place le rappel  
`atq` : affiche toutes les taches programmées dont celle précédente.  
`atrm 1` : remove le job 1 (tâche 1 a effectuer retirée de la liste)  

**2. Est-ce que le message s’est aﬀiché? Si la réponse est non, essayez de trouver la cause du problème (par exemple en vous aidant des logs, du manuel...)**  
Ne marche pas selon le journal de log : `exec failed for mail command : No such file or directory`. La commande `at` n'envoie pas ses sorties dans le terminal mais par mail à l'utilisateur. Ainsi, l'utilisateur pourra consulter ses mails à n'importe quel moment et ne pas râter l'envoie de la notification par at car il n'était pas connecté à ce moment là. La commande s'exécute même si l'utilisateur n'est pas loggué.  
On installe donc un paquet de gestion de mails : `sudo apt install mailutils`. On peut consulter ses mails avec `mail`.On navigue ensuite grâce aux numéros des mails.  
Nous cherchons maintenant à ne plus recevoir la notification par mail, mais directement sur le terminal. Il faut d'abord trouver le chemin de notre terminal actuel pour que le résultat de la commande arrive au bon endroit. Avec `tty`, on obtient `/dev/tty1`, c'est le nom de notre terminal. On redirige donc la commande vers ce terminal:  
`echo ‘echo Réunion Rappel>/dev/tty1’ | at now +3 minutes`.  
Il faut préter attention à la place des côtes! Attention : ce qui est placé dans des guillemets doubles est **interprété**.

**3. Pour tester le fonctionnement de cron, commencez par programmer l’exécution d’une tâche simple, l’aﬀichage de “Il faut réviser pour l’examen!”, toutes les 3 minutes.**  

**4. Programmez l’exécution d’une commande tous les jours, toute l’année, tous les quarts d’heure**  

**5. Programmez l’exécution d’une commande toutes les cinq minutes à partir de 2 (2, 7, 12, etc.) à 18 heures les 1er et 15 du mois :**  

**6. Programmez l’exécution d’une commande du lundi au vendredi à 17 heures**  

**7. Modifiez votre crontab pour que les messages ne soient plus envoyés par mail, mais redirigés dans un fichier de log situé dans votre dossier personnel**  

**8. Videz votre crontab**  


## Exercice 5: Surveillance de l’activité du système  

**1. Dans la console virtuelle tty1, lancez la commande htop, puis tapez la commande w dans tty2. Qu’aﬀiche cette commande?**  
`htop` permet de lister et gérer les processus en cours d’exécution.  
Pour passer sous tty2 on fait : CTRL + ALT + F2.  
La commande `w` affiche les différents terminaux tty ouverts (tty1 et tty2) ainsi que l'heure de login sur le terminal et l'utilisateur responsable du terminal. Il y a également plusieurs infos à propos du terminal (IDLE, JCPU, PCPU, WHAT).  

**2. Comment aﬀicher l’historique des dernières connexions à la machine?**  
La commande `last` permet d'afficher la liste des connexions utilisateur sur une machine.  
Si on fait `last tty2`, on voit la liste des connexions à tty2. Ici il n’y a qu’une connexion et on peut voir qu’on y est toujours connecté.  

**3. Quelle commande permet d’obtenir la version du noyau?**  
`uname -r` : 5.3.0  

**4. Comment récupérer toutes les informations sur le processeur, au format JSON?**  
`lshw` est un programme permettant d'extraire des informations détaillées de la configuration matérielle de la machine.  
`sudo lshw -json` : indique qu’on veut la sortie au format json  
`sudo lshw -json >>info_proc.json` : permet de stocker la sortie dans un fichier  

**5. Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl?**  
`journalctl | grep "Linux version" | tail -10 | cut -d" "  -f1-3`  
`journalctl` permet d’avoir accès au journal des log. Dans ces log, on a notamment des logs qui concernent l’allumage de la machine. A chaque allumage, la première ligne est constituée d’un rappel de la distribution linux de la machine. Il y a donc systématiquement marqué "Linux version". On fait donc un `grep` pour ne conserver que ces lignes qui correspondent à un allumage de machine.  
De façon arbitraire, on décide de ne regarder que les 10 dernières connexions à la machine avec `tail`. La ligne étant un peu longue et inutile à conserver pour cet exercice, on fait un `cut` aux espaces et on ne conserve que les 3 premières colonnes qui correspondent à la date et à l’heure du log.

Ou tout simplement après quelques recherches supplémentaires : `journalctl --list-boots | tail -10`

**6. Comment aﬀicher tout ce qu’il s’est passé sur la machine lors de l’avant-dernier boot?**  
`journalctl -S -2d -U -1d`  
Le -S signifie Since et le U Until. On remonte dans le temps avec -2d: -2day : il y a deux jours. On sélectionne les log entre il y a deux jours et il y a un jour, c'est à dire avant-hier.  

**7. Faites en sortes que lors d’une connexion à la machine, les utilisateurs soient prévenus par un message à l’écran d’une maintenance le 26 mars à minuit.**  
Editer le fichier /etc/motd (le créer si besoin): `sudo nano /etc/motd`. Ecrire : Maintenance le 26/03 à minuit!
motd = Message Of The Day.
Après un `sudo reboot`, le message s'inscrit bien lors d'une connexion.

**8. Ecrivez un script bash qui permet de calculer le k-ième nombre de Fibonacci : Fk = Fk−1 + 
Fk−2, avec F0 = F1 = 1.  
Lancez le calcul de F100 puis lancez la commande tload depuis un autre terminal virtuel. Que constatez-vous?   Interrompez ensuite le calcul avec CTRL+C et observez la conséquence sur l’aﬀichage de tload.**  
Dans /Documents/Fibonacci.sh:  
```
F0=0;
F1=1;
for i in `seq 1 100000`; 
	do 
		F2=$(($F1+$F0));
		F0=$F1;
		F1=$F2; 
	done
echo $F2;
```
La commande `tload` permet d'obtenir une représentation graphique de la charge moyenne du système.  
On compile avec chmod u+x fibonacci.sh. On lance le script avec ./fibonacci.sh.  
On passe dans le terminal 2 avec CTRL + ALT +F2 et on lance `tload`.  
On repasse dans le terminal 1 (CTRL + ALT + F1) et on arrête le processus avec CTRL + C  
On repasse dans le terminal 2.  
On peut observer que tant que le processus était actif, la charge moyenne du système augmentait très vite. Lorsqu'on a arrêté le processus, la charge est redescendue graduellement jusqu'à atteindre 0.  

## Exercice 6. Interception de signaux  

**La commande interne trap permet de redéfinir des gestionnaires pour les signaux reçus par un processus. Un cas d’utilisation typique est la suppression des fichiers temporaires créés par un script lorsque celui-ci est interrompu.**  

**1. Commencez par écrire un script qui recopie dans un fichier tmp.txt chaque ligne saisie au clavier par l’utilisateur**  
```
#! /bin/bash
read -p 'Saisissez du texte:' texte
echo $texte>>./tmp
```
Pour compiler : `chmod u+x recopie.sh`. Pour lancer le script : `./recopie.sh`. Tous le etxte tappé par l'utlisateur sera stocké dans le fichier tmp.  

**2. Lancez votre script et appuyez sur CTRL+Z. Que se passe-t-il? Comment faire pour que le script poursuive son exécution?**  
Après un CTRL+Z, on obtient la ligne suivante : `[1]+Stopped	./recopie.sh`. Le CTRL+Z met en pause le processus courant (recopie.sh) et redonne accès au terminal. Il détruit les sorties en attente : les lignes rentrées avant ne sont pas recopiées dans tmp et ne sont pas sauvegardées.  
Le `[1]` correspond au numero du job stoppé. Si on veut voir tous les jobs stoppés par CTRL+Z : faire `jobs`.  
Pour reprendre le processus, il y a deux solutions :  
- Faire `bg %1` : cela permet à l’exécution de continuer en arrière-plan (BackGround). Le %1 correspond au numero du processus stoppé
- Faire `fg %1` : cela permet au processus de reprendre en avant-plan (ForeGround). On perd de nouveau la main sur le terminal et tout ce qu’on écrit sera recopié dans tmp. C’est la solution que l’on choisit.  

**3. Toujours pendant l’exécution du script, appuyez sur CTRL+C. Que se passe-t-il?**  
CTRL+C : arrête le processus courant comme un kill. Aucune ligne d’information. Impossible de reprendre le processus.

**4. Modifiez votre script pour redéfinir les actions à effectuer quand le script reçoit les signaux SIGTSTP (= CTRL+Z) et SIGINT (= CTRL+C) : dans le premier cas, il doit aﬀicher ”Impossible de me placer en arrière-plan”, et dans le second cas, il doit aﬀicher ”OK, je fais un peu de ménage avant” avant de supprimer le fichier temporaire et terminer le script.**  
On fait d'abord kill -l pour obtenir les chiffres correspondant aux signaux. CTRL+Z=SIGTSP=20 ; CTRL+C=SIGINT=2.  
```
#! /bin/bash
trap "echo 'OK, je fais un peu de ménage avant' ; rm ./tmp ; exit" 2
trap "echo 'Impossible de me placer en arrière-plan'" 20
read -p 'Saisissez du texte:' texte
echo $texte>>./tmp
```
exit permet de sortir du processus.  
La commande trap s'écrit sous la forme suivante : trap "commande(s)" Numéro(s)_signal.  

**5. Testez le nouveau comportement de votre script en utilisant d’une part les raccourcis clavier, d’autre part la commande kill**  
Le script fonctionne avec les nouveaux raccourcis clavier. On ne peut pas utiliser kill car n'est que recopié dans le fichier tmp et pas considéré comme une commande.  
La commande `kill pid` tue le processus dont l'identifiant a été sélectionné. Pour obtenir l'identifiant pid d'un processus : faire `pidof nom_processus`. `kill` a plusieurs options (dont CTRL+Z et CTRL+C). On peut choisir la façon de l'exécuter avec la forme : `kill -s num_signal`.  

**6. Relancez votre script et faites immédiatement un CTRL+C : vous obtenez un message d’erreur vous indiquant que le fichier tmp.txt n’existe pas. A l’aide de la commande interne test, corrigez votre script pour que ce message n’apparaisse plus.**  
```
#! /bin/bash
if [ -e ./tmp ]; then
trap "echo 'OK, je fais un peu de ménage avant' ; rm ./tmp ; exit" 2
fi
trap "echo 'Impossible de me placer en arrière-plan'" 20
read -p 'Saisissez du texte:' texte
echo $texte>>./tmp
```
La commande interne test est symbolisée ici par les crochets du if. Le -e vérifie que le fichier tmp existe bien avant de pouvoir mettre en place la redirection de signaux grâce à trap.  


## Exercice 7. Sauvegardes, tar  

**1. Placez-vous dans votre dossier personnel, puis créez deux archives de ce dossier : une archive1.tar en indiquant * comme argument, et une archive2.tar en indiquant . comme argument. Comparez les deux archives. Que constatez-vous? Expliquez.**  
```
tar -cvf archive1.tar *
tar -cvf archive2.tar .
```
- -c : crée l'archive  
- -v : active le mode verbeux qui affiche tout ce que fait la compression  
- -f : utilise l'argument donné en paramètre comme élément à archiver  

L'archive avec * comme argument fait une copie de tous les fichiers et dossiers présents dans notre dossier personnel. Au contraire, l'archive avec . comme argument fait une copie de l'ensemble des fichiers et dossiers (même ceux cachés, ceux de configuration...) accessible depuis notre dossier personnel.  

**2. Quel est le problème avec la commande `tar cvf archive.tar * .*`?**  
Elle copie plusieurs fois les mêmes éléments. * et .* sont considérés comme deux éléments différents à archiver.

**3. Créez un dossier test contenant trois fichier tata, toto et tutu puis retirez les droits d’exécution sur ce dossier. Créez une archive de ce dossier. Que constatez-vous? Expliquez.**  
```
mkdir test
touch ./test/tutu ./test/tata ./test/toto
chmod a-x ./test
tar -cvf archive_test.tar ./test
```
On ne peut pas créer cette archive. Le droit d'exécution pour un dossier correspond au droit de traverser le répertoire, c'est à dire le droit d'accéder au contenu du répertoire. La commande tar ne peut donc pas s'exécuter correctement : elle ne peut accéder au contenu du dossier test.


