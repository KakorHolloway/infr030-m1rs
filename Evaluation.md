# Evaluation

But de l'évaluation :

Un nouveau projet va démarrer, pour cela, ce dernier va avoir besoin de mettre en place un outil sur Kubernetes : Un GLPI

Afin de mettre ces outils en place voici ce qu'il va vous falloir créer :

Deux deployment d'un replica chacun. 
Un secret pour stocker le ou les mots de passe
Un deployment aura l'image harbor.kakor.ovh/public/glpi:latest (elestio/glpi hors harbor)
L'autre sera une base de donnée mariadb avec l'image harbor.kakor.ovh/public/mariadb:latest (mariadb hors harbor)
Pour contacter le pod mariadb, il faudra mettre en place un service écoutant le pod mariadb sur le port 3306.
Avec un ingress et le service lié à glpi, exposez l'application sur Internet. 
Dans le pod GLPI, changez via une configmap le logo de GLPI. 

# Rendu attentu 

Dans un zip, en rendu donnez-moi :

- Un schéma sur draw.io des composants installés
- Les fichiers yaml des composants installés sur Kubernetes (attention à la copie...)
- Un README.md donnant le nom des participants et une description des différents objets créés (par ex: on a un secret nommé michel qui permet de créer une variable d'environnement sur le conteneur x... En cas d'objet utilisés qui n'ont pas été vu en cours, justifiez en partant de la doc leurs utilisation. )
- Une capture d'écran de GLPI configuré

- Que pourrions nous faire pour améliorer le déploiement ? 