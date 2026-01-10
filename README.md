# 🐘 Installation d'un Chunkserver MooseFS
[![MooseFS](https://img.shields.io/badge/MooseFS-Distributed%20FS-red?style=flat-square&logo=linux)](https://moosefs.com/)

## 🧾 Prérequis

* 🖥️ Système : **Debian 12 et Debian 13**
* 🔐 Accès `sudo` ou root
* 🧠 Serveur MooseFS Master déjà fonctionnel
* 💽 Un disque dur supplémentaire pour le stockage des chunks

> [!caution]
> ✅ Cette documentation a été **testée et validée** sur une machine virtuelle Proxmox sous **Debian 12**.  
> ❌ Si vous rencontrez des problèmes, vérifiez votre configuration réseau, DNS et vos disques.

---

## 🧹 (Optionnel) Étendre la partition root (si nécessaire)

```bash
lvremove /dev/pve/data
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root
```

## 🐂 Ajouter les dépôts MooseFS

```bash
sudo mkdir -p /etc/apt/keyrings
curl https://repository.moosefs.com/moosefs.key | \
  gpg -o /etc/apt/keyrings/moosefs.gpg --dearmor

echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/moosefs.gpg] http://repository.moosefs.com/moosefs-4/apt/debian/bookworm bookworm main" | \
  sudo tee /etc/apt/sources.list.d/moosefs.list
```

## 🔄 Mise à jour du système

```bash
sudo apt update
sudo apt dist-upgrade -y
sudo apt autoremove -y
```

## 📦 Installation des paquets nécessaires

```bash
sudo apt install -y moosefs-client moosefs-chunkserver moosefs-metalogger dfc
```

## 📁 Préparer les répertoires de données

```bash
sudo mkdir -p /mnt/moosefs_chunks
sudo mkdir -p /mnt/moosefs_data
sudo mkdir -p /var/lib/mfs

sudo chown -R mfs:mfs /mnt/moosefs_chunks
sudo chown -R mfs:mfs /var/lib/mfs
```

## 🛠️ Configuration des fichiers MooseFS

### 📌 1. Définir le disque des chunks

```bash
sudo nano /etc/mfs/mfshdd.cfg
```

### Pour un disque vide (dédié entièrement à MooseFS)

```bash
/mnt/moosefs_chunks
```

**MooseFS** utilisera tout l'espace disponible sur le disque, moins l'espace de sécurité par défaut.

### Ou Pour un disque avec système (partage avec d'autres données)

```bash
/mnt/moosefs_chunks =100GiB 

# 100GiB dépend de votre configuration matérielle et de vos besoins.
```

**MooseFS** utilisera 100GiB sur ce disque, laissant le reste pour le système ou d'autres applications.

### 📌 2. Vérifier la configuration du chunkserver

```bash
sudo nano /etc/mfs/mfschunkserver.cfg
```

## 📇 Configurer le fichier /etc/hosts

```bash
sudo nano /etc/hosts
```

Ajouter par exemple :

```bash
192.168.1.10 npx-1.lan npx-1 mfsmaster
```

## 🔁 Monter automatiquement MooseFS au démarrage

```bash
sudo nano /etc/fstab
```

Ajouter au début : 

```bash
#MooseFS
mfsmount        /mnt/moosefs_data   fuse    mfsmaster=mfsmaster,mfsport=9421,_netdev,nonempty   0 0
```

## 🚀 Activer et démarrer les services

```bash
sudo systemctl daemon-reload

# Chunkserver
sudo systemctl enable moosefs-chunkserver.service
sudo systemctl start moosefs-chunkserver.service
sudo systemctl status moosefs-chunkserver.service

# Metalogger (optionnel)
sudo systemctl enable moosefs-metalogger
sudo systemctl start moosefs-metalogger
sudo systemctl status moosefs-metalogger
```
