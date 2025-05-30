# 🐘 Installation d'un Chunkserver MooseFS (Debian 12)

## 🧾 Prérequis

* 🖥️ Système : **Debian 12**
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
