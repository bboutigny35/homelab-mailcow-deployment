# 📧 Procédure d'Installation et de Configuration de Mailcow (Debian)

Ce guide détaille le déploiement complet d'un serveur de messagerie Mailcow sur une distribution Debian moderne (11 ou 12), en respectant les standards de sécurité et de configuration réseau.

## 📋 Prérequis

* **Système d'exploitation :** Debian 11 ou 12 (installation minimale recommandée).
* **Ressources matérielles :** Minimum 6 Go de RAM (8 Go recommandés si ClamAV est activé), 20 Go d'espace disque minimum (à adapter en fonction de vos besoin mais il ne faut pas ouvblier que vos mails et leurs pièces jointes seront stockés ici ce qui peut vite saturer l'espace dont vous disposez).
* **Privilèges :** Accès `root` ou utilisateur avec les droits `sudo`.
* **Réseau :** Une adresse IP publique fixe.
* **DNS :** Un nom de domaine avec accès à la gestion de la zone DNS.

Pour ma part, j'utilise Proxmox VE comme hyperviseur et j'ai créé une VM avec les éléments suivants :

<p align="center">
    <img src="/assests/vm-mail-specs.png" alt="Capture VM Specs">
</p>

**NB :** L'éditeur de texte utilisé dépend de vous, ici j'ai fait le choix de 'nano' car c'est le plus utilisé mais j'utilise 'vim' à titre personnel.

---

## 🛠️ Étape 1 : Préparation du système hôte

1. **Mise à jour du système :**

        sudo apt update && sudo apt upgrade -y

2. **Installation des dépendances essentielles :**

**NB :** bind9-dnsutils fournit les outils dig et nslookup, indispensables pour le diagnostic réseau et la vérification des enregistrements DNS (SPF, DKIM, DMARC).

        sudo apt install -y curl git ca-certificates software-properties-common nano bind9-dnsutils

3. **Configuration du nom d'hôte (Hostname) :**

Définissez le FQDN (Fully Qualified Domain Name) de votre serveur mail.

        sudo hostnamectl set-hostname mail.votre-domaine.com

**NB :** L'exemple est donné avec un FQDN puisque c'est le sujet, mais il est tout à fait possible de laisser un nom d'hôte plus classique, pour ma part SRV-BBH-37005.

Éditez le fichier /etc/hosts pour y lier l'IP locale/publique du serveur :

        sudo nano /etc/hosts

