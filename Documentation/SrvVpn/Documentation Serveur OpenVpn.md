
Documentation - Mise en place d’un serveur VPN OpenVPN 


## Introduction
Pour permettre un accès sécurisé à distance au réseau de l’entreprise j’ai mis en place un serveur VPN OpenVPN 
Le VPN établit un tunnel chiffré entre le client et le serveur, pour donc garantir la confidentialité et l’intégrité des échanges


1. Installation des paquets nécessaires
Mettre à jour la liste des paquets et installer OpenVPN :

    sudo apt-get update
    sudo apt-get install curl

Il faut télécharger le script d'installation avec l'url suivante : 

    curl -O https://raw.githubusercontent.com/Angristan/openvpn-install/master/openvpn-install.sh

Il faut ajouter les droits d'exécution et exécuter le script pour commencer le configuration du server openvpn
    chmod +x openvpn-install.sh
    sudo ./openvpn-install.sh

Au lancement du script, un message de bienvenue s’affiche :
    "Welcome to the OpenVPN installer !"


2. Lancement du script d’installation

Le script commence par détecter automatiquement l’adresse IPv4 du serveur et propose  de valider cette adresse ou d’en préciser une autre, comme une adresse IP publique ou un nom de domaine

3. Paramètres réseau et options
    2.1 Support IPv6
        Le script propose d’activer ou non le support IPv6
        J’ai répondu non personnelement car j'en ai pas besoins 

    2.2 Port d’écoute
        Par défaut OpenVPN écoute sur le port 1194 UDP
        Pour masquer la présence du VPN il est possible de choisir un port personnalisé ce que j'ai fait et j'ai mis 44912, pour rendre la connexion plus discrète

    2.3 Choix du protocole
        Le script propose UDP (par défaut, plus rapide) ou TCP (plus compatible avec certains pare-feu)
        J’ai pris UDP

    2.4 Serveurs DNS
        Le script demande quel serveur DNS utiliser pour les clients VPN.
        J’ai choisi les serveurs Cloudflare 1.1.1.1 pour leur rapidité et respect de la vie privée

    2.5 Compression
        Pour des raisons de sécurité il est conseillé de ne pas utiliser la compression
         J’ai donc refusé la compression

    2.6 Paramètres de chiffrement
        Le script propose une configuration sécurisée par défaut avec possibilité de personnaliser les options
        J’ai conservé la configuration par défaut

Après ces choix le script procède à l’installation complète du serveur VPN

4. Création du premier profil client
    Le script propose de crée un client et créer un fichier qu'on peut protéger avec un mot de passe 
        Le certificat du client dans /etc/openvpn/easy-rsa/pki/issued/<nom du client>.crt
        La clé privée du client dans /etc/openvpn/easy-rsa/pki/private/<nom du client>.key

Puis pour rajouter de nouveaux clients il faut simplement relancer le script et choisir l'option d'ajout d'utilisateur

5. Si le serveur est derrière un routeur en NAT, il faut configurer la redirection du port UDP choisi (par exemple 44912) de l’adresse IP publique vers l’adresse locale du serveur VPN

6. Connexion au VPN depuis un client

    7.1 Transfert du fichier client
        Le fichier .ovpn est transféré sur la machine cliente via SCP, WinSCP ou autre

    7.2 Sur Windows
        Installer OpenVPN GUI ou OpenVPN Connect
        Copier le fichier .ovpn dans C:\Program Files\OpenVPN\config

Lancer le client OpenVPN, la nouvelle connexion apparaît
Cliquer sur Connecter et saisir le mot de passe si demandé

    7.3 Validation
Une fois connecté la machine cliente reçoit une adresse IP dans le sous-réseau 


