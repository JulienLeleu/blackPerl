# Serveur web en Perl - Black Perl v1.0
### Edouard CATTEZ - Julien LELEU
-----

## Tâches accomplies
  - [X] Configuration
  - [X] Traitement des requêtes GET
    - [X] Vers le système de fichiers
    - [X] Vers les programmes CGI
    - [ ] Paramètres des CGI
  - [X] Journal des événements
  - [X] Traitement des requêtes en parallèle
  - [X] Gestion des erreurs

## Détail des fonctionnalités
------
### Configuration
Le serveur BlackPerl charge la configuration du fichier `comanche.conf`. Dans ce fichier, nous pouvons insérer des commentaires avec le marqueur '#' en debut de ligne. Les paramètres sont stockés dans une hashmap sous la forme :

  - $conf{sets}{...} pour les paramètres globaux tels que le port, le fichier par défaut, etc...
  - $conf{routes}{...} pour les paramètres de type "route <regex> to <regex>"
  - $conf{exec}{...} pour les paramètres de type "exec <regex> to <regex>"

La procédure est lancée au démarrage du serveur. Ci-dessous un exemple de fichier de configuration :

```
# Port d’écoute
set port 8080

# Page renvoyée par défaut
set error /var/www/index.html

# Fichier d’index dans les répertoires
set index index.html

# Nombre maximal de requêtes simultanées (>0)
set clients 10

# Journal des évènements
set logfile /var/log/comanche.log

# Routes de projection
route ^/(.*)$ to /var/www/\1
exec ^/(.*)\.exe(.*)$ from /var/lib/cgi/\1\2
```

### Traitement des requêtes GET
BlackPerl gère uniquement les requetes de type GET et répond au protocole HTTP/1.1 défini dans la [RFC 2616](www.ietf.org/rfc/rfc2616.txt). Il est capable de gérer les codes HTTP standards ci-dessous :
```
 - $HTTP{200} = 'OK';
 - $HTTP{400} = 'Bad Request';
 - $HTTP{403} = 'Forbidden';
 - $HTTP{404} = 'Not Found';
 - $HTTP{405} = 'Method Not Allowed';
 - $HTTP{415} = 'Unsupported Media Type';
 - $HTTP{503} = 'Service Unavailable';
 - $HTTP{505} = 'HTTP Version Not Supported';
```

### Système de fichiers
BlackPerl est capable de retourner le contenu d'un fichier de type html, png ou texte. L'erreur 415 (Unsupported Media Type) est renvoyée au client pour tout autre format. Il gère également les repertoires en renvoyant le fichier d'index par défaut quand il est présent. Dans le cas contraire, il renvoie une réprésentation HTML du répertoire avec une liste de son contenu.

### CGI
Les CGI sont utilisées afin de dynamiser les sites web. Elles les dynamisent sous la forme de scripts écrits dans des langages de programmation tels que python, bash, perl etc... [En savoir plus sur les CGI](http://www.ietf.org/rfc/rfc3875)

### Paramètres des CGI
Les paramètres permettent de dynamiser d'avantage le rendu des pages. Toutefois, dans cette version du serveur, ceux-ci ne sont pas pris en compte.

### Journal des événements
Une trace de l'activité sur le serveur est conservée dans un fichier de log définit dans le fichier de configuration. Nous sommes ainsi capable de connaître les activités ci-dessous :

- Démarrage du serveur
- Accès aux pages
- Execution de CGI
- Requêtes retournant une erreur 404
- Arrêt du serveur

[Bug] Lorsque l'on stoppe le serveur avec `^C` plutôt qu'avec la commande `./comanche stop`, l'arrêt du serveur n'est pas recensé dans ce fichier.

### Traitement des requêtes en parallèle
Le serveur BlackPerl est capable de desservir plusieurs clients simultanément grâce à l'utilisation d'un processus (ouvrier) pour chaque client.

## Fonctionnement
-----
### Lancement du serveur
Pour lancer le serveur, exécutez la commande `./comanche start`. Le serveur démarre si ce dernier n'est pas déjà lancé en vérifiant la présence du fichier caché `.serv_pid` qui contient son pid (sinon il le créé), puis charge la configuration. Si le chargement de la configuration échoue, le serveur s'arrête immédiatement.

### Statut du serveur
Pour connaitre l'état du serveur et obtenir des informations tel que son PID, le nombre de requêtes reçues et traitées, le nombre d'ouvriers actifs avec leur PID, exécutez la commande `./comanche status`.

### Fermeture du serveur
La fermeture peut-etre déclenchée par deux actions différentes :

- Soit en éxecutant la commande `./comanche stop`
- Soit en effectuant le contrôle clavier `^C`

A la fermeture du serveur, le ficher caché `.serv_pid` est supprimé.

### Tester le serveur
Afin de tester la robustesse du serveur, vous pouvez exécuter la commande `siege` après l'avoir préalablement installée depuis les packets linux.

```
# Exécute 100 clients en simultanés sur l'adrese localhost:8080 
siege -c 100 http://localhost:8080
```
