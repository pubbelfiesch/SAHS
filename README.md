# SAHS
Simple Alpine based home server, a.k.a DIY NAS

# Overview
These instructions describe on how to create a simple home server. Features:
- Lightweight linux basis (Alpine)
- Network share for Windows, MacOs, Linux, using SMB, and next to nonexistant rights management
- Backups via a secondary machine outside the local network (to be implemented)
- Docker environment for services, without crazy network settings
  
Most of them are ~~stolen~~ inspired from the excellent guide 3.0 of [dither8](https://dither8.xyz/guide/alpine-linux-nas-3/ "Link to dither8's website"). This git repository extends his work, and backs it up. 

# Step 0: Preconsiderations and architecture
Main NAS hardware is an HP ProDesk 600 G4 with Intel 8500T, 32GB ram, 260Gb nvme SSD, 1x 12TB Ironwolf HDD  
Backup server is an HP ProDesk 405 G4 with AMD Ryzen 2200G, 8GB Ram, 128GB SSD, 1x 12TB Ironwolf HDD (recertified)  
ZFS is used as file system, and ZFS replication is used as backup mechanism (to be implemented). No Mirror/raid is used per machine.  
No typical diy NAS operating system is used due to personal preference, mostly to start with a minimalist approach and learn from there.  
