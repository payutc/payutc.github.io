---
layout: default
title: Installation
section: install
---

# Pré-requis

* PHP (payutc est testé sur PHP 5.3 et PHP 5.4, la version 5.4 est recommandée pour les développeurs)
* Apache ou Nginx (ou un autre serveur web si vous savez le configurer)

# Installation du serveur

Par la suite, on suppose que tout sera installé dans `/var/www/payutc`.

Créer un fork de [payutc/server](https://github.com/payutc/server) en cliquant sur le bouton **Fork** dans Github puis le cloner en local dans `/var/www/payutc`:

    $ git clone git@github.com:*votre_username_github*/server.git

Ceci créé le dossier `server` avec le code dedans. Voir [git flow](git-flow) pour plus de détails sur l'utilisation de git.

payutc dépend de [ginger](https://github.com/simde-utc/ginger), l'outil cotisant du BDE, pour la récupération des informations sur les utilisateurs. Afin de faire des tests, on installe [faux-ginger](https://github.com/simde-utc/faux-ginger), qui expose une API similaire à ginger sans avoir à l'installer en entier. Toujours dans `/var/www/payutc` :
    
    $ git clone git@github.com:simde-utc/faux-ginger.git

Ensuite, aller dans `faux-ginger/users.php` et ajouter une entrée avec son login dans `$users` (le `badge_uid` doit être unique pour chaque utilisateur et peut être choisi au hasard).

# Installation des dépendances

L'outil utilise pour la gestion des dépendances est [Composer](http://getcomposer.org). Il faut l'installer dans le dossier `/var/www/payutc/server` :

    $ curl -sS https://getcomposer.org/installer | php

Ensuite, pour télécharger les dépendances :

    $ php composer.phar install

# Configuration du serveur

La configuration se fait dans le fichier `config.inc.php` (qui est donc dans le dossier `/var/www/payutc/server`). Le fichier `config.inc.dist.php` est fourni à titre d'exemple :

    $ cp config.inc.dist.php config.inc.php

Puis ouvrir ce fichier et changer :

* les paramètres de connexion à la base de données (la base doit être créée à la main, dans PHPMyAdmin par exemple)
* l'URL du CAS : pour utiliser le CAS de l'UTC, mettre `https://cas.utc.fr/cas/`. Pour pouvoir développer sans connexion Internet, installer [faux-cas](https://github.com/payutc/faux-cas)
* l'URL à laquelle le serveur est accessible en local (`http://localhost/payutc/server/web/` pour une configuration standard d'Apache)
* les paramètres de [Payline](http://www.payline.com/) devraient fonctionner, sinon il est possible d'ouvrir un compte de test gratuitement et immédiatement sur leur site
* ginger : pour faux-ginger, l'URL est de la forme `http://localhost/payutc/faux-ginger/index.php/v1/` et la clé `fauxginger`
* la variable `slim_config` définit si le serveur est en mode debug. Mettre `production` et `true` pour l'activer, `development` et `false` pour le désactiver. **Attention** : en mode debug, le serveur affiche une page HTML lorsqu'une exception est lancée, ce que les clients ne savent pas interpréter. Il faut donc activer le mode debug du serveur uniquement pour débuguer le serveur, pas les clients.
* la configuration des logs par défaut est suffisante.

Il faut créer le dossier pour les logs avec les droits corrects :

    $ mkdir logs
    $ chmod 777 logs

# Remplissage de la base de données

La structure de la base de données est gérée par le système de migrations. Pour l'initialiser à la dernière version, taper :

    $ php db.php migrations:migrate

Pour ajouter des données de test, faire :

    $ php db.php dbal:import dev_data.sql

Cette étape va aussi valider que la configuration de la base de données est correcte puisqu'elle utilise les données du fichier de configuration.

# Configuration du serveur web

On peut tester que tout fonctionne en allant par exemple sur [http://localhost/payutc/server/web/MYACCOUNT/getCasUrl](http://localhost/payutc/server/web/MYACCOUNT/getCasUrl)

## Apache

Pour Apache, la configuration par défaut (surtout dans notre cas où tout est installé dans `/var/www/payutc`) devrait être suffisante et il n'est pas obligatoire de rajouter un Virtual Host.

Afin de gérer les URL, on utilise le module `rewrite` d'Apache. La configuration est dans le fichier `web/.htaccess`. Si Apache renvoie des erreurs 404, c'est probablement que ce fichier n'est pas lu. Dans ce cas, il faut aller dans la configuration du Virtual Host (`/etc/apache2/sites-enabled/000-default` sous Ubuntu par défaut) et ajouter (ou remplacer) la ligne :

    <Directory /var/www>
    ...
    AllowOverride All
    ...
    </Directory>

Si Apache renvoie des erreurs 500 et indique une erreur dans le fichier `.htaccess`, c'est probablement que le module n'est pas chargé. Dans ce cas :

    $ sudo a2enmod rewrite

Ne pas oublier de redémarrer Apache après avoir modifié sa configuration ou activé le module.

## Nginx

Nginx ne supporte pas les fichiers `.htaccess`.