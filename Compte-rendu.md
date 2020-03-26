# Administration système - TP TP 6 - Gestion des disques, Boot, Gestion des logs


## Auteurs :

*BIARD Gauthier*

*Le 28/02/2020*


***

## Exercice 1. Disques et partitions

**1. Dans l'interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement alloués ; puis démarrez la VM**

&nbsp;

**2. Vérifiez que ce nouveau disque dur est bien détecté par le système**

On exécute la commande :

```bash
ll /dev/sd*
```
On observe alors, avec l'apparition du fichier sdb, que le disque dur a bien été créé.

> On peut vérifier la taille du disque durà l'aide de la commande *lsblk*. A ce moment là,on remarque que *sdb* a bien une taille de 5Go.

&nbsp;

**3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83), et une seconde partition de 3 Go en NTFS (n°7)**

Afin de partitionner le disque, nous exécutons la commande suivante :
```bash
fdisk /dev/sdb
```
Ensuite, nous exécutons les commandes suivantes à la suite :
```bash
n
p #pour primary
1 #en premiere section
2048 #en première section
(2 000 000 000 / 512) + 2048) #avec 512 la taille d'une section en fin de première section
```
Ainsi,nous avons créer la première section.

Afin de créer la deuxième section, il suffit de répéter l'opération.

> Après la création on tape la commande `t` puis le code haxedecimal "7" pour avec la partition en NTFS.

&nbsp;

**4. A ce stade, les partitions ont été créées, mais elles n'ont pas été formatées avec leur système de fichiers. A l'aide de la commande mkfs, formatez vos deux partitions ( pensez à consulter le manuel !)**

Nous formatons la partition 1 au format ext2 :
```bash
sudo mkfs.ext4 /dev/sdb1
```

Nous formatons la partition 2 au format NTFS :
```bash
sudo mkfs.ntfs -f /dev/sdb2
```

&nbsp;

**5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?**

Le disque dur n'est pas monté donc la commande `df -T`ne fonctionne pas.

&nbsp;

**6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des UUID en raison de l'impossibilité d'effectuer des copier-coller)**

Nous créons le dossier *data* et *win*, puis on modifie le fichier fstab :
```bash
sudo mkdir /data
sudo mkdir /win
sudo nano /etc/fstab

/dev/sdb1   /data   ext4    defaults 0   0
/dev/sdb2   /win    ntfs    defaults 0   0
```

&nbsp;

**7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration**

&nbsp;

**8. Montez votre clé USB dans la VM**

```bash
ll /dev/sd*

mkdir /media/usb
mount /dev/sdc1 /media/usb
```

&nbsp;

**9. Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d'installer les Additions invité de VirtualBox**

Afin de créer le fichier partagé via VirtualBox, nous allons dans :
   - option
   - dossier partagé
   - nom du dossier : Shared_folder  
   - [ ]Lecture seule  
   - [x]Montage automatique  
   - [x]Montage permanent  

```bash
sudo mount /dev/cdrom /media/cdrom
sudo /media/cdrom/./VBoxLinuxAdditions.run
sudo reboot

mkdir ~/shared_folder
sudo nano /etc/fstab
> Shared_folder    /home/server/shared_folder    vboxsf  comment=systemd.automount     0       0
mount -a
sudo reboot
```

&nbsp;

## Exercice 2. Personnalisation de GRUB

**1. Commencez par changer l'extension du fichier /etc/default/grub.d/50-curtin-settings.cfg s'il est présent dans votre environnement (vous pouvez aussi commenter son contenu).**

Le fichier n'est pas présent.

&nbsp;

**2. Modifiez le fichier /etc/default/grub pour que le menu de GRUB s'affiche pendant 10 secondes ; passé ce délai, le premier OS du menu doit être lancé automatiquement.**

On remplace :
```bash
GRUB_TIMEOUT=0
```
par :
```bash
GRUB_TIMEOUT=10
```

&nbsp;

**3. Lancez la commande update-grub**

&nbsp;

**4. Redémarrez votre VM pour valider que les changements ont bien été pris en compte**

&nbsp;

**5. On va augmenter la résolution de GRUB et de notre VM. Cherchez sur Internet le ou les paramètres à rajouter au fichier grub.**

Nous exécutons la commande :
```bash
sudo nano /etc/default/grub 
```
Il faut décommenter la ligne :
```bash
GRUB_GFXMODE="1600x1200x32"
```
Il faut rajouter les deux lignes suivantes :
```bash
export GRUB_COLOR_NORMAL="light-gray/black"
export GRUB_COLOR_HIGHLIGHT="green/black"
```
&nbsp;

**6. On va à présent ajouter un fond d'écran. Il existe un paquet en proposant quelques uns : grub2-splash-images (après installation, celles-ci sont disponibles dans /usr/share/images/grub).**

```bash
sudo apt install grub2-splashimages
```
> On rerouve une liste d'images en extention .tga avec la commande : `ls /usr/share/images/grub`.

&nbsp;

**7. Il est également possible de configurer des thèmes. On en trouve quelques uns dans les dépôts (grub2-themes-*). Installez-en un.**

```bash
sudo nano /etc/default/grub 
export GRUB_THEME="/boot/grub/themes/ubuntu-mate/theme.txt"
```

&nbsp;

**8. Ajoutez une entrée permettant d'arrêter la machine, et une autre permettant de la redémarrer.**

&nbsp;

**9. Configurer GRUB pour que le clavier soit en français**

```bash
sudo nano /etc/default/grub 

export GRUB_TERMINAL_INPUT=at_keyboard 
export LANG=fr_FR

sudo update-grub
```

&nbsp;

## Exercice 3. Noyau

**1. Commencez par installer le paquet build-essential, qui contient tous les outils nécessaires (compilateurs, bibliothèques) à la compilation de programmes en C (entre autres).**

&nbsp;

**2. Créez un fichier hello.c contenant le code suivant :**
```bash
#include <linux/module.h>
#include <linux/kernel.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("John Doe");
MODULE_DESCRIPTION("Module hello world");
MODULE_VERSION("Version 1.00");

int init_module(void)
{
printk(KERN_INFO "[Hello world] - La fonction init_module() est appelée.\n");
return 0;
}

void cleanup_module(void)
{
printk(KERN_INFO "[Hello world] - La fonction cleanup_module() est appelée.\n");
}
```

&nbsp;

**3. Créez également un fichier Makefile :**
```bash
obj-m += hello.o

all:
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

install:
cp ./hello.ko /lib/modules/$(shell uname -r)/kernel/drivers/misc
```

**4. Compilez le module à l'aide de la commande make, puis installez-le à l'aide de la commande make install**

&nbsp;

**5. Chargez le module ; vérifiez dans le journal du noyau que le message "La fonction init_module() est appelée" a bien été inscrit, synonyme que le module a été chargé ; confirmez avec la commande lsmod.**

&nbsp;

**6. Utilisez la commande modinfo pour obtenir des informations sur le module hello.ko ; vous devriez notamment voir les informations figurant dans le fichier C.**

&nbsp;

**7. Déchargez le module ; vérifiez dans le journal du noyau que le message "La fonction cleanup_module() est appelée" a bien été inscrit, synonyme que le module a été déchargé ; confirmez avec la commande lsmod.**

&nbsp;

**8. Pour que le module soit chargé automatiquement au démarrage du système, il faut l'inscrire dans le fichier /etc/modules. Essayez, et vérifiez avec la commande lsmod après redémarrage de la machine.**