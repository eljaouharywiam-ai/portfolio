# Analyser un annuaire Active Directory via RSAT

## Contexte

Projet réalisé en laboratoire virtualisé : administrer un domaine Active Directory
existant depuis un poste client, sans ouvrir de session interactive sur le
contrôleur de domaine.

- **Domaine :** `isfo.com` — NetBIOS `ISFO` — DN `DC=isfo,DC=com`
- **Serveur :** `SRV` — Windows Server, contrôleur de domaine (AD DS) — `192.168.1.90`
- **Poste d'administration :** `DESKTOP-6VCDFG3` — Windows 10 — `192.168.1.20`,
  joint au domaine `isfo.com`

## Objectif

Installer les outils RSAT sur le poste Windows 10, explorer l'annuaire avec la
console ADUC (`dsa.msc`), analyser sa structure avec le module PowerShell
`ActiveDirectory`, puis produire un inventaire des objets du domaine exporté
en CSV.

RSAT (Remote Server Administration Tools) fournit les consoles et les modules
d'administration. Il n'installe pas le rôle AD DS et ne transforme pas le poste
en contrôleur de domaine.

## Étapes réalisées

### 1. Jonction du poste au domaine

Le poste Windows 10 a été joint au domaine `isfo.com` (`Add-Computer`),
avec le DNS pointant vers le contrôleur de domaine (`192.168.1.90`).
Vérification après redémarrage :

```powershell
(Get-WmiObject Win32_ComputerSystem).Domain
# isfo.com
```

Connexion au poste avec le compte de domaine `ISFO\Administrator`, puis
vérification de la connectivité : `ping isfo.com` résolu en `192.168.1.90`,
4/4, 0 % de perte.

### 2. Installation de RSAT sur Windows 10

Fonctionnalité facultative installée sur le poste :

> RSAT : outils Active Directory Domain Services et services LDS
> (Lightweight Directory Services)

Statut final : **Installé**.

### 3. Exploration de l'annuaire avec ADUC

Ouverture de la console depuis le poste d'administration :

```powershell
dsa.msc
```

Domaine `isfo.com` exploré dans ADUC : unités d'organisation, conteneurs
`Computers` et `Users`, utilisateurs et groupes. L'OU `ntic` contient les
sous-OU `tri` et `tmsir`, ainsi que l'utilisateur `TRI3`. L'OU `stagiaire`
contient notamment les utilisateurs `modele service` et `wiam eljhry`.

### 4. Analyse PowerShell

Module importé et vérifié :

```powershell
Import-Module ActiveDirectory
Get-Module ActiveDirectory
# Manifest 1.0.1.0 — ActiveDirectory
```

Informations du domaine :

```powershell
Get-ADDomain | Select-Object DNSRoot, NetBIOSName, DistinguishedName
# isfo.com | ISFO | DC=isfo,DC=com
```

Unités d'organisation :

```powershell
Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName
```

Utilisateurs, groupes, ordinateurs et contrôleurs de domaine :

```powershell
Get-ADUser -Filter * | Select-Object Name, SamAccountName
Get-ADGroup -Filter * | Select-Object Name, GroupCategory, GroupScope
Get-ADComputer -Filter * | Select-Object Name
Get-ADDomainController -Filter * | Select-Object HostName, Site
```

### 5. Export CSV

```powershell
Get-ADUser -Filter * | Select-Object Name, SamAccountName |
  Export-Csv "C:\AD_Users.csv" -NoTypeInformation -Encoding UTF8

Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName |
  Export-Csv "C:\AD_OU.csv" -NoTypeInformation -Encoding UTF8
```

Fichiers exportés dans le labo et vérifiés par `dir C:\AD*.csv` :
`AD_Users.csv` et `AD_OU.csv` (versions reconstituées en UTF-8 dans ce dossier).

## Résultats

| Élément | Résultat |
| --- | --- |
| Domaine | `isfo.com` (`ISFO`, `DC=isfo,DC=com`) |
| Unités d'organisation | 8 : Domain Controllers, stagiaire, ntic, tri (sous ntic), tmsir (sous ntic), formateurs, comapny, stagiaires |
| Conteneurs | Computers, Users, ForeignSecurityPrincipals, Managed Service Accounts |
| Utilisateurs | 12 (voir `AD_Users.csv`) |
| Groupes | Groupes par défaut du domaine, majoritairement Security / DomainLocal |
| Ordinateurs | 3 : SRV, PC-WIWI, DESKTOP-6VCDFG3 |
| Contrôleurs de domaine | 2 : SRV.isfo.com, pc-wiwi.isfo.com — site Default-First-Site-Name |

## Différence OU / conteneur

Une OU peut recevoir une GPO liée et des délégations d'administration, et
contenir des OU imbriquées. Un conteneur par défaut (Users, Computers) ne
peut pas recevoir de GPO liée. L'organisation en OU sert à structurer
l'annuaire par fonction (formation, formateurs, stagiaires) et à cibler
les stratégies de groupe.

## Fichiers

- `README.md` — ce document
- `AD_Users.csv` — inventaire des utilisateurs (Name, SamAccountName)
- `AD_OU.csv` — inventaire des unités d'organisation (Name, DistinguishedName)
- `rapport-tp-rsat-ad.docx` — rapport du TP pour le formateur (9 pages, captures des étapes réussies uniquement)
