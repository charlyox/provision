# Configuration serveur L'atelier 62

# A partir d'une machine debian 12 bookworm
sudo apt update -y
sudo apt upgrade -y

# Install the necessary dependency packages for HTTPS transport and certificate management.
sudo apt -y install lsb-release apt-transport-https ca-certificates

# sury key
sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg

# SURY PPA repository information add to APT sources
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list

# update then
sudo apt update -y

# php versions available @apt
sudo apt policy php

#? sudo apt-get install software-properties-common
sudo apt install -y php8.3
sudo apt install php8.3-fpm php8.3-dev php8.3-xml php8.3-zip unzip php8.3-curl php8.3-gd php8.3-mbstring php8.3-xdebug php8.3-ssh2 php8.3-mysql -y

# php version
php -v

#php modules
php -m

# Enable PHP-FPM to automatically start at boot.
sudo systemctl enable php8.3-fpm

# start service
sudo systemctl start php8.3-fpm

# Installation de apache
sudo apt install apache2 -y

# démarrage 
sudo systemctl start apache2

# paquets supplémentaires
sudo apt install imagemagick

# Création du dossier /var/cuisine/recette.domain.fr/public et /var/www/domain.fr/public.

sudo mkdir /var/cuisine/dyslexiefrance/public
sudo mkdir /var/www/public

# sudo nano /etc/apache2/apache2.conf

	Modifié ce bloc : (/public et AllowOverride)
	<Directory /var/www/public/>
		Options Indexes FollowSymLinks
		AllowOverride All
		Require all granted
	</Directory>

# sudo nano /etc/apache2/conf-available/security.conf
	#ServerTokens OS
	ServerTokens Prod

	#ServerSignature On
	ServerSignature Off

# Pas de modification des php.ini - on garde ceux par défaut pour l'instant
# Pas de modification non plus de /etc/php/7.4/fpm/pool.d/www.conf et 8.0 même s'il existe des différences.




# PRODUCTION sudo nano /etc/apache2/sites-available/latelier62.fr.conf

	<VirtualHost *:80>
		ServerName latelier62.fr
		DocumentRoot "/var/www/public"

		<Directory /var/www/public>
			Options +Indexes +Includes +FollowSymLinks +MultiViews
			AllowOverride All
			FallbackResource /index.php
			Require all granted
		</Directory>
		<FilesMatch ".php$">
			SetHandler "proxy:unix:/var/run/php/php8.0-fpm.sock|fcgi://localhost/"
	    	</FilesMatch>
	</VirtualHost>

# ajout utilsateur
sudo adduser gerg
tgi86er46

usermod -a -G $phpPoolUser www-data

# Copier le fichier /etc/php/8.3/fpm/pool.d/www.conf en user.conf
Modifier les occurences de www-data et changer par user

a2enmod rewrite

#sudo nano /etc/hosts ! TODO  Ajouter la correcte syntaxe cf le vrai fichier incluant le nom du serveur
	=> Ajouter la ligne :
	127.0.0.1 dyslexiefrance.latelier62.fr sainteloi
	127.0.0.1 latelier62.fr sainteloi

# Activer le module proxy_fcgi qui fait le lien entre php-fpm et apache
	sudo a2enmod proxy_fcgi
	sudo service php7.4-fpm restart
	sudo service apache2 restart

#Suppression du site par défaut puisqu'on va se mettre au même endroit (sinon il prend notre place et passe devant nous)
	sudo a2dissite 000-default.conf

# GANDI. Modifié l'adresse IP dans l'enregistrement A du serveur DNS. Ajouté également un sous-domaine (enregistrement A) "recette", pointant vers la même IP.


# MariaDB
	sudo apt install mariadb-server
	sudo apt-get install php8.0-mysql


# Activation du ssl
	sudo a2enmod ssl
		// Installation snap :
		sudo apt update
		sudo apt install snapd
	sudo snap install --classic certbot
	sudo certbot --apache
	# Normalement le renouvellement automatique est activé. A vérifier dans trois mois (5 décembre)
	sudo certbot renew --dry-run


	# Changement de config vhosts ?  Modifier /etc/apache2/sites-available/http-vhosts-le-ssl.conf et redémarrer service apache.

# Création de Base de données MariaDB et son utilisateur associé :
	MariaDB [(none)]> CREATE DATABASE dyslexia_db;
	MariaDB [(none)]> CREATE USER 'ron_davis'@'localhost' IDENTIFIED BY 'oEIj64RSEgyU+45';
	MariaDB [(none)]> GRANT ALL ON dyslexia_db.* TO 'ron_davis'@'localhost';
	MariaDB [(none)]> FLUSH PRIVILEGES;
	MariaDB [(none)]> QUIT;

	MariaDB [(none)]> CREATE DATABASE dyslexia_db_2;
	MariaDB [(none)]> CREATE USER 'dyslexic'@'localhost' IDENTIFIED BY '_YiF46hJ8_dr';
	MariaDB [(none)]> GRANT ALL ON dyslexia_db_2.* TO 'dyslexic'@'localhost';
	MariaDB [(none)]> FLUSH PRIVILEGES;
	MariaDB [(none)]> QUIT;

	MariaDB [(none)]> CREATE DATABASE fltavocatsproduction;
	MariaDB [(none)]> CREATE USER 'faby'@'localhost' IDENTIFIED BY '_Yi_6gYLlXiVhJ8_dr';
	MariaDB [(none)]> GRANT ALL ON fltavocatsproduction.* TO 'faby'@'localhost';
	MariaDB [(none)]> FLUSH PRIVILEGES;
	MariaDB [(none)]> QUIT;


	#Utilisateur omnipotent pou Datagrip
	MariaDB [(none)]> CREATE USER 'god'@'%' IDENTIFIED BY 'er_4stg_f6_';
	MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'god'@'%' WITH GRANT OPTION;
	MariaDB [(none)]> FLUSH PRIVILEGES;
	MariaDB [(none)]> QUIT;

