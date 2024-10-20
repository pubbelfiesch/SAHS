This part will contain the philosophical questions, motivation, and cursed knowledge for this project

# Overview
| Part | Description & Link|
| ----------- | ----------- |
| 00 | [Overview and ReadMe](https://github.com/pubbelfiesch/SAHS/) |
| 01 | [Alpine installation until the server can be remotely managed via SSH](part_1.md) | 
| 02 | [Setup of local network shares via SMB, permissions, and file transfer from existing HDDs](part_2.md) |
| 03 | [Optional: Setup of docker, a reverse proxy, and a local domain](part_3.md) |
| 04 | [Installation and setup of a VPN, Tailscale, for external access](part_4.md) |
| 05 | [Setup of a secondary machine for off-site backups](part_5.md) |
| 99 | [Philosophical considerations, and cursed knowledge, reflecting why the nas is build as it is](part_99)

# Step 0: Preconsiderations and architecture
Main NAS hardware is an HP ProDesk 600 G4 with Intel 8500T, 32GB ram, 260Gb nvme SSD, 1x 12TB Ironwolf HDD  
Backup server is an HP ProDesk 405 G4 with AMD Ryzen 2200G, 8GB Ram, 128GB SSD, 1x 12TB Ironwolf HDD (recertified)  
ZFS is used as file system, and ZFS replication is used as backup mechanism (to be implemented). No Mirror/raid is used per machine.  
No typical diy NAS operating system is used due to personal preference, mostly to start with a minimalist approach and learn from there.  

To be continued