Ajoutez la ligne suivante (remplacez l'IP par celle de votre serveur) :

        127.0.0.1 localhost
        198.51.100.10 mail.votre-domaine.com mail

**NB :** Ici aussi vous pouvez tout à fait faire le choix de conserver le nom d'hôté classique plutôt que d'indiquer votre nom de domaine.

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

**NB :** Déconnectez-vous puis reconnectez-vous pour que ce changement prenne effet.

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

Vous pouvez consulter le fichier par défaut [ici](/configs/mailcow.conf)

Paramètres fréquents à vérifier/modifier :

* HTTP_PORT et HTTPS_PORT : Si vous utilisez un Reverse Proxy (comme Nginx Proxy Manager) en amont, modifiez ces ports (ex: HTTP_PORT=8080, HTTPS_PORT=8443) pour éviter les conflits.

* TZ : Définissez votre fuseau horaire (ex: TZ=Europe/Paris).

## 🚀 Étape 5 : Déploiement et Démarrage

1. **Téléchargement des images Docker :**

        sudo docker compose pull

2. **Démarrage de la stack Mailcow :**

        sudo docker compose up -d

Le premier démarrage peut prendre quelques minutes le temps d'initialiser les bases de données et de générer les certificats cryptographiques.

<p align="center">
    <img src="/assests/docker-compose-up.png" alt="Capture docker compose">
</p>

Vous pouvez ensuite vérifier que les services sont bien actifs avec la commande dédiée :

    docker ps

<p align="center">
    <img src="/assests/docker-ps.png" alt="Capture docker compose">
</p>

## 🌐 Étape 6 : Configuration via l'interface Web (DNS & Domaines)

Une fois les conteneurs démarrés, accédez à l'interface d'administration via votre navigateur : https://mail.votre-domaine.com (ou l'IP du serveur).

Identifiants par défaut :

* Utilisateur : admin

* Mot de passe : moohoo (⚠️ Changez ce mot de passe immédiatement après la première connexion !)

<p align="center">
    <img src="/assests/login-mailcow.png" alt="Capture login page mailcow">
</p>

**NB :** J'ai réalisé quelques modifications des paramètres mais la page de connexion est la même, seules les images peuvent changer.

1. **Ajout du domaine de messagerie**

Lorsque vous vous êtes connecté, vous arrivez sur le dahsboard principal :

<p align="center">
    <img src="/assests/mailcow-home-admin.png" alt="Capture home dashboard">
</p>

Dans le menu supérieur droit, allez dans Courriel > Configuration.

<p align="center">
    <img src="/assests/mailcow-courriel-domain.png" alt="Capture courriel dashboard">
</p>

Cliquez sur l'onglet Domaines puis sur le bouton Ajouter un domaine. (Bouton vert)

Entrez votre nom de domaine (ex: votre-domaine.com), définissez le quota global et validez.

<p align="center">
    <img src="/assests/mail-courriel-domain-add.png" alt="Capture add domain">
</p>

2. **Configuration des enregistrements DNS (La clé de la délivrabilité)**

Pour ne pas atterrir dans les spams, votre zone DNS publique doit être rigoureusement configurée.

Dans l'interface Mailcow, cliquez sur le bouton DNS (bouton bleau à droite) à côté de votre domaine fraîchement créé. Mailcow vous affichera tous les enregistrements à créer chez votre registraire (OVH, Cloudflare, etc.).

<p align="center">
    <img src="/assests/mailcow-dns-parameters.png" alt="Capture DNS parameters">
</p>

Les enregistrements critiques à configurer :

* A / AAAA : Fait pointer mail.votre-domaine.com vers l'IP de votre serveur.

* MX (Mail Exchange) : Indique que mail.votre-domaine.com gère les mails pour votre-domaine.com (Priorité 10).

* SPF (TXT) : Autorise votre IP à envoyer des mails pour votre domaine (v=spf1 mx a -all).

* DKIM (TXT) : Signature cryptographique de vos emails. Récupérez la clé publique générée par Mailcow dans l'interface et créez l'enregistrement TXT associé (dkim._domainkey).

* DMARC (TXT) : Indique la politique de rejet en cas d'échec SPF/DKIM (ex: v=DMARC1; p=reject; rua=mailto:postmaster@votre-domaine.com;).

3. **Création des boîtes mail**

Allez dans l'onglet Boîtes de réception.

<p align="center">
    <img src="/assests/mailcow-courriel-mailbox.png" alt="Capture mailbox creation">
</p>

Cliquez sur Ajouter une boîte aux lettres.(bouton vert)

<p align="center">
    <img src="/assests/mailcow-courrier-mailbox-add.png" alt="Capture mailbox add">
</p>

Vous pouvez y renseigner un certain nombre d'éléments, comme :

* L'identifiant : il s'agit de la partie située à gauche du '@' de votre adresse mail (vous n'avez pas besoin de saisir le '@votre-nom-de-domaine.com' puisque vous avez la possibilité de choisir l'un des domaines que vous aurez renseigné au préalable) ;

* Le nom complet : il s'agit de l'alias afficher sur la messagerie de vos correspondants lorsque vous envoyez des mails (ex: Jean DUPONT (jean.dupont@gmail.com)) ;

* Le mot de passe : vous pouvez choisir un mot de passe par défaut pour la première connexion, mais vous remarquez qu'il est possible de générer un mot de passe automatiquement, ce qui peut être pratique, mais vous pouvez aussi utiliser un logiciel externe tel que BitWarden ;

* Je passe un peu sur les autres paramêtres utilisables mais n'hésitez pas à naviguer pour vous familiariser avec l'environnement.

Créez les adresses souhaitées (ex: contact@votre-domaine.com)....

## 🩺 Étape 7 : Tests et Validation

Pour valider le bon fonctionnement de votre infrastructure et sa délivrabilité :

* Envoyez un mail depuis votre nouvelle adresse vers un service de test externe (ex: Gmail, Outlook).

* Utilisez Mail-Tester : L'objectif est d'obtenir le score parfait de 10/10.

* Vérifiez la bonne propagation DNS avec la commande locale, ici l'exemple pour la DMARC :

        dig +short TXT _dmarc.votre-domaine.com

Le résultat doit retourner exactement la chaîne configurée à l'étape 6.

<p align="center">
    <img src="/assests/mailcow-dmarc-test-dig.png" alt="Capture dmarc test">
</p>