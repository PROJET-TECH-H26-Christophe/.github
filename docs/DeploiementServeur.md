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
```

## Configuration du reverse proxy (Nginx)

Cette section explique comment configurer **Nginx** pour :

- écouter sur le domaine `martinelaplante.com`
- rediriger le trafic HTTPS (443) vers l’API Node.js interne (port 3000)

---

### 1. Installer Nginx

```bash
sudo dnf install nginx -y
```

Démarrer et activer Nginx :
```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```
Vérifier :
```bash
sudo systemctl status nginx
```

### 2. Configurer nginx

Créer un fichier de configuration :
```bash
sudo nano /etc/nginx/conf.d/martinelaplante.com.conf
```
Ajouter la configuration suivante :
```bash
server {
    listen 80;
    server_name martinelaplante.com www.martinelaplante.com;

    # Redirection HTTP → HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name martinelaplante.com www.martinelaplante.com;

    # Certificats SSL (à générer avec certbot)
    ssl_certificate /etc/letsencrypt/live/martinelaplante.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/martinelaplante.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';

        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;

        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 3. Installer Certbot (SSL Let's Encrypt)

```bash
sudo dnf install epel-release -y
sudo dnf install certbot python3-certbot-nginx -y
```
Générer le certificat SSL :
```bash
sudo certbot --nginx -d martinelaplante.com -d www.martinelaplante.com
```

### 4. Tester et recharger Nginx

Tester la configuration :
```bash
sudo nginx -t
```
Recharger :
```bash
sudo systemctl reload nginx
```

