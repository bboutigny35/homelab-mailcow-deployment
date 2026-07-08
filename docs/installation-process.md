# 📧 Procédure d'Installation et de Configuration de Mailcow (Debian)

Ce guide détaille le déploiement complet d'un serveur de messagerie Mailcow sur une distribution Debian moderne (11 ou 12), en respectant les standards de sécurité et de configuration réseau.

## 📋 Prérequis

* **Système d'exploitation :** Debian 11 ou 12 (installation minimale recommandée).
* **Ressources matérielles :** Minimum 6 Go de RAM (8 Go recommandés si ClamAV est activé), 20 Go d'espace disque.
* **Privilèges :** Accès `root` ou utilisateur avec les droits `sudo`.
* **Réseau :** Une adresse IP publique fixe.
* **DNS :** Un nom de domaine avec accès à la gestion de la zone DNS.

---

## 🛠️ Étape 1 : Préparation du système hôte

1. **Mise à jour du système :**

    sudo apt update && sudo apt upgrade -y

2. **Installation des dépendances essentielles :**

**Note :** bind9-dnsutils fournit les outils dig et nslookup, indispensables pour le diagnostic réseau et la vérification des enregistrements DNS (SPF, DKIM, DMARC).

    sudo apt install -y curl git ca-certificates software-properties-common nano bind9-dnsutils

3. **Configuration du nom d'hôte (Hostname) :**

Définissez le FQDN (Fully Qualified Domain Name) de votre serveur mail.

    sudo hostnamectl set-hostname mail.votre-domaine.com

Éditez le fichier /etc/hosts pour y lier l'IP locale/publique du serveur :

    sudo nano /etc/hosts

Ajoutez la ligne suivante (remplacez l'IP par celle de votre serveur) :

    127.0.0.1 localhost
    198.51.100.10 mail.votre-domaine.com mail

4. **Préparation du routage réseau (IP Forwarding) :**

Docker a besoin de router les paquets entre ses conteneurs et l'interface externe.

    sudo nano /etc/sysctl.conf

Décommentez ou ajoutez cette ligne :

    net.ipv4.ip_forward=1

Appliquez la modification :

    sudo sysctl -p

## 🛠️ Étape 2 : Installation de Docker et Docker Compose

Afin d'éviter les paquets obsolètes des dépôts par défaut, il est recommandé d'utiliser le script d'installation officiel de Docker.

1. **Téléchargement et exécution du script officiel :**

    curl -fsSL [https://get.docker.com](https://get.docker.com) -o get-docker.sh
    sudo sh get-docker.sh

2. **Ajout de l'utilisateur au groupe Docker (Optionnel mais recommandé) :**

    sudo usermod -aG docker $USER

**Note :** Déconnectez-vous puis reconnectez-vous pour que ce changement prenne effet.

3. **Vérification de l'intallation :**

    docker composer version

## ⚙️ Étape 3 : Installation de Mailcow

1. **Clonage du dépôt officiel :**

L'installation de Mailcow doit se faire dans le répertoire /opt/.

    sudo git clone [https://github.com/mailcow/mailcow-dockerized](https://github.com/mailcow/mailcow-dockerized) /opt/mailcow-dockerized

Une fois cloné, placez-vous dans le répertoire de mailcow :

    cd /opt/mailcow-dockerized

2. **Génération du fichier de configuration :**

Lancez le script de génération. Il vous demandera le FQDN de votre serveur (ex: mail.votre-domaine.com).

    sudo ./generate_config.sh

## 🔧 Étape 4 : Paramétrage avancé (mailcow.conf)

Le fichier de configuration principal est mailcow.conf. Vous pouvez l'éditer pour ajuster les paramètres selon votre infrastructure.

    sudo nano mailcow.conf

Paramètres fréquents à vérifier/modifier :

* HTTP_PORT et HTTPS_PORT : Si vous utilisez un Reverse Proxy (comme Nginx Proxy Manager) en amont, modifiez ces ports (ex: HTTP_PORT=8080, HTTPS_PORT=8443) pour éviter les conflits.

* TZ : Définissez votre fuseau horaire (ex: TZ=Europe/Paris).

* SKIP_CLAMD=y : Si vous avez moins de 6 Go de RAM, il est fortement conseillé de désactiver l'antivirus ClamAV pour éviter les crashs (OOM).

## 🚀 Étape 5 : Déploiement et Démarrage

1. **Téléchargement des images Docker :**

    sudo docker compose pull

2. **Démarrage de la stack Mailcow :**

    sudo docker compose up -d

Le premier démarrage peut prendre quelques minutes le temps d'initialiser les bases de données et de générer les certificats cryptographiques.

## 🌐 Étape 6 : Configuration via l'interface Web (DNS & Domaines)

Une fois les conteneurs démarrés, accédez à l'interface d'administration via votre navigateur : https://mail.votre-domaine.com (ou l'IP du serveur).

Identifiants par défaut :

* Utilisateur : admin

* Mot de passe : moohoo (⚠️ Changez ce mot de passe immédiatement après la première connexion !)

1. **Ajout du domaine de messagerie**

Dans le menu supérieur, allez dans Configuration > Configuration du Courrier.

Cliquez sur l'onglet Domaines puis sur le bouton Ajouter un domaine.

Entrez votre nom de domaine (ex: votre-domaine.com), définissez le quota global et validez.

2. **Configuration des enregistrements DNS (La clé de la délivrabilité)**

Pour ne pas atterrir dans les spams, votre zone DNS publique doit être rigoureusement configurée.

Dans l'interface Mailcow, cliquez sur le bouton DNS à côté de votre domaine fraîchement créé. Mailcow vous affichera tous les enregistrements à créer chez votre registraire (OVH, Cloudflare, etc.).

Les enregistrements critiques à configurer :

* A / AAAA : Fait pointer mail.votre-domaine.com vers l'IP de votre serveur.

* MX (Mail Exchange) : Indique que mail.votre-domaine.com gère les mails pour votre-domaine.com (Priorité 10).

* SPF (TXT) : Autorise votre IP à envoyer des mails pour votre domaine (v=spf1 mx a -all).

* DKIM (TXT) : Signature cryptographique de vos emails. Récupérez la clé publique générée par Mailcow dans l'interface et créez l'enregistrement TXT associé (dkim._domainkey).

* DMARC (TXT) : Indique la politique de rejet en cas d'échec SPF/DKIM (ex: v=DMARC1; p=reject; rua=mailto:postmaster@votre-domaine.com;).

3. **Création des boîtes mail**

Allez dans l'onglet Boîtes aux lettres.

Cliquez sur Ajouter une boîte aux lettres.

Créez les adresses souhaitées (ex: contact@votre-domaine.com).

## 🩺 Étape 7 : Tests et Validation

Pour valider le bon fonctionnement de votre infrastructure et sa délivrabilité :

* Envoyez un mail depuis votre nouvelle adresse vers un service de test externe (ex: Gmail, Outlook).

* Utilisez Mail-Tester : L'objectif est d'obtenir le score parfait de 10/10.

* Vérifiez la bonne propagation DNS avec la commande locale :

    dig +short TXT _dmarc.votre-domaine.com

Le résultat doit retourner exactement la chaîne configurée à l'étape 6.