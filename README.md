# 🌐 Projet AWS - Infrastructure Web + Base de Données

## 📌 Objectif

Mettre en place une infrastructure cloud avec :
- Un serveur Web Apache hébergeant un site PHP
- Un serveur MariaDB en réseau privé
- Des sauvegardes automatiques
- Un monitoring CloudWatch

---

## 📚 Sommaire

- [Schéma Réseau](#-schéma-réseau)
- [Justification des choix](#-justification-des-choix)
- [Étapes de mise en place](#-étapes-de-mise-en-place)
- [Commandes utiles](#-commandes-utiles)
- [Documentation d’exploitation](#-documentation-dexploitation)

---

## 🗺️ Schéma Réseau

> 📷 Voir `/captures/schema-reseau.png`

- VPC avec 2 subnets (Public et Privé)
- EC2 Web dans le subnet public (`10.0.1.0/24`)
- EC2 DB dans le subnet privé (`10.0.131.0/24`)
- Règles de sécurité :
  - Port HTTP (80) ouvert vers le WebServer
  - Port SSH (22) limité à mon IP uniquement
  - Port 3306 (MySQL) accessible uniquement depuis le WebServer

---

## 🧠 Justification des choix

| Élément           | Choix fait                           | Raison                                         |
|------------------|--------------------------------------|-----------------------------------------------|
| Base de données  | MariaDB                              | Léger, compatible MySQL, facile à déployer    |
| Architecture     | Subnet public (Web) + privé (DB)     | Sécurité réseau et isolation                  |
| Taille des EC2   | t2.micro                             | Inclus dans le tier gratuit AWS               |
| Sauvegarde       | `mysqldump` + cron                   | Simple, efficace pour un petit projet         |
| Supervision      | Agent CloudWatch                     | Monitoring CPU, RAM, disque                   |

---

## ⚙️ Étapes de mise en place

1. Création du VPC + subnets + routage
2. Déploiement des EC2 dans les bons subnets
3. Configuration Apache + PHP + MariaDB
4. Test de connexion Web ➝ DB
5. Mise en place de la sauvegarde automatique
6. Supervision via CloudWatch agent

---

## 💡 Commandes utiles

```bash
# Connexion DB distante
mysql -u webuser -p -h 10.0.131.7 BDD

# Test sauvegarde
mysqldump -u root -p BDD > /home/ubuntu/test-backup.sql

# Vérifier service CloudWatch
systemctl status amazon-cloudwatch-agent

# IP publique de l’instance
curl ifconfig.me

📘 Documentation d’exploitation

➕ Ajouter un site web :
sudo cp -r monsite/ /var/www/html/
sudo chown -R www-data:www-data /var/www/html/
sudo systemctl reload apache2

🗃️ Ajouter une base de données :
CREATE DATABASE nouvelle_db;
CREATE USER 'utilisateur'@'10.0.1.0' IDENTIFIED BY 'motdepasse';
GRANT ALL PRIVILEGES ON nouvelle_db.* TO 'utilisateur'@'10.0.1.0';
FLUSH PRIVILEGES;

♻️ Restaurer une BDD :
mysql -u root -p BDD < backup-2025-06-04.sql