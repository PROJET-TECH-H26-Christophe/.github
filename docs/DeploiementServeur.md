# Déploiement du serveur — Trieur de patates

Ce document explique comment déployer le serveur du projet **Trieur de patates** sur une machine **Rocky Linux**.

L'objectif est d'installer et configurer l'environnement nécessaire pour faire fonctionner :

- l'API **Node.js**
- la base de données **MariaDB**
- le reverse proxy **Nginx**
- le broker MQTT **Mosquitto**
- **Node-RED**
- **OpenCV**

---

## Prérequis

Avant de commencer, assurez-vous d'avoir :

- un serveur **Rocky Linux**
- un accès **SSH**
- un utilisateur avec les droits `sudo`
- un nom de domaine ou une adresse IP publique

---

## 1. Configuration du firewall

Afin d'assurer le bon fonctionnement du projet, certains ports doivent être ouverts sur le serveur.

### Ports utilisés

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| 80 | TCP | HTTP | Accès web via Nginx |
| 443 | TCP | HTTPS | Accès sécurisé (SSL) |
| 1880 | TCP | Node-RED | Interface web Node-RED |
| 1883 | TCP | MQTT | Communication MQTT non sécurisée (local) |
| 8883 | TCP | MQTT TLS | MQTT sécurisé (utilisé par le Pi) |
| 9001 | TCP | WebSocket MQTT | MQTT via WebSocket (app mobile) |

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

---

## 2. Installation et configuration de Nginx

Nginx sert de reverse proxy — il redirige le trafic HTTPS (port 443) vers l'API Node.js interne (port 3000).

### Installer Nginx