# Ouverture de MariaDB au monde entier : --- non fait pour dyslexie France
	sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
	commenter la ligne :
		# bind-address = x.x.x.x
	Ctrl O, Ctrl X
	sudo systemctl restart mysql

# Création d'un utilisateur pour l'accès depuis l'extérieur --- non fait pour dyslexie France
	sudo mysql
	GRANT ALL PRIVILEGES ON dyslexia_db_2.* TO 'php_storm'@'%' IDENTIFIED BY 'hreiof_64_d46z8' WITH GRANT OPTION;

# Installation POSTFIX pour permettre l'envoi de mail depuis le serveur :
	sudo apt-get update
	sudo apt install -y mailutils
		OK by pressing TAB and ENTER
		Internet Site and press ENTER.
		System mail name should be your domain name eg. example.com, press ENTER.
	sudo nano /etc/postfix/main.cf
		Change to inet_interfaces = loopback-only
	sudo systemctl restart postfix

	sudo nano /etc/postfix/main.cf
    		mydestination = localhost.latelier62.fr, localhost

	sudo systemctl restart postfix




# Installation WordPress en recette

	cd /var/cuisine/dyslexiefrance/public
	sudo wget https://wordpress.org/latest.tar.gz  https://fr.wordpress.org/latest-fr_FR.tar.gz
	sudo tar xfz latest.tar.gz
	sudo mv wordpress/* ./
	sudo rmdir ./wordpress/
	sudo rm -f latest.tar.gz
	sudo mv wp-config-sample.php ../wp-config.php
=> Editer wp-config.php
	Ajouter :
	define('FS_METHOD', 'direct');

#Focus File permissions
	L' utilisateur FTP  deviendra propriétaire de tous les fichiers du site. C'est à dire tout ce qui se situe sous "recette".
	Le groupe de ces fichiers est www-data, le groupe Apache, comme conseillé dans la doc Ubuntu / Apache

	sudo chown -R latelier *
	sudo chgrp -R www-data *

_____________________________________________________________________________


# Troubleshooting
	* Redirection infinie sur wp-admin/
		=> Créer un .htaccess avec les infos de base de WordPress. Attention à son propriétaire et ses permissions
	* Language inchangeable (English - US) - ajouter le répertoire wp-content/languages
	* "Your installation of WordPress doesn't require FTP credentials to perform updates." => Pb de propriétaire et/ou de permissions sur 		wp-content

=> En config / développement, garder des permissions molles (775 et 664) c'est en production qu'on les resserera.
	sudo find . -type f -exec chmod 664 -- {} +
	sudo find . -type d -exec chmod 775 -- {} +


=> Configuration pour envoi de mail
	Symptôme : les mails envoyés depuis php ne sont simplement pas envoyés.
	Setting d'un serveur smtp simplement dans WordPress.


=> Augmentation de la taille max d'upload :
	sudo nano /etc/php/7.4/fpm/php.ini
		upload_max_filesize = 8M
		post_max_size = 8M
		max_execution_time=60 (s)
	sudo service php7.4-fpm restart


=> Installation Imagick
	sudo apt install imagemagick
	sudo apt install php7.4-imagick
	sudo service apache2 restart



# CHANGEMENT DE NOM DE DOMAINE
	Attention, il faut prévoir de changer d'abord le réglage d'url dans WordPress sous peine de ne plus pouvoir y accéder

	Etapes prévues :
		1 - Ajout des enregistrements DNS nouveau domaine (domaine principal + sous-domaine recette)
		2 - Ajout des nouveaux fichiers (un par site) dans sites-available + a2ensite
		3 - Ajout des nouveaux domaines dans le fichier hosts
			3BIS - Modification de la conf ssl et ajout des nouveaux hosts dans /etc/apache2/sites-available/http-vhosts-le-ssl.conf
			3TER - Déplacement du pointage des anciens domaines (fallback)
		4 - Modification de l'url du site dans WordPress.
		5 - Anciens domaines : modifier les répertoires de destination.

MISE EN PRODUCTION
	- "Hardening WordPress" et notamment reprise des permissions


UPGRADE SYSTEME
18.04 - 20.04

sudo do-release-upgrade -m server

// Upgrade PHP
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install php8.2-fpm php8.2-dev php8.2-xml php8.2-zip unzip php8.2-curl php8.2-gd php8.2-mbstring php8.2-xdebug php8.2-ssh2 php8.2-mysql -y --allow-unauthenticated

