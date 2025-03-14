# Surveillance et Sécurisation du Serveur

Ce projet regroupe plusieurs outils pour surveiller et sécuriser un serveur Linux :

- **Fail2Ban** : Protection contre les attaques par force brute.
- **MSMTP** : Envoi d'alertes par e-mail.
- **inotify-tools** : Surveillance des fichiers de configuration.

## 📌 Installation et Configuration

### 🔹 1. Fail2Ban
Fail2Ban protège contre les tentatives de connexion répétées sur SSH et d'autres services sensibles.

**Installation :**
```bash
sudo apt update && sudo apt install fail2ban -y
```

**Configuration :**
Fichier : `/etc/fail2ban/jail.local`
```ini
[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 5
bantime = 3600
action = %(action_mwl)s
```

**Redémarrer le service :**
```bash
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban
```

**Vérifier l'état :**
```bash
sudo fail2ban-client status sshd
```

---

### 🔹 2. MSMTP (Envoi d'alertes e-mail)
MSMTP permet d'envoyer des e-mails depuis un serveur sans MTA complexe.

**Installation :**
```bash
sudo apt install msmtp msmtp-mta -y
```

**Configuration :**
Fichier : `~/.msmtprc`
```ini
account gmail
host smtp.gmail.com
port 587
from emmanueladounvo408@gmail.com
auth on
user emmanueladounvo408@gmail.com
password [mot_de_passe]
tls on
tls_starttls on
logfile ~/.msmtp.log

account default : gmail
```

**Test d'envoi :**
```bash
echo -e "Subject: Test Email\n\nCeci est un test" | msmtp --debug adounvosteve@gmail.com
```

---

### 🔹 3. Inotify (Surveillance de fichiers)

**Installation :**
```bash
sudo apt install inotify-tools -y
```

**Script de surveillance :** `monitor.sh`
```bash
#!/bin/bash
config_dir="/etc/nginx"
admin_email="adounvosteve@gmail.com"

send_email() {
    local file=$1
    echo -e "Subject: Modification de configuration - $file\n\nLe fichier de configuration $file a été modifié." | msmtp $admin_email
}

inotifywait -m -r -e modify,create,delete $config_dir |
while read path action file; do
    send_email "$path$file"
done
```

**Lancer la surveillance :**
```bash
./monitor.sh &
```

---

### 🔹 4. Copie et Restauration Automatique de Configuration

Pour éviter toute perte de configuration importante, un script sauvegarde les fichiers et les restaure en cas de modification non autorisée.

**Script de sauvegarde/restauration :** `backup_restore.sh`
```bash
#!/bin/bash
config_dir="/etc/nginx"
backup_dir="/backup_nginx"
admin_email="adounvosteve@gmail.com"

mkdir -p $backup_dir
cp -r $config_dir/* $backup_dir/

echo "Sauvegarde initiale effectuée."

inotifywait -m -r -e modify,create,delete $config_dir |
while read path action file; do
    cp -r $backup_dir/* $config_dir/
    echo -e "Subject: Alerte Sécurité\n\nUne modification non autorisée a été détectée et restaurée." | msmtp $admin_email
    echo "Modification détectée et restaurée : $path$file"
done
```

**Lancer la surveillance :**
```bash
./backup_restore.sh &
```

---

### 📌 Conclusion
Ce projet permet de renforcer la sécurité d'un serveur Linux en surveillant les logs, en envoyant des alertes par e-mail et en assurant la restauration automatique des configurations critiques. 🚀

