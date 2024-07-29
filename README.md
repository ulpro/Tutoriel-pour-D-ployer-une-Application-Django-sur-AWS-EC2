Voici un modèle complet de README pour déployer une application Django sur une instance Amazon EC2 avec Nginx et Gunicorn. Ce modèle inclut les instructions détaillées et les hypothèses faites :

---

# Tutoriel pour Déployer une Application Django sur AWS EC2

Si vous souhaitez mettre en ligne votre application Web Django et la rendre accessible au monde entier, le déploiement sur une instance Amazon Elastic Compute Cloud (EC2) est une solution à la fois économique et évolutive. Ce tutoriel vous guidera à travers les étapes nécessaires pour déployer une application Django sur EC2 et configurer Nginx comme serveur Web.

## Conditions Prérequises

Avant de commencer, assurez-vous que les éléments suivants sont en place :
1. **Instance AWS EC2** : Lancez une instance EC2 via la console de gestion AWS. Sélectionnez le système d'exploitation souhaité (dans ce guide, nous utilisons Ubuntu OS) et configurez vos groupes de sécurité pour autoriser le trafic entrant sur les ports 80 (HTTP) et 22 (SSH).
2. **Application Django** : Votre application Web Django doit être prête à être déployée. Assurez-vous qu'elle fonctionne correctement en local. Il est recommandé de transférer le projet sur GitHub pour un accès facile.

## I. Connexion à l'Instance EC2 via SSH

Pour accéder à votre instance EC2, utilisez la clé privée générée lors de la création de l'instance. Remplacez `/path/to/your/key.pem` par le chemin vers votre fichier de clé privée et `your_ec2_ip` par l'adresse IP publique ou le DNS de votre instance :

```bash
ssh -i /path/to/your/key.pem ubuntu@your_ec2_ip
```

Si vous n'avez pas configuré de clé privée, accédez à l'instance directement depuis le tableau de bord EC2.

## II. Mise à Jour du Système

Mettez à jour les packages de votre système :

```bash
sudo apt update
sudo apt upgrade
```

## III. Installation des Logiciels Requis

Installez les logiciels nécessaires : `python3-pip`, `python3-venv`, `nginx`, et `supervisor`. Démarrez le service Nginx s'il n'est pas déjà en cours d'exécution :

```bash
sudo apt install python3-pip python3-venv nginx supervisor
sudo service nginx start
```

## IV. Configuration de l'Environnement Virtuel

Il est recommandé de créer un environnement virtuel pour votre projet. Voici comment procéder depuis `/home/ubuntu` :

```bash
python3 -m venv alice
source alice/bin/activate
```

## V. Cloner/Télécharger les Fichiers du Projet

Si votre projet est disponible sur GitHub, clonez-le dans le répertoire `/home/ubuntu`. Vous pouvez également utiliser SCP pour transférer les fichiers si nécessaire :

```bash
cd /home/ubuntu
git clone https://github.com/yourusername/your-repository.git
```

## VI. Installation de Gunicorn et des Autres Dépendances

Installez les dépendances de votre projet et Gunicorn :

```bash
pip install -r requirements.txt
pip install gunicorn
```

## VII. Configuration de Gunicorn

Créez un fichier de configuration pour Gunicorn dans `/etc/supervisor/conf.d/` :

```bash
cd /etc/supervisor/conf.d/
sudo touch gunicorn.conf
sudo vim gunicorn.conf
```

Ajoutez le contenu suivant au fichier `gunicorn.conf` :

```ini
[program:gunicorn]
directory=/home/ubuntu/your-project-directory
command=/home/ubuntu/alice/bin/gunicorn --workers 3 --bind unix:/home/ubuntu/your-project-directory/spy.sock your_project.wsgi:application
autostart=true
autorestart=true
stderr_logfile=/var/log/gunicorn/gunicorn.err.log
stdout_logfile=/var/log/gunicorn/gunicorn.out.log

[group:guni]
programs=gunicorn
```

**Hypothèses :**
1. Votre projet est situé dans `/home/ubuntu/your-project-directory`.
2. Le fichier WSGI de votre projet est dans `/home/ubuntu/your-project-directory/`.
3. Votre environnement virtuel s'appelle `alice` et se trouve dans `/home/ubuntu/alice`.

Ajustez les chemins en fonction de votre configuration.

## VIII. Création des Dossiers pour les Logs

Créez les dossiers pour les fichiers journaux :

```bash
sudo mkdir /var/log/gunicorn
```

## IX. Mise à Jour du Service Supervisor

Pour connecter le service Nginx à Supervisor, exécutez les commandes suivantes :

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl status
```

Assurez-vous que le service Gunicorn est opérationnel. Si ce n'est pas le cas, consultez les journaux pour plus de détails.

## X. Mise à Jour du Fichier `nginx.conf`

Pour éviter des problèmes de permissions, modifiez le fichier `nginx.conf` pour utiliser l'utilisateur `root` au lieu de `www-data` :

```bash
sudo vi /etc/nginx/nginx.conf
```

Modifiez la ligne :

```nginx
user root;
```

## XI. Création du Fichier `django.conf` pour Nginx

Créez un fichier de configuration pour Nginx dans `/etc/nginx/sites-available/` :

```bash
cd /etc/nginx/sites-available/
sudo vi django.conf
```

Ajoutez le contenu suivant au fichier `django.conf` :

```nginx
server {
    listen 80;
    server_name 52.91.16.86;  # Remplacez par l'adresse IP ou le nom de domaine

    # Emplacement des fichiers statiques
    location /static/ {
        alias /home/ubuntu/your-project-directory/static/;
    }

    # Gestion des requêtes vers l'application Django via Gunicorn
    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/your-project-directory/spy.sock;
    }
}
```

Testez la configuration Nginx :

```bash
sudo nginx -t
```

Si le test est réussi, redémarrez Nginx :

```bash
sudo ln -s /etc/nginx/sites-available/django.conf /etc/nginx/sites-enabled/
sudo service nginx restart
```

## XII. Redémarrage des Services

Redémarrez les services pour appliquer les modifications :

```bash
sudo supervisorctl restart gunicorn
sudo systemctl restart nginx
```

Votre application Django devrait maintenant être accessible via l'adresse IP de votre instance EC2. Vous pouvez également configurer un nom de domaine pour accéder à votre application.

---

N'hésitez pas à adapter ce README en fonction des spécificités de votre projet et de votre environnement.
