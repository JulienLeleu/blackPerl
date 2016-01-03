Serveur web en Perl - Black Perl v1.0
======

Binôme
------

- Edouard CATTEZ
- Julien LELEU

Tâches accomplies
------
  - [X] Configuration
  - [X] Traitement des requetes GET
    - [X] Vers le système de fichiers
    - [X] Vers les programmes CGI
    - [ ] Paramètres des CGI
  - [X] Journal des événements
  - [X] Traitement des requêtes en parallèle
  - [X] Gestion des erreurs
    - [X] 200 OK
    - [X] 400 Bad Request
    - [X] 403 Forbidden
    - [X] 404 Not Found
    - [X] 405 Method Not Allowed
    - [X] 415 Unsupported Media Type
    - [X] 503 Service Unavailable
    - [X] 505 HTTP Version Not Supported


Détail des fonctionnalités
------
##Configuration
Le serveur BlackPerl charge la configuration du fichier 'comanche.conf'. Il y a possibilité dans ce fichier d'y insérer des commentaires avec le marqueur '#' en debut de phrase. Ces paramètres sont stockés dans une hashmap sous la forme : 
  - $conf{global}{...} pour les paramètres globaux tels que le port, le fichier par défaut, etc...
  - $conf{routes}{...} pour les paramètres de type "route <regex> to <regex>"
  - $conf{exec}{...} pour les paramètres de type "exec <regex> to <regex>"
La procédure est lancée au démarrage du serveur.

Les paramètres utilisables sont les suivants :
  - port : le numéro du port TCP sur lequel BlackPerl ecoute.
  - error : chemin de la page HTML par défaut.
  - index : nom de base des fichiers dont le contenu est retourné quand la ressource demandée correspond à un repertoire et qu'un tel fichier existe dans ce répertoire.
  - logfile : le chemin vers le journal
  - clients : le nombre maximal de clients

##Traitement des requetes GET
BlackPerl gère uniquement les requetes de type GET. Le systeme de route fait usage des expressions régulières.

### Système de fichiers
BlackPerl est totalement capable de retourner le contenu d'un fichier de type html, png ou texte en spécifiant le Content-Type. Dans le cas contraire, une erreur de type 415 (Unsupported Media Type) est renvoyée. Il gère également les repertoires en renvoyant le fichier index quand il est présent. Dans le cas contraire, il renvoie une liste HTML avec son contenu.

### CGI
Les CGI sont utilisées afin de dynamiser les sites web hebergés par BlackPerl. Elles les dynamisent notamment grâce à l'utilisation de langages de programmation tels que python, bash, perl etc...
[En savoir plus sur les CGI](http://www.ietf.org/rfc/rfc3875)

### Paramètres des CGI
Les paramètres permettent de dynamiser d'avantage le rendu des pages. Dans cette version ceux-ci ne sont pas pris en compte.

##Gestion des erreurs
BlackPerl respecte le protocole HTTP/1.1 défini dans la [RFC 2616](www.ietf.org/rfc/rfc2616.txt).
Il est capable de gérer ces 8 erreurs standards, stockées dans une hashmap pour éviter toute redondance de code :
 - `$HTTP{200} = 'OK';`
 - `$HTTP{400} = 'Bad Request';`
 - `$HTTP{403} = 'Forbidden';`
 - `$HTTP{404} = 'Not Found';`
 - `$HTTP{405} = 'Method Not Allowed';`
 - `$HTTP{415} = 'Unsupported Media Type';`
 - `$HTTP{503} = 'Service Unavailable';`
 - `$HTTP{505} = 'HTTP Version Not Supported';`

## Journal des événements
BlackPerl gére un fichier de log configurable dans le fichier `comanche.conf`.
Ce fichier (comanche.log) recense plusieurs actions : 
- Démarrage du serveur.
- Accès aux pages.
- Execution de CGI.
- Requêtes retournant une erreur 404.
- Arrêt du serveur.

Cependant, lorsque l'on stoppe le serveur avec `^C`, l'action n'est pas recensée dans ce fichier.

##Traitement des requêtes en parallèle
Le serveur BlackPerl gère bien évidemment les requetes multiples. Il est capable de desservir plusieurs clients simultanément grâce à l'utilisation d'un processus pour chaque client.


Fonctionnement
=============

##Lancement du serveur
--------------

Pour lancer le serveur, il suffit d'executer la commande suivante : `./comanche start`.
Celui-ci démarre si ce dernier n'est pas déjà lancé en vérifiant la présence du fichier caché qui contient son pid (sinon il le créé), puis charge la configuration (et échoue si un des paramètres de configuration est incorrect).

Ensuite, le serveur crée le fichier comanche.log.

##Status du serveur
--------------
Pour connaitre l'etat du serveur et obtenir des informations tel que son PID, le nombre de requetes recues et traitées, le nombre d'ouvriers actifs suivi de leurs PID, il est nécessaire d'executer la commande suivante : `./comanche status`.

##Fermeture du serveur
--------------

La fermeture peut-etre déclenchée par deux actions différentes : 
  - Soit en executant la commande `./comanche stop`
  - Soit à l'aide de `^C`
A la fermeture du serveur, celui-ci supprime le fichier caché contenant son pid pour le prochain lancement.