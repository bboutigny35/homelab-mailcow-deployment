# 📧 Déploiement et Sécurisation d'un Serveur de Messagerie (Mailcow)

## 📌 Contexte du Projet
Dans le cadre de ma montée en compétences continue en administration système et réseau, j'ai déployé une infrastructure de messagerie complète et autonome au sein de mon HomeLab. 

L'objectif de ce projet est de démontrer ma capacité à maîtriser la chaîne complète d'un service critique : de la préparation de l'OS hôte jusqu'à la validation de la délivrabilité, en passant par le routage, la conteneurisation et la gestion fine des enregistrements DNS.

## 🛠️ Stack Technique
* **Système d'exploitation :** Debian (configuration native, gestion stricte via `nftables`)
* **Conteneurisation :** Docker & Docker Compose
* **Solution Mail :** Mailcow (Postfix, Dovecot, SOGo, Rspamd)
* **Réseau & Sécurité :** Unbound (résolution DNS locale), Reverse Proxy, Let's Encrypt
* **Protocoles gérés :** SMTP, IMAP, configuration avancée DNS (MX, TXT, A)

## 🚀 Réalisations et Défis Techniques
* **Installation "From Scratch" :** Déploiement complet en évitant les surcouches inutiles (exclusion d'outils obsolètes ou conflictuels pour privilégier les standards Debian).
* **Résolution DNS Complexe :** Dépannage et configuration des flux réseaux internes au réseau Docker (`unbound-mailcow`) pour assurer la résolution des requêtes vers l'extérieur.
* **Sécurisation et Délivrabilité :** Paramétrage rigoureux des enregistrements **SPF, DKIM et DMARC**. 
  * *Résultat :* Obtention d'un score de délivrabilité de 10/10 lors des tests (absence totale de marquage spam).

## 📂 Contenu de ce dépôt
* `docs/procedure_installation.md` : Ma documentation standardisée pour le déploiement sur une base Debian propre.
* `assets/` : Captures d'écran des tests de délivrabilité et de l'architecture.

## 🎯 Compétences illustrées
> Ce projet illustre mon appétence pour les infrastructures robustes, le troubleshooting réseau (utilisation de `dig`, analyse des logs Docker) et la rédaction de documentation technique claire.