# Installation d'un Chunkserver MooseFS

## Prérequis

* Système d'exploitation : Debian 12 ou Debian 13
* Accès administrateur (sudo ou root)
* Serveur MooseFS Master déjà fonctionnel et accessible
* Un disque dur dédié ou de l'espace disponible pour le stockage des chunks

> [!caution]
> Cette documentation a été testée et validée sur une machine virtuelle Proxmox sous Debian 12 et 13.  
> En cas de problème, vérifiez votre configuration réseau, DNS et vos disques.

> **Documentation Master Server :**  
> Pour l'installation du serveur Master, consultez : [https://github.com/Mayse-55/MooseFS-Master](https://github.com/Mayse-55/MooseFS-Master)

---

## 1. Extension de la partition root (optionnel)

Si vous utilisez Proxmox et nécessitez de l'espace supplémentaire :

```bash
lvremove /dev/pve/data
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root
```

---

## 2. Configuration des dépôts MooseFS

**Debian 12**
```bash
# mkdir -p /etc/apt/keyrings
# curl https://repository.moosefs.com/moosefs.key | gpg -o /etc/apt/keyrings/moosefs.gpg --dearmor

echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/moosefs.gpg] http://repository.moosefs.com/moosefs-4/apt/debian/bookworm bookworm main" > /etc/apt/sources.list.d/moosefs.list
```
**Debian 13**
```bash
sudo mkdir -p /etc/apt/keyrings
curl https://repository.moosefs.com/moosefs.key | \
  gpg -o /etc/apt/keyrings/moosefs.gpg --dearmor

echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/moosefs.gpg] http://repository.moosefs.com/moosefs-4/apt/debian/trixie trixie main" > /etc/apt/sources.list.d/moosefs.list
```

---

## 3. Mise à jour du système

```bash
sudo apt update
sudo apt dist-upgrade -y
sudo apt autoremove -y
```

---

## 4. Installation des paquets

```bash
sudo apt install -y moosefs-client moosefs-chunkserver moosefs-metalogger dfc
```

---

## 5. Préparation des répertoires

```bash
sudo mkdir -p /mnt/moosefs_chunks
sudo mkdir -p /mnt/moosefs_data
sudo mkdir -p /var/lib/mfs

sudo chown -R mfs:mfs /mnt/moosefs_chunks
sudo chown -R mfs:mfs /var/lib/mfs
```

---

## 6. Configuration du Chunkserver

### 6.1. Définition du stockage des chunks

```bash
sudo nano /etc/mfs/mfshdd.cfg
```

**Option A : Disque dédié (recommandé)**

```bash
/mnt/moosefs_chunks
```

MooseFS utilisera tout l'espace disponible avec une marge de sécurité par défaut.

**Option B : Disque partagé avec limitation**

```bash
/mnt/moosefs_chunks =100GiB
```

MooseFS limitera son utilisation à 100 GiB. Ajustez cette valeur selon vos besoins.

### 6.2. Vérification de la configuration

```bash
sudo nano /etc/mfs/mfschunkserver.cfg
```

Vérifiez les paramètres principaux :
- `WORKING_USER = mfs`
- `WORKING_GROUP = mfs`
- `DATA_PATH = /var/lib/mfs`

---

## 7. Configuration de la résolution DNS

```bash
sudo nano /etc/hosts
```

Ajout de l'entrée pour le Master Server :

```bash
# MooseFS Master Server
192.168.1.10    npx-1.lan npx-1 mfsmaster
```

> **Important :** Remplacez `192.168.1.10` par l'adresse IP réelle de votre serveur Master.

---

## 8. Configuration du montage automatique

```bash
sudo nano /etc/fstab
```

Ajout de la ligne de montage :

```bash
# MooseFS - Montage automatique
mfsmount    /mnt/moosefs_data    fuse    mfsmaster=mfsmaster,mfsport=9421,_netdev,nonempty    0 0
```

---

## 9. Activation et démarrage des services

### 9.1. Rechargement de la configuration systemd

