Serveur web en Perl - Black Perl v1.0
======

Binôme
------

- Edouard CATTEZ
- Julien LELEU

Tâches accomplies
------
  - [X] gestion du port d'écoute
  - [X] protocole HTTP/1.1
  - [X] gestion de la page par défaut
  - [X] gestion des fichiers index dans les répertoires
  - [X] gestion des logs
  - [X] gestion des clients en //
  - [X] gestion du max de clients
  - [X] routes statiques
  - [X] routes avec expression régulières
  - [X] cgi statiques
  - [X] cgi avec expression régulières
  - [ ] paramtres de cgi

Détail
------
### Gestion du port d'coute
Comanche est capable d'écouter sur le port configuré dans le fichier de configuration. Cependant si la configuration est rechargée avec la commande `perl comanche reload`, la mise  jour du port d'écoute ne s'effectue pas correctement.

### protocole HTTP/1.1
Comanche répond uniquement en respectant le protocole HTTP/1.1 défini dans la [RFC 2616](www.ietf.org/rfc/rfc2616.txt). Il est capable de répondre  trois erreurs standards : 
- 404 : Page non trouvée.
- 400 : Protocole non respecté.
- 501 : Service non disponible.

### Gestion de la page par défaut
Comanche utilise la page par défaut comme page  renvoyée si il y a une erreur de type 404. En effet, le client doit être informé que la page demandée n'existe pas à travers un message d'erreur ainsi qu'un header contenant le code "404 Not found"

### Gestion des logs
Comanche est capable de gérer un fichier de log configurable dans le fichier de configuration de comanche (comanche.conf). Ce fichier de log recense plusieurs actions : 
- Démarrage du serveur.
- Accès aux pages.
- Execution de CGI.
- Requêtes retournant une erreur 404.
- Arrêt du serveur.

### Gestion des clients en parallèle
Comanche est capable de desservir plusieurs clients en simultané grâce à l'utilisation d'un fork pour chaque client qui se connecte.

### Gestion du max de clients
Comanche est capable aussi grâce au fichier de configuration de limiter le nombre de clients qui font une requête en simultané. Ce nombre de clients est configurable via le fichier de configuration.

### Routes statiques
Comanche gère un systeme de "routing" permettant la réécriture d'url comme sous apache2 via le module rewrite.

### Routes avec expression régulière
Afin de dynamiser le système de réécriture d'url, un systme utilsant les expressions régulières a été mis en place.

### CGI statiques
Les CGI sont utilisées afin de dynamiser les sites web que peux heberger comanche. Ce système permet de dynamiser les sites web grâce à l'utilisation de langages de programmation tels que python, bash, perl etc...

### CGI avec expressions régulires
L'utilisation des expressions régulières dans le paramétrages des cgi permet de généraliser les url vers l'execution des scripts. Cette pratique permet tout comme pour les routes de simplifier l'accès à celles-ci.

### Paramtres de cgi
Les paramètres permettent de dynamiser d'avantage le rendu des pages. Malheureusement, dans cette version les paramètres des cgi ne sont pas pris en compte. [En savoir plus sur les CGI](http://www.ietf.org/rfc/rfc3875)


Développement
=============

Implémentation
--------------

### Détermination du type du fichier à utiliser
Afin de répondre correctement au protocole HTTP/1.1, comanche doit être capable de determiner le type mime de la ressource qui lui est demandé. Lors de l'implémentation, plusieurs méthodes peuvent être utilisées afin de répondre à cette demande : 
- l'utilisation de modules permettant de determiner le type mime est une solution qui peut être envisagée mais cependant elle oblige l'utilisateur de comanche d'installer le module en question.
- Une autre solution aurait été d'utiliser le fichier /etc/mime.types, cependant l'utilisation de ce fichier limite l'execution du serveur sur un systeme unix.
- La dernière solution qui a donc été choisie était de determiner le type mime à partir de l'extention de la ressource. Dans le projet il y a donc une correspondance qui s'effectue entre l'extension et le mimetype écrit directement dans le code de comanche. 


### Accès aux processus
L'accès aux processus se fait à l'aide de la comande `fork`. Le script initialise le serveur dans un processus fils afin de rendre la main directement à l'utilisateur. Ensuite afin d'informer le serveur que l'on veut effectuer des actions, on utilise 4 signaux :
-    $SIG{CHLD} : permettant d'indiquer au proccesus père (le serveur) la mort d'un fils (d'un client) 
-    $SIG{USR1} : permet d'obtenir les informations sur le serveur
-    $SIG{QUIT} : permet d'indiquer au serveur qu'il doit s'arrêter
-    $SIG{HUP}  : permet de recharger la configuration du serveur. 

### Fichier de configuration
Le fichier de configuration est lu par comanche afin de régler quelques variables comme par exemple le port d'écoute du serveur. Ces paramètres sont stockés dans un hash sous la forme : 
- $conf{global}{...} pour les paramètres globaux tels que le port, le fichier par défaut, etc...
- $conf{routes}{...} pour les paramètres de type "route ma_route to dossier_dest"
- $conf{exec}{...} pour les paramètres de type "exec unCGI from un_dossier"
La procédure est lancée au demarrage du serveur ainsi lorsque l'on demande au serveur de recharger la configuration via la commande `./comanche reload`
