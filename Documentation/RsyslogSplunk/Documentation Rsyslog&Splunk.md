Documentation : Mise en place de la collecte de logs avec Rsyslog et Splunk


# Introduction

Centraliser les logs est indispensable pour bien surveiller une infrastructure
Au lieu d’avoir les événements dispersé sur chaque machine, on rassemble tout en un seul endroit. Ça facilite l’analyse, la détection rapide des problèmes, et la mise en place d’alertes

Dans cette doc je vais expliquer comment mettre en place cette collecte avec rsyslog pour récupérer et transférer les logs, et pour splunk les visualiser et générer des alertes



## 1. Rsyslog : collecte et transfert des logs

### 1.1 Comment ça marche ?

Rsyslog c'est un service présent sur la plupart des systèmes Linux
Il récupère les logs  et peut les envoyer vers un serveur distant

### 1.2 Configuration côté serveur client

Sur chaque serveur, il faut dire à rsyslog d’envoyer ses logs vers le serveur central
Pour ça, on crée ou modifie un fichier de config, par exemple `/etc/rsyslog.d/60-forward.conf`, en ajoutant cette ligne :

*.* @@172.16.3.6:1514

Donc pour expliquer ça : 
- `*.*` veut dire tous les logs sur tout.  
- `@@` indique qu’on utilise le protocole TCP, plus fiable que UDP ( upd c'est @ ) 
- `17.16.3.6` correspond à l’adresse de notre serveur rsyslog 
- Le port `1514` est celui choisi pour la collecte 


### 1.3 Configuration côté serveur rsyslog central

Ce serveur doit être capable de recevoir les logs
On active donc l’écoute TCP sur le port 1514 en ajoutant cette config dans `/etc/rsyslog.conf` ou dans un fichier dédié :

    module(load="imtcp")
    input(type="imtcp" port="1514")

Il faut également configurer l'envoie des logs vers splunk et se rendre dans 
        /etc/rsyslog.d/forward-to-splunk.conf
Pour y mettre 
        *.* @@172.16.3.7:1514

L'ip 172.16.3.7 représente l'ip du Splunk 


Ensuite on redémarre le service :

    sudo systemctl restart rsyslog



# 2. Installation de Splunk Enterprise

    2.1 Téléchargement
        Crée un compte sur le site officiel  de splunk et télécharger la dernière version 

    2.2 Installation sur le serveur dédié à splunk 
        cd /tmp
    wget -O splunk.tgz "url de téléchargement pris au préalable"
    sudo tar -xvzf splunk.tgz -C /opt

    2.3 Premier démarrage
        sudo /opt/splunk/bin/splunk start --accept-license

Créer un compte administrateur pour accéder à l’interface web (généralement disponible sur le port 8000 et l'url s'affichera)

    2.4 Démarrage automatique au boot
    Pour ce faciliter la tâche on peut configurer Splunk pour qu’il démarre automatiquement :
        sudo /opt/splunk/bin/splunk enable boot-start



3.  Splunk : réception, analyse des logs
    32.1 Mise en place de la réception dans Splunk
        Dans Splunk on configure une entrée TCP sur le port 1514 pour écouter les logs transmis par le serveur rsyslog 
        Cette étape se fait via l’interface graphique sous Settings > Data Inputs > TCP

    3.2 Indexation et recherche
        Une fois reçus, les logs sont indexés dans un index dédié (par exemple syslog), ce qui permet de les rechercher rapidement

    3.3 Création d’un dashboard simple
        J’ai créé un dashboard dans Splunk pour visualiser les connexions SSH, par exemple
        La requête utilisée ressemble à ça :

            index=* sourcetype=syslog "sshd"

        Cette recherche récupère tous les événements liés au service SSH, qu’ils soient réussis ou non



    3.4  Configuration d’une alerte
        Pour être alerté en cas de problème, j’ai mis en place une alerte suite à une recherche pour des erreurs avec le mot clé error dedans 
        Il faut, pour crée une alerte, aller dans recherche mettre : 
                index=* "error"
        Puis enregistrer en tant qu'alerte en choisissant la fréquence d'exécution de l'alerte puis aussi configurer les action d'alerte donc le mail.
        Puis, l’alerte envoie un mail à l’équipe pour qu’elle puisse intervenir rapidement


4. Résumé du fonctionnement global
Les serveurs clients envoient leurs logs au serveur rsyslog central via TCP port 1514
Le serveur rsyslog central et relaie ces logs vers Splunk, toujours via TCP port 1514

Splunk reçoit, indexe, affiche et alerte sur ces données

# Conclusion

Avec cette configuration, on obtient une collecte fiable et centralisée des logs, une analyse en temps réel dans Splunk, et des alertes pour ne rien manquer
C’est une base solide pour la supervision et la sécurité d’une infrastructure