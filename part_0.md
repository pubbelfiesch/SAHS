This part will contain the philosophical questions and motivations for this project 

# Step 0: Preconsiderations and architecture
Main NAS hardware is an HP ProDesk 600 G4 with Intel 8500T, 32GB ram, 260Gb nvme SSD, 1x 12TB Ironwolf HDD  
Backup server is an HP ProDesk 405 G4 with AMD Ryzen 2200G, 8GB Ram, 128GB SSD, 1x 12TB Ironwolf HDD (recertified)  
ZFS is used as file system, and ZFS replication is used as backup mechanism (to be implemented). No Mirror/raid is used per machine.  
No typical diy NAS operating system is used due to personal preference, mostly to start with a minimalist approach and learn from there.  