```bash
sudo systemctl daemon-reload
```

### 9.2. Activation du Chunkserver

```bash
sudo systemctl enable moosefs-chunkserver.service
sudo systemctl start moosefs-chunkserver.service
sudo systemctl status moosefs-chunkserver.service
```

### 9.3. Activation du Metalogger (optionnel)

```bash
sudo systemctl enable moosefs-metalogger.service
sudo systemctl start moosefs-metalogger.service
sudo systemctl status moosefs-metalogger.service
```

---

## 10. Vérifications post-installation

### 10.1. Vérification du statut du Chunkserver

```bash
sudo mfschunkserver -v
sudo systemctl status moosefs-chunkserver
```

### 10.2. Vérification de la connexion au Master

```bash
mfscli -SCS
```

Cette commande affiche la liste des Chunkservers connectés au Master.

### 10.3. Vérification de l'espace disque

```bash
mfscli -SHD
df -h | grep moosefs
```

---

## 11. Montage du système de fichiers

### 11.1. Montage manuel (si nécessaire)

```bash
sudo mkdir -p /mnt/moosefs_data
sudo mount -t moosefs mfsmaster: /mnt/moosefs_data
```

### 11.2. Vérification du montage

```bash
df -h | grep moosefs
mount | grep moosefs
```

---

## 12. Commandes d'administration

### 12.1. Surveillance de l'état du serveur

```bash
# État général du Chunkserver
sudo systemctl status moosefs-chunkserver

# Informations détaillées sur les disques
mfscli -SHD

# Statistiques du cluster
mfscli -SIN
```

### 12.2. Gestion du service

```bash
# Redémarrage du Chunkserver
sudo systemctl restart moosefs-chunkserver

# Arrêt du Chunkserver
sudo systemctl stop moosefs-chunkserver

# Consultation des logs
sudo journalctl -u moosefs-chunkserver -f
```

---

## 13. Dépannage

### 13.1. Le Chunkserver ne se connecte pas au Master

Vérifiez :
- La résolution DNS du hostname `mfsmaster`
- La connectivité réseau vers le Master (port 9420, 9421, 9422)
- Les logs du Chunkserver : `sudo journalctl -u moosefs-chunkserver -n 50`

### 13.2. Espace disque non reconnu

Vérifiez :
- Les permissions du répertoire `/mnt/moosefs_chunks` (propriétaire mfs:mfs)
- La configuration dans `/etc/mfs/mfshdd.cfg`
- Les logs pour détecter les erreurs d'E/S

### 13.3. Problèmes de montage

Vérifiez :
- Le service `moosefs-client` est installé
- FUSE est disponible : `ls -l /dev/fuse`
- Le Master est accessible : `ping mfsmaster`

---

## 14. Recommandations

### 14.1. Production

Pour un environnement de production :
- Déployez au minimum 3 Chunkservers pour assurer la redondance
- Utilisez des disques dédiés pour le stockage des chunks
- Configurez un objectif de réplication minimum de 2 (deux copies par fichier)
- Surveillez régulièrement l'espace disque disponible

### 14.2. Performance

Pour optimiser les performances :
- Utilisez XFS comme système de fichiers pour les partitions de chunks
- Privilégiez des connexions réseau Gigabit Ethernet ou supérieures
- Évitez de stocker les chunks sur des partitions système
- Distribuez les Chunkservers sur différents serveurs physiques

### 14.3. Maintenance

- Surveillez régulièrement les logs système
- Vérifiez l'état de santé des disques (SMART)
- Planifiez des maintenances pour les mises à jour système
- Documentez la topologie de votre cluster

---

## 15. Ressources

- Site officiel : https://moosefs.com
- Documentation : https://moosefs.com/support
- Dépôt GitHub : https://github.com/moosefs/moosefs
- Support technique : support@moosefs.com

---

## 16. Informations légales

MooseFS est distribué sous licence GPL v2.

Copyright 2008-2025 Jakub Kruszona-Zawadzki, Saglabs SA
