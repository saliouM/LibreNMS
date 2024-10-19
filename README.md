Voici une version réorganisée et structurée de votre README pour l'installation de LibreNMS :



### Partie 1 

# LibreNMS

## Introduction
LibreNMS est un outil de surveillance de réseau open-source, idéal pour le suivi de votre infrastructure réseau. Ce guide vous aidera à installer LibreNMS sur un serveur Linux.

## Prérequis
Avant de commencer l'installation, assurez-vous d'avoir :

- Un serveur Linux exécutant une version prise en charge par LibreNMS.
- Un accès à la ligne de commande de votre serveur.
- Un serveur web (NGINX recommandé, mais nous allons utiliser Apache).

## Installation de LibreNMS

### 1. Mise à jour du système
Assurez-vous que votre système est à jour :
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Installation des paquets nécessaires
#### Pour Ubuntu 20.04
```bash
apt install software-properties-common
add-apt-repository universe
LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php
apt update
apt install acl curl apache2 fping git graphviz imagemagick libapache2-mod-fcgid mariadb-client mariadb-server mtr-tiny nmap php-cli php-curl php-fpm php-gd php-gmp php-json php-mbstring php-mysql php-snmp php-xml php-zip rrdtool snmp snmpd whois python3-pymysql python3-dotenv python3-redis python3-setuptools python3-systemd python3-pip unzip traceroute
```

#### Pour Ubuntu 22.04
```bash
apt install software-properties-common
LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php
apt update
apt install acl curl fping git graphviz imagemagick mariadb-client mariadb-server mtr-tiny nginx-full nmap php-cli php-curl php-fpm php-gd php-gmp php-json php-mbstring php-mysql php-snmp php-xml php-zip rrdtool snmp snmpd unzip python3-pymysql python3-dotenv python3-redis python3-setuptools python3-psutil python3-systemd python3-pip whois traceroute
```

### 3. Configuration de MariaDB
Sécurisez votre installation de MariaDB :
```bash
sudo mysql_secure_installation
```

#### Configuration de MariaDB
Modifiez le fichier de configuration :
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
Ajoutez les lignes suivantes dans la section `[mysqld]` :
```
innodb_file_per_table=1
lower_case_table_names=0
```
Créez une base de données et un utilisateur pour LibreNMS :
```bash
sudo mysql -u root -p
```
Exécutez les commandes suivantes :
```sql
CREATE DATABASE librenms CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'librenms'@'localhost' IDENTIFIED BY 'votre_mot_de_passe';
GRANT ALL PRIVILEGES ON librenms.* TO 'librenms'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

#### Redémarrer MariaDB
```bash
systemctl enable mariadb
systemctl restart mariadb
```

### 4. Configuration de PHP-FPM
Copiez et modifiez le fichier de configuration pour LibreNMS :
```bash
sudo cp /etc/php/8.3/fpm/pool.d/www.conf /etc/php/8.3/fpm/pool.d/librenms.conf
sudo nano /etc/php/8.3/fpm/pool.d/librenms.conf
```
Changez `[www]` à `[librenms]` et modifiez `user` et `group` :
```ini
[librenms]
user = librenms
group = librenms
listen = /run/php-fpm-librenms.sock
```

### 5. Création d'un utilisateur système pour LibreNMS
Créez un utilisateur dédié et assignez-lui les permissions nécessaires :
```bash
sudo useradd librenms -d /opt/librenms -M -r -s /bin/bash
sudo usermod -aG www-data librenms
```

### 6. Clonage du dépôt LibreNMS
Clonez le dépôt dans `/opt/librenms` :
```bash
sudo git clone https://github.com/librenms/librenms.git /opt/librenms
```

### 7. Installation des dépendances PHP
Installez PHP et les modules nécessaires :
```bash
sudo apt install php8.1 php8.1-cli php8.1-curl php8.1-gd php8.1-mysql php8.1-zip php8.1-xml php8.1-mbstring php8.1-snmp php8.1-json php8.1-bcmath php8.1-fpm php8.1-intl php8.1-common -y
```

### 8. Configuration d'Apache
Créez un fichier de configuration pour LibreNMS dans Apache :
```bash
sudo nano /etc/apache2/sites-available/librenms.conf
```
Ajoutez les lignes suivantes :
```apache
<VirtualHost *:80>
  DocumentRoot /opt/librenms/html/
  ServerName example.com

  <Directory "/opt/librenms/html/">
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/librenms_error.log
  CustomLog ${APACHE_LOG_DIR}/librenms_access.log combined

  <Directory /opt/librenms/>
    Require all granted
  </Directory>
</VirtualHost>
```
Activez la configuration et redémarrez Apache :
```bash
a2dissite 000-default
a2enmod proxy_fcgi setenvif rewrite
a2ensite librenms.conf
systemctl restart apache2
systemctl restart php8.3-fpm
```

### 9. Configuration de PHP
Modifiez le fichier de configuration PHP :
```bash
sudo nano /etc/php/8.1/apache2/php.ini
```
Assurez-vous que les lignes suivantes sont configurées :
```ini
memory_limit = 512M
upload_max_filesize = 64M
max_execution_time = 300
date.timezone = "Africa/Libreville"  # Remplacez par votre timezone
```

### 10. Initialisation de LibreNMS
Donnez les bonnes permissions au répertoire de LibreNMS :
```bash
sudo chown -R librenms:librenms /opt/librenms
sudo chmod -R 775 /opt/librenms
```
Accédez au répertoire et installez les dépendances avec Composer :
```bash
cd /opt/librenms
sudo -u librenms ./scripts/composer_wrapper.php install --no-dev
```
Pour l'installation globale de Composer :
```bash
wget https://getcomposer.org/composer-stable.phar
mv composer-stable.phar /usr/bin/composer
chmod +x /usr/bin/composer
```

### 11. Configuration de la timezone
```bash
sudo nano /etc/php/8.3/fpm/php.ini
sudo nano /etc/php/8.3/cli/php.ini
timedatectl set-timezone Etc/UTC
```

### 12. Accès à l'interface web
Ouvrez votre navigateur et accédez à votre serveur via l'URL (exemple : `http://example.com`). Vous serez dirigé vers l'interface d'installation web pour compléter la configuration.

