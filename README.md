# Roger-Skyline-1
Introduction à l'administration système

Installation et configuration de la VM

ram de 1024 octets
disque dur de 8go
disque dur à taille fixe
- création de l’utilisateur non root aetche et de son mot de passe
partition primaire de 4.2go dans /
partition logique dans /home 
édit du fichier /etc/apt/sources.list pour que le gestionnaire de paquets apt fonctionne correctement
apt-get update et apt-get upgrade pour maintenir à jour le gestionnaire de paquets
apt-get install sudo vim ssh ufw fail2ban apache2 portsentry mailutils pour installer les dépendances nécessaires au projet

Partie Réseau

	Droits utilisateurs
ajout de l’utilisateur aetche au groupe sudo avec la commande adduser aetche sudo qui permet à l’utilisateur d’executer n’importe quelle commande
en mode root, il est également possible d’éditer le fichier /etc/sudoers avec la commande visuel. On rajoute l’utilisateur supplémentaire dans la partie #User privilege specification

	Configurer une adresse IP fixe
On définit une adresse IP fixe en éditant le fichier de configuration du réseau
sudo vim /etc/network/interfaces
on redémarre le network service pour rendre les changements effectifs : sudo service networking restart
on peut utiliser la commande ip addr ou /sbin/ifconfig pour vérifier que les changements ont bien été pris en compte 
	
	Service SSH
editer le fichier /etc/ssh/sshd_config en mettant le port à 63000 (par exemple)
pour se connecter : ssh aetche@192.168.1.15 -p 63000
pour créer un accès avec des public keys, utiliser la commande ssh-keygen -t rsa (génère une clé de type rsa)
deux fichiers sont crées dans .ssh/ —> id_rsa est le fichier qui contient la clé privée qu’il ne faut pas dévoiler —> id_rsa.pub contient la clé publique qui sera mise sur les serveurs
il faut maintenant ajouter cette clé sur les serveurs avec la commande ssh-copy-id -i ~/.ssh/id_rsa.pub aetchego@192.168.1.15 -p 63000
ensuite on peut vérifier que la clé a été ajoutée dans le fichier .ssh/authorized_keys
éditer le fichier /etc/ssh/sshd_config en changeant la ligne PermitRootLogin no

	Configuration du firewall
pour savoir si ufw est actif : sudo ufw status
s’il ne l’est pas : sudo enable ufw
pour voir les règles de configuration du pare-feu : sudo ufw status verbose (par défaults, toutes les connections entrantes sont refusées, et les sortantes sont autorisées)
Pour ajouter des règles de pare-feu : sudo ufw allow [port]
on ajoute donc 63000 (port SSH), 80 (HTTP) et 443 (HTTPS)

	Protection contre les DOS
utilisation de fail2ban = fail2ban est une application qui analyse les logs de divers services (SSH, Apache, FTP…) en cherchant des correspondances entre des motifs définis dans ses filtres et les entrées des logs. Lorsqu'une correspondance est trouvée une ou plusieurs actions sont exécutées. Typiquement, fail2ban cherche des tentatives répétées de connexions infructueuses dans les fichiers journaux et procède à un bannissement en ajoutant une règle au pare-feu iptables pour bannir l'adresse IP de la source.
copier coller le fichier de configuration et l’éditer de cette manière https://www.supinfo.com/articles/single/2660-proteger-votre-vps-apache-avec-fail2ban
tail -f /var/log/fail2ban.log pour regarder les logs

	Protection contre les scans sur les ports ouverts
Votre serveur peut faire l'objet de différents scans de ports afin d'identifier, par exemple, les services qui sont en place sur votre serveur ou encore le système d’exploitation installé (ce que permet Nmap, par exemple). Ces informations pourraient être ensuite exploitées par une personne malveillante afin d’atteindre à l’intégrité de votre serveur. Pour se protéger contre ces pratiques, vous pouvez mettre en place portsentry qui bloquera les adresses IP des connexions à l'origine de ces scans.
stopper portsentry durant la configuration systemctl stop portsentry
ajouter son adresse IP au fichier /etc/portsentry/portsentry.ignore.static
modification de /etc/default/portsentry
Si vous choisissez les modes atcp et audp ("a" signifie avancé) dans /etc/defaults/portsentry, inutile de préciser les ports, Portsentry va vérifier les ports utilisés et automatiquement "lier" les ports disponibles. C'est l’option la plus efficace. Donc avec cette option, portsentry établit une liste des ports d'écoute, TCP et UDP, et bloque l'hôte se connectant sur ​​ces ports sauf s'il est présent dans le fichier portsentry.ignore configuré auparavant.
modification de /etc/portsentry/portsentry.conf afin de bloquer les scans UDP/TCP
décommenter la ligne de commande qui utilise iptable pour bloquer les IP
redémarrer portsentry sudo service portsentry restart (vérifier avec tail -n 5 /var/log/syslog)
TEST = nmap -v 192.168.1.15 puis regarder sudo grep « attackalert » —binary-files=text /var/log/syslog

	Arrêter les services dont nous n’avons pas besoin 
voir les services = sudo systemctl soit sudo service —status-all (+ sils sont actifs sinon -)
sudo systemctl disable nom.service (pour empêcher le service de démarrer automatiquement)
sudo systemctl stop nom.service (pour arrêter le service)
