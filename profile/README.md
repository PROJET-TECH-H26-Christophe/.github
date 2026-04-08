<!-- TABLE OF CONTENTS -->
<!--
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>
-->


<!-- ABOUT THE PROJECT -->

## À propos du projet : Trieur de patates

Le projet **Trieur de patates** est un système technologique complet visant à automatiser le tri de pommes de terre selon différents critères (taille, couleur, défauts, etc.).

Ce projet intègre plusieurs composantes interconnectées :

-  **Application mobile** : interface utilisateur pour le suivi, le contrôle et la visualisation des données  
-  **API backend** : gestion des communications, du traitement des données et de la logique  
-  **Serveur** : hébergement des services et base de données 
-  **Prototype physique** : système avec capteurs et caméra permettant la détection et le tri des patates

---

## Objectifs du projet

Ce projet a pour objectif de :

-  Aider au tri des patates selon différents critères (taille, couleur, défauts)  
- Utiliser l'IA pour détecter et analyser les caractéristiques des patates  
- Traiter les données en temps réel afin d’optimiser le processus de tri  
- Fournir une visualisation des résultats via une interface utilisateur  

### Technologies utilisées

* [![Node.js][Node.js]][Node-url]
* [![React Native][React Native]][ReactNative-url]
* [![Python][Python]][Python-url]
* [![OpenCV][OpenCV]][OpenCV-url]
* [![Nginx][Nginx]][Nginx-url]
* [![Rocky Linux][RockyLinux]][RockyLinux-url]
* [![Mosquitto][Mosquitto]][Mosquitto-url]
* [![MariaDB][MariaDB]][MariaDB-url]


<!-- GETTING STARTED -->
## Mise en route

Il suffit de cloner le projet et suivre les documentations ci-dessous

- [Documentation Configuration Deploiement du serveur](https://github.com/PROJET-TECH-H26-Christophe/.github/blob/main/docs/DeploiementServeur.md)
- [Documentation App Native](https://github.com/PROJET-TECH-H26-Christophe/PrjTechnoNative/blob/main/README.md)
- [Documentation API](https://github.com/PROJET-TECH-H26-Christophe/PrjTechnoApi/blob/main/README.md)
- [Documentation Objet Connecté](https://github.com/PROJET-TECH-H26-Christophe/PrjTechnoObjCon/blob/main/README.md)
- [Documentation Base de données](https://github.com/PROJET-TECH-H26-Christophe/.github/blob/main/docs/BD.md)
<!-- USAGE EXAMPLES -->
## Screenshot (À venir)

<!-- ROADMAP -->
##  Roadmap

###  Analyse & conception
- [x] Définir les objectifs du système de tri (critères : taille, couleur, défauts)
- [x] Concevoir l’architecture globale (App, API, mobile, serveur)
- [x] Réaliser les schémas (Base de données, Communications)
- [x] Choisir les technologies et outils

---

### Prototype physique
- [x] Concevoir et imaginer un prototype
- [x] Intégrer les capteurs et la caméra
- [x] Tester la capture d’image sur Raspberry Pi

---

### Analyse par IA
- [x] Implémenter la capture d’image avec Python
- [ ] Intégrer OpenCV pour le traitement d’image
- [ ] Détecter les patates
- [ ] Classifier les patates selon les critères définis
- [ ] Optimiser les performances

---

### Communication & MQTT
- [x] Installer et configurer Mosquitto (broker MQTT)
- [x] Définir les topics MQTT (ex: `camera/upload`, `sort/result`)
- [x] Envoyer les données depuis le module Python
- [x] Recevoir et traiter les messages côté serveur / Node-RED
- [x] Sécuriser les communications (authentification, TLS)

---

###  API
- [x] Développer une API Node.js
- [ ] Gérer les données (résultats de tri, images, logs)
- [x] Connecter une base de données MariaDB

---

### Application mobile
- [x] Créer l’application avec React Native
- [x] Implémenter l’authentification
- [ ] Afficher les données de tri en temps réel
- [ ] Ajouter des statistiques
- [x] Connecter l’application à l’API

---

### Infrastructure & déploiement
- [x] Configurer le serveur Rocky Linux
- [ ] Installer et configurer Nginx (reverse proxy)
- [ ] Déployer l’API et les services
- [ ] Configurer les services (systemd, PM2)
- [x] Mettre en place la sécurité (firewall, SSL)

---

### Tests & validation
- [ ] Tester chaque composant individuellement
- [ ] Tester le système complet (end-to-end)
- [ ] Corriger les bugs et améliorer la stabilité

---

<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
<!-- LINKS -->
[Node.js]: https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=node.js&logoColor=white
[Node-url]: https://nodejs.org/

[React Native]: https://img.shields.io/badge/React_Native-20232A?style=for-the-badge&logo=react&logoColor=61DAFB
[ReactNative-url]: https://reactnative.dev/

[Python]: https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white
[Python-url]: https://www.python.org/

[OpenCV]: https://img.shields.io/badge/OpenCV-27338e?style=for-the-badge&logo=opencv&logoColor=white
[OpenCV-url]: https://opencv.org/

[Nginx]: https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white
[Nginx-url]: https://nginx.org/

[RockyLinux]: https://img.shields.io/badge/Rocky_Linux-10B981?style=for-the-badge&logo=rockylinux&logoColor=white
[RockyLinux-url]: https://rockylinux.org/

[Mosquitto]: https://img.shields.io/badge/Mosquitto-3C5280?style=for-the-badge&logo=eclipsemosquitto&logoColor=white
[Mosquitto-url]: https://mosquitto.org/

[MariaDB]: https://img.shields.io/badge/MariaDB-003545?style=for-the-badge&logo=mariadb&logoColor=white
[MariaDB-url]: https://mariadb.org/

[Playwright.dev]: https://img.shields.io/badge/Playwright-2EAD33?style=for-the-badge&logo=playwright&logoColor=white
[Playwright-url]: https://playwright.dev/