### 13. Finalisation et configuration SNMP
Configurez SNMP sur le serveur :
```bash
sudo cp /opt/librenms/snmpd.conf.example /etc/snmp/snmpd.conf
sudo systemctl enable snmpd
sudo systemctl restart snmpd
```
Activez la tâche cron pour LibreNMS :
```bash
sudo cp /opt/librenms/librenms.nonroot.cron /etc/cron.d/librenms
```

### 14. Vérifications
Vérifiez les permissions :
```bash
sudo chown -R librenms:librenms /opt/librenms
sudo chmod 771 /opt/librenms
sudo setfacl -d -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
sudo setfacl -R -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
```

### 15. Redémarrer Apache
```bash
sudo systemctl restart apache2
```

### 16. Activer l'auto-complétion des commandes `lnms`
Pour utiliser la touche `Tab` pour compléter les commandes `lnms` :
```bash
ln -s /opt/librenms/lnms /usr/bin/lnms
cp /opt/librenms/misc/lnms-completion.bash /etc/bash_completion.d/
```

### 17. Configurer SNMPD
1. Copiez le fichier de configuration exemple :
   ```bash
   cp /opt/librenms/snmpd.conf.example /etc/snmp/snmpd.conf
   ```

2. Modifiez le fichier de configuration pour définir votre propre chaîne de communauté :
   ```bash
   vi

 /etc/snmp/snmpd.conf
   ```

3. Redémarrez le service SNMP :
   ```bash
   systemctl restart snmpd
   ```

### 18. Configuration des mises à jour
Ajoutez des mises à jour automatiques dans le fichier de cron pour LibreNMS :
```bash
sudo nano /etc/cron.d/librenms
```
Ajoutez la ligne suivante :
```
* * * * * librenms /opt/librenms/crons.php > /dev/null 2>&1
```

### Partie 2

Voici la mise à jour complète avec vos suggestions :

---

### Configuration SNMP sur les équipements Cisco

Pour permettre à LibreNMS de surveiller vos équipements Cisco, vous devez configurer SNMP sur chaque appareil. Voici comment procéder :

#### 1. Connexion à votre équipement Cisco
Connectez-vous à votre routeur ou switch Cisco via SSH ou en utilisant la console.

#### 2. Accéder au mode de configuration
Entrez dans le mode de configuration global :
```bash
enable
configure terminal
```

#### 3. Configurer SNMP
Ajoutez une communauté SNMP, par exemple, `public` :
```bash
snmp-server community public RO
```
Remplacez `public` par une chaîne de communauté plus sécurisée.

- **Configurer l'adresse de l'hôte LibreNMS :**
```bash
snmp-server host <adresse_ip_de_librenms> version 2c public
```

- **Configurer le trap SNMP :**
```bash
snmp-server enable traps
```

- **Configurer la version SNMP :**
```bash
snmp-server version 2c
```

#### 4. Assurez-vous de la connectivité
- Assurez-vous que les ports SNMP (UDP 161 pour les requêtes, UDP 162 pour les traps) sont ouverts sur tous les pare-feux entre le switch et l'hôte de gestion SNMP.
- Si vous utilisez SNMPv3, la configuration nécessite l'utilisation d'utilisateurs et d'authentification.

#### 5. Vérification de la configuration SNMP
Après avoir configuré SNMP, vérifiez la configuration en utilisant la commande suivante :
```bash
show running-config | include snmp
```

#### 6. Sauvegarder la configuration
N'oubliez pas de sauvegarder votre configuration :
```bash
write memory
```
ou
```bash
copy running-config startup-config
```

---

### Exemple de Configuration SNMP Avancée

Voici un exemple de configuration SNMP avancée pour un switch Cisco (modèle : WS-C2960-24PC-L) :

```bash
enable
configure terminal

! Activer tous les traps listés
snmp-server enable traps auth-framework
snmp-server enable traps bridge
snmp-server enable traps cluster
snmp-server enable traps config
snmp-server enable traps config-copy
snmp-server enable traps config-ctid
snmp-server enable traps copy-config
snmp-server enable traps cpu
snmp-server enable traps dot1x
snmp-server enable traps energywise
snmp-server enable traps entity
snmp-server enable traps envmon
snmp-server enable traps errdisable
snmp-server enable traps flash
snmp-server enable traps fru-ctrl
snmp-server enable traps mac-notification
snmp-server enable traps port-security
snmp-server enable traps power-ethernet group 1
snmp-server enable traps rep
snmp-server enable traps snmp
snmp-server enable traps storm-control storm-control trap-rate 1000
snmp-server enable traps stpx
snmp-server enable traps syslog
snmp-server enable traps transceiver all
snmp-server enable traps tty
snmp-server enable traps vlan-membership
snmp-server enable traps vlancreate
snmp-server enable traps vlandelete
snmp-server enable traps vtp
```

---

Cette section permet de configurer des traps SNMP avancés pour la gestion des événements sur un switch Cisco. Assurez-vous de tester la configuration après l’avoir appliquée pour vérifier la réception des traps SNMP par l'hôte LibreNMS.

