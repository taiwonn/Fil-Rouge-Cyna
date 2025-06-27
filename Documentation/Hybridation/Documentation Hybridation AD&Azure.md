# Documentation : Hybridation entre Active Directory local et Microsoft Entra ID (Azure AD)

---

## Introduction

L’hybridation entre un Active Directory local et Microsoft Entra ID permet de synchroniser les identités et de gérer les accès à la fois localement et dans le cloud; ça facilite l’administration des utilisateurs, la sécurisation des connexions, et l’accès aux services Microsoft365 ou autres applications cloud

Dans ce projet, j’ai mis en place cette hybridation en créant un tenant Microsoft Entra dédié, puis en installant et configurant Azure AD Connect pour synchroniser les comptes locaux vers le cloud


## 1. Création du tenant Microsoft Entra

Pour disposer d’un environnement cloud propre j’ai créé un tenant Microsoft Entra nommé *Cyna*

Ce tenant est indépendant du tenant de l’école auquel j’avais initialement accès
Cela permet de gérer uniquement les utilisateurs et ressources de l'organisation



## 2. Installation et configuration d’Azure AD Connect

### 2.1 Téléchargement et lancement

Depuis le portail Microsoft Entra j’ai téléchargé le programme d’installation d’Azure AD Connect
Je l’ai exécuté sur un serveur membre du domaine del' Active Directory local

### 2.2 Configuration de la synchronisation

Pendant l’installation, j’ai renseigné :  

- Les informations d’identification d’un compte administrateur local AD
- Les informations d’un compte administrateur dans le tenant Microsoft Entra *Cyna* 
- Le mode de synchronisation (synchronisation des identités, mots de passe, etc.) 
- Le filtrage pour limiter les objets synchronisés, si nécessaire

### 2.3 Lancement de la synchronisation

Une fois configuré, le service Azure AD Connect a commencé à synchroniser les utilisateurs, groupes et contacts de l’AD local vers le tenant cloud.


## 3. Avantages et fonctionnement

- Les utilisateurs peuvent utiliser leurs mêmes identifiants locaux pour se connecter aux services cloud.  
- La gestion des comptes est centralisée et simplifiée.  
- L’authentification peut être renforcée via des fonctionnalités comme la multi-authentification (MFA) dans Microsoft Entra.  
- L’accès aux applications cloud est sécurisé et fluide.


## Conclusion

Mettre en place l’hybridation entre Active Directory local et Microsoft Entra ID via Azure AD Connect est une étape clé pour moderniser la gestion des identités dans une organisation.  
Et ça offre une meilleure sécurité, une gestion simplifiée, et une expérience utilisateur cohérente entre local et cloud.
