# Déploiement du serveur — Trieur de patates

Ce document explique comment déployer le serveur du projet **Trieur de patates** sur une machine **Rocky Linux**.

L’objectif est d’installer et configurer l’environnement nécessaire pour faire fonctionner :

- l’API **Node.js**
- la base de données **MariaDB**
- le reverse proxy **Nginx**
- le broker MQTT **Mosquitto**
- OpenCV
- NodeRed

---

## Prérequis

Avant de commencer, assurez-vous d’avoir :

- un serveur **Rocky Linux**
- un accès **SSH**
- un utilisateur avec les droits `sudo`
- un nom de domaine ou une adresse IP publique
- le code source du projet disponible sur GitHub
- les ports nécessaires ouverts dans le firewall

## Configuration du firewall (ports à ouvrir)

Afin d’assurer le bon fonctionnement du projet **Trieur de patates**, certains ports doivent être ouverts sur le serveur.

### Ports utilisés

| Port | Protocole | Service | Description |
|------|----------|--------|------------|
| 80   | TCP | HTTP | Accès web via Nginx |
| 443  | TCP | HTTPS | Accès sécurisé (SSL) |
| 1880 | TCP | Node-RED | Interface web Node-RED |
| 1883 | TCP | MQTT | Communication MQTT |
| 8883 | TCP | MQTT TLS | MQTT sécurisé |
| 9001 | TCP | WebSocket MQTT | MQTT via WebSocket |

---

### Ouvrir les ports avec firewalld

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https

sudo firewall-cmd --permanent --add-port=1880/tcp
sudo firewall-cmd --permanent --add-port=1883/tcp
sudo firewall-cmd --permanent --add-port=8883/tcp
sudo firewall-cmd --permanent --add-port=9001/tcp

sudo firewall-cmd --reload
