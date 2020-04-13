# Documentation

## A- Présentation

### I- Diaporama

[Lien de la présentation](https://www.canva.com/design/DAD1GkHHv_Q/quOE6o_ii10qIKzrpJ7FTA/view?utm_content=DAD1GkHHv_Q&utm_campaign=designshare&utm_medium=link&utm_source=sharebutton&fbclid=IwAR1G7Pt1T8zxhEBWF1_YGUfD1B7f0-tGOz6sgU-_uOqTxCaPXd-dLiSOtv8)

### II- Site

[Lien vers notre site héberger sur notre raspberry](koeltoxik.ddns.net)

## B- Initialisation du projet

### I- Pré-requis

**Matériels**

Raspberry Pi 4 (Raspbian Buster)

Livebox


### II- Installation et initialisation du projet

**Configuration de la raspberry**

Installation de raspbian 10 (Buster)

Ouverture et fowarding des ports 22, 80, 443 et 8000

Mise en place de clé privée pour désactiver la connection ssh par mot de passe


**Installation des dépendances**

php7.3

php7.3-mysql

PHP7.3-xml

php7.3-mbstring

php7.3-curl

php7.3-zip

MariaDB


**Clonage du dépot git et attribution de droit**
```bash
git clone --recurse-submodules https://github.com/phanan/koel.git
git checkout v4.2.2
chown -R www-data:www-data /var/www/koel
```

*Création d'une base de donnée*
```bash
sudo mysql -u root -p
mysql> CREATE DATABASE koel DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
mysql> CREATE USER 'koel-db-user'@'localhost' IDENTIFIED BY 'koel-pass';
mysql> GRANT ALL PRIVILEGES ON koel.* TO 'koel-db-user'@'localhost' WITH GRANT OPTION;
mysql> exit;
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

*Installation de composer et npm*
```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'e0012edf3e80b6978849f5eff0d4b4e4c79ff1609dd1e613307e16318854d24ae64f26d17af3ef0bf7cfb710ca74755a') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
composer install
```
En cas d'erreur (ARM):
Enlever cypress du package.json et ajouter les assets à la main
Enlever le bar.gif ou la modifier en fichier PNG


*Lancement du projet et du serveur local à l'aide de php artisan*
```bash
php artisan koel:init
php artisan serv
```

*Vérification sur localhost*

Vérifier le premier rendu visuel sur http://localhost:8000


**III- Mise en place d'un reverse Proxy**

*Création d'un vHost pour ajouter koel en sous domaine*
```bash
vim etc/nginx/sites-available/koel.conf

