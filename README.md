# 🐮 Installation et configuration de MooseFS (Client + Chunkserver)

## 🔧 Étapes rapides

```bash
# (Optionnel) Étapes de gestion de disque (à faire si tu veux étendre la partition root)
lvremove /dev/pve/data
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root

# Ajouter les dépôts MooseFS
mkdir -p /etc/apt/keyrings
curl https://repository.moosefs.com/moosefs.key | gpg -o /etc/apt/keyrings/moosefs.gpg --dearmor
echo "deb [arch=amd64 signé par=/etc/apt/keyrings/moosefs.gpg] http://repository.moosefs.com/moosefs-4/apt/debian/bookworm bookworm main" | sudo tee /etc/apt/sources.list.d/moosefs.list

# Mise à jour système
apt update
apt dist-upgrade -y
apt autoremove -y

# Installation des paquets nécessaires
apt install moosefs-client moosefs-chunkserver moosefs-metalogger dfc -y

# Création des dossiers de montage
mkdir -p /mnt/moosefs_chunks
mkdir -p /mnt/moosefs_data
mkdir -p /var/lib/mfs

# Définir les bons droits
chown -R mfschunkserver:mfschunkserver /mnt/moosefs_chunks
chown -R mfschunkserver:mfschunkserver /var/lib/mfs

# Configurer le disque MooseFS (chunk)
nano /etc/mfs/mfshdd.cfg
# ➜ Ajouter : /mnt/moosefs_chunks

# Configurer le fichier du chunkserver si besoin
nano /etc/mfs/mfschunkserver.cfg

# Configurer le nom du master dans /etc/hosts si nécessaire
nano /etc/hosts

# Activer et démarrer les services
systemctl daemon-reload
systemctl enable moosefs-chunkserver.service
systemctl start moosefs-chunkserver.service
systemctl status moosefs-chunkserver.service

# (Optionnel) Activer le metalogger si nécessaire
systemctl enable moosefs-metalogger
systemctl start moosefs-metalogger
systemctl status moosefs-metalogger
