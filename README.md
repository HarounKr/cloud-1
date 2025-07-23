# Cloud-1

Ce projet a pour objectif de déployer une application web composée de plusieurs services (**WordPress**, **phpMyAdmin**, une **base de données** et **Nginx**) sur plusieurs serveurs distants à l’aide d’**Ansible** et **Docker**.

---

## Prérequis

- Une machine distante hébergée chez un fournisseur de cloud (AWS, Scaleway, OVH, etc.)
- **Docker** et **Docker Compose** installés sur la/les **machines distantes de déploiement**
- **Ansible** installé sur la **machine d’exécution**
- **Python** installé sur la/les **machines distantes de déploiement** (nécessaire pour Ansible)
- **Clé publique SSH autorisée** pour se connecter aux serveurs distants

---

## Ansible – Rôles, Templates et Vault

### Utilisation des rôles

Les **rôles Ansible** permettent d’organiser le projet en composants réutilisables, chacun responsable d’une tâche spécifique.

- Le rôle `create_ansible_user` gère :
  - La création d’un **utilisateur non-root**
  - L’**attribution des droits sudo**
  - La **configuration de l’accès SSH**

- Le rôle install_docker sert à installer docker, docker-compose
- Le rôle run_docker gère :
  - La **création des dossiers requis** pour le déploiement de l'application
  - La **génération des fichiers de configuration** nécessaires :
    - `docker-compose.yml.j2`
    - `wp-config.php.j2`
    - `app.conf.j2`
    - Script Python de lancement (`init-letsencrypt.py.j2`)
  - Tous ces fichiers sont produits dynamiquement grâce aux **templates Jinja2**
- Le rôle systemctl_config pour :
  - La génération du **fichier de configuration `systemd`**
  - L’activation du service pour garantir la **persistance** de l’application et son **lancement automatique au démarrage**


### Remarque

Les fichiers situés dans `vars/` de chaque rôle sont **chiffrés avec `ansible-vault`** car ils contiennent des **mots de passe** (ansible-vault encrypt roles/run_docker/vars/main.yml)
Il n’est donc **pas possible de lancer le projet** sans connaître la clé de chiffrement.
> Il est toutefois possible de **recréer les fichiers chiffrés à partir de zéro**,  
> en renseignant les bonnes valeurs, en vous référant aux **fichiers templates `.j2`**  
> (comme `wp-config.php.j2` ou `docker-compose.yml.j2` par exemple) pour savoir quelles variables sont nécessaires.

En temps normal, le projet peut être lancer en étant à la racine avec la commande :
```bash
ansible-playbook playbook.yml --ask-vault-pass
```