sudo ln -s /etc/nginx/sites-available/koel.conf /etc/nginx/sites-enabled/
```

*Configuration de Nginx*

[Générateur de fichier Nginx](https://www.digitalocean.com/community/tools/nginx)

[Exemple de configuration pour koel](https://github.com/phanan/koel/blob/master/nginx.conf.example)

*Port Forwarding*
```bash 
netstat -ntl
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8000
sudo iptables -t nat -L
sudo sh -c "iptables-save > /etc/iptables.rules"
sudo apt-get install iptables-persistent
 ```

*Lancement de Nginx*
```bash
sudo service nginx restart
sudo systemctl enable nginx
```

**IV- Mettre à jour Koel**

*Regarder la dernière version disponible*

[Lien du dépôt des Maj](https://github.com/phanan/koel/releases)


*Mettre à jour*
```bash
git checkout v4.2.2
```

## C- Architecture du site

**I- Les views**

Nos views se situent dans deux endroits:
Pour nos .vue: /ressources/assets/js/components
Pour nos .blade.php: /ressources/views


**II- Nos configurations d'API**

Nos API ont tout d abord été configurées dans le .env puis dans différent fichier en fonction de l API.


**III- Nos routes et controller**

Pour nos routes: /routes
Pour nos controller: /App/Http/Controller


**IV- Emplacement des fichiers Upload**
```bash
Les Uploads effectués sur le site se trouve à deux endroits reliés par un lien symbolique:
Dans notre dossier Koel: /public/file
Sur notre Raspberry: /home/pi/Music
```

## D- Développement du projet

**I- Quelques commandes**


*php artisan migrate*
```bash
php artisan migrate >>> Execute les migrations restantes.
php artisan migrate:refresh >>> Supprime la base de données et refais les migration.
php artisan migrate:refresh --seed >>> Variante qui permet de lancer les seeders en plus.
php artisan migrate:reset >>> Supprime la base de données.
```

*php artisan make*
```bash
php artisan make:controller Name >>> Création de controller dans le dossier 'app/Http/Controllers'.
php artisan make:middleware Name >>> Création de middleware dans le dossier 'app/Http/Middleware'.
php artisan make:migration Name >>> Création de fichier de migration dans le dossier 'database/migrations'.
php artisan make:seeder Name >>> Création de seeder dans le dossier 'database/seeds'.
```

*Pour lancer un serveur en local*
```bash
php artisan serv
```

*Pour compiler les .vue*
```bash
yarn build
```

*Pour transferer les fichiers sur la raspberry*
```bash
scp -r -p chemin/vers/dossier/source user@ip:chemin/vers/dossier/destination
```

*Pour créer un lien symbolique entre deux fichiers*
```bash
ln –s /chemin/dossier1 /chemin/dossier2 
```

**II- Ajout d'un Register au niveau du login**
```bash
Tout en gardant le format de koel, nous avons ajouté un bouton register au niveau du login qui ouvre un modal sous forme de pop up dans le même esprit que koel
```

**III- Ajout d'une page Upload**
```bash
Toujours dans le même esthétique que koel, nous avons ajouté une nouvelle page /file pour permettre d uploader des musiques directement sur l application et notre dossier source, il faut tout de même qu un administrateur face la synchronisation pour que les musiques s'affichent pour garder un minimum de contrôle
```

**IV- Ajout de plusieurs API**
```bash
Ajout de l API Last.fm pour avoir le cover, l album et l artiste
Ajout de l API Youtube pour avoir les vidéos lors de la lecture d'une musique
Ajout de l API Pusher pour prendre le contrôle de KoelDesktop depuis son téléphone (disponible uniquement sur MacOS)
```

## E- Problèmes rencontrés

**I- Structure ARM**

*Les dépendances*
```bash
Certaines dépendances ne sont pas supportés par l ARM comme cypress ou les gif, ce qui bloque le rendu de certain visuel ou l init du projet
```

*La compilation*
```bash
Il est impossible de compiler avec un npm run watch ou yarn build sur de l ARM donc de voir les modifications des fichiers
```

**II- L'installation de l'API de google pour l'authentification**
```bash
Après avoir ajouté le plugin pour l API de google, notre page de login chargeait à l infini...
```

**III- Mise en place d'un systeme d'envoi de mail**
```bash
Après avoir mis en place l environnement et créer un compte sur mailgun et postman, il était impossible de voir si les mails étaient bien reçu car il fallait passer sur une version payante.
```

**IV- Le Token de connection**
```bash
Le token de connection nous a complétement bloqué sur les middlewares et controller pour notre upload car nous n avons pas réussi à le garder lorsque l on passait sur le /file
```

**V- La compréhension du code**
Ce qui nous a pris le plus de temps sur ce projet, comprendre et s approprier un code que nous n avons pas écrit était vraiment un défi car ce sont des techs que nous ne connaissions pas et nous avons du nous mettre à niveau sur beaucoup de points pour pouvoir développer sans avoir de conflit.

## F- Sites utiles

**I- Koel**
[Site Officiel] (https://koel.dev/)
[Koel Issues] (https://github.com/phanan/koel/issues)

**II- Laravel**
[Site Officiel] (https://laravel.com/)
[Documentation Officiel] (https://laravel.com/docs/7.x)
[Documentation Unofficiel] (http://laravel.sillo.org/)

**III- API**
[API Directory] (https://www.programmableweb.com/apis/directory)
[Site Officiel Last.fm] (https://www.last.fm/fr/)
[API Last.fm] (https://www.last.fm/api/?lang=fr&)
[API Youtube] (https://developers.google.com/youtube)
[API Pusher] (https://pusher.com/docs/channels)

**IV- Mail**
[Mailgun] (https://www.mailgun.com/)
[Postman] (https://www.postman.com/)

**V- Réseau**
[NoIp] (https://www.noip.com/)
[Nginx Documentation] (https://nginx.org/en/docs/)
[Nginx Config] (https://www.digitalocean.com/community/tools/nginx)

**VI- Raspberry**
[Site officiel de Raspberry] (https://www.raspberrypi.org/)
[Site officiel de Raspbian] (https://www.raspbian.org/)

