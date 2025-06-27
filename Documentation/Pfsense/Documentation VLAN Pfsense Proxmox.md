Documentation sur la mise en place des vlan avec Proxmox 


### Introduction
La segmentation réseau via des vlan est sert à organiser et sécuriser un réseau d’entreprise.
Elle permet de séparer logiquement différents types de trafic tout en utilisant la même infrastructure physique.

Dans ce projet, j’ai configuré plusieurs vlan sur pfsense, qui joue le rôle de routeur et pare-feu, en lien avec Proxmox, la plateforme de virtualisation.
Cette configuration garantit un réseau structuré, plus sûr et mieux adapté aux besoins spécifiques des différents services.


## 1. Préparation des interfaces sur pfsense
Sur pfsense, on dispose généralement de deux interfaces principales :

WAN, qui correspond à la connexion externe vers Internet.

LAN, qui sert au réseau interne.

Dans notre cas, l’interface LAN ne reçoit pas d’adresse IP directe, car elle sert de support aux vlan
Tous les vlan transitent par cette interface, qui transporte les trames vlan taguées

        - Important !!
        Si après avoir créé ou modifié les interfaces vlan, l’interface web de pfsense devient inaccessible, il est possible de désactiver temporairement le pare-feu depuis la console Proxmox en tapant : pfctl -d  
        ça permet de retrouver l’accès pour corriger la configuration


## 2. Création des vlan dans pfsense
    Cliquer sur Add pour créer un nouveau vlan
    Sélectionner l’interface physique (ici l’interface LAN sur laquelle les vlan vont transiter, par exemple vmbr1 dans Proxmox)
    Attribuer un tag VLAN unique qui identifie ce réseau (par exemple 10 pour le vlan Serveurs dans mon cas)
    Donner une description claire et explicite comme vlan serveurs pour faciliter la gestion ultérieure
    Enregistrer et répéter pour chaque vlan nécessaire

## 3. Assignation des interfaces vlan
    Une fois les vlan créés, ils doivent être ajoutés en tant qu’interfaces dans pfsense :
    Aller dans Interfaces > Interface Assignments
    Cliquer sur Add pour ajouter chaque vlan à la liste des interfaces
    Activer chacune des interfaces nouvellement créées en cochant la case Enable interface
    Donner un nom descriptif identique au vlan (exemple : vlan Serveurs)
    Configurer une adresse IP statique pour chaque interface selon le plan d’adressage défini 

## 4. Configuration des services DHCP
    Dans ce projet, la majorité des machines disposent d’adresses IP statiques, ce qui simplifie la gestion et évite les conflits
    Cependant pour certains vlan comme le vlan WiFi, le DHCP est activé pour permettre aux appareils de récupérer automatiquement une adresse IP donc :
    Se rendre dans Services > DHCP Server
    Sélectionner le vlan concerné
    Activer le serveur DHCP et définir la plage d’adresses IP disponible

## 5. Configuration réseau dans Proxmox

    Pour que les machines virtuelles soient correctement intégrées dans les vlan, la configuration dans Proxmox est la suivante :
    Choisir la carte réseau physique correspondant au LAN (par exemple vmbr1)
    Associer le tag vlan correspondant à la machine virtuelle (par exemple tag vlan 10 pour le vlan Serveurs)
    Configurer une adresse IP statique dans la machine virtuelle, en respectant la plage du vlan et en renseignant la passerelle adéquate


### Conclusion
La mise en place de vlan via pfsense et Proxmox permet une segmentation efficace du réseau, renforçant la sécurité et facilitant la gestion des flux.
Cette organisation évite les interférences entre services, limite les risques de propagation d’incidents, et offre une meilleure visibilité sur le trafic réseau.
