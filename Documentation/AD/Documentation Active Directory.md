Documentation - Active Directory ( GPO, MDP, UTILISATEURS, OU)


Comment mettre en place un Active Directory


## Introduction 
Active Directory est un service indispensable pour gérer de façon centralisée les utilisateurs, ordinateurs et ressources d’un réseau d’entreprise.
L’objectif de cette installation est d’avoir un annuaire structuré, sécurisé et facile à administrer, permettant d’appliquer des règles uniformes et donc de contrôler les accès.

Dans ce document je détaille toutes les étapes que j’ai suivies pour installer et configurer un AD, depuis l’installation du rôle jusqu’à la mise en place des politiques de sécurité.
Je donne aussi des exemples concrets basés sur mon propre environnement, tout en expliquant les choix techniques faits.




## 1. Installation du rôle Active Directory
La première étape consiste à installer le rôle Active Directory Domain Services sur un serveur Windows.
Ce rôle permet de transformer le serveur en contrôleur de domaine, c’est-à-dire le serveur qui gère l’annuaire et l’authentification.

Pour cela, j’ai utilisé le Gestionnaire de serveur pour ajouter un nouveau rôle, puis j’ai sélectionné AD DS.


## 2. Promotion du serveur en contrôleur de domaine
Comme il s’agissait de la première installation, j’ai choisi de créer une nouvelle forêt.
J’ai donné comme nom de domaine EntrepriseCyna.local. Ce nom interne évite les conflits avec des noms publics.

Il a fallut définir le Windows serveur pour bénéficier de fonctionnalités modernes tout en assurant une bonne compatibilité.
Un mot de passe a été défini pour le mode de restauration des services d’annuaire (DSRM), nécessaire en cas de dépannage.

Après validation le serveur a redémarré et est devenu contrôleur de domaine.

# 3. Configuration du DNS
Le DNS est essentiel pour qu’Active Directory fonctionne correctement, car il permet aux machines du réseau de localiser les contrôleurs de domaine.

Le service DNS s’est installé automatiquement lors de la promotion du serveur.
Une zone DNS correspondant au domaine EntrepriseCyna.local a été créée.

J’ai vérifié la résolution DNS avec la commande nslookup sur le serveur et depuis des postes clients.





# 4. Organiser l’annuaire avec des OU

Pour que ce soit simple à gérer, j’ai créé des *Unités Organisationnelles*.  
Ça permet de ranger les utilisateurs et les ordinateurs par groupe logique.

Par exemple j'ai mis en place les OU configurations suivantes  :  
- Une OU pour Genève, avec sous-OU “utilisateurs” et “ordinateurs”.  
- Une OU pour Paris, avec des sous-OU pour les commerciaux et les sédentaires.  
- Une OU “Sécurité” pour les groupes d’administrateurs qui ont des droits spéciaux.

Cette organisation aide à appliquer des règles différentes selon les besoins et ça évite que tout soit mélangé.


# 5. Faire rejoindre les serveurs au domaine

Pour que les serveurs puissent appliquer les règles définies dans Active Directory et que les utilisateurs puissent s’authentifier avec leurs comptes centralisés, il faut que ces serveurs deviennent membres du domaine.  

## 5.1 Sous Windows

Voici les étapes à suivre pour joindre un serveur Windows au domaine :

1. Ouvrir les *Propriétés système* :  
   - Cliquer sur *Domaine ou groupe de travail* 
   - Dans l’onglet *Nom de l’ordinateur* cliquer sur *Modifier...*

2. Changer l’appartenance :  
   - Sélectionner *Domaine* au lieu de *Groupe de travail*
   - Entret le nom du domaine, ici *EntrepriseCyna.local*
   - Cliquet sur OK

3. Saisis les identifiants d’un compte administrateur du domaine pour valider

4. Redémarrer la machine quand c’est demandé

Une fois redémarré, le serveur fait officiellement partie du domaine et applique les politiques associées.


## 5.2 Sous Linux (Debian/Ubuntu)

Pour joindre un serveur Linux au domaine Active Directory, on utilise généralement l’outil `realmd` qui simplifie les choses. Voici la démarche que j’ai suivie :  

- 1. Installer les paquets nécessaires :

```bash
sudo apt update
sudo apt install realmd sssd sssd-tools adcli samba-common-bin packagekit
```
- 2. Vérifier que le domaine est accessible via DNS
Avant de joindre le domaine, il faut s’assurer que le serveur peut résoudre le nom du domaine de  l'AD :

```bash
nslookup EntrepriseCyna.local
```
- 3. Découvrir le domaine

```bash
sudo realm discover EntrepriseCyna.local
```

- 4. Joindre le domaine

Pour joindre le domaine, utilise la commande suivante (remplace Administrateur par ton compte administrateur AD) :

```bash
sudo realm join --user=Administrator EntrepriseCyna.local
```
Tu seras invité à saisir le mot de passe de ce compte

- 5. Vérifier l’adhésion au domaine
Pour vérifier que la machine est bien membre du domaine, lance :

```bash
realm list
```
- 6. Vérifier depuis l'AD si l'hadésion à bien été prise en compte dans Computers 




## 6. Sécuriser avec des GPO

Pour que tout soit sécurisé, j’ai configuré des GPO qu’on applique partout

# 6.1 Sur les mots de passe

- Longueur minimum de 12 caractères, pour que ce soit costaud
- Complexité activée : majuscules, chiffres, caractères spéciaux, pour éviter les mots trop simples 
- Historique sur 24 mots de passe pour que personne ne réutilise toujours les mêmes 
- Durée de vie de 6 mois, pour forcer à changer régulièrement

Ces réglages viennent des bonnes pratiques de sécurité, ils sont basés sur les recommandantions de l'ANSSI 

## 6.2 Sur le verrouillage des comptes

Pour éviter les attaques par devinette de mot de passe, j’ai mis un verrouillage après 5 tentatives ratées, pendant 15 minutes

## 6.3 D’autres règles

- Interdire l’accès à l’invite de commandes et PowerShell, histoire d’empêcher des commandes dangereuses sur les postes 
- Bloquer le panneau de configuration pour éviter que les utilisateurs touchent aux réglages système
- Bloquer le Microsoft Store pour limiter l’installation d’applications non autorisées
- Verrouiller la session automatiquement au bout de 10 minutes d’inactivité



## Conclusion

Avec cette config, on a un Active Directory organisé, sécurisé, et prêt à gérer les utilisateurs et machines.  
Les règles sur les mots de passe et le verrouillage protègent contre les attaques, et l’organisation en OU permet d’appliquer les bonnes politiques aux bons groupes.