```bash
sudo dnf install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

### Installer Certbot (SSL Let's Encrypt)

```bash
sudo dnf install epel-release -y
sudo dnf install certbot python3-certbot-nginx -y
sudo certbot --nginx -d votre-domaine.com -d www.votre-domaine.com
```

### Configurer Nginx

Créer un fichier de configuration en remplaçant `votre-domaine.com` par votre domaine :
```bash
sudo nano /etc/nginx/conf.d/votre-domaine.com.conf
```

Ajouter la configuration suivante :
```nginx
server {
    listen 80;
    server_name votre-domaine.com www.votre-domaine.com;

    # Redirection HTTP → HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name votre-domaine.com www.votre-domaine.com;

    # Certificats SSL (générés avec certbot)
    ssl_certificate /etc/letsencrypt/live/votre-domaine.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/votre-domaine.com/privkey.pem;

    # API Node.js
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

    # Node-RED
    location /nodered/ {
        proxy_pass http://127.0.0.1:1880/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Tester et recharger Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 3. Installation et configuration de MariaDB

### Installer MariaDB

```bash
sudo dnf install mariadb-server -y
sudo systemctl enable mariadb
sudo systemctl start mariadb
```

### Sécuriser MariaDB

```bash
sudo mysql_secure_installation
```

| Question | Réponse recommandée |
|----------|--------------------|
| Set root password | Y |
| Remove anonymous users | Y |
| Disallow root login remotely | Y |
| Remove test database | Y |
| Reload privilege tables | Y |

### Créer la base de données et l'utilisateur

Se connecter à MariaDB :
```bash
sudo mysql -u root -p
```

Créer la base de données :
```sql
CREATE DATABASE PrjTechno;
```

Créer un utilisateur dédié (remplacer `MotDePasse` par un mot de passe fort) :
```sql
CREATE USER 'prjtechno_user'@'localhost' IDENTIFIED BY 'MotDePasse';
GRANT ALL PRIVILEGES ON PrjTechno.* TO 'prjtechno_user'@'localhost';
FLUSH PRIVILEGES;
```

---

## 4. Installation et configuration de Mosquitto (MQTT)

### Installer Mosquitto

```bash
sudo dnf install epel-release -y
sudo dnf makecache
sudo dnf install mosquitto -y
```

### Copier les certificats SSL

Mosquitto a besoin de ses propres copies des certificats Let's Encrypt :
```bash
sudo mkdir -p /etc/mosquitto/certs
sudo cp /etc/letsencrypt/live/votre-domaine.com/fullchain.pem /etc/mosquitto/certs/
sudo cp /etc/letsencrypt/live/votre-domaine.com/privkey.pem /etc/mosquitto/certs/
sudo chown mosquitto:mosquitto /etc/mosquitto/certs/*.pem
```

### Renouvellement automatique des certificats

Les certificats Let's Encrypt se renouvellent tous les 90 jours. Il faut aussi mettre à jour les copies Mosquitto automatiquement.

Créer un script de renouvellement :
```bash
sudo nano /etc/letsencrypt/renewal-hooks/deploy/mosquitto.sh
```

Contenu :
```bash
#!/bin/bash
cp /etc/letsencrypt/live/votre-domaine.com/fullchain.pem /etc/mosquitto/certs/fullchain.pem
cp /etc/letsencrypt/live/votre-domaine.com/privkey.pem /etc/mosquitto/certs/privkey.pem
chown mosquitto:mosquitto /etc/mosquitto/certs/*.pem
systemctl restart mosquitto
```

Rendre le script exécutable :
```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/mosquitto.sh
```

### Configurer Mosquitto

La configuration est séparée en deux fichiers.

**Fichier 1 — Config de base** (`/etc/mosquitto/conf.d/default.conf`) :
```bash
sudo nano /etc/mosquitto/conf.d/default.conf
```

Contenu :
```
listener 1883
allow_anonymous false
password_file /etc/mosquitto/passwd
```

**Fichier 2 — Config TLS** (`/etc/mosquitto/conf.d/mqtts.conf`) :
```bash
sudo nano /etc/mosquitto/conf.d/mqtts.conf
```

Contenu :
```
listener 8883
protocol mqtt
certfile /etc/mosquitto/certs/fullchain.pem
keyfile /etc/mosquitto/certs/privkey.pem

listener 9001
protocol websockets
certfile /etc/mosquitto/certs/fullchain.pem
keyfile /etc/mosquitto/certs/privkey.pem
```

### Créer les utilisateurs MQTT

```bash
sudo mosquitto_passwd -c /etc/mosquitto/passwd pythonapp
sudo mosquitto_passwd /etc/mosquitto/passwd nodered
```

### Démarrer Mosquitto

```bash
sudo systemctl enable mosquitto
sudo systemctl start mosquitto
```

---

## 5. Installation et configuration de Node-RED

### Installer Node-RED

```bash
sudo npm install -g node-red
```

### Créer un utilisateur dédié

```bash
sudo useradd -m -s /bin/bash nodered
```

### Créer le service systemd

```bash
sudo nano /etc/systemd/system/nodered.service
```

Contenu :
```ini
[Unit]
Description=Node-RED
After=network.target

[Service]
Type=simple
User=nodered
Group=nodered
WorkingDirectory=/home/nodered
ExecStart=/usr/bin/node-red
Restart=always
RestartSec=5
KillSignal=SIGINT
SyslogIdentifier=Node-RED
Environment=NODERED_API_KEY=votre-cle-api-nodered

[Install]
WantedBy=multi-user.target
```

> **Note** : La clé d'api nodered peut être générer grâce au script dans le repo d'api (voir doc api).

### Démarrer Node-RED

```bash
sudo systemctl daemon-reload
sudo systemctl enable nodered
sudo systemctl start nodered
```

### Configurer l'authentification Node-RED

Par défaut, l'interface Node-RED est accessible sans mot de passe. Il faut activer l'authentification.

Générer un hash du mot de passe :
```bash
node-red admin hash-pw
```

Entrer le mot de passe désiré — copier le hash retourné.

Modifier le fichier de configuration Node-RED :
```bash
sudo nano /home/nodered/.node-red/settings.js
```

Trouver et décommenter la section `adminAuth` en remplaçant le hash par celui généré :
```javascript
adminAuth: {
    type: "credentials",
    users: [{
        username: "admin",
        password: "$2y$08$votre-hash-genere-ici",
        permissions: "*"
    }]
},
```

Redémarrer Node-RED pour appliquer les changements :
```bash
sudo systemctl restart nodered
```

L'interface Node-RED est maintenant accessible via `https://votre-domaine.com/nodered/` avec les identifiants configurés.

### Importer les flows Node-RED

Accéder à l'interface et importer les flows JSON fournis dans le repo de l'organisation :

| Flow | Description |
|------|-------------|
| Temperature Upload | Reçoit les données DHT22 via MQTT et les envoie à l'API |
| Potato Upload | Reçoit les photos du Pi, valide le scanner et sauvegarde |
| Scanner | Valide les scanners via MQTT |
| Heartbeat | Met à jour le LastSeenAt des scanners dans la BD |
| Production Active | Retourne la production active au Pi via MQTT |

---

## 6. Installation d'OpenCV

### Installer les dépendances Python

```bash
sudo dnf install python3 python3-pip -y
pip3 install opencv-python numpy --break-system-packages
```

### Vérifier l'installation

```bash
python3 -c "import cv2; print(cv2.__version__)"
```

---

## 7. Vérification finale

Vérifier que tous les services sont actifs :

```bash
sudo systemctl status nginx
sudo systemctl status mariadb
sudo systemctl status mosquitto
sudo systemctl status nodered
sudo systemctl status projet-techno-api
```

Tester l'API :
```bash
curl https://votre-domaine.com/api/ping
# Réponse attendue : pong
```
