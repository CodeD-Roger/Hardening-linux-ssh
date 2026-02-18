# 🔐 Script de Durcissement de Sécurité Linux

Script Bash automatisé de durcissement de sécurité pour serveurs Debian/Ubuntu — idéal pour les VPS, serveurs dédiés et machines virtuelles.

---

## À quoi sert ce script ?

Ce script applique une série complète de mesures de sécurité sur un système Debian/Ubuntu. Il crée automatiquement un utilisateur administrateur sécurisé si aucun n'existe, configure SSH avec authentification par clé, durcit les paramètres du noyau (sysctl), installe et configure Fail2Ban contre les attaques par force brute, initialise AIDE pour la surveillance de l'intégrité des fichiers, lance RKHunter pour la détection de rootkits, applique des politiques de mots de passe strictes, désactive les services et modules noyau inutiles, et génère des logs détaillés de chaque action effectuée. Le script est conçu pour être relancé en toute sécurité et sauvegarde chaque fichier de configuration avant de le modifier.

> **Prérequis :** Doit être exécuté en tant que root sur un système Debian/Ubuntu. Pour les serveurs distants, une clé SSH doit être configurée pour root (ou utiliser le flag `--local-vm`).

---

## 🚀 Commandes disponibles

### Mode interactif (recommandé pour une première utilisation)

```bash
sudo bash hardening_script.sh
```
Lance le script en mode interactif. Le script pose des questions au fil de l'exécution, notamment si vous souhaitez désactiver la connexion SSH en tant que root. Un utilisateur sudo sécurisé est créé automatiquement s'il n'en existe aucun, avec un nom d'utilisateur aléatoire et un mot de passe fort de 20 caractères. Une pause de 60 secondes est accordée pour tester le nouveau compte dans un terminal séparé avant toute modification SSH.

---

### Mode automatique — désactivation de root SSH

```bash
sudo bash hardening_script.sh --disable-root-login
```
Lance le script en mode entièrement automatique et désactive la connexion SSH en tant que root sans aucune interaction. Idéal pour les déploiements automatisés ou les pipelines CI/CD. Une pause de 10 secondes est accordée avant la désactivation pour permettre une interruption d'urgence via `Ctrl+C`.

---

### Mode automatique — conservation de root SSH

```bash
sudo bash hardening_script.sh --keep-root-login
```
Lance le script en mode entièrement automatique tout en conservant la connexion SSH en tant que root activée. Un utilisateur sudo sécurisé est quand même créé. À utiliser si vous n'êtes pas encore prêt à désactiver root, mais souhaitez tout de même appliquer le durcissement complet du système.

---

### Mode VM locale / environnement de test

```bash
sudo bash hardening_script.sh --local-vm
```
Lance le script sans vérifier la présence de clés SSH pour root. Conçu pour les machines virtuelles locales, les environnements de test ou les machines sans accès SSH distant. L'authentification par mot de passe reste activée dans ce mode.

---

### Ignorer la création d'utilisateur

```bash
sudo bash hardening_script.sh --skip-user-creation
```
Lance le script en ignorant l'étape de création d'un utilisateur sudo. À utiliser uniquement si un compte sudo non-root existe déjà sur le système. **Attention :** si aucun utilisateur sudo n'existe et que la connexion root SSH est désactivée, vous risquez d'être définitivement bloqué hors de votre serveur.

---

### Activer le verrouillage de compte PAM

```bash
sudo bash hardening_script.sh --enable-pam-lockout
```
Active le verrouillage automatique des comptes après 10 tentatives de connexion échouées (durée : 5 minutes). **Désactivé par défaut** car il peut provoquer des blocages accidentels. Pour déverrouiller un compte manuellement : `faillock --user NOM_UTILISATEUR --reset`. Fail2Ban (activé par défaut) offre une protection équivalente pour SSH sans ce risque.

---

### Combinaisons utiles

```bash
# VPS en production : automatique, root désactivé
sudo bash hardening_script.sh --disable-root-login

# VM de test : pas de clé SSH requise, root conservé
sudo bash hardening_script.sh --local-vm --keep-root-login

# Serveur avec utilisateur existant, sécurisation maximale automatique
sudo bash hardening_script.sh --skip-user-creation --disable-root-login

# Test avec verrouillage PAM activé, root conservé
sudo bash hardening_script.sh --keep-root-login --enable-pam-lockout
```

---

## 🛡️ Ce que le script applique

| Domaine | Actions |
|---|---|
| **SSH** | Authentification par clé, désactivation de root (optionnel), timeout, protocole 2, bannière légale |
| **Fail2Ban** | Protection SSH contre les attaques par force brute, bannissement après 5 tentatives |
| **Noyau (sysctl)** | Désactivation du forwarding IP, cookies SYN TCP, filtrage de paquets, restriction dmesg/kptr |
| **AIDE** | Initialisation de la base de référence pour la surveillance de l'intégrité des fichiers |
| **RKHunter** | Scan de rootkits et mise à jour de la base de données |
| **Auditd** | Journalisation des modifications de /etc/passwd, shadow, sudoers, modules noyau |
| **Mots de passe** | Politique d'expiration (90 jours), complexité minimale (14 caractères), SHA512 |
| **Modules noyau** | Désactivation des protocoles réseau inutilisés (dccp, sctp, rds, tipc) et systèmes de fichiers |
| **Permissions** | Durcissement des permissions sur /etc/passwd, shadow, group, crontab, clés SSH |
| **Services** | Désactivation des services inutiles (avahi, cups, rpcbind, snmpd, etc.) |
| **AppArmor** | Activation et application de profils de sécurité |
| **Umask** | Restriction des permissions par défaut à 027 |

---

## 📋 Logs générés

Tous les logs sont sauvegardés dans `/var/log/hardening/` :

```
/var/log/hardening/
├── main/
│   ├── execution.log       # Log complet de l'exécution
│   └── summary.log         # Résumé des étapes
├── tools/
│   ├── lynis.log           # Rapport Lynis (avant/après)
│   ├── rkhunter.log        # Résultats du scan RKHunter
│   ├── aide.log            # Initialisation AIDE
│   ├── fail2ban.log        # Configuration Fail2Ban
│   └── auditd.log          # Chargement des règles auditd
├── auto-fixes/
│   └── remediation.log     # Liste de toutes les corrections appliquées
├── configs/
│   └── backups/            # Sauvegardes de tous les fichiers modifiés
└── IMPORTANT_CREDENTIALS.txt  # Identifiants du nouvel utilisateur (à supprimer après usage !)
```

---

## ⚠️ Notes importantes

- **Toujours tester** la connexion avec le nouvel utilisateur dans un terminal séparé avant de fermer la session root.
- Les identifiants du nouvel utilisateur sont sauvegardés dans `/var/log/hardening/IMPORTANT_CREDENTIALS.txt` — **copiez ce fichier et supprimez-le après usage**.
- Le script peut être relancé plusieurs fois en toute sécurité : les anciens logs sont archivés automatiquement.
- Sur les systèmes sans clé SSH configurée pour root, utilisez obligatoirement le flag `--local-vm`.
